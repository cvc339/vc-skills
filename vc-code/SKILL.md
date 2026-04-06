# /vc-code — Padrões de Codificação Profissional

Como um dev sênior escreve código. Não é sobre funcionar — é sobre funcionar, ser legível,
ser modificável, e não quebrar quando for necessário mexer em produção.

## Filosofia

A diferença entre código amador e profissional não aparece no dia 1.
Aparece no dia 90, quando precisa:
- Corrigir um bug sem quebrar outra coisa
- Adicionar uma feature sem reescrever tudo
- Outro desenvolvedor (humano ou AI) entender o que foi feito

O código que "resolve o problema imediato" mas cria problemas futuros
não resolve o problema — apenas adia.

## Lições do mundo real

### O que aconteceu no ACAM (o que NÃO fazer):

| Prática | Consequência real |
|---|---|
| Tudo num único arquivo HTML (1.230 linhas) | Cada ajuste era arriscado, impossível testar isoladamente |
| React + Babel transpilado no browser | Complexidade desnecessária, debug difícil |
| Logo em base64 inline (centenas de KB) | Arquivo enorme, impossível de ler |
| Arquivo duplicado (standalone) | Correção feita numa, esquecida na outra |
| Schema sem versionamento | Tabela inexistente descoberta quando usuário tentou usar → 502 |
| `const` dentro do try, usado no catch | ReferenceError que crashava o servidor |
| `res.status()` dentro de função sem acesso ao `res` | Servidor travava sem responder → 502 |
| Nomes misturados (created_at vs criado_em) | Queries falhando silenciosamente |
| Funções SQLite num banco PostgreSQL | Incompatibilidade não detectada |

### O que funcionou no SGI-IDAL (o que FAZER):

| Prática | Benefício real |
|---|---|
| TypeScript strict | Erros de tipo detectados antes de rodar |
| Cálculos isolados em lib/calculo/ | 87 testes, fórmulas seguras de refatorar |
| Design system com showcase | Todas as 13 páginas consistentes sem retrabalho |
| Migrations versionadas | Schema sempre sincronizado com o código |
| Decisões documentadas (05_DECISOES.md) | Contexto preservado entre sessões |
| Componentes reutilizáveis | Mudança visual propagada automaticamente |

## Antes de codificar qualquer página ou componente

**Ler o design system do projeto.** Se o projeto tem `/styleguide` (ou `styles.css`
com variáveis), toda página DEVE usar os componentes e tokens existentes.

Checklist automático ao criar/modificar interface:
1. Consultar componentes disponíveis no projeto (`/styleguide` ou `src/components/ui/`)
2. Usar cores semânticas para status (nunca cores hardcoded)
3. Seguir a estrutura padrão de página definida no design system
4. Se precisar de componente que não existe: criar no design system primeiro, usar depois
5. Consultar `CLAUDE.md` na raiz do projeto para regras específicas

**Esta regra se aplica sempre que `/vc-code` for chamada, sem necessidade de
chamar `/vc-design` separadamente.**

## Princípios (não negociáveis)

### 1. Um arquivo, uma responsabilidade

Nunca misturar na mesma unidade:
- Dados/configuração com lógica
- Lógica de negócio com apresentação
- Múltiplas features não relacionadas

**No Next.js/TypeScript:**
```
src/lib/calculo/cg.ts          ← fórmula pura (testável)
src/app/(dashboard)/idal/page.tsx  ← apresentação (usa a fórmula)
src/components/ui/gauge.tsx     ← componente visual (reutilizável)
```

**Mesmo em projetos simples (Express/HTML):**
```
routes/consultas.js             ← rota (recebe request, responde)
services/analise.js             ← lógica (processamento)
utils/database.js               ← acesso a dados
public/ferramenta.html          ← interface
public/js/analytics.js          ← utilitário reutilizável
```

### 2. Funções puras para lógica de negócio

Lógica de negócio = funções que recebem dados e retornam resultado. Sem side effects.
Sem acessar banco, sem chamar API, sem mexer no DOM.

```typescript
// BOM: função pura, testável
function calcularCG(entregas: Entrega[]): number {
  const merito = entregas.filter(e => e.atendido).length / entregas.length
  const modo = entregas.filter(e => e.modo_correto).length / entregas.length
  const tempo = entregas.filter(e => e.tempestivo).length / entregas.length
  return merito * 0.5 + modo * 0.2 + tempo * 0.3
}

// RUIM: mistura lógica com acesso a dados
async function calcularCG(licencaId: string) {
  const { data } = await supabase.from('entregas').select('*').eq('license_id', licencaId)
  // ... cálculo misturado com query
}
```

A função pura pode ter 87 testes. A impura precisa de banco rodando para testar.

### 3. Error handling que preserva contexto

```typescript
// BOM: variável no escopo externo, catch com contexto
let consultaId: number | null = null
try {
  const result = await criarConsulta(dados)
  consultaId = result.id
  await processarAnalise(consultaId)
} catch (error) {
  if (consultaId) {
    await reembolsarCreditos(usuarioId, consultaId)
  }
  res.status(500).json({
    erro: error.message,
    consulta_id: consultaId,
    creditos_devolvidos: consultaId ? true : false
  })
}

// RUIM: const no escopo do try, catch sem acesso, servidor trava
try {
  const consultaId = await criarConsulta(dados) // const dentro do try
  await processarAnalise(consultaId)
} catch (error) {
  if (consultaId) { // ReferenceError: consultaId is not defined → CRASH
    await reembolsarCreditos(usuarioId, consultaId)
  }
}
```

### 4. Nunca confiar em dados externos

Toda boundary (API, formulário, arquivo, banco) valida:

```typescript
// Dados do formulário
const area = parseFloat(req.body.area)
if (isNaN(area) || area <= 0 || area > 100000) {
  return res.status(400).json({ erro: 'Área inválida' })
}

// Resposta de API externa
const response = await fetch(url, { signal: AbortSignal.timeout(10000) })
if (!response.ok) {
  throw new Error(`API retornou ${response.status}`)
}
const data = await response.json()
if (!data || !data.resultado) {
  throw new Error('Resposta da API inválida')
}
```

### 5. Nomes que explicam

```typescript
// BOM
const diasAteVencimento = differenceInDays(dataVencimento, new Date())
const estaVencida = diasAteVencimento < 0
const precisaAlerta = diasAteVencimento <= 30 && diasAteVencimento >= 0

// RUIM
const d = diff(dv, new Date())
const v = d < 0
const a = d <= 30 && d >= 0
```

Nomes de funções descrevem o que fazem:
- `calcularCG()` não `calc()`
- `verificarMataAtlantica()` não `check()`
- `reembolsarCreditos()` não `rollback()`

### 6. Configuração separada de código

Valores que mudam entre ambientes ou ao longo do tempo não ficam no código:

```typescript
// BOM: configuração explícita
const CREDITOS_ANALISE = 5        // constante nomeada
const TIMEOUT_API = 10000         // constante nomeada
const UFEMG = await fetchUfemg()  // valor dinâmico da API

// RUIM: magic numbers
if (usuario.creditos < 5) { ... }              // 5 o que?
const r = await fetch(url, { timeout: 10000 }) // por que 10000?
const v = area * 5.5310                        // valor hardcoded que muda todo ano
```

### 7. Código que se deleta fácil

Quando uma feature é removida ou substituída, deve ser possível apagar o código
sem quebrar outras coisas. Isso exige:

- Baixo acoplamento entre módulos
- Sem dependências circulares
- Interfaces claras entre componentes

A calculadora v1 do ACAM não podia ser deletada porque era referenciada por 3 páginas
e tinha um arquivo duplicado. A v2 é um único arquivo, referenciado num único ponto do dashboard.

### 8. Consistência absoluta

Dentro de um projeto, tudo segue o mesmo padrão:
- Nomes de colunas: sempre `criado_em` ou sempre `created_at` — nunca os dois
- Formato de data: sempre `date-fns` com `ptBR` — nunca misturar com `toLocaleDateString`
- Error handling: sempre o mesmo pattern (let externo + try/catch)
- Respostas de API: sempre `{ sucesso: true, dados }` ou `{ sucesso: false, erro }`

### 9. Sem duplicação

Se o mesmo código aparece em dois lugares, ele deveria estar em um lugar e ser importado.

```typescript
// BOM: utilitário reutilizado
// utils/formatacao.ts
export function formatarMoeda(valor: number): string {
  return new Intl.NumberFormat('pt-BR', { style: 'currency', currency: 'BRL' }).format(valor)
}

export function pluralizar(unidade: string, quantidade: number): string {
  if (unidade === 'hectare' && quantidade > 1) return 'hectares'
  return unidade
}

// RUIM: mesma lógica repetida em 5 arquivos diferentes
```

### 10. Testes onde importa

Não testar tudo — testar o que quebra. Prioridade:

1. **Fórmulas de cálculo** (é o core do negócio)
2. **Fluxos financeiros** (créditos, pagamentos)
3. **Transformação de dados** (parsing, conversões)
4. **Edge cases** (zero, negativo, null, array vazio)

```typescript
// calculo/cg.test.ts
describe('calcularCG', () => {
  it('retorna 1.0 quando todas as entregas atendem tudo', () => {
    expect(calcularCG(entregasPerfeitas)).toBe(1.0)
  })

  it('retorna 0.0 quando nenhuma entrega atende', () => {
    expect(calcularCG(entregasZeradas)).toBe(0.0)
  })

  it('retorna 0.5 quando apenas mérito atende', () => {
    expect(calcularCG(entregasApenasMerito)).toBe(0.5)
  })

  it('lida com array vazio sem crash', () => {
    expect(calcularCG([])).toBe(0)
  })
})
```

## Anti-patterns que parecem resolver mas criam problemas

| Parece bom | Por que é ruim |
|---|---|
| Tudo num arquivo só ("mais simples") | Impossível testar, impossível manter, impossível reutilizar |
| Copiar e colar ("mais rápido") | Bug corrigido num lugar, esquecido no outro |
| Catch vazio `catch(e) {}` | Erro silencioso → bug invisível → 502 inexplicável |
| `any` no TypeScript ("compila") | Perde toda a proteção de tipo — bugs em runtime |
| console.log para debug ("funciona") | Logs inúteis em produção, informação sensível exposta |
| Hardcoded "por enquanto" | "Por enquanto" dura meses — vide UFEMG no ACAM |
| Comentário `// TODO: melhorar` | Nunca será feito — ou faz agora ou aceita como está |

## Quando usar esta skill

- Antes de começar a codificar uma feature nova
- Quando revisar código de outro (humano ou AI)
- Quando sentir que está fazendo "puxadinho" para resolver
- Na dúvida entre "funciona" e "funciona direito"

## Regras

- Se precisa de comentário para explicar, o código não está claro o suficiente
- Se precisa de duplicação, falta uma abstração
- Se precisa de hack, falta entendimento do problema
- Se funciona mas não sabe por quê, não está pronto
- O código que você escreve será mantido por alguém (incluindo você) daqui a 6 meses. Escreva para essa pessoa.
