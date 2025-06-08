## Objetivo do Projeto

O analista de dados foi incumbido de compreender os padrões de vendas de uma plataforma de e-commerce e prever o faturamento futuro a partir de dados históricos. Para isso, foram definidas as seguintes questões principais:

- Identificar os meses de maior receita.  
- Verificar se determinadas categorias apresentam desempenho estatisticamente diferente.  
- Avaliar a capacidade de estimar o valor de venda a partir de variáveis como quantidade, categoria de produto, *fulfilment* e nível de serviço de frete.  

O objetivo final é implementar um modelo preditivo capaz de receber informações de mês, dia da semana, quantidade e características do pedido para gerar uma previsão precisa do valor de venda.

---

## Coleta & Preparação dos Dados

1. **Origem dos dados**  
   - Dataset público disponível no Kaggle (“Unlock Profits with e-Commerce Sales Data”).  
   - Carregado em um **DataFrame** do Pandas, estrutura tabular que facilita a manipulação de linhas e colunas em Python.

2. **Limpeza de dados**  
   - **Remoção de valores ausentes essenciais**: exclusão de registros sem `Amount` (valor da venda) ou `currency` (moeda), variáveis-alvo do modelo.  
   - **Preenchimento de campos de envio**: colunas `ship-city`, `ship-state` e `ship-country` receberam o rótulo “Desconhecido” quando vazias; códigos postais nulos convertidos para 0.

3. **Conversão e engenharia de data**  
   - Coluna `Date` convertida para o tipo `datetime` do Pandas, permitindo extração de variáveis temporais:  
     - `month`: mês do pedido (1 a 12).  
     - `weekday`: dia da semana (0 = segunda-feira, …, 6 = domingo).

4. **Tratamento de outliers**  
   - Filtragem de valores de `Amount` acima do **percentil 99**, eliminando os 1 % mais extremos.  
   - **Percentil**: medida estatística que indica o valor abaixo do qual se encontra uma dada porcentagem dos dados.

---

## Análise Exploratória & Estatística

1. **Estatísticas descritivas**  
   - Uso de `df.describe()` para obter média, desvio padrão, mínimo e máximo de variáveis numéricas.  
   - **Desvio padrão**: quantifica a dispersão dos dados em torno da média.

2. **Tendência mensal**  
   - Gráfico de linha mostrou concentração de vendas nos meses 4, 5 e 6 de 2022, sugerindo sazonalidade ou janela de análise restrita.

3. **Distribuição do valor de venda**  
   - Histograma com curva de densidade evidenciou **assimetria positiva**: a maioria das vendas abaixo de ₹ 1 000, com cauda longa de valores elevados.

4. **Boxplot por categoria**  
   - Demonstrou diferentes amplitudes de variabilidade entre categorias, com outliers mais frequentes em “Set” e “Western Dress”.  
   - **Boxplot**: gráfico que mostra quartis e valores extremos, facilitando a identificação de dispersão e outliers.

5. **Teste ANOVA**  
   - Aplicação de **Análise de Variância (ANOVA)** para comparar médias de `Amount` entre categorias de produto.  
   - **P-valor** < 0,05 indica diferença estatisticamente significativa entre as médias.

---

## Modelagem & Avaliação de Modelos

1. **Seleção de variáveis**  
   - **Numéricas**: `month`, `weekday`, `Qty` (quantidade de unidades).  
   - **Categóricas**: `Category`, *Fulfilment* (Amazon ou comerciante) e *ship-service-level* (nível de prioridade de frete).

2. **Codificação one-hot**  
   - Conversão de cada categoria em colunas binárias (0/1) para compatibilidade com algoritmos de aprendizado de máquina.

3. **Modelos testados e métricas**  

| Modelo               | MAE    | RMSE   | R²   |
|----------------------|--------|--------|------|
| Regressão Linear     | 151,27 | 211,41 | 0,43 |
| Random Forest        | 147,99 | 207,34 | 0,46 |
| RF sem outliers      | 141,34 | 195,66 | 0,46 |
| Gradient Boosting    | 141,37 | 195,55 | 0,46 |

4. **Definições de métricas**  
   - **MAE** (Mean Absolute Error): erro médio absoluto, em moeda local.  
   - **RMSE** (Root Mean Squared Error): raiz do erro quadrático médio, penaliza discrepâncias maiores.  
   - **R²** (Coeficiente de Determinação): proporção da variância explicada pelo modelo.

---

## Importância das Variáveis (Feature Importance)

As dez variáveis mais relevantes no modelo **Random Forest sem outliers** foram ordenadas por importância, refletindo a contribuição de cada preditor para reduzir o erro:

1. `Category_kurta`  
2. `Category_Top`  
3. `Fulfilment_Merchant`  
4. `Category_Set`  
5. `Category_Bottom`  
6. `month`  
7. `Qty`  
8. `ship-service-level_Standard`  
9. `Category_Western Dress`  
10. `weekday`

- **Feature Importance**: soma da redução de impureza (ex.: redução de variância) proporcionada por cada variável em todas as árvores do ensemble.

---

## Conclusões & Próximos Passos

1. **Melhor modelo**  
   - O **Random Forest sem outliers** apresentou o melhor equilíbrio entre erro e explicação de variância (MAE = 141,34; RMSE = 195,66; R² = 0,46).

2. **Limitações**  
   - Apenas 46 % da variância explicada, indicando oportunidade de melhoria.  
   - Dados concentrados em três meses limitam a capacidade de generalização.

3. **Recomendações**  
   - Incluir variáveis adicionais: indicadores de promoção, faixa de preço e agrupamento geográfico (`ship-country`).  
   - Aplicar engenharia de features avançada, por exemplo codificação de interações entre categoria e *fulfilment*.  
   - Testar algoritmos de boosting mais sofisticados (XGBoost, LightGBM) e otimizar hiperparâmetros via **Grid Search** (busca exaustiva em combinação de parâmetros) ou **Random Search** (busca aleatória em espaço de hiperparâmetros).  
   - Estabelecer pipeline de monitoramento em produção com retraining periódico à medida que novos dados sejam coletados.
