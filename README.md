# 📊 Dashboard Comercial

## 🎯 Propósito

Este projeto tem como objetivo a construção de um dashboard de Business Intelligence (BI) para uma empresa com atividade comercial. A solução foi desenvolvida no **Power BI**, utilizando os seguintes pilares:

- **Modelagem dimensional** com a aplicação do modelo **Snowflake**;
- **Tratamento de dados** por meio do **Power Query**;
- **Construção de indicadores-chave** de desempenho no varejo, utilizando **medidas DAX**.

---

## 🧩 Modelo Dimensional - Snowflake

Abaixo, o diagrama que representa a modelagem dimensional em formato **Snowflake**, adotada para organizar as informações de forma eficiente e escalável:

![Modelo Snowflake](./imagens_doc/Snowflake.png)

---

## 📐 Principais Medidas DAX

A seguir, estão descritas as principais medidas DAX utilizadas no dashboard, agrupadas conforme sua finalidade de análise de desempenho.

### 🔍 Indicadores de Performance

| Indicador             | Fórmula DAX                                                                 |
|-----------------------|-----------------------------------------------------------------------------|
| **Receita Bruta**     | `SUM(fato_vendas[Valor Total])`                                             |
| **Tributos**          | `SUMX(fato_vendas, fato_vendas[Qtde] * RELATED(dim_produto[Tributos]))`     |
| **Receita Líquida**   | `[Receita Bruta] - [Tributos]`                                              |
| **Faturamento Bruto** | `SUMX(fato_vendas, fato_vendas[Qtde] * RELATED(dim_produto[Preco_Unitario]))`|
| **Custo**             | `SUMX(fato_vendas, fato_vendas[Qtde] * RELATED(dim_produto[Custo]))`        |
| **Margem**            | `[Receita Líquida] - [Custo]`                                               |
| **% Margem**          | `DIVIDE([Margem],[Receita Liquida],0)`                                      |
| **Desconto**          | `[Faturamento Bruto] - [Receita bruta]`                                     |

### 🔧 Medidas Dinâmicas no Power BI (DAX)
1. Medida Selecionada  
``` Medida Selecionada = SELECTEDVALUE(aux_Medidas[Medida]) ```  
Descrição:
Captura o valor selecionado pelo usuário no slicer da tabela aux_Medidas. Serve como base para definir qual métrica será analisada nas medidas dinâmicas seguintes.

2. Medida Dinâmica - Valor
```
Medida Dinamica - Valor = VAR MedidaSelecionada = SELECTEDVALUE(aux_Medidas[Medida])
RETURN
SWITCH(
    TRUE(),
    MedidaSelecionada = "Quantidade", [Quantidade],
    MedidaSelecionada = "Faturamento Bruto", [Faturamento Bruto],
    MedidaSelecionada = "Receita Bruta", [Receita bruta],
    MedidaSelecionada = "Desconto", [Desconto],
    MedidaSelecionada = "Tributos", [Tributos],
    MedidaSelecionada = "Receita Liquida", [Receita Liquida],
    MedidaSelecionada = "Custo", [Custos],
    MedidaSelecionada = "Margem", [Margem],
    BLANK()
)
```
Descrição:
Retorna a medida correspondente ao valor selecionado. Ex: se o usuário escolher "Margem", retorna [Margem].

3. Medida Dinâmica - Valor LY (Last Year)
```
Medida Dinamica - Valor LY = 
VAR MedidaSelecionada = SELECTEDVALUE(aux_Medidas[Medida])
RETURN
SWITCH(
    TRUE(),
    MedidaSelecionada = "Quantidade", [Quantidade LY],
    MedidaSelecionada = "Faturamento Bruto", [Faturamento Bruto LY],
    MedidaSelecionada = "Receita Bruta", [Receita bruta LY],
    MedidaSelecionada = "Desconto", [Desconto LY],
    MedidaSelecionada = "Tributos", [Tributo LY],
    MedidaSelecionada = "Receita Liquida", [Receita Liquida LY],
    MedidaSelecionada = "Custo", [Custos LY],
    MedidaSelecionada = "Margem", [Margem LY],
    BLANK()
)
```
Descrição:
Mesmo conceito da medida anterior, mas retorna o valor correspondente do ano anterior, possibilitando comparações YoY.

4. Medida Dinâmica - YoY (Year over Year)
```
Medida Dinamica - YoY = 
DIVIDE(
    [Medida Dinamica - Valor] - [Medida Dinamica - Valor LY],
    [Medida Dinamica - Valor LY],
    0
)
```
Descrição:
Calcula a variação percentual ano contra ano da medida selecionada:

Fórmula: (Atual - LY) / LY

Usa DIVIDE para evitar erros de divisão por zero.

5. Medida Dinâmica - Share (Participação)
```
Medida Dinamica - Share = 
VAR TotalProduto = CALCULATE([Medida Dinamica - Valor], ALLSELECTED(dim_produto[Descricao_Produto]))
VAR TotalSubcategoria = CALCULATE([Medida Dinamica - Valor], ALLSELECTED(dim_subcategoria[Subcategoria]))
VAR TotalCategoria = CALCULATE([Medida Dinamica - Valor], ALLSELECTED(dim_categoria[Categoria]))
RETURN
SWITCH(
    TRUE(),
    ISINSCOPE(dim_produto[Descricao_Produto]), DIVIDE([Medida Dinamica - Valor], TotalProduto, 0),
    ISINSCOPE(dim_subcategoria[Subcategoria]), DIVIDE([Medida Dinamica - Valor], TotalSubcategoria, 0),
    ISINSCOPE(dim_categoria[Categoria]), DIVIDE([Medida Dinamica - Valor], TotalCategoria, 0)
)
```
Descrição:
Calcula a participação (%) da medida atual em relação ao total no nível hierárquico selecionado (produto, subcategoria ou categoria).


## Dashboard


