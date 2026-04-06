# /vc-plan — Planejar Antes de Construir

Esta é a primeira skill a ser usada. Antes de qualquer código, antes do init, antes do design.
Planejar é o que separa um projeto profissional de um protótipo que vira produção por acidente.

## Filosofia

O maior erro em projetos de software não é código ruim — é construir a coisa errada,
ou construir a coisa certa sem entender o domínio. Código se refatora. Arquitetura errada
se reescreve. Mas um projeto que não entendeu o problema do usuário é irrecuperável.

As melhores práticas da indústria (Domain-Driven Design, Architecture Decision Records,
Requirements Engineering) convergem em um ponto: **entender antes de construir**.

## O que esta skill produz

Ao final do planejamento, você terá:

1. **Documento de Visão** — O que é, para quem, qual problema resolve
2. **Mapa de Domínio** — Entidades, relacionamentos, regras de negócio
3. **Lista de Módulos** — Organizados por prioridade (MVP → V2 → futuro)
4. **Decisões Arquiteturais** — Stack, multi-tenancy, auth, deploy — com justificativas
5. **Riscos Identificados** — O que pode dar errado e como mitigar

## Processo

### Fase 1: Entender o Problema

Antes de pensar em tecnologia, responder:

1. **Quem usa?** Perfil do usuário (advogado, engenheiro, gestor, todos?)
2. **Qual o problema?** O que o usuário faz hoje que é ruim/lento/inseguro?
3. **Qual o valor?** Por que alguém pagaria/usaria isso?
4. **Qual o domínio regulatório?** (se aplicável) Quais leis/normas regem o processo?
5. **Quem são os stakeholders?** Órgãos, clientes, parceiros que interagem com o sistema?

Documentar em: `docs/claude-context/00_VISAO_GERAL.md`

### Fase 2: Modelar o Domínio

Não é modelar o banco de dados. É modelar o **mundo real** que o software representa.

Para cada entidade do domínio:
- **O que é?** (definição em uma frase)
- **Quem cria?** (qual ator)
- **O que contém?** (atributos principais)
- **Como se relaciona?** (com quais outras entidades)
- **Quais estados tem?** (pendente → ativo → concluído → arquivado)
- **Quais regras de negócio se aplicam?** (validações, cálculos, restrições)

Exemplo do SGI-IDAL:
```
Condicionante
  - É uma obrigação ambiental vinculada a uma licença
  - Criada pelo operador ao cadastrar a licença
  - Contém: descrição, tipo (periódica/contínua/única), frequência, prazo
  - Relaciona-se com: licença (N:1), entregas (1:N), evidências (1:N)
  - Estados: vigente, cumprida, vencida, em análise
  - Regras: gera ciclos automáticos baseado na frequência e validade da licença
```

Documentar em: `docs/claude-context/02_REGRAS_NEGOCIO.md`

### Fase 3: Definir Módulos e Prioridades

Listar TODOS os módulos imaginados, depois classificar:

| Prioridade | Critério | Exemplos |
|---|---|---|
| **MVP** | Sem isso o produto não existe | Auth, dashboard, funcionalidade core |
| **V1** | Necessário para o primeiro cliente | Relatórios, exportação, alertas |
| **V2** | Diferencial competitivo | Integrações, API pública, mobile |
| **Futuro** | Bom ter, não urgente | Marketplace, analytics avançado |

Para cada módulo:
- Nome e descrição em uma frase
- Tabelas principais envolvidas
- Nível de multi-tenancy (N1/N2/N3)
- Dependências de outros módulos

### Fase 3b: Plano de Execução

Transformar a lista de módulos em um plano concreto com fases, entregas e critérios de avanço.

**Estrutura do plano:**

```markdown
## Fase 0 — Fundação (antes de qualquer feature)
Skills: /vc-init → /vc-design → /vc-data (tabelas core)
Entregas:
  - [ ] Projeto criado e rodando
  - [ ] Design system aprovado (/styleguide funcionando)
  - [ ] Tabelas core criadas com RLS
  - [ ] Auth funcionando (login, registro, middleware)
  - [ ] ARCHITECTURE.md e CLAUDE.md escritos
Critério para avançar: Build funcional, auth OK, styleguide aprovado.

## Fase 1 — MVP (funcionalidade mínima para o primeiro uso real)
Módulos: [listar]
Entregas:
  - [ ] [Módulo A] CRUD completo
  - [ ] [Módulo B] Cadastro e listagem
  - [ ] Dashboard com visão geral
  - [ ] /vc-security: isolamento verificado
Critério para avançar: Um usuário real consegue completar o fluxo principal.

## Fase 2 — V1 (pronto para o primeiro cliente)
Módulos: [listar]
Entregas:
  - [ ] [Módulo C] completo
  - [ ] Relatórios / exportações
  - [ ] Alertas por email
  - [ ] /vc-security: re-verificação completa
  - [ ] /vc-review: checklist pré-deploy aprovado
Critério para avançar: Cliente pode usar sem acompanhamento constante.

## Fase 3 — V2 (evolução pós-feedback)
Módulos: [listar]
Entregas:
  - [ ] Features baseadas em feedback real
  - [ ] Integrações
  - [ ] Otimizações de performance
```

**Regras do plano:**
- Cada fase tem entregas concretas (checkboxes), não descrições vagas
- Cada fase tem critério objetivo para avançar
- Dependências entre módulos determinam a ordem dentro da fase
- O plano é um documento vivo — atualizar conforme o projeto evolui

Documentar em: `docs/claude-context/PLANO_EXECUCAO.md`

Este documento é o mapa do projeto. Ao iniciar cada sessão de trabalho,
o Claude lê o plano e sabe onde o projeto está e o que fazer em seguida.

### Fase 4: Decisões Arquiteturais

Para cada decisão técnica importante, registrar:

```markdown
### [data] Título da Decisão

**Contexto:** Por que precisamos decidir isso?
**Decisão:** O que decidimos e por quê
**Alternativas:** O que mais consideramos
**Consequências:** O que isso implica (positivo e negativo)
```

Decisões obrigatórias a tomar:
1. **Multi-tenancy?** Uma organização ou várias? Se várias: como isolar dados?
2. **Auth:** Supabase Auth, Auth.js, custom JWT?
3. **Deploy:** Vercel, Railway, AWS, outro?
4. **Pagamentos?** Se sim: Stripe, Mercado Pago, outro?
5. **Email:** Resend, SendGrid, SES?
6. **Armazenamento de arquivos:** Supabase Storage, S3, local?

Documentar em: `docs/claude-context/05_DECISOES.md`

### Fase 5: Identificar Riscos

O que pode dar errado? Para cada risco:

| Risco | Probabilidade | Impacto | Mitigação |
|---|---|---|---|
| Vazamento de dados entre tenants | Alta (se não testar RLS) | Crítico | /vc-security após cada migration |
| API externa indisponível | Média | Alto | Timeout + fallback + reembolso |
| Schema inconsistente | Alta (se sem migrations) | Crítico | /vc-data com migrations versionadas |
| Norma legal desatualizada | Baixa | Alto | Documentar versão da norma, verificar anualmente |

Lições do ACAM (riscos que se materializaram):
- Tabela referenciada que não existia → 502 em produção
- Créditos debitados sem reembolso em caso de erro → perda financeira do usuário
- Colunas com nomes inconsistentes → queries falhando silenciosamente
- Funções SQLite em banco PostgreSQL → incompatibilidade não detectada

Lições do SGI-IDAL:
- 9 tabelas sem RLS por meses → dados potencialmente expostos entre organizações
- View com SECURITY DEFINER → bypass de RLS

## Fluxo completo de um projeto

```
/vc-plan          Entender, modelar, decidir, documentar
    ↓
/vc-init          Criar projeto com estrutura profissional
    ↓
/vc-design        Criar design system antes de qualquer página
    ↓
/vc-data          Modelar banco com migrations e RLS
    ↓
[construir]       Features, usando /vc-norma para legislação
    ↓
/vc-security      Verificar isolamento de dados
    ↓
/vc-review        Checklist pré-deploy
    ↓
deploy
    ↓
[iterar]          Novas features → /vc-data → build → /vc-security → /vc-review → deploy
```

## Regras

- Não pular para código sem ter 00_VISAO_GERAL e 02_REGRAS_NEGOCIO escritos
- Não definir tabelas sem ter modelado o domínio primeiro
- Toda decisão tem justificativa documentada — "porque sim" não é justificativa
- Se o fundador não consegue explicar o módulo em uma frase, o módulo não está definido
- Riscos identificados antes viram prevenção. Riscos descobertos depois viram incidentes.
