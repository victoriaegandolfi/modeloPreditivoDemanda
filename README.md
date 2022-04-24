# modeloPreditivoDemanda
Projeto final do curso de DS da Awari, em que se contrói um modelo preditivo para demanda de categorias de produtos.

Apresentação do projeto
Esse projeto tem o objetivo de criar um modelo preditivo para prever a demanda diária de categorias de produtos do inventário da Magazine Luiza. Os dados são referentes ao período de junho de 2016 a junho de 2017 e originalmente estavam em formato csv.

O gerenciamento de um inventário é crucial para que empresas possam controlar seu estoque e maximizar as vendas. Assim, esse case parte do problema de gerenciamento de estoque e propõe como solução um modelo preditivo da demanda dos produtos mais vendidos na empresa. Uma rede como a Magazine Luiza encontra oportunidades ao analisar seus dados e prever a demanda de seus produtos, para gereciar seu estoque e controlar seus preços.

Recomenda-se a leitura do projeto no próprio notebook, que apresenta todos os gráficos e representações visuais do processo de análise.

Com esse objetivo em mente, o projeto é dividido nos seguintes tópicos:

1. Coleta e tratamento dos dados

Os dados disponibilizados estão em formato csv e foram importados para realizar a análise. A fim de criar um modelo preditivo, é preciso limpar e tratar os dados da tabela.

Primeiramente, é preciso verificar se há dados faltantes ou nulos na base de dados. Não existem valores nulos na base de dados, então não foi necessário lidar com os dados faltantes.

Além disso, as variáveis de data foram formatadas no formato datetime.

2. Análise exploratória dos dados

A análise exploratória dos dados tem o objetivo de aprender sobre o comportamento do processo de inventário da empresa. Como mencionado, os dados são referentes ao período de junho de 2016 a junho de 2017.

Com isso em mente, a análise se divide nos seguintes tópicos:

2.1. Pedidos válidos e cancelados

A partir da análise é possível perceber que 14% dos pedidos feitos são cancelados, o que não é uma porcentagem alta ao se pensar em um e-commerce. Dos pedidos válidos, a grande maioria foi entregue ou está em rota de entrega.

2.2. Quantidades vendidas ao longo do tempo

Ao observar a quantidade de itens vendidos ao longo do tempo, é possível observar o comportamento das vendas no período de um ano.
Os gráficos abaixo mostram dois picos de venda: um ao final de novembro (provavelmente devido à black friday) e um no início de janeiro (provavelmente por causa das ofertas de início de ano.

Além disso, alguns meses possuem picos de venda na segunda semana: julho, setembro e março. Com mais informações, seria interessante buscar algum tipo de padrão sobre esses picos.

A decomposição dos dados em busca de tendência, sazonalidade e resíduo confirma que os picos em novembro e janeiro realmente são sazonais e que não há um ciclo de sazonalidade a curto prazo.

2.3. Receita ao longo do tempo

O gráfico abaixo mostra o comportamento da receita bruta e da receita líquida ao longo do tempo. Os picos de venda no final de novembro e no início de janeiro realmente geram um pico de receita nesses períodos. Além disso, nesses picos, a receita líquida é significativamente menor que a receita bruta, sinalizando que os produtos provavelmente possuem um custo considerável. 
No tópico abaixo, observa-se que esses picos de venda são referentes a venda de uma categoria específica de produtos, sobre a qual incidem mais taxas e impostos que em outras categorias. Esse comportamento pode explicar porque a diferença entre a receita bruta e a líquida nesses dois picos.

2.4. Categorias de produtos vendidos ao longo do tempo

O gráfico mostra que uma categoria de produtos claramente se sobrepõe às outras. Além disso, os picos de venda no final de novembro e no início de janeiro, em grande parte, se dão pela venda dessa categoria específica de produtos.


3. Preparação da base para modelos preditivos

Essa análise apenas contará com produtos válidos, ou seja, produtos que não foram cancelados ou detectados como fraude. Para isso, exclui-se os produtos não válidos da base de dados.

A base de dados original está agrupada por id do pedido. Para prever a demanda diária por categoria de produto, agrupa-se a base de dados por data e por categoria de produto. Em seguida, calcula-se a média diária de quantidade, preço, pis/cofins, icms, taxa de substituição e custo líquido por categoria.

Analisando as quantidades pedidas por categorias, percebe-se que algumas categorias possuem poucos produtos pedidos. Portanto, o modelo de previsão será aplicado apenas nas três maiores categorias. Para selecionar o modelo regressivo, os testes serão feitos na categoria com maior quantidade de produtos vendidos.

A fim de preparar os dados para rodar os modelos regressivos, é preciso dividir a base de dados em dados de treino (aqui, 80% da base) e dados de teste (20% dos dados).

A visualização gráfica dos dados de treino apresenta alguns indícios sobre a base de dados. Os histogramas mostram que as variáveis estão em diferentes escalas e os bloxplots mostram que a base de dados contém outliers, o que indica que será necessário realizar algum método de feature scaling que não seja sensível a outliers. Os métodos de scaling servem para impedir que uma variável que naturalmente possui números maiores que outra tenha sua influência artificialmente aumentada no modelo, enviesando as previsões.

Como método de scaling que reage bem a outliers, aqui seleciona-se o robust scaler. O robust scaler usa a distância interquartil dos dados para criar a escala padronizada, por isso, os outliers são acomodados sem enviesar a escala da variável. Vale aqui comparar os boxplots antes e depois do scaling, que mostram como as variáveis agora estão com distribuições similares, o que tende a melhorar a acurácia do modelo.

Além disso, a matriz de correlação mostra que algumas variáveis parecem ter correlação positiva umas com as outras, portanto fez-se um teste de multicolinearidade por meio de um mapa de calor e do cálculo do fiv (fator inflacionário de variância). O mapa de calor mostra que as variáveis pis e custo líquido possuem forte correlação entre si (0,84) e o fator inflacionário de variância confirma que as duas variáveis são colineares porque tem um valor de 11,83. Como padrão, define-se que um valor FIV>10 já demonstra multicolinearidade. Provavelmente, o custo líquido do produto já inclui o valor do pis/cofins e, por isso, a alta correlação entre as variáveis. Por isso, opta-se aqui por remover a variável pis/cofins do modelo.

Após separar a nova base de dados e promover o scaling, os testes já não apresentam nenhuma multicolinearidade entre as variáveis. Portanto, os modelos de regressão estão prontos para ser rodados.

4. Modelos regressivos

O modelo preditivo será baseado em modelos de regressão, já que o objetivo é prever a variável de demanda, que é contínua.

Como modelos regressivos, aqui usa-se a regressão linear múltipla e dois modelos diferentes de árvore de decisão (uma tradicional e uma com gradient boosting.

Para cada modelo, optou-se pelo modelo de validação cruzada, que permite que sejam feitos diversos teste com a base de dados de treino para verificar a acurácia do modelo. Como métrica de avaliação dos modelos, compara-se o RMSE. Essa métrica permite que se avalie a acurácia do modelo e penaliza modelos com grandes níveis de erros. Além disso, a métrica está na mesma unidade das variáveis, o que torna o modelo mais explicativo. Quanto menor o RMSE, melhor a performance do modelo.

5. Avaliação de modelos

A fim de comparar a performance dos modelos, esse tópico calcula o erro deles e verifica qual alcança o menor RMSE nos dados de treino e na previsão. O modelo com melhor performance é o de regressão linear múltipla, com RMSE de 9.64. Portanto, esse modelo será aplicado para as três categorias de produtos mais vendidas do inventário.

6. Modelo regressivo em produção

Esse tópico aplica o modelo regressivo com melhor performance para as três categorias de produtos mais vendidas do inventário.

Os modelos apresentaram menores índices de erro para as duas primeiras categorias (8,14 e 11,35). Já a terceira categoria mais vendida apresentou um RMSE maior de 84,36 no medo de regressão linear. A diferença na performance do modelo pode ser explicada pelos histogramas das categorias por preço, que mostram que as duas categorias mais vendidas tem números de quantidades vendidas mais próximos e média de preço similares. Assim, a similaridade entre as categorias pode explicar a performance similar dos modelos para as mesmas. Ainda assim, um erro médio de 84 unidades diárias não é tão alto para uma média de 543 unidades diárias vendidas. Portanto, aqui nesse estudo, se mantém esse modelo para a categoria. Porém, seria interessante construir um modelo específico para essa categoria daqui em diante.

7. Conclusão

Esse projeto teve como objetivo criar um modelo regressivo de predição da média de quantidades diárias vendidas para as três categorias que geram maior rendimento no inventário.

O modelo de regressão linear múltipla construído apresentou os menores índices de erro e consegue prever satisfatoriamente a média da demanda diária dessas categorias. Para prever a média de quantidade diária, leva em conta as seguintes variáveis: preço, custo líquido, icms e taxa de substituição.

Para a terceira categoria de produtos mais vendida, é necessário fazer um teste específico para criar um modelo de predição com melhor performance, ainda que esse possa ser usado a curto prazo como modelo preditor.

8. Referências

1. MÜLLER, Andreas C.; GUIDO, Sarah. Introduction to Machine Learning with Python: A Guide for Data Scientists. United States of America: O’Reilly Media, 2017.
2. https://aprenderdatascience.com/regressao-polinomial/
3. https://www.atoti.io/when-to-perform-a-feature-scaling/
4. https://towardsdatascience.com/all-about-feature-scaling-bcc0ad75cb35
5. https://medium.com/turing-talks/turing-talks-20-regressão-de-ridge-e-lasso-a0fc467b5629
6. https://www.analyticsvidhya.com/blog/2021/05/complete-guide-to-regularization-techniques-in-machine-learning/
7. https://nathaliatito.medium.com/scikit-learn-ou-statsmodels-avaliando-meu-modelo-de-regressão-f4c04b361fa7
8. https://www.datacamp.com/community/tutorials/xgboost-in-python?utm_source=adwords_ppc&utm_medium=cpc&utm_campaignid=14989519638&utm_adgroupid=127836677279&utm_device=c&utm_keyword=&utm_matchtype=b&utm_network=g&utm_adpostion=&utm_creative=332602034364&utm_targetid=aud-392016246653:dsa-429603003980&utm_loc_interest_ms=&utm_loc_physical_ms=1001773&gclid=EAIaIQobChMIwpe3sLWO9AIVjYTICh2uEQRuEAAYASAAEgLxmvD_BwE
9. https://www.datageeks.com.br/xgboost/
10. http://arquivo.ufv.br/saeg/saeg70.htm
11. https://towardsdatascience.com/ensemble-methods-bagging-boosting-and-stacking-c9214a10a205
12. https://quantdare.com/what-is-the-difference-between-bagging-and-boosting/
13. https://towardsdatascience.com/what-are-the-best-metrics-to-evaluate-your-regression-model-418ca481755b
14. https://medium.com/turing-talks/como-avaliar-seu-modelo-de-regressão-c2c8d73dab96
15. https://towardsdatascience.com/avoid-r-squared-to-judge-regression-model-performance-5c2bc53c8e2e
16. https://www.dataquest.io/blog/learning-curves-machine-learning/
17. https://machinelearningmastery.com/overfitting-machine-learning-models/
18. https://machinelearningmastery.com/training-validation-test-split-and-cross-validation-done-right/
19. https://medium.com/turing-talks/turing-talks-10-introdução-à-predição-a75cd61c268d
