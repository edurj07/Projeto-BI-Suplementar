# Copazz Saúde — Business Intelligence em Saúde Suplementar

> **Projeto independente de portfólio, desenvolvido com dados 100% fictícios.** Copazz Saúde é uma marca fictícia; este projeto não é oficial e não apresenta informações de uma operadora real.

## Objetivo
Simular a construção de uma solução de BI para gestão de saúde suplementar, integrando informações assistenciais, financeiras, cadastrais e de satisfação. O foco é demonstrar **ETL, qualidade de dados, modelagem dimensional, DAX, SQL, indicadores estratégicos e comunicação gerencial**.

## Dashboard interativo
🚧 **Em preparação.** O dashboard HTML será publicado via GitHub Pages após a inclusão do `index.html` e de suas dependências. **Não há link funcional disponível ainda.**

Endereço planejado: https://edurj07.github.io/Projeto-BI-Suplementar/

## Indicadores e análises
| Área | Indicadores e perspectivas |
| --- | --- |
| Financeiro | Recebimentos, despesas assistenciais, evolução mensal, relação despesa/receita |
| Assistencial | Quantidade de guias, custo médio por guia, despesas por procedimento e prestador, Pareto de custos |
| Beneficiários | Distribuição por plano, idade, faixa etária, município e região |
| Satisfação | Nota média de satisfação, distribuição de avaliações e resolução no primeiro contato |
| Qualidade | Duplicidades, chaves ausentes, valores negativos e completude cadastral |

**Critérios de interpretação:** a razão despesa/recebimento é uma aproximação gerencial e não equivale necessariamente à sinistralidade regulatória. Não há valor monetário de glosa na base, portanto uma taxa de glosa financeira não deve ser calculada sem dados adicionais. A situação cadastral atual não é suficiente para calcular churn histórico sem snapshots ou eventos de entrada/saída.

## Arquivos de dados
Os CSVs sintéticos serão adicionados à pasta `dados/`:

- `Dim_Beneficiarios.csv` — cadastro e perfil de beneficiários.
- `Dim_Prestadores.csv` — cadastro e segmentação de prestadores.
- `Fato_Atendimentos.csv` — guias, procedimentos e valores assistenciais.
- `Atendimentos_Raw.csv` — dados brutos com inconsistências intencionais para exercícios de ETL.
- `Fato_Mensalidades.csv` — competências, contribuições e recebimentos.
- `Fato_Satisfacao.csv` — pesquisas e avaliações de atendimento.
- `Dicionario_Dados.csv` — descrição dos campos.
- `Catalogo_Indicadores.csv` — definições e finalidades gerenciais.

**Granularidade:** na simulação, cada guia corresponde a uma linha em `Fato_Atendimentos`. Em produção, uma guia pode conter vários itens; portanto, duplicidades não devem ser removidas apenas pelo número da guia sem conhecer a chave de cada item.

## Modelagem dimensional
Modelo estrela com relacionamentos **1:N, filtro unidirecional**:

- `Dim_Beneficiarios[ID_Beneficiario]` → fatos de atendimentos, mensalidades e satisfação.
- `Dim_Prestadores[ID_Prestador]` → fato de atendimentos.
- `Dim_Calendario[Data]` → datas correspondentes dos fatos.

Evita-se relacionar tabelas fato diretamente por beneficiário, porque possuem granularidades distintas e isso pode inflar contagens e somatórios.

## Exemplos DAX
```dax
Despesa Assistencial = SUM(Fato_Atendimentos[Valor_Pago])
Recebimentos = SUM(Fato_Mensalidades[Valor_Recebido])
Guias = DISTINCTCOUNT(Fato_Atendimentos[ID_Guia])
Custo Medio por Guia = DIVIDE([Despesa Assistencial], [Guias])
Relacao Despesa Receita = DIVIDE([Despesa Assistencial], [Recebimentos])
```

## Exemplos SQL
**Despesas por tipo de procedimento**
```sql
SELECT Tipo_Procedimento, COUNT(*) AS Total_Guias,
       SUM(Valor_Pago) AS Despesa_Total
FROM Fato_Atendimentos
GROUP BY Tipo_Procedimento
ORDER BY Despesa_Total DESC;
```

**Prestadores com despesas superiores a R$ 50 mil**
```sql
SELECT ID_Prestador, COUNT(*) AS Total_Guias,
       SUM(Valor_Pago) AS Despesa_Total
FROM Fato_Atendimentos
GROUP BY ID_Prestador
HAVING SUM(Valor_Pago) > 50000
ORDER BY Despesa_Total DESC;
```

**Identificação de guias repetidas na origem**
```sql
SELECT ID_Guia, COUNT(*) AS Ocorrencias
FROM Atendimentos_Raw
GROUP BY ID_Guia
HAVING COUNT(*) > 1;
```

## Governança e qualidade
A base bruta contém, intencionalmente, linhas repetidas, prestadores ausentes, valores negativos e procedimentos sem identificação. Recomenda-se preservar a origem, criar uma camada tratada e registrar exceções. Valores negativos podem representar estornos; não devem ser eliminados automaticamente sem regra de negócio.

## Tecnologias
Power BI, Power Query, DAX, SQL, Excel, HTML/CSS/JavaScript e GitHub Pages.

## Publicação web
Para disponibilizar o dashboard: adicionar `index.html` e todos os recursos necessários à raiz do repositório; substituir referências visuais de **Copass** por **Copazz**; eliminar caminhos locais como `file:///`; ativar **Settings → Pages → Deploy from a branch → main → /(root)** e testar a URL em janela anônima.

## Escopo e limitações
Os dados são sintéticos, os resultados são demonstrativos e não representam indicadores reais. **O arquivo `.pbix` não será disponibilizado neste repositório.**

## Autor
[edurj07](https://github.com/edurj07) — projeto de portfólio em análise de dados e Business Intelligence.
