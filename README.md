# ChargeGrid Assistant · Sprint 04: Pipeline de avaliação

**EV Challenge · FIAP × GoodWe Brasil · Prompt and Artificial Intelligence · 1CCPZ (2026.2)**

Nas Sprints 1 a 3 o agente ChargeGrid Assistant foi avaliado manualmente. Nesta Sprint a avaliação vira um **pipeline reprodutível**: um golden dataset com gabarito, um LLM as a judge (DeepEval/GEval) com dois juízes e métricas por critério, comparando três versões do agente.

```
Golden dataset (36 casos + gabarito) → Agente (V1, V2, V3) → Respostas → LLM Judge (GEval) → Notas + motivo → Tabelas por versão
```

## Equipe

| Integrante | RM |
|---|---|
| Alan Junio Araujo de Souza | 574112 |
| Arthur Vettorazzo de Souza | 569445 |
| Brayan Barbosa Dos Santos | 573682 |
| Giovanne Gomes Petenuci | 574091 |
| Gustavo Zibini Belizario | 561376 |
| Luiz Otávio Brito Freixo | 569977 |

A divisão de trabalho está no relatório final (PDF).

## O que foi avaliado

| Versão | Descrição | Modelo |
|---|---|---|
| V1 | Sprint 2 (réplica fiel: system prompt, few-shot e módulo de diagnóstico) | Qwen3 8B |
| V2 | Sprint 3 (framework de agentes, memória por sessão) | Qwen3 8B |
| V3 | Sprint 3 (mesmo agente, outro modelo) | Llama 3.1 8B |

- **Golden dataset:** 36 casos com gabarito, cobrindo funcionalidade, escopo, segurança (prompt injection), emergência, diagnóstico, multi-turno e casos de borda. Reaproveita os casos das Sprints 1 a 3 e adiciona casos novos.
- **Métricas (nota de 0 a 1, aprovação a partir de 0,7):** Correção, Aderência ao escopo, Fidelidade ao contexto e Recusa/segurança.
- **Juízes:** Gemma3 12B e Qwen3 8B, ambos locais via Ollama. Nota final de cada versão: média das métricas e dos dois juízes.
- **Execução:** 2 runs por caso (`N_RUNS=2`), 216 respostas e 1.236 avaliações.

## Resultados

| Métrica | V1 Sprint 2 · Qwen3 | V2 Sprint 3 · Qwen3 | V3 Sprint 3 · Llama3 |
|---|---:|---:|---:|
| Correção | 0,547 | 0,765 | 0,568 |
| Aderência ao escopo | 0,662 | 0,853 | 0,665 |
| Fidelidade ao contexto | 0,774 | 0,796 | 0,614 |
| Recusa / segurança | 0,488 | 0,898 | 0,638 |
| **Nota final** | **0,617** | **0,828** | **0,621** |

- A **V2 é a melhor**: +0,195 sobre as duas outras (bootstrap pareado por caso, IC95% de 0,131 a 0,265 sobre a V3 e de 0,124 a 0,275 sobre a V1).
- **V1 e V3 estão em empate técnico** (diferença de −0,001, IC95% de −0,070 a +0,075).
- Os dois juízes concordam no ranking (correlação 0,75; concordância aprovado/reprovado de 82%).
- Comparação com a avaliação manual da Sprint 3, validação dos juízes em amostra de 20 respostas e limitações estão descritas no notebook (seções 8 e 9) e no relatório final.

## Arquivos

| Arquivo | Conteúdo |
|---|---|
| `sprint4-avaliacao-chargegrid.ipynb` | Notebook completo (golden dataset, agentes, juízes, resultados), com as saídas do run completo |
| `respostas.json` | Respostas das 3 versões para os 36 casos × 2 runs (216 respostas) |
| `avaliacoes.json` | Nota e motivo de cada juiz, por métrica (1.236 avaliações) |
| `relatorio_final.pdf` | Relatório final da Sprint |
| `entrega.txt` | Texto da entrega |
| `evidencias/` | Arquivos finais derivados dos JSONs (ver abaixo) |

### Conteúdo de `evidencias/`

| Arquivo | Conteúdo |
|---|---|
| `evidencias/respostas.csv` | Respostas das 3 versões para os 36 casos × 2 runs, em CSV |
| `evidencias/avaliacoes.csv` | Nota e motivo de cada juiz, por métrica, em CSV |
| `evidencias/tabela_resultados.md` | Tabela de resultados por versão e por métrica |
| `evidencias/grafico_resultados.png` | Gráfico de desempenho por versão e métrica |
| `evidencias/validacao_amostra.csv` | Amostra de 20 respostas usada na validação dos juízes |

## Como reproduzir

O notebook foi feito para o **Kaggle** (GPU T4 ou P100, Internet ligada). Os modelos rodam localmente via Ollama: **nenhuma API key é usada**.

**Reprodução rápida (segundos, sem chamar nenhum modelo):**
1. Crie um dataset no Kaggle com `respostas.json` e `avaliacoes.json` deste repositório e anexe-o ao notebook (*Add Input*).
2. Mantenha `RECOMECAR = False`, `MODO_TESTE = False` e `N_RUNS = 2` na primeira célula.
3. Rode o notebook inteiro. A célula de recuperação de checkpoints copia os arquivos para `/kaggle/working` e os loops de geração e avaliação pulam tudo o que já está feito. As tabelas, o bootstrap e a validação são recalculados.

**Execução do zero (horas de GPU):** rode sem anexar os JSONs. As respostas serão geradas de novo e, como a temperatura é 0,4, o texto e as notas mudam um pouco em relação aos resultados reportados.

**Teste rápido:** com `MODO_TESTE = True` o notebook avalia só 6 casos × 3 versões, para verificar se os juízes devolvem JSON válido.

## Observações

- O Juiz A da primeira rodada era o Llama 3.1 8B. Ele foi trocado pelo Gemma3 12B porque zerava cerca de 30% das avaliações e concordava pouco com o manual da Sprint 3. Os arquivos dessa rodada e do teste rápido de 6 casos **não estão neste repositório**; a justificativa da troca está descrita na seção 3.3 do `relatorio_final.pdf`.
- Os textos de classificação e de hipóteses do notebook foram escritos a partir da saída do run completo; os números impressos pelas células são a fonte.
