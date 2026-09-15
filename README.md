# Tech Challenge — Fase 3 | Big Data & Analytics

Pipeline de dados e analytics construído sobre as pesquisas **State of Data Brasil (2021–2025)**, cobrindo todo o ciclo de vida do dado — da ingestão bruta ao relatório executivo — com arquitetura em camadas na AWS.

## 🎯 Objetivo

Transformar as pesquisas anuais do State of Data (Data Hackers/Kaggle) em uma pipeline de dados robusta, respondendo a 7 perguntas de negócio sobre o mercado de dados no Brasil:

1. Como está estruturado o mercado de dados no Brasil hoje?
2. Quais perfis profissionais são mais valorizados?
3. Como está a diversidade de gênero na área?
4. Quais tecnologias e ferramentas estão em alta?
5. Qual o nível de adoção de Inteligência Artificial?
6. Como o mercado varia por região e senioridade?
7. Quais os desafios e oportunidades para quem investe em dados?

## 🏗️ Arquitetura

```
Kaggle (5 CSVs, 2021–2025)
        │  boto3 + kagglehub
        ▼
  S3 · Bronze (CSV bruto, sem tratamento)
        │  Glue Job — PySpark
        ▼
  S3 · Silver (formato longo/EAV, catalogada)
        │  Glue Job — Spark SQL (pivot)
        ▼
  S3 · Gold (dimensões filtradas, catalogada)
        │
        ▼
  Amazon Athena  ──►  Power BI (dashboards e métricas via DAX)
```

**Stack:** Amazon S3 · AWS Glue (PySpark / Spark SQL) · AWS Glue Data Catalog · Amazon Athena · Power BI

## 🧱 Camadas e decisões de design

### Bronze
Apenas os CSVs originais, sem nenhuma transformação, sem Crawler e sem catalogação. O checklist do projeto exige PySpark no *tratamento* dos dados, não na ingestão — catalogar a Bronze seria esforço sem retorno, e o Crawler lidava mal com os cabeçalhos longos e inconsistentes das perguntas da pesquisa.

### Silver
Como cada ano tem um conjunto de colunas (perguntas) diferente, os dados são **despivotados (melt)** para um formato longo (pergunta/resposta), eliminando a necessidade de unificar nomes de coluna manualmente entre os 5 anos. Antes do melt, cada respondente recebe uma chave técnica (`id_resposta`), garantindo que as respostas de uma mesma pessoa possam ser reconstituídas depois. As perguntas são classificadas pelo **código estrutural** embutido (ex.: `P1_a`, `3.f`), não pelo texto, já que o texto muda entre edições da pesquisa mas o código é mais estável.

### Gold
Filtra e pivota apenas as dimensões usadas nas análises (idade, gênero, cargo, faixa salarial, região, tempo de experiência etc.), uma linha por respondente/ano. Não há métricas pré-agregadas nesta camada — percentuais e agregações são calculados no Power BI via DAX, mantendo o pipeline focado em entregar dado limpo e no grão certo.

### Idempotência
Toda escrita nas camadas Silver e Gold é precedida por `purge_s3_path()`, garantindo que reexecutar um Job produza sempre o mesmo resultado, sem duplicação de dados.

## 📊 Dashboards e apresentações

- **Dashboard Power BI**: análise interativa das 7 perguntas de negócio, com gráficos de composição de cargos, distribuição salarial, evolução de diversidade de gênero, ferramentas em alta e adoção de IA.
- **Apresentação executiva**: síntese dos principais insights para tomada de decisão, com recomendações estratégicas.
- **Apresentação técnica**: explicação da arquitetura e do raciocínio por trás de cada decisão de engenharia.

## 📁 Fonte dos dados

Pesquisas *State of Data Brasil* (Data Hackers), disponíveis no Kaggle, edições 2021 a 2025.

## 👤 Autor

Emanoel Dlugokenski — Pós Tech em Data Analytics (FIAP)
