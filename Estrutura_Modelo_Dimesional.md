
## 🔹 Estrutura da Tabela Fato: `FactSales`

A tabela `FactSales` armazena os dados transacionais do processo de vendas e se conecta às tabelas de dimensão para análises analíticas. Abaixo estão os campos com suas descrições:

| Nome do campo  | Tipo    | Descrição                                        |
| -------------- | ------- | ------------------------------------------------ |
| row_id         | INTEGER | Identificador da linha original da fonte         |
| order_id       | STRING  | Identificador único do pedido                    |
| customer_id    | STRING  | ID natural do cliente (rastreabilidade)          |
| customer_key   | INTEGER | Chave substituta do cliente (para joins)         |
| product_key    | INTEGER | Chave substituta do produto (para joins)         |
| order_date_key | INTEGER | Chave da data do pedido (formato YYYYMMDD)       |
| ship_date_key  | INTEGER | Chave da data de envio (formato YYYYMMDD)        |
| order_date     | DATE    | Data do pedido (coluna usada no particionamento) |
| sales          | NUMERIC | Valor de vendas                                  |
| profit         | NUMERIC | Lucro associado                                  |
| quantity       | INTEGER | Quantidade vendida                               |
| discount       | NUMERIC | Desconto aplicado                                |
| shipping_cost  | NUMERIC | Custo de envio                                   |
| order_priority | STRING  | Prioridade do pedido                             |
| ship_mode_id   | STRING  | Chave substituta do modo de envio                |
| region_id      | STRING  | Chave substituta da região                       |
| market_id      | STRING  | Chave substituta do mercado                      |

## 🔹 Tabelas de Dimensão

As tabelas de dimensão armazenam dados descritivos relacionados às entidades envolvidas nas transações da tabela fato. Elas são usadas para enriquecer a análise e permitir cortes por atributos específicos.

#### 📘 `DimCustomer`  

Contém informações dos clientes que realizaram pedidos.  

| Nome do campo | Tipo   | Descrição                                |
| ------------- | ------ | ---------------------------------------- |
| customer_id   | STRING | ID natural do cliente                    |
| customer_name | STRING | Nome do cliente                          |
| segment       | STRING | Segmento do cliente                      |
| customer_key  | STRING | Chave substituta do cliente (para joins) |

#### 📗 `DimProduct`  

Armazena os dados dos produtos vendidos.  

| Nome do campo | Tipo   | Descrição                                |
| ------------- | ------ | ---------------------------------------- |
| product_id    | STRING | ID natural do produto                    |
| product_name  | STRING | Nome do produto                          |
| category      | STRING | Categoria do produto                     |
| sub_category  | STRING | Subcategoria do produto                  |
| product_key   | STRING | Chave substituta do produto (para joins) |

#### 📙 `DimDate`  

Utilizada para análises temporais e agregações por períodos.  

| Nome do campo    | Tipo    | Descrição                        |
| ---------------- | ------- | -------------------------------- |
| date_key         | INTEGER | Chave da data (formato YYYYMMDD) |
| date             | DATE    | Data no formato DATE             |
| year             | INTEGER | Ano                              |
| month            | INTEGER | Mês                              |
| weeknum          | INTEGER | Número da semana                 |
| day_of_week_num  | INTEGER | Número do dia da semana (1 a 7)  |
| day_of_week_name | STRING  | Nome do dia da semana            |

#### 📕 `DimRegion`  

Agrupa informações geográficas relacionadas ao local do pedido.  

| Nome do campo | Tipo   | Descrição                  |
| ------------- | ------ | -------------------------- |
| region_id     | STRING | Chave substituta da região |
| region        | STRING | Nome da região             |
| city          | STRING | Cidade                     |
| state         | STRING | Estado                     |
| country       | STRING | País                       |

#### 📘 `DimShipMode`  

Lista os modos de envio disponíveis no sistema.  

| Nome do campo | Tipo   | Descrição                         |
| ------------- | ------ | --------------------------------- |
| ship_mode_id  | STRING | Chave substituta do modo de envio |
| ship_mode     | STRING | Nome do modo de envio             |

#### 📗 `DimMarket`  

Contém informações sobre o mercado ou região de atuação.  

| Nome do campo | Tipo   | Descrição                            |
| ------------- | ------ | ------------------------------------ |
| market_id     | STRING | Chave substituta do mercado          |
| market        | STRING | Nome do mercado principal            |
| market2       | STRING | Nome do segundo mercado (subdivisão) |
