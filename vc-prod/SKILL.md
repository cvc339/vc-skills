# /vc-prod — Manutenção em Produção com Dados Reais

O sistema está em operação. Clientes usam todos os dias. Dados reais estão no banco.
Cada mudança agora carrega risco: um bug pode comprometer dados que o cliente levou meses
para cadastrar. Uma migration mal feita pode apagar registros irrecuperáveis.

Esta skill orienta como trabalhar com segurança em sistemas em produção.

## Filosofia

Em desenvolvimento, o pior que acontece é perder tempo.
Em produção, o pior que acontece é perder dados do cliente.

A regra número um: **nunca executar nada destrutivo em produção sem antes ter certeza
de que pode reverter.**

## Antes de qualquer mudança

### 1. Classificar a mudança

| Tipo | Risco | Exemplos |
|---|---|---|
| **Código frontend** | Baixo | Ajuste visual, novo botão, texto |
| **Código backend (lógica)** | Médio | Nova rota, correção de cálculo, novo endpoint |
| **Código backend (dados)** | Alto | Alterar como dados são salvos/lidos |
| **Migration (aditiva)** | Médio | ADD COLUMN, CREATE TABLE, CREATE INDEX |
| **Migration (destrutiva)** | CRÍTICO | DROP TABLE, DROP COLUMN, ALTER TYPE, DELETE |
| **Migration (RLS)** | Alto | Alterar policies pode bloquear acesso legítimo |

### 2. Backup antes de migrar

Antes de qualquer migration que altera estrutura:

**Supabase (SGI-IDAL):**
```sql
-- Verificar último backup automático no dashboard Supabase
-- Settings > Database > Backups

-- Para tabelas específicas, exportar antes:
-- Dashboard > Table Editor > [tabela] > Export CSV
```

**Para dados críticos, snapshot manual:**
```sql
-- Criar cópia da tabela antes de alterar
CREATE TABLE tabela_backup_20260406 AS SELECT * FROM tabela;

-- Verificar que a cópia está completa
SELECT count(*) FROM tabela;
SELECT count(*) FROM tabela_backup_20260406;
```

**Railway/Express (ACAM):**
```sql
-- Exportar via pg_dump antes de qualquer alteração
-- No terminal com acesso ao banco:
pg_dump -t tabela_nome DATABASE_URL > backup_tabela_20260406.sql
```

### 3. Testar a migration em ambiente de desenvolvimento ANTES

Nunca executar migration diretamente em produção pela primeira vez.

```bash
# Supabase: resetar banco local e testar
supabase db reset  # Recria tudo do zero com todas as migrations

# Se a migration passa no reset local, é segura para produção
```

## Regras para migrations em produção

### Migrations aditivas (seguras)

Estas são seguras e podem ser aplicadas sem medo:

```sql
-- Adicionar coluna (não quebra nada existente)
ALTER TABLE tabela ADD COLUMN IF NOT EXISTS nova_coluna TEXT;

-- Criar tabela nova (não afeta tabelas existentes)
CREATE TABLE IF NOT EXISTS nova_tabela (...);

-- Criar índice (não altera dados)
CREATE INDEX IF NOT EXISTS idx_nome ON tabela(coluna);

-- Adicionar policy RLS (não remove acesso existente)
CREATE POLICY "nome" ON tabela ...;
```

### Migrations destrutivas (PERIGO)

Estas podem causar perda de dados irreversível:

```sql
-- ❌ NUNCA em produção sem backup:
DROP TABLE tabela;              -- Apaga todos os dados
DROP COLUMN coluna;             -- Apaga dados da coluna
ALTER COLUMN coluna TYPE novo;  -- Pode falhar se dados incompatíveis
DELETE FROM tabela WHERE ...;   -- Apaga registros
TRUNCATE tabela;                -- Apaga todos os registros
```

**Se precisar fazer algo destrutivo:**

1. Fazer backup da tabela (CREATE TABLE ... AS SELECT)
2. Testar em desenvolvimento
3. Executar em horário de baixo uso
4. Verificar imediatamente após execução
5. Manter backup por pelo menos 30 dias

### Migrations de rename/refactor

Quando precisa renomear uma coluna sem perder dados:

```sql
-- ERRADO: apaga dados
ALTER TABLE tabela DROP COLUMN nome_antigo;
ALTER TABLE tabela ADD COLUMN nome_novo TEXT;

-- CERTO: preserva dados
ALTER TABLE tabela ADD COLUMN IF NOT EXISTS nome_novo TEXT;
UPDATE tabela SET nome_novo = nome_antigo WHERE nome_novo IS NULL;
-- Depois de confirmar que tudo funciona:
-- ALTER TABLE tabela DROP COLUMN nome_antigo;
```

A coluna antiga só é removida **depois** de confirmar que:
- O código novo funciona
- Os dados foram migrados
- Nenhum código referencia a coluna antiga

### Migrations de RLS

Alterações em RLS podem bloquear acesso legítimo:

```sql
-- Antes de alterar policy, verificar quantos registros ela afeta:
SELECT count(*) FROM tabela; -- Total
-- Simular a nova policy:
SELECT count(*) FROM tabela WHERE [nova_condição]; -- Quantos passam

-- Se o count é muito diferente, a policy vai bloquear registros legítimos
```

## Corrigir bugs em produção

### Fluxo seguro

```
1. Reproduzir o bug em desenvolvimento (NUNCA debugar em produção)
2. Entender a causa raiz (não aplicar band-aid)
3. Escrever a correção
4. Testar localmente
5. Se envolve migration: fazer backup antes
6. Deploy
7. Verificar que o bug foi corrigido E que nada mais quebrou
```

### O que fazer quando o bug afeta dados

Se dados já foram corrompidos/perdidos:

1. **Parar** — não tentar corrigir no impulso
2. **Avaliar o impacto** — quantos registros afetados? Quais clientes?
3. **Verificar backups** — existe backup recente?
4. **Planejar a recuperação** — script SQL para restaurar dados do backup
5. **Executar com verificação** — restaurar e confirmar integridade
6. **Comunicar o cliente** — se ele percebeu, explicar e mostrar que foi resolvido

### Rollback de deploy

Se um deploy causou problema:

**Vercel (SGI-IDAL):**
- Dashboard > Deployments > encontrar deploy anterior > "..." > Promote to Production
- Isso reverte instantaneamente para a versão anterior

**Railway (ACAM):**
- O Railway faz deploy automático a cada push
- Para reverter: `git revert HEAD && git push` (cria commit que desfaz o anterior)
- Ou no dashboard Railway: Deployments > Rollback

## Implementar features novas em produção

### Feature flags (quando a feature é grande)

Se a feature leva mais de um dia para implementar, use feature flags:

```typescript
// Não liberar feature incompleta para todos
const FEATURE_NOVO_RELATORIO = process.env.FEATURE_NOVO_RELATORIO === 'true'

if (FEATURE_NOVO_RELATORIO) {
  // Código novo
} else {
  // Código atual (mantém funcionando)
}
```

No SGI-IDAL, isso já existe no módulo de feature flags (`organizations.modules`).

### Migrations incrementais (nunca big bang)

Adicionar tabelas/colunas novas ANTES de deployar o código que as usa:

```
Deploy 1: Migration que cria tabela nova (código não usa ainda)
Deploy 2: Código que escreve na tabela nova (leitura ainda da antiga)
Deploy 3: Código que lê da tabela nova (se tudo OK)
Deploy 4: Remover código antigo (se confirmado em produção)
```

Isso garante que se o deploy 2 falhar, a tabela já existe e o rollback é seguro.

## Monitoramento pós-deploy

Após cada deploy, verificar:

- [ ] Aplicação está respondendo (acessar a URL principal)
- [ ] Login funciona
- [ ] Feature principal funciona (criar/editar/visualizar um registro)
- [ ] Logs do servidor não mostram erros novos
- [ ] Banco de dados está conectado

**Para o SGI-IDAL:**
- [ ] RLS continua funcionando (dados do org A não aparecem para org B)
- [ ] Edge Functions estão rodando (verificar logs no Supabase)

**Para o ACAM:**
- [ ] Créditos debitam e reembolsam corretamente
- [ ] Análise completa funciona (não dá 502)
- [ ] PDF gera corretamente

## Comunicação com o cliente

Quando uma mudança afeta o que o cliente vê:

| Situação | O que fazer |
|---|---|
| Bug corrigido (sem perda de dados) | Não precisa comunicar — corrigir e seguir |
| Bug corrigido (com dados afetados) | Comunicar o que aconteceu, o que foi feito, e confirmar dados restaurados |
| Feature nova | Comunicar brevemente o que muda |
| Manutenção planejada (downtime) | Avisar com antecedência, informar horário e duração |

## Regras de ouro

1. **Backup antes de migration destrutiva** — sem exceção
2. **Testar em dev antes de produção** — sem exceção
3. **Nunca debugar em produção** — reproduzir localmente
4. **Migrations aditivas primeiro** — coluna nova antes do código que a usa
5. **Rollback sempre possível** — se não sabe como reverter, não faça
6. **Dados do cliente são sagrados** — perder dados é inaceitável
7. **Na dúvida, não faça** — perguntar é melhor que quebrar
