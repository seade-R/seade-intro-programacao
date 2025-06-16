# Relatórios reproduzíveis: transformando tarefas repetitivas em código eficiente

## Apresentação

Quantas vezes você já teve que gerar o mesmo relatório mensal, apenas atualizando os dados? Copiar e colar tabelas no Word, refazer gráficos no Excel, atualizar números no texto... É um trabalho tedioso e propenso a erros. E se você pudesse apertar um botão (aaaaargh! Mas esse é um botão do bem, prometo!) e ter seu relatório pronto com os dados mais recentes?

Neste tutorial, vamos aprender a criar relatórios automatizados e reproduzíveis usando RMarkdown. A ideia é simples: você escreve o relatório uma vez, misturando texto e código R, e toda vez que os dados forem atualizados, você gera um novo relatório com um único comando. 

Vamos trabalhar com um caso prático: um relatório mensal de acompanhamento de indicadores municipais que precisa ser atualizado toda vez que o Seade disponibiliza novos dados. Mas as técnicas que aprenderemos servem para qualquer situação em que você precise gerar relatórios periódicos.

## O que é RMarkdown e por que você vai adorá-lo

RMarkdown é como um Word vitaminado para quem trabalha com dados. Você escreve texto normalmente, mas pode inserir pedaços de código R que são executados e seus resultados (tabelas, gráficos, números) aparecem automaticamente no documento final.

Vamos começar instalando o que precisamos:

```r
# Se ainda não tem o RMarkdown instalado
install.packages("rmarkdown")
install.packages("knitr")

# Nossos velhos conhecidos que usaremos no relatório
library(tidyverse)
library(janitor)
```

## Criando seu primeiro documento RMarkdown

No RStudio, vá em File > New File > R Markdown. 

Escolha "Document" e "HTML" por enquanto. Dê um título como "Relatório Mensal de Indicadores". O RStudio vai criar um documento de exemplo que podemos apagar e começar do zero.

A estrutura de um documento RMarkdown é simples:

1. **Cabeçalho YAML** (aquela parte entre `---`)
2. **Texto em Markdown** (uma linguagem simples de formatação)
3. **Chunks de código R** (pedaços de código entre ``` ` ``` )

Vamos criar nosso relatório do zero. Delete tudo e comece com:

```markdown
---
title: "Relatório Mensal de Indicadores Municipais"
author: "Seu Nome"
date: "`r format(Sys.Date(), '%B de %Y')`"
output: html_document
---
```

Viu aquele pedacinho de código R na data? É assim que misturamos código com texto. A data será sempre a atual quando você gerar o relatório!


### Pausa para reflexão


1) Modifique o formato de data na linha `date:` utilizando diferentes códigos de formatação. Comece substituindo `%B` por `%b`. O que muda na saída do documento?

Faça o mesmo teste com:
 - `%M`
 - `%m`
 - `%A`
 - `%a`

Depois, experimente acrescentar ao formato:

- `%H`
- `%I`
- `%S`
- `%p`

Observe os efeitos de cada modificação no resultado final.

2) Experimente criar outras automações no YAML.

Utilize código R inline para preencher campos automaticamente. Por exemplo:

```markdown
author: "`r Sys.info()[['user']]`"
output_file: "`r paste0('relatorio_', format(Sys.Date(), '%Y%m%d'), '.html')`"
```

Explore a possibilidade de gerar nomes de arquivos dinâmicos, preencher autor com base no sistema ou incluir informações como data e hora atual. Quais outras automações seriam úteis no seu fluxo de trabalho?


## Preparando um relatório reproduzível

Vamos criar um relatório que analisa dados de população e emprego dos municípios paulistas. Primeiro, vamos estruturar nosso documento:

````markdown
# Introdução

Este relatório apresenta os principais indicadores dos municípios paulistas 
atualizados até `r format(Sys.Date(), '%d/%m/%Y')`.

```{r setup, include=FALSE}
# Configurações iniciais
knitr::opts_chunk$set(
  echo = FALSE,    # Não mostra o código, só os resultados
  message = FALSE, # Não mostra mensagens
  warning = FALSE  # Não mostra avisos
)

# Carregando pacotes
library(tidyverse)
library(janitor)
library(knitr)     # Para tabelas bonitas
```

````

Perceba que criamos um chunk (pedaço) de código chamado "setup" com `include=FALSE`. Isso significa que ele será executado, mas não aparecerá no relatório final. É aqui que fazemos nossas configurações iniciais.
Cuidado: cada chunk deve ter um nome único, portanto evite reutilizar nomes em outros trechos do documento. Caso dois chunks tenham o mesmo nome, o RMarkdown pode gerar erros na hora da renderização ou executar apenas o primeiro, ignorando o restante.

## Buscando dados atualizados

A grande sacada de um relatório reproduzível é buscar sempre os dados mais recentes. Vamos criar um chunk para isso:

````markdown
```{r carrega-dados}
# URL dos dados - sempre atualizado pelo SEADE
url_populacao <- "https://raw.githubusercontent.com/seade-R/egesp-seade-intro-programacao/main/data/populacao_municipal.csv"

# Carregando os dados
pop_municipios <- read_csv2(url_populacao) %>% 
  clean_names()

# Pegando o ano mais recente disponível
ano_mais_recente <- max(pop_municipios$ano)

# Filtrando apenas o ano mais recente
pop_atual <- pop_municipios %>% 
  filter(ano == ano_mais_recente)
```

## Análise automática dos dados

```{r analise-geral}
# Total de municípios
total_municipios <- n_distinct(pop_atual$cod_ibge)

# População total do estado
pop_total_estado <- pop_atual %>% 
  summarise(populacao_total = sum(populacao)) %>% 
  pull(populacao_total)

# Os 5 maiores municípios
maiores_municipios <- pop_atual %>% 
  arrange(desc(populacao)) %>% 
  slice_head(n = 5) %>% 
  select(localidade, populacao)
```

````

## Escrevendo o texto do relatório com valores dinâmicos

Agora vem a parte legal. Vamos escrever o texto do relatório inserindo os valores calculados:

````markdown
## Panorama Geral

O Estado de São Paulo possui **`r total_municipios` municípios** com uma 
população total de **`r format(pop_total_estado, big.mark = ".", decimal.mark = ",")`** 
habitantes em `r ano_mais_recente`.

### Maiores Municípios

Os cinco maiores municípios do estado concentram 
`r round(sum(maiores_municipios$populacao)/pop_total_estado * 100, 1)`% 
da população total:

```{r tabela-maiores, results='asis'}
maiores_municipios %>% 
  mutate(populacao = format(populacao, big.mark = ".", decimal.mark = ",")) %>% 
  kable(col.names = c("Município", "População"),
        align = c("l", "r"))
```

````

Viu como é poderoso? O texto se atualiza automaticamente com os dados!

## Adicionando gráficos que se atualizam

Vamos criar um gráfico que sempre mostra os 10 maiores municípios:

````markdown
### Distribuição Populacional

```{r grafico-populacao, fig.width=8, fig.height=6}
pop_atual %>% 
  arrange(desc(populacao)) %>% 
  slice_head(n = 10) %>% 
  mutate(localidade = fct_reorder(localidade, populacao)) %>% 
  ggplot(aes(x = populacao, y = localidade)) +
  geom_col(fill = "steelblue") +
  scale_x_continuous(labels = scales::number_format(big.mark = ".", 
                                                   decimal.mark = ",")) +
  labs(title = paste("Os 10 maiores municípios de São Paulo em", ano_mais_recente),
       x = "População",
       y = "") +
  theme_minimal()
```

````

## Criando seções dinâmicas

E se quisermos criar uma seção para cada região administrativa? Podemos fazer loops no RMarkdown!

````markdown
## Análise por Região Administrativa

```{r analise-regioes, results='asis'}
# Primeiro, vamos simular uma variável de região (em dados reais você já teria isso)
pop_atual <- pop_atual %>% 
  mutate(regiao = case_when(
    localidade %in% c("São Paulo", "Guarulhos", "São Bernardo do Campo") ~ "Região Metropolitana",
    localidade %in% c("Campinas", "Sorocaba", "Jundiaí") ~ "Interior Próximo",
    TRUE ~ "Interior"
  ))

# Agora vamos criar uma seção para cada região
regioes <- unique(pop_atual$regiao)

for(reg in regioes) {
  cat("\n\n### ", reg, "\n\n")
  
  dados_regiao <- pop_atual %>% 
    filter(regiao == reg)
  
  cat("Esta região possui", nrow(dados_regiao), "municípios com uma população total de",
      format(sum(dados_regiao$populacao), big.mark = ".", decimal.mark = ","), 
      "habitantes.\n\n")
}
```

````

## Tornando o relatório ainda mais automático

Para não precisar abrir o RStudio toda vez, você pode criar um script R simples:

```r
# arquivo: gerar_relatorio.R

# Renderiza o relatório
rmarkdown::render(
  input = "relatorio_mensal.Rmd",
  output_file = paste0("Relatorio_", format(Sys.Date(), "%Y_%m"), ".html"),
  output_dir = "relatorios/"
)

# Mensagem de sucesso
cat("Relatório gerado com sucesso!\n")
```

## Automatizando a execução do relatório

Agora vem a cereja do bolo: fazer o relatório rodar sozinho, sem você precisar fazer nada! Vamos ver três formas de fazer isso, dependendo do seu sistema operacional.

### Preparando o script para automação

Primeiro, vamos criar um script R mais robusto que vai gerar nosso relatório:

```r
# arquivo: executar_relatorio_automatico.R

# Carrega bibliotecas necessárias
library(rmarkdown)

# Define diretórios
dir_projeto <- "C:/MeusProjetos/RelatoriosMensais"  # Ajuste para seu caminho
setwd(dir_projeto)

# Cria pasta para relatórios se não existir
if(!dir.exists("relatorios_gerados")) {
  dir.create("relatorios_gerados")
}

# Gera nome do arquivo com data
nome_arquivo <- paste0("Relatorio_Indicadores_", 
                      format(Sys.Date(), "%Y_%m_%d"), 
                      ".html")

# Log de execução
cat("Iniciando geração do relatório:", date(), "\n")

# Tenta gerar o relatório
tryCatch({
  rmarkdown::render(
    input = "relatorio_mensal.Rmd",
    output_file = nome_arquivo,
    output_dir = "relatorios_gerados/",
    quiet = FALSE
  )
  cat("Relatório gerado com sucesso:", nome_arquivo, "\n")
}, error = function(e) {
  cat("ERRO ao gerar relatório:", e$message, "\n")
})

cat("Processo finalizado:", date(), "\n")
```

### Windows: Usando o Agendador de Tarefas

No Windows, vamos usar o Agendador de Tarefas (Task Scheduler):

1. **Crie um arquivo .bat** para executar o R:

```batch
@echo off
REM arquivo: rodar_relatorio.bat

REM Caminho do R (ajuste se necessário)
"C:\Program Files\R\R-4.3.0\bin\Rscript.exe" "C:\MeusProjetos\RelatoriosMensais\executar_relatorio_automatico.R" > "C:\MeusProjetos\RelatoriosMensais\log_execucao.txt" 2>&1
```

2. **Configure no Agendador de Tarefas**:
   - Abra o Agendador de Tarefas (pesquise por "Task Scheduler")
   - Clique em "Criar Tarefa Básica"
   - Dê um nome como "Relatório Mensal Automático"
   - Escolha a frequência (diária, semanal, mensal)
   - Defina o horário de execução
   - Escolha "Iniciar um programa"
   - Procure e selecione seu arquivo `rodar_relatorio.bat`

### Linux/Mac: Usando cron

No Linux ou Mac, o cron é nosso amigo:

1. **Abra o crontab**:
```bash
crontab -e
```

2. **Adicione uma linha para executar o script**:

```bash
# Executa todo dia 1 de cada mês às 8h da manhã
0 8 1 * * /usr/local/bin/Rscript /home/usuario/relatorios/executar_relatorio_automatico.R >> /home/usuario/relatorios/log_execucao.txt 2>&1

# Ou toda segunda-feira às 9h
0 9 * * 1 /usr/local/bin/Rscript /home/usuario/relatorios/executar_relatorio_automatico.R >> /home/usuario/relatorios/log_execucao.txt 2>&1
```

A sintaxe do cron é: minuto hora dia mês dia_da_semana comando

### Enviando o relatório por e-mail automaticamente

Que tal fazer o relatório chegar na caixa de entrada de quem precisa? Vamos adicionar isso ao nosso script:

```r
# Adicione ao final do executar_relatorio_automatico.R

# Instale o pacote se ainda não tiver
# install.packages("blastula")

library(blastula)

# Configurar credenciais do email (faça isso uma vez)
# create_smtp_creds_file(
#   file = "email_creds",
#   user = "seu_email@gmail.com",
#   provider = "gmail"
# )

# Enviar email
email <- compose_email(
  body = md(paste0(
    "Olá,\n\n",
    "Segue o relatório mensal de indicadores municipais atualizado.\n\n",
    "Data de geração: ", format(Sys.Date(), "%d/%m/%Y"), "\n\n",
    "Atenciosamente,\n",
    "Sistema Automático de Relatórios"
  ))
) %>%
  add_attachment(
    file = file.path("relatorios_gerados", nome_arquivo)
  )

# Enviar
smtp_send(
  email,
  to = c("destinatario1@email.com", "destinatario2@email.com"),
  from = "seu_email@gmail.com",
  subject = paste("Relatório Mensal -", format(Sys.Date(), "%B/%Y")),
  credentials = creds_file("email_creds")
)
```

## Monitorando e debugando execuções automáticas

É importante saber se tudo está funcionando. Vamos melhorar nosso script com logs detalhados:

```r
# Função para escrever logs
escrever_log <- function(mensagem) {
  cat(format(Sys.time(), "%Y-%m-%d %H:%M:%S"), "-", mensagem, "\n",
      file = "log_relatorios.txt", append = TRUE)
}

# Use no script
escrever_log("Iniciando geração do relatório")

# Verificar se os dados estão disponíveis
if(!url.exists(url_populacao)) {
  escrever_log("ERRO: URL dos dados não está acessível")
  stop("Dados não disponíveis")
}
```

## Exercício prático

Vamos criar um relatório que você possa usar no seu trabalho:

1. Pense em um relatório que você faz periodicamente
2. Identifique:
   - Quais dados mudam?
   - Quais análises se repetem?
   - Quais gráficos/tabelas são sempre os mesmos?

3. Crie a estrutura básica:

````markdown
---
title: "Seu Relatório Aqui"
date: "`r format(Sys.Date(), '%d/%m/%Y')`"
output: html_document
---

```{r setup, include=FALSE}
knitr::opts_chunk$set(echo = FALSE, message = FALSE, warning = FALSE)
library(tidyverse)
```

# Dados Gerais

```{r carrega-dados}
# Coloque aqui o código para carregar seus dados
```

Os dados foram atualizados em `r format(Sys.Date(), '%d/%m/%Y')` e 
contêm informações sobre...

````

## Dicas finais para relatórios verdadeiramente reproduzíveis

1. **Use sempre URLs ou caminhos relativos** - nunca caminhos absolutos como "C:/Users/João/Desktop"
2. **Documente suas fontes** - inclua de onde vêm os dados
3. **Versione seu código** - use Git (falaremos disso em outro tutorial!)
4. **Teste com dados diferentes** - seu código deve funcionar mesmo quando os dados mudam
5. **Mantenha logs** - você vai agradecer quando algo der errado
6. **Teste a automação** - rode manualmente primeiro, depois automatize

## Indo além

O RMarkdown pode gerar:
- PDFs (precisa instalar LaTeX - é um pouco chato, mas vale a pena)
- Word (para quem insiste em editar depois...)
- Apresentações (sim, PowerPoint também!)
- Dashboards interativos (com flexdashboard)

## Conclusão

Parabéns! Você acaba de aprender a transformar aquela tarefa chata e repetitiva em um processo completamente automatizado. Da próxima vez que seu chefe pedir "aquele relatório de sempre", você pode responder: "Ele já está na sua caixa de entrada, toda segunda-feira às 9h".

## Dica de Leitura

O YAML oferece diversas opções de configuração, mas neste momento abordaremos apenas os elementos essenciais. Para um estudo mais aprofundado, recomenda-se a leitura do [Capítulo 9 do *R Markdown Crash Course*](https://zsmith27.github.io/rmarkdown_crash-course/lesson-4-yaml-headers.html).

Além disso, para explorar em maior profundidade as funcionalidades do R Markdown, vale consultar o [*R Markdown Cookbook*](https://bookdown.org/yihui/rmarkdown-cookbook/), que reúne uma ampla variedade de exemplos e práticas recomendadas.



