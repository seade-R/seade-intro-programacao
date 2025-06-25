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

# Verificando estrutura da tabela, similar a names() em R
dbListFields(minha_conexao_sqlite, "rendimento_escolar")
```

O `dbWriteTable()` cria uma nova tabela no banco de dados com a estrutura do data frame e insere todos os dados de uma vez. É como fazer um 'save' permanente dos dados em formato SQL. As funções `dbListTables()` e `dbListFields()` nos ajudam a explorar o que existe no banco.

### Executando consultas SQL

SQL (Structured Query Language) é a linguagem padrão para consultar bancos de dados. As principais cláusulas para consumo de dados são:
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
# NÃO RODAR - Exemplos de conexão MySQL para diferentes cenários
# Descomente e configure apenas a conexão que você precisar usar

#  CONEXÃO LOCAL (servidor na mesma máquina) 
# minha_conexao_mysql <- dbConnect(
#   RMySQL::MySQL(),
#   host = "localhost",
#   port = 3306,
#   user = "seu_usuario",
#   password = Sys.getenv("MYSQL_PASSWORD"),  # Usar variáveis de ambiente
#   dbname = "nome_do_banco"
# )

#  CONEXÃO REMOTA POR ENDEREÇO IP 
# minha_conexao_mysql <- dbConnect(
#   RMySQL::MySQL(),
#   host = "192.168.1.50",  # IP do servidor MySQL
#   port = 3306,
#   user = "seu_usuario",
#   password = Sys.getenv("MYSQL_PASSWORD"),
#   dbname = "nome_do_banco"
# )

#  CONEXÃO REMOTA POR HOSTNAME/DOMÍNIO 
# minha_conexao_mysql <- dbConnect(
#   RMySQL::MySQL(),
#   host = "mysql-server.empresa.com",  # Hostname ou FQDN
#   port = 3306,
#   user = "seu_usuario",
#   password = Sys.getenv("MYSQL_PASSWORD"),
#   dbname = "nome_do_banco"
# )

#  EXEMPLO COM SERVIDOR EM NUVEM 
# minha_conexao_cloud <- dbConnect(
#   RMySQL::MySQL(),
#   host = "mysql-instance.us-east-1.rds.amazonaws.com",  # RDS AWS
#   port = 3306,
#   user = "admin_user",
#   password = Sys.getenv("MYSQL_PASSWORD"),
#   dbname = "producao_db"
# )

#  CONEXÃO COM PORTA PERSONALIZADA 
# minha_conexao_custom <- dbConnect(
#   RMySQL::MySQL(),
#   host = "10.0.0.100",  # IP interno da rede corporativa
#   port = 3307,          # Porta personalizada
#   user = "analytics_user",
#   password = Sys.getenv("MYSQL_PASSWORD"),
#   dbname = "warehouse_db"
# )
```

### Boas práticas de segurança

A segurança das credenciais de banco de dados é fundamental em ambientes corporativos. Nunca inclua senhas diretamente no código, pois isso pode levar a vazamentos acidentais quando o código for compartilhado, versionado no Git ou visualizado por outras pessoas. As variáveis de ambiente oferecem uma camada de proteção ao manter as credenciais separadas do código-fonte.

Além disso, essa prática facilita a migração entre ambientes (desenvolvimento, teste, produção) sem necessidade de alterar o código, apenas as variáveis de ambiente. Em equipes de desenvolvimento, cada pessoa pode ter suas próprias credenciais configuradas localmente, enquanto servidores de produção usam credenciais específicas do ambiente.

```r
# Configurar variável de ambiente antes de executar o R:
# export MYSQL_PASSWORD="sua_senha"
# No código, usar:
password = Sys.getenv("MYSQL_PASSWORD")

# Exemplo com configurações de segurança adicionais:
# minha_conexao_segura <- dbConnect(
#   RMySQL::MySQL(),
#   host = "mysql-server.empresa.com",
#   port = 3306,
#   user = Sys.getenv("MYSQL_USER"),      # Usuário também em variável
#   password = Sys.getenv("MYSQL_PASSWORD"),
#   dbname = Sys.getenv("MYSQL_DB"),      # Nome do banco em variável
#   ssl.ca = "/path/to/ca-cert.pem",      # Certificado SSL
#   timeout = 10                          # Timeout de conexão
# )

# Sempre fechar a conexão quando terminar
# dbDisconnect(minha_conexao_segura)
```


A função `Sys.getenv()` permite acessar variáveis de ambiente configuradas na máquina local. Essas variáveis são úteis, por exemplo, para armazenar credenciais ou caminhos de diretórios de forma segura.
Você pode aprender a configurar variáveis de ambiente nos seguintes sistemas operacionais: [Windows](https://learn.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2003/cc736637(v=ws.10)?redirectedfrom=MSDN) e para [MacOS](https://forum.macmagazine.com.br/topic/221523-tutorial-como-criar-vari%C3%A1veis-de-ambiente-no-macos/).

Alternativamente, caso você esteja utilizando o RStudio (incluindo o RStudio Server usado neste curso), é possível utilizar a função `rstudioapi::askForPassword()` para solicitar informações sensíveis de forma interativa e sem exibição direta no console. Apesar do nome, essa função pode ser usada para qualquer tipo de entrada textual (como host, usuário, etc.), não apenas senhas:

```r
# minha_conexao_mysql <- dbConnect(
#   RMySQL::MySQL(),
#   host = rstudioapi::askForPassword("Insira o host"),
#   port = 3306,
#   user = rstudioapi::askForPassword("Insira seu usuário"),
#   password = rstudioapi::askForPassword("Insira sua senha"),
#   dbname = "nome_do_banco"
# )
```

Essa abordagem é útil em ambientes interativos, pois evita que as credenciais fiquem visíveis no código ou console.

Por fim, vale destacar algumas boas práticas adicionais de segurança:
  - Usar conexões SSL quando disponível
  - Configurar timeout de conexão
  - Fechar conexões após o uso
  - Usar usuários com privilégios mínimos necessários


### Operações com MySQL

```r
# Estrutura similar ao SQLite
# dbListTables(minha_conexao_mysql)
# dbGetQuery(minha_conexao_mysql, "SELECT * FROM tabela LIMIT 10")
# tbl(minha_conexao_mysql, "tabela") %>% filter(...) %>% collect()
# dbDisconnect(minha_conexao_mysql)
```

As operações com MySQL seguem a mesma lógica utilizada para o SQLite. A principal diferença está na etapa de conexão, que exige o uso de credenciais (como host, usuário e senha), conforme mostramos anteriormente. Uma vez conectados, é possível listar tabelas, executar comandos SQL diretamente ou integrar com o `dplyr` para manipulação mais fluida.


## Conexões com demais bancos SQL

Além do MySQL e SQLite, é comum se deparar com outros sistemas de bancos de dados relacionais. Abaixo mostramos como estabelecer conexões com alguns dos mais utilizados. A manipulação posterior (listar tabelas, executar queries SQL ou usar `dplyr`) segue a mesma lógica para todos os casos.

#### MariaDB

```r
# install.packages("RMariaDB")
library(RMariaDB)

minha_conexao_mariadb <- dbConnect(
  RMariaDB::MariaDB(),
  host = rstudioapi::askForPassword("Insira o host"),
  port = 3306,
  user = rstudioapi::askForPassword("Insira seu usuário"),
  password = rstudioapi::askForPassword("Insira sua senha"),
  dbname = "nome_do_banco"
)
```

#### PostgreSQL

```r
# install.packages("RPostgres")
library(RPostgres)

minha_conexao_pg <- dbConnect(
  RPostgres::Postgres(),
  host = rstudioapi::askForPassword("Insira o host"),
  port = 5432,
  user = rstudioapi::askForPassword("Insira seu usuário"),
  password = rstudioapi::askForPassword("Insira sua senha"),
  dbname = "nome_do_banco"
)
```

#### SQL Server (Microsoft)

```r
# install.packages("odbc")
library(odbc)

minha_conexao_sqlserver <- dbConnect(
  odbc::odbc(),
  Driver   = "SQL Server",
  Server   = rstudioapi::askForPassword("Insira o servidor"),
  Database = "nome_do_banco",
  UID      = rstudioapi::askForPassword("Insira seu usuário"),
  PWD      = rstudioapi::askForPassword("Insira sua senha"),
  Port     = 1433
)
```

#### Oracle

```r
# install.packages("odbc")
library(odbc)

minha_conexao_oracle <- dbConnect(
  odbc::odbc(),
  Driver = "Oracle",
  DBQ    = rstudioapi::askForPassword("Insira o endereço do banco"),
  UID    = rstudioapi::askForPassword("Insira seu usuário"),
  PWD    = rstudioapi::askForPassword("Insira sua senha")
)
```


Em todos esses casos, uma vez criada a conexão, as operações são semelhantes:

```r
# dbListTables(minha_conexao)
# dbGetQuery(minha_conexao, "SELECT * FROM tabela LIMIT 10")
# tbl(minha_conexao, "tabela") %>% filter(...) %>% collect()
# dbDisconnect(minha_conexao)
```

Lembre-se de sempre finalizar sua sessão com `dbDisconnect()` para liberar os recursos da conexão.


## DuckDB: Banco de dados analítico


Por fim, vale destacar o **DuckDB**, um banco de dados relacional projetado especificamente para análise de dados em ambiente local. Desenvolvido a partir de 2019, o DuckDB tem ganhado reconhecimento na comunidade de ciência de dados pela sua combinação de simplicidade operacional, alto desempenho e integração nativa com linguagens como R e Python.

O principal diferencial do DuckDB reside na sua arquitetura: ele consegue processar arquivos diretamente do disco — incluindo formatos como CSV, Parquet, JSON e Excel — **sem exigir que os dados sejam carregados previamente na memória**. Esta capacidade permite análises de conjuntos de dados extensos mesmo em máquinas com recursos limitados, evitando problemas de memória insuficiente e reduzindo consideravelmente o tempo entre a pergunta analítica e sua resposta. Além disso, funciona como uma biblioteca embarcada, executando dentro do próprio processo do R sem depender de servidores externos ou configurações complexas.

Outro aspecto relevante é sua **abordagem estendida ao SQL**. Embora mantenha total compatibilidade com comandos SQL padrão, o DuckDB oferece extensões e funções especializadas que atendem às necessidades específicas dos analistas de dados. Essas funcionalidades ampliam as possibilidades analíticas diretamente em SQL, proporcionando uma experiência mais integrada e eficiente. A documentação completa dessas extensões está disponível em duckdb.org/docs/stable/sql/dialect/overview, oferecendo um guia detalhado para utilizar adequadamente a ferramenta.


### Configuração inicial

```r
# Conexão em memória
minha_conexao_duckdb <- dbConnect(duckdb())

# Conexão persistente em arquivo
# minha_conexao_duckdb <- dbConnect(duckdb(), dbdir = "meu_banco.duckdb")
```

A conexão em memória é perfeita para análises temporárias. Use arquivo quando precisar persistir os dados entre sessões.

### Métodos de transferência de dados

```r
# Criar conexão em memória
minha_conexao_duckdb <- dbConnect(duckdb())

# Criar conexão persistente em arquivo (similar ao que vimos com o SQLite)
minha_conexao_duckdb2 <- dbConnect(duckdb(), dbdir = "meu_banco.duckdb")

# Sempre fechar a conexão quando terminar
# dbDisconnect(minha_conexao_duckdb2)
```

A diferença é crucial: `dbWriteTable()` duplica os dados dentro do banco, ocupando memória adicional. Já `duckdb_register()` cria apenas um 'atalho' para o data frame original - ideal quando os dados já estão no R e queremos apenas consultá-los com SQL.

### Leitura direta de arquivos

A grande vantagem do DuckDB é que ele pode processar arquivos sem carregá-los na memória. Isso é extremamente importante para grandes volumes de dados. 

```r
# Instalar a extensão httpfs para ler URLs
db_exec(minha_conexao_duckdb, "INSTALL httpfs")
db_exec(minha_conexao_duckdb, "LOAD httpfs")
# também funciona com o 'dbExecute' que aprendemos do DBI!

# Processando CSV diretamente
resultado_csv <- dbGetQuery(minha_conexao_duckdb, "
  SELECT 
    dep_adm,
    COUNT(*) as total,
    AVG(CAST(REPLACE(tx_aprovacao, ',', '.') AS DOUBLE)) as media
  FROM read_csv(
    'https://repositorio.seade.gov.br/dataset/1532141b-bff2-4e23-84a7-519a635a5d3b/resource/6d7d84a7-9e74-43dc-bea9-231eec2179a2/download/educacao_rendimento.csv',
    delim = ';',            # Delimitador ponto e vírgula para formato brasileiro
    encoding = 'latin-1',   # encoding latin-1 para acentos em português brasileiro
    header = true           # usar a primeira linha (o "header") como nome das variáveis
  )
  WHERE dep_adm IN ('Estadual', 'Municipal')
  GROUP BY dep_adm
")
```

O `read_csv_auto()` é uma função especial do DuckDB que lê e processa arquivos sem carregá-los no R. Isso economiza memória e tempo, especialmente com arquivos grandes.
Usualmente, o DuckDB detecta automaticamente o delimitador, encoding e tipos de dados; porém é mais seguro descrevê-los manualmente, quando possível. 

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
flights_duck <- as_duckdb_tibble(flights)

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
#install.packages("tictoc") # para marcação de tempo
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

## Analisando dados maiores que a memória

Uma das grandes vantagens do duckplyr é sua capacidade de trabalhar com dados que não cabem na memória RAM do seu computador. Isso é especialmente útil quando você precisa analisar arquivos gigantescos - aqueles que fariam o R travar ou simplesmente não conseguiriam ser carregados.

### Por que isso é importante?

Imagine que você tem um arquivo CSV de 50 GB com dados de vendas de uma empresa multinacional. Com dplyr tradicional, você precisaria:
1. Tentar carregar tudo na memória (provavelmente travaria)
2. Dividir o arquivo em pedaços menores manualmente
3. Processar cada pedaço separadamente
4. Juntar os resultados depois

Com duckplyr, você simplesmente aponta para o arquivo e trabalha normalmente, como se fosse um data frame pequeno. O DuckDB cuida de toda a complexidade por você.

### Exemplo prático: dados de voos

Vamos trabalhar com uma versão estendida do dataset `nycflights13`, que está disponível como arquivos Parquet remotos. Esses arquivos contêm dados de voos de 2022 a 2024, com milhões de registros distintos.

```{r}
# Definindo as URLs dos arquivos Parquet
year <- 2022:2024
base_url <- "https://blobs.duckdb.org/flight-data-partitioned/"
files <- paste0("Year=", year, "/data_0.parquet")
urls <- paste0(base_url, files)

# Vamos ver o que temos
tibble(urls)
```

### Configurando a extensão httpfs

Para ler arquivos diretamente da internet, precisamos da extensão `httpfs` do DuckDB:

```{r}
# Instalando e carregando a extensão (caso ainda não tenha feito)
#db_exec("INSTALL httpfs")
#db_exec("LOAD httpfs")

# Criando uma "conexão" com os arquivos remotos
flights <- read_parquet_duckdb(urls)
```

O `read_parquet_duckdb()` é uma função especial que cria uma referência aos arquivos Parquet sem baixá-los. É como criar um "atalho" que permite trabalhar com os dados como se fossem um data frame comum, mas que na verdade estão armazenados remotamente.

### Explorando os dados sem carregá-los

Uma característica importante: por padrão, o duckplyr protege sua memória evitando materializar resultados muito grandes automaticamente:

```{r}
# Isso vai dar erro propositalmente para proteger sua memória
nrow(flights)
# Error: Materialization would result in more than 9090 rows. Use collect() or as_tibble() to materialize.
```

Mas podemos ver a estrutura dos dados:

```{r}
# Visualizando os primeiros registros (busca apenas algumas linhas)
flights

# Contando registros por ano (operação de agregação, resultado pequeno)
flights %>%
  count(Year)
```

Observe que a contagem funciona normalmente porque o resultado é pequeno (apenas 3 linhas), mas tentar acessar todas as linhas de uma vez é bloqueado para proteger sua memória.

### Realizando análises complexas

Agora vem a mágica: podemos fazer análises complexas em milhões de registros como se fosse um data frame normal:

```{r}
# Análise de atrasos por ano e mês
resultado_voos <- flights %>%
  mutate(InFlightDelay = ArrDelay - DepDelay) %>%
  summarise(
    .by = c(Year, Month),
    atraso_medio_voo = mean(InFlightDelay, na.rm = TRUE),
    atraso_mediano_voo = median(InFlightDelay, na.rm = TRUE),
    total_voos = n()
  ) %>%
  filter(Year < 2024) %>%  # Excluindo 2024 por estar incompleto
  arrange(Year, Month)

# Visualizando o plano de execução (opcional, mas interessante)
resultado_voos %>% explain()
```

O `explain()` mostra como o DuckDB vai executar a consulta. Note que ele:
- Só lê as colunas necessárias (`Year`, `Month`, `DepDelay`, `ArrDelay`)
- Aplica filtros diretamente nos arquivos (`Year < 2024`)
- Só processa 2 dos 3 arquivos (porque 2024 é excluído)

### Coletando os resultados

Quando estamos satisfeitos com nossa análise, usamos `collect()` para trazer os resultados finais para a memória:

```{r}
# Medindo o tempo de execução
system.time({
  resultado_final <- resultado_voos %>% collect()
})

# Visualizando resultados
head(resultado_final, 10)
```

Impressionante! Analisamos milhões de registros em segundos, processando apenas os dados necessários e trazendo para a memória apenas o resultado final (24 linhas, no caso).

### Trabalhando com arquivos locais

O mesmo princípio funciona com arquivos no seu computador:

```{r}
# Exemplo com arquivos locais (não execute se não tiver os arquivos)
# flights_local <- read_parquet_duckdb("dados/voos_*.parquet")
# 
# # Ou com CSVs
# vendas_grandes <- read_csv_duckdb("dados/vendas_2024_*.csv")
# 
# # Análise normal
# vendas_por_regiao <- vendas_grandes %>%
#   group_by(regiao, mes) %>%
#   summarise(
#     vendas_totais = sum(valor_venda),
#     ticket_medio = mean(valor_venda),
#     .groups = "drop"
#   ) %>%
#   collect()
```

### Estratégias para dados muito grandes

Quando trabalhando com dados gigantescos, algumas dicas importantes:

1. **Filtre cedo**: Use `filter()` o mais cedo possível na pipeline para reduzir o volume de dados processados.

```{r}
# Bom: filtra logo no início
resultado <- flights %>%
  filter(Year == 2023, Month %in% 6:8) %>%  # Só verão de 2023
  group_by(Origin, Dest) %>%
  summarise(voos_totais = n()) %>%
  collect()

# Ruim: filtra depois de processar tudo
# resultado <- flights %>%
#   group_by(Origin, Dest) %>%
#   summarise(voos_totais = n()) %>%
#   filter(voos_totais > 1000) %>%  # Muito tarde!
#   collect()
```

2. **Selecione apenas colunas necessárias**: Se o dataset tem 100 colunas mas você só precisa de 5, selecione apenas essas 5.

```{r}
# Eficiente: só as colunas que vamos usar
flights %>%
  select(Year, Month, Origin, Dest, DepDelay, ArrDelay) %>%
  filter(Year == 2023) %>%
  # ... resto da análise
  collect()
```

3. **Agregue antes de coletar**: Sempre que possível, reduza os dados no servidor antes de trazer para o R.

```{r}
# Bom: agrega primeiro, depois coleta resultado pequeno
resumo <- flights %>%
  filter(Year == 2023) %>%
  group_by(Month) %>%
  summarise(
    total_voos = n(),
    atraso_medio = mean(DepDelay, na.rm = TRUE)
  ) %>%
  collect()  # Só 12 linhas vêm para a memória

# Evite: trazer milhões de linhas para depois agregar no R
# dados_brutos <- flights %>% 
#   filter(Year == 2023) %>% 
#   collect()  # Milhões de linhas!
# resumo <- dados_brutos %>% group_by(...) # Muito lento
```

### Limitações importantes

O duckplyr tem algumas limitações com dados remotos que é importante conhecer:

1. **Funções não suportadas**: Nem todas as funções do dplyr funcionam com DuckDB. Quando isso acontece, há fallback automático para dplyr, mas isso pode ser lento com dados grandes.

2. **Ordem dos resultados**: Por padrão, os resultados podem vir em ordem diferente do dplyr tradicional (por questões de performance). Se a ordem importa, use `arrange()` explicitamente.

3. **Materialização automática**: Para proteger sua memória, resultados muito grandes não são materializados automaticamente. Use `collect()` conscientemente.

### Quando usar esta abordagem

Esta estratégia de análise é ideal quando você tem:

- **Arquivos muito grandes** (> 1GB) que não cabem confortavelmente na memória
- **Múltiplos arquivos** que você quer analisar como um conjunto único
- **Dados remotos** que você não quer ou não pode baixar completamente
- **Análises que resultam em dados pequenos** (agregações, sumários, etc.)

A combinação duckplyr + arquivos remotos é perfeita para o mundo moderno, onde os dados ficam cada vez maiores mas nossa necessidade geralmente é extrair informações específicas, não carregando tudo na memória.


## Conclusão

Cada tecnologia apresentada tem seu nicho específico:

- **SQLite** é ideal para desenvolvimento local e prototipagem
- **MySQL, MariaDB** e afins atendem necessidades de sistemas corporativos
- **DuckDB** oferece performance superior para análise de dados
- **duckplyr** permite otimização com mudanças mínimas no código

A escolha depende do contexto: volume de dados, infraestrutura disponível e requisitos de performance. O domínio dessas ferramentas permite trabalhar eficientemente com dados de qualquer escala.

## Exercícios práticos

1. **Análise de performance**: Compare o tempo de execução de agregações complexas usando `dplyr` tradicional, `duckplyr` e SQL direto no DuckDB.

2. **Processamento sem carregar**: Utilize DuckDB para analisar um arquivo CSV grande sem carregá-lo completamente na memória.

3. **Pipeline integrado**: Crie um fluxo que use SQLite para armazenamento persistente e DuckDB para análises rápidas.

