# /vc-norma — Integrar Legislação e Cálculos Regulatórios

Esta skill orienta como incorporar legislação ambiental brasileira nas ferramentas do projeto.
Precisão normativa é inegociável — o público são advogados e engenheiros ambientais
que usam os resultados em processos administrativos reais.

## Filosofia

Uma ferramenta jurídico-ambiental que erra a referência legal perde toda a credibilidade.
Melhor não ter a informação do que ter a informação errada. Quando há dúvida sobre uma norma,
declarar a limitação é mais profissional do que chutar.

## Princípios para implementar legislação

### 1. Isolar a lógica de cálculo
Fórmulas regulatórias DEVEM estar em `src/lib/calculo/` (ou equivalente), separadas da UI.
Cada fórmula é uma função pura: recebe dados, retorna resultado. Sem side effects.

Isso permite:
- Testes unitários para cada fórmula
- Atualização da norma sem mexer na interface
- Rastreabilidade: cada função documenta a base legal

### 2. Documentar a base legal no código
Toda função de cálculo regulatório deve ter um comentário com:
- Nome da norma (ex: Decreto 47.749/2019)
- Artigo/parágrafo específico
- Data de consulta/verificação

```typescript
/**
 * Calcula compensação 2:1 em vegetação nativa
 * Base legal: Decreto 47.749/2019, Art. 45-50
 * Última verificação: 2026-04-06
 */
function calcularCompensacao(areaSuprimida: number): number {
  return areaSuprimida * 2
}
```

### 3. Distinguir critérios obrigatórios de preferenciais
A interface DEVE deixar claro ao usuário o que é obrigatório e o que é preferencial.
Nunca misturar os dois na mesma lista sem distinção visual.

### 4. Incluir disclaimers onde necessário
Ferramentas que usam sensoriamento remoto, estimativas ou dados de terceiros:
- "Os resultados NÃO substituem vistoria de campo, laudos técnicos especializados ou parecer do órgão ambiental"
- "Servem como subsídio preliminar para tomada de decisão"
- Informar a fonte e data dos dados (ex: "MapBiomas Coleção 9 (2023)")

Ferramentas que calculam valores monetários:
- Informar o ano e valor de referência (UFEMG, TJLP, etc.)
- Indicar que são estimativas sujeitas a atualização

### 5. Tratar valores atualizáveis
Valores que mudam periodicamente (UFEMG, UPF, salário mínimo, tabelas de referência):
- Armazenar no banco de dados, gerenciável pelo admin
- Frontend consulta a API para obter o valor vigente
- Manter fallback local com aviso ao usuário se usar valor desatualizado
- Nunca hardcoded em múltiplos arquivos

### 6. Testar as fórmulas
Toda fórmula regulatória deve ter testes unitários:
- Valores conhecidos (casos reais, se possível)
- Edge cases (zero, negativos, valores muito altos)
- Casos-limite das faixas/classificações

### 7. Formatos brasileiros
- Valores monetários: R$ 1.234,56
- Datas: dd/MM/yyyy (date-fns com locale ptBR)
- Hectares: plural quando > 1 (1 hectare, 2 hectares)
- m³ e kg: não mudam no plural
- CPF/CNPJ: formatados com pontos/barras

## Legislação de referência (domínio ambiental MG)

### Federal
- **Lei 9.985/2000** — SNUC (Sistema Nacional de Unidades de Conservação)
- **Lei 11.428/2006** — Lei da Mata Atlântica
- **Lei 12.651/2012** — Código Florestal

### Estadual (Minas Gerais)
- **Decreto 47.749/2019** — Compensação em Mata Atlântica
- **Lei 20.922/2013** — Políticas florestal e de biodiversidade em MG
- **Resolução SEMAD 3.263/2023** — IDAL (Índice de Desempenho Ambiental de Licenciados)
- **Resolução Conjunta SEMAD/IEF 3.102/2021** — Aproveitamento de material lenhoso
- **Portaria IEF 27/2017** — Compensação ambiental de mineração

### Unidades fiscais
- **UFEMG** — Unidade Fiscal do Estado de Minas Gerais (atualizada anualmente pela Fazenda de MG)

## Regras

- Referência legal errada é mais grave que bug de interface — trate com prioridade máxima
- Se não tem certeza da norma: diga "verificar" em vez de inventar
- Toda atualização normativa = atualizar código + testes + documentação
- O público confia nos números. Eles vão para processos administrativos reais.
