#  Aula 3 - Tidyverse para manipulação de dados

## Objetivos Gerais

No primeiro encontro trabalhamos com a gramática do pacote `dplyr` para manipular data frames. Nos distanciamos dela na aula passada para aprendermos um pouco sobre os aspectos elementares da linguagem R. Voltaremos ao uso do `dplyr` hoje, começando pelas diversas formas de se importar dados em R.


## Roteiro

0 - Às 08h30 faremos um encontro virtual (Teams) de 30 a 45 minutos para falarmos dos objetivos de hoje e tirarmos dúvidas sobre a aula passada. Hoje teremos duas aulas ao longo do dia. Esta, pela manhã, e seguiremos com a Aula 3 (parte 2) na parte da tarde. Tome fôlego e vamos juntos!

1 - Se deixou algum tutorial inacabado do encontro anterior, comece a ele. Caso contrário, prossiga.

2 - Comece pelo [Tutorial 5](/tutorial/tutorial-05.md) que trata da importação de dados em R.

3 - A seguir vá para o [Tutorial 6](/tutorial/tutorial-06.md), que retoma o exemplo da Pesquisa SEADE Investimentos e discute um pouco sobre a produção de tabelas na gramática do `dplyr`. 

4 - Se houver tempo, podemos começar o [Tutorial 7](/tutorial/tutorial-07.md), que apresenta o pacote `janitor` para confecção de tabelas de frequência em R de um jeito simples.

5 - Faremos uma pausa às 12h30, para o almoço. Retornaremos às 13h30 para a segunda parte da aula.

## Dica de Leitura

Os [Capítulo 5](https://r4ds.had.co.nz/transform.html), [Capítulo 11](https://r4ds.had.co.nz/data-import.html) e [Capítulo 12](https://r4ds.had.co.nz/tidy-data.html) de R for Data Science, de Wickham e Grolemund, são excelentes leituras para acompanhar essa aula.


## Desafio: Importação, limpeza e análise exploratória básica

### Atividades Principais

- Escolha uma base real utilizada frequentemente no seu trabalho, armazenada em formato diferente de CSV (por exemplo, Excel ou SPSS).
- Importe os dados utilizando as funções apropriadas (`read_excel()`, `read_sav()`).
- Com o pacote `janitor`, realize:
    - Limpeza dos nomes das variáveis.
    - Criação de tabelas de frequência simples e cruzadas para análise inicial.
    - Transformação de pelo menos uma variável textual em fator ordenado (`factor()` e `recode()`).


### Atividade Opcional

Crie uma função personalizada que automatize o processo básico de importação e limpeza dos dados (nomes e fatores) para futuras bases similares. Documente cuidadosamente cada elemento dessa função.

### Documentação

Utilize um arquivo .R (ou opcionalmente .Rmd), incluindo comentários detalhados explicando cada passo.
