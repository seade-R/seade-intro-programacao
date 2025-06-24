# Integrando R com Bancos de Dados SQL

Neste tutorial, aprenderemos a conectar o R com bancos de dados SQL, uma habilidade essencial para trabalhar com grandes volumes de dados ou acessar informações armazenadas em sistemas corporativos.

Abordaremos três tecnologias principais:

  1. **SQLite**: banco de dados embarcado, ideal para desenvolvimento e aprendizado
  2. **MySQL**: amplamente utilizado em ambientes corporativos
  3. **DuckDB**: banco de dados analítico otimizado para ciência de dados

   
## Instalação e configuração

```r
# Instalando os pacotes necessários
install.packages(c("DBI", "RSQLite", "RMySQL", "duckdb", "duckplyr", "nycflights13"))

# Carregando as bibliotecas
library(tidyverse)
library(DBI)        # Interface padrão para bancos de dados
library(RSQLite)    # Conectividade com SQLite
library(RMySQL)     # Conectividade com MySQL
library(duckdb)     # Banco de dados analítico DuckDB
library(duckplyr)   # dplyr otimizado com DuckDB
```

## SQLite: Banco de dados embarcado

SQLite é uma excelente opção para começar por não requerer instalação de servidor. O banco de dados é armazenado em um único arquivo, facilitando o compartilhamento e backup.

### Criando um banco de dados

```r
# Carregando dados de exemplo
edu_rendimento <- read_csv2(
  "https://repositorio.seade.gov.br/dataset/1532141b-bff2-4e23-84a7-519a635a5d3b/resource/6d7d84a7-9e74-43dc-bea9-231eec2179a2/download/educacao_rendimento.csv",
  locale = locale(encoding = "Latin1")
)

# Limpeza dos dados
edu_rendimento <- edu_rendimento %>% 
  janitor::clean_names() %>% 
  mutate(
    tx_aprovacao = as.numeric(str_replace(tx_aprovacao, ",", ".")),
    tx_reprovacao = as.numeric(str_replace(tx_reprovacao, ",", ".")),
    tx_abandono = as.numeric(str_replace(tx_abandono, ",", "."))
  )

# Estabelecendo conexão com SQLite
minha_conexao_sqlite <- dbConnect(RSQLite::SQLite(), "educacao_sp.db")

# Verificando a conexão
minha_conexao_sqlite
```

O `dbConnect()` cria uma conexão com o banco de dados. No caso do SQLite, se o arquivo 'educacao_sp.db' não existir, ele será criado automaticamente. Essa conexão é como uma 'ponte' entre o R e o banco de dados - todas as operações passarão por ela.

#### Pausa para estilo de escrita

Para melhor compreensão, vamos analisar diferentes implementações da mesma operação usando dplyr.

Originalmente, fizemos a operação "de dentro para fora", colocando duas funções em uma mesma linha:

```r
# Limpeza dos dados
edu_rendimento <- edu_rendimento %>% 
  janitor::clean_names() %>% 
  mutate(
    tx_aprovacao = as.numeric(str_replace(tx_aprovacao, ",", ".")),
    tx_reprovacao = as.numeric(str_replace(tx_reprovacao, ",", ".")),
    tx_abandono = as.numeric(str_replace(tx_abandono, ",", "."))
  )
```

Um pouco mais verboso seria fazer:

```r
# Limpeza dos dados
edu_rendimento <- edu_rendimento %>% 
  janitor::clean_names() %>% 
  mutate(
    tx_aprovacao = str_replace(tx_aprovacao, ",", "."),
    tx_aprovacao = as.numeric(tx_aprovacao),
    tx_reprovacao = str_replace(tx_reprovacao, ",", "."),
    tx_reprovacao = as.numeric(tx_reprovacao),
    tx_abandono = str_replace(tx_abandono, ",", "."),
    tx_abandono = as.numeric(tx_abandono)
  )
```

Ou podemos ainda abusar dos _pipes_, embora eu pessoalmente não goste desse estilo de escrita:

```r
# Limpeza dos dados
edu_rendimento <- edu_rendimento %>% 
  janitor::clean_names() %>% 
  mutate(
    tx_aprovacao = str_replace(tx_aprovacao, ",", ".") %>% as.numeric(),
    tx_reprovacao = str_replace(tx_reprovacao, ",", ".") %>% as.numeric(),
    tx_abandono = str_replace(tx_abandono, ",", ".") %>% as.numeric()
  )
```

Existem métodos mais avançados para modificar múltiplas colunas simultaneamente, mas estão além do escopo deste curso. O importante agora é desenvolver confiança com a gramática básica do dplyr - dominar bem as funções fundamentais que aprendemos. Com essa base sólida, você poderá naturalmente explorar recursos mais avançados conforme sua experiência e necessidades evoluírem. Lembre-se também que não existe apenas uma forma 'correta' de escrever código: cada pessoa desenvolve seu próprio estilo, e o mais importante é que seu código seja claro, consistente e compreensível para você e para outros que possam lê-lo.

### Voltando ao SQL: Realizando operações básicas

```r
# Criando tabela e inserindo dados
dbWriteTable(minha_conexao_sqlite, "rendimento_escolar", edu_rendimento)

# Listando tabelas disponíveis
dbListTables(minha_conexao_sqlite)

# Verificando estrutura da tabela
dbListFields(minha_conexao_sqlite, "rendimento_escolar")
```

O `dbWriteTable()` cria uma nova tabela no banco de dados com a estrutura do data frame e insere todos os dados de uma vez. É como fazer um 'save' permanente dos dados em formato SQL. As funções `dbListTables()` e `dbListFields()` nos ajudam a explorar o que existe no banco.

### Executando consultas SQL

SQL (Structured Query Language) é a linguagem padrão para consultar bancos de dados. As principais cláusulas são:
- `SELECT`: escolhe quais colunas retornar
- `FROM`: indica de qual tabela buscar
- `WHERE`: filtra linhas com base em condições
- `GROUP BY`: agrupa dados para cálculos
- `ORDER BY`: ordena os resultados

```r
# Consulta simples
resultado <- dbGetQuery(minha_conexao_sqlite, "
  SELECT * 
  FROM rendimento_escolar 
  LIMIT 10
")

head(resultado)

# Agregação de dados
medias_sql <- dbGetQuery(minha_conexao_sqlite, "
  SELECT 
    dep_adm,
    escolaridade,
    AVG(tx_aprovacao) as media_aprovacao,
    COUNT(*) as n_registros
  FROM rendimento_escolar
  WHERE dep_adm IN ('Estadual', 'Municipal', 'Privada')
    AND tx_aprovacao IS NOT NULL
  GROUP BY dep_adm, escolaridade
  ORDER BY media_aprovacao DESC
")

print(medias_sql)
```

### Integração com dplyr

O pacote dbplyr permite usar a sintaxe familiar do dplyr com bancos de dados:

```r
# Criando referência à tabela (sem carregar dados)
tbl_rendimento <- tbl(minha_conexao_sqlite, "rendimento_escolar")

# Usando sintaxe dplyr
resultado_dplyr <- tbl_rendimento %>% 
  filter(dep_adm %in% c("Estadual", "Municipal", "Privada")) %>% 
  group_by(dep_adm, escolaridade) %>% 
  summarise(
    media_aprovacao = mean(tx_aprovacao, na.rm = TRUE),
    n = n()
  ) %>% 
  arrange(desc(media_aprovacao))

# Coletando resultados para o R
dados_local <- resultado_dplyr %>% collect()

# Visualizando o SQL gerado
resultado_dplyr %>% show_query()
```

Importante: `tbl()` não carrega os dados na memória - cria apenas uma referência. As operações são traduzidas para SQL e executadas no banco. Só quando usamos `collect()` é que os dados são trazidos para o R. O `show_query()` revela a 'mágica' do dbplyr: ele converte automaticamente nosso código dplyr em SQL. Isso é útil para aprender SQL e para otimizar consultas complexas.

### Encerrando conexão

```r
dbDisconnect(minha_conexao_sqlite)
```

Sempre feche as conexões quando terminar para liberar recursos.

## MySQL: Integração com bancos corporativos

MySQL é amplamente utilizado em ambientes corporativos. A conexão requer informações do servidor, credenciais e nome do banco.

### Estabelecendo conexão

```r
# Conexão básica
minha_conexao_mysql <- dbConnect(
  RMySQL::MySQL(),
  host = "localhost",
  port = 3306,
  user = "seu_usuario",
  password = Sys.getenv("MYSQL_PASSWORD"),  # Usar variáveis de ambiente
  dbname = "nome_do_banco"
)
```

### Boas práticas de segurança

```r
# Configurar variável de ambiente antes de executar o R:
# export MYSQL_PASSWORD="sua_senha"

# No código, usar:
password = Sys.getenv("MYSQL_PASSWORD")
```

Nunca coloque senhas diretamente no código. Variáveis de ambiente protegem informações sensíveis.

### Operações com MySQL

```r
# Estrutura similar ao SQLite
# dbListTables(minha_conexao_mysql)
# dbGetQuery(minha_conexao_mysql, "SELECT * FROM tabela LIMIT 10")
# tbl(minha_conexao_mysql, "tabela") %>% filter(...) %>% collect()
# dbDisconnect(minha_conexao_mysql)
```

As operações são idênticas ao SQLite; a principal diferença está na conexão inicial. 


## DuckDB: Banco de dados analítico

DuckDB é otimizado para análise de dados, oferecendo performance superior para operações analíticas e capacidade de ler arquivos diretamente.

### Configuração inicial

```r
library(duckdb)

# Conexão em memória
minha_conexao_duckdb <- dbConnect(duckdb())

# Conexão persistente em arquivo
# minha_conexao_duckdb <- dbConnect(duckdb(), dbdir = "meu_banco.duckdb")
```

A conexão em memória é perfeita para análises temporárias. Use arquivo quando precisar persistir os dados entre sessões.

### Métodos de transferência de dados

```r
# Método 1: Cópia física dos dados
dbWriteTable(minha_conexao_duckdb, "edu_table", edu_rendimento)

# Método 2: Referência virtual (mais eficiente)
duckdb_register(minha_conexao_duckdb, "edu_view", edu_rendimento)

# Ambos podem ser consultados igualmente
dbGetQuery(minha_conexao_duckdb, "SELECT COUNT(*) FROM edu_table")
dbGetQuery(minha_conexao_duckdb, "SELECT COUNT(*) FROM edu_view")
```

A diferença é crucial: `dbWriteTable()` duplica os dados dentro do banco, ocupando memória adicional. Já `duckdb_register()` cria apenas um 'atalho' para o data frame original - ideal quando os dados já estão no R e queremos apenas consultá-los com SQL.

### Leitura direta de arquivos

A grande vantagem do DuckDB é que ele pode processar arquivos sem carregá-los na memória. Isso é extremamente importante para grandes volumes de dados. 

```r
# Processando CSV diretamente
resultado_csv <- dbGetQuery(minha_conexao_duckdb, "
  SELECT 
    dep_adm,
    COUNT(*) as total,
    AVG(CAST(REPLACE(tx_aprovacao, ',', '.') AS DOUBLE)) as media
  FROM read_csv_auto('https://repositorio.seade.gov.br/dataset/1532141b-bff2-4e23-84a7-519a635a5d3b/resource/6d7d84a7-9e74-43dc-bea9-231eec2179a2/download/educacao_rendimento.csv')
  WHERE dep_adm IN ('Estadual', 'Municipal')
  GROUP BY dep_adm
")

# Leitura de arquivos Parquet
# dbGetQuery(minha_conexao_duckdb, "
#   SELECT * FROM read_parquet('dados/*.parquet')
#   WHERE ano = 2023
# ")
```

O `read_csv_auto()` é uma função especial do DuckDB que lê e processa arquivos sem carregá-los no R. Isso economiza memória e tempo, especialmente com arquivos grandes. O DuckDB detecta automaticamente o delimitador, encoding e tipos de dados.

### Integração com dplyr

```r
library(nycflights13)

# Registrando data frame como view
duckdb_register(minha_conexao_duckdb, "flights", flights)

# Análise usando dplyr
analise_voos <- tbl(minha_conexao_duckdb, "flights") %>%
  filter(month == 6) %>%
  group_by(origin, dest) %>%
  summarise(
    n_voos = n(),
    atraso_medio = mean(arr_delay, na.rm = TRUE),
    .groups = "drop"
  ) %>%
  filter(n_voos > 50) %>%
  arrange(desc(atraso_medio))

# Coletando resultados
resultado <- analise_voos %>% collect()
```

### Configurações de performance

```r
# Limitando uso de memória
dbExecute(minha_conexao_duckdb, "SET memory_limit = '2GB'")

# Verificando configurações
dbGetQuery(minha_conexao_duckdb, "SELECT * FROM duckdb_settings() WHERE name = 'memory_limit'")
```

Essas configurações são úteis quando trabalhando com dados muito grandes ou em máquinas com memória limitada.

### Encerrando conexão

```r
# Removendo registros de views
duckdb_unregister(minha_conexao_duckdb, "edu_view")
duckdb_unregister(minha_conexao_duckdb, "flights")

# Fechando conexão
dbDisconnect(minha_conexao_duckdb, shutdown = TRUE)
```

## duckplyr: Otimização transparente

O pacote duckplyr acelera operações dplyr usando DuckDB internamente:

```r
library(duckplyr)

# Convertendo para formato otimizado
flights_duck <- as_duckplyr_df(flights)

# Código dplyr padrão, execução otimizada
resultado <- flights_duck %>%
  filter(!is.na(dep_delay)) %>%
  group_by(origin, dest, carrier) %>%
  summarise(
    n_voos = n(),
    atraso_medio = mean(arr_delay, na.rm = TRUE),
    distancia_total = sum(distance),
    .groups = "drop"
  ) %>%
  filter(n_voos > 365) %>%
  arrange(desc(atraso_medio))

head(resultado, 10)
```

O `as_duckplyr_df()` converte um data frame comum em um formato especial que usa DuckDB internamente para todas as operações. O código continua idêntico, mas a execução é muito mais rápida para operações como group_by e summarise.

### Comparação de performance

Com 5 milhões de linhas, a diferença de performance se torna evidente. O duckplyr usa algoritmos otimizados do DuckDB para agregações, resultando em processamento várias vezes mais rápido.

```r
# Criando dataset de teste
set.seed(42)
dados_grandes <- tibble(
  id = 1:5000000,
  categoria = sample(LETTERS, 5000000, replace = TRUE),
  valor = rnorm(5000000, 100, 20),
  grupo = sample(1:1000, 5000000, replace = TRUE)
)

# Benchmark com dplyr padrão
library(tictoc)
tic()
resultado_dplyr <- dados_grandes %>%
  group_by(categoria, grupo) %>%
  summarise(
    n = n(),
    media = mean(valor),
    total = sum(valor),
    .groups = "drop"
  )
tempo_dplyr <- toc()

# Benchmark com duckplyr
dados_duck <- as_duckplyr_df(dados_grandes)
tic()
resultado_duck <- dados_duck %>%
  group_by(categoria, grupo) %>%
  summarise(
    n = n(),
    media = mean(valor),
    total = sum(valor),
    .groups = "drop"
  )
tempo_duck <- toc()
```

## Boas práticas

### 1. Gerenciamento de conexões

```r
# Usar on.exit() em funções para garantir fechamento
processar_dados <- function() {
  con <- dbConnect(duckdb())
  on.exit(dbDisconnect(con, shutdown = TRUE))
  
  # Processamento aqui
}
```

O `on.exit()` garante que a conexão seja fechada mesmo se houver erros durante o processamento.

### 2. Eficiência de memória

```r
# Preferir referências virtuais para dados grandes
# Em vez de: dbWriteTable(con, "tabela", dados_grandes)
# Usar: duckdb_register(con, "tabela", dados_grandes)
```

### 3. Processamento direto de arquivos

```r
# Em vez de carregar e processar:
# dados <- read_csv("arquivo_grande.csv")
# resultado <- dados %>% group_by(x) %>% summarise(n = n())

# Processar diretamente:
con <- dbConnect(duckdb())
resultado <- dbGetQuery(con, "
  SELECT x, COUNT(*) as n 
  FROM read_csv_auto('arquivo_grande.csv')
  GROUP BY x
")
```

Essa abordagem é especialmente valiosa quando o arquivo é maior que a memória disponível.

## Exercícios práticos

1. **Análise de performance**: Compare o tempo de execução de agregações complexas usando dplyr tradicional, duckplyr e SQL direto no DuckDB.

2. **Processamento sem carregar**: Utilize DuckDB para analisar um arquivo CSV grande sem carregá-lo completamente na memória.

3. **Pipeline integrado**: Crie um fluxo que use SQLite para armazenamento persistente e DuckDB para análises rápidas.

## Conclusão

Cada tecnologia apresentada tem seu nicho específico:

- **SQLite** é ideal para desenvolvimento local e prototipagem
- **MySQL** atende necessidades de sistemas corporativos
- **DuckDB** oferece performance superior para análise de dados
- **duckplyr** permite otimização com mudanças mínimas no código

A escolha depende do contexto: volume de dados, infraestrutura disponível e requisitos de performance. O domínio dessas ferramentas permite trabalhar eficientemente com dados de qualquer escala.
