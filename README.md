# Curva ABC de Faturamento — E-commerce Olist

Análise de concentração de faturamento por categoria de produto, aplicando a Curva ABC (regra de Pareto 80/20) sobre dados reais de um e-commerce brasileiro, com o cálculo analítico feito **100% em SQL** (CTEs + window functions).

## O problema de negócio

Empresas de varejo/e-commerce costumam sofrer com dois problemas opostos no estoque: capital parado em produtos de baixo giro, e ruptura em produtos essenciais que geram a maior parte do faturamento. A Curva ABC resolve isso classificando produtos (ou, neste caso, categorias) em três grupos:

- **Classe A**: poucas categorias, concentrando a maior parte do faturamento — exigem controle rígido de estoque
- **Classe B**: impacto intermediário
- **Classe C**: cauda longa — muitas categorias, baixo impacto individual no faturamento

## Fonte de dados

[Olist Brazilian E-Commerce Dataset](https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce) (Kaggle) — dados reais de pedidos de um marketplace brasileiro entre 2016 e 2018.

Arquivos utilizados: `orders`, `order_items`, `products`, `product_category_name_translation`.

## Decisões de tratamento de dados (ETL)

O dataset bruto exigiu decisões de negócio antes de qualquer cálculo, documentadas aqui por transparência:

| Decisão | Justificativa |
|---|---|
| Considerar apenas pedidos com `order_status = 'delivered'` | Pedidos registrados não são necessariamente vendas concluídas (cancelamento, indisponibilidade, etc.) |
| Quantidade vendida = contagem de linhas em `order_items` | O dataset não possui uma coluna de quantidade — cada unidade vendida é uma linha própria |
| Análise por **categoria** de produto (não por produto individual) | O catálogo tem ~33 mil produtos individuais sem nome legível; agrupar por categoria (73 valores) torna a análise legível para stakeholders não técnicos |
| Categorias mantidas em português | Público-alvo do relatório é o mercado brasileiro |
| Produtos sem categoria informada descartados (~1,8% da base) | Escolha por simplicidade; impacto mínimo no resultado |

## Pipeline técnico

1. **Extract**: leitura dos CSVs brutos com Pandas
2. **Transform**: filtros e merges (`orders` + `order_items` + `products`), agregação por categoria (faturamento e quantidade)
3. **Load**: carga do resumo agregado em SQLite (`banco_olist.db`)
4. **Análise (SQL puro)**: a classificação ABC é calculada inteiramente em SQL, usando:
   - **CTEs encadeadas** (`WITH`) para organizar o cálculo em etapas legíveis
   - **Window functions** (`SUM() OVER (ORDER BY ...)` e `SUM() OVER ()`) para o faturamento acumulado e o total geral, sem colapsar as linhas
   - **CASE WHEN** para a classificação final em A/B/C

A decisão de fazer a classificação em SQL (em vez de delegar ao Pandas) foi deliberada: evidencia domínio de SQL analítico, habilidade central para a vaga de analista de dados.

## Resultado

De 73 categorias analisadas:

| Classe | Categorias | % do total de categorias |
|---|---|---|
| A | 16 | ~22% |
| B | 16 | ~22% |
| C | 41 | ~56% |

**Categoria líder**: `beleza_saude`, com R$ 1,23 milhão em faturamento — a maior categoria isolada do catálogo.

## Insight de negócio

A distribuição real foge do padrão clássico 80/20: a Classe A tem 16 categorias (22% do catálogo), não a minoria extrema (~10-20%) que a regra de Pareto costuma sugerir. Isso indica um catálogo relativamente diversificado, sem dependência excessiva de poucas categorias — um sinal operacional positivo, já que reduz o risco de ruptura concentrada em poucos itens.

## Limitações conhecidas

- Reembolsos e devoluções pós-entrega não são capturados de forma confiável neste dataset, então o faturamento "real" pode estar levemente superestimado
- A análise usa o período completo disponível no dataset; não há segmentação temporal (ex: sazonalidade, comparação ano a ano)

## Stack técnica

- Python (Pandas)
- SQLite
- SQL (CTEs, window functions, CASE WHEN)
- Matplotlib

## Estrutura do repositório

```
data/raw/            # CSVs originais do Kaggle
notebooks/           # Notebook com o pipeline completo (ETL + análise + gráfico)
banco_olist.db       # Banco gerado pelo pipeline
requirements.txt
```

## Como executar

```bash
pip install -r requirements.txt
jupyter notebook notebooks/01_etl_curva_abc.ipynb
```
