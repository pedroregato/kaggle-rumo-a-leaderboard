# Make Data Count: comparação dos 3 primeiros lugares

Data da análise: 2026-09-20

## Resumo executivo

Os três primeiros lugares chegaram perto de 0,78 no placar privado (0,797, 0,786 e 0,773) porque descobriram que os rótulos da competição vinham de fontes automatizadas (EuropePMC e DataCite) e passaram a reproduzir essas fontes em vez de extrair menções do texto por conta própria.

A diferença apareceu na classificação Primary/Secondary. O 1º lugar usou CatBoost com metadados (títulos, autores e ano) para DOIs e um LLM zero-shot (Qwen2.5-Coder 32B) para accession IDs. O 2º combinou regras fortes com MedGemma-4B e BiomedBERT. O 3º usou um ensemble de 6 DeBERTa-v3 Large com regras. Os três também omitem accession IDs nos artigos em que já há DOI previsto.

## Contexto: de onde vieram os rótulos

Os rótulos de treino e teste foram montados a partir de menções extraídas por NER do EuropePMC e de mapeamentos artigo-dataset do DataCite, conforme confirmado pelos organizadores. Por isso, quem usou essas fontes como candidatas superou qualquer regex pura, e o desafio real ficou na classificação do tipo.

Duas consequências práticas apareceram nos três writeups. Alguns artigos tinham rótulo "Missing" e não contavam na métrica, então precisavam sair da validação local. E o F1 penaliza mais um erro de tipo (um falso negativo e um falso positivo) do que uma menção não encontrada (um erro só).

## Comparação lado a lado

| Dimensão | 1º · Ali e Kea | 2º · Mohsin e Nikhil | 3º · BRBR |
| --- | --- | --- | --- |
| Score privado | 0,79732 | 0,78648 | 0,77292 |
| Entradas | 389 | 223 | 160 |
| Fonte dos DOIs | Data Citation Corpus v4.1 (origem DataCite) | Corpus v2 + dump público DataCite 2024 | Regex + dump DataCite 2024 + Crossref 2025 |
| Fonte dos accession IDs | Corpus (origem EuropePMC) + mapeamento EuropePMC original, com famílias escolhidas a dedo | Mapeamento EuropePMC, excluindo famílias rótulo "Missing" | Regex + EuropePMC, ignorando GCA_, HGNC e rs* |
| Filtro de precisão | Só IDs presentes no PDF/XML, texto normalizado | Regex tolerante a espaços, sem versão .v1/.v2 do Dryad | Busca no XML só se o PDF falhar; ignora DOIs figshare |
| Classificação de DOI | 6 CatBoost com metadados (título, autores, ano) | Regras + MedGemma-4B com LoRA, contexto por BM25 | Regras + 6 DeBERTa-v3 Large |
| Classificação de accession ID | Qwen2.5-Coder 32B AWQ zero-shot + voto por família | BiomedBERT ajustado (300 caracteres de contexto) | 6 DeBERTa-v3 Large |
| Regras de tipo | Nenhuma forçada por família | SAMN e EMDB Primary; `isSupplementTo` Primary; DOI muito citado Secondary | Dryad e SAMN Primary; título ou autores parecidos Primary; 5+ citações Secondary |
| Se há DOI no artigo | Não prevê accession IDs | Não prevê accession IDs | Ignora o DOI quando há accession IDs |

## 1º lugar: Ali e Kea (0,797)

A aposta foi não extrair nada do texto: as candidatas vêm do Data Citation Corpus v4.1 e do mapeamento EuropePMC, e o texto do artigo só confirma se o ID aparece. No treino, esse filtro de DOIs deu 302 verdadeiros positivos de 325 rótulos, com F1 de 0,935 na etapa de detecção.

Para o tipo de DOI, a classificação é 100% baseada em metadados, sem olhar o contexto no texto. Seis CatBoost, treinados com validação agrupada por artigo, comparam título, autores e ano do artigo e do dataset (via Crossref e DataCite). A similaridade de título e de autores foi o sinal mais forte, com F1 de 0,87 fora da amostra nos DOIs corretamente detectados.

Para accession IDs, um prompt simples e zero-shot no Qwen2.5-Coder 32B (quantizado AWQ, servido com vLLM) lê só o trecho ao redor do ID. As saídas são restritas a A ou B, e depois cada família de repositório recebe o tipo mais frequente naquele artigo. Um ponto de risco: não forçaram SAMN como Primary, o que custou 0,001 no privado.

O 1º lugar reporta também os resultados por parte (privado): só DOI 0,350, só accession 0,664, solução completa 0,797. O tempo total no treino foi de 29 minutos.

## 2º lugar: Mohsin e Nikhil (0,786)

O diferencial foi provar que regras sozinhas já bastavam para medalha de ouro: 0,869 no público e 0,739 no privado, sem modelo. As regras incluem SAMN e EMDB como Primary, `isSupplementTo` no DataCite como Primary, o primeiro artigo de um dataset citado por vários como Primary (os demais Secondary) e mais de 4 ocorrências do mesmo repositório como Secondary. Também acharam que 25 dos 29 falsos positivos vinham de tabelas online-only de um único artigo, e essa regra rendeu +0,003 no público e +0,004 no privado.

Mohsin classificou DOIs com MedGemma-4B e LoRA. O trabalho maior foi montar o contexto (menos de 2048 tokens): trecho ao redor do ID, autores e resumo do dataset (do dump DataCite) e os 3 chunks do artigo mais parecidos com esse resumo por BM25. Com isso e só a regra `isSupplementTo`, chegou a 0,880 público e 0,784 privado.

Nikhil treinou um BiomedBERT separado para DOIs e para accession IDs. Os truques de estabilidade foram: substituir os outros IDs por "other dataset id", marcar o alvo como "Prediction {ID}", usar batch de 256, tirar a média de várias sementes e escolher o limiar no OOF médio. Contexto de 300 caracteres para accession IDs e 150 para DOIs, porque contexto maior gerava overfitting em DOIs. O tempo perdido com um modelo NER e a classificação de accession IDs que teve OOF 0,91 mas LB muito pior foram as maiores dores citadas.

## 3º lugar: BRBR (0,773)

A recuperação é a mais convencional: regex para DOIs e accession IDs, cruzada com o dump DataCite 2024 e com termos minerados do EuropePMC, mais metadados do dump Crossref 2025. Ignoram figshare, GCA_, HGNC e rs* por serem fontes constantes de falsos positivos, e procuram no XML apenas quando o PDF não traz a menção.

O argumento central da equipe é que a classificação pesa mais no F1: um erro de tipo conta duas vezes. Por isso investiram em seis DeBERTa-v3 Large treinados com StratifiedGroupKFold (estratificado por tipo, agrupado por artigo), com entrada de 1.000 caracteres de contexto, 500 caracteres iniciais do artigo e títulos do artigo e do dataset.

As duas técnicas que estabilizaram o treino foram gradient clipping e EMA dos pesos (média móvel exponencial, decay 0,9995, começando após 50 passos). Tentaram também LLMs como Qwen2.5 7B com cabeça de classificação, mas o treino foi instável e os DeBERTa foram mais rápidos e melhores. No teste, rodam dois modelos por vez (um em cada T4) e fazem média simples dos seis, o que permite ajustar o limiar diretamente. Regras antes do modelo: Dryad e SAMN são Primary, título ou autores parecidos (distância de edição) são Primary, e 5 ou mais citações marcam Secondary.

## O que o código do 1º lugar mostra além do writeup

Só o 1º lugar linkou um notebook público ("MDC 1st Place Solution: Catboost and Qwen"). O 2º e o 3º lugar têm apenas o writeup, então esta seção cobre só o 1º.

O notebook roda em 31 minutos em 2 GPUs T4, sem internet, instalando as bibliotecas (PyMuPDF, vLLM, logits-processor-zoo) a partir de um dataset de wheels. A versão publicada tira uma feature duplicada do CatBoost e marca 0,79790 no privado, contra 0,79702 da submissão original segundo o próprio notebook.

| Aspecto | O que o código faz |
| --- | --- |
| Dados de treino | 719 rótulos válidos em 214 artigos, após remover 347 linhas "Missing" |
| Balanceamento | DOIs: 215 Primary e 110 Secondary. Accession IDs: 55 Primary e 339 Secondary |
| Fontes externas | Corpus v4.1 (5,2 mi de linhas EuropePMC e 1,4 mi DataCite), mapeamento EuropePMC (8,4 mi de linhas), Crossref, DataCite, BioSample |
| Busca do DOI no texto | Três estratégias em texto normalizado (NFKC, minúsculas): trecho exato, texto comprimido sem ruído e regex com ruído entre caracteres |
| Resultado da detecção de DOI | 321 previstos, 302 acertos, 19 falsos positivos, 23 perdidos (F1 0,935) |
| Features do CatBoost | Coincidência e diferença de ano, coincidência de editora e periódico, contagem de citações por dataset e por artigo, similaridade de título (TF-IDF, cosseno de caracteres e Jaccard) |
| Cuidado contra vazamento | Similaridade de título calculada dentro de cada fold, com TF-IDF ajustado só no treino |
| Hiperparâmetros | MultiClass, 1000 iterações, taxa 0,03, profundidade 6, parada antecipada em 20, pesos de classe balanceados, bootstrap Bayesiano, 6 folds |

Dois detalhes práticos chamam atenção. A métrica diferencia maiúsculas: DOIs devem sair em minúsculas e accession IDs com a mesma caixa do artigo. E o desbalanceamento inverso entre as duas famílias (DOIs tendem a Primary, accession IDs a Secondary) é coerente com os três times tratarem os dois casos em pipelines separados, embora os writeups não citem isso como motivo.

## Diferenciais, lições e o que não funcionou

O padrão comum aos três é tratar a competição como reconstrução de um processo de rotulagem, não como um problema de NER. Os quatro pontos abaixo separam as escolhas deles.

1. **Recuperação barata, classificação cara.** Todos usam Corpus, DataCite e EuropePMC para achar candidatos e gastam o esforço de modelagem no tipo Primary/Secondary.
2. **Conflito entre DOI e accession ID.** O 1º e o 2º ignoram accession IDs quando o artigo tem DOI previsto. O 3º faz o inverso: ignora o DOI quando há accession IDs. O 1º e o 3º relatam ganho no placar com essa escolha; o 2º a baseou no padrão do treino.
3. **Metadados contra contexto.** O 1º não olha o texto ao redor do DOI. O 2º monta contexto elaborado com BM25. O 3º usa 1.000 caracteres de contexto mais o início do artigo.
4. **Estabilidade com poucos dados.** Com cerca de 700 rótulos, o 1º usou CatBoost, o 2º fez batch grande e média de sementes, e o 3º usou EMA e gradient clipping.

| Time | O que não funcionou |
| --- | --- |
| 1º | Qwen para DOI (pior que CatBoost, mesmo com metadados); relações do DataCite como regra; features de contexto para DOI; prompts mais elaborados, inclusive em chinês; OCR de melhor qualidade (bom, mas lento demais) |
| 2º | Modelo NER (tempo perdido); classificador de accession IDs com OOF 0,91 e LB bem pior; relações do DataCite além de `isSupplementTo` (pouco confiáveis) |
| 3º | Ajustar LLMs (como Qwen2.5 7B) com cabeça de classificação: treino instável, DeBERTa foi mais rápido e melhor |

Uma ressalva vale para qualquer conclusão: os próprios vencedores dizem que a correlação entre validação local e placar foi instável, e que o privado usa cerca de 70% do teste. Por isso, tome as diferenças de cerca de 0,01 entre os três com cautela: os writeups não medem essa margem de erro.

## Uso no Text Intelligence Lab

Este caso serve como estudo didático de três lições de NLP aplicado, cada uma ligada a um dos times. Estas são sugestões de módulos, não algo que os writeups proponham.

1. **Entender a origem dos rótulos antes de modelar** (1º lugar): mostrar como validar contra rótulos gerados por um pipeline e por que descartar artigos "Missing" da validação.
2. **Baseline por regras** (2º lugar): reproduzir as regras de tipo e medir quanto do F1 elas dão sozinhas, antes de treinar qualquer modelo.
3. **Treino estável com poucos dados** (3º lugar): comparar DeBERTa com e sem EMA e gradient clipping em um conjunto de cerca de 700 exemplos.

Um próximo passo natural é rodar o notebook do 1º lugar em uma cópia sua, já que ele é o único com código público, e então reimplementar as regras do 2º e o ensemble do 3º a partir dos writeups para comparar as três abordagens no mesmo conjunto de treino.

## Fontes

Todas as páginas abaixo foram abertas em 20/09/2026.

- [Leaderboard da competição](https://www.kaggle.com/competitions/make-data-count-finding-data-references/leaderboard)
- [Writeup do 1º lugar](https://www.kaggle.com/competitions/make-data-count-finding-data-references/writeups/1st-place-solution)
- [Writeup do 2º lugar (versão atualizada)](https://www.kaggle.com/competitions/make-data-count-finding-data-references/writeups/2nd-place-solution)
- [Writeup do 3º lugar](https://www.kaggle.com/competitions/make-data-count-finding-data-references/writeups/3rd-place-solution)
- [Notebook do 1º lugar: Catboost and Qwen](https://www.kaggle.com/code/keakohv/mdc-1st-place-solution-catboost-and-qwen)
