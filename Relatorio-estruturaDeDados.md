# 📊 Projeto de Exploração – Estrutura de Dados (ETL)

### Entendimento do Problema

**Contexto:**  
A Super Store precisa organizar e estruturar seus dados para facilitar consultas, análises e decisões estratégicas.

**Objetivo:**  
Construir um sistema tabular relacional utilizando o modelo dimensional, com tabelas fato e dimensão, alimentado por um processo ETL (Extract, Transform, Load).

---

### 🔵  Conexão e Importação dos Dados

- **Projeto ID:** `estrutura-de-dados-473122`  
- **Dataset Principal:** `dataBase`  
- **Tabela Bruta:** `superstore`

---

## Qualidade dos Dados

### 🔵  Verificação de nulos

A query `SUM(CASE WHEN coluna IS NULL THEN 1 ELSE 0 END)` foi aplicada para todas as colunas da base `superstore`.  

**Resultado:** Nenhuma coluna apresentou valores nulos entre as 51.290 linhas avaliadas.

---

### 🔵  Identificação de duplicados

Foi realizada uma análise de duplicidade em diferentes níveis:

- Inicialmente, 38 duplicatas foram encontradas usando `order_id + product_id`.
- Refinando com `customer_ID + region + product_name`, o total caiu para 33.
- A verificação final considerou a chave composta:

**Resultado final:**

| Métrica         | Valor   |
|-----------------|---------|
| Total de linhas | 51.290  |
| Linhas únicas   | 51.255  |
| Duplicados      | 35      |

### Interpretação

Os 35 registros duplicados eram idênticos em todas as colunas-chave, indicando redundância sem novas informações. A presença dessas duplicatas poderia distorcer as análises de vendas, lucro e quantidade.

### ✅ Ação Tomada

Foi criada a tabela intermediária `superstore_cleaned` com as duplicatas removidas, garantindo a integridade das futuras tabelas fato e dimensão.

---

### 🔵 Identificar Dados Discrepantes

Foi realizada uma consulta SQL para identificar possíveis inconsistências em variáveis numéricas da base de dados, tais como valores nulos, valores negativos ou valores fora do intervalo esperado.

Durante a criação das tabelas fato e dimensão, foi aplicado o uso da função `LOWER()` para as variáveis categóricas, garantindo a padronização dos dados em letras minúsculas. Isso evita inconsistências causadas por variações de maiúsculas/minúsculas e facilita análises posteriores.

Foi realizada a verificação das principais tabelas da base de dados para identificar inconsistências  numéricas e textuais.

Na tabela **FactSales**, que contém 51.255 registros, não foram encontrados valores nulos nas chaves ou métricas principais. No entanto, foram identificados 12.541 registros com valores negativos na coluna de lucro (`profit`). Isso pode indicar devoluções, descontos ou ajustes financeiros que precisam ser analisados.

Nas tabelas de dimensão **DimCustomer**, **DimProduct**, **DimDate**, **DimRegion**, **DimShipMode** e **DimMarket**, não foram encontradas inconsistências relevantes, como valores nulos ou fora do esperado, nas colunas analisadas.

### 🔷 Análise de Lucro Negativo

A análise da tabela FactSales mostrou que 12.541 registros (24,5%) apresentaram profit < 0. Para investigar esse padrão, os registros foram segmentados por faixa de desconto, avaliando a relação entre descontos concedidos e ocorrência de prejuízo

### Resultados da Análise

| Faixa de Desconto | Total de Registros | Registros com Lucro Negativo | % Lucro Negativo |
|-------------------|--------------------|------------------------------|------------------|
| Sem desconto      | 29.446             | 13                           | 0,04%            |
| 1% a 10%          | 4.213              | 888                          | 21,08%           |
| 11% a 20%         | 6.312              | 1.487                        | 23,56%           |
| 21% a 30%         | 924                | 575                          | 62,23%           |
| > 30%             | 10.360             | 9.578                        | 92,45%           |

### 📌 Interpretação

- A grande maioria dos pedidos sem desconto apresenta lucro positivo.
- À medida que o desconto aumenta, a porcentagem de registros com lucro negativo cresce significativamente.
- Pedidos com desconto superior a 30% apresentam uma alta incidência (92,45%) de lucro negativo, indicando que esses descontos provavelmente estão causando prejuízo nas vendas.

### ✅ Considerações Finais

- Esse padrão sugere que o lucro negativo está fortemente associado a descontos elevados.
- É importante investigar se os descontos altos são intencionais (promoções ou liquidações) ou representam problemas como erros de cálculo ou políticas inadequadas de desconto.
- Recomenda-se monitorar e revisar a política de descontos para assegurar a saúde financeira do negócio.

Essa análise ajuda a entender o impacto dos descontos no lucro e direciona ações para melhorar a rentabilidade.

---

### 📌 Tratamento e Criação da Tabela Intermediária

Tabela Intermediária: `superstore_cleaned`

**Variáveis disponíveis:**  
`customer_ID, customer_name, segment, product_id, product_name, category, sub_category, order_id, row_id, order_date, ship_date, sales, profit, quantity, discount, shipping_cost, order_priority, ship_mode, region, city, state, country, market, market2, year, weeknum, unknown, rn`

---

### Pesquisar dados de outras fontes

Integrar dados de concorrentes multinacionais ao projeto da Super Store por meio de extração automática de informações (web scraping), utilizando a função IMPORTHTML do Google Planilhas.

Ferramenta Utilizada, Foi o Google Planilhas com a função:
```
=IMPORTHTML("https://en.wikipedia.org/wiki/List_of_supermarket_chains","table",1)
```
📚 Fonte dos Dados

Página da Wikipedia:
https://en.wikipedia.org/wiki/List_of_supermarket_chains

A tabela utilizada contém cadeias de supermercados multinacionais e regionais, organizadas por país e continente.

### 🔵   Projetar estrutura de base de dados (tabelas de fatos e dimensões)

Foi adotado o **modelo dimensional** (star schema), em que:

- Tabelas de dimensão armazenam atributos descritivos das entidades do negócio (quem, o quê, onde, quando).

- Tabela fato registra eventos transacionais com métricas quantitativas (vendas, lucro, quantidade).

As tabelas foram criadas a partir da base limpa (`superstore_cleaned`) utilizando comandos `CREATE TABLE AS SELECT`, promovendo consistência, organização e melhor performance em consultas analíticas.

### 📌 Padronização dos Dados

Durante a construção das tabelas, as variáveis categóricas foram padronizadas com LOWER() para evitar discrepâncias de caixa (maiúsculas/minúsculas). Além disso, as dimensões foram deduplicadas e receberam chaves estáveis geradas por `FARM_FINGERPRINT` (surrogate keys), mantendo os identificadores naturais (ex.: `customer_ID`, `product_id`) para rastreabilidade.

### 🔵  Criar estrutura de base de dados (tabelas de fatos e dimensões)

### 📌 Observações Técnicas

- Todas as tabelas foram construídas com base no conceito de **modelagem estrela**.
- As **chaves primárias** das tabelas de dimensão foram utilizadas como **chaves estrangeiras** na tabela fato.
- Para manter unicidade e performance, IDs foram gerados usando funções de hash (`FARM_FINGERPRINT`) em colunas compostas.
- A tabela `FactSales` contém apenas dados normalizados e referenciados, mantendo o armazenamento eficiente e preparado para consultas OLAP.

--->  Link
  
### 🔵  Agendar atualizações de tabelas
