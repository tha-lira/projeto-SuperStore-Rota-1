# 📊 Projeto de Exploração – Estrutura de Dados (ETL)

### Entendimento do Problema

**Contexto**:
A Super Store precisa organizar e estruturar seus dados para facilitar consultas, análises e decisões estratégicas.

**Objetivo**:
Construir um sistema tabular relacional utilizando o modelo dimensional, com tabelas fato e dimensão, alimentado por um processo ETL (Extract, Transform, Load).

## 🟦 2.1 Processar e preparar base de dados

### 🔵  Conexão e Importação dos Dados

- **Projeto ID:** `estrutura-de-dados-473122`  
- **Dataset Principal:** `dataBase`  
- **Tabela Bruta:** `superstore`

### 🔵 Identificar e tratar valores nulos

Foi aplicada a query `SUM(CASE WHEN coluna IS NULL THEN 1 ELSE 0 END)` em todas as colunas da base `superstore`.  

**Resultado:** Nenhuma coluna apresentou valores nulos entre as 51.290 linhas avaliadas.

### 🔵  Identificar e tratar valores duplicados

A análise de duplicidade foi realizada em diferentes níveis:

- 1ª verificação: `order_id + product_id` → encontrados 38 duplicados.
- 2ª verificação (mais refinada):` customer_ID + region + product_name` → total caiu para 33.
- Verificação final: construída chave composta (`customer_ID + order_id + product_id + order_date + ship_date`) para confirmar duplicados reais.

**Resultado final:**

| Métrica         | Valor   |
|-----------------|---------|
| Total de linhas | 51.290  |
| Linhas únicas   | 51.255  |
| Duplicados      | 35      |

📌 **Interpretação**: Os 35 registros duplicados eram idênticos em todas as colunas-chave, indicando redundância sem informação nova. Se não tratados, poderiam distorcer métricas como vendas, lucro e quantidade.

✅ **Ação Tomada**: Foi criada a tabela intermediária `superstore_cleaned` com as duplicatas removidas, garantindo a integridade das futuras tabelas fato e dimensão.

### 🔵 Identificar e tratar dados discrepantes 

Também foi realizada uma análise de possíveis inconsistências em variáveis numéricas e categóricas.

- **Padronização aplicada**: funções LOWER() e TRIM() em variáveis de texto para evitar discrepâncias de maiúsculas/minúsculas ou espaços extras.

- **FactSales**: 51.255 registros válidos, sem valores nulos nas métricas principais.

- **Profit (Lucro**): 12.541 registros (24,5%) com valores negativos, possivelmente indicando devoluções, descontos excessivos ou ajustes financeiros.

- Demais dimensões (DimCustomer, DimProduct, DimDate, DimRegion, DimShipMode e DimMarket): nenhuma inconsistência relevante foi identificada.

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

✅ **Considerações Finais**

Esse padrão sugere forte impacto dos descontos elevados na rentabilidade. Recomenda-se revisar a política de descontos e investigar se esses valores decorrem de promoções planejadas ou de falhas nos processos de precificação.

### 🔷 Tabela Intermediária Criada

Tabela Intermediária: `superstore_cleaned`

**Variáveis disponíveis:**  
`customer_ID, customer_name, segment, product_id, product_name, category, sub_category, order_id, row_id, order_date, ship_date, sales, profit, quantity, discount, shipping_cost, order_priority, ship_mode, region, city, state, country, market, market2, year, weeknum, unknown, rn`

### 🔵 Pesquisar dados de outras fontes

Para enriquecer a análise, foram integrados dados de concorrentes multinacionais de supermercados.

📌 **Coleta dos Dados**
A extração foi realizada por meio de web scraping com a função IMPORTHTML do Google Planilhas, aplicada à página da Wikipedia:

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

Foi adotado o modelo dimensional (star schema), no qual:

**Tabelas de Dimensão** → armazenam atributos descritivos (quem, o quê, onde, quando).

**Tabela Fato** → registra eventos transacionais e métricas (vendas, lucro, quantidade).

📌 **Padronização**

- Uso de LOWER() e TRIM() para evitar inconsistências textuais.

- Deduplicação das dimensões.

- Criação de surrogate keys com FARM_FINGERPRINT, mantendo IDs naturais para rastreabilidade.

📌 **Modelagem Visual**
O modelo foi representado no Lucidchart, destacando a FactSales no centro, conectada às dimensões DimCustomer, DimProduct, DimDate, DimRegion, DimShipMode e DimMarket.

Essa estrutura facilita análises sob diferentes perspectivas (clientes, produtos, regiões, datas e envios).

👉 [Visualizar o diagrama](https://github.com/tha-lira/projeto-SuperStore-Rota-1/blob/main/Modelo%20Dimensional%20-%20An%C3%A1lise%20de%20Vendas%20(Star%20Schema).pdf)

### 🔵 Criar estrutura de base de dados (tabelas de fatos e dimensões)

As tabelas foram implementadas no BigQuery:

- **FactSales** → núcleo do modelo, contendo métricas de vendas.

- **DimCustomer, DimProduct, DimDate, DimRegion, DimShipMode, DimMarket** → fornecem contexto descritivo.

📌 **Boas Práticas Aplicadas**

- Modelagem estrela → simplicidade e eficiência em consultas OLAP.

- Chaves primárias das dimensões como chaves estrangeiras na fato.

- Surrogate keys via FARM_FINGERPRINT para unicidade e joins eficientes.

- FactSales contém apenas dados normalizados e referenciados.

👉 [Estrutura detalhada do modelo dimensional](https://github.com/tha-lira/projeto-SuperStore-Rota-1/blob/main/Estrutura_Modelo_Dimesional.md)
  
### 🔵  Agendar atualizações de tabelas

Para garantir que a base se mantenha atualizada, foi projetado um pipeline lógico de atualização, considerando dependências entre tabelas:

Superstore (tabela bruta) → dados de entrada.

superstore_cleaned (intermediária) → remoção de duplicatas e padronização.

Dimensões (DimCustomer, DimProduct, DimDate, DimRegion, DimShipMode, DimMarket) → devem ser atualizadas antes da fato.

FactSales → última a ser atualizada, consolidando métricas e conectando dimensões.

⚠️ Importante: neste projeto, o pipeline foi apenas projetado conceitualmente, sem automação em ferramentas externas.

### 🔵 Conclusão Final e Próximos Passos

Ao longo deste projeto, foi desenvolvido um processo completo de ETL (Extract, Transform, Load) aplicado ao dataset da Super Store, com foco na construção de uma estrutura dimensional (Star Schema) para análise eficiente de dados.

📌 **Principais Entregas**

**Qualidade dos Dados**:

- Verificação completa de nulos → nenhum valor ausente encontrado.

- Identificação e tratamento de duplicados → 35 registros redundantes removidos.

- Tratamento de inconsistências → padronização de variáveis categóricas e análise de métricas discrepantes (ex.: lucro negativo).

**Tabela Intermediária (superstore_cleaned)**:

- Base consolidada e limpa, utilizada como fonte confiável para construção do modelo dimensional.

**Pesquisa de Outras Fontes**:

- Integração de dados externos sobre redes de supermercados multinacionais via IMPORTHTML.

- Tratamento de inconsistências e seleção de variáveis relevantes para benchmarking internacional.

**Modelagem Dimensional**:

- Estrutura em estrela construída no BigQuery, com a FactSales no centro conectada às dimensões DimCustomer, DimProduct, DimDate, DimRegion, DimShipMode e DimMarket.

- Aplicação de boas práticas: surrogate keys, padronização textual e deduplicação de dimensões.

- Diagrama visual criado no Lucidchart, facilitando a compreensão do modelo.

**Pipeline de Atualização (conceitual)**:

- Definição da ordem lógica de atualização das tabelas (da bruta até a fato).

📊 **Benefícios da Solução**

- Garantia de integridade e consistência dos dados.

- Estrutura escalável e otimizada para consultas analíticas (OLAP).

- Possibilidade de análises sob diferentes perspectivas (clientes, produtos, regiões, períodos, modos de envio).

- Base integrada para benchmarking internacional, permitindo comparação com grandes redes multinacionais.
