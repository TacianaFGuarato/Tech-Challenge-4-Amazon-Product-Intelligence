# Medidas DAX — Tech Challenge 4 | Amazon Product Intelligence

Este documento apresenta as **medidas DAX e a coluna calculada** utilizadas nas cinco páginas do dashboard, com regras de negócio, resultados de referência e cuidados de interpretação.

> **Escopo:** documentação da implementação de Power BI desenvolvida durante o projeto. As expressões abaixo são fórmulas de referência; confira os nomes do modelo dentro do seu arquivo `.pbix` e mantenha nele a versão final efetivamente usada.

## Modelo e preparação

- **Tabela de produtos:** `amazon_powerbi` — uma linha por `product_id` (1.351 produtos).
- **Tabela de expressões:** `amazon_temas` — frequências agregadas por `tipo_texto` e `expressao` (20 linhas); não possui granularidade de produto.
- **Fonte:** `data/processed/amazon_powerbi_corrigido.csv` e `data/processed/amazon_temas.csv`.
- **Importação dos CSVs:** delimitador **ponto e vírgula** (`;`); campos decimais com **vírgula**; usar **Alterar tipo usando localidade → Português (Brasil)** quando necessário.
- **Tipos:** `rating`, `rating_count`, `discounted_price`, `actual_price`, `economia_valor`, `sentiment_score` numéricos; `discount_percentage` numérico 0–100; `product_id` e `cluster` (se preferir) como identificadores, e categorias como texto.
- **Nome da tabela:** se no seu Power BI ela estiver como `amazon_powerbi_corrigido`, substitua `amazon_powerbi` nas fórmulas.
- **Fórmulas:** criar individualmente por **Modelagem → Nova medida**; a classificação por **Modelagem → Nova coluna**. Um arquivo `.dax` não é importado automaticamente no Power BI Desktop.
- **Filtros:** resultados esperados são calculados **sem segmentadores nem filtros cruzados**; valores devem mudar quando o usuário interagir com o relatório.

## Critérios analíticos

**Produto de atenção:** `rating < 4,0` e `rating_count >= 16.020`. **Candidato a oportunidade:** `rating >= 4,3` e `rating_count <= 1.106`. Os limites foram estabelecidos durante a EDA a partir da distribuição de avaliações; são regras exploratórias, não modelos preditivos.

**Importante:** `rating_count` mede a **quantidade acumulada de avaliações**, usada como proxy de engajamento. Não mede vendas, audiência nem visibilidade diretamente. A classificação VADER foi aplicada ao **texto agregado de comentários por produto**, não a avaliações individuais.

## Coluna calculada — Página 5

Criar em `amazon_powerbi` via **Nova coluna**:

```dax
Classificação Estratégica =
SWITCH(
    TRUE(),
    ISBLANK(amazon_powerbi[rating]) ||
        ISBLANK(amazon_powerbi[rating_count]),
        "Dados insuficientes",
    amazon_powerbi[rating] < 4 &&
        amazon_powerbi[rating_count] >= 16020,
        "Atenção",
    amazon_powerbi[rating] >= 43 / 10 &&
        amazon_powerbi[rating_count] <= 1106,
        "Oportunidade",
    "Monitoramento"
)
```

**Conferência sem filtros:** Atenção = **49**, Oportunidade = **81**, Dados insuficientes = **3**, Monitoramento = **1.218**. Total = **1.351**.

A coluna permite segmentar a tabela de produtos. Ela é uma classificação estática por registro: selecionar outras categorias ou faixas apenas filtra os registros existentes.
## Página 1 — Visão Executiva

### Total Produtos
**Finalidade:** Quantidade de IDs de produto únicos no contexto de filtros.
**Referência sem filtros:** 1.351.

```dax
Total Produtos =
DISTINCTCOUNT(amazon_powerbi[product_id])
```

### Avaliação Média
**Finalidade:** Nota média dos produtos com avaliação disponível.
**Referência sem filtros:** 4,09.

```dax
Avaliação Média =
AVERAGE(amazon_powerbi[rating])
```

### Desconto Médio (%)
**Finalidade:** Converte o desconto médio de uma escala 0–100 para percentual; formatar como porcentagem.
**Referência sem filtros:** 46,69%.

```dax
Desconto Médio (%) =
DIVIDE(AVERAGE(amazon_powerbi[discount_percentage]), 100)
```

### Preço Promocional Médio
**Finalidade:** Média dos preços promocionais; formatar em rúpias indianas.
**Referência sem filtros:** ₹ 3.304,65.

```dax
Preço Promocional Médio =
AVERAGE(amazon_powerbi[discounted_price])
```

### Produtos de Atenção
**Finalidade:** Critério operacional definido na análise: nota < 4 e pelo menos 16.020 avaliações.
**Referência sem filtros:** 49.

```dax
Produtos de Atenção =
COALESCE(
    COUNTROWS(
        FILTER(
            amazon_powerbi,
            NOT ISBLANK(amazon_powerbi[rating]) &&
            NOT ISBLANK(amazon_powerbi[rating_count]) &&
            amazon_powerbi[rating] < 4 &&
            amazon_powerbi[rating_count] >= 16020
        )
    ),
    0
)
```

### Produtos com Oportunidade
**Finalidade:** Critério operacional: nota ≥ 4,3 e no máximo 1.106 avaliações; não comprova baixa visibilidade.
**Referência sem filtros:** 81.

```dax
Produtos com Oportunidade =
COALESCE(
    COUNTROWS(
        FILTER(
            amazon_powerbi,
            NOT ISBLANK(amazon_powerbi[rating]) &&
            NOT ISBLANK(amazon_powerbi[rating_count]) &&
            amazon_powerbi[rating] >= 43 / 10 &&
            amazon_powerbi[rating_count] <= 1106
        )
    ),
    0
)
```

### Total Categorias
**Finalidade:** Categorias principais distintas.
**Referência sem filtros:** 9.

```dax
Total Categorias =
DISTINCTCOUNTNOBLANK(amazon_powerbi[categoria_principal])
```

### Mediana de Avaliações
**Finalidade:** Mediana da quantidade de avaliações por produto; não é o total de avaliações.
**Referência sem filtros:** 4.740.

```dax
Mediana de Avaliações =
MEDIAN(amazon_powerbi[rating_count])
```


## Página 2 — Produtos e Categorias

### Produtos Classificados
**Finalidade:** Produtos com cluster atribuído pelo K-Means.
**Referência sem filtros:** 1.348.

```dax
Produtos Classificados =
COALESCE(
    COUNTROWS(
        FILTER(amazon_powerbi, NOT ISBLANK(amazon_powerbi[cluster]))
    ),
    0
)
```

### Total Clusters
**Finalidade:** Quantidade de grupos atribuídos, excluindo nulos.
**Referência sem filtros:** 5.

```dax
Total Clusters =
DISTINCTCOUNTNOBLANK(amazon_powerbi[cluster])
```

### Cobertura K-Means (%)
**Finalidade:** Porcentagem de produtos com classificação; formatar como porcentagem.
**Referência sem filtros:** 99,78%.

```dax
Cobertura K-Means (%) =
DIVIDE([Produtos Classificados], [Total Produtos], 0)
```

### Produtos Não Classificados
**Finalidade:** Registros excluídos do agrupamento por valores ausentes nas variáveis do modelo.
**Referência sem filtros:** 3.

```dax
Produtos Não Classificados =
[Total Produtos] - [Produtos Classificados]
```

### Avaliações do Produto
**Finalidade:** Usar somente com product_id no contexto; isoladamente indica o máximo do catálogo (426.973), NÃO a soma de avaliações.
**Referência sem filtros:** Variável por produto.

```dax
Avaliações do Produto =
MAX(amazon_powerbi[rating_count])
```

### % Produtos por Cluster
**Finalidade:** Participação de cada perfil na seleção atual; sem filtro de cluster, o resultado 100% é esperado.
**Referência sem filtros:** 26,80% para o maior cluster.

```dax
% Produtos por Cluster =
DIVIDE(
    [Total Produtos],
    CALCULATE(
        [Total Produtos],
        REMOVEFILTERS(
            amazon_powerbi[perfil_cluster],
            amazon_powerbi[cluster]
        )
    ),
    0
)
```

### % Cluster na Categoria
**Finalidade:** Mesma lógica de participação acima, usada na matriz com categoria nas linhas e perfil nas colunas.
**Referência sem filtros:** 40,00% para alto engajamento em Computers & Accessories.

```dax
% Cluster na Categoria =
DIVIDE(
    [Total Produtos],
    CALCULATE(
        [Total Produtos],
        REMOVEFILTERS(
            amazon_powerbi[perfil_cluster],
            amazon_powerbi[cluster]
        )
    ),
    0
)
```

### Produtos Alto Engajamento
**Finalidade:** Contagem do perfil de alto engajamento e desconto elevado.
**Referência sem filtros:** 362.

```dax
Produtos Alto Engajamento =
COALESCE(
    CALCULATE(
        [Total Produtos],
        KEEPFILTERS(
            amazon_powerbi[perfil_cluster] = "Alto engajamento e desconto elevado"
        )
    ),
    0
)
```

### Produtos Menor Satisfação
**Finalidade:** Contagem do cluster de menor satisfação; não se confunde com a regra dos 49 produtos de atenção.
**Referência sem filtros:** 182.

```dax
Produtos Menor Satisfação =
COALESCE(
    CALCULATE(
        [Total Produtos],
        KEEPFILTERS(amazon_powerbi[perfil_cluster] = "Menor satisfação")
    ),
    0
)
```


## Página 3 — Preços e Estratégia Comercial

### Preço Original Médio
**Finalidade:** Média dos preços anunciados sem desconto.
**Referência sem filtros:** ₹ 5.691,18.

```dax
Preço Original Médio =
AVERAGE(amazon_powerbi[actual_price])
```

### Economia Média Potencial
**Finalidade:** Média da diferença entre preço original e promocional; não reflete economias realizadas em vendas.
**Referência sem filtros:** ₹ 2.386,52.

```dax
Economia Média Potencial =
AVERAGE(amazon_powerbi[economia_valor])
```

### Produtos com Desconto
**Finalidade:** Produtos com desconto positivo.
**Referência sem filtros:** 1.304.

```dax
Produtos com Desconto =
COALESCE(
    COUNTROWS(
        FILTER(
            amazon_powerbi,
            NOT ISBLANK(amazon_powerbi[discount_percentage]) &&
            amazon_powerbi[discount_percentage] > 0
        )
    ),
    0
)
```

### % Produtos com Desconto
**Finalidade:** Participação dos produtos com desconto positivo; formatar em percentual.
**Referência sem filtros:** 96,52%.

```dax
% Produtos com Desconto =
DIVIDE([Produtos com Desconto], [Total Produtos], 0)
```

### Desconto Máximo (%)
**Finalidade:** Maior percentual de desconto; formatar em percentual.
**Referência sem filtros:** 94%.

```dax
Desconto Máximo (%) =
DIVIDE(MAX(amazon_powerbi[discount_percentage]), 100)
```


## Página 4 — Customer Insights

### Produtos Sentimento Positivo
**Finalidade:** Produtos cujo texto agregado tem sentimento predominantemente positivo.
**Referência sem filtros:** 1.284.

```dax
Produtos Sentimento Positivo =
COALESCE(
    CALCULATE(
        [Total Produtos],
        KEEPFILTERS(amazon_powerbi[sentimento] = "Positivo")
    ),
    0
)
```

### Produtos Sentimento Negativo
**Finalidade:** Produtos cujo texto agregado tem sentimento predominantemente negativo.
**Referência sem filtros:** 61.

```dax
Produtos Sentimento Negativo =
COALESCE(
    CALCULATE(
        [Total Produtos],
        KEEPFILTERS(amazon_powerbi[sentimento] = "Negativo")
    ),
    0
)
```

### Produtos Sentimento Neutro
**Finalidade:** Produtos cujo texto agregado tem sentimento predominantemente neutro.
**Referência sem filtros:** 6.

```dax
Produtos Sentimento Neutro =
COALESCE(
    CALCULATE(
        [Total Produtos],
        KEEPFILTERS(amazon_powerbi[sentimento] = "Neutro")
    ),
    0
)
```

### % Sentimento Positivo
**Finalidade:** Porcentagem de PRODUTOS com textos agregados classificados como positivos; não é % de clientes satisfeitos.
**Referência sem filtros:** 95,04%.

```dax
% Sentimento Positivo =
DIVIDE([Produtos Sentimento Positivo], [Total Produtos], 0)
```

### Score Médio Sentimento
**Finalidade:** Média de polaridade VADER, entre −1 e +1; formatar como número decimal, NÃO porcentagem.
**Referência sem filtros:** 0,84.

```dax
Score Médio Sentimento =
AVERAGE(amazon_powerbi[sentiment_score])
```

### Menções Temas Positivos
**Finalidade:** Usar por expressao; o agregado soma menções a expressões e pode contar o mesmo produto mais de uma vez.
**Referência sem filtros:** 2.208 (soma das expressões selecionadas).

```dax
Menções Temas Positivos =
CALCULATE(
    SUM(amazon_temas[produtos_mencionando]),
    amazon_temas[tipo_texto] = "Positivo"
)
```

### Menções Temas Negativos
**Finalidade:** Usar por expressao; não representa produtos únicos nem o total de comentários negativos.
**Referência sem filtros:** 102 (soma das expressões selecionadas).

```dax
Menções Temas Negativos =
CALCULATE(
    SUM(amazon_temas[produtos_mencionando]),
    amazon_temas[tipo_texto] = "Negativo"
)
```


## Página 5 — Recomendações Estratégicas

### % Produtos de Atenção
**Finalidade:** Proporção de produtos de atenção dentro do contexto atual.
**Referência sem filtros:** 3,63%.

```dax
% Produtos de Atenção =
DIVIDE([Produtos de Atenção], [Total Produtos], 0)
```

### % Produtos com Oportunidade
**Finalidade:** Proporção de candidatos a oportunidade dentro do contexto atual.
**Referência sem filtros:** 6,00%.

```dax
% Produtos com Oportunidade =
DIVIDE([Produtos com Oportunidade], [Total Produtos], 0)
```

### Avaliação Média - Atenção
**Finalidade:** Nota média apenas dos produtos que cumprem a regra de atenção.
**Referência sem filtros:** 3,82.

```dax
Avaliação Média - Atenção =
CALCULATE(
    [Avaliação Média],
    KEEPFILTERS(
        FILTER(
            amazon_powerbi,
            NOT ISBLANK(amazon_powerbi[rating]) &&
            NOT ISBLANK(amazon_powerbi[rating_count]) &&
            amazon_powerbi[rating] < 4 &&
            amazon_powerbi[rating_count] >= 16020
        )
    )
)
```

### Engajamento Mediano - Atenção
**Finalidade:** Mediana da contagem de avaliações dos produtos de atenção; não representa visualizações.
**Referência sem filtros:** 25.824.

```dax
Engajamento Mediano - Atenção =
CALCULATE(
    [Mediana de Avaliações],
    KEEPFILTERS(
        FILTER(
            amazon_powerbi,
            NOT ISBLANK(amazon_powerbi[rating]) &&
            NOT ISBLANK(amazon_powerbi[rating_count]) &&
            amazon_powerbi[rating] < 4 &&
            amazon_powerbi[rating_count] >= 16020
        )
    )
)
```

## Onde utilizar as medidas no dashboard

| Página | Visual recomendado | Campos / medidas |
|---|---|---|
| 1 — Visão Executiva | Cartões | `[Total Produtos]`, `[Avaliação Média]`, `[Desconto Médio (%)]`, `[Preço Promocional Médio]`, `[Produtos de Atenção]`, `[Produtos com Oportunidade]`, `[Total Categorias]`, `[Mediana de Avaliações]` |
| 1 | Barras por categoria | `categoria_principal` + `[Total Produtos]` / `[Avaliação Média]` / medidas de atenção e oportunidade |
| 2 — Produtos e Categorias | Barras por cluster | `perfil_cluster` + `[Total Produtos]` |
| 2 | Matriz composição cluster/categoria | Linhas = `categoria_principal`; Colunas = `perfil_cluster`; Valores = `[% Cluster na Categoria]` |
| 2 | Dispersão engajamento × satisfação | Detalhes/Valores = `product_id`; Eixo X = **Máximo** de `rating_count` (escala logarítmica); Eixo Y = **Média** de `rating`; Legenda = `perfil_cluster` |
| 3 — Preços e Estratégia | Colunas comparativas | Categoria + `[Preço Original Médio]` e `[Preço Promocional Médio]` |
| 3 | Satisfação por desconto | `faixa_desconto` + `[Avaliação Média]` |
| 3 | Dispersão preço × satisfação | `product_id`; X = **Média** `discounted_price` (escala logarítmica); Y = **Média** `rating`; Legenda = `faixa_desconto` |
| 4 — Customer Insights | Rosca / barras 100% empilhadas | `sentimento`, `categoria_principal` + `[Total Produtos]` |
| 4 | Expressões frequentes | `amazon_temas[expressao]` + `[Menções Temas Positivos]` ou `[Menções Temas Negativos]` |
| 5 — Recomendações Estratégicas | Cartões / barras | `[Produtos de Atenção]`, `[Produtos com Oportunidade]`, `[% Produtos de Atenção]`, `[% Produtos com Oportunidade]` |
| 5 | Tabela prioritária | `product_id`, `product_name`, `categoria_principal`, `rating`, `rating_count`, `perfil_cluster`, `sentimento`, `Classificação Estratégica` |

## Cuidados e validação

1. **Evitar medidas redundantes:** `[Total Produtos]` é reutilizada em vários gráficos. Não criar uma nova medida para cada visual se o cálculo for idêntico.
2. **Não confundir engajamento com soma de avaliações:** `[Avaliações do Produto] = MAX(rating_count)` é útil no **contexto de um produto**, mas um cartão isolado exibirá o máximo do catálogo (426.973), e não a soma total.
3. **Percentuais por cluster:** com `perfil_cluster` em branco no visual, `[% Produtos por Cluster]` tende a 100%; usar em gráficos agrupados por cluster ou na matriz, não isoladamente como KPI.
4. **Score VADER:** `0,84` deve aparecer como número decimal, **não 84%**. E 95,04% de textos predominantemente positivos **não equivale** a 95,04% de consumidores satisfeitos.
5. **Frequências de expressões:** 2.208 (temas positivos) e 102 (temas negativos) são **somas de contagens de expressões selecionadas**; podem contar um produto mais de uma vez e não devem aparecer como número de clientes.
6. **Categorias pequenas:** uma categoria com 1 ou 2 produtos pode gerar percentuais extremos; incluir sempre `[Total Produtos]` nas dicas de ferramentas e, na visão executiva, considerar Top N por volume.
7. **Textos e separadores:** as fórmulas usam `,` como separador de argumentos. Caso sua instalação exija `;`, adapte-os sem alterar literais. O número `43 / 10` foi escrito assim para evitar ambiguidades de separador decimal.
8. **Filtros cruzados:** ao verificar resultados como 49 e 81, limpe filtros de página, segmentadores e seleções de visuais.
9. **Power BI é a fonte executável:** este documento é a versão documentada; confira o arquivo `.pbix` final antes de publicar para garantir correspondência exata de nomes e regras.

## Referência dos indicadores (base completa)

| Indicador | Valor |
|---|---:|
| Produtos únicos | 1.351 |
| Categorias principais | 9 |
| Nota média | 4,09 |
| Desconto médio | 46,69% |
| Preço promocional médio | ₹ 3.304,65 |
| Preço original médio | ₹ 5.691,18 |
| Produtos com desconto positivo | 1.304 |
| Produto de atenção | 49 |
| Candidato a oportunidade | 81 |
| Produtos classificados pelo K-Means | 1.348 |
| Clusters | 5 |
| Sentimento textual predominante positivo | 1.284 |
| Sentimento textual predominante negativo | 61 |
| Sentimento textual predominante neutro | 6 |
| Score médio VADER | 0,84 |

*Os valores deste documento foram conferidos na base corrigida em `data/processed/amazon_powerbi_corrigido.csv`. Recomenda-se conferir no `.pbix` depois de aplicar a versão final das fórmulas e os tipos corretos de dados.*
