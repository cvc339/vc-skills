# Manual de Uso — vc-skills
## Padrões Vieira Castro para Claude Code

**Autor:** Construído em parceria com Claude Code  
**Data:** Abril/2026  
**Versão:** 1.0  

---

## Passo a passo para projeto novo

### 1. Criar o repositório no GitHub

- Acesse **github.com/new**
- Nome do projeto (ex: `sgi-outorgas`)
- Visibilidade: **Private**
- Clique **Create repository**
- **Não** marcar README nem .gitignore (o `/vc-init` cria tudo)

### 2. Clonar na pasta de projetos

No Claude Code:
```
Você: Clona o repositório github.com/cvc339/sgi-outorgas em C:\Users\User\projetos
```

### 3. Abrir o projeto no VS Code

- Abra a pasta do projeto clonado no VS Code
- Abra o Claude Code **nessa pasta** (importante: o Claude precisa estar na pasta do projeto)

### 4. Rodar as skills na sequência

```
Você: /vc-plan
Você: [descreve o projeto, domínio, público-alvo]
...planejamento concluído...

Você: /vc-init
...projeto criado...

Você: /vc-design
...design system criado...

Você: /vc-data
...banco modelado...

[construir features]
```

### 5. Commit e push

Cada commit e push vai direto para o repositório do projeto.
O Claude faz isso quando você pede ou quando faz sentido no fluxo.

### Onde cada coisa fica

| O que | Onde fica |
|---|---|
| **Skills** (ferramentas de orientação) | `github.com/cvc339/vc-skills` → instaladas em `~/.claude/skills/` |
| **Projeto novo** (código) | `github.com/cvc339/nome-projeto` → clonado em `C:\Users\User\projetos\nome-projeto` |

**As skills orientam. O projeto é onde o código vive.** São repositórios separados.

---

## O que são as vc-skills?

São 9 conjuntos de instruções que orientam o Claude Code a trabalhar seguindo
padrões profissionais em todas as fases de um projeto de software.

Pense nelas como uma **caixa de ferramentas**. Cada skill é uma ferramenta
com propósito específico. Você escolhe qual usar conforme a tarefa.

### Onde estão instaladas

```
C:\Users\User\.claude\skills\vc-skills\
├── SKILL.md              ← Índice geral
├── setup                 ← Script de instalação
├── vc-plan\SKILL.md      ← Planejamento
├── vc-init\SKILL.md      ← Iniciar projeto
├── vc-design\SKILL.md    ← Design system
├── vc-data\SKILL.md      ← Banco de dados
├── vc-code\SKILL.md      ← Padrões de código
├── vc-norma\SKILL.md     ← Legislação
├── vc-security\SKILL.md  ← Segurança
├── vc-review\SKILL.md    ← Pré-deploy
└── vc-prod\SKILL.md      ← Produção
```

---

## Como usar

### Regra principal

**As skills NÃO são automáticas.** Você precisa chamar cada uma quando for necessário,
digitando o nome no chat com o Claude Code. Exemplo:

```
Você: /vc-plan
```

Quando você digita isso, o Claude Code carrega as instruções daquela skill e passa
a seguir aquele padrão durante a conversa. É como dizer: "agora trabalhe no modo
planejamento" ou "agora trabalhe no modo segurança".

### Quando chamar cada skill

Não precisa decorar — basta seguir o fluxo natural do projeto:

```
ESTOU COMEÇANDO UM PROJETO NOVO
  → /vc-plan (primeiro sempre)
  → /vc-init (depois de planejar)
  → /vc-design (antes de qualquer tela)
  → /vc-data (antes de qualquer código backend)

ESTOU CONSTRUINDO FEATURES
  → /vc-code (quando quiser lembrar os padrões)
  → /vc-norma (quando a feature envolver legislação ou cálculos)

ESTOU PRONTO PARA SUBIR
  → /vc-security (se mexeu no banco ou em RLS)
  → /vc-review (sempre antes de deploy)

O SISTEMA ESTÁ EM PRODUÇÃO E PRECISO MEXER
  → /vc-prod (antes de qualquer alteração em sistema com dados reais)
```

---

## Guia detalhado de cada skill

### 1. /vc-plan — Planejar Antes de Construir

**Quando usar:** Sempre que for iniciar um projeto novo.

**O que faz:** Orienta a definir:
- Para quem é o projeto e qual problema resolve
- Quais são as entidades do domínio (ex: licença, condicionante, empresa)
- Quais módulos existem e qual a prioridade de cada um
- Quais decisões técnicas tomar (banco, auth, deploy)
- Quais riscos existem e como prevenir

**Como usar na prática:**

```
Você: /vc-plan
Você: Quero construir um sistema para gestão de outorgas de recursos hídricos.
      O público são empresas que possuem outorgas e precisam monitorar vencimentos,
      condicionantes e relatórios periódicos.

Claude: [vai seguir o processo da skill: entender o domínio, listar entidades,
        propor módulos, sugerir decisões técnicas, identificar riscos]
```

**Resultado esperado:** Documentos `00_VISAO_GERAL.md`, `02_REGRAS_NEGOCIO.md`
e `05_DECISOES.md` prontos.

---

### 2. /vc-init — Iniciar Projeto

**Quando usar:** Depois do planejamento estar pronto.

**O que faz:** Cria toda a estrutura do projeto:
- Projeto Next.js com TypeScript
- Supabase configurado
- Estrutura de pastas profissional
- Middleware de autenticação
- Documentação (ARCHITECTURE.md, CLAUDE.md, docs/claude-context/)
- Build funcional com um comando

**Como usar na prática:**

```
Você: /vc-init
Você: Vamos criar o projeto. O planejamento está em docs/claude-context/.
      O nome do projeto é sgi-outorgas.

Claude: [vai criar o projeto seguindo todo o checklist da skill]
```

**Resultado esperado:** Projeto rodando com `npm run dev`, estrutura completa.

---

### 3. /vc-design — Criar Design System

**Quando usar:** Logo após o init, ANTES de criar qualquer página.

**O que faz:** Cria um styleguide vivo dentro do projeto:
- Paleta de cores (primária, cinza, semânticas)
- Tipografia com hierarquia definida
- Componentes shadcn/ui instalados com showcase
- Página /styleguide acessível no navegador

**Como usar na prática:**

```
Você: /vc-design
Você: O projeto é de gestão ambiental. Quero uma identidade institucional,
      profissional, cor primária verde-oliva. O público são gestores e engenheiros.

Claude: [vai criar o design system completo com tokens, componentes e showcases]
```

**Resultado esperado:** Acessar `localhost:3000/styleguide` e ver toda a identidade visual.

**Uso posterior (durante o projeto):**

Quando precisar de um componente novo:
```
Você: Preciso de um componente de Timeline para mostrar histórico de eventos.
      Use /vc-design para adicionar ao styleguide primeiro.
```

---

### 4. /vc-data — Modelar Banco de Dados

**Quando usar:** Antes de escrever código backend. Após definir o domínio.

**O que faz:** Orienta a criar:
- Migrations SQL versionadas
- RLS (Row Level Security) em toda tabela
- Triggers para consistência de dados
- Índices em colunas usadas por RLS

**Como usar na prática:**

```
Você: /vc-data
Você: Preciso criar a tabela de outorgas. Ela pertence a um empreendimento
      que pertence a uma organização. Cada outorga tem número, órgão emissor,
      data de emissão, validade, e status.

Claude: [vai criar a migration SQL seguindo o padrão multi-tenant com RLS]
```

**Resultado esperado:** Arquivo em `supabase/migrations/` pronto para deploy.

---

### 5. /vc-code — Padrões de Codificação

**Quando usar:** Sempre que estiver construindo features. Pode chamar
no início de uma sessão de desenvolvimento para "ativar" o modo sênior.

**O que faz:** Define como escrever código profissional:
- Um arquivo, uma responsabilidade
- Funções puras para lógica de negócio
- Error handling que preserva contexto
- Nomes que explicam
- Sem duplicação, sem magic numbers

**Como usar na prática:**

```
Você: /vc-code
Você: Vou implementar o cálculo de vencimento de outorgas. O prazo é
      calculado com base na data de emissão e na duração em anos.

Claude: [vai implementar seguindo os padrões: função pura, testável,
        com nomes claros e sem hardcoded]
```

---

### 6. /vc-norma — Legislação e Cálculos

**Quando usar:** Quando a feature envolver legislação ambiental, fórmulas
regulatórias, referências legais, ou cálculos oficiais.

**O que faz:** Orienta a:
- Isolar fórmulas em funções puras com testes
- Documentar a base legal no código
- Incluir disclaimers obrigatórios
- Tratar valores atualizáveis (UFEMG, etc.)
- Usar formatos brasileiros (moeda, data, unidades)

**Como usar na prática:**

```
Você: /vc-norma
Você: Preciso implementar o cálculo de vazão outorgada conforme a
      Resolução ANA 707/2004. A fórmula é...

Claude: [vai implementar com base legal documentada, testes, e disclaimer]
```

---

### 7. /vc-security — Verificar Segurança

**Quando usar:**
- Depois de criar tabelas novas
- Depois de alterar políticas RLS
- Antes de cada deploy importante
- Periodicamente (a cada 2-3 semanas)

**O que faz:** Executa verificações de isolamento:
- Tabelas sem RLS
- Policies INSERT sem WITH CHECK
- Views com SECURITY DEFINER
- Teste de vazamento entre organizações

**Como usar na prática:**

```
Você: /vc-security
Você: Acabei de criar 3 tabelas novas para o módulo de outorgas.
      Verifica se o isolamento está correto.

Claude: [vai rodar as queries de verificação e reportar findings]
```

---

### 8. /vc-review — Checklist Pré-Deploy

**Quando usar:** Sempre antes de fazer push/deploy.

**O que faz:** Verifica rapidamente:
- Schema vs. código (tabelas e colunas existem?)
- Error handling (catch responde ao cliente?)
- Segurança (secrets protegidos?)
- Interface (design system seguido?)
- Documentação atualizada?

**Como usar na prática:**

```
Você: /vc-review
Você: Fiz várias alterações hoje. Verifica se está tudo certo para subir.

Claude: [vai rodar o checklist e dar veredicto: "pode subir" ou "corrigir X issues"]
```

---

### 9. /vc-prod — Manutenção em Produção

**Quando usar:** Sempre que for mexer num sistema que já tem dados reais de clientes.

**O que faz:** Orienta a:
- Classificar o risco da mudança
- Fazer backup antes de migrations
- Testar em dev antes de produção
- Deploy incremental (adicionar antes, remover depois)
- Rollback se algo der errado
- Verificar após cada deploy

**Como usar na prática:**

```
Você: /vc-prod
Você: Preciso adicionar uma coluna na tabela de outorgas do SGI que
      já está em produção com dados reais. Como proceder?

Claude: [vai orientar: backup → migration aditiva → teste → deploy → verificar]
```

---

## Cenários práticos completos

### Cenário 1: Projeto novo do zero

```
Sessão 1 — Planejamento:
  /vc-plan
  "Quero criar um sistema de gestão de outorgas..."
  [resultado: documentação do domínio, módulos, decisões]

Sessão 2 — Fundação:
  /vc-init
  "Cria o projeto sgi-outorgas conforme o planejamento"
  [resultado: projeto Next.js funcional]

  /vc-design
  "Define o design system. Identidade verde-água, público técnico."
  [resultado: /styleguide com todos os componentes]

Sessão 3 — Banco de dados:
  /vc-data
  "Cria as tabelas: organizations, enterprises, outorgas, condicionantes"
  [resultado: migrations com RLS]

  /vc-security
  "Verifica o isolamento"
  [resultado: confirmação de segurança]

Sessão 4+ — Features:
  /vc-code
  "Implementa o módulo de cadastro de outorgas"
  [resultado: código profissional]

  /vc-review
  "Verifica antes de subir"
  [resultado: "pode subir" ou issues a corrigir]
```

### Cenário 2: Correção de bug em produção

```
  /vc-prod
  "O sistema está dando erro ao salvar nova outorga.
   No console aparece: column 'tipo_outorga' does not exist"

  [Claude vai:
   1. Identificar que a coluna falta no banco
   2. Criar migration aditiva (ADD COLUMN IF NOT EXISTS)
   3. Orientar backup antes
   4. Testar localmente
   5. Fazer deploy
   6. Verificar após deploy]
```

### Cenário 3: Feature nova em sistema em produção

```
  /vc-prod
  "Preciso adicionar módulo de alertas de vencimento no SGI que
   já está em produção com 5 clientes usando."

  /vc-data
  "Cria as tabelas de alertas"

  /vc-security
  "Verifica isolamento das tabelas novas"

  /vc-code
  "Implementa a Edge Function de envio de alertas"

  /vc-review
  "Verifica antes de subir"
```

---

## Dicas importantes

### 1. Não precisa usar todas em toda sessão

Se está apenas ajustando um texto na interface, não precisa chamar nenhuma skill.
Chame quando a tarefa justificar.

### 2. Pode combinar skills na mesma sessão

```
Você: /vc-data
Você: Cria a tabela X
[...]
Você: /vc-security
Você: Agora verifica o isolamento
```

### 3. As skills são editáveis

Se quiser ajustar algum padrão, os arquivos estão em:
```
C:\Users\User\.claude\skills\vc-skills\[nome]\SKILL.md
```

Você pode editar livremente. São arquivos Markdown simples.

### 4. Skills funcionam em qualquer projeto

Embora baseadas no SGI-IDAL, as skills se adaptam. Se estiver num projeto
Express/Railway (como o ACAM), o Claude detecta o stack e aplica o padrão
correspondente.

### 5. Na dúvida sobre qual skill usar

Pergunte ao Claude:
```
Você: Preciso fazer [descrição da tarefa]. Qual skill devo usar?
```

---

## Referência rápida

| Preciso... | Usar |
|---|---|
| Começar um projeto novo | `/vc-plan` → `/vc-init` |
| Definir a identidade visual | `/vc-design` |
| Criar tabelas no banco | `/vc-data` |
| Codificar uma feature | `/vc-code` |
| Implementar cálculo legal | `/vc-norma` |
| Verificar segurança de dados | `/vc-security` |
| Subir para produção | `/vc-review` |
| Mexer em sistema com dados reais | `/vc-prod` |

---

## Reinstalação (se necessário)

Se precisar reinstalar as skills:

```
As skills estão em: C:\Users\User\.claude\skills\vc-skills\
Não precisam de instalação especial — são arquivos Markdown.
Se forem apagadas, pedir ao Claude Code para recriá-las.
```

---

## Retrospectiva: a regra mais importante

Ao final de cada projeto ou fase significativa, peça:

```
Você: O que aprendemos nesse projeto que deveria ir para as skills?
```

O Claude vai revisar o que foi feito, identificar o que foi difícil, o que quebrou,
o que funcionou melhor que o esperado, e propor atualizações nas skills.

**É assim que as skills evoluem.** Não por planejamento teórico, mas por experiência real.
Cada projeto ensina algo. A retrospectiva transforma esse aprendizado em prevenção
para o projeto seguinte.

As skills são a sua memória institucional de engenharia. Sem retrospectiva,
essa memória para de crescer.

---

## Reinstalação (se necessário)

Se precisar reinstalar as skills:

```
As skills estão em: C:\Users\User\.claude\skills\vc-skills\
Repositório: github.com/cvc339/vc-skills
Para reinstalar: git clone https://github.com/cvc339/vc-skills.git ~/.claude/skills/vc-skills
```

---

> *"Tudo o que é vibe coding é gerado por IA, mas nem tudo o que é gerado por IA é vibe coding."*
> — Deborah Folloni

Este manual existe para garantir que o código gerado com IA seja feito com
a devida diligência — não no modo random, mas com padrão, planejamento e disciplina.

*Construído a partir da experiência real com o SGI-IDAL (referência de qualidade)
e o ACAM (registro de problemas a evitar). Abril/2026.*
