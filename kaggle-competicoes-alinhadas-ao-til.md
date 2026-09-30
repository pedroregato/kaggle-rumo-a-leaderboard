# Competições do Kaggle alinhadas ao TIL

- **Data da pesquisa:** 2026-09-20
- **Fontes:** lista oficial de competições do Kaggle filtrada pela tag NLP e a visão geral de cada competição listada abaixo
- **Escopo:** nove competições candidatas; só as visões gerais foram lidas, não os writeups dos vencedores

## Resumo

Há quatro competições contínuas ou de conhecimento e cinco competições encerradas que cobrem, juntas, quase toda a trilha do TIL, da classificação com TF-IDF (Aulas 4 a 7) até agentes com ferramentas (Aulas 17 a 19). As contínuas permitem que o aluno submeta a qualquer momento; as encerradas oferecem dados e discussões públicas para estudo.

Nenhuma competição em português foi encontrada: a busca por "portuguese" na página de competições do Kaggle voltou vazia. A segunda página da lista de NLP não carregou, então a lista pode estar incompleta.

## Panorama

| Competição | Status | Tarefa | Métrica | Escala (times) |
| --- | --- | --- | --- | --- |
| [Natural Language Processing with Disaster Tweets](https://www.kaggle.com/competitions/nlp-getting-started) | Contínua (Getting Started) | Classificação binária: o tweet fala de um desastre real? | F1 | 454 |
| [Contradictory, My Dear Watson](https://www.kaggle.com/competitions/contradictory-my-dear-watson) | Contínua (Getting Started) | Contradição, implicação ou neutro em texto multilíngue | Acurácia | 50 |
| [LLM Classification Finetuning](https://www.kaggle.com/competitions/llm-classification-finetuning) | Contínua; não concede pontos nem medalhas | Prever qual resposta de chatbot o usuário prefere (dados do Chatbot Arena) | Log loss | 232 |
| [Learning Agency Lab - Automated Essay Scoring 2.0](https://www.kaggle.com/competitions/learning-agency-lab-automated-essay-scoring-2) | Encerrada (2024) | Pontuar redações de estudantes | Cohen Kappa | 2.706 |
| [MAP - Charting Student Math Misunderstandings](https://www.kaggle.com/competitions/map-charting-student-math-misunderstandings) | Encerrada (jul a out/2025) | Classificar equívocos matemáticos em explicações abertas de alunos | MAP@3 | 1.857 |
| [Eedi - Mining Misconceptions in Mathematics](https://www.kaggle.com/competitions/eedi-mining-misconceptions-in-mathematics) | Encerrada (set a dez/2024) | Ligar equívocos a alternativas erradas de questões de múltipla escolha | MAP@25 | 1.446 |
| [Kaggle - LLM Science Exam](https://www.kaggle.com/competitions/kaggle-llm-science-exam) | Encerrada (jul a out/2023) | Responder perguntas científicas difíceis, no formato open-book | MAP@3 | 2.664 |
| [U.S. Patent Phrase to Phrase Matching](https://www.kaggle.com/competitions/us-patent-phrase-to-phrase-matching) | Encerrada (mar a jun/2022) | Similaridade entre frases de patentes | Correlação de Pearson | 1.889 |
| [Information Retrieval Agent 2025](https://www.kaggle.com/competitions/information-retrieval-agent-2025) | Encerrada (nov a dez/2025) | Agente que monta um dataset de 5.000 páginas da Wikipedia com limite de chamadas à API | Métrica personalizada | 222 |

O número de times de cada competição foi lido na página de participação. A competição de Disaster Tweets e a de Watson também aparecem na lista de competições ativas com a tag NLP.

## Mapa por aula do TIL

Sugestões minhas, baseadas no tema e na métrica de cada competição; não substituem a leitura dos writeups.

| Aula do TIL | Competição sugerida | Uso pedagógico |
| --- | --- | --- |
| 4. TF-IDF | Automated Essay Scoring 2.0 | Baseline com TF-IDF em textos longos, com métrica de concordância |
| 5. Primeiro classificador de textos | Disaster Tweets | Classificação binária curta, com dataset pequeno que roda em notebook |
| 6. Avaliação de classificadores | Disaster Tweets; MAP | F1 e análise de erros; ranking com MAP@3 |
| 7. Seleção de modelos e tuning | Disaster Tweets | Comparar modelos e ajustar hiperparâmetros com submissões reais |
| 9. Word Embeddings | U.S. Patent Phrase to Phrase Matching; Eedi | Similaridade semântica e busca por proximidade |
| 10 e 11. Transformers e BERT | Watson; MAP; Patent Phrase | Ajuste fino de modelos em classificação, inferência e similaridade |
| 12. Baselines clássicos fortes | Essay Scoring 2.0; Disaster Tweets; Watson | Medir quanto um baseline forte deixa para o Transformer ganhar |
| 13 e 13B. Métricas e cenários | Essay Scoring 2.0 (Cohen Kappa); MAP (MAP@k); LLM Classification (log loss); Patent (Pearson) | Quatro famílias de métrica diferentes do F1 macro já usado no `EDU-ORCH-001` |
| 13C. Roteamento e utilidade | LLM Classification Finetuning | Decidir entre modelos com base em probabilidades e custo |
| 14. Fundamentos de LLMs | LLM Classification Finetuning | Comparação de respostas de LLMs com dados do Chatbot Arena |
| 15. Retrieval e busca semântica | Eedi; LLM Science Exam | Recuperar candidatos por similaridade; formato open-book |
| 16. RAG | LLM Science Exam | Perguntas que se beneficiam de recuperar contexto antes de responder |
| 17 a 19. Ferramentas, workflows e MCP | Information Retrieval Agent 2025 | Agente com API externa e orçamento de chamadas |

## Sugestões de uso nos Evidence Labs

1. **Métricas alternativas.** O `EDU-ORCH-001` mede F1 macro. Um laboratório sobre Essay Scoring 2.0 (Cohen Kappa) ou LLM Classification (log loss) mostraria a alunos como a escolha da métrica muda a comparação entre sistemas.
2. **Retrieval.** O Eedi permite um laboratório de recuperação por semelhança, que prepara as Aulas 15 e 16.
3. **Ferramentas.** A Information Retrieval Agent 2025 tem a estrutura de um agente com orçamento de chamadas, útil para a Aula 17. A página menciona entrega no Moodle, o que sugere um trabalho de curso e não uma competição aberta; a reutilização precisa ser avaliada.

## O que não foi verificado

- Os writeups dos vencedores de cada competição.
- A política de internet e de recursos de cada competição de código; o TIL executa com internet desligada por padrão, então cada uma deve ser conferida.
- Licença e regras de redistribuição dos dados.
- Se o Disaster Tweets e o Watson têm variantes em português.
- A competição de NLP da IOAI 2026, cuja página não permitiu confirmar datas e status.

## Fontes

- [Competições do Kaggle, filtro NLP](https://www.kaggle.com/competitions?tagIds=13204-NLP)
- [Competições do Kaggle, NLP ativas](https://www.kaggle.com/competitions?listOption=active&tagIds=13204-NLP)
- Páginas de cada competição, linkadas na tabela do panorama.
