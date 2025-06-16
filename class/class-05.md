# Aula 5 - Bases relacionais e R + Power BI

## Objetivos Gerais

Neste último encontro veremos como combinar data frames de diferentes origens que se relacionam por meio de uma ou mais variáveis chave. Utilizaremos novos verbos do dplyr, de sufixo `_join` para trabalhar com bases de dados relacionais.

A seguir, veremos em pequenos vídeos como integrar R e Power BI. Finalmente, para quem estiver confortável com a linguagem, convém aprender um pouco da 'gramática' básica de R.

## Roteiro

0 - Às 8h30 faremos um encontro virtual (Teams) para falarmos dos objetivos de hoje e tirarmos dúvidas sobre a aula passada.

1 - Se deixou algum tutorial inacabado do encontro anterior, comece por ele. Caso contrário, prossiga.

2 - Comece pelo tutorial no qual parou: [Tutorial 1](/tutorial/tutorial-01.md), [Tutorial 2](/tutorial/tutorial-02.md), [Tutorial 3](/tutorial/tutorial-03.md), [Tutorial 4](/tutorial/tutorial-04.md), [Tutorial 5](/tutorial/tutorial-05.md), [Tutorial 6](/tutorial/tutorial-06.md), [Tutorial 7](/tutorial/tutorial-07.md), [Tutorial 8](/tutorial/tutorial-08.md) e [Tutorial 9](/tutorial/tutorial-09.md)

3 - Na sequência, vá para o [Tutorial 10](/tutorial/tutorial-10.md) que apresenta como trabalhar com bases de dados relacionais utilizando os verbos do `dplyr`.

4 - O [Tutorial 11](/tutorial/tutorial-11.md) não traz nenhuma novidade em relação ao anterior, mas utiliza os verbos `_join` para um tipo de combinação de dados bastante comum: tabelas de indivíduos e domicílios em uma pesquisa amostral domiciliar, a TICDOM, realizada pelo CETIC-NIC.

## Opcional

Os tutoriais a seguir são opcionais e foram preparados para quem quiser ir além dos conteúdos principais do curso:

5 - [Tutorial 12](/tutorial/tutorial-12.md): Integração com Power BI.
Neste tutorial, acompanhado de vídeos curtos, você verá as formas mais simples de integrar o R ao Power BI, permitindo importar, transformar e visualizar dados diretamente no Power BI usando scripts em R., opcional, apresenta, com acompanhamento de vídeos curtos, as maneiras mais simples de como integrar R e Power BI.

6 - 

7 - [Tutorial 14](/tutorial/tutorial-14.md): Relatórios Reproduzíveis com RMarkdown.
Aprenda a automatizar relatórios combinando texto, código e dados atualizados em um único documento. O tutorial mostra como gerar análises dinâmicas, criar seções por grupo, agendar execuções automáticas e enviar relatórios por e-mail sem intervenção manual.

8 - [Tutorial 15](/tutorial/tutorial-15.md): Análise de Dados do Início ao Fim.
 Neste tutorial de encerramento, o objetivo é reunir e articular várias técnicas aprendidas ao longo do curso, mostrando o fluxo completo de análise: limpeza, organização, tratamento de valores ausentes, identificação de problemas de estrutura nos dados e primeiros passos em análise exploratória e modelagem. O foco está em como pensar o processo e combinar diferentes etapas para extrair informações relevantes de conjuntos de dados complexos.

 
## Instalação do R e RStudio em sua máquina local

Trabalhamos, ao longo de todo o curso, com RStudio instalado no servidor do SEADE. Agora, para que você possa seguir usando R em sua vida, você vai precisar instalar a ferramenta na sua máquina local, seu desktop pessoal ou notebook.

O R e RStudio funcionam juntos, portanto, precisamos instalar os dois. Pense que o R é o motor e RStudio é a lataria do carro (a parte visível). Ambos são inteiramente gratuitos e já vêm com distribuições compiladas para Windows, Mac e Linux.

A instalação é bastante fácil e, em geral, basta seguir as instruções da tela.

1.  Para instalar o R, baixe a versão adequada para seu computador em: <https://cloud.r-project.org/>

2.  Para instalar o RStudio, baixe a versão adequada para seu computador em: <https://www.rstudio.com/products/rstudio/download/>

3.  Uma vez instalados o R e o RStudio, basta abrir o RStudio e começar a criar seus projetos. Lembre-se que os pacotes que usamos ao longo do curso vão precisar ser instalados em sua máquina. Esse processo vai acontecer uma única vez e, nas próximas vezes que for usar o R em sua máquina, basta carregas as bibliotecas.

## Avaliação do curso

Criamos um formulário para que você possa avaliar a qualidade do curso oferecido. Assim, esperamos aprimorar os conteúdos e metodologia para as próximas edições. Acesse e preencha o formulário [aqui](https://forms.gle/DRwwt25QohxD4p596).

## Dicas de Leitura

O livro 'R for Data Science' tem excelente capítulo dados relacionais ([Capítulo 13](https://r4ds.had.co.nz/relational-data.html)).

Para a integração entre R e Power BI, convém ler a documentação da Microsoft: (1) [Executar scripts do R no Power BI Desktop](https://docs.microsoft.com/pt-br/power-bi/connect-data/desktop-r-scripts); (2) [Uso do R no Editor do Power Query](https://docs.microsoft.com/pt-br/power-bi/connect-data/desktop-r-in-query-editor); (3) [Criar visuais do Power BI usando o R](https://docs.microsoft.com/pt-br/power-bi/create-reports/desktop-r-visuals).


## Desafio: Integração e análise de bases relacionais


### Atividade Principal

  - Escolha ao menos duas bases que possam ser conectadas através de variáveis-chave.
  - Realize diferentes tipos de junções (inner_join, left_join, right_join), indicando qual das opções foi a mais adequada e justifique claramente sua escolha.
  - Visualize as diferenças entre as junções utilizando tabelas ou gráficos simples (com funções dos pacotes janitor ou ggplot2).


### Atividade Opcional 

  - Exporte e visualize os resultados obtidos no Power BI usando scripts do R.
  - Realize uma junção de bases utilizando múltiplas variáveis-chave simultaneamente. Descreva claramente no script o motivo dessa abordagem e os desafios técnicos encontrados durante o processo.


### Documentação

Apresente claramente cada etapa da análise em um arquivo .R (ou opcionalmente .Rmd), detalhando todas as decisões tomadas e os resultados obtidos.

