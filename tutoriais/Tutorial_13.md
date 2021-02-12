# Tutorial 13 - Manipulação de texto no R: pacote _stringr_

### Material para o tutorial

Nossa primeira tarefa será obter um conjunto de textos com o qual trabalharemos. Classicamente, tutoriais de R sobre strings e mineração de texto utilizam "corpus" (já veremos o que é isso) de literatura moderna.

Para tornar nosso exemplo mais interessante, vamos recuperar as notícias do DataFolha que utilizamos no Tutorial 4. A primeira ferramenta que veremos neste tutorial é o pacote _stringr_. Além de carregarmos o pacote _strigr_, também utilizaremos os pacotes _rvest_ e _dplyr_. Para abrir o arquivo com os dados que utilizaremos, também utilizaremos o pacote _readr_:

```{r}
library(dplyr)
library(rvest)
library(stringr)
library(readr)
```

Feito isso, vamos utliizar os dados coletados no DOU na busca pelo termo "quilombo". Para não realizarmos o scrap mais uma vez, vamos abrir um ".csv" que está disponível na página do curso no GitHub com a função *read_csv*:

```{r}
dados_dou <- read_csv("https://raw.githubusercontent.com/thiagomeireles/cebrap_afro_2021/main/data/dados_dou.csv",
                      col_types = cols(data = col_date(format = "%d/%m/%Y")))
```

Perceba que importamos (e adaptamos) a variável "data" como do tipo *data*. Perceba que precisamos especificar, também, o padrão da variável como dia, mês e ano.

Aproveitemos para ver algumas funções novas do pacote _stringr_. Várias delas, como veremos, são semelhantes a funções de outros pacotes com as quais já trabalhamos. Há, porém, algumas vantagens ao utilizá-las: bugs e comportamentos inesperados corrigidos, uso do operador "pipe", nomes intuitivos e sequência de argumentos intuitivos.

_str\_replace\_all_ substitui no texto um padrão por outro, respectivamente na sequência de argumentos. Seu uso é semelhante à função _gsub_, mas os argumentos estão em ordem intuitiva. Por exemplo, vamos retornar o url_pesquisa ao que extraímos do site da Imprensa Nacional substituindo a base dos urls ("https://www.in.gov.br") por por "nada" nos url:

```{r}
dados_dou <- dados_dou %>% 
  mutate(url_resultado = str_replace_all(url_pesquisa, "https://www.in.gov.br", ""))

head(dados_dou[, c("url_pesquisa", "url_resultado")])
```

### Funcionalidades do _stringr_

Antes de observar as funcionalidades do pacote *stringr*, vamos extrair o texto das pubicações para um vetor com o qual vamos trabalhar a partir de agora no tutorial.

```{r}
publicacoes <- dados_dou$texto
```

Qual é o tamanho de cada publicação? Vamos aplicar _str\_length_ para descobrir. Seu uso é semelhante ao da função _nchar_:

```{r}
len_publicacoes <- str_length(publicacoes)
len_publicacoes
```

Vamos agora observar quais são os publicacoes que mencionam "indígena" e "cultura". Para tanto, usamos _str\_detect_

```{r}
str_detect(publicacoes, "indígena")
str_detect(publicacoes, "cultura")
```

Poderíamos usar o vetor lógico resultante para gerar um subconjunto dos publicacoes, apenas com aqueles nos quais as palavras "indígena" e "cultura" são mencionadas. Mais simples, porém, é utilizar a função _str\_subset_, que funciona tal qual _str\_detect_, mas resulta num subconjunto em lugar de um vetor lógico:

```{r}
publicacoes_indigena <- str_subset(publicacoes, "indígena")
publicacoes_cultura  <- str_subset(publicacoes, "cultura")
```

Se quisessemos apenas a posição no vetor dos publicacoes que contêm "indígena", _str\_which_ faria o trabalho:

```{r}
str_which(publicacoes, "indígena")
str_which(publicacoes, "cultura")
```

Voltando ao vetor completo, quantas vezes "indígena" é mencionada em cada publicacoes? E cultura? Qual é o máximo de menções a "indígena" em uma única publicação? E cultura

```{r}
str_count(publicacoes, "indígena")
max(str_count(publicacoes, "indígena"))
str_count(publicacoes, "cultura")
max(str_count(publicacoes, "cultura"))
```

Vamos fazer uma substituição nos publicacoes. No lugar de "indígena" colocaremos a expressão "indígena, aracterística que devemos observar,". E no lugar de "cultura", "cultura, além do que diz a Fundação Palmares," Podemos fazer a substituição com _str\_replace_ ou com _str\_replace\_all_. A diferença entre ambas é que _str\_replace_ substitui apenas a primeira ocorrênca encontrada, enquanto _str\_replace\_all_ substitui todas as ocorrências.

```{r}
str_replace(publicacoes_indigena[1:3], "indígena", "indígena, aracterística que devemos observar,")
str_replace_all(publicacoes_indigena[1:3], "indígena", "indígena, aracterística que devemos observar,")

str_replace(publicacoes_cultura[1:3], "cultura", "cultura, além do que diz a Fundação Palmares,")
str_replace_all(publicacoes_cultura[1:3], "cultura", "cultura, além do que diz a Fundação Palmares,")
```

Em vez de substituir, queremos conhecer a posição das ocorrências de "indígena" e de "cultura". Com _str\_locate_ e _str\_locate\_all_, respectivamente para a primeira ocorrência e todas as ocorrências, obtemos a posição de começo e fim do padrão buscado:

```{r}
str_locate(publicacoes_indigena, "indígena")
str_locate_all(publicacoes_indigena, "indígena")

str_locate(publicacoes_cultura, "cultura")
str_locate_all(publicacoes_cultura, "cultura")
```

Finalmente, notemos que os publicacoes começam sempre mais ou menos da mesma forma. Vamos retirar os 100 primeiros caracteres de cada publicação para observá-los. Usamos a função _str\_sub_, semelhante à função _substr_, para extrair um padaço de uma string:

```{r}
str_sub(publicacoes, 1, 130)
```

As posições para extração de exerto podem ser variáveis. Por exemplo, vamos usar "len_publicacoes" que criamos acima para extrair os 50 últimos caracteres de cada publicação:

```{r}
str_sub(publicacoes, (len_publicacoes - 50), len_publicacoes)
```

Infelizmente, não há tempo suficiente para entrarmos neste tutorial em um tema extremamante útil: expressões regulares. Expressões regulares, como podemos deduzir pelo nome, são expressões que nos permite localizar -- e, portanto, substituir, extrair, parear, etc -- sequências de caracteres com determinadas caraterísticas -- por exemplo, "quaisquer caracteres entre parênteses", ou "qualquer sequência entre espaços que comece com 3 letras e termine com 4 números" (placa de automóvel).

Você pode ler um pouco sobre expressões regulares no R [aqui](https://rstudio-pubs-static.s3.amazonaws.com/74603_76cd14d5983f47408fdf0b323550b846.html) em aula se terminar os três tutoriais de hoje. Com o uso de expressões regulares, outros dois pares de funções são bastante úteis _str\_extract_, _str\_extract\_all_, _str\_match_ e _str\_match\_all_.

## Nuvem de Palavras

Com a função _wordcloud_ do pacote de mesmo nome, podemos rapidamente visualizar as palavras discursadas tendo o tamanho como função da frequência (vamos limitar a 50 palavras):

```{r}
# install.packages(c("slam", "tm", "wordcloud")) 
# Retirem o # para instalar

library(wordcloud)
wordcloud(publicacoes, max.words = 50)
wordcloud(publicacoes_cultura, max.words = 50)
wordcloud(publicacoes_indigena, max.words = 50)
```

Não muito bonitas e com diversas palavras que não fazem muito sentido, não? Voltaremos a fazer nuvem de palavras depois de aprendermos outras maneiras de trabalharmos o texto como dado no R.