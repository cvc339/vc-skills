# /vc-data — Modelar Banco de Dados

Esta skill define o schema do banco ANTES de qualquer feature ser construída.
O resultado é um conjunto de migrations SQL versionadas, com RLS configurado,
índices criados, e triggers prontos.

## Filosofia

O banco de dados é a fundação. Se a modelagem está errada, todo o código acima dela
será remendo. Definir o schema com cuidado no início economiza semanas de refatoração.

Lições aprendidas no ACAM (o que NÃO fazer):
- Schema criado inline no initSchema() sem versionamento
- Colunas referenciadas no código que não existiam no banco (502 em produção)
- Nomes misturados (created_at vs. criado_em)
- Funções SQLite em banco PostgreSQL
- Tabelas faltantes descobertas apenas quando o usuário tentava usar

## Stack obrigatória

- **PostgreSQL** via Supabase
- **Migrations SQL** versionadas em `supabase/migrations/`
- **RLS** (Row Level Security) em toda tabela que contém dados de usuários
- **Triggers** para consistência de dados (organization_id redundante)

## Modelo Multi-Tenant (quando aplicável)

Se o projeto atende múltiplas organizações:

### Nível 1 — Organizacional
- Âncora: `organization_id`
- Dados compartilhados na organização
- RLS: `organization_id = get_user_organization_id()`
- Colunas obrigatórias: `id UUID PK`, `organization_id`, `created_by`, `created_at`, `updated_at`

### Nível 2 — Contextual (ex: licença, projeto, caso)
- Âncora: `[contexto]_id` + `organization_id` redundante
- Dados vinculados a uma entidade específica
- RLS: via `organization_id` (redundante, populado por trigger)
- Trigger: `set_organization_from_[contexto]()` obrigatório

### Nível 3 — Pessoal
- Âncora: `user_id`
- Dados não compartilhados
- RLS: `user_id = auth.uid()`

### Modelo Simplificado (single-tenant)
Se o projeto não precisa de multi-tenancy, usar N1 simplificado:
- Toda tabela com `user_id` para controle de acesso
- RLS com `user_id = auth.uid()`
- Sem organization_id, sem triggers

## Processo de criação de tabela

### 1. Classificar a tabela
- É dados da organização? → N1
- É dados vinculados a uma entidade (licença, projeto)? → N2
- É dados pessoais do usuário? → N3
- É referência pública (lista IBAMA, tipos)? → Pública

### 2. Criar migration

Arquivo: `supabase/migrations/YYYYMMDDHHMMSS_create_nome_tabela.sql`

Estrutura obrigatória:
```sql
-- Tipo: N1 | N2 | N3 | Pública
-- Tabela: nome_tabela
-- Descrição: propósito claro em uma frase

BEGIN;

CREATE TABLE IF NOT EXISTS nome_tabela (
  id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  -- [âncora conforme nível] --
  -- [campos específicos] --
  created_by      UUID REFERENCES auth.users(id),
  created_at      TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  updated_at      TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- Índices (obrigatório para FKs usadas em RLS)
CREATE INDEX IF NOT EXISTS idx_nome_tabela_[campo] ON nome_tabela([campo]);

-- RLS
ALTER TABLE nome_tabela ENABLE ROW LEVEL SECURITY;

-- Políticas (SELECT, INSERT com WITH CHECK, UPDATE, DELETE)
DROP POLICY IF EXISTS "select_policy" ON nome_tabela;
CREATE POLICY "select_policy" ON nome_tabela FOR SELECT
  USING ([condição]);

DROP POLICY IF EXISTS "insert_policy" ON nome_tabela;
CREATE POLICY "insert_policy" ON nome_tabela FOR INSERT
  WITH CHECK ([condição]);

-- ... UPDATE e DELETE seguem o mesmo padrão

COMMIT;

-- ROLLBACK (comentado, para referência)
-- DROP TABLE IF EXISTS nome_tabela;
```

### 3. Regras obrigatórias

1. **Toda migration é idempotente**: DROP IF EXISTS, ADD COLUMN IF NOT EXISTS, CREATE INDEX IF NOT EXISTS
2. **Toda política INSERT tem WITH CHECK**: nunca NULL
3. **user_id NUNCA em RLS de tabelas corporativas**: usar organization_id
4. **organization_id redundante exige trigger**: frontend não envia, trigger resolve
5. **Índice em toda FK usada por RLS**: performance obrigatória
6. **Nomes consistentes**: snake_case, em inglês para colunas estruturais (id, created_at, updated_at), português permitido para colunas de domínio

### 4. Validar após criação

Queries de verificação:
```sql
-- Policies INSERT sem WITH CHECK (deve retornar 0)
SELECT tablename, policyname FROM pg_policies
WHERE schemaname = 'public' AND cmd = 'INSERT' AND with_check IS NULL;

-- Tabelas sem RLS (deve retornar apenas tabelas públicas)
SELECT tablename FROM pg_tables
WHERE schemaname = 'public' AND NOT rowsecurity;
```

### 5. Documentar

Atualizar `docs/claude-context/03_BANCO_DADOS.md` com:
- Nome da tabela, tipo (N1/N2/N3), propósito
- Colunas e tipos
- Relacionamentos
- Políticas RLS

## Anti-patterns proibidos

| Errado | Correto |
|--------|---------|
| Schema inline no código (initSchema) | Migrations SQL versionadas |
| Colunas sem verificação de existência | ALTER TABLE ADD COLUMN IF NOT EXISTS |
| created_at em algumas, criado_em em outras | Nomenclatura consistente em todo o projeto |
| Funções SQLite (datetime()) em PostgreSQL | NOW(), INTERVAL do PostgreSQL |
| Tabela referenciada sem existir | Toda tabela criada em migration antes do uso |
| RLS desabilitado "por enquanto" | RLS ativo desde a criação da tabela |
| INSERT sem WITH CHECK | WITH CHECK coerente com USING |

## Regras

- Criar o schema ANTES de qualquer código que faz queries
- Toda tabela nova = nova migration (não editar migrations antigas)
- Testar localmente: `supabase db reset` deve recriar tudo sem erros
- Se não sabe o nível (N1/N2/N3): perguntar antes de implementar
- O banco é a fonte de verdade. O código se adapta ao banco, não o contrário.
