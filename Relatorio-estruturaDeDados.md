# 📊 Projeto de Exploração – Estrutura de Dados (ETL)

### Entendimento do Problema

**Contexto**:
A Super Store precisa organizar e estruturar seus dados para facilitar consultas, análises e decisões estratégicas.

**Objetivo**:
Construir um sistema tabular relacional utilizando o modelo dimensional (star schema), com tabelas fato e dimensão, alimentado por um processo **ETL (Extract, Transform, Load)**.

## 🟦 2.1 Processar e preparar base de dados

### 🔵  Conexão e Importação dos Dados

- **Projeto ID:** `estrutura-de-dados-473122`  
- **Dataset Principal:** `dataBase`  
- **Tabela Bruta:** `superstore`

Os dados foram carregados para o BigQuery, servindo como base inicial para as etapas de limpeza, padronização e modelagem dimensional.

### 🔵 Identificar e tratar valores nulos

Foi aplicada a query `SUM(CASE WHEN coluna IS NULL THEN 1 ELSE 0 END)` em todas as colunas da base `superstore`.  

**Resultado**: Nenhuma coluna apresentou valores nulos entre as 51.290 linhas avaliadas.

✅ Isso indica que a base inicial já possuía consistência em termos de preenchimento de campos obrigatórios.

### 🔵  Identificar e tratar valores duplicados

A análise de duplicidade foi realizada em diferentes níveis:

- `order_id + product_id` → encontrados 38 duplicados.
- ` customer_ID + region + product_name` → total caiu para 33.
- Verificação final: Chave composta (`customer_ID + order_id + product_id + order_date + ship_date`) → 35 duplicados confirmados.

📊 **Resumo da Análise de Duplicados**

| Métrica         | Valor   |
|-----------------|---------|
| Total de linhas | 51.290  |
| Linhas únicas   | 51.255  |
| Duplicados      | 35      |

📌 **Interpretação**: Os 35 registros eram totalmente redundantes, sem novas informações. Se não tratados, poderiam distorcer métricas como vendas, lucro e quantidade.

✅ **Ação Tomada**: Foi criada a tabela intermediária `superstore_cleaned` com as duplicatas removidas, garantindo a integridade das futuras tabelas fato e dimensão.

### 🔵 Identificar e tratar dados discrepantes 

Foi realizada uma análise de consistência em variáveis numéricas e categóricas:

- **Padronização aplicada**: uso de **LOWER()** e **TRIM()** para variáveis textuais, garantindo uniformidade.
  
- **FactSales**: 
- 51.255 registros válidos.

- Nenhum nulo nas métricas principais.

- 12.541 registros (24,5%) com profit < 0.

Esses valores negativos foram investigados em função dos descontos aplicados.

### 🔷 Análise de Lucro Negativo

Os registros de lucro negativo foram segmentados por faixa de desconto:

| Faixa de Desconto | Total de Registros | Registros com Lucro Negativo | % Lucro Negativo |
|-------------------|--------------------|------------------------------|------------------|
| Sem desconto      | 29.446             | 13                           | 0,04%            |
| 1% a 10%          | 4.213              | 888                          | 21,08%           |
| 11% a 20%         | 6.312              | 1.487                        | 23,56%           |
| 21% a 30%         | 924                | 575                          | 62,23%           |
| > 30%             | 10.360             | 9.578                        | 92,45%           |

📌**Interpretação**

- Pedidos sem desconto quase sempre geram lucro.

- À medida que o desconto aumenta, cresce a incidência de prejuízo.

- Acima de 30% de desconto, a maioria esmagadora dos pedidos gera lucro negativo (92,45%).

✅ **Recomendação**: revisar a política de descontos, investigando se os valores decorrem de promoções planejadas ou falhas nos processos de precificação.

### 🔷 Tabela Intermediária Criada

Foi criada a tabela consolidada `superstore_cleaned`, com **51.255 registros válidos**, livre de duplicados e com variáveis padronizadas.

**Variáveis disponíveis:**  
`customer_ID, customer_name, segment, product_id, product_name, category, sub_category, order_id, row_id, order_date, ship_date, sales, profit, quantity, discount, shipping_cost, order_priority, ship_mode, region, city, state, country, market, market2, year, weeknum, unknown, rn`

### 🔵 Pesquisar dados de outras fontes

Para enriquecer a análise, foram integrados dados de concorrentes internacionais.

📌 **Coleta dos Dados**
A extração foi realizada por meio de **web scraping** com a função `IMPORTHTML` do Google Planilhas, aplicada à página da Wikipedia:

```
=IMPORTHTML("https://en.wikipedia.org/wiki/List_of_supermarket_chains","table",1)
```

A tabela original apresentava as seguintes colunas: Company, Headquarters, Served countries, Map, Number of locations e Number of employees.

📌**Tratamento**

**Mantidas**: Company, Headquarters, Served countries (colunas consistentes).
**Descartadas**: Map, Number of locations, Number of employees (alto volume de nulos e baixa relevância).
**Tratamento extra**: registros sem valores em Company ou Headquarters foram removidos.

**Resultado Final**
Tabela com informações limpas de empresas, suas sedes e países de atuação. Esse conjunto poderá ser usado como benchmarking internacional para comparar a presença da Super Store frente a grandes redes globais.

📚 Fonte dos Dados
👉 [Wikipedia – List of supermarket chains](https://en.wikipedia.org/wiki/List_of_supermarket_chains)

### 🔵 Projetar estrutura de base de dados (tabelas de fatos e dimensões)

Foi adotado o modelo dimensional (star schema):

Tabelas de Dimensão: armazenam atributos descritivos (quem, o quê, onde, quando).

Tabela Fato (FactSales): centraliza eventos transacionais com métricas (vendas, lucro, quantidade).

📌 **Boas práticas aplicadas**:

Uso de LOWER() e TRIM() → padronização textual.

Deduplicação de dimensões.

Chaves substitutas (FARM_FINGERPRINT) → garantem unicidade, mantendo IDs naturais para rastreabilidade.

📌 **Representação Visual**:
O diagrama foi elaborado no Lucidchart, com a FactSales no centro, conectada às dimensões DimCustomer, DimProduct, DimDate, DimRegion, DimShipMode e DimMarket.

👉 [Visualizar o diagrama](https://github.com/tha-lira/projeto-SuperStore-Rota-1/blob/main/Modelo%20Dimensional%20-%20An%C3%A1lise%20de%20Vendas%20(Star%20Schema).pdf)

### 🔵 Criar estrutura de base de dados (tabelas de fatos e dimensões)

**FactSales**: núcleo central do modelo, com métricas de vendas.

**DimCustomer, DimProduct, DimDate, DimRegion, DimShipMode, DimMarket**: tabelas descritivas de suporte.

📌 **Observações Técnicas**:

Modelo em estrela → consultas OLAP mais simples e rápidas.

Chaves primárias das dimensões usadas como FK na fato.

Surrogate keys via FARM_FINGERPRINT → unicidade e joins eficientes.

FactSales contém apenas dados normalizados e referenciados.

👉 [Estrutura detalhada do modelo dimensional](https://github.com/tha-lira/projeto-SuperStore-Rota-1/blob/main/Estrutura_Modelo_Dimesional.md)
  
### 🔵  Agendar atualizações de tabelas

Foi projetado um pipeline lógico de atualização, considerando dependências:

1. Superstore (bruta): dados de entrada.

2. superstore_cleaned (intermediária): remoção de duplicatas e padronização.

3. Dimensões: atualizadas em seguida (DimCustomer, DimProduct, DimDate, DimRegion, DimShipMode, DimMarket).

4. FactSales: última a ser atualizada, consolidando métricas e conectando dimensões.

⚠️ Neste projeto, o pipeline foi projetado apenas conceitualmente, sem automação.

### 🔵 Conclusão Final e Próximos Passos

📌 **Principais Entregas**

**Qualidade dos Dados**:

- Nenhum nulo.

- 35 duplicados removidos.

- Análise de lucros negativos relacionada a descontos.

**Tabela Intermediária**:

- superstore_cleaned como base confiável para modelagem.

**Dados Externos**:

- Benchmarking internacional com supermercados multinacionais.

**Modelagem Dimensional**:

- Estrutura em estrela no BigQuery.

- Diagrama criado no Lucidchart.

**Pipeline (conceitual)**:

- Sequência lógica definida para atualização das tabelas.

📊 **Benefícios da Solução**

- Garantia de integridade e consistência dos dados.

- Estrutura escalável e otimizada para consultas analíticas.

- Possibilidade de análises sob múltiplas perspectivas (clientes, produtos, regiões, períodos, modos de envio).

- Base integrada para benchmarking internacional.

🚀** Próximos Passos**

1. Automatizar o pipeline de atualização (Airflow, Dataflow ou Cloud Composer).

2. Aplicar Slowly Changing Dimensions (SCD) para rastrear mudanças históricas em dimensões (ex.: mudança de região ou segmento de cliente).

3. Construir dashboards no Power BI ou Looker Studio para apoiar decisões gerenciais.

4. Implantar monitoramento de qualidade dos dados (checagem periódica de nulos, duplicados e discrepâncias).
