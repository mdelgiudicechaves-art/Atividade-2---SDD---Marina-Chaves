# Sistema de Suporte à Decisão (SSD) para Expansão de Infraestrutura de Armazenamento em Nuvem

Este repositório contém o estudo de viabilidade técnica, econômica e operacional para a implementação de uma nova unidade de armazenamento de dados em nuvem. A análise está estruturada na forma de um **Sistema de Suporte à Decisão (SSD)**, avaliando 9 alternativas regionais de infraestrutura sob a ótica de **CUSTO**, **RISCO** e **TEMPO**.

---

## 1. Mapeamento da Decisão e Arcabouço Metodológico

### Classificação do Problema Decisório
A decisão de expansão de data center situa-se no nível **Tático-Estratégico** e possui natureza **Semiestruturada** (Quadro de Gorry e Scott Morton):
* **Dimensão Estruturada:** Métricas quantitativas extraídas da telemetria, tais como custo fixo e variável das instâncias ($USD$), vazão de rede ($MB/s$) e latência ($ms$).
* **Dimensão Não Estruturada:** Avaliação qualitativa da tolerância ao risco operacional, priorização do perfil de cliente (foco em menor custo versus foco em menor tempo de transferência) e ponderação dos pesos decisórios.

### Ciclo Decisório de Simon
1. **Inteligência:** Extração e consolidação de 1.000 registros de telemetria multi-cloud (`Cloud_Dataset.csv`) cobrindo 3 provedores (`AWS`, `Azure`, `GCP`) e 3 regiões (`us-east`, `us-west`, `asia-south`).
2. **Concepção:** Modelagem das alternativas de alocação e definição de indicadores explícitos para Risco, Custo e Tempo.
3. **Escolha:** Aplicação de testes não paramétricos de hipóteses e implementação do algoritmo multicritério **TOPSIS** (*Technique for Order of Preference by Similarity to Ideal Solution*).
4. **Implementação:** Simulação de cenários de sensibilidade e definição de plano de governança para monitoramento contínuo do sistema.

---

## 2. Definção dos Indicadores do Triângulo Decisório

### A. CUSTO (Dimensão Financeira)
* **Custo Operacional Direto (`avg_cost`):** Valor médio gasto por hora por instância ($USD/h$).
* **Projeção Mensal do Cluster (`monthly_cost_100_vms`):** Custo projetado para manter uma operação de 100 instâncias virtuais ativas ($USD/mês$).
* **Eficiência Financeira de Dados (`cost_efficiency`):** Volume de dados transferidos por unidade monetária ($MB / USD$).

### B. RISCO (Dimensão Operacional e SLA)
* **Risco de Saturação da Infraestrutura (`scale_up_rate`):** Proporção de observações em que a utilização atingiu o gargalo, exigindo autoscaling (medida direta do risco de indisponibilidade ou estouro de capacidade).
* **Risco de Instabilidade da Carga (`util_volatility`):** Desvio padrão da taxa de utilização dos servidores, medindo a volatilidade do sistema.
* **Risco de Cauda na Latência (`p95_latency`):** Latência no percentil 95 ($ms$), representando o pior cenário de tempo de resposta para contratos de SLA.

### C. TEMPO (Dimensão de Desempenho e Vazão)
* **Latência Média de Resposta (`avg_latency`):** Tempo médio de viagem de pacote na rede ($ms$).
* **Vazão de Transferência (`avg_throughput`):** Velocidade de escrita/leitura continuada ($MB/s$).
* **Tempo de Transferência de Carga de 1 TB (`time_1tb_minutes`):** Janela temporal necessária para mover um lote de 1 Terabyte de dados (em minutos).

---

## 3. Consolidação dos Dados e Análise Comparativa

A tabela abaixo resume os indicadores calculados diretamente a partir dos microdados de telemetria:

| Provedor | Região | Custo Médio ($/h) | Custo Mensal (100 VMs) | Scale-Up / Saturação (%) | Volatilidade Utilização (%) | Latência p95 (ms) | Throughput (MB/s) | Tempo Transf. 1 TB (min) |
| :--- | :--- | :---: | :---: | :---: | :---: | :---: | :---: | :---: |
| **Azure** | **us-west** | **$0,0505** | **$3.633,95** | **20,18%** | **20,14%** | **287,50** | **1030,22** | **16,18** |
| **GCP** | **us-east** | $0,0570 | $4.104,00 | 24,58% | 14,81% | 272,46 | 1075,89 | 15,49 |
| **GCP** | **us-west** | $0,0551 | $3.963,70 | 27,62% | 19,16% | 282,69 | 1078,41 | 15,45 |
| **AWS** | **us-east** | $0,0589 | $4.239,00 | 36,54% | 17,11% | 272,42 | 1131,71 | 14,73 |
| **Azure** | **asia-south** | $0,0580 | $4.178,57 | 33,04% | 19,16% | 278,72 | 1078,77 | 15,45 |
| **GCP** | **asia-south** | $0,0586 | $4.219,20 | 31,25% | 18,63% | 282,31 | 1065,02 | 15,65 |
| **Azure** | **us-east** | $0,0596 | $4.294,35 | 30,48% | 19,10% | 278,42 | 1080,41 | 15,43 |
| **AWS** | **us-west** | $0,0612 | $4.407,69 | 29,46% | 16,29% | 277,02 | 1067,02 | 15,62 |
| **AWS** | **asia-south** | $0,0644 | $4.634,92 | 30,84% | 17,28% | 275,22 | 1156,97 | 14,41 |

### Destaques das Três Dimensões:
1. **Liderança em CUSTO:** A localização **Azure us-west** apresenta o menor custo operacional mensal ($3.633,95), gerando uma economia de **21,6%** em relação à opção mais dispendiosa (**AWS asia-south**, de $4.634,92).
2. **Liderança em RISCO:** O provedor **GCP us-east** entrega o menor risco de oscilação (volatilidade de 14,81%) e a menor latência p95 (272,46 ms). Já a opção **Azure us-west** possui o menor risco de gargalo de capacidade (apenas 20,18% de solicitações de *scale up*).
3. **Liderança em TEMPO:** A opção **AWS asia-south** entrega a maior velocidade de transferência (1156,97 MB/s), concluindo a movimentação de 1 TB em 14,41 minutos, comparado a 16,18 minutos da **Azure us-west**.

---

## 4. Validação Estatística das Variáveis

Para fundamentar as premissas analíticas do SSD, aplicou-se um protocolo rigoroso de testes de hipóteses:

### 1. Testes de Normalidade de Shapiro-Wilk
* **Custo Monetário:** $W = 0,6012, \ p = 5,959 \times 10^{-43}$
* **Latência de Rede:** $W = 0,9965, \ p = 0,0260$
* **Vazão / Throughput:** $W = 0,9955, \ p = 0,0052$
* **Utilização de CPU/Memória:** $W = 0,9936, \ p = 0,0003$
* **Conclusão:** Todas as variáveis operacionais apresentaram desvio estatisticamente significativo da distribuição normal ($p < 0,05$). Por essa razão, análises de variância paramétricas foram substituídas por métodos não paramétricos.

### 2. Teste de Kruskal-Wallis (Diferenças entre Provedores)
* **Latência de Rede:** $H = 6,1451, \ p = 0,0463$. Confirma que há diferença estatisticamente significante de desempenho de rede entre os provedores ao nível de confiança de 95%.
* **Custo Direto:** $H = 0,7466, \ p = 0,6884$. Indica que a precificação geral agregada não difere entre marcas, mas varia em função da região geográfica e do tipo de máquina alocada.

### 3. Teste Qui-Quadrado de Independência ($\chi^2$)
* **Provedor vs. Ação de Escalonamento (`target`):** $\chi^2 = 4,6315, \ p = 0,3272$. A probabilidade de ocorrência de gargalo de infraestrutura não possui associação com a escolha do provedor em si, mas sim com a oscilação pontual da demanda regional.

---

## 5. Modelagem Multicritério TOPSIS e Análise de Sensibilidade

O algoritmo TOPSIS calcula a distância euclidiana de cada opção em relação à Solução Ideal Positiva ($PIS$) e à Solução Ideal Negativa ($NIS$).

### Matriz de Pesos por Cenário de Decisão

| Cenário | Peso Custo | Peso Risco (Saturação + Volatilidade) | Peso Tempo (Throughput + Latência) |
| :--- | :---: | :---: | :---: |
| **1. Equilibrado (Padrão)** | 35% | 35% (20% Latência + 15% Saturação) | 30% (Throughput) |
| **2. Foco em Redução de Custo** | 50% | 25% | 25% |
| **3. Foco em Risco / Alta Disponibilidade** | 20% | 45% | 35% |

### Ranking das Melhores Opções por Cenário

1. **Cenário Equilibrado:**
   * **1º Lugar:** Azure (us-west) — Score TOPSIS: **0,6837**
   * **2º Lugar:** GCP (us-east) — Score TOPSIS: **0,5881**
   * **3º Lugar:** GCP (us-west) — Score TOPSIS: **0,5686**

2. **Cenário Foco em Custo:**
   * **1º Lugar:** Azure (us-west) — Score TOPSIS: **0,7993**
   * **2º Lugar:** GCP (us-west) — Score TOPSIS: **0,6262**
   * **3º Lugar:** GCP (us-east) — Score TOPSIS: **0,5590**

3. **Cenário Foco em Baixo Risco e Desempenho:**
   * **1º Lugar:** Azure (us-west) — Score TOPSIS: **0,6510**
   * **2º Lugar:** GCP (us-east) — Score TOPSIS: **0,6406**
   * **3º Lugar:** AWS (asia-south) — Score TOPSIS: **0,4205**

**Recomendação Estratégica do SSD:** A alternativa **Azure us-west** consolida-se como a escolha ótima dominante em todos os cenários. Ela apresenta o menor custo financeiro e a menor taxa de eventos de saturação de capacidade, compensando a ligeira diferença no tempo absoluto de transferência de dados.

---

## 6. Instruções de Uso

1. Copie o arquivo do dataset `Cloud_Dataset.csv` (ou `Cloud_Dataset_2.csv`) para a pasta de trabalho do projeto.
2. Abasteça o código presente em `SSD_Cloud_Expansion.py` (ou execute as células do notebook em um ambiente Jupyter/Colab).
3. Os gráficos analíticos de Custo, Risco e Tempo serão gerados automaticamente na pasta local.
