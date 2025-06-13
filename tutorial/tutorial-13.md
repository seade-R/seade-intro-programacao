Vou verificar o conteúdo do link do DuckDB para complementar o tutorial com informações oficiais.Excelente! A documentação oficial do DuckDB traz várias funcionalidades importantes que não mencionei. Vou atualizar a seção do DuckDB no tutorial para incluir essas features poderosas:

# Integrando R com Bancos de Dados SQL

Até agora, trabalhamos com dados que cabiam confortavelmente na memória do nosso computador. Mas e quando os dados são grandes demais? Ou quando precisamos acessar dados que estão em um banco de dados corporativo? É hora de aprender a conectar R com bancos de dados SQL!

Neste tutorial, vamos aprender três abordagens diferentes:
1. **SQLite**: perfeito para criar bancos de dados locais e aprender SQL
2. **MySQL**: muito usado em empresas e aplicações web
3. **DuckDB**: a nova sensação para análise de dados grandes, super rápido!

Por que isso é importante? Imagine trabalhar com dados do Censo ou com milhões de registros de vendas. Carregar tudo na memória pode travar seu computador. Com SQL, você processa os dados onde eles estão, trazendo para o R apenas o que precisa.

Vamos começar instalando e carregando os pacotes necessários:

```r
# Pacotes principais
install.packages(c("DBI", "RSQLite", "RMySQL", "duckdb", "duckplyr", "nycflights13"))

# Carregando
library(tidyverse)
library(DBI)        # Interface padrão para bancos de dados
library(RSQLite)    # Para SQLite
library(RMySQL)     # Para MySQL
library(duckdb)     # Para DuckDB
library(duckplyr)   # dplyr com superpoderes do DuckDB
```

## SQLite: Seu primeiro banco de dados

SQLite é perfeito para começar porque:
- Não precisa instalar nenhum servidor
- O banco de dados é apenas um arquivo
- Funciona em qualquer computador
- É surpreendentemente poderoso para dados médios

### Criando um banco de dados SQLite

Vamos criar um banco com dados educacionais que usamos no tutorial anterior:

```r
# Primeiro, vamos carregar alguns dados
edu_rendimento <- read_csv2(
  "https://repositorio.seade.gov.br/dataset/1532141b-bff2-4e23-84a7-519a635a5d3b/resource/6d7d84a7-9e74-43dc-bea9-231eec2179a2/download/educacao_rendimento.csv",
  locale = locale(encoding = "Latin1")
)

# Limpando os dados (como fizemos antes)
edu_rendimento <- edu_rendimento %>% 
  janitor::clean_names() %>% 
  mutate(
    tx_aprovacao = str_replace(tx_aprovacao, ",", ".") %>% as.numeric(),
    tx_reprovacao = str_replace(tx_reprovacao, ",", ".") %>% as.numeric(),
    tx_abandono = str_replace(tx_abandono, ",", ".") %>% as.numeric()
  )

# Criando conexão com SQLite
# O arquivo será criado automaticamente
con_sqlite <- dbConnect(RSQLite::SQLite(), "educacao_sp.db")

# Verificando a conexão
con_sqlite
```

### Salvando dados no banco

```r
# Escrevendo a tabela no banco
# dbWriteTable cria a tabela e insere os dados
dbWriteTable(con_sqlite, "rendimento_escolar", edu_rendimento)

# Listando as tabelas no banco
dbListTables(con_sqlite)

# Verificando os campos da tabela
dbListFields(con_sqlite, "rendimento_escolar")
```

### Consultando dados com SQL

Agora vem a parte divertida! Podemos usar SQL diretamente:

```r
# Query simples - primeiras 10 linhas
resultado <- dbGetQuery(con_sqlite, "
  SELECT * 
  FROM rendimento_escolar 
  LIMIT 10
")

head(resultado)

# Query mais complexa - médias por tipo de escola
medias_sql <- dbGetQuery(con_sqlite, "
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

### Usando dplyr com bancos de dados

A mágica do R: você pode usar dplyr com bancos de dados!

```r
# Criando uma referência à tabela (não carrega os dados!)
tbl_rendimento <- tbl(con_sqlite, "rendimento_escolar")

# Agora podemos usar dplyr normalmente
resultado_dplyr <- tbl_rendimento %>% 
  filter(dep_adm %in% c("Estadual", "Municipal", "Privada")) %>% 
  group_by(dep_adm, escolaridade) %>% 
  summarise(
    media_aprovacao = mean(tx_aprovacao, na.rm = TRUE),
    n = n()
  ) %>% 
  arrange(desc(media_aprovacao))

# IMPORTANTE: os dados ainda estão no banco!
# Para trazer para o R, use collect()
dados_local <- resultado_dplyr %>% collect()

print(dados_local)
```

### Vendo o SQL gerado pelo dplyr

Uma função muito útil é `show_query()`, que mostra o SQL que o dplyr está gerando:

```r
# Veja o SQL que o dplyr criou
tbl_rendimento %>% 
  filter(dep_adm %in% c("Estadual", "Municipal", "Privada")) %>% 
  group_by(dep_adm) %>% 
  summarise(media = mean(tx_aprovacao, na.rm = TRUE)) %>% 
  show_query()
```

### Fechando a conexão

Sempre feche as conexões quando terminar!

```r
dbDisconnect(con_sqlite)
```

## MySQL: Conectando a bancos corporativos

MySQL (e seu primo MariaDB) são muito usados em empresas. A conexão é um pouco mais complexa porque precisamos de:
- Servidor (host)
- Porta
- Usuário e senha
- Nome do banco

### Conectando ao MySQL

```r
# Exemplo de conexão (você precisará ajustar para seu servidor)
# NUNCA coloque senhas diretamente no código em produção!
con_mysql <- dbConnect(
  RMySQL::MySQL(),
  host = "localhost",      # ou IP do servidor
  port = 3306,            # porta padrão do MySQL
  user = "seu_usuario",
  password = "sua_senha",  # Em produção, use variáveis de ambiente!
  dbname = "nome_do_banco"
)

# Para ambientes de produção, use variáveis de ambiente:
# password = Sys.getenv("MYSQL_PASSWORD")
```

### Exemplo prático com MySQL

Se você tiver acesso a um servidor MySQL, o processo é similar ao SQLite:

```r
# Listando tabelas disponíveis
# dbListTables(con_mysql)

# Lendo uma tabela inteira (cuidado com o tamanho!)
# dados <- dbReadTable(con_mysql, "nome_da_tabela")

# Query SQL
# resultado <- dbGetQuery(con_mysql, "
#   SELECT column1, column2, COUNT(*) as total
#   FROM tabela
#   WHERE condicao = 'valor'
#   GROUP BY column1, column2
# ")

# Usando dplyr
# tbl_mysql <- tbl(con_mysql, "nome_da_tabela")
# resultado <- tbl_mysql %>%
#   filter(condicao == "valor") %>%
#   group_by(column1) %>%
#   summarise(total = n()) %>%
#   collect()

# Sempre desconecte!
# dbDisconnect(con_mysql)
```

### Dicas de segurança para MySQL

1. **Nunca coloque senhas no código!** Use:
   ```r
   # No terminal, antes de abrir o R:
   # export MYSQL_PASSWORD="sua_senha_segura"
   
   # No R:
   password = Sys.getenv("MYSQL_PASSWORD")
   ```

2. **Use apenas as permissões necessárias** - geralmente só leitura

3. **Cuidado com queries grandes** - use LIMIT para testar primeiro

## DuckDB: O banco de dados analítico moderno

DuckDB é uma revolução porque:
- É um banco de dados analítico completo dentro do R
- Extremamente rápido para agregações e joins
- Pode ler arquivos CSV e Parquet diretamente
- Sintaxe SQL completa ou via dplyr/dbplyr

### Criando e conectando ao DuckDB

```r
library(duckdb)

# Conexão em memória (padrão)
con_duck <- dbConnect(duckdb())

# Ou salvando em arquivo
# con_duck <- dbConnect(duckdb(), dbdir = "meu_banco.duckdb")

# Para modo read-only (múltiplos processos)
# con_duck <- dbConnect(duckdb(), dbdir = "banco.duckdb", read_only = TRUE)
```

### Transferência eficiente de dados

```r
# Método 1: dbWriteTable (copia os dados)
dbWriteTable(con_duck, "edu_table", edu_rendimento)

# Método 2: duckdb_register (cria uma VIEW virtual - não copia!)
# Isso é MUITO mais eficiente para dados grandes
duckdb_register(con_duck, "edu_view", edu_rendimento)

# Ambos podem ser consultados da mesma forma
resultado1 <- dbGetQuery(con_duck, "SELECT COUNT(*) FROM edu_table")
resultado2 <- dbGetQuery(con_duck, "SELECT COUNT(*) FROM edu_view")

# A diferença: edu_view não ocupa espaço no banco!
```

### Lendo arquivos diretamente com DuckDB

Uma das features mais poderosas do DuckDB:

```r
# Lendo CSV diretamente - sem carregar no R!
resultado_csv <- dbGetQuery(con_duck, "
  SELECT 
    dep_adm,
    COUNT(*) as total,
    AVG(CAST(REPLACE(tx_aprovacao, ',', '.') AS DOUBLE)) as media
  FROM read_csv_auto('https://repositorio.seade.gov.br/dataset/1532141b-bff2-4e23-84a7-519a635a5d3b/resource/6d7d84a7-9e74-43dc-bea9-231eec2179a2/download/educacao_rendimento.csv')
  WHERE dep_adm IN ('Estadual', 'Municipal')
  GROUP BY dep_adm
")

print(resultado_csv)

# Lendo múltiplos arquivos Parquet
# dbGetQuery(con_duck, "
#   SELECT * FROM read_parquet('dados/*.parquet', hive_partitioning = true)
#   WHERE ano = 2023
# ")
```

### dbplyr com DuckDB: O melhor dos dois mundos

```r
library(dbplyr)
library(nycflights13)

# Registrando o data frame flights como VIEW
duckdb_register(con_duck, "flights", flights)

# Agora podemos usar dplyr!
analise_voos <- tbl(con_duck, "flights") %>%
  filter(month == 6) %>%
  group_by(origin, dest) %>%
  summarise(
    n_voos = n(),
    atraso_medio = mean(arr_delay, na.rm = TRUE),
    .groups = "drop"
  ) %>%
  filter(n_voos > 50) %>%
  arrange(desc(atraso_medio))

# Ver o SQL gerado
analise_voos %>% show_query()

# Executar e coletar
resultado <- analise_voos %>% collect()
```

### Lendo arquivos CSV e Parquet via dbplyr

```r
# Preparando um CSV para exemplo
write.csv(mtcars, "mtcars.csv", row.names = FALSE)

# Lendo CSV diretamente via dplyr/dbplyr
carros <- tbl(con_duck, "read_csv_auto('mtcars.csv')") %>%
  group_by(cyl) %>%
  summarise(
    n = n(),
    mpg_medio = mean(mpg),
    hp_medio = mean(hp)
  ) %>%
  collect()

print(carros)

# Para Parquet com partições
# tbl(con_duck, "read_parquet('dados/**/*.parquet', hive_partitioning = true)") %>%
#   filter(ano == 2023, mes == 3) %>%
#   summarise(total = sum(valor))
```

### Configurando limite de memória

```r
# Limitando o uso de memória do DuckDB
dbExecute(con_duck, "SET memory_limit = '2GB'")

# Verificando a configuração
dbGetQuery(con_duck, "SELECT * FROM duckdb_settings() WHERE name = 'memory_limit'")
```

### Prepared statements com DuckDB

Para inserções seguras e eficientes:

```r
# Criando uma tabela de exemplo
dbExecute(con_duck, "CREATE TABLE produtos (nome VARCHAR, preco DECIMAL, qtd INTEGER)")

# Inserção simples com prepared statement
dbExecute(con_duck, 
  "INSERT INTO produtos VALUES (?, ?, ?)", 
  list('Notebook', 3500.00, 10)
)

# Para múltiplas inserções
stmt <- dbSendStatement(con_duck, "INSERT INTO produtos VALUES (?, ?, ?)")
dbBind(stmt, list('Mouse', 50.00, 100))
dbBind(stmt, list('Teclado', 150.00, 50))
dbBind(stmt, list('Monitor', 800.00, 25))
dbClearResult(stmt)

# Verificando
dbGetQuery(con_duck, "SELECT * FROM produtos")
```

### Fechando conexões e limpando

```r
# Removendo registro de VIEW (libera referência ao data frame)
duckdb_unregister(con_duck, "edu_view")
duckdb_unregister(con_duck, "flights")

# Fechando conexão
dbDisconnect(con_duck, shutdown = TRUE)
```

## duckplyr: Análise de dados em alta velocidade

O duckplyr é uma implementação do dplyr que usa DuckDB por baixo dos panos:

```r
library(duckplyr)
library(nycflights13)

# Convertendo para duckplyr_df
flights_duck <- as_duckplyr_df(flights)

# Agora toda operação dplyr é acelerada!
resultado <- flights_duck %>%
  filter(!is.na(dep_delay)) %>%
  group_by(origin, dest, carrier) %>%
  summarise(
    n_voos = n(),
    atraso_medio = mean(arr_delay, na.rm = TRUE),
    distancia_total = sum(distance),
    .groups = "drop"
  ) %>%
  filter(n_voos > 365) %>%  # Rotas com mais de 1 voo por dia em média
  arrange(desc(atraso_medio))

# Top 10 piores rotas
head(resultado, 10)
```

### Comparando velocidade com dados maiores

```r
# Criando um dataset maior para teste
set.seed(42)
dados_grandes <- tibble(
  id = 1:5000000,
  categoria = sample(LETTERS, 5000000, replace = TRUE),
  valor = rnorm(5000000, 100, 20),
  grupo = sample(1:1000, 5000000, replace = TRUE)
)

# Teste com dplyr normal
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

# Teste com duckplyr
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

# duckplyr é MUITO mais rápido para agregações!
```

## Comparando as opções

| Característica | SQLite | MySQL | DuckDB | duckplyr |
|---------------|---------|--------|---------|----------|
| Instalação | Nenhuma | Servidor | Nenhuma | Nenhuma |
| Velocidade | Boa | Boa | Excelente | Excelente |
| Sintaxe | SQL | SQL | SQL/dplyr | dplyr puro |
| Dados externos | Não | Não | CSV/Parquet | Via DuckDB |
| Uso típico | Apps locais | Web/Empresa | Análise | Análise rápida |

## Boas práticas

1. **Sempre feche conexões**
   ```r
   # Use on.exit() em funções
   minha_analise <- function() {
     con <- dbConnect(duckdb())
     on.exit(dbDisconnect(con, shutdown = TRUE))
     
     # Seu código aqui
   }
   ```

2. **Use register em vez de write para dados grandes**
   ```r
   # Evite copiar dados desnecessariamente
   # Ruim: dbWriteTable(con, "tabela", dados_gigantes)
   # Bom: duckdb_register(con, "tabela", dados_gigantes)
   ```

3. **Aproveite a leitura direta de arquivos**
   ```r
   # Em vez de:
   # dados <- read_csv("arquivo_enorme.csv")
   # resultado <- dados %>% group_by(x) %>% summarise(n = n())
   
   # Faça:
   con <- dbConnect(duckdb())
   resultado <- dbGetQuery(con, "
     SELECT x, COUNT(*) as n 
     FROM read_csv_auto('arquivo_enorme.csv')
     GROUP BY x
   ")
   ```

## Exercícios práticos

1. **Comparação de velocidade**:
   - Crie um data frame com 10 milhões de linhas
   - Compare agregações complexas: dplyr vs duckplyr vs DuckDB SQL
   - Faça um gráfico dos tempos

2. **Análise sem carregar dados**:
   - Baixe um CSV grande (dados públicos)
   - Use DuckDB para analisar sem carregar no R
   - Compare memória usada: read_csv vs DuckDB

3. **Pipeline completo**:
   - Use SQLite para armazenar dados processados
   - Use DuckDB para análises rápidas
   - Exporte resultados para visualização

## Conclusão

Agora você tem um arsenal completo para trabalhar com dados de qualquer tamanho:

- **SQLite**: Perfeito para organizar e persistir dados localmente
- **MySQL**: Essencial para dados corporativos
- **DuckDB**: O canivete suíço da análise - rápido, flexível e poderoso
- **duckplyr**: Mesma sintaxe dplyr, velocidade turbinada

A melhor parte? Você não precisa escolher apenas um! Use SQLite para organizar, DuckDB para analisar rapidamente, e duckplyr quando quiser manter seu código dplyr mas precisar de mais velocidade.

