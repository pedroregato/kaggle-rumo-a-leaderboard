# EDU-MDC-001: plano do Evidence Lab Make Data Count

- **Status:** rascunho para revisão
- **Data:** 2026-09-20
- **Camada do curso:** II, Model Engineering, com saída para a Aula 13C
- **Formato de referência:** `TIL EDU ORCH 001 Baseline vs Transformer`
- **Nome sugerido no Kaggle:** `TIL EDU MDC 001 Rules vs Metadata Classifier`

## 1. Propósito

Produzir evidência comparável para a Aula 13C (Model Routing, Orchestration and Utility) usando uma tarefa real de extração e classificação em textos científicos: dado o texto de um artigo, achar as referências a dados de pesquisa e classificar cada uma como `Primary` (dado gerado no próprio estudo) ou `Secondary` (dado reutilizado).

A pergunta experimental é: **quanto de desempenho cada nível de complexidade compra, e a que custo?** Os níveis são regras simples, um classificador clássico com metadados e, opcionalmente, um LLM zero-shot. O experimento não procura provar que o sistema mais sofisticado vence. Ele mede qualidade, latência e custo computacional sob o mesmo conjunto de avaliação, como faz o `EDU-ORCH-001`.

O caso ilustra o princípio central do curso: complexidade arquitetural precisa ser conquistada por evidência. Nos writeups da competição, o 2º lugar obteve 0,739 no placar privado só com regras (pipeline completo, não comparável diretamente com este laboratório), e o 1º lugar usou modelos diferentes por família de identificador, uma forma de roteamento.

## 2. O que já está definido

| Tema | Decisão |
| --- | --- |
| Formato | Evidence Lab com notebook próprio e CSV de evidência compatível com a 13C |
| Sistemas comparados | A: baseline por regras. B: CatBoost com metadados. C: LLM zero-shot (opcional). D: composição roteada por família (linha derivada) |
| Sistemas fora do escopo do núcleo | DeBERTa-v3 Large (o fine-tuning do DistilBERT no `EDU-ORCH-001` levou 6h45 em CPU) |
| Internet | Desligada por padrão, como nos notebooks oficiais |
| Aulas de apoio | Cinco notebooks curtos no padrão Objectives até Summary, com `qN.hint()` e `qN.solution()` |
| Repositório | GitHub como fonte de verdade; Kaggle como ambiente de execução |

## 3. Fontes e o que foi verificado

Este plano se baseia em: a página da competição Make Data Count - Finding Data References, os writeups do 1º, 2º e 3º lugares, o notebook público do 1º lugar, o notebook do curso e o `EDU-ORCH-001`.

Não foram verificados: o `EDU-ORCH-002`, o conteúdo do repositório `text-intelligence-lab`, os seus datasets publicados no Kaggle e as licenças dos datasets externos usados pelo 1º lugar. Os itens abaixo que dependem disso estão marcados como decisões em aberto.

Os números do treino (719 rótulos válidos, 214 artigos, distribuição por tipo) foram lidos no notebook do 1º lugar, calculados sobre os arquivos de treino da competição.

## 4. Hipóteses a testar

Estas são hipóteses, não resultados esperados.

1. As regras sozinhas obtêm F1 de tipo bem acima de um classificador que sempre prevê a classe majoritária.
2. Os metadados (similaridade de título, autores e ano entre artigo e dataset) melhoram o F1 de tipo para DOIs, mas pouco para accession IDs, cujos metadados de contexto são mais fracos.
3. Uma composição que roteia por família (DOI para o sistema B, accession ID para o sistema A ou C) supera qualquer sistema usado sozinho em pelo menos uma das duas dimensões, qualidade ou custo.

## 5. Contrato experimental (proposta)

Fixado antes de observar resultados, como no `EDU-ORCH-001`.

| Item | Proposta |
| --- | --- |
| Unidade de observação | Uma menção: tupla (`article_id`, `dataset_id`) com o tipo verdadeiro |
| Tarefa avaliada | Classificar o tipo (`Primary` ou `Secondary`) de cada menção **dada** (menções verdadeiras como entrada) |
| Métrica principal | F1 macro do tipo, por família e no total |
| Métricas complementares | Acurácia; F1 por tupla (`article_id`, `dataset_id`, `type`) nas menções detectadas; F1 de detecção reportado à parte |
| Split | Fixo e agrupado por `article_id`, com semente 42; nenhum artigo em treino e validação ao mesmo tempo |
| Entrada comum | Metadados e texto ao redor da menção, conforme cada sistema |
| Latência | Por menção, `batch_size=1`, com média, p50 e p95, medindo só a classificação |
| Custo | Proxy computacional: segundos de execução por 1.000 menções classificadas; sem custo monetário |
| Evidência | Só exportada quando dados reais forem carregados e todos os sistemas do núcleo forem medidos |

A escolha de avaliar o tipo sobre menções dadas isola a pergunta que interessa à 13C (qual sistema classifica melhor e a que custo) e separa o problema de detecção, que depende de fontes externas. A alternativa de avaliar o pipeline completo está na seção de decisões em aberto.

Métrica principal em F1 macro porque as duas famílias são desbalanceadas em sentidos opostos: nos DOIs há 215 `Primary` e 110 `Secondary`, e nos accession IDs há 55 `Primary` e 339 `Secondary`.

## 6. Dataset Card (a publicar no notebook)

| Campo | Conteúdo |
| --- | --- |
| Origem | Competição Make Data Count - Finding Data References (Kaggle); artigos do EuropePMC |
| Como os rótulos foram gerados | A partir de menções extraídas por NER do EuropePMC e de mapeamentos artigo-dataset do DataCite, segundo o writeup do 1º lugar, que cita confirmação dos organizadores |
| Unidade | Uma menção de dataset em um artigo |
| Tamanho usado | 719 rótulos válidos em 214 artigos, depois de remover 347 linhas `Missing` |
| Distribuição | DOIs: 325 (215 `Primary`, 110 `Secondary`). Accession IDs: 394 (55 `Primary`, 339 `Secondary`) |
| Limitações | Rótulos gerados por processo automático, com o viés desse processo; conjunto pequeno; artigos `Missing` não contam na métrica da competição; domínio de ciências da vida |
| Licença e governança | A verificar: regras da competição e licença de cada dataset externo. O TIL registra as informações e não presume que o artefato derivado herda a licença de origem |

## 7. Recursos e política de internet

Internet desligada. Os recursos entram como inputs do notebook e são descobertos de forma determinística, com falha controlada se ausentes (`RESOURCES_READY`).

| Recurso | Uso | Situação |
| --- | --- | --- |
| Dados da competição (PDF, XML, `train_labels.csv`) | Texto e rótulos | Exige entrar na competição e aceitar as regras |
| Metadados do artigo (Crossref) | Features do sistema B | A confirmar se há dataset público reutilizável |
| Metadados do dataset (DataCite: autores e ano) | Features do sistema B | A confirmar |
| PyMuPDF | Converter PDF em texto | Instalação offline a resolver |
| Modelo de linguagem para o sistema C | LLM zero-shot | Opcional; a definir modelo e cota de GPU |

O notebook do 1º lugar usa datasets próprios da equipe com corpus e metadados. Antes de depender deles, é preciso confirmar que são públicos, qual a licença e se permanecem disponíveis. Alternativa: gerar as tabelas de metadados a partir dos arquivos da competição.

## 8. Sistemas

**A. Baseline por regras.** Regras de tipo derivadas dos writeups, sem modelo: por exemplo, `SAMN` e `EMDB` tendem a `Primary`; DOI de repositório muito citado tende a `Secondary`. O laboratório mede acerto de cada regra separadamente. Referências: 2º lugar (regras) e 3º lugar (Dryad e `SAMN` como `Primary`; muitas citações como `Secondary`).

**B. CatBoost com metadados.** Features de coincidência e diferença de ano, coincidência de editora e periódico, contagem de citações e similaridade de título e autores. Validação agrupada por artigo, com similaridade de título calculada dentro de cada fold para evitar vazamento. Ponto de partida de hiperparâmetros: os do notebook do 1º lugar (MultiClass, 1000 iterações, taxa 0,03, profundidade 6, parada antecipada em 20, pesos de classe balanceados).

**C. LLM zero-shot (opcional, GPU).** Prompt simples com o trecho ao redor da menção e saída restrita a duas opções, como no 1º lugar. Só entra na evidência se houver GPU e o gate de tempo for atendido.

**D. Composição roteada (linha derivada).** Combina os melhores sistemas por família, com base nas medições de A, B e C. Não é um modelo novo: é a linha que a 13C usa para discutir roteamento.

## 9. Estrutura do notebook

Espelha o `EDU-ORCH-001`, para o aluno reconhecer o padrão.

| Seção | Conteúdo |
| --- | --- |
| 1 | Dataset Card |
| 2 | Contrato experimental |
| 3 | Imports, semente e `RUN_MODE` (`EVIDENCE` no Kaggle, `SMOKE` fora) |
| 4 | Descoberta determinística de recursos |
| 5 | Carga de dados e construção do split agrupado |
| 6 | Gates de integridade |
| 7 | Sistema A: regras |
| 8 | Sistema B: CatBoost |
| 9 | Sistema C: LLM zero-shot (opcional) |
| 10 | Benchmark de latência e proxy de custo |
| 11 | Evidência consolidada e linha D |
| 12 | Export do CSV |
| 13 | Interpretação: de Model Selection a Compound AI Systems |
| 14 | Manifest da execução |

### Gates de integridade

O notebook interrompe se algum falhar.

1. Nenhum `article_id` aparece em treino e validação ao mesmo tempo.
2. Depois de remover `Missing`, só existem os tipos `Primary` e `Secondary`.
3. As contagens do treino completo batem com o esperado (719 rótulos, 214 artigos).
4. DOIs em minúsculas e accession IDs com a caixa original do artigo, pois a métrica da competição diferencia maiúsculas.
5. Features de similaridade calculadas apenas com dados de treino do fold.
6. Nenhuma regra do sistema A usa rótulos do conjunto de teste.

### Modos de execução

`EVIDENCE` grava evidência real. `SMOKE` roda em ambiente sem os recursos e não fabrica linhas, como no `EDU-ORCH-001`.

## 10. Contrato do CSV de evidência

Arquivo `til-model-evidence.csv`, no mesmo caminho do `EDU-ORCH-001` (`/kaggle/working/model-evidence` no Kaggle e `data/model-evidence` localmente).

Campos mínimos, exigidos pela 13C: `system`, `quality`, `cost_per_1000`, `latency_ms`.

Campos adicionais: `quality_metric`, `accuracy`, `family` (`doi`, `accession` ou `all`), `latency_p50_ms`, `latency_p95_ms`, `cost_unit`, `cost_method`, `source` (`EDU-MDC-001`), `measured_at`, `evidence_status`, `dataset`, `hardware`, `sample_size`, `model_version`, `notes`.

Uma linha por sistema e por família. Linhas derivadas (sistema D) trazem `evidence_status` distinto de `measured`, para não se confundirem com medições diretas. A convenção exata para linhas derivadas deve ser alinhada com a 13C.

## 11. Aulas de apoio

Notebooks curtos, cada um com o padrão Objectives, Context, Environment, Experiment, Observation, Interpretation, Exercise, Reproducibility e Summary.

| Apoio | Tema | Conecta com o Evidence Lab |
| --- | --- | --- |
| 1 | Origem dos rótulos e artigos `Missing` | Seções 1 e 6 |
| 2 | Métrica e o custo do erro de tipo | Seção 2 |
| 3 | Detectar menções no texto do PDF | Decisão sobre detecção |
| 4 | Baseline por regras | Sistema A |
| 5 | Classificador com metadados e validação agrupada | Sistema B |

O conteúdo sobre DeBERTa com EMA e sobre LLMs fica como leitura complementar dos writeups, sem execução no núcleo.

## 12. Roadmap por fases

Tamanho é uma estimativa relativa: P (pequeno), M (médio), G (grande).

| Fase | Entregável | Critério de saída | Tamanho |
| --- | --- | --- | --- |
| 0 | Decisões abertas resolvidas (seção 15) | Todas as decisões registradas em `docs/decisions/` | P |
| 1 | Dados e Dataset Card | Split agrupado reproduzível e contagens conferidas | M |
| 2 | Métrica e gates com testes | Testes passam; um erro de tipo conta como um falso negativo e um falso positivo | P |
| 3 | Sistema A (regras) | F1 do baseline e tabela de acerto por regra | M |
| 4 | Sistema B (CatBoost) | F1 fora da amostra sem vazamento e importância de features | M |
| 5 | Benchmark e evidência | CSV exportado com gates passando | M |
| 6 | Sistema C (opcional) | Linha no CSV ou justificativa registrada de exclusão | G |
| 7 | Integração com a 13C e aulas de apoio | 13C consome o CSV; aulas de apoio publicadas como Draft | G |
| 8 | Validação | Execução limpa no Kaggle, revisão pedagógica e notebook versionado | M |

## 13. Estrutura no repositório

Usa as pastas do TIL descritas por você (`course/`, `data/`, `docs/`, `experiments/`); os caminhos exatos devem ser conferidos contra o repositório.

```text
docs/experiments/edu-mdc-001/plan.md        # este plano
docs/decisions/                             # decisões da fase 0
experiments/edu-mdc-001/                    # notebook canônico e scripts
data/model-evidence/til-model-evidence.csv  # saída local do CSV
data/manifests/edu-mdc-001.md               # manifesto de inputs (sem os dados)
course/                                     # aulas de apoio, quando publicadas
```

Os dados da competição não são versionados no repositório. Confirme nas regras da competição o que pode ser redistribuído.

## 14. Definição de pronto

Segue os critérios do curso para uma aula estar pronta.

1. O notebook executa no Kaggle com os inputs declarados e internet desligada.
2. Os outputs esperados são observáveis.
3. Warnings e erros relevantes foram avaliados.
4. O conteúdo passou por revisão pedagógica.
5. O notebook canônico está versionado no GitHub.
6. O aluno consegue seguir sem instruções externas.
7. O CSV gerado é consumido pela 13C sem ajuste manual.
8. Os termos novos entraram no Glossário Vivo.

## 15. Decisões em aberto

| # | Decisão | Recomendação | Alternativa |
| --- | --- | --- | --- |
| D1 | Tarefa avaliada: só o tipo sobre menções dadas, ou o pipeline completo (detecção e tipo) | Só o tipo, com detecção reportada à parte | Pipeline completo, mais fiel à competição e mais dependente de fontes externas |
| D2 | Origem dos metadados de artigo e dataset | Confirmar datasets públicos reutilizáveis | Gerar a partir dos arquivos da competição |
| D3 | Tamanho do conjunto | Conjunto completo de treino, se o tempo permitir | Subconjunto fixo menor |
| D4 | Sistema C (LLM) | Opcional, na fase 6 | Fora da primeira versão |
| D5 | Convenção para linhas derivadas no CSV | Alinhar com a 13C | Não exportar a linha D |
| D6 | Nome e numeração finais | `EDU-MDC-001`, seguindo `EDU-ORCH` | A definir por você |
| D7 | Onde ficam as aulas de apoio | Numeradas como apoio ao laboratório | Aulas independentes na trilha |

## 16. Riscos

| Risco | Impacto | Mitigação |
| --- | --- | --- |
| Datasets externos mudarem ou saírem do ar | Sistemas B e C deixam de rodar | Manifesto com nome e versão; teste periódico |
| Rótulos com viés do processo automático | Resultado não generaliza para rótulos humanos | Registrar no Dataset Card e discutir na interpretação |
| Conjunto pequeno (719 rótulos) | Resultados instáveis entre semestres | Split fixo, semente fixa, intervalos de variação entre folds |
| Competição encerrada (Late Submission) | O aluno não disputa prêmio | Dizer isso no notebook e tratar o placar como referência |
| Aceite das regras para baixar dados | Aluno não acessa | Passo explícito no início dos apoios |
| Falta de GPU para o sistema C | Sistema C não roda | Manter opcional; registrar exclusão justificada |

## 17. Glossário: termos novos (PT-BR e EN)

| PT-BR | EN |
| --- | --- |
| Menção a dataset | Dataset mention |
| Identificador de acesso (accession ID) | Accession ID |
| Citação primária e secundária | Primary and secondary data citation |
| Vazamento de dados | Data leakage |
| Validação cruzada agrupada | Grouped cross-validation |
| Rótulo gerado automaticamente | Automatically generated label |
| Regra heurística | Heuristic rule |

## 18. Próximos passos imediatos

1. Resolver D1, D2 e D6, que destravam a fase 1.
2. Rodar o notebook do 1º lugar em uma cópia sua e anotar tempo, inputs e erros.
3. Ler o `EDU-ORCH-002` para alinhar a convenção de linhas derivadas (D5).
4. Registrar as decisões em `docs/decisions/`.
5. Começar pela fase 1 (dados e Dataset Card), a mais barata e a que expõe primeiro os riscos de dados.

## Referências

- [Competição Make Data Count - Finding Data References](https://www.kaggle.com/competitions/make-data-count-finding-data-references)
- [Writeup do 1º lugar](https://www.kaggle.com/competitions/make-data-count-finding-data-references/writeups/1st-place-solution)
- [Writeup do 2º lugar](https://www.kaggle.com/competitions/make-data-count-finding-data-references/writeups/2nd-place-solution)
- [Writeup do 3º lugar](https://www.kaggle.com/competitions/make-data-count-finding-data-references/writeups/3rd-place-solution)
- [Notebook do 1º lugar](https://www.kaggle.com/code/keakohv/mdc-1st-place-solution-catboost-and-qwen)
- [Text Intelligence Lab Course](https://www.kaggle.com/code/pedrogentil/text-intelligence-lab-course)
- [TIL EDU ORCH 001 Baseline vs Transformer](https://www.kaggle.com/code/pedrogentil/til-edu-orch-001-baseline-vs-transformer)
