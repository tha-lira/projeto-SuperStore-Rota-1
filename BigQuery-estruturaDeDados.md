# Documentação Técnica - Estrutura de Dados

### 🔍 Identificar valores nulos

-- 🔎 Contagem total de linhas e verificação de valores nulos em todas as colunas

```
SELECT
  COUNT(*) AS total_linhas, -- Total de registros na tabela
  
  -- Para cada coluna, soma os casos em que o valor é NULL
  SUM(CASE WHEN category IS NULL THEN 1 ELSE 0 END) AS nulos_category,
  SUM(CASE WHEN city IS NULL THEN 1 ELSE 0 END) AS nulos_city,
  SUM(CASE WHEN country IS NULL THEN 1 ELSE 0 END) AS nulos_country,
  SUM(CASE WHEN customer_id IS NULL THEN 1 ELSE 0 END) AS nulos_customer_id,
  SUM(CASE WHEN customer_name IS NULL THEN 1 ELSE 0 END) AS nulos_customer_name,
  SUM(CASE WHEN discount IS NULL THEN 1 ELSE 0 END) AS nulos_discount,
  SUM(CASE WHEN market IS NULL THEN 1 ELSE 0 END) AS nulos_market,
  SUM(CASE WHEN unknown IS NULL THEN 1 ELSE 0 END) AS nulos_unknown,
  SUM(CASE WHEN order_date IS NULL THEN 1 ELSE 0 END) AS nulos_order_date,
  SUM(CASE WHEN order_id IS NULL THEN 1 ELSE 0 END) AS nulos_order_id,
  SUM(CASE WHEN order_priority IS NULL THEN 1 ELSE 0 END) AS nulos_order_priority,
  SUM(CASE WHEN product_id IS NULL THEN 1 ELSE 0 END) AS nulos_product_id,
  SUM(CASE WHEN product_name IS NULL THEN 1 ELSE 0 END) AS nulos_product_name,
  SUM(CASE WHEN profit IS NULL THEN 1 ELSE 0 END) AS nulos_profit,
  SUM(CASE WHEN quantity IS NULL THEN 1 ELSE 0 END) AS nulos_quantity,
  SUM(CASE WHEN region IS NULL THEN 1 ELSE 0 END) AS nulos_region,
  SUM(CASE WHEN row_id IS NULL THEN 1 ELSE 0 END) AS nulos_row_id,
  SUM(CASE WHEN sales IS NULL THEN 1 ELSE 0 END) AS nulos_sales,
  SUM(CASE WHEN segment IS NULL THEN 1 ELSE 0 END) AS nulos_segment,
  SUM(CASE WHEN ship_date IS NULL THEN 1 ELSE 0 END) AS nulos_ship_date,
  SUM(CASE WHEN ship_mode IS NULL THEN 1 ELSE 0 END) AS nulos_ship_mode,
  SUM(CASE WHEN shipping_cost IS NULL THEN 1 ELSE 0 END) AS nulos_shipping_cost,
  SUM(CASE WHEN state IS NULL THEN 1 ELSE 0 END) AS nulos_state,
  SUM(CASE WHEN sub_category IS NULL THEN 1 ELSE 0 END) AS nulos_sub_category,
  SUM(CASE WHEN year IS NULL THEN 1 ELSE 0 END) AS nulos_year,
  SUM(CASE WHEN market2 IS NULL THEN 1 ELSE 0 END) AS nulos_market2,
  SUM(CASE WHEN weeknum IS NULL THEN 1 ELSE 0 END) AS nulos_weeknum

FROM `estrutura-de-dados-473122.dataBase.superstore`;
```

### 🔍 Identificar valores duplicados

-- 🔎 Verifica registros duplicados considerando apenas order_id + product_id

```
-- A ideia é checar se há o mesmo produto repetido dentro de um mesmo pedido.

SELECT 
  order_id, 
  product_id, 
  COUNT(*) AS qtd_duplicados
FROM `estrutura-de-dados-473122.dataBase.superstore`
GROUP BY order_id, product_id
HAVING COUNT(*) > 1;  -- Retorna apenas os pares que aparecem mais de uma vez
```

-- 🔎 Refinamento: verifica duplicados considerando mais colunas (chave lógica mais forte)

```
-- Aqui agrupamos por customer_ID, order_id, product_id, region e product_name
-- para garantir que os casos identificados são realmente duplicações lógicas

SELECT
  customer_ID,
  order_id,
  product_id,
  region,
  product_name,
  COUNT(*) AS qtd_duplicados
FROM `estrutura-de-dados-473122.dataBase.superstore`
GROUP BY customer_ID, order_id, product_id, region, product_name
HAVING COUNT(*) > 1 -- Retorna apenas combinações que se repetem
ORDER BY qtd_duplicados DESC; -- Ordena para visualizar os casos mais frequentes primeiro
```

-- 🔎 Cálculo final do total de duplicados

```
-- Estratégia: criar uma chave composta (customer_ID + order_id + product_id + order_date + ship_date)
-- Essa chave garante unicidade no nível de cliente, pedido, produto e datas
-- Depois, compara o total de linhas com o total de linhas únicas

SELECT 
  COUNT(*) AS total_linhas, -- Contagem total da tabela
  COUNT(DISTINCT CONCAT(customer_ID, order_id, product_id, order_date, ship_date)) AS linhas_unicas, -- Linhas únicas pela chave composta
  COUNT(*) - COUNT(DISTINCT CONCAT(customer_ID, order_id, product_id, order_date, ship_date)) AS qtd_duplicados_chave -- Diferença = duplicados exatos
FROM `estrutura-de-dados-473122.dataBase.superstore`;
```

---

-- 🔎 Criação da tabela sem duplicados 

```
CREATE TABLE `estrutura-de-dados-473122.dataBase.superstore_cleaned` AS
WITH deduplicated AS (
  SELECT *,
    ROW_NUMBER() OVER (
      PARTITION BY customer_id, order_id, product_id, order_date, ship_date
      ORDER BY row_id
    ) AS rn
  FROM `estrutura-de-dados-473122.dataBase.superstore`
)
SELECT *
FROM deduplicated
WHERE rn = 1;
```

📌 O que esse código faz:

- ROW_NUMBER() atribui um número sequencial para cada linha dentro de cada grupo definido por:

```
customer_id, order_id, product_id, order_date, ship_date
```

- PARTITION BY cria grupos onde há duplicatas lógicas.

- ORDER BY row_id garante que a menor row_id seja mantida.

- WHERE rn = 1 mantém apenas uma linha de cada grupo → ou seja, remove duplicados reais.

- A tabela resultante é salva como `superstore_cleaned`.

---

### 🔍 Criaçao das tabelas de fatos e dimensões

1. Tabela Fato – FactSales

```
CREATE OR REPLACE TABLE `estrutura-de-dados-473122.dataBase.FactSales`
PARTITION BY order_date  -- agora usa a coluna DATE criada
CLUSTER BY product_key, customer_key
AS
SELECT
  sc.row_id,
  sc.order_id,
  sc.customer_id,

  -- Surrogate keys determinísticas
  FARM_FINGERPRINT(CAST(sc.customer_id AS STRING)) AS customer_key,
  FARM_FINGERPRINT(CAST(sc.product_id AS STRING)) AS product_key,

  -- Chaves de Data
  CAST(FORMAT_DATE('%Y%m%d', DATE(sc.order_date)) AS INT64) AS order_date_key,
  CAST(FORMAT_DATE('%Y%m%d', DATE(sc.ship_date)) AS INT64) AS ship_date_key,
  DATE(sc.order_date) AS order_date,  -- usada para particionamento

  -- Métricas
  SAFE_CAST(sc.sales AS NUMERIC) AS sales,
  SAFE_CAST(sc.profit AS NUMERIC) AS profit,
  SAFE_CAST(sc.quantity AS INT64) AS quantity,
  SAFE_CAST(sc.discount AS NUMERIC) AS discount,
  SAFE_CAST(sc.shipping_cost AS NUMERIC) AS shipping_cost,

  -- Padronização
  LOWER(TRIM(sc.order_priority)) AS order_priority,

  -- Chaves de dimensão
  CAST(FARM_FINGERPRINT(LOWER(TRIM(sc.ship_mode))) AS STRING) AS ship_mode_id,
  CAST(FARM_FINGERPRINT(CONCAT(
    LOWER(TRIM(sc.region)), '||',
    LOWER(TRIM(sc.city)), '||',
    LOWER(TRIM(sc.state)), '||',
    LOWER(TRIM(sc.country))
  )) AS STRING) AS region_id,
  CAST(FARM_FINGERPRINT(CONCAT(LOWER(TRIM(sc.market)), '||', LOWER(TRIM(sc.market2)))) AS STRING) AS market_id

FROM `estrutura-de-dados-473122.dataBase.superstore_cleaned` sc;
```

2. Tabela de Dimensão - DimCustomer

```
CREATE OR REPLACE TABLE `estrutura-de-dados-473122.dataBase.DimCustomer` AS
SELECT DISTINCT
  customer_id,
  LOWER(TRIM(customer_name)) AS customer_name,
  LOWER(TRIM(segment)) AS segment,
  CAST(FARM_FINGERPRINT(CAST(customer_id AS STRING)) AS STRING) AS customer_key
FROM `estrutura-de-dados-473122.dataBase.superstore_cleaned`
WHERE customer_id IS NOT NULL;
```

3. Tabela de Dimensão - DimProduct

```
CREATE OR REPLACE TABLE `estrutura-de-dados-473122.dataBase.DimProduct` AS
SELECT DISTINCT
  product_id,
  LOWER(TRIM(product_name)) AS product_name,
  LOWER(TRIM(category)) AS category,
  LOWER(TRIM(sub_category)) AS sub_category,
  CAST(FARM_FINGERPRINT(CAST(product_id AS STRING)) AS STRING) AS product_key
FROM `estrutura-de-dados-473122.dataBase.superstore_cleaned`
WHERE product_id IS NOT NULL;
```

4. Tabela de Dimensão - DimDate

```
CREATE OR REPLACE TABLE `estrutura-de-dados-473122.dataBase.DimDate` AS
WITH dates AS (
  SELECT DATE(order_date) AS dt FROM `estrutura-de-dados-473122.dataBase.superstore_cleaned` WHERE order_date IS NOT NULL
  UNION DISTINCT
  SELECT DATE(ship_date) AS dt FROM `estrutura-de-dados-473122.dataBase.superstore_cleaned` WHERE ship_date IS NOT NULL
)
SELECT
  CAST(FORMAT_DATE('%Y%m%d', dt) AS INT64) AS date_key,
  dt AS date,
  EXTRACT(YEAR FROM dt) AS year,
  EXTRACT(MONTH FROM dt) AS month,
  EXTRACT(ISOWEEK FROM dt) AS weeknum,         -- semana ISO
  EXTRACT(DAYOFWEEK FROM dt) AS day_of_week_num, -- 1=Sunday .. 7=Saturday
  FORMAT_DATE('%A', dt) AS day_of_week_name
FROM dates
ORDER BY dt;
```

5. Tabela de Dimensão - DimRegion
 
```
CREATE OR REPLACE TABLE `estrutura-de-dados-473122.dataBase.DimRegion` AS
SELECT DISTINCT
  CAST(FARM_FINGERPRINT(CONCAT(
    LOWER(TRIM(region)), '||',
    LOWER(TRIM(city)), '||',
    LOWER(TRIM(state)), '||',
    LOWER(TRIM(country))
  )) AS STRING) AS region_id,
  LOWER(TRIM(region)) AS region,
  LOWER(TRIM(city)) AS city,
  LOWER(TRIM(state)) AS state,
  LOWER(TRIM(country)) AS country
FROM `estrutura-de-dados-473122.dataBase.superstore_cleaned`
WHERE region IS NOT NULL OR city IS NOT NULL OR state IS NOT NULL OR country IS NOT NULL;
```

6. Tabela de Dimensão - DimShipMode

```
CREATE OR REPLACE TABLE `estrutura-de-dados-473122.dataBase.DimShipMode` AS
SELECT DISTINCT
  CAST(FARM_FINGERPRINT(LOWER(TRIM(ship_mode))) AS STRING) AS ship_mode_id,
  LOWER(TRIM(ship_mode)) AS ship_mode
FROM
  `estrutura-de-dados-473122.dataBase.superstore_cleaned`
WHERE
  ship_mode IS NOT NULL;
```

7. Tabela de Dimensão - DimMarket

```
CREATE OR REPLACE TABLE `estrutura-de-dados-473122.dataBase.DimMarket` AS
SELECT DISTINCT
  CAST(FARM_FINGERPRINT(CONCAT(LOWER(TRIM(market)), '||', LOWER(TRIM(market2)))) AS STRING) AS market_id,
  LOWER(TRIM(market)) AS market,
  LOWER(TRIM(market2)) AS market2
FROM `estrutura-de-dados-473122.dataBase.superstore_cleaned`
WHERE market IS NOT NULL;
```

### 🔍 dentificar dados discrepantes em variáveis ​​numérica

```
-- Investigar se esta associada a alta incidencia de lucro negativo ao desconto gerado ao cliente.

SELECT
  CASE 
    WHEN discount > 0.3 THEN '> 30%'
    WHEN discount > 0.2 THEN '21% a 30%'
    WHEN discount > 0.1 THEN '11% a 20%'
    WHEN discount > 0 THEN '1% a 10%'
    ELSE 'Sem desconto'
  END AS faixa_desconto,
  
  COUNT(*) AS total_registros,
  SUM(CASE WHEN profit < 0 THEN 1 ELSE 0 END) AS registros_lucro_negativo,
  ROUND(100.0 * SUM(CASE WHEN profit < 0 THEN 1 ELSE 0 END) / COUNT(*), 2) AS perc_lucro_negativo
FROM
  `estrutura-de-dados-473122.dataBase.superstore_cleaned`
GROUP BY
  faixa_desconto
ORDER BY
  faixa_desconto;
```
