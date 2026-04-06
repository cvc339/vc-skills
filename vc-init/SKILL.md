# /vc-init — Iniciar Projeto Profissional

Quando um novo projeto começa, esta skill define toda a fundação antes da primeira feature.
O objetivo: qualquer desenvolvedor (humano ou AI) que abrir o projeto entende a estrutura,
o padrão e sabe como contribuir em 15 minutos.

## O que esta skill produz

Ao final do init, o projeto terá:

1. **Estrutura de pastas** que torna a arquitetura visível
2. **Design system** com tokens, cores e componentes definidos (via /vc-design)
3. **Schema do banco** com padrão multi-tenant e RLS (via /vc-data)
4. **Documentação viva** que se mantém atualizada entre sessões
5. **Build funcional** — o projeto roda com um comando

## Stack padrão

| Camada | Tecnologia | Motivo |
|--------|-----------|--------|
| **Framework** | Next.js 14+ (App Router) | Full-stack integrado, SSR, API routes |
| **Linguagem** | TypeScript strict | Tipagem previne bugs em produção |
| **Banco** | PostgreSQL via Supabase | RLS nativo, auth integrado, edge functions |
| **UI** | shadcn/ui + Tailwind CSS | Componentes acessíveis, customizáveis, sem vendor lock-in |
| **Ícones** | Lucide React | Consistente, tree-shakeable |
| **Testes** | Vitest | Rápido, compatível com TypeScript |
| **Deploy** | Vercel + Supabase | Zero-config, scaling automático |

Se o projeto tem necessidade específica que justifique outra escolha (ex: geoespacial pesado
que precisa de Express), justifique e documente. Mas o padrão é este.

## Passo a passo

### 1. Criar o projeto

```bash
npx create-next-app@latest nome-projeto --typescript --tailwind --eslint --app --src-dir
cd nome-projeto
```

### 2. Configurar Supabase

```bash
npx supabase init
```

Criar `.env.local`:
```
NEXT_PUBLIC_SUPABASE_URL=
NEXT_PUBLIC_SUPABASE_ANON_KEY=
SUPABASE_SERVICE_ROLE_KEY=
```

Criar `.env.local.example` com as mesmas variáveis sem valores.

### 3. Instalar dependências base

```bash
# UI
npx shadcn@latest init  # style: new-york, base: neutral, css variables: yes
npx shadcn@latest add button card badge alert input select checkbox label

# Utilitários
npm install date-fns zod lucide-react sonner recharts

# Testes
npm install -D vitest @testing-library/react jsdom
```

### 4. Criar estrutura de pastas

```
src/
  app/
    (auth)/              # Login, registro, recuperação de senha
      login/page.tsx
      registro/page.tsx
    (dashboard)/         # Rotas protegidas (layout com sidebar)
      layout.tsx         # Sidebar + header
      page.tsx           # Dashboard principal
    styleguide/          # Design system vivo (via /vc-design)
      page.tsx
      components/
      navigation.ts
    api/                 # API routes (se necessário)
  components/
    ui/                  # shadcn/ui (gerado automaticamente)
    layout/              # Header, Sidebar, Footer
    forms/               # Componentes de formulário reutilizáveis
  lib/
    supabase/
      client.ts          # createClient() para client-side
      server.ts          # createClient() para server-side
    calculo/             # Lógica de negócio isolada (com testes)
  contexts/              # React contexts (auth, theme, etc.)
  types/                 # Interfaces TypeScript
  middleware.ts          # Proteção de rotas
docs/
  claude-context/        # Documentação para manter contexto entre sessões
    00_VISAO_GERAL.md
    01_ARQUITETURA.md
    02_REGRAS_NEGOCIO.md
    03_BANCO_DADOS.md
    04_PADROES_CODIGO.md
    05_DECISOES.md
supabase/
  migrations/            # SQL versionado
  functions/             # Edge Functions
```

### 5. Criar documentação fundacional

**ARCHITECTURE.md** na raiz:
- Modelo de dados (N1/N2/N3 se multi-tenant, ou estrutura simplificada)
- Templates SQL para criação de tabelas
- Regras obrigatórias do projeto
- Anti-patterns proibidos

**CLAUDE.md** na raiz:
- Instruções para desenvolvimento com AI
- Regra de atualização de contexto após mudanças
- Design system: onde encontrar, como usar
- Padrões de código (imports, tipos, componentes)

**docs/claude-context/00_VISAO_GERAL.md**:
- O que é o projeto, para quem, qual problema resolve
- Módulos planejados
- Stack e justificativa

### 6. Configurar middleware de autenticação

```typescript
// src/middleware.ts
import { createServerClient } from '@supabase/ssr'
import { NextResponse } from 'next/server'
import type { NextRequest } from 'next/server'

export async function middleware(request: NextRequest) {
  // Proteger rotas do dashboard
  // Redirecionar para /login se não autenticado
  // Permitir acesso a /login, /registro, /styleguide
}

export const config = {
  matcher: ['/((?!_next/static|_next/image|favicon.ico|styleguide).*)']
}
```

### 7. Executar /vc-design

Criar o design system antes de qualquer página. Ver skill /vc-design.

### 8. Executar /vc-data

Modelar o banco de dados antes de qualquer feature. Ver skill /vc-data.

### 9. Verificar

Ao final do init:
- [ ] `npm run build` passa sem erros
- [ ] `npm run dev` abre o projeto
- [ ] /styleguide mostra o design system
- [ ] ARCHITECTURE.md existe e está completo
- [ ] CLAUDE.md existe com instruções claras
- [ ] docs/claude-context/ tem pelo menos 00 a 05
- [ ] .env.local.example existe com todas as variáveis
- [ ] Supabase conecta e auth funciona

## Regras

- Não criar arquivos vazios ou com TODO. Tudo que existe funciona.
- Não instalar dependências "para o futuro". Instalar quando usar.
- Documentar toda decisão técnica em 05_DECISOES.md.
- O projeto deve rodar com: `git clone` → `cp .env.local.example .env.local` → preencher → `npm install` → `npm run dev`.
- Nenhum secret commitado. Nunca.
