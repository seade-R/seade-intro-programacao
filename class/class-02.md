# Aula 2 - Do básico ao intermediário

## Objetivos Gerais

No primeiro encontro trabalhamos com a gramática do pacote *dplyr* para manipular data frames. Hoje vamos dar alguns passos para trás e aprender os usos mais elementares da linguagem R.

## Roteiro

<!--- 0 - Às 13h30 faremos um encontro virtual (via Teams) para falarmos dos objetivos de hoje e tirarmos dúvidas que possam ter ficado da aula anterior. --->

0 -  Começaremos discutindo os desafios da aula passada e esclarecendo dúvidas que possam ter permanecido.

1 - Se deixou algum tutorial inacabado do encontro anterior, comece por ele. Caso contrário, prossiga.

2 - Assista ao breve vídeo sobre como gerenciar projetos no RStudio: <https://www.youtube.com/watch?v=ukU9iNkPX60&t=3s>. Uma boa dica de leitura sobre o assunto (opcional e para o final da manhã de hoje: <https://curso-r.github.io/zen-do-r/rproj-dir.html#diret%C3%B3rio-de-trabalho>).

<!--- 3 - Em seguida, os dois materiais a seguir sobre boas práticas na redação de códigos: (i) a [seção 1.5](http://electionsbr.com/livro/introducao-ao-r.html#criando-objetos) do livro *Usando R: um guia para cientistas políticos*; e o [capítulo 4](https://r4ds.had.co.nz/workflow-basics.html) do livro *R for Data Science*. --->

3 - Em seguida, leia o material a seguir sobre boas práticas na redação de códigos: [capítulo 4](https://r4ds.had.co.nz/workflow-basics.html) do livro *R for Data Science*.

4 - A seguir vá para o [Tutorial 3](/tutorial/tutorial-03.md). Nele você aprenderá um pouco mais sobre data frames, vetores e operações matemáticas em R. Este tutorial trata dos fundamentos da manipulação de dados da gramática do pacote `base`, que veremos em paralelo à do `dplyr`.

<!--- 5 - Vamos fazer um *check point* do aprendizado até aqui e tratar das dúvidas coletivamente no meio do nosso horário, na sala do Teams. Se você não tiver dúvidas e preferir seguir trabalhando de forma autônoma, vá em frente!--->

5 - Em seguida, comece o [Tutorial 4](/tutorial/tutorial-04.md) sobre operadores relacionais e lógicos e controle de fluxo.

6 - Assista ao [breve vídeo](https://www.youtube.com/watch?v=IRjLs5P5A9c) sobre como enviar e fazer download de arquivos no servidor de RStudio no qual estamos trabalhando. Esse passo não é necessário caso você esteja rodando rodando o RStudio em sua máquina local.

<!--- 8 - Como no 1º encontro, ao final, faremos uma nova conversa coletiva na sala do Teams. --->

## Dica de Leitura

Uma boa leitura para acompanhar este pedaço do curso são as partes II e III do livro [*Hands-on Programming With R*](https://rstudio-education.github.io/hopr/), de Garret Grolemund.

Em seguida, para aprender boas práticas na redação de códigos é sugerido o [capítulo 4](https://r4ds.had.co.nz/workflow-basics.html) do livro *R for Data Science*, caso ainda não tenha terminado durante a aula.

Caso queira praticar mais os condicionais que vimos durante a aula, recomendo a leitura dos seguintes textos:
   
   - Para aprofundar em If, If else e ˋìfelse()ˋ veja o [seguinte tutorial](https://analisereal.com/2016/03/02/programacao-no-r-if-if-else-e-ifelse-2/) (em português).
   - Para aprofundar em loops ˋfor{}ˋ e discussão de vetorização em R, veja o [seguinte tutorial](https://analisereal.com/2016/02/27/loops-no-r-usando-o-for-2/) (em português) ou [Capítulo 11.3](https://rstudio-education.github.io/hopr/loops.html#for-loops) do Hands of Programming with R (em inglês).
   - Para loops ˋwhile{}ˋ veja o [Capítulo 11.4](https://rstudio-education.github.io/hopr/loops.html#while-loops) do Hands on Programming with R (em inglês).
			- Para expandir o conhecimento em funções, veja o [seguinte tutorial](https://analisereal.com/2015/04/04/funcoes-definicao-argumentos-e-operadores-binarios/) (em português) ou o  [Capítulo 25](https://r4ds.hadley.nz/functions.html) do R for Data Science (em inglês).


## Desafio: Aplicação de lógica condicional


### Atividade Principal

   - Construa manualmente um conjunto simplificado de dados relevante ao seu trabalho.
   - Utilize operadores relacionais e condicionais (if/else) para criar uma variável categórica baseada em critérios definidos por você (por exemplo: níveis de prioridade, faixas de valores etc.).
   - Produza ao menos duas tabelas com a função básica table() para apresentar a relação da nova variável com variáveis originais.

### Atividade Opcional

Implemente uma estrutura de repetição simples (for) combinada com condicionais (if/else) para automatizar a criação ou transformação de variáveis no seu conjunto de dados. Documente claramente cada etapa realizada.

### Documentação
Crie um arquivo em formato .R (ou, opcionalmente, .Rmd), com comentários detalhados sobre todas as decisões e etapas realizadas.


