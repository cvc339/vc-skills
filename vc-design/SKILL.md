# /vc-design — Definir Design System

Esta skill cria o design system do projeto ANTES de qualquer interface ser construída.
O resultado é um styleguide vivo dentro da aplicação — código que serve como referência
visual e técnica para todas as páginas que serão construídas depois.

## Filosofia

O styleguide não é documentação. É código que roda dentro do projeto. Quando alguém
(humano ou AI) precisa saber como usar um Badge de status, abre /styleguide/components/badge
e vê exemplos visuais com código copiável, contextualizados para o domínio do projeto.

Isso evita inconsistências desde o início. O SGI-IDAL foi construído assim e o resultado
é que todas as 13 páginas seguem o mesmo padrão visual sem nenhuma correção retroativa.

## O que esta skill produz

1. **Design tokens** (cores, tipografia, espaçamentos, border-radius)
2. **Página principal do styleguide** com paleta de cores e tipografia
3. **Páginas de showcase** para cada componente, com exemplos no contexto do projeto
4. **Navegação do styleguide** indexando todos os componentes
5. **Variáveis CSS** no globals.css (light + dark mode)

## Processo

### 1. Definir identidade visual com o fundador

Perguntar:
- Qual é o domínio do projeto? (ambiental, jurídico, financeiro, saúde...)
- Qual a sensação desejada? (institucional, técnico, acolhedor, autoritativo...)
- Há cor primária definida? (marca existente, preferência, ou definir juntos)
- Público-alvo? (profissionais técnicos, gestores, cidadãos, todos?)

### 2. Configurar design tokens

No `globals.css`, definir escalas completas:

```css
:root {
  /* Primary scale (10 tons) */
  --primary-50: ...;
  --primary-100 a --primary-900: ...;

  /* Grey scale (10 tons) */
  --grey-50 a --grey-900: ...;

  /* Cores semânticas (obrigatórias) */
  --success: ...;           /* Verde - status positivo */
  --warning: ...;           /* Amarelo - atenção */
  --info: ...;              /* Azul - informativo */
  --maintenance: ...;       /* Roxo - em andamento */
  --offline: ...;           /* Vermelho - negativo/destrutivo */

  /* Cores de gráficos (5 cores para charts) */
  --chart-1 a --chart-5: ...;

  /* Radius */
  --radius: 0.75rem;        /* 12px padrão */
}
```

### 3. Instalar componentes shadcn/ui

Componentes mínimos obrigatórios para um projeto profissional:

**Layout:** Card, Tabs, Dialog, Sheet, Separator
**Forms:** Input, Select, Checkbox, Switch, Label, Calendar, Textarea
**Data:** Table, Badge, Progress, Avatar
**Feedback:** Alert, AlertDialog, Toast (Sonner)
**Navigation:** Button, DropdownMenu

### 4. Criar página principal do styleguide

`src/app/styleguide/page.tsx`:
- Paleta Primary (10 tons visuais)
- Paleta Grey (10 tons visuais)
- Cores semânticas (5 blocos com nome e uso)
- Cores de gráficos
- Escala tipográfica (H1 a Small, com exemplos no contexto do projeto)
- Border radius (visual de sm a full)
- Preview de componentes base (Buttons, Badges, Cards, Alerts)
- Resumo do design (cor primária, fonte, estilo, sensação)

### 5. Criar páginas de showcase por componente

Para cada componente instalado, criar `src/app/styleguide/components/[nome]/page.tsx`:

Conteúdo obrigatório de cada showcase:
- `"use client"` no topo
- Título h1 e descrição do componente
- Seções em Cards com: variantes, tamanhos, estados
- **Exemplos com cores semânticas** (success, warning, info, maintenance, offline)
- **Exemplos contextualizados** para o domínio do projeto (não genéricos)
- Bloco `<pre>` com código de uso copiável
- Ícones Lucide

### 6. Criar layout e navegação do styleguide

`src/app/styleguide/layout.tsx`:
- Sidebar com navegação entre componentes
- Sem autenticação (acesso livre, mesmo sem login)

`src/app/styleguide/navigation.ts`:
- Array tipado com seções (Fundação, Componentes)
- Atualizar sempre que adicionar novo componente

### 7. Componentes customizados do domínio

Se o projeto precisa de componentes que não existem no shadcn/ui:
1. Criar o componente em `src/components/ui/`
2. Criar a página de showcase no styleguide
3. Documentar variantes e uso

Exemplo do SGI-IDAL: componente `Gauge` para visualizar o indicador IDAL.

## Padrões visuais obrigatórios

### Cores semânticas para status
Nunca usar cores arbitrárias para status. Sempre usar o sistema:
- `bg-success text-success-foreground` → positivo
- `bg-warning text-warning-foreground` → atenção
- `bg-info text-info-foreground` → neutro/informativo
- `bg-maintenance text-maintenance-foreground` → em andamento
- `bg-offline text-offline-foreground` → negativo

### Tipografia
- H1: text-2xl font-bold (títulos de página)
- H2: text-xl font-semibold (seções)
- Body: text-base (parágrafo)
- Descrições: text-muted-foreground
- Dados numéricos em destaque: text-2xl ou text-3xl font-bold

### Layout padrão de página
```
div.p-6.space-y-6
  ├── div (header: h1 + descrição + botão ação)
  ├── div.grid (cards de resumo/KPIs)
  └── Card (conteúdo principal: tabela, formulário, etc.)
```

### Responsividade
- Mobile first sempre
- Grid: grid-cols-1 md:grid-cols-2 lg:grid-cols-3
- Sidebar: hidden md:block
- Inputs e botões: min-height 44px para touch

## Depois do styleguide criado: como usar

### Adicionar novo componente (durante o projeto)

Quando uma página precisa de um componente que não existe no styleguide:

1. **Instalar:** `npx shadcn@latest add [nome-componente]`
2. **Criar showcase:** `src/app/styleguide/components/[nome]/page.tsx`
   - "use client" no topo
   - Título h1 e descrição do componente
   - Seções em Cards com variantes, tamanhos e estados
   - Exemplos com cores semânticas (success, warning, info, maintenance, offline)
   - Exemplos contextualizados para o domínio do projeto
   - Bloco `<pre>` com código de uso
   - Ícones Lucide
3. **Atualizar navegação:** Em `src/app/styleguide/navigation.ts`, adicionar o item

**A regra é: nunca usar um componente em uma página se ele não tem showcase no styleguide.**

### Construir páginas usando o styleguide

Ao criar ou refatorar uma página funcional:

1. **Analisar o layout necessário:** colunas, sidebar, seções, hierarquia de informação
2. **Mapear para componentes existentes no styleguide** — priorizar o que já está documentado
3. **Aplicar padrões de domínio:**
   - Status → Badge com cores semânticas
   - Métricas/KPIs → Card com Progress ou valor em destaque
   - Listas → Card com hover:bg-muted/50
   - Formulários → Label + Input/Select
   - Tabelas → Table com Badge para status
4. **Seguir a estrutura padrão de página** (header → cards resumo → conteúdo principal)
5. **Se faltar componente** → adicionar ao styleguide primeiro (passo anterior), depois usar

### Regra crítica para refatoração

Ao refatorar páginas existentes:
- **NÃO ALTERAR:** lógica de negócio, funções de cálculo, hooks, chamadas de API, validações
- **APENAS ALTERAR:** componentes visuais, classes CSS, estrutura de layout, ícones

Isso garante que melhorias visuais nunca quebram funcionalidade.

## Regras

- Criar o styleguide ANTES de qualquer página funcional
- Todo componente novo precisa de showcase no styleguide ANTES de ser usado em páginas
- Nunca usar cores hardcoded em páginas — sempre via tokens/classes semânticas
- O styleguide é parte do projeto, não documentação externa
- Se o fundador aprovar o styleguide, ele é o contrato visual — toda página segue
- Exemplos no showcase devem usar termos do domínio do projeto, não lorem ipsum
- Refatoração visual nunca altera lógica de negócio
