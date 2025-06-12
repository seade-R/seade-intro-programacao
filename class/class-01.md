#  Aula 1 - Manipulação de dados no R e introdução

## Objetivos Gerais

Neste primeiro encontro vamos conhecer o ambiente de programação de R e o software RStudio, bem como seu uso no servidor RStudio do SEADE. A seguir, começaremos o estudo da linguagem R pelo seu uso mais comum: a manipulação de dados. Vamos nos habituar à linguagem da melhor maneira possível, usando-a.

## Roteiro

1- Nos encontraremos em uma sala virtual (pelo Teams) às 13h30. Faremos uma breve apresentação das e dos participantes e conversaremos sobre o funcionamento do curso. Veremos como utilizar o RStudio no servidor da Fundação SEADE.

2- Assista ao breve vídeo sobre a interface RStudio: https://www.youtube.com/watch?v=7yhw_xYWqlU

3- A primeira experiência de programação no curso será com o [Tutorial 1](/tutorial/tutorial-01.md). Diferentemente de cursos convencionais de R, nos quais se apresenta primeiro R como linguagem de programação, vamos começar com aquilo que é distintivo em R, que é sua vocação para análise de dados. Em particular, vamos começar trabalhando com _data frames_, que são as matrizes de dados, e a gramática do pacote _dplyr_, utilizando como exemplo os dados da pesquisa SEADE Investimentos.

4 - No [Tutorial 2](/tutorial/tutorial-02.md) veremos mais alguns aspectos básicos de manipulação de dados com _dplyr_. Utilizaremos dados de casos e óbitos de COVID-19 nos municípios do Estado de São Paulo.

5 - Finalizaremos o primeiro dia de curso com um encontro às 17h00 para conversamos sobre a forma do curso e sobre o conteúdo dos tutoriais, além de discutirmos ajustes no programa.

Lembre-se: é mais importante aprender com calma do que concluir os tutoriais. Siga seu ritmo.

## Opcional

**Uma forma eficiente de aprender línguas:** aprender uma nova língua (inclusive linguagens de programação!) pode parecer uma tarefa difícil ou intimidadora no começo. Entretanto, usando a lógica da eficiência de Pareto, você pode rapidamente dominar o essencial. Para entender melhor essa abordagem, confira os vídeos [Fastest Way to Learn ANY Programming Language: 80-20 rule](https://www.youtube.com/watch?v=14BL_FwQCpM&si=CcEPRASj438IfAXi) e [Learn Any Programming Language In 3 Hours!](https://www.youtube.com/watch?v=7SKVQyA4TjU).  

Lembre-se: o fundamental é dominar bem o núcleo básico da língua, usando-o como base para expandir gradualmente seu conhecimento através da prática contínua.

_Para pensar_: caso você queira, no final da aula, veja este [tutorial](/tutorial/tutorial-atribuicao-variavel.md) para entender por que preferimos em R o símbolo '<-' para a criação de objetos, ao invés de '='. 

## Dica de Leitura

Um dos principais livros para aprender a programar em R é [_Hands-on Programming With R_](https://rstudio-education.github.io/hopr/), de Garret Grolemund. Após o encontro de hoje, você pode ler o Prefácio e os Capítulos 1 e 2, que compõem a Part I.

Durante a leitura, não se preocupe se você sentir dificuldade em algum tópico. Os primeiros tutoriais que faremos no curso lidam com a maior parte dos assuntos do livro.

Nosso curso será inteiramente feito via Github. Caso não esteja acostumado com plataformas de versionamento e compartilhamento de código, recomendo as seguintes leituras adicionais: [Documentação de introdução ao GitHub](https://docs.github.com/pt/get-started) e [Noções Básicas de Github](https://docs.github.com/pt/get-started/git-basics). Se preferir algo mais didático, recomendo o seguinte [curso da Microsoft](https://learn.microsoft.com/pt-br/training/modules/intro-to-git/0-introduction).


## Desafio: Manipulação de dados com dplyr


O objetivo desses desafios semanais é permitir que você coloque em prática e consolide os conteúdos apresentados em cada aula. A cada semana teremos um novo desafio, que deverá ser realizado e entregue até o nosso próximo encontro.

Por ser um curso prático, recomendamos fortemente que você utilize bases de dados reais, que façam parte da sua rotina profissional. Dessa forma, será possível aplicar diretamente o que foi aprendido em situações concretas, facilitando a absorção e a relevância do aprendizado.

Para esta primeira semana, sugerimos que você já comece a refletir sobre quais conjuntos de dados utilizará ao longo do curso, garantindo uma experiência prática mais proveitosa e consistente.

Lembre-se de que todos os problemas que surgirem durante os desafios são bem-vindos e importantes para nosso processo de aprendizado. Não hesite em trazer suas dúvidas, dificuldades ou questões específicas relacionadas ao uso dos seus dados para os nossos encontros. Esses momentos de discussão coletiva têm o objetivo de auxiliar diretamente no desenvolvimento dos seus projetos, esclarecendo dúvidas práticas e promovendo a troca de experiências entre colegas.

Para o desafio dessa semana, revisaremos os principais verbos do _dplyr_.

### Atividades Principais

Escolha uma base de dados utilizada em seu ambiente profissional e realize as seguintes atividades utilizando o pacote dplyr:

  - Renomeie variáveis para tornar seus nomes mais claros e informativos.
  - Crie pelo menos duas variáveis derivadas usando mutate().
  - Utilize filter() para criar subconjuntos específicos dos dados.
  - Produza uma tabela com agrupamentos e resumos usando group_by() e summarise().    
  - Ordene os resultados com a função arrange().
    
### Atividade Opcional

Combine pelo menos quatro funções diferentes do dplyr em uma única operação encadeada utilizando o operador pipe (%>%). Descreva detalhadamente no script cada etapa realizada e o motivo das escolhas feitas.

### Documentação

Utilize um arquivo em formato .R (ou, opcionalmente, .Rmd) e inclua comentários detalhados descrevendo cada passo do código.

