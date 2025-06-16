#  Aula 4 - ggplot2: Gramática de gráficos

## Objetivos Gerais

Na parte da tarde vamos seguir com a gramática do `dplyr` para transformar os dados e produzir tabelas. Utilizaremos um pacote chamado `janitor`. No meio do caminho, aprenderemos um pouco sobre `factors`, que é a estrutura de dados utilizada em R para variáveis categóricas, e pode ser muito útil para o dia-a-dia de trabalho com dados na gestão pública.

A seguir, aprenderemos uma nova 'gramática' de R: a gramática de gráficos do pacote `ggplot2`. Este pacote é certamente uma das razões pelas quais R se tornou popular (e uma das vantagens de R sobre Python). Trabalharemos algumas das inúmeros 'geometrias' do pacote, nos preocupando menos em colecionar funções e mais em compreender a estrutura dessa gramática.

## Roteiro

0 - Retomaremos a segunda parte da Aula 3, começando pela análise dos desafios e esclarecimento de dúvidas.

1 - Se deixou algum tutorial inacabado até aqui, comece por ele. Caso contrário, prossiga.

2 - Comece pelo tutorial no qual parou: [Tutorial 1](/tutorial/tutorial-01.md), [Tutorial 2](/tutorial/tutorial-02.md), [Tutorial 3](/tutorial/tutorial-03.md), [Tutorial 4](/tutorial/tutorial-04.md), [Tutorial 5](/tutorial/tutorial-05.md) e [Tutorial 6](/tutorial/tutorial-06.md)

3 - Na sequência, vá para o [Tutorial 7](/tutorial/tutorial-07.md), que apresenta o pacote `janitor` para limpeza de dados e construção de tabelas, simples e de duas entradas.

4 - Finalmente vá para o [Tutorial 8](/tutorial/tutorial-08.md) que introduz uma das especialidades de R: a gramática de gráficos do pacote `ggplot2`.

## Opcional

5 - Em algum momento do curso, ou após, seria interessante ter uma exposição à gramática básica do R. Se, e somente se, você estiver confortável com a gramática do `dplyr` e houver tempo, faça (ou pelo menos comece a fazer) o [Tutorial 9](/tutorial/tutorial-09.md).

## Dicas de Leitura

- Voltando à gramática de gráficos, não trabalhamos, propositalmente, com todos os elementos visuais secundários às geometrias: títulos, legenda, eixos, etc. Você pode aprender um pouco mais explorando os exemplos do livro [R Graphics Cookbook](https://r-graphics.org/): gráficos de barras no [Capítulo 3](https://r-graphics.org/chapter-bar-graph); gráficos de linha no [Capítulo 4](https://r-graphics.org/chapter-line-graph); ou gráficos de dispersão no [Capítulo 5](https://r-graphics.org/chapter-scatter). Finalmente, se tiver tempo, explore a [página do R Graphics Cookbook](http://www.cookbook-r.com/Graphs/) (que é diferente do livro).

- Uma leitura alternativa e introdutória sobre `ggplot2` é o [Capítulo 3](https://r4ds.had.co.nz/data-visualisation.html) do livro 'R for Data Science'. Se o Tutorial 8 não foi suficiente para você aprender, recomendo esta leitura.

- O livro 'R for Data Science' tem um excelente capítulo sobre Factors ([Capítulo 15](https://r4ds.had.co.nz/factors.html)).


## Desafio: Visualizações com dados reais


### Atividade Principal

  - Utilize uma base própria e produza ao menos três gráficos diferentes usando diferentes geometrias do ggplot2 (`geom_bar()`, `geom_histogram()`, `geom_density()`, `geom_point()`, etc.).
  -  Customize cada visualização (cores, títulos, eixos) para facilitar sua compreensão.
  -  Para pelo menos um gráfico, combine previamente a manipulação dos dados com funções do pacote `dplyr` para melhorar a clareza do resultado final.

### Atividade Opcional

Crie um gráfico que utilize múltiplas camadas gráficas sobrepostas (diferentes geometrias), ajustando manualmente escalas, cores e temas (`theme()`, `scale_*`). Explique detalhadamente cada decisão tomada no processo.


### Documentação
Crie um arquivo .R (ou opcionalmente .Rmd), documentando detalhadamente as decisões gráficas e as manipulações prévias realizadas.
