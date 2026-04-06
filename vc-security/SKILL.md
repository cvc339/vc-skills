# /vc-security — Verificar Isolamento e Segurança de Dados

Esta skill existe por causa de uma experiência real: no SGI-IDAL, após meses de desenvolvimento,
descobrimos que 9 tabelas com dados de clientes não tinham RLS. Qualquer usuário autenticado
poderia acessar dados de outras organizações. O problema só foi encontrado por acaso.

Isso nunca mais pode acontecer. Esta skill é a verificação sistemática que previne vazamentos.

## Quando usar

- Após criar novas tabelas (migration)
- Após modificar políticas RLS
- Antes de cada deploy para produção
- Periodicamente (a cada sprint/ciclo)
- Quando adicionar novo módulo ao sistema

## Verificações

### 1. Tabelas sem RLS

Toda tabela com dados de usuários/organizações DEVE ter RLS habilitado.

```sql
-- Tabelas sem RLS (deve retornar apenas tabelas de referência pública)
SELECT schemaname, tablename
FROM pg_tables
WHERE schemaname = 'public'
  AND NOT rowsecurity
ORDER BY tablename;
```

**Resultado esperado:** Apenas tabelas de referência (tipos, categorias IBAMA, etc.)
**Se retornar tabela com dados de usuário/organização:** CRÍTICO — habilitar RLS imediatamente.

### 2. Políticas INSERT sem WITH CHECK

```sql
-- Deve retornar 0 linhas
SELECT tablename, policyname
FROM pg_policies
WHERE schemaname = 'public'
  AND cmd = 'INSERT'
  AND with_check IS NULL;
```

**Se retornar qualquer linha:** CRÍTICO — usuário pode inserir dados em organização alheia.

### 3. Views com SECURITY DEFINER

```sql
-- Views que bypassam RLS (verificar se é intencional)
SELECT viewname, definition
FROM pg_views
WHERE schemaname = 'public';
```

Verificar se alguma view usa `SECURITY DEFINER` em vez de `SECURITY INVOKER`.
- DEFINER = executa com permissões do criador (bypassa RLS)
- INVOKER = executa com permissões de quem consulta (respeita RLS)

**Usar INVOKER como padrão.** DEFINER apenas se houver justificativa documentada.

### 4. Consistência de organization_id

Para tabelas N2 com organization_id redundante:

```sql
-- Inconsistências (deve retornar 0)
-- Adaptar para as tabelas do projeto
SELECT t.id, t.organization_id as org_tabela, e.organization_id as org_real
FROM tabela_n2 t
JOIN licenses l ON l.id = t.license_id
JOIN enterprises e ON e.id = l.enterprise_id
WHERE t.organization_id != e.organization_id;
```

**Se retornar linhas:** trigger `set_organization_from_license` não está funcionando.

### 5. FKs sem índice usadas em RLS

```sql
-- Verificar que FKs usadas em EXISTS nas policies têm índice
SELECT
    t.relname AS table_name,
    a.attname AS column_name
FROM pg_constraint c
JOIN pg_class t ON t.oid = c.conrelid
JOIN pg_attribute a ON a.attrelid = t.oid AND a.attnum = ANY(c.conkey)
WHERE c.contype = 'f'
  AND t.relnamespace = 'public'::regnamespace
ORDER BY t.relname, a.attname;
```

Comparar com índices existentes. Toda FK usada em policy RLS precisa de índice.

### 6. Teste de vazamento (o mais importante)

Este teste simula um usuário tentando acessar dados de outra organização.

**Manualmente (via Supabase SQL Editor):**

```sql
-- 1. Identificar duas organizações diferentes
SELECT id, name FROM organizations LIMIT 5;

-- 2. Identificar um usuário de cada organização
SELECT u.id, u.email, ou.organization_id
FROM auth.users u
JOIN organization_users ou ON ou.user_id = u.id
LIMIT 10;

-- 3. Com um usuário da org A, tentar acessar dados da org B
-- (executar como o usuário via set_config)
SELECT set_config('request.jwt.claims', '{"sub": "[user_id_org_A]"}', true);

-- 4. Consultar cada tabela sensível
SELECT count(*) FROM tabela_n1; -- Deve retornar apenas dados da org A
SELECT count(*) FROM tabela_n2; -- Idem
```

**Se qualquer consulta retornar dados da org B:** CRÍTICO — RLS não está funcionando.

### 7. Secrets e configuração

- [ ] `.env` / `.env.local` no `.gitignore`
- [ ] Nenhum secret hardcoded no código (grep por `sk-`, `key=`, `password=`, `secret=`)
- [ ] Service role key usada apenas server-side (nunca no cliente)
- [ ] `NEXT_PUBLIC_` apenas em variáveis que podem ser públicas

### 8. Autenticação

- [ ] Middleware protege rotas do dashboard
- [ ] Rotas de API validam sessão
- [ ] Token expirado redireciona para login
- [ ] Logout limpa sessão completamente

## Formato do output

```
VERIFICAÇÃO DE SEGURANÇA: [projeto] — [data]

TABELAS SEM RLS:     [X] OK (0 expostas) / [ ] N tabelas expostas
POLICIES INSERT:     [X] OK (0 sem WITH CHECK) / [ ] N vulneráveis
VIEWS:               [X] OK (todas INVOKER) / [ ] N com DEFINER
CONSISTÊNCIA ORG_ID: [X] OK (0 inconsistências) / [ ] N linhas
ÍNDICES FK/RLS:      [X] OK / [ ] N faltando
TESTE VAZAMENTO:     [X] OK (isolamento confirmado) / [ ] VAZAMENTO DETECTADO
SECRETS:             [X] OK / [ ] N expostos
AUTH:                [X] OK / [ ] N issues

ISSUES:
1. [CRÍTICO] tabela X sem RLS — dados de N orgs expostos
2. ...

VEREDICTO: Seguro para produção / N issues CRÍTICOS a resolver
```

## Severidades

- **Vazamento de dados entre organizações:** Sempre CRÍTICO. Não negocie.
- **Policy sem WITH CHECK:** CRÍTICO — permite inserção em org alheia.
- **View com DEFINER:** ALTO — pode ser intencional, mas verificar.
- **FK sem índice:** MÉDIO — funciona, mas degrada performance.
- **Secret em log:** ALTO — pode ter sido indexado.

## Regras

- Rodar esta verificação após CADA migration que cria ou altera tabela
- Vazamento de dados é inaceitável — mesmo em desenvolvimento
- "Vou arrumar depois" não existe para segurança de dados — arrumar agora
- Se uma tabela não precisa de RLS, documentar por que (tabela pública de referência)
- O teste de vazamento (item 6) é o mais importante — os outros são preparatórios
