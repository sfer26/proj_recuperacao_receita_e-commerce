## **Projeto de Recuperação de Receita: Análise e Modelo Preditivo**
#### 📊**1. Visão Geral do Desafio**
- **1.1. Objetivos de Negócio:**
    - Analisar os dados de eventos de carrinho para entender os principais motivos do abandono.
    - Propor estratégias acionáveis para as equipes de Vendas e Marketing com o objetivo de reduzir a taxa de abandono e recuperar receita.
    - Desenvolver um modelo preditivo para identificar clientes com alta probabilidade de conversão para campanhas de remarketing.
        
- **1.2. Entregáveis:**
    - Dashboard interativo no Looker Studio.
    - Notebook Jupyter/Colab com o processo de análise e o modelo preditivo.
    - Relatório de métricas de performance do modelo.
    - Esta documentação (`README.md`).

- **1.3. Tecnologias Utilizadas:**
    - Python (Pandas, Scikit-learn, Matplotlib, Seaborn)
    - Google Cloud Platform (Firestore)
    - Looker Studio
#### 🔄**2. Pipeline de Dados e Preparação (ETL)**

- **2.1. Extração dos Dados (Extract):**
    - Fonte de dados: Duas coleções (`shoppingCart` e `shoppingCartV2`) no banco de dados NoSQL Firestore.
    - Desafio inicial: Identificação da estrutura correta dos dados (coleções na raiz vs. subcoleções), resolvido com a exploração programática das coleções.
- **2.2. Limpeza e Transformação (Transform):**
    - **Unificação:** Consolidação das duas coleções em um DataFrame único.
    - **Seleção de Features:** Remoção de colunas com dados sensíveis (PII) ou IDs irrelevantes para a análise agregada (`cpf`, `email`, `customer_id`, etc.).
    - **Renomeação:** Padronização dos nomes das colunas para português e maior clareza.
    - **Tratamento de Nulos:** Investigação aprofundada dos valores nulos na coluna `preco`, identificando que estavam concentrados em Março de 2023. A imputação foi feita de forma robusta, utilizando a mediana do período adjacente (Abril de 2023) para evitar viés.
- **2.3. Engenharia de Features:**
    - **Features Temporais:** Criação de colunas (`ano`, `mes`, `dia_semana`, `hora`, `periodo_dia`) para permitir a análise de sazonalidade.
    - **Features de Negócio:** Criação de colunas categóricas para simplificar a análise, como `faixa_preco`, `tipo_plano` (Simples/Combo) e a variável alvo `converteu` (True/False).
- **2.4. Arquitetura do Pipeline:**
    - **Fluxo Ideal (Proposto):** `Firestore -> Colab (processamento) -> BigQuery (tabela tratada) -> Looker Studio`.
    - **Fluxo Implementado (Contingência):** Devido a um bloqueio de permissão de escrita no BigQuery, foi adotado um fluxo alternativo: `Firestore -> Colab (processamento) -> CSV Final -> Looker Studio (via File Upload)`. Esta adaptação garantiu a entrega do projeto.
#### 🔍**3. Análise Exploratória (EDA) e Principais Insights**
- **3.1. O Tamanho do Problema:** Cerca de 63% dos carrinhos são abandonados, representando o principal ponto de perda de receita.
- **3.2. Análise Temporal (O "Quando"):**
    - **Pico de Conversão:** Segundas-feiras e o horário das 14h são os períodos de maior eficiência.
    - **Oportunidade de Nutrição:** As manhãs (9h-12h) apresentam alto volume de navegação, mas baixa conversão, indicando um período de pesquisa.
- **3.3. Análise de Produto e Preço (O "Onde" e "Por Quê"):**
    - **Produto Mais Crítico:** O plano "Assinatura Tech Social ou Aluno" concentra a maior parte da receita abandonada.
    - **Incerteza de Valor:** Planos 'Simples' e produtos de faixa de preço 'Média' apresentam as maiores taxas de abandono, sugerindo hesitação do cliente.
- **3.4. Análise de Pagamento (O Atrito na Experiência):**
    - O `boleto` é o método com a maior proporção de abandonos, provavelmente devido à complexidade e tempo de espera.

#### **📈4. Dashboard Interativo no Looker Studio**

- **4.1. Narrativa e Estrutura:**
    - O dashboard foi estruturado em 4 páginas para contar uma história:
        1. **Visão Geral:** Apresenta o tamanho do problema e os padrões temporais.
        2. **Diagnóstico do Abandono:** Aprofunda nas causas (Onde, Quando e Por Quê).
        3. **Plano de Ação:** Traduz os insights em recomendações estratégicas.
        4. **Recomendações**

- **4.2. Design e Paleta de Cores:**
    - Foi utilizada a paleta "Confiança e Inteligência" (dark mode com tons de azul e turquesa) e cores semânticas (verde para sucesso, laranja/vermelho para alerta) para uma interface intuitiva.

-  **4.3. Link para o Dashboard:** [Link Dashboard Looker](https://lookerstudio.google.com/reporting/15f90470-ce52-43d1-82b5-6b3992f7db19)
    - **O que o usuário encontra:** Uma visão macro dos abandonos, análises temporais e de produto, um diagnóstico das causas e um plano de ação estratégico baseado nos dados.

---
### **🤖5. Modelo Preditivo de Recuperação de Vendas**

- **5.1. Objetivo:**  
    - Prever a coluna `converteu` para identificar clientes que abandonaram o carrinho, mas possuem alta probabilidade de conversão.
- **5.2. Pré-processamento:**
    - Para preparar os dados para os modelos, as colunas categóricas (`metodo_pagamento`, `dia_semana`, `faixa_preco`, `tipo_plano`) foram transformadas em variáveis numéricas binárias usando a técnica **One-Hot Encoding**. Isso garantiu que os algoritmos pudessem processar as informações de forma correta, sem atribuir uma ordem de valor incorreta a categorias de texto.
- **5.3. Algoritmos e Resultados:**
    - Foram testados dois modelos de aprendizado de máquina para prever a conversão: **Random Forest** e **Regressão Logística**. A avaliação focou em métricas como **Acurácia**, **Precisão** e, principalmente, **Recall**, que é a métrica mais relevante para o objetivo de encontrar a maior quantidade possível de oportunidades de venda.

|Modelo Testado|Acurácia|Precisão (Converteu)|Recall (Converteu)|
|---|---|---|---|
|**Random Forest (Inicial)**|0.78|0.68|0.26|
|**Random Forest (Otimizado)**|0.75|0.00|0.00|
|**Regressão Logística**|0.49|0.24|**0.50**|
- **5.4. Análise e Escolha do Modelo:**
    - O modelo **Random Forest (Inicial)**, apesar de sua alta acurácia (78%), demonstrou um `Recall` de apenas 26%, indicando que perdia a maior parte das oportunidades de conversão. Após a otimização, o modelo perdeu totalmente a capacidade de identificar clientes que converteriam.
    - O modelo de **Regressão Logística**, apesar de ter uma acurácia geral mais baixa (49%), alcançou um `Recall` de **50%**. Essa métrica é a mais crítica para o objetivo de negócio, pois indica que o modelo é capaz de identificar metade de todos os clientes que, de fato, converteram. Embora sua precisão seja baixa, a capacidade de encontrar leads valiosos supera o custo de lidar com "falsos positivos".
- **5.5. Conclusão e Impacto Estimado:**
- O modelo de **Regressão Logística** foi selecionado como a solução mais adequada para os objetivos de negócio. Embora apresente uma acurácia geral de 49%, sua capacidade de identificar corretamente 50% dos clientes que efetivamente convertem (Recall = 50%) o torna a ferramenta ideal para **maximizar a recuperação de leads qualificados**.
- **Potencial de Recuperação de Receita:** Considerando a taxa de abandono atual de 📉 63% e a efetividade do modelo (Recall de 50%), *estima-se que a implementação desta solução possa **recuperar até 31,5% da receita total perdida**por carrinhos abandonados*. Este cálculo conservador considera apenas os casos em que o modelo identifica corretamente oportunidades de conversão.
- **Próximos Passos:** A análise dos coeficientes da regressão logística permitirá, em trabalhos futuros, identificar quais variáveis exercem maior influência na probabilidade de conversão, possibilitando o refinamento das estratégias de recuperação e a otimização contínua do modelo preditivo.

  ---

[LinkedIn](https://www.linkedin.com/in/stellafern/)

Email: sdib2626@gmail.com
