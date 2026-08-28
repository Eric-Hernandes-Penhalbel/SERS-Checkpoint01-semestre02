# Análise Temporal e Caracterização da Carga Elétrica Supervisionada e Micro/Mini Geração Distribuída (MMGD) no Estado de São Paulo

## 1. Visão Geral do Projeto
Este repositório contém a resolução do **Desafio Final** da disciplina de **Soluções em Energias Renováveis e Sustentáveis**, pertencente ao curso de **Ciência da Computação**.

O objetivo do estudo é realizar uma análise descritiva, quantitativa e estruturada sobre a dinâmica de carga elétrica no estado de São Paulo (**Área de Carga SP**), integrada ao Sistema Interligado Nacional (SIN). A investigação fundamenta-se em dados de carga verificada consumidos diretamente via **API pública do Operador Nacional do Sistema Elétrico (ONS)** para o período de **01/08/2025 a 07/08/2025**.

A análise engloba a identificação de padrões intradia (dias úteis vs. fins de semana), a mensuração do impacto da Micro e Mini Geração Distribuída (MMGD) na curva de carga global e a aplicação de técnicas de agregação estatística para suporte ao planejamento e à tomada de decisão no setor elétrico.

---

## 2. Integrantes do Grupo
* **Enzo Ricardo Silva** — RM: 571333
* **Eric Hernandes Penhalbel** — RM: 570237
* **João Guilherme Figuereido** — RM: 572697
* **Ryan Luther Roque** — RM: 572993
* **Matheus Borges Soares** — RM: 574085

---

## 3. Metodologia e Pipeline de Dados

O fluxo de processamento de dados e análise técnica foi implementado em Python e está estruturado nas seguintes etapas:

```
┌──────────────────────┐    ┌──────────────────────┐    ┌──────────────────────┐
│  Consulta API ONS    │───>│  Engenharia de Dados │───>│ Análise Estatística  │
│  (JSON via HTTP)     │    │  (Pandas DataFrames) │    │  & Agregações Temporal│
└──────────────────────┘    └──────────────────────┘    └──────────────────────┘
                                                                    │
                                                                    ▼
┌──────────────────────┐    ┌──────────────────────┐    ┌──────────────────────┐
│   Exportação CSV     │<───│  Visualização Gráfica│<───│ Avaliação do Impacto │
│   para Repositório   │    │  (Matplotlib/Seaborn)│    │     da MMGD (Solar)  │
└──────────────────────┘    └──────────────────────┘    └──────────────────────┘
```

1. **Ingestão via API Pública**: Requisitados dados semi-horários do endpoint oficial do ONS (`apicarga.ons.org.br/prd/cargaverificada`), utilizando parâmetros restritos ao Subsistema/Área de Carga de São Paulo (`SP`) no intervalo amostral.
2. **Tratamento e Estruturação**: Normalização dos registros JSON em estruturas tabulares do Pandas (`pd.DataFrame`), conversão do campo UTC `din_referenciautc` em objetos `datetime` e indexação temporal.
3. **Engenharia de Recursos (Feature Engineering)**: Extração de componentes temporais (Hora, Dia, Dia da Semana) e segmentação lógica entre dias úteis e finais de semana.
4. **Exportação Modular (.csv)**: Para garantir reprodutibilidade e modularidade, os recortes analíticos e métricas consolidadas foram exportados em formato CSV estruturado no repositório.

---

## 4. Estrutura do Repositório e Arquivos CSV

O repositório está organizado conforme a seguinte estrutura de arquivos:

```
.
├── Desafio_Final_Energia_ONS_API_Final.ipynb   # Notebook principal com pipeline completo
├── README.md                                    # Documentação técnica do projeto
├── dados_carga_sp_completo.csv                  # Dataset bruto tratado retornado da API
├── consumo_diario_total.csv                     # Agregação do consumo total diário (MWh)
├── perfil_medio_horario.csv                    # Carga média semi-horária/horária típica
├── impacto_mmgd_horario.csv                     # Injeção e penetração relativa da MMGD
└── comparativo_dias_uteis_fim_semana.csv        # Contraste de carga entre tipos de dia
```

### Detalhamento dos Arquivos CSV de Saída

| Arquivo CSV | Separador | Descrição do Conteúdo |
| :--- | :---: | :--- |
| `dados_carga_sp_completo.csv` | `,` | Série temporal completa (336 registros de 30 min) contendo carga global, supervisionada, não supervisionada e MMGD. |
| `consumo_diario_total.csv` | `,` | Consolidação do consumo diário em MWh (integração da potência média semi-horária no tempo). |
| `perfil_medio_horario.csv` | `,` | Média, desvio padrão e valores extremos (mín/máx) da carga agregada por horário do dia. |
| `impacto_mmgd_horario.csv` | `,` | Mapeamento da injeção de geração distribuída solar/eólica por horário e seu efeito de abatimento na curva global. |
| `comparativo_dias_uteis_fim_semana.csv` | `,` | Matriz comparativa da demanda elétrica segmentada entre dias úteis e finais de semana. |

---

## 5. Principais Indicadores e Dicionário de Variáveis

O conjunto de dados fornecido pelo ONS abrange os seguintes campos principais:

* `cod_areacarga`: Identificador geográfico da área de carga (`SP`).
* `din_referenciautc`: Estampa de tempo em formato UTC (intervalos semi-horários).
* `val_cargaglobal`: Carga elétrica total da área (MWmédios), representando a demanda integral observada.
* `val_cargasupervisionada`: Carga medida diretamente pelo sistema de supervisão do ONS.
* `val_carganaosupervisionada`: Carga estimada/medida de pequeno porte não supervisionada diretamente em tempo real.
* `val_cargammgd`: Estimativa de geração distribuída (predominantemente solar fotovoltaica) em operação no mesmo intervalo.

---

## 6. Resultados e Discussões Técnicas

1. **Comportamento Curva "Duck Curve" (Efeito MMGD)**: A inclusão do parâmetro `val_cargammgd` evidencia a acentuada curva do pato durante os períodos de maior irradiação solar (entre 10h00 e 16h00). A geração distribuída alivia a demanda da rede supervisionada no meio do dia, porém gera uma rampa de subida pronunciada no período do crepúsculo (18h00–21h00), momento de pico de consumo residencial aliado à perda de geração solar.
2. **Variabilidade Semanal**: Observa-se uma redução substancial da carga global nos dias de final de semana (sábado e domingo), explicada pela paralisação/redução de atividades do setor industrial e comercial de grande porte no estado de São Paulo.
3. **Consistência dos Dados**: O indicador `val_consistencia` manteve-se zerado ao longo de toda a amostragem, atestando a integridade das medições fornecidas pela plataforma do ONS.

---

## 7. Requisitos e Instruções de Execução

### Pré-requisitos
Para executar o notebook e reproduzir os arquivos do projeto, certifique-se de possuir o **Python 3.8+** instalado juntamente com as bibliotecas:

```bash
pip install pandas matplotlib seaborn requests
```

### Execução
1. Clone o repositório:
   ```bash
   git clone https://github.com/Eric-Hernandes-Penhalbel/SERS-Checkpoint01-semestre02.git
   ```
2. Abra o arquivo `Desafio_Final_Energia_ONS_API_Final.ipynb` em seu ambiente Jupyter Notebook ou Google Colab.
3. Execute todas as células em ordem sequencial para realizar as requisições HTTP, gerar os DataFrames, salvar os arquivos `.csv` e plotar as visualizações gráficas.

---

## 8. Referências Técnicas
* **ONS — Operador Nacional do Sistema Elétrico**: Portal de Dados Abertos (Dataset Carga de Energia Verificada). Disponível em: <https://dados.ons.org.br/dataset/carga-energia-verificada>.
* **Documentação da API ONS**: <https://apicarga.ons.org.br/prd/cargaverificada>.
