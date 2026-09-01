# Sistema de Suporte à Decisão (SSD) para Expansão Multi-Cloud

<p align="left">
  <img src="https://img.shields.io/badge/Python-3776AB?style=for-the-badge&logo=python&logoColor=white"/>
  <img src="https://img.shields.io/badge/Pandas-2C2D72?style=for-the-badge&logo=pandas&logoColor=white"/>
  <img src="https://img.shields.io/badge/Jupyter-F37626?style=for-the-badge&logo=jupyter&logoColor=white"/>
</p>

## Visão Geral do Projeto

Este repositório contém um **Produto Mínimo Viável (MVP)** desenvolvido na forma de um *Jupyter Notebook*, focado em embasar o planejamento estratégico de uma empresa de armazenamento em nuvem que busca abrir uma nova região/data center. 

O objetivo principal deste projeto é processar dados brutos de uso de instâncias em nuvem (Kaggle) e transformá-los em **indicadores de negócio acionáveis**. O leitor encontrará neste notebook um pipeline completo de Análise Exploratória de Dados (EDA), Engenharia de Variáveis e Visualização de Séries Temporais, culminando em insights diretos sobre **Risco, Custo e Tempo**.

---

## Metodologia Analítica

A abordagem metodológica do notebook foi estruturada no ciclo padrão de Ciência de Dados, focando na extração de valor para tomada de decisão:

1. **Coleta e Sanitização de Dados:** 
   O dataset `Cloud_Dataset.csv` é importado e avaliado quanto à sua integridade (tipagem de dados, tratamento de nulos e estruturação cronológica).
2. **Engenharia de Variáveis (Feature Engineering):** 
   Criação de novos eixos analíticos baseados nas variáveis originais para traduzir dados técnicos em métricas de negócio:
   - **Risco:** Definição da flag `sla_violation` (identificando instâncias com latência acima de 200ms) e `capacity_gap` (avaliando gargalos de utilização vs. capacidade disponível).
   - **Tempo:** Cálculo de `time_diff_secs`, medindo o intervalo de frequência e escalonamento dos recursos.
3. **Análise Multivariada e Visualização:**
   Utilização de bibliotecas gráficas com identidade visual padronizada (*viridis/mako*) para explorar a correlação entre provedores (ex: AWS vs Azure), tipos de instâncias, horários de pico e gargalos operacionais.

---

## Métodos de Análise Financeira

A viabilidade de expansão depende estritamente da eficiência de custos. Neste projeto, a modelagem financeira foi construída sobre três pilares analíticos principais:

1. **Eficiência de Custo (`cost_efficiency`):** 
   Calculada através da razão entre o Custo Bruto e o Uso de CPU. Permite identificar quais instâncias ou regiões entregam mais processamento pelo menor preço, evidenciando o real custo-benefício (TCO - *Total Cost of Ownership*).
2. **Custo Unitário Computacional (`cost_per_vcpu`):** 
   Normalização do custo pelo número de vCPUs alocadas, permitindo uma comparação justa entre máquinas virtuais de diferentes tamanhos (*t2.micro*, *B1s*, etc.).
3. **Análise de Tendência de Custo (Série Temporal):**
   Aplicação de técnicas de suavização de séries temporais (como **Média Móvel de 12 horas**) sobre o custo médio horário. Este método isola flutuações e ruídos momentâneos de mercado (ou de escalonamento elástico), revelando a tendência real de queima de capital (*burn rate*) ao longo do tempo.

---

## Visão Geral dos Resultados Obtidos

Ao final da execução do notebook, o sistema gera visualizações cruciais para a tomada de decisão (vide artefatos gerados, como o `time_series_plot.png`):

* **Comportamento de Custos ao Longo do Tempo:** O gráfico de variação de custo cruzado com a Média Móvel permite prever a estabilidade financeira de operar em determinadas regiões. Observamos momentos de pico de custo que coincidem com janelas específicas de operação, sugerindo a necessidade de estratégias de escalonamento dinâmico (*Auto-scaling*).
* **Carga de Demanda e Ociosidade:** O gráfico de área detalhando o Uso de CPU (%) ao longo dos dias revelou os horários de pico crítico de carga de trabalho. Isso cruza diretamente com o indicador de `capacity_gap`, mostrando se a empresa estaria pagando por capacidade ociosa ou arriscando quebras de SLA por subprovisionamento durante os picos.
* **Veredito para Expansão:** Com base no balanço entre *SLA Violations* (Risco) e *Cost Efficiency* (Retorno), o notebook instrumentaliza os gestores para escolherem matematicamente qual região ou provedor oferece a infraestrutura mais resiliente e barata para a nova empreitada.

---

### Estrutura e Execução
**Ambiente:** O notebook pode ser executado em IDEs locais (Jupyter Lab, VS Code) ou ambientes em nuvem (Google Colab).

**Dataset:** Certifique-se de que o arquivo Cloud_Dataset.csv está no mesmo diretório raiz do notebook (ou faça o upload para o ambiente do Colab quando solicitado nas primeiras células).

* **Fluxo de Células:**
O código está parametrizado. As células iniciais definem configurações globais (como a seed para reprodutibilidade e estilos do matplotlib).
As funções de agregação de dados e resampling temporal (resample('1h')) foram escritas utilizando métodos nativos do Pandas, o que permite fácil modificação da janela de tempo (ex: alterar para '1D' para visões diárias).
**Exportação de Artefatos:** Os gráficos finais são automaticamente salvos no diretório atual (ex: time_series_plot.png), facilitando a extração dos resultados para apresentações executivas.

---

## Usabilidade do Código e Guia de Execução

O código foi desenvolvido visando alta legibilidade, modularidade e fácil reprodução.

### Pré-requisitos e Dependências
Para rodar este projeto localmente, recomenda-se a criação de um ambiente virtual (`venv` ou `conda`) com Python 3.8+. As principais bibliotecas requeridas são:
```text
pandas
numpy
matplotlib
seaborn
scipy
scikit-learn

---
