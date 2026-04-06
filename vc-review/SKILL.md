# /vc-review — Verificar Antes de Subir para Produção

Checklist de verificação que rodamos antes de cada deploy. Não é auditoria profunda —
é a última barreira entre o código e os usuários. Rápido, objetivo, focado no que quebra.

## Quando usar

- Antes de fazer push/deploy
- Após uma sessão com muitas alterações
- Quando houver mudanças em banco, rotas ou cálculos

## Checklist

### Schema e Banco
- [ ] Toda tabela referenciada em queries existe (CREATE TABLE no migration)
- [ ] Toda coluna referenciada em INSERT/UPDATE/SELECT existe na tabela
- [ ] Nomes de colunas consistentes em todo o projeto
- [ ] SQL usa sintaxe PostgreSQL (NOW(), INTERVAL, não SQLite)
- [ ] Migrations são idempotentes (IF NOT EXISTS)
- [ ] RLS ativo em tabelas com dados de usuários

### Código
- [ ] Todo try/catch responde ao cliente (nunca deixa request pendurada)
- [ ] Variáveis usadas no catch estão no escopo correto (let fora do try)
- [ ] Imports apontam para arquivos que existem
- [ ] Rotas novas registradas no server/app (app.use ou next.config)

### Segurança
- [ ] .env/.env.local não está commitado
- [ ] Nenhum secret hardcoded no código
- [ ] Rotas protegidas exigem autenticação
- [ ] Tokens JWT têm expiração

### Operações financeiras (se aplicável)
- [ ] Créditos debitados têm reembolso em caso de erro
- [ ] Transação registrada no histórico
- [ ] Mensagem clara ao usuário sobre créditos insuficientes

### Interface (se houve mudanças visuais)
- [ ] Componentes usam design system (não HTML/CSS arbitrário)
- [ ] Cores semânticas para status
- [ ] Responsivo (funciona em mobile)
- [ ] Loading states tratados
- [ ] PDF funciona onde exigido (se aplicável)

### Cálculos regulatórios (se houve mudanças)
- [ ] Fórmulas têm testes unitários
- [ ] Referência legal documentada no código
- [ ] Disclaimers presentes nos resultados
- [ ] Valores atualizáveis vêm da API/banco (não hardcoded)

### Documentação
- [ ] docs/claude-context/ atualizado se houve mudança significativa
- [ ] ARCHITECTURE.md atualizado se houve mudança de schema
- [ ] 05_DECISOES.md atualizado se houve decisão técnica

## Formato

```
PRÉ-DEPLOY: [projeto] — [data]

[X] Schema         OK
[X] Código         OK
[X] Segurança      OK
[ ] Financeiro     1 issue
[X] Interface      OK
[X] Cálculos       OK
[X] Documentação   OK

ISSUES:
1. [CRÍTICO] consultas.js:230 — consultaId fora de escopo no catch

VEREDICTO: Corrigir 1 issue antes de subir
```

## Severidades

- **CRÍTICO**: Pode causar erro em produção (schema, crash, perda de dados/créditos). Bloqueia deploy.
- **ALTO**: Funcionalidade quebrada mas não crasha o servidor. Corrigir antes se possível.
- **MÉDIO**: Inconsistência visual ou documentação desatualizada. Pode subir, registrar para corrigir.
- **BAIXO**: Melhoria desejável. Não bloqueia nada.

## Regras

- Schema inconsistente é sempre CRÍTICO — já causou 502 em produção
- Créditos sem reembolso é sempre CRÍTICO
- Não bloqueie deploy por issues BAIXO
- Se tudo está limpo: "Pode subir." — uma frase e pare
