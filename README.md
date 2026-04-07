# 🚀 Rocket Lab - Data Engineering com Arquitetura Medallion

Um projeto completo de **Data Engineering** utilizando a arquitetura **Medallion (Bronze → Silver → Gold)** em **Databricks**, com dados de e-commerce brasileiro. O projeto demonstra boas práticas em ingestão de dados, transformação, limpeza e análise.

---

## 📋 Índice

- [Visão Geral](#visão-geral)
- [Arquitetura do Projeto](#arquitetura-do-projeto)
- [Estrutura de Camadas](#estrutura-de-camadas)
- [Notebooks do Projeto](#notebooks-do-projeto)
- [Como Executar](#como-executar)
- [Estrutura de Dados](#estrutura-de-dados)
- [Boas Práticas Implementadas](#boas-práticas-implementadas)
- [Status do Projeto](#status-do-projeto)

---

## 🎯 Visão Geral

Este projeto implementa um pipeline de dados end-to-end que:

- ✅ **Ingere** dados brutos de e-commerce de uma área de landing
- ✅ **Transforma** dados na camada Bronze (padronização)
- ✅ **Enriquece** dados na camada Silver (limpeza e validação)
- ✅ **Analisa** dados na camada Gold (business insights)
- ✅ **Automatiza** execução via agendamento diário

**Tecnologia:** Databricks | **Formato:** Delta Lake | **Linguagem:** PySpark (Python)

---

## 🏗️ Arquitetura do Projeto

```
┌─────────────────────────────────────────────────────────────┐
│                    LANDING (Raw Data)                       │
│                   CSV Files armazenados                     │
└────────────────────┬────────────────────────────────────────┘
                     │
        ┌────────────▼────────────┐
        │   01 - BRONZE LAYER     │
        │   (Padronização básica) │
        │   10 tabelas delta      │
        └────────────┬────────────┘
                     │
        ┌────────────▼────────────┐
        │   02 - SILVER LAYER     │
        │   (Transformação)       │
        │   Limpeza & Validação   │
        │   10+ tabelas delta     │
        └────────────┬────────────┘
                     │
        ┌────────────▼────────────┐
        │   03 - GOLD LAYER       │
        │   (Business Insights)   │
        │   Análises & Views      │
        └─────────────────────────┘
```

---

## 📦 Estrutura de Camadas

### 🥉 **BRONZE** - Ingestão e Padronização
- **Objetivo:** Armazenar dados brutos com formato padronizado
- **Formato:** Delta Lake com timestamp_ingestion
- **Operação:** Append incremental (novos dados adicionados)
- **Tabelas:**
  - `tb_customers` - Informações dos clientes
  - `tb_orders` - Pedidos realizados
  - `tb_order_items` - Itens dos pedidos
  - `tb_order_payments` - Pagamentos dos pedidos
  - `tb_order_reviews` - Avaliações dos clientes
  - `tb_products` - Catálogo de produtos
  - `tb_sellers` - Informações dos vendedores
  - `tb_geolocalizacao` - Dados geográficos
  - `tb_product_category_name_translation` - Tradução de categorias
  - `tb_cotacao_dolar` - Cotações diárias do dólar

### 🥈 **SILVER** - Limpeza e Transformação
- **Objetivo:** Garantir qualidade dos dados através de validações
- **Formato:** Delta Lake com coluna de qualidade (flag_qualidade)
- **Operação:** Overwrite com schema evolution
- **Estratégias:**
  - Remoção de duplicatas com Window Functions
  - Tratamento de valores nulos com coalesce
  - Conversão de tipos de dados
  - Validações de negócio com flags
  - Separação de dados válidos e inválidos (auditoria)

- **Tabelas Principais (válidos):**
  - `dim_consumidores_validos` - Dimensão de clientes
  - `fat_pedidos_validos` - Fato de pedidos com cálculos de prazos
  - `fat_itens_pedidos_validos` - Fato de itens com preço e frete
  - `fat_pagamentos_pedidos_validos` - Fato de pagamentos
  - `fat_avaliacoes_pedidos_validos` - Avaliações dos pedidos
  - `dim_produtos_validos` - Dimensão de produtos
  - `dim_vendedores_validos` - Dimensão de vendedores
  - `dim_categoria_produtos_traducao_validos` - Tradução de categorias
  - `dim_cotacao_dolar_validos` - Cotações preenchidas (forward fill)
  - `fat_pedido_total` - Fato consolidada com valores em BRL e USD

- **Otimização:** OPTIMIZE com ZORDER BY em colunas principais

### 🥇 **GOLD** - Análises e Insights
- **Objetivo:** Fornecer visões de negócio prontas para análise
- **Formato:** Delta Lake com mode overwrite e schema evolution
- **Operação:** Acesso via views e tabelas agregadas

- **Tabelas Analíticas:**
  - `fat_vendas_comercial` - Vendas por período, categoria e produto
    - Métricas: Total de pedidos, QTD itens, receita BRL/USD, ticket médio
  
  - `fat_avaliacoes_clientes` - Satisfação por produto/vendedor
    - Métricas: Avaliação média, satisfação positiva/negativa, percentual satisfação
    - Rankings: Produto mais/menos avaliado, Vendedor top/bottom

---

## 📓 Notebooks do Projeto

| # | Nome | Descrição | Duração Est. |
|---|------|-----------|--------------|
| 00 | **Atividades.ipynb** | Checklist e status do projeto | - |
| 01 | **Atividade_preparando_ambiente.ipynb** | Setup inicial - Catalog, Schemas, Volumes | 1-2 min |
| 02 | **Atividade_land_to_bronze.ipynb** | Ingestão de CSVs da landing → Bronze | 5-10 min |
| 03 | **Atividade_bronze_to_silver.ipynb** | Transformação e validação → Silver | 15-20 min |
| 04 | **Atividade_silver_to_gold.ipynb** | Análises e insights → Gold | 5-10 min |

### Fluxo de Execução

```
Atividade_preparando_ambiente
        ↓
Atividade_land_to_bronze
        ↓
Atividade_bronze_to_silver
        ↓
Atividade_silver_to_gold
```

---

## 🚀 Como Executar

### Pré-requisitos
- ✅ Acesso a um workspace Databricks
- ✅ Arquivos CSV na pasta Landing
- ✅ Permissões para criar catalogs e schemas

### Passo a Passo

1. **Preparar o Ambiente**
   ```
  Execute o notebook: Atividade_preparando_ambiente.ipynb
   - Cria catálogo 'medalhao_at'
   - Cria schemas: bronze, silver, gold
   - Cria volume 'landing'
   ```

2. **Ingerir Dados**
   ```
  Execute o notebook: Atividade_land_to_bronze.ipynb
   - Lê CSVs da landing
   - Armazena em tabelas Delta (Bronze)
   - Adiciona timestamp_ingestion
   ```

3. **Transformar e Limpar**
   ```
  Execute o notebook: Atividade_bronze_to_silver.ipynb
   - Remove duplicatas com Window Functions
   - Valida regras de negócio
   - Preenche valores nulos com estratégias específicas
   - Otimiza com ZORDER BY
   ```

4. **Gerar Insights**
   ```
  Execute o notebook: Atividade_silver_to_gold.ipynb
   - Cria tabelas analíticas
   - Calcula métricas de negócio
   - Gera rankings e comparações
   ```

### Agendamento
O pipeline está configurado para executar **diariamente às 13h** com 4 tasks em sequência:
1. Atividade_preparando_ambiente
2. Atividade_land_to_bronze
3. Atividade_bronze_to_silver
4. Atividade_silver_to_gold

---

## 📊 Estrutura de Dados

### Exemplo: Fluxo do Pedido

```
Landing (CSV)
    ↓
Bronze (tb_orders, tb_order_items, tb_order_payments)
    ↓
Silver (fat_pedidos_validos, fat_itens_pedidos_validos, 
        fat_pagamentos_pedidos_validos, dim_cotacao_dolar_validos)
    ↓
Gold (fat_pedido_total com valores em BRL e USD)
    ↓
Analysis (fat_vendas_comercial, fat_avaliacoes_clientes)
```

### Principais Cálculos Silver

#### fat_pedidos_validos
- `tempo_entrega_dias` = data_entrega_real - data_compra
- `tempo_entrega_estimado_dias` = data_entrega_estimada - data_compra
- `diferenca_entrega_dias` = data_entrega_real - data_entrega_estimada
- `entrega_no_prazo` = "Sim" se diferença ≤ 0 | "Não" se > 0

#### dim_cotacao_dolar_validos
- **Forward Fill:** Preenche finais de semana com cotação anterior (API não publica nos finais de semana)
- Calendário completo criado com `F.sequence()` e `F.explode()`

#### fat_pedido_total
- `valor_total_pago_usd` = valor_pagamento_brl / cotacao_do_dia
- Join com tabela de cotação usando data do pedido

### Principais Cálculos Gold

#### fat_vendas_comercial
```sql
Agregação: ano_venda, mes_venda, categoria_produto
- total_pedidos = COUNT(DISTINCT id_pedido)
- qtd_itens_vendidos = COUNT(id_item_pedido)
- receita_total_brl = SUM(preco)
- receita_total_usd = Proporcional ao valor em USD
- ticket_medio_brl = receita_total_brl / total_pedidos
```

#### fat_avaliacoes_clientes
```sql
Agregação: categoria_produto, nome_vendedor, estado_vendedor
- total_avaliacoes = COUNT(id_avaliacao)
- avaliacao_media = AVG(nota_avaliacao)
- total_avaliacoes_positivas = COUNT(nota >= 4)
- total_avaliacoes_negativas = COUNT(nota <= 2)
- percentual_satisfacao = (positivas / total) * 100
```

---

## ✨ Boas Práticas Implementadas

### 1. **Qualidade de Dados**
- ✅ Flags de qualidade em cada tabela
- ✅ Separação de dados válidos e inválidos (auditoria)
- ✅ Validações de negócio específicas por tabela

### 2. **Performance**
- ✅ OPTIMIZE com ZORDER BY em tabelas fato
- ✅ Índices nas colunas de join principais
- ✅ Window Functions para remoção eficiente de duplicatas

### 3. **Manutenibilidade**
- ✅ Nomes de colunas em português (padrão do negócio)
- ✅ Comentários explicativos em células de código
- ✅ Funções reutilizáveis para padrões comuns

### 4. **Transformação de Dados**
- ✅ Imputação inteligente de nulos (coalesce com lógica de negócio)
- ✅ Padronização: UPPER, INITCAP, TRIM
- ✅ Conversão de tipos explícita com .cast()
- ✅ Tradução de valores (status, tipos de pagamento)

### 5. **Evolutividade**
- ✅ `.option("overwriteSchema", "true")` na Gold para novas colunas
- ✅ Ingestão incremental na Bronze com append
- ✅ Estrutura preparada para novos tapes de análise

---

## 📈 Status do Projeto

### ✅ Completo
- [x] Preparação do ambiente (Atividade_preparando_ambiente)
- [x] Ingestão Bronze (Atividade_land_to_bronze)
- [x] Transformação Silver (Atividade_bronze_to_silver)
- [x] Análises Gold (Atividade_silver_to_gold)
- [x] Otimização e performance
- [x] Agendamento diário com dependências
- [x] Tratamento de erros e auditoria
- [x] Notebooks em formato .ipynb com comentários
- [x] Arquivo .yaml do Job exportado do Databricks
- [x] Screenshot da execução bem-sucedida do Job
- [x] Publicação em repositório GitHub público

---

## 📞 Informações Adicionais

### Catálogo e Schemas
```
Catálogo: medalhao_at
├── bronze (Dados Brutos)
├── silver (Dados Transformados)
└── gold (Dados de Negócio)
```

### Volumes
```
Landing: /Volumes/medalhao_at/default/landing/
Armazena os CSVs brutos do e-commerce
```

### Frequência de Atualização
- **Agendamento:** Diariamente às 13h (UTC-3)
- **Modo:** Append na Bronze, Overwrite na Silver e Gold
- **Retenção:** Total (histórico completo)

---

**Criado com ❤️ para Data Engineering**