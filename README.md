# Tech Challenge 4 — Amazon Product Intelligence

**Pós Tech | Analytical Solutions and Machine Learning Applied to Business**  
**Problema de negócio:** compreender a relação entre **preço → desconto → engajamento → satisfação** e propor decisões para o portfólio da Amazon.

## Visão geral

Este projeto aplica tratamento de dados, análise exploratória, correlações estatísticas, **K-Means** e análise de sentimentos com **VADER**. Os resultados são disponibilizados em um dashboard interativo de cinco páginas no **Power BI**, acompanhados de relatório executivo e slides para apresentação à liderança.

### Principais resultados

| Indicador | Valor |
|---|---:|
| Registros na base original | 1.465 |
| Produtos únicos após deduplicação | **1.351** |
| Categorias principais | 9 |
| Avaliação média | 4,09 / 5 |
| Desconto médio | 46,69% |
| Preço promocional médio | ₹ 3.304,65 |
| Produtos de atenção | 49 |
| Oportunidades potenciais | 81 |
| Produtos classificados em K-Means | 1.348 (5 clusters) |
| Textos agregados com sentimento positivo predominante | 1.284 |

> Quantidade de avaliações (`rating_count`) é um **proxy de engajamento**, não prova de vendas ou visualizações. Os sentimentos foram calculados sobre **textos agregados por produto**, não comentários individualizados. Correlações não demonstram causalidade.

## Diferenciais da solução

- **Qualidade de dados:** verificação dos registros repetidos e seleção documentada de um representante por `product_id`.
- **Engenharia de atributos:** economia potencial, faixas de preços/descontos e transformação logarítmica de engajamento.
- **Análise estatística:** correlações de Pearson (≈ −0,162) e Spearman (≈ −0,150) para desconto × avaliação.
- **Aprendizado não supervisionado:** comparação de K=2 a K=8, seleção de cinco perfis, Silhouette ≈ 0,236 e estabilidade entre inicializações via ARI médio ≈ 0,985.
- **NLP:** classificação de sentimentos com VADER e detecção de expressões frequentes em comentários agregados.
- **Aplicação de negócio:** priorização de produtos de atenção e oportunidades, com indicadores e filtros no Power BI.

## Organização do repositório

```text
Tech_Challenge_4_GitHub/
├── README.md
├── requirements.txt
├── .gitignore
├── notebooks/
│   └── tech_challenge_4_amazon.ipynb
├── data/
│   ├── raw/
│   │   └── .gitkeep                 # colocar amazon.csv aqui (não versionado)
│   └── processed/
│       ├── amazon_powerbi_corrigido.csv
│       └── amazon_temas.csv
├── dashboard/
│   ├── README.md                  # como usar o dashboard e suas fontes
│   └── medidas_powerbi.dax        # todas as medidas e a coluna calculada
│                                   # (adicionar Tech_Challenge_4_Amazon.pbix)
├── docs/
│   ├── Relatorio_Executivo_Tech_Challenge_4_Amazon.docx
│   └── medidas_dax.md             # explicações, indicadores e mapeamento dos visuais
├── apresentacao/
│   └── Apresentacao_Executiva_Tech_Challenge_4_Amazon.pptx
└── assets/
    └── .gitkeep
```

## Como executar

1. Tenha **Python 3.10+** e o dataset original `amazon.csv` (o mesmo utilizado no Tech Challenge).
2. Na pasta do projeto, crie e ative um ambiente virtual e instale as dependências:

   ```bash
   python -m venv .venv
   # Windows (PowerShell): .venv\Scripts\Activate.ps1
   # Linux/macOS: source .venv/bin/activate
   python -m pip install -r requirements.txt
   ```

3. Salve o dataset em **`data/raw/amazon.csv`**. A base bruta não foi incluída no repositório, pois sua redistribuição depende da licença/fonte original.
4. Abra **`notebooks/tech_challenge_4_amazon.ipynb`** no JupyterLab, VS Code ou Google Colab e execute as células em ordem.
5. O notebook grava `data/processed/amazon_powerbi_corrigido.csv` e `data/processed/amazon_temas.csv` com **separador `;` e decimal `,`** (para importar no Power BI com localidade Português/Brasil).
6. Abra o dashboard (`dashboard/Tech_Challenge_4_Amazon.pbix`) quando for adicionado; atualize os caminhos dos CSVs na consulta Power Query, se necessário.

**Alternativa rápida:** os CSVs tratados foram incluídos em `data/processed/` para permitir explorar os dados no Power BI sem executar o notebook. Verifique as regras de uso do conjunto de dados antes de tornar um repositório público.

## Resultados de negócio

- **Electronics** concentra 30 dos 49 produtos de atenção. Recomenda-se investigar comentários e qualidade desses itens.
- **Home & Kitchen** reúne 33 das 81 oportunidades potenciais (boas notas e poucas avaliações).
- A correlação entre desconto e avaliação foi **negativa e fraca**; preços promocionais e percentuais de desconto não devem ser interpretados isoladamente como prova de satisfação.
- A segmentação revela **cinco perfis exploratórios** de produtos. A separação entre grupos é limitada, mesmo com estabilidade entre diferentes inicializações.
- Comentários agregados indicam temas recorrentes de custo-benefício, qualidade e uso, com ressalvas quanto ao contexto das expressões e à granularidade do texto.

## Medidas DAX e regras de negócio no Power BI

Para manter o trabalho **auditável e reproduzível**, as fórmulas dos KPIs e a classificação estratégica dos produtos estão documentadas separadamente do notebook Python:

- **[Catálogo de medidas DAX](docs/medidas_dax.md):** finalidade de cada medida, fórmula, resultado de referência, visual associado e limites de interpretação; organizado pelas cinco páginas.
- **[Expressões DAX (.dax)](dashboard/medidas_powerbi.dax):** fórmulas para consulta e cópia individual no Power BI Desktop.

As regras mais relevantes são: **atenção = nota < 4 e pelo menos 16.020 avaliações**; **oportunidade = nota ≥ 4,3 e até 1.106 avaliações**. Esses limiares foram definidos na análise exploratória. A documentação diferencia explicitamente contagem de avaliações (proxy de engajamento) de vendas e visibilidade.

> O arquivo `.dax` **não é um arquivo importável automaticamente** pelo Power BI. As medidas válidas na entrega são as contidas no `.pbix`; antes da publicação final, conferir nomes de tabelas e eventuais ajustes feitos diretamente no Power BI.

## Materiais de apresentação

- [Relatório executivo (Word)](docs/Relatorio_Executivo_Tech_Challenge_4_Amazon.docx)
- [Apresentação executiva (PowerPoint)](apresentacao/Apresentacao_Executiva_Tech_Challenge_4_Amazon.pptx)
- [Notebook principal](notebooks/tech_challenge_4_amazon.ipynb)

## Fonte, escopo e ética de uso

Este material foi desenvolvido como atividade acadêmica. A origem/atribuição e as condições de redistribuição do dataset original **devem ser confirmadas na fonte de onde foi obtido** antes da publicação pública. Os valores monetários estão em **rúpias indianas (₹)** e não há dados transacionais de vendas. A análise não estabelece relações de causa e efeito. Também não inclui um modelo preditivo supervisionado: a abordagem de Machine Learning é de **agrupamento não supervisionado**.
