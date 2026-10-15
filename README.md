# Auditor Inteligente de Planilhas | Análise e Dashboard de Vendas

**Projeto de portfólio em Análise de Dados e Business Intelligence**: auditoria e tratamento de uma base demonstrativa de vendas com Python/Pandas, análise de indicadores comerciais e visualização em um dashboard HTML interativo.

**Tecnologias:** `Python` · `Pandas` · `Jupyter Notebook` · `Excel/XLSX` · `HTML` · `CSS` · `JavaScript` · `GitHub Pages`  
**Abordagem:** Data Cleaning · Data Quality · Análise Exploratória · KPIs Comerciais · Visualização de Dados

> **📊 [ACESSAR O DASHBOARD INTERATIVO — GITHUB PAGES](https://ayrton-marquezin-venancio.github.io/auditor-inteligente-planilhas/)**  

![Prévia do dashboard Auditor Inteligente — Visão Geral](assets/dashboard-preview.png)

## Visão geral

O projeto simula um cenário de análise comercial: a base de vendas recebida apresenta **registros duplicados, preços ausentes, quantidades negativas e quantidades não informadas**. O objetivo foi **identificar, tratar ou sinalizar esses problemas sem ocultar movimentações relevantes**, gerar indicadores confiáveis e construir um painel executivo navegável.

O trabalho foi realizado em duas frentes complementares:

1. **Auditoria e análise (Python/Pandas):** tratamento documentado em notebook, verificações de qualidade e construção das métricas.
2. **Visualização (dashboard HTML):** aplicação estática interativa, alimentada pela planilha final, com filtros múltiplos e gráficos atualizados conforme as seleções.

> **Escopo atual:** solução desenvolvida para uma estrutura específica de vendas. **Não é um pipeline universal de limpeza automática nem utiliza modelos de Machine Learning para tratar os dados.**

## Resultados verificados

Os valores abaixo foram confrontados com o CSV original, o arquivo XLSX tratado e os registros incorporados ao dashboard.

| Indicador | Resultado |
|---|---:|
| Registros na base bruta | **1.015** |
| Duplicatas exatas removidas | **15** |
| Registros na base tratada | **1.000** |
| Preços unitários ausentes preenchidos | **12** |
| Registros classificados como venda válida | **978** |
| Movimentações classificadas como troca/devolução | **10** |
| Registros pendentes de revisão | **12** |
| Faturamento bruto das vendas válidas | **R$ 432.044,80** |
| Ticket médio **por registro válido** | **R$ 441,76** |
| Unidades vendidas nas vendas válidas | **4.452** |
| Valor associado a trocas/devoluções | **-R$ 1.868,00** |
| Período analisado | **01/01/2026 a 30/09/2026** |

### Insights comerciais

- **Maior volume em unidades:** Short, com **655 unidades**.
- **Maior faturamento por produto:** Tênis, com **R$ 119.230,30**.
- **Pagamento mais utilizado e de maior faturamento:** Pix, com **422 registros válidos** e **R$ 191.405,20**.
- **Mês de maior faturamento:** agosto/2026, com **R$ 58.663,70**.
- **Dia de maior faturamento:** 01/09/2026, com **R$ 5.257,10**.

Os valores negativos foram **classificados** como trocas/devoluções segundo a regra deste conjunto de dados; **não equivalem necessariamente a reembolsos ou prejuízo financeiro**.

## Dados e regras do projeto

### Esquema de entrada

| Coluna | Descrição |
|---|---|
| `id_venda` | Identificador da movimentação |
| `data` | Data da venda no padrão `AAAA-MM-DD` |
| `produto` | Nome do produto |
| `quantidade` | Unidades informadas; pode conter valor ausente ou negativo |
| `preco_unitario` | Preço unitário registrado |
| `forma_pagamento` | Pix, Crédito, Débito ou Dinheiro |

### Colunas produzidas

| Coluna | Regra |
|---|---|
| `status` | Classificação de `quantidade`: **Venda Válida** (> 0), **Troca/Devolução** (< 0), **Zero - REVISAR** (= 0), **Ausente - REVISAR** (nula). Na base final não ocorreram quantidades iguais a zero. |
| `valor_total` | `quantidade × preco_unitario`; permanece ausente quando não há quantidade. |

### Processo técnico — da base bruta ao resultado

**01 · Importação e inspeção inicial**  
Leitura do CSV com `pd.read_csv()`, conferência de dimensões e amostra das linhas (`shape`, `head`). Levantamento de ausências por coluna com `isna().sum()`.

**02 · Diagnóstico de duplicidade**  
Investigação de **linhas completamente duplicadas** com `duplicated()` e de **IDs repetidos** com `duplicated(subset="id_venda", keep=False)`. Comparações por identificador ajudam a distinguir uma cópia exata de um possível conflito entre registros.

**03 · Preservação e limpeza inicial**  
Criação de `df_limpo = df.copy()` para conservar o DataFrame original em memória e remoção das **15 duplicatas exatas** com `drop_duplicates()`, resultando em **1.000 linhas**.

**04 · Padronização textual**  
Normalização da coluna `produto` com `str.strip()` e `str.title()`, eliminando espaços periféricos e padronizando a capitalização. Conferência de frequências dos nomes.

**05 · Tratamento de preços ausentes**  
Conferência dos preços conhecidos por produto (`groupby`, `nunique`, `first`). O valor de referência é alinhado às linhas com `groupby("produto")["preco_unitario"].transform("first")` e utilizado em `fillna()`. **12 preços unitários foram preenchidos** e não restaram preços ausentes na base final. Essa regra pressupõe um preço conhecido consistente para cada produto **nesta base**.

**06 · Classificação das movimentações**  
Contagem e inspeção de quantidades ausentes e negativas; aplicação de uma função Python `classificar_quantidade()` com `apply()`. **12 registros ficaram sinalizados para revisão**, sem imputação artificial da quantidade; **10** foram classificados como trocas/devoluções.

**07 · Cálculo financeiro com segregação de status**  
Criação de `valor_total` e cálculo do faturamento **somente sobre** `status == "Venda Válida"`. Movimentações negativas e registros incompletos permanecem na base, mas são analisados separadamente.

**08 · Indicadores e rankings**  
Cálculo de quantidade de registros, unidades vendidas, ticket médio por registro válido, agrupamentos de produtos e meios de pagamento (`groupby`, `agg`, `sum`, `sort_values`, `idxmax`).

**09 · Análise temporal**  
Conversão da coluna de datas com `pd.to_datetime(format="%Y-%m-%d", errors="coerce")`; cálculo dos faturamentos diário e mensal com agrupamentos por data e `.dt.to_period("M")`.

**10 · Validações e resumo executivo**  
Comparação entre **faturamento bruto, soma diária e soma mensal**, com arredondamento para duas casas decimais; verificação de contagens por classificação e integridade dos IDs; consolidação dos KPIs em `resumo_executivo`.

Exemplo representativo da lógica financeira utilizada:

```python
vendas_validas = df_limpo[df_limpo["status"] == "Venda Válida"]
faturamento_bruto = vendas_validas["valor_total"].sum()
ticket_medio = vendas_validas["valor_total"].mean()
```

## Dashboard interativo

O dashboard V3 foi desenvolvido em **HTML + CSS + JavaScript nativo**, reunidos em um arquivo autônomo, **sem backend, bibliotecas externas nem conexão obrigatória com a internet**. A interface disponibiliza:

- **Visão Geral:** cartões de KPIs, evolução da receita e insights.
- **Vendas e Faturamento:** análise temporal diária/mensal.
- **Produtos:** rankings por unidades e por faturamento e comparações entre produtos.
- **Formas de Pagamento:** distribuição por frequência e contribuição financeira.
- **Auditoria:** categorias de status, dados pendentes e movimentações negativas.
- **Base de Dados:** consulta dos registros, pesquisa, ordenação, paginação e exportação do recorte em CSV.

Os filtros de **período, produto e pagamento** aceitam **múltiplas seleções**, funcionam em combinação e atualizam os indicadores e gráficos. Cliques em visualizações relevantes abrem detalhamentos. O painel usa uma identidade visual preto-grafite e transições discretas.

**Como funciona:** os **1.000 registros da planilha tratada estão incorporados ao HTML**, e as métricas exibidas nos diferentes recortes são recalculadas no navegador. O painel **não acompanha automaticamente alterações no notebook ou no XLSX**: para refletir novos dados, o HTML precisa ser atualizado ou a arquitetura precisa evoluir para importação dinâmica.

## Estrutura do repositório

```text
.
├── README.md
├── auditor_inteligente.ipynb            # Notebook de tratamento e análise
├── vendas_projeto_auditor.csv           # Base demonstrativa original
├── auditor_planilha_tratada.xlsx        # Resultado final do tratamento
├── dashboard_auditor_inteligente_v3.html # Arquivo HTML do dashboard
├── index.html                           # Cópia do dashboard para GitHub Pages
├── requirements.txt
├── .gitignore
├── .nojekyll                         # Publicação estática sem processamento Jekyll
└── assets/
    └── dashboard-preview.png
```

## Como reproduzir a análise

1. Clone ou baixe o repositório e abra sua pasta no terminal.
2. Crie um ambiente virtual e instale as dependências:

```bash
python -m venv .venv
# Ative o ambiente virtual conforme seu sistema operacional.
pip install -r requirements.txt
```

3. Inicie o Jupyter:

```bash
jupyter notebook
```

4. Abra `auditor_inteligente.ipynb` e execute as células **de cima para baixo**, com `vendas_projeto_auditor.csv` na mesma pasta. O notebook lê esse nome de arquivo explicitamente.
5. Consulte `auditor_planilha_tratada.xlsx` como o resultado tratado disponibilizado neste repositório. Para reproduzir a exportação, ao fim do notebook execute:

```python
df_limpo.to_excel("auditor_planilha_tratada.xlsx", index=False)
```

> **Atenção:** o notebook atual foi estruturado para essa base específica. Novos CSVs devem ser analisados quanto a nomes de colunas, tipos e regras de negócio antes de reutilizar a rotina.

## Como disponibilizar o dashboard no GitHub Pages

A versão do dashboard destinada à publicação já está em `index.html`, acompanhada de `.nojekyll` para publicar os arquivos estáticos diretamente. **O repositório ainda precisa ser publicado/configurado no GitHub para haver um endereço público real.**

1. Crie um repositório público chamado `auditor-inteligente-planilhas` (ou adapte o nome nas URLs) e envie os arquivos deste projeto para a **raiz da branch `main`**, mantendo `index.html` na raiz.
2. Acesse **Settings → Pages → Build and deployment**.
3. Em **Source**, selecione **Deploy from a branch**; em **Branch**, escolha **`main`** e a pasta **`/(root)`**; salve.
4. Aguarde o GitHub disponibilizar a URL e confirme o endereço apresentado em **Settings → Pages**.
5. **Teste o acesso em uma janela anônima** e substitua o link destacado no início deste README pelo endereço real, semelhante a:

```text
https://SEU-USUARIO.github.io/auditor-inteligente-planilhas/
```

A publicação é uma hospedagem **estática**: não executa Python e não necessita de servidor próprio. **O conteúdo incorporado ao HTML ficará público**, portanto não publique informações confidenciais de clientes ou empresas.

Documentação oficial: [Configurando a origem de publicação do GitHub Pages](https://docs.github.com/pt/pages/getting-started-with-github-pages/configuring-a-publishing-source-for-your-github-pages-site).

## Desenvolvimento assistido por IA — transparência de autoria

Este projeto utilizou **desenvolvimento assistido por Inteligência Artificial (*AI-assisted development*)** como apoio à aprendizagem, à documentação e à implementação:

- **Notebook:** desenvolvi de maneira incremental as etapas de auditoria e análise em Python/Pandas, utilizando IA como apoio para esclarecimento de conceitos, revisão de sintaxe, identificação de erros, depuração e discussão de abordagens. As regras de tratamento foram definidas e validadas durante o desenvolvimento.
- **Dashboard:** defini os requisitos funcionais, a organização das análises e um **prompt inicial de minha autoria**. Esse prompt foi **refinado com auxílio de IA**, e a implementação completa da interface HTML/CSS/JavaScript foi **gerada com IA sob minha direção**, passando por iterações de layout, filtros, interatividade e conferência dos indicadores.
- **Responsabilidade pelos resultados:** as decisões de negócio, a interpretação das métricas, as verificações e a comunicação das limitações fazem parte do desenvolvimento do projeto.

**Distinção importante:** a IA foi utilizada **no processo de desenvolvimento**, não como um modelo de IA integrado à execução do tratamento. A solução atual opera com **regras determinísticas em Pandas** e gráficos/interações no navegador.

## Limitações atuais

- A estrutura de entrada e os tratamentos estão vinculados às **seis colunas** previstas no CSV demonstrativo.
- A imputação de preço por produto é válida apenas quando essa regra for compatível com a política comercial. **Não deve ser aplicada automaticamente em cenários com descontos, promoções ou preços variáveis.**
- Quantidade negativa foi tratada como **classificação de auditoria**, sem prova de reembolso financeiro.
- Registros sem quantidade **não foram corrigidos artificialmente**: continuam pendentes na planilha final.
- O notebook ainda exige execução manual e não fornece, por si só, upload de diferentes bases com reconhecimento automático de estrutura.
- O dashboard atual carrega dados incorporados, **sem atualização automática a partir de novo CSV/XLSX**.
- As somas financeiras são feitas sobre valores numéricos de ponto flutuante; uso futuro em contexto contábil deve considerar precisão decimal e conciliações mais rigorosas.

## Roadmap — automação para bases empresariais

A evolução planejada é converter o trabalho atual em um **pipeline de qualidade e tratamento de dados configurável por regras de negócio**:

| Etapa | Evolução planejada | Resultado esperado |
|---|---|---|
| **1 · Refatoração** | Separar importação, diagnóstico, limpeza, classificação, métricas e exportação em funções/módulos | Execução reaproveitável fora das células do notebook |
| **2 · Entrada flexível** | Suportar upload de arquivos `.csv` e `.xlsx` e configuração de delimitador, encoding e abas | Menos intervenção manual na importação |
| **3 · Mapeamento de esquema** | Mapear colunas equivalentes (`qtd`, `quantidade`, `qtde_vendida`) e validar campos obrigatórios | Compatibilidade com layouts distintos |
| **4 · Regras por empresa** | Criar configurações JSON/YAML para preços, devoluções, obrigatoriedade e políticas de preenchimento | Tratamentos compatíveis com o negócio |
| **5 · Validação e rastreabilidade** | Preservar o bruto, gerar logs de antes/depois, separar erros bloqueantes de pendências e adicionar testes automatizados | Auditoria reproduzível e verificável |
| **6 · Exportação e dashboard** | Produzir planilha final, relatório de exceções, métricas e atualização dos dados publicados no dashboard | Fluxo completo do arquivo recebido até o painel |

**Princípio técnico:** automatizar somente correções justificadas por regras explícitas; decisões ambíguas devem gerar alertas ou exigir aprovação, e não alterar silenciosamente os dados.

---

**Contexto:** projeto de portfólio com dados demonstrativos. Os resultados apresentados não representam uma operação comercial real.  
**Foco:** Python, Pandas, auditoria de qualidade de dados, métricas de vendas e visualização interativa.
