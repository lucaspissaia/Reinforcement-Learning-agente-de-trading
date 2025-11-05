# 🤖 Agente de Trading com Reinforcement Learning (Q-Learning)

Este repositório contém o Trabalho Final de **Aprendizado por Reforço** do **MBA em Data Science & AI da FIAP (10DTSR)**.

O objetivo foi desenvolver e avaliar um agente de trading automatizado, baseado no algoritmo **Q-Learning**, para gerenciar um portfólio de múltiplos ativos (VALE3, PETR4, BRFS3). O agente deveria aprender uma política de decisão (Comprar, Vender ou Manter) para maximizar o retorno financeiro.

---

##  CHALLENGE O Desafio: RL em Mercados Financeiros

O mercado financeiro apresenta desafios únicos para o Aprendizado por Reforço:
* **Espaço de Estados Contínuo:** O preço dos ativos e o saldo da conta podem assumir infinitos valores, tornando uma "Q-Table" tabular (baseada em estados discretos) impraticável ou ineficiente.
* **Não-Estacionariedade:** O comportamento do mercado muda ao longo do tempo (ex: crises, booms). Uma política aprendida em um ano pode não funcionar no ano seguinte.
* **Validação:** Um simples *train/test split* não é válido. É crucial usar métodos que respeitem a ordem temporal dos dados para evitar *lookahead bias* (vazamento de dados do futuro).

## 🔬 Metodologia: Da Prova de Conceito à Validação Temporal

O projeto foi estruturado em uma série de experimentos iterativos para tentar superar esses desafios.

### 1. Ambiente de Simulação (Custom Gym)
Foi criado um ambiente customizado (`TradingEnv`) usando a biblioteca `gymnasium` (Gym).
* **Estado (State):** Inicialmente `[saldo_em_conta, preco_ativo_1, preco_ativo_2, ...]`.
* **Ação (Action):** Um único valor discreto que decodifica uma ação para cada ativo (0=Manter, 1=Comprar, 2=Vender). Ex: 3 ativos = 3³ = 27 ações possíveis.
* **Recompensa (Reward):** A variação percentual do valor total da carteira a cada passo (dia).

### 2. V1: Q-Learning Tabular Simples
* **Abordagem:** O agente usava apenas o saldo e os preços atuais como estado.
* **Resultado:** Recompensa média próxima de 0.0. O agente não aprendeu uma estratégia lucrativa, como esperado, devido ao espaço de estados ser muito grande e contínuo.

### 3. V2: Enriquecimento de Estado (Feature Engineering)
Para dar mais contexto ao agente, o estado foi enriquecido com indicadores técnicos:
* Média Móvel Curta (5 dias)
* Média Móvel Longa (20 dias)
* Momentum (Retorno percentual diário)
* **Resultado:** O Sharpe Ratio melhorou ligeiramente, mas ainda permaneceu próximo de 0. O agente continuou sem uma estratégia clara.

### 4. V3: Validação Temporal (Walk-Forward Validation)
Esta foi a etapa crucial. Em vez de treinar em todo o dataset de uma vez, implementamos uma validação cruzada temporal:
1.  **Split 1:** Treina em (2020-2022), Testa em (2023).
2.  **Split 2:** Treina em (2021-2023), Testa em (2024).
3.  **Split 3:** Treina em (2022-2024), Testa em (2025).

* **Resultado:** O agente apresentou **desempenho inconsistente**. Em alguns splits, superou o mercado; em outros, teve um desempenho muito inferior.

### 5. V4: Otimização com Grid Search
Para garantir que o mau desempenho não era apenas devido a hiperparâmetros ruins, combinamos a Validação Temporal (V3) com um **Grid Search** para testar várias combinações de `alpha` (taxa de aprendizado), `gamma` (desconto) e `epsilon` (exploração).

---

## 📊 Conclusões Finais

O projeto demonstrou com sucesso as limitações do Q-Learning tabular para problemas de trading algorítmico:

1.  **Falha na Generalização:** A validação temporal provou que o agente não consegue generalizar sua política para novos dados de mercado (períodos de teste). Ele **sofre de *overfitting***, memorizando os padrões do período de treino, mas falhando em se adaptar a novos regimes de mercado.

2.  **Ineficiência do Q-Learning Tabular:** O espaço de estados do mercado financeiro é vasto demais. Discretizar os preços e indicadores em uma Q-Table gigante não é uma abordagem escalável ou eficaz.

3.  **Resultado Final:** Mesmo após a otimização de hiperparâmetros (Grid Search), o **Sharpe Ratio médio permaneceu próximo de 0.0**, indicando que a estratégia do agente não oferece retorno ajustado ao risco superior ao do mercado (ou aleatório).

A validação rigorosa foi essencial: sem a validação temporal, um simples teste em um período de treino favorável poderia dar a **falsa impressão de sucesso**.

## 💡 Próximos Passos
A dificuldade encontrada não invalida o uso de RL para trading, mas sim da abordagem *tabular*. Os próximos passos lógicos seriam:

* **Deep Q-Networks (DQN):** Substituir a Q-Table por uma Rede Neural. Isso permite que o agente lide com estados contínuos e encontre padrões complexos, generalizando melhor o aprendizado.
* **Engenharia de Recompensa:** Testar funções de recompensa mais sofisticadas (ex: Sharpe Ratio diferencial) em vez de apenas o retorno diário.

## 🛠️ Stack Tecnológica
* **Linguagem:** Python
* **Dados:** yfinance (Coleta de dados de mercado)
* **Análise:** Pandas, NumPy, Matplotlib
* **Ambiente de RL:** Gymnasium (Custom Env)
* **Modelagem:** Scikit-learn (StandardScaler)

## 👥 Autores
* Erika Koyanagui
* Fabio Asnis Campos da Silva
* Lucas Huber Pissaia
* Matheus Raeski
