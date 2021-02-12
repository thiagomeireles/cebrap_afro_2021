# Tutorial 15 - Análise de texto no R - pacote _tidytext_

Caso esteja em uma nova seção, carregue o nosso banco do DOU e extraia o vetor com os textos como fizemos no tutorial passado:

```{r}
library(readr)

dados_dou <- read_csv("https://raw.githubusercontent.com/thiagomeireles/cebrap_afro_2021/main/data/dados_dou.csv",
                      col_types = cols(data = col_date(format = "%d/%m/%Y")))

publicacoes <- dados_dou$texto
```

## Uma abordagem "tidy" para texto

Corpora são os objetos clássicos para processamento de linguagem natural. No R, porém, há uma tendência a deixar tudo "tidy". Vamos ver uma abordagem "tidy", ou seja, com data frames no padrão do _tidyverse_, para texto.

Vamos fazer uma rápida introdução, mas recomendo fortemente a leitura do livro [Text Mininig with R](http://tidytextmining.com/), disponível o formato "bookdown".

Comecemos carregando os seguintes pacotes e, caso ainda não tenha algum deles instalado, o faça.

```{r}
library(tidytext)
library(dplyr)
library(ggplot2)
library(tidyr)
```

Vamos recriar o data frame com as publicações:

```{r}
publicacoes_df <- tibble(doc_id = as.character(1:length(publicacoes)), 
                         text = publicacoes)
glimpse(publicacoes_df)
```

### Tokens

A primeira função interessante do pacote _tidytext_ é justamente a tokenização de um texto:

```{r}
publicacoes_token <- publicacoes_df %>%
  unnest_tokens(word, text)
glimpse(publicacoes_token)
```

Note que a variável "_id\_publicacao", criada por nós, é mantida. "text", porém, se torna "words", na exata sequência do texto. Veja que o formato de um "tidytext" é completamnte diferente de um Corpus.

Como excluir stopwords nessa abordagem? Precisamos de um data frame com stopwords. Vamos recriar o vetor stopwords_pt utilizado no tutorial anterior, que é a versão ampliada das stopwords disponíveis no R, e criar um data frame com tal vetor:

```{r}
stopwords_pt <- c(stopwords("pt"), "nº", "n", "º",  "conceição", "dias", "cnpj", "josé", "santa", "art", "processo", "portaria")
stopwords_pt_df <- data.frame(word = stopwords_pt)
```

Com _anti\_join_ (lembra dessa função?) mantemos em "publicacoes\_token" apenas as palavras que não estao em "stopwords\_pt\_df"

```{r}
publicacoes_token <- publicacoes_token %>%
  anti_join(stopwords_pt_df, by = "word")
```

Vamos utilizar uma expressão regular para eliminar os números que permaneceram para observar a integração entre o *dplyr* e as expressões regulares no *stringr*. Você pode acessar esse [guia tidy](https://stringr.tidyverse.org/articles/regular-expressions.html) para expressões regulares.

```{r}
publicacoes_token <- publicacoes_token %>% 
  filter(!grepl("[0-9]", word))
```


Para observarmos a frequência de palavras nos publicacoes, usamos _count_, do pacote _dplyr_:

```{r}
publicacoes_token %>%
  count(word, sort = TRUE)
```

Com _ggplot_, podemos construir um gráfico de barras dos temos mais frequêntes, por exemplo, com frequência maior do que 800. Neste ponto do curso, nada do que estamos fazendo abaixo deve ser novo a você:

```{r}
publicacoes_token %>%
  count(word, sort = TRUE) %>%
  filter(n > 800) %>%
  mutate(word = reorder(word, n)) %>%
  ggplot(aes(word, n)) +
    geom_col() +
    xlab(NULL) +
    coord_flip()
```

Incorporando a função _wordcloud_ a nossa análise:

```{r}
publicacoes_token %>%
  count(word, sort = TRUE) %>%
  with(wordcloud(word, n, max.words = 100))
```

Vemos que alguns termos não fazem muito sentido ainda, não? Algumas das palavras poderiam estar nas *stopwords*. Para se divertir e entender melhor o funcionamento, altere essas palavras e os parâmetros dos gráficos que produzimos quando tiver um pouco de tempo.

O que mais importa aqui é que a abordagem "tidy" para texto nos mantém no território confortável da manipulação de data frames e, particularmente, me parece mais atrativa do que a abordagem via Corpus para um conjunto grande de casos.

### Bigrams

Já produzimos duas vezes a tokenização do texto, sem, no entanto, refletir sobre esse procedimento. Tokens são precisam ser formados por palavras únicas. Se o objetivo for, por exemplo, observar a ocorrência conjunta de termos, convém trabalharmos com bigrams (tokens de 2 palavras) ou ngrams (tokens de n palavras). Vejamos como:

```{r}
publicacao_bigrams <- publicacoes_df %>%
  unnest_tokens(bigram, text, token = "ngrams", n = 2)
```

Note que, ao tokenizar o texto, automaticamente foram excluídas as as pontuações e as palavras foram alteradas para minúscula (use o argumento "to_lower = FALSE" caso não queira a conversão). Vamos contar os bigrams:

```{r}
publicacao_bigrams %>%
  count(bigram, sort = TRUE)
```

Como não aliminamos números ou stopwords, ficou bem estranho, né? Como, porém, excluir as stopwords quando elas ocorrem em bigrams? Em primeiro, temos que separar os bigrams e duas palavras, uma em cada coluna:

```{r}
bigrams_separated <- publicacao_bigrams %>%
  separate(bigram, c("word1", "word2"), sep = " ")
```

E, a seguir, filter o data frame excluindo as stopwords (note que aproveitamos o vetor "stopwords_pt"):

```{r}
bigrams_filtered <- bigrams_separated %>%
  filter(!word1 %in% stopwords_pt,
         !word2 %in% stopwords_pt,
         !grepl("[0-9]", word1),
         !grepl("[0-9]", word2))
```

ou, usando _anti\_join_, como anteriormente:

```{r}
bigrams_filtered <- bigrams_separated %>%
  anti_join(stopwords_pt_df, by = c("word1" = "word")) %>%
  anti_join(stopwords_pt_df, by = c("word2" = "word")) %>% 
  filter(!grepl("[0-9]", word1),
         !grepl("[0-9]", word2))
```

Produzindo a frequência de bigrams:

```{r}
bigram_counts <- bigrams_filtered %>% 
  count(word1, word2, sort = TRUE)
```

Reunindo as palavras do bigram que foram separadas para excluirmos as stopwords:

```{r}
bigrams_united <- bigrams_filtered %>%
  unite(bigram, word1, word2, sep = " ")
```

A abordagem "tidy" traz uma tremenda flexibilidade. Se, por exemplo, quisermos ver com quais palavras a palavra "quilombo" é antecedida:

```{r}
bigrams_filtered %>%
  filter(word2 == "quilombo") %>%
  count(word1, sort = TRUE)
```

Ou precedida:

```{r}
bigrams_filtered %>%
  filter(word1 == "quilombo") %>%
  count(word2, sort = TRUE)
```

Ou ambos:

```{r}
bf1 <- bigrams_filtered %>%
  filter(word2 == "quilombo") %>%
  count(word1, sort = TRUE) %>%
  rename(word = word1)

bf2 <- bigrams_filtered %>%
  filter(word1 == "quilombo") %>%
  count(word2, sort = TRUE) %>%
  rename(word = word2)

bind_rows(bf1, bf2) %>%
  arrange(-n)
```

Super simples, não? Mesmo que o resultado não pareça ter muito significado, podemos criar uma hipótese de que os resultados para "quilombo" nos dizem muito pouco sobre políticas direcionadas e, em grande parte, estão relacionados a endereços. Isso também mostra que é super legal! rs

### Ngrams

Repetindo o procedimento para "trigrams":

```{r}
publicacoes_df %>%
  mutate(text = removeNumbers(text)) %>% 
  unnest_tokens(trigram, text, token = "ngrams", n = 3) %>%
  separate(trigram, c("word1", "word2", "word3"), sep = " ") %>%
  anti_join(stopwords_pt_df, by = c("word1" = "word")) %>%
  anti_join(stopwords_pt_df, by = c("word2" = "word")) %>%
  anti_join(stopwords_pt_df, by = c("word3" = "word")) %>%
  count(word1, word2, word3, sort = TRUE)
```

"ug gestão programa" é o "trigram" mais frequente nas publicacoes do DOU com o termo "quilombo", seguido por "caixa econômica federal". Agora podemos formular novas hipóteses? O que você acha?

### Redes de palavras

Para encerrar, vamos a um dos usos mais interessantes do ngrams: a construção de redes de palavras. Precisaremos de dois pacotes, _igraph_ e _ggraph_, já utilizados no tutorial do Twitter.

```{r}
library(igraph)
library(ggraph)
```

Em primeiro lugar, transformaremos nosso data frame em um objeto da classe _igraph_, do pacote de mesmo nome, usado para a presentação de redes no R (lembrem que precisamos criar uma sequência para reprodutivilidade com a função *set.seed*):

```{r}
set.seed(123)

bigram_graph <- bigram_counts %>%
  filter(n > 150) %>%
  graph_from_data_frame()
```

A seguir, com o pacote _ggraph_, faremos o grafo a partir dos bigrams dos publicacoes da deputada:

```{r}
ggraph(bigram_graph, layout = "fr") +
  geom_edge_link() +
  geom_node_point() +
  geom_node_text(aes(label = name), vjust = 1, hjust = 1)
```

Note que são formadas pequenas associações entre termos que, par a par, caminham juntos, diferente dos mapas de redes como o que construímos com o Twitter que mostram as diferentes conexões e possíveis posicionamentos dos usuários. Novamente, não vamos explorar aspectos analíticos da mineração de texto aqui, mas estas associações são informações de grande interessa a depender dos objetivos da análise.