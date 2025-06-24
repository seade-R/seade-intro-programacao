# Análise de Dados: transformando dados reais em conhecimento

Chegamos ao final do nosso curso! Nos tutoriais anteriores, aprendemos diversas técnicas de manipulação, transformação e visualização de dados. Agora é hora de juntar tudo em um exemplo prático e dar um passo além.

Neste tutorial, vamos trabalhar com dados educacionais reais do Seade e você descobrirá uma verdade fundamental da ciência de dados: a maior parte do trabalho não está na análise sofisticada, mas sim na limpeza e preparação dos dados.

Vamos enfrentar desafios reais como:

    - Problemas de encoding (acentos que aparecem como símbolos estranhos)
    - Números com vírgula em vez de ponto decimal
    - Dados que já vêm pré-agregados de formas complexas
    - Valores missing que têm significado importante

Este é apenas o começo da jornada em análise de dados. Para ir além, estudar estatística é fundamental - aqui veremos apenas o básico sobre correlações e regressões. Mas o importante é entender o processo: como um analista pensa ao receber dados bagunçados e precisa transformá-los em insights.

Vamos começar carregando os pacotes necessários:

```r
library(tidyverse)
library(janitor)
library(corrplot)  # para visualização de correlações
```

## Dados de Rendimento Escolar do Seade

Para este tutorial, utilizaremos dados de rendimento escolar (aprovação, reprovação e abandono) dos municípios paulistas. 

Uma observação importante: a função `read_csv()` (e suas variantes como `read_csv2()`) pode importar dados diretamente de URLs da internet, sem necessidade de baixar o arquivo primeiro. Isso é muito prático para trabalhar com dados públicos disponíveis online!

Vamos começar carregando os dados:

```r
# Tentando carregar os dados de rendimento escolar
edu_rendimento <- read_csv2("https://repositorio.seade.gov.br/dataset/1532141b-bff2-4e23-84a7-519a635a5d3b/resource/6d7d84a7-9e74-43dc-bea9-231eec2179a2/download/educacao_rendimento.csv")

# Vamos ver o que aconteceu
glimpse(edu_rendimento)
head(edu_rendimento)
```

## O problema do encoding

Observe que os dados têm caracteres estranhos! Isso acontece porque o arquivo está salvo com encoding Latin-1 (também conhecido como ISO-8859-1), mas o R está tentando lê-lo como UTF-8.

### O que é encoding?

Encoding é a forma como os caracteres são representados em bytes no computador. Os encodings mais comuns são:
- **UTF-8**: padrão moderno, suporta todos os caracteres do mundo
- **Latin-1 (ISO-8859-1)**: padrão antigo, comum em dados brasileiros mais antigos
- **Windows-1252**: similar ao Latin-1, usado no Windows

Quando o encoding está errado, caracteres especiais (acentos, ç, etc.) aparecem como símbolos estranhos. Vamos corrigir isso:

```r
# Carregando com o encoding correto
edu_rendimento <- read_csv2(
  "https://repositorio.seade.gov.br/dataset/1532141b-bff2-4e23-84a7-519a635a5d3b/resource/6d7d84a7-9e74-43dc-bea9-231eec2179a2/download/educacao_rendimento.csv",
  locale = locale(encoding = "Latin1")
)

# Dados de municípios também precisam do encoding correto
municipios <- read_csv2(
  "https://repositorio.seade.gov.br/dataset/1617c335-f5ab-426c-b175-280d4e41ec1c/resource/1871ac05-7b7f-4c13-9b4a-23c7d41fd988/download/codigos_municipios_regioes.csv",
  locale = locale(encoding = "Latin1")
)

# Limpando nomes
edu_rendimento <- edu_rendimento %>% clean_names()
municipios <- municipios %>% clean_names()

# Vamos ver os dados
glimpse(edu_rendimento)
```

## Exercício rápido

1. Tente carregar os dados sem especificar encoding. O que acontece com a coluna que deveria conter nomes de municípios?
2. Experimente usar `encoding = "UTF-8"`. Funciona?
3. Como você descobriria o encoding correto se não soubesse que é Latin1? (Dica: às vezes é tentativa e erro!)

## Entendendo a estrutura dos dados

Antes de começar qualquer análise, precisamos entender como os dados estão organizados. Vamos descobrir: 
- Cada linha representa o quê? 
- Temos dados duplicados? 
- Os dados já vêm agregados de alguma forma?

Essas perguntas são fundamentais para evitar erros de interpretação!

```r
# Vendo alguns registros
edu_rendimento %>% 
  select(cod_ibge, dep_adm, escolaridade) %>% 
  head(20)

# Para um município específico, vamos ver todas as linhas
# Lembra do filter? Vamos filtrar pelo código de São Paulo
edu_rendimento %>% 
  filter(cod_ibge == 3550308) %>%  # São Paulo
  select(cod_ibge, dep_adm, escolaridade, tx_aprovacao) %>% 
  print(n = 30)
```

Observe que os dados já estão desagregados! Para cada município, temos:
- **Total**: agregação de todas as escolas
- **Estadual**: apenas escolas estaduais
- **Municipal**: apenas escolas municipais  
- **Privada**: apenas escolas privadas
- **Pública**: agregação de estaduais + municipais

**Cuidado**: Isso significa que não podemos simplesmente calcular médias com todos os dados, pois estaríamos contando algumas escolas múltiplas vezes!

## Preparando os dados corretamente

Agora vamos resolver alguns problemas técnicos comuns em dados brasileiros:

```r
# Convertendo vírgulas em pontos (padrão brasileiro vs padrão R)
# str_replace() substitui um padrão por outro
# Depois convertemos para numérico com as.numeric()
edu_rendimento <- edu_rendimento %>% 
  mutate(
    tx_aprovacao = as.numeric(str_replace(tx_aprovacao, ",", ".")),
    tx_reprovacao = as.numeric(str_replace(tx_reprovacao, ",", ".")),
    tx_abandono = as.numeric(str_replace(tx_abandono, ",", "."))
  )

# Fazendo o join com municípios para ter os nomes
# left_join mantém todas as linhas da esquerda (edu_rendimento)
edu_completo <- edu_rendimento %>% 
  left_join(municipios, by = "cod_ibge")

# Verificando se funcionou
edu_completo %>% 
  filter(!is.na(municipios)) %>% 
  select(municipios, dep_adm, escolaridade, tx_aprovacao) %>% 
  head(10)
```

## Verificando a qualidade dos dados

Dados reais frequentemente contêm erros e inconsistências. Vamos investigar:

```r
# Verificando valores impossíveis
# As taxas são porcentagens, então devem estar entre 0 e 100
problemas_valores <- edu_completo %>% 
  filter(
    tx_aprovacao < 0 | tx_aprovacao > 100 |
    tx_reprovacao < 0 | tx_reprovacao > 100 |
    tx_abandono < 0 | tx_abandono > 100
  )

# Registros com valores fora do intervalo [0,100]:
nrow(problemas_valores)

# Verificando valores missing
valores_na <- edu_completo %>% 
  filter(
    is.na(tx_aprovacao) | is.na(tx_reprovacao) | is.na(tx_abandono)
  )

# Registros com valores NA:
nrow(valores_na)

# Vamos ver alguns exemplos de missing
valores_na %>% 
  select(municipios, dep_adm, escolaridade, tx_aprovacao, tx_reprovacao, tx_abandono) %>% 
  head(10)
```

### Identificando padrões nos dados missing

```r
# Há padrão nos valores missing? Vamos contar por categorias
# Por tipo de escola
valores_na %>% 
  count(dep_adm) %>% 
  arrange(desc(n))

# Por escolaridade
valores_na %>% 
  count(escolaridade) %>% 
  arrange(desc(n))

# Combinação de ambos
valores_na %>% 
  count(dep_adm, escolaridade) %>% 
  arrange(desc(n))
```

Os valores missing geralmente indicam que aquele tipo de escola não existe no município para aquele nível de ensino. Por exemplo, muitos municípios pequenos não têm escolas privadas.

### Verificando a consistência das somas

Em dados de porcentagem, a soma das partes deve dar 100%. Vamos verificar:

```r
# Calculando a soma das taxas
edu_completo <- edu_completo %>% 
  mutate(
    soma_taxas = tx_aprovacao + tx_reprovacao + tx_abandono,
    diff_100 = abs(soma_taxas - 100)  # abs() pega o valor absoluto
  )

# Verificando a distribuição das diferenças
# Estatísticas das diferenças em relação a 100%:
edu_completo %>% 
  filter(!is.na(soma_taxas)) %>% 
  pull(diff_100) %>% 
  summary()

# Quantos casos têm diferença significativa?
# case_when() é como um if-else múltiplo
edu_completo %>% 
  filter(!is.na(soma_taxas)) %>% 
  mutate(
    categoria_soma = case_when(
      diff_100 <= 0.5 ~ "Normal (arredondamento)",
      diff_100 <= 2 ~ "Pequena diferença",
      TRUE ~ "Verificar"  # TRUE é o "else" final
    )
  ) %>% 
  count(categoria_soma) %>% 
  mutate(pct = round(n/sum(n) * 100, 1))
```

Como vemos, todos os casos têm somas iguais a 100%. Isso nem sempre é o caso! Sempre desconfie de dados muito "perfeitos".

### Identificando e tratando valores suspeitos

```r
# Valores todos zero (podem indicar falta de dados)
suspeitos_zero <- edu_completo %>% 
  filter(
    tx_aprovacao == 0 & tx_reprovacao == 0 & tx_abandono == 0,
    !is.na(tx_aprovacao)  # Garantir que não são NA
  )

# Registros com todas as taxas zeradas:
nrow(suspeitos_zero)

# Aprovação de 100% - vale investigar
aprovacao_perfeita <- edu_completo %>% 
  filter(tx_aprovacao == 100)

# Registros com aprovação de 100%:
nrow(aprovacao_perfeita)

# Vamos ver alguns exemplos
aprovacao_perfeita %>% 
  select(municipios, dep_adm, escolaridade, tx_aprovacao, tx_reprovacao, tx_abandono) %>% 
  head(10)
```

### Identificando outliers

Outliers são valores extremos que podem indicar erros ou casos especiais:

```r
# Taxas de abandono muito altas (vamos usar 20% como corte)
outliers_abandono <- edu_completo %>% 
  filter(
    dep_adm %in% c("Estadual", "Municipal", "Privada"),  # %in% verifica se está na lista
    tx_abandono > 20
  ) %>% 
  select(municipios, dep_adm, escolaridade, tx_abandono) %>% 
  arrange(desc(tx_abandono))

# Casos com abandono > 20%:
nrow(outliers_abandono)

# Top 10 piores casos de abandono:
print(outliers_abandono, n = 10)

# Taxas de reprovação muito altas
outliers_reprovacao <- edu_completo %>% 
  filter(
    dep_adm %in% c("Estadual", "Municipal", "Privada"),
    tx_reprovacao > 30
  ) %>% 
  select(municipios, dep_adm, escolaridade, tx_reprovacao) %>% 
  arrange(desc(tx_reprovacao))

# Casos com reprovação > 30%:
nrow(outliers_reprovacao)

# Top 10 piores casos de reprovação:
print(outliers_reprovacao, n = 10)
```

## O problema crucial: agregações múltiplas

O maior desafio destes dados não são os valores missing ou pequenas inconsistências, mas sim a estrutura com múltiplos níveis de agregação. Vamos visualizar isso:

```r
# Exemplo para um município grande
# Dados de São Paulo - Ensino Médio:
edu_completo %>% 
  filter(municipios == "São Paulo", 
         escolaridade == "Médio") %>% 
  select(dep_adm, tx_aprovacao, tx_reprovacao, tx_abandono) %>% 
  arrange(dep_adm)
```

Note que temos:
- **Total**: média ponderada de TODAS as escolas
- **Pública**: média ponderada de Estadual + Municipal
- **Estadual, Municipal, Privada**: dados desagregados reais

### Demonstrando o problema da dupla contagem

Vamos ver por que incluir as agregações distorce as análises:

```r
# ERRADO - calculando média com todos os dados
media_errada <- edu_completo %>% 
  filter(escolaridade == "Médio") %>% 
  summarise(
    media_aprovacao = mean(tx_aprovacao, na.rm = TRUE),
    n_registros = n()
  )

# CÁLCULO ERRADO (com agregações)
# Média de aprovação: 
round(media_errada$media_aprovacao, 1)
# Registros usados:
media_errada$n_registros

# CORRETO - apenas dados desagregados
media_correta <- edu_completo %>% 
  filter(
    escolaridade == "Médio",
    dep_adm %in% c("Estadual", "Municipal", "Privada")  # Só escolas reais, não agregações
  ) %>% 
  summarise(
    media_aprovacao = mean(tx_aprovacao, na.rm = TRUE),
    n_registros = n()
  )

# CÁLCULO CORRETO (sem agregações)
# Média de aprovação:
round(media_correta$media_aprovacao, 1)
# Registros usados:
media_correta$n_registros

# Diferença:
round(media_errada$media_aprovacao - media_correta$media_aprovacao, 1)
# pontos percentuais
```

## Criando o dataset limpo para análise

Agora vamos criar uma versão limpa dos dados para análise:

```r
# Removendo agregações e casos problemáticos
edu_limpo <- edu_completo %>% 
  filter(
    # MAIS IMPORTANTE: remover agregações (Total e Pública)
    dep_adm %in% c("Estadual", "Municipal", "Privada"),
    # Remover valores impossíveis (se houver)
    tx_aprovacao >= 0 & tx_aprovacao <= 100,
    tx_reprovacao >= 0 & tx_reprovacao <= 100,
    tx_abandono >= 0 & tx_abandono <= 100
  ) %>% 
  # Marcar casos especiais para análise posterior
  mutate(
    observacao = case_when(
      tx_aprovacao == 100 ~ "Aprovação perfeita",
      tx_abandono > 20 ~ "Abandono alto",
      tx_reprovacao > 30 ~ "Reprovação alta",
      tx_aprovacao == 0 & tx_reprovacao == 0 & tx_abandono == 0 ~ "Todos zeros",
      TRUE ~ "Normal"
    )
  )

# RESUMO DA LIMPEZA
# Dados originais (total):
nrow(edu_completo)

# Dados sem agregações:
nrow(filter(edu_completo, dep_adm %in% c("Estadual", "Municipal", "Privada")))

# Dados limpos finais:
nrow(edu_limpo)

# Registros removidos (além das agregações):
nrow(filter(edu_completo, dep_adm %in% c("Estadual", "Municipal", "Privada"))) - nrow(edu_limpo)
```

## Análise exploratória dos dados limpos

### Quantos municípios têm cada tipo de escola?

```r
# Por tipo de escola
# n_distinct() conta valores únicos
edu_limpo %>% 
  group_by(dep_adm) %>% 
  summarise(
    n_municipios = n_distinct(municipios),
    n_registros = n()
  ) %>% 
  mutate(
    pct_municipios = round(n_municipios / n_distinct(edu_limpo$municipios) * 100, 1)
  )

# Quantos municípios não têm escolas privadas?
# Estratégia: listar municípios COM privada, depois ver quem está fora
municipios_sem_privada <- edu_limpo %>% 
  group_by(municipios) %>% 
  summarise(tem_privada = any(dep_adm == "Privada")) %>%  # any() retorna TRUE se pelo menos um é TRUE
  filter(!tem_privada) %>% 
  nrow()

# Municípios sem escolas privadas:
municipios_sem_privada
```

## Estatísticas descritivas básicas

### Usando summary() por tipo de escola

O comando `summary()` nos dá uma visão geral da distribuição:

```r
# TAXA DE APROVAÇÃO POR TIPO DE ESCOLA

# --- Estadual ---
edu_limpo %>% 
  filter(dep_adm == "Estadual") %>% 
  pull(tx_aprovacao) %>% 
  summary()

# --- Municipal ---
edu_limpo %>% 
  filter(dep_adm == "Municipal") %>% 
  pull(tx_aprovacao) %>% 
  summary()

# --- Privada ---
edu_limpo %>% 
  filter(dep_adm == "Privada") %>% 
  pull(tx_aprovacao) %>% 
  summary()
```

### Estatísticas mais detalhadas

```r
# Estatísticas por tipo de escola e escolaridade
# Vamos calcular várias medidas de uma vez
stats_detalhadas <- edu_limpo %>% 
  group_by(dep_adm, escolaridade) %>% 
  summarise(
    n = n(),
    # Estatísticas de aprovação
    aprov_media = mean(tx_aprovacao, na.rm = TRUE),
    aprov_mediana = median(tx_aprovacao, na.rm = TRUE),
    aprov_dp = sd(tx_aprovacao, na.rm = TRUE),  # desvio padrão
    aprov_min = min(tx_aprovacao, na.rm = TRUE),
    aprov_max = max(tx_aprovacao, na.rm = TRUE),
    # Contando NAs
    n_na = sum(is.na(tx_aprovacao))
  ) %>% 
  arrange(escolaridade, desc(aprov_media))

# Estatísticas detalhadas por tipo e nível:
print(stats_detalhadas, n = 15)
```

## Visualizações: explorando as distribuições

### Histogramas por tipo de escola

Histogramas mostram a frequência de valores em intervalos (bins):

```r
# Histograma geral
edu_limpo %>% 
  filter(!is.na(tx_aprovacao)) %>% 
  ggplot(aes(x = tx_aprovacao)) +
  geom_histogram(binwidth = 5, fill = "steelblue", color = "white") +
  facet_grid(escolaridade ~ dep_adm) +  # grade: linhas ~ colunas
  labs(
    title = "Distribuição das Taxas de Aprovação",
    subtitle = "Por tipo de escola e nível de escolaridade",
    x = "Taxa de Aprovação (%)",
    y = "Frequência"
  ) +
  theme_minimal()
```

### Comparação com boxplots

Boxplots são ótimos para comparar distribuições. Eles mostram:
- **Linha central**: mediana (50% dos dados estão acima/abaixo)
- **Caixa**: 1º quartil (25%) até 3º quartil (75%)
- **Bigodes**: valores mínimo e máximo (excluindo outliers)
- **Pontos**: outliers (valores extremos)

```r
# Boxplots para comparar medianas e dispersão
edu_limpo %>% 
  filter(!is.na(tx_aprovacao)) %>% 
  ggplot(aes(x = dep_adm, y = tx_aprovacao, fill = dep_adm)) +
  geom_boxplot(alpha = 0.7) +
  facet_wrap(~escolaridade) +
  labs(
    title = "Taxa de Aprovação por Tipo de Escola",
    subtitle = "Comparação entre níveis de escolaridade",
    x = "",
    y = "Taxa de Aprovação (%)"
  ) +
  theme_minimal() +
  theme(legend.position = "none")  # remove legenda redundante
```

### Densidade para comparar distribuições

Gráficos de densidade são como histogramas "suavizados". São úteis para comparar formas de distribuições:

```r
# Gráfico de densidade - mais suave que histograma
edu_limpo %>% 
  filter(!is.na(tx_aprovacao),
         escolaridade == "Médio") %>% 
  ggplot(aes(x = tx_aprovacao, fill = dep_adm)) +
  geom_density(alpha = 0.6) +  # alpha controla transparência
  labs(
    title = "Distribuição das Taxas de Aprovação - Ensino Médio",
    subtitle = "Comparação entre tipos de escola",
    x = "Taxa de Aprovação (%)",
    y = "Densidade",
    fill = "Tipo de Escola"
  ) +
  theme_minimal() +
  scale_fill_brewer(palette = "Set2")  # paleta de cores mais bonita
```

## Análise regional

```r
# Performance por região administrativa
regional_stats <- edu_limpo %>% 
  filter(!is.na(reg_administrativa)) %>% 
  group_by(reg_administrativa, dep_adm, escolaridade) %>% 
  summarise(
    n_municipios = n_distinct(municipios),
    media_aprov = mean(tx_aprovacao, na.rm = TRUE),
    media_reprov = mean(tx_reprovacao, na.rm = TRUE),
    media_aband = mean(tx_abandono, na.rm = TRUE),
    .groups = "drop"  # evita warning sobre agrupamento
  ) %>% 
  filter(n_municipios >= 3)  # Regiões com pelo menos 3 municípios para ser representativo

# Top 10 regiões - Ensino Médio Estadual:
regional_stats %>% 
  filter(escolaridade == "Médio",
         dep_adm == "Estadual") %>% 
  arrange(desc(media_aprov)) %>% 
  head(10)

# Visualização regional
regional_stats %>% 
  filter(escolaridade == "Médio") %>% 
  ggplot(aes(x = reorder(reg_administrativa, media_aprov),  # reorder ordena por média
             y = media_aprov, 
             fill = dep_adm)) +
  geom_col(position = "dodge") +  # position dodge coloca barras lado a lado
  coord_flip() +  # inverte eixos para melhor leitura
  labs(
    title = "Taxa de Aprovação por Região - Ensino Médio",
    x = "Região Administrativa",
    y = "Taxa Média de Aprovação (%)",
    fill = "Tipo"
  ) +
  theme_minimal()
```

## Transformando dados: formato largo para longo

Para algumas análises, precisamos reorganizar os dados. O formato "longo" tem uma linha para cada observação:

```r
# pivot_longer "empilha" colunas
edu_long <- edu_limpo %>% 
  pivot_longer(
    cols = c(tx_aprovacao, tx_reprovacao, tx_abandono),
    names_to = "tipo_taxa",
    values_to = "valor"
  ) %>% 
  mutate(
    tipo_taxa = recode(tipo_taxa,
      "tx_aprovacao" = "Aprovação",
      "tx_reprovacao" = "Reprovação", 
      "tx_abandono" = "Abandono"
    )
  )

# Visualização com dados em formato longo
edu_long %>% 
  filter(!is.na(valor)) %>% 
  group_by(escolaridade, dep_adm, tipo_taxa) %>% 
  summarise(media = mean(valor), .groups = "drop") %>% 
  ggplot(aes(x = tipo_taxa, y = media, fill = dep_adm)) +
  geom_col(position = "dodge") +
  facet_wrap(~escolaridade) +
  labs(
    title = "Médias das Taxas por Tipo e Nível",
    x = "",
    y = "Taxa Média (%)",
    fill = "Tipo de Escola"
  ) +
  theme_minimal() +
  theme(axis.text.x = element_text(angle = 45, hjust = 1))  # rotaciona texto do eixo x
```

## Análise de correlações

Correlação mede a relação linear entre duas variáveis, variando de -1 a 1:
- **1**: correlação positiva perfeita 
- **0**: sem correlação linear
- **-1**: correlação negativa perfeita

```r
# Correlações entre as taxas por tipo de escola
correlacoes <- edu_limpo %>% 
  filter(!is.na(tx_aprovacao) & !is.na(tx_reprovacao) & !is.na(tx_abandono)) %>% 
  group_by(dep_adm) %>% 
  summarise(
    n = n(),
    cor_aprov_reprov = cor(tx_aprovacao, tx_reprovacao),
    cor_aprov_aband = cor(tx_aprovacao, tx_abandono),
    cor_reprov_aband = cor(tx_reprovacao, tx_abandono)
  )

# Correlações entre taxas por tipo de escola:
print(correlacoes)

# Visualização de correlação
edu_limpo %>% 
  filter(dep_adm == "Estadual",
         escolaridade == "Médio",
         !is.na(tx_reprovacao) & !is.na(tx_abandono)) %>% 
  ggplot(aes(x = tx_reprovacao, y = tx_abandono)) +
  geom_point(alpha = 0.5) +
  geom_smooth(method = "lm", se = FALSE, color = "red") +  # linha de regressão
  labs(
    title = "Relação entre Reprovação e Abandono",
    subtitle = "Escolas Estaduais - Ensino Médio",
    x = "Taxa de Reprovação (%)",
    y = "Taxa de Abandono (%)"
  ) +
  theme_minimal()
```

## Comparações interessantes

### Municípios onde escolas municipais superam estaduais

```r
# Comparando desempenho municipal vs estadual
# pivot_wider "espalha" dados de formato longo para largo
comparacao <- edu_limpo %>% 
  filter(escolaridade == "Fundamental") %>% 
  select(municipios, dep_adm, tx_aprovacao) %>% 
  pivot_wider(names_from = dep_adm, values_from = tx_aprovacao) %>% 
  filter(!is.na(Estadual) & !is.na(Municipal)) %>%  # só municípios com ambos os tipos
  mutate(diferenca = Municipal - Estadual) %>% 
  arrange(desc(diferenca))

# Top 10 municípios onde escolas municipais têm melhor desempenho:
head(comparacao, 10)

# Visualização das diferenças
comparacao %>% 
  ggplot(aes(x = Estadual, y = Municipal)) +
  geom_point(alpha = 0.5) +
  geom_abline(intercept = 0, slope = 1, color = "red", linetype = "dashed") +  # linha de igualdade
  labs(
    title = "Comparação: Escolas Municipais vs Estaduais",
    subtitle = "Ensino Fundamental - Linha vermelha indica igualdade",
    x = "Taxa de Aprovação - Estadual (%)",
    y = "Taxa de Aprovação - Municipal (%)"
  ) +
  theme_minimal()
```

## Modelagem simples

Regressão linear nos ajuda a entender como variáveis se relacionam. O modelo `lm()` estima como a taxa de aprovação varia com tipo de escola e escolaridade:

```r
# Como o tipo de escola afeta as taxas?
# y ~ x1 + x2 + x1:x2 significa: y depende de x1, x2 e da interação entre eles
modelo <- lm(tx_aprovacao ~ dep_adm + escolaridade + dep_adm:escolaridade, 
             data = edu_limpo)

# summary() mostra os coeficientes e significância
summary(modelo)

# Interpretação simplificada:
# - Coeficientes positivos: aumentam a taxa de aprovação
# - Coeficientes negativos: diminuem a taxa de aprovação
# - Valores p < 0.05: efeito estatisticamente significativo

# Visualizando as previsões do modelo
edu_limpo %>% 
  filter(!is.na(tx_aprovacao)) %>% 
  mutate(previsto = predict(modelo, .)) %>% 
  ggplot(aes(x = tx_aprovacao, y = previsto, color = dep_adm)) +
  geom_point(alpha = 0.3) +
  geom_abline(intercept = 0, slope = 1, color = "black", linetype = "dashed") +
  facet_wrap(~escolaridade) +
  labs(
    title = "Valores Observados vs Previstos",
    subtitle = "Pontos próximos à linha = modelo prevê bem",
    x = "Taxa de Aprovação Observada (%)",
    y = "Taxa de Aprovação Prevista (%)",
    color = "Tipo"
  ) +
  theme_minimal()
```

## Exercícios práticos

1. **Investigação de municípios**: 
   - Identifique municípios com 100% de aprovação. São casos reais ou possíveis erros?
   - Encontre municípios que tenham os três tipos de esfera administrativa (escola pública municipal, escola pública estadual, escola privada) nos dois níveis de ensino disponíveis (Fundamental, Médio).
   - Encontre municípios onde escolas privadas têm desempenho pior que públicas.

2. **Análise por porte de município**:
   - Crie categorias de tamanho baseadas no número de tipos de escola
   - Compare o desempenho entre municípios grandes e pequenos

3. **Análise de equidade**:
   - Calcule a diferença entre escolas privadas e públicas por município
   - Onde a desigualdade é maior?

## Conclusão: lições sobre dados reais

Neste tutorial, aprendemos várias lições importantes sobre trabalhar com dados reais:

### 1. **Estrutura é fundamental**
O maior desafio não foram valores missing ou pequenas inconsistências, mas entender que os dados vinham pré-agregados em múltiplos níveis. Ignorar isso levaria a análises completamente erradas!

### 2. **Problemas técnicos são comuns**
- Encoding incorreto (Latin-1 vs UTF-8)
- Formato numérico brasileiro (vírgulas)
- Códigos compostos que precisam parsing

### 3. **Valores missing têm significado**
NAs muitas vezes indicam que aquele tipo de escola não existe no município - uma informação importante por si só!

### 4. **Contexto importa**
Comparar escolas privadas de São Paulo com as de municípios pequenos pode ser enganoso sem considerar o contexto socioeconômico.

### 5. **Documentação é essencial**
Sempre documente:
- Que filtros foram aplicados
- Por que certas decisões foram tomadas
- Quantos registros foram afetados

