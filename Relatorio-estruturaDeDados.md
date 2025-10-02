# 📊 Projeto de Exploração – Estrutura de Dados (ETL)

1. Entendimento do Problema
   
**Contexto**: A Super Store precisa organizar e estruturar seus dados para facilitar consultas, análises e decisões estratégicas.

**Objetivo**: Construir um sistema tabular relacional utilizando o modelo dimensional (star schema), com tabelas fato e dimensão, alimentado por um processo ETL (Extract, Transform, Load).

2. Processamento e Preparação da Base de Dados

2.1 Conexão e Importação dos Dados

- Projeto ID: estrutura-de-dados-473122

- Dataset Principal: dataBase

- Tabela Bruta: superstore

Os dados foram carregados para o BigQuery, servindo como base inicial para as etapas de limpeza, padronização e modelagem dimensional.

2.2 Identificação e Tratamento de Valores Nulos

Foi aplicada a query SUM(CASE WHEN coluna IS NULL THEN 1 ELSE 0 END) em todas as colunas da base superstore.

Resultado: Nenhuma coluna apresentou valores nulos entre as 51.290 linhas avaliadas.

Isso indica que a base inicial já possuía consistência em termos de preenchimento de campos obrigatórios.

2.3 Identificação e Tratamento de Valores Duplicados

A análise de duplicidade foi realizada em diferentes níveis:

- order_id + product_id → encontrados 38 duplicados.

- customer_ID + region + product_name → total caiu para 33.

- Verificação final: Chave composta (customer_ID + order_id + product_id + order_date + ship_date) → 35 duplicados confirmados.

| Métrica         | Valor   |
|-----------------|---------|
| Total de linhas | 51.290  |
| Linhas únicas   | 51.255  |
| Duplicados      | 35      |

Interpretação: Os 35 registros eram totalmente redundantes, sem novas informações. Se não tratados, poderiam distorcer métricas como vendas, lucro e quantidade.

Ação tomada: Foi criada a tabela intermediária superstore_cleaned com as duplicatas removidas, garantindo a integridade das futuras tabelas fato e dimensão.

2.4 Identificação e Tratamento de Dados Discrepantes

Análise de consistência em variáveis numéricas e categóricas:

- Padronização aplicada: uso de LOWER() e TRIM() para variáveis textuais, garantindo uniformidade.

- Na tabela FactSales:

  - 51.255 registros válidos.
  
  - Nenhum nulo nas métricas principais.

12.541 registros (24,5%) com lucro (profit) negativo.

Esses valores negativos foram investigados em função dos descontos aplicados.

2.5 Tabela Intermediária Criada

Foi criada a tabela consolidada superstore_cleaned, com 51.255 registros válidos, livre de duplicados e com variáveis padronizadas.

Variáveis disponíveis:
customer_ID, customer_name, segment, product_id, product_name, category, sub_category, order_id, row_id, order_date, ship_date, sales, profit, quantity, discount, shipping_cost, order_priority, ship_mode, region, city, state, country, market, market2, year, weeknum, unknown, rn

2.6 Pesquisa e Integração de Dados de Outras Fontes

Para enriquecer a análise, foram integrados dados de concorrentes internacionais.

Coleta dos Dados:
Extração via web scraping com a função IMPORTHTML do Google Planilhas, aplicada à página da Wikipedia:

```
=IMPORTHTML("https://en.wikipedia.org/wiki/List_of_supermarket_chains","table",1)
```

A tabela original possuía as colunas: Company, Headquarters, Served countries, Map, Number of locations e Number of employees.

Tratamento:

- Mantidas: Company, Headquarters, Served countries (colunas consistentes).

- Descartadas: Map, Number of locations, Number of employees (alto volume de nulos e baixa relevância).

- Registros sem valores em Company ou Headquarters foram removidos.

Resultado Final:
Tabela limpa com informações sobre empresas, suas sedes e países de atuação, para benchmarking internacional da presença da Super Store frente a grandes redes globais.

👉 [Wikipedia – List of supermarket chains](https://en.wikipedia.org/wiki/List_of_supermarket_chains)

---

3. Modelagem Dimensional e Organização da Base de Dados
   
3.1 Definição do Modelo Dimensional (Star Schema)

Foi adotado o modelo estrela (star schema), amplamente utilizado em ambientes analíticos por permitir consultas otimizadas e interpretações mais intuitivas.

- Tabela Fato (FactSales): centraliza os eventos transacionais (vendas, lucro, quantidade etc.).

- Tabelas de Dimensão: descrevem os atributos relacionados às entidades envolvidas (cliente, produto, tempo, região etc.).

🔍 Resumo das Dimensões:

- DimCustomer: informações dos clientes

- DimProduct: dados dos produtos vendidos

- DimDate: dimensões temporais

- DimRegion: dados geográficos

- DimShipMode: modos de envio

- DimMarket: mercados de atuação

📌 Boas Práticas Aplicadas:

| Prática                                              | Objetivo                                         |
| ---------------------------------------------------- | ------------------------------------------------ |
| `LOWER()` e `TRIM()`                                 | Padronização textual                             |
| Deduplicação de dimensões                            | Evita redundâncias e inconsistências             |
| Chaves substitutas (`FARM_FINGERPRINT`)              | Garante unicidade e melhora performance em joins |
| Separação clara entre IDs naturais e chaves técnicas | Rastreabilidade e normalização                   |

👉 [Visualizar o diagrama](https://github.com/tha-lira/projeto-SuperStore-Rota-1/blob/main/Modelo%20Dimensional%20-%20An%C3%A1lise%20de%20Vendas%20(Star%20Schema).pdf)

3.2 Implementação das Tabelas Fato e Dimensão

Tabela Fato – FactSales:

- Armazena os dados transacionais (vendas, lucros, quantidades).

- Conectada às dimensões por chaves substitutas.

- Apenas dados limpos e padronizados são carregados.

Tabelas de Dimensão:

- DimCustomer

- DimProduct

- DimDate

- DimRegion

- DimShipMode

- DimMarket

🧩 Aspectos Técnicos Importantes:

- Modelo em estrela (star schema) otimiza operações OLAP.

- Uso de surrogate keys (FARM_FINGERPRINT) para garantir unicidade.

- Tabelas de dimensão alimentam visualizações com filtros e agrupamentos.

👉 [Estrutura detalhada do modelo dimensional](https://github.com/tha-lira/projeto-SuperStore-Rota-1/blob/main/Estrutura_Modelo_Dimesional.md)
  
3.3 Projeto do Pipeline de Atualização

Foi desenhado um pipeline lógico com as seguintes etapas:

1. Tabela Bruta (superstore)
Dados originais carregados no BigQuery.

2. Tabela Intermediária (superstore_cleaned)
Aplicação de limpeza (remoção de duplicatas, padronização textual).

3. Criação das Dimensões
Enriquecimento com chaves substitutas e deduplicação.

4. Construção da Tabela Fato (FactSales)
Consolidação de métricas, uso de FKs e referências às dimensões.

⚠️ Observação:

O pipeline ainda não foi automatizado, mas está documentado e pronto para futura orquestração com ferramentas como Cloud Composer, Apache Airflow ou Dataform.

3.4 Representação Visual do Pipeline

O fluxo de atualização dos dados segue a seguinte sequência lógica:


***/*/*/*/*/*/

📌 Esse pipeline garante uma estrutura modular e escalável, facilitando futuras automações com ferramentas como Airflow, Dataflow ou Cloud Composer.

3.5 Considerações sobre Slowly Changing Dimensions (SCD)

O projeto contempla suporte futuro a **dimensões historicamente variantes (Slowly Changing Dimensions - SCD)**, fundamentais para garantir rastreabilidade e integridade temporal.

#### Estratégias propostas:

| Situação                          | Tipo SCD | Tratamento Proposto                                  |
|----------------------------------|----------|------------------------------------------------------|
| Cliente muda de segmento         | Tipo 2   | Criar nova linha com nova chave e vigência temporal  |
| Produto muda de categoria        | Tipo 1   | Sobrescrever valor diretamente na dimensão           |
| Subcategoria alterada parcialmente | Tipo 3 | Adicionar nova coluna para manter valor anterior     |

📌 Essas práticas asseguram que mudanças importantes nas entidades não impactem análises históricas, nem distorçam relatórios gerenciais.

3.6 Performance e Escalabilidade  

O modelo foi desenvolvido considerando práticas de otimização para ambientes em nuvem (BigQuery).

#### Técnicas aplicadas:

| Técnica                                | Benefício                                                       |
|----------------------------------------|------------------------------------------------------------------|
| `PARTITION BY order_date`              | Melhora performance em filtros por data, reduz custo de leitura |
| `CLUSTER BY product_key, customer_key` | Acelera joins e agregações por produto e cliente                |

📌 Isso torna o ambiente preparado para grandes volumes de dados e consultas de alta complexidade, com custo otimizado.


































---

4. Análise de Dados e Regras de Governança
   
4.1 Análise de Lucro Negativo por Faixa de Desconto

Foi realizada uma segmentação dos pedidos com lucro negativo, cruzando com as faixas de desconto aplicadas:

| Faixa de Desconto | Total de Registros | Registros com Lucro Negativo | % Lucro Negativo |
|-------------------|--------------------|------------------------------|------------------|
| Sem desconto      | 29.446             | 13                           | 0,04%            |
| 1% a 10%          | 4.213              | 888                          | 21,08%           |
| 11% a 20%         | 6.312              | 1.487                        | 23,56%           |
| 21% a 30%         | 924                | 575                          | 62,23%           |
| > 30%             | 10.360             | 9.578                        | 92,45%           |

📌 Interpretação:

- Pedidos sem desconto praticamente não geram prejuízo.

- A incidência de lucro negativo cresce exponencialmente conforme o desconto aumenta.

- Com descontos acima de 30%, mais de 90% dos pedidos resultam em prejuízo.

✅ Recomendação: revisar a política de descontos. Investigar se os prejuízos são estratégicos (ex: campanhas promocionais) ou indesejados (ex: erro de precificação, concessão manual não controlada).

4.2 Regras de Negócio Críticas e Governança de Dados

Foram definidas regras de negócio essenciais para garantir integridade, confiabilidade e rastreabilidade dos dados analíticos.

🔎 **Métricas a serem sempre validadas**

| Métrica    | Regra de Validação                            | Justificativa                                                 |
|------------|-----------------------------------------------|---------------------------------------------------------------|
| `profit`   | Não deve ser negativo sem justificativa clara | Margem negativa pode indicar prejuízo ou erro de precificação |
| `discount` | Valores acima de 30% devem ser sinalizados    | Descontos elevados afetam o lucro diretamente                 |
| `quantity` | Quantidade vendida deve ser positiva          | Evita registros incorretos ou devoluções não tratadas         |

⚠️ Ações Recomendadas:

- Criar flags automáticas:

  - is_profit_negative
  
  - is_discount_outlier

- Implementar checagens no pipeline ETL (durante ou antes da carga na FactSales).

- Criar alertas ou relatórios automáticos para monitoramento contínuo.

👉 Essas práticas promovem governança de dados, facilitando auditoria, conformidade e qualidade analítica.


5. Visão Analítica e Aplicação Prática
   
5.1 Exploração do Modelo em Ferramentas de BI

Com a estrutura dimensional pronta, o próximo passo natural é conectar o modelo a ferramentas de visualização para análises estratégicas.

💡 Recomendações:

- Conectar ao Power BI, Looker Studio, ou outra plataforma de BI.

- Criar dashboards interativos com indicadores de desempenho (KPIs), permitindo cortes por tempo, região, produto e cliente.

📊 Visualizações sugeridas:

- Gráfico de rentabilidade por faixa de desconto

- Ranking de produtos mais vendidos

- Análise de margem por categoria/subcategoria

- Evolução temporal das vendas (ano, mês, semana)

- Comparativo da atuação da Super Store vs concorrentes (benchmark externo)

- Filtros dinâmicos por segmento de cliente, região e período
