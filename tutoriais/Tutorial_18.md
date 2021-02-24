# Tutorial 18: Modelos de análise de texto

## Carregando pacotes

Vamos utilizar a função de carregar os pacotes que criamos no Tutorial 17:

```{r}
inst_load <- function(req_pack)
{
  remain_pack <- req_pack[!(req_pack %in% installed.packages()[,"Package"])];
  
  if(length(remain_pack)) 
  {
    install.packages(remain_pack);
  }
  for(package_name in req_pack)
  {
    library(package_name,character.only=TRUE,quietly=TRUE);
  }
}
```

Definir os pacotes que queremos carregar:

```{r}
packages_to_load <- c("quanteda", "quanteda.textmodels", "dplyr", "stringr", "readr", "ggplot2", "viridis", "stm")
```

E chamar por eles na biblioteca:

```{r}
inst_load(packages_to_load)
```

## Carregando os dados

Faremos o mesmo processo do Tutorial 17, carregando os dados que capturamos no DOU, criar a variável de data, criar nosso corpus base, atribuir os metadados com a função *docvars* e criar um segundo corpus com uma amostra apenas para o ano de 2021:

```{r, message=FALSE, warning=FALSE}
dados_dou <- read_csv("https://raw.githubusercontent.com/thiagomeireles/cebrap_afro_2021/main/data/dados_dou.csv",
                      col_types = cols(data = col_date(format = "%d/%m/%Y"))) %>% 
  mutate(ano = format(data, "%Y"))

corpus_dou <- corpus(dados_dou$texto)

docvars(corpus_dou, "titulo") <- dados_dou$titulo
docvars(corpus_dou, "data")   <- dados_dou$data
docvars(corpus_dou, "edicao") <- dados_dou$edicao
docvars(corpus_dou, "ano")    <- dados_dou$ano

corpus_dou_2021 <- corpus_subset(corpus_dou, ano == 2021)
```

## Análise dos textos

Antes de criar nossa Matriz Documento-Recurso, vamos recriar nosso vetor de *stopwords*:

```{r}
stopwords_pt <- c(stopwords("pt"), "nº", "n", "º",  "conceição", "dias", "cnpj", "josé", "santa", "art", "processo", "portaria", "+", "r", "$", "1°", "0001-04","1524422177k660001", "ug", "°", "diretor", "a")
```

### Legibilidade dos textos

Um primeiro passo que faremos e é bastante importante é testar a legibilidade dos textos com a função [*textstat_readability*](https://www.rdocumentation.org/packages/quanteda/versions/1.0.0/topics/textstat_readability).

```{r}
head(textstat_readability(corpus_dou_2021),2)
```

De forma mais simples, quanto maior o score na escala de Flesch, mais fácil a leitura do texto. No caso dos dois primeiros que utilizamos como exemplo, a leitura é bastante complexa. Um texto de fácil compreensão possui score próximo a 80. Existem outras medidas que são apresentadas na documentação do link acima.

### Frequência dos termos

Aqui um primeiro passo é criar uma Matriz Documento-Recurso removendo as *stopwords*:

```{r}
dfm_dou_2021 <- dfm(corpus_dou_2021,
                    remove = stopwords_pt,
                    stem = FALSE, 
                    remove_punct = TRUE,
                    remove_numbers = TRUE) 
```

Para observar a frequência de textos o pacote *quanteda* oferece a função [*textstat_frequency*](https://www.rdocumentation.org/packages/quanteda/versions/2.1.2/topics/textstat_frequency). Vamos fazer um gráfico com apenas os 20 termos mais frequentes, usando a função *filter* a partir da variável `rank`. A partir disso, utilizamos o ggplot, reordenando o recurso (feature) no eixo do menos para o mais frequente, utilizando a frequência no eixo y e preenchendo com o próprio recurso (diferenciação de cores entre cada um). Mais uma vez utilizamos o *geom_bar* com a opção *stat* definida como `identity` porque fornecemos os valores do eixo y. A função *scale_fill_viridis_d* utiliza uma paleta de cores direcionada para daltônicos (isso é interessante saber, não?) e o "d" no final indica que é a opção para variáveis discretas. A função *coord_flip* troca o eixo x e y de posição. Por fim, de novidade, temos a *legend.position* como "none" para retirar a legenda e alteramos o tamanho da fonte do eixo "x" com a *asis.text*.

```{r}
textstat_frequency(dfm_dou_2021) %>% 
  filter(rank<=20) %>% 
  ggplot(aes(x = reorder(feature, rank), y = frequency, fill = feature)) +
  geom_bar(stat="identity") +
  scale_fill_viridis_d() +             
  xlab("Termos") + ylab("Frequência") +
  coord_flip() +
  theme_bw() + 
  theme(legend.position = "none",          # Remove a legenda
        axis.text=element_text(size=10))   # Altera o tamanho da fonte
```

### Análise de n-gramas

Antes de fazer a análise, criaremos um novo objeto da classe *token* a partir do `corpus_dou_2021` removendo as pontuações e os símbolos. A partir do *pipe* faremos duas etapas de remoção: na primeira removeremos as *stopwords* (aqui especificadas no padrão) e na segunda os números a partir de uma [expressão regular](https://rstudio.com/wp-content/uploads/2016/09/RegExCheatsheet.pdf).

```{r}
tokens_publicacoes <- tokens(corpus_dou_2021,
                             remove_punct = TRUE,
                             remove_symbols = TRUE,
                             padding = TRUE) %>% 
  tokens_remove(pattern = c(stopwords("pt"), "nº", "n", "º",  "conceição", "dias", "cnpj", "josé", "santa", "art", "processo", "portaria", "+", "r", "$", "1°", "0001-04","1524422177k660001", "ug", "°", "diretor", "a"), padding = TRUE) %>%
  tokens_remove(pattern = '[[:digit:]]', valuetype = "regex", padding = TRUE)
```

O pacote *quanteda* oferece a função [*textstat_collocations*](https://www.rdocumentation.org/packages/quanteda/versions/2.1.2/topics/textstat_collocations) para análise de bigramas e n-gramas. Aqui utilizaremos o objeto *token* criado trabalhando com n-gramas de 2 a 4 termos (opção *size*). A partir disso, selecionaremos as 20 combinações mais frequentes para fazermos um gráfico. No ggplot, na aesthetics utilizaremos a variável dos n-gramas, `collocation`, no eixo x de forma ordenada, e a contagem, `count`, no eixo y. Mais uma vez criamos o gráfico de barras preenchendo mais uma vez com os n-gramas. As outras opções são as mesmas utilizadas no gráfico anterior.

```{r}
textstat_collocations(tokens_publicacoes, size = 2:4, tolower = TRUE) %>%
  slice_max(count, n = 20) %>% 
  ggplot(aes(x=reorder(collocation, count), y = count)) +
  geom_bar(stat = "identity", aes(fill=collocation)) +
  scale_fill_viridis_d() +
  xlab("Termos") + ylab("Frequência") +
  coord_flip() +
  theme_bw() + 
  theme(legend.position = "none", axis.text=element_text(size=10))
```

### Diferença entre textos

Primeiro criaremos uma nova matriz com grupos direcionados a cada um dos anos a partir do corpus completo:

```{r}
dfm_dou <- dfm(corpus_dou,
               groups='ano',
               remove = stopwords_pt,
               stem = FALSE, 
               remove_punct = TRUE,
               remove_numbers = TRUE)
```

Para testar as diferenças entre os grupos de textos, utilizamos a função *textstat_keyness* que utiliza a técnica chamada [Keyness](http://www.thegrammarlab.com/?nor-portfolio=understanding-keyness) para identificar a diferença entre dois corpus ou elementos dentro de um mesmo corpus. Aqui utilizaremos os anos para diferenciar, tendo 2021 como alvo da análise (ou seja, comparando com 2020).

```{r}
tstatkeyness <- textstat_keyness(dfm_dou, target = '2021')
head(tstatkeyness,15)
```

É importante entender que os resultados são sensíveis ao número de obserações tanto para as estatística de chi-quadrado como p-valor, quanto maior os corpus, maior a chance de encontrarmos resultados estatisticamente significantes.

Vamos dar uma olhada no gráfico gerado a partir do resultado obtido com a função *textplot_keyness* selecionando apenas os 15 casos de maior valor:

```{r}
textplot_keyness(tstatkeyness, n = 15)
```

### Semelhança entre textos

Para ver a semelhança entre os textos, uma matriz agrupada pelas edições das publicações.

```{r}
dfm_dou <- dfm(corpus_dou,
               groups='edicao',
               remove = stopwords_pt,
               stem = FALSE, 
               remove_punct = TRUE,
               remove_numbers = TRUE)
```

Agora realizamos o teste de similaridade aplicado a essas pubicações com o [método de cosseno](https://sites.temple.edu/tudsc/2017/03/30/measuring-similarity-between-texts-in-python/#:~:text=The%20cosine%20similarity%20is%20the,the%20similarity%20between%20two%20documents.). Caso queiram pesquisar sobre os métodos aplicados, olhem a [documentação da função](https://www.rdocumentation.org/packages/quanteda/versions/2.1.1/topics/textstat_simil). A partir da nossa matriz, vamos comparar as publicações das edições "8" e "14" com as demais e entre elas próprias.

```{r}
similaridade <- textstat_simil(dfm_dou,
                               dfm_dou[c("8", "14"), ],
                               margin = "documents",
                               method = "cosine")

head(similaridade)
```

De forma intuitiva, quanto maior o valor (de 0 a 1), mais semelhantes são os discursos. Podemos observar isso graficamente:

```{r}
dotchart(as.list(similaridade)$"8", xlab = "Similaridade de cosseno (Edição 8)", pch = 19)
```

Outra possibilidade é a utilização dessas distâncias para produzir um dendograma clusterizando as edições. Utilizaremos  dfm com as publicações agrupadas pelas edições. A partir dela, utilizamos a funçao `dfm_trim()` para selecionar os recursos a partir de um "corte" na frequência. Aqui pegamos somente termos que apareçam ao menos 5 vezes e em três documentos. 

```{r, eval = FALSE}
dfm_arvore <- dfm_subset(dfm_dou,
                         !is.na(edicao))

trimDfm <- dfm_trim(dfm_arvore, min_termfreq = 5, min_docfreq = 3)

trimDfm
```

Agora obtemos as distâncias da *dfm* normalizadas.

```{r}
tstat_dist <- textstat_dist(dfm_weight(trimDfm, scheme = "prop"))
```

Em seguida, realizamos a clusterização hirárquica do objeto da distância:

```{r}
cluster_edicao <- hclust(as.dist(tstat_dist))
```

E atribuímos rótulos com os nomes das edições:

```{r}
cluster_edicao$labels <- dfm_arvore$edicao
```

Por fim, plotamos o dendograma.

```{r, fig.width = 8, fig.height = 5}
plot(cluster_edicao, ylab = "", xlab = "", sub = "",
     main = "Distância Euclidiana da frequência de tokens normalizada")
```

Perceba que é como uma "árvore" em que é possível diferenciar quais "ramos" estão próximos ou não. 

### Posições de Documentos

Aqui temos um exemplo de dimensionamento da posição de documentos não supervisionada utilizando nosso corpus completo com o Modelo "Wordfish". Para exemplos de modelagem de textos observem a [Documentação do Pacote *quanteda.textmodels*](https://www.rdocumentation.org/packages/quanteda.textmodels/versions/0.9.1). 

```{r}
wfm <- textmodel_wordfish(dfm_dou)
textplot_scale1d(wfm, margin = "documents")
```

Também é possível fazer o gráfico a partir dos recursos, sendo possível destacar alguns que nos interessem. 

```{r}
textplot_scale1d(wfm, margin = "features",
                 highlighted = c("quilombo", "cultura", "indígena",
                                 "financiamento", "recurso"))
```

### Dispersão de palavras chave

Com o comando [*texplot_xray*](https://www.rdocumentation.org/packages/quanteda/versions/2.1.2/topics/textplot_xray) conseguimos selecionar padrões de palavras ao longo de um ou mais textos. Com isso, o formato do gráfico depende do número de objetos da classe *kwic* que chamamos. Se utilizarmos apenas um documento, as palavras-chave são chamadas uma abaixo da outra; se chamarmos múltiplos documentos, eles que são colocados um abaixo do outro e as palavras ficam lado a a lado. Aqui chamaremos por 5 palabras chave utilizando os primeiros 15 textos do corpus das publicações de 2021:

```{r}
textplot_xray(kwic(corpus_dou_2021[1:15], "quilombo"), 
              kwic(corpus_dou_2021[1:15], "cultura"),
              kwic(corpus_dou_2021[1:15], "indígena"),
              kwic(corpus_dou_2021[1:15], "financiamento"),
              kwic(corpus_dou_2021[1:15], "recurso"))
```
### Diversidade léxica

Aqui veremos como observar a [diversidade léxica ou complexidade dos textos](https://textinspector.com/help/lexical-diversity/). Com a função *textstat_lexdiv* acessamos nossa matriz recurso para ver quantas palavras lexicalmente diferentes temos nos textos. Aqui mantivemos todas as medidas possíveis pela função, pode explorar um pouco mais sobre elas acessando sua [documentação](https://www.rdocumentation.org/packages/quanteda/versions/1.3.4/topics/textstat_lexdiv).

```{r}
head(textstat_lexdiv(dfm_dou_2021, measure = "all" ,drop=T) )
```

### Conexões entre os termos (redes de palavras)

Para criar nossa rede de palavras, o primeiro passo é identificar quais são os recursos que queremos, vamos obter 40 e extrair seus nomes:

```{r}
top_feat <- names(topfeatures(dfm_dou_2021, 40))
head(top_feat)
```

Em um segundo passo, criaremos uma matriz de coocorrência de recursos com a função [*fcm*](https://www.rdocumentation.org/packages/quanteda/versions/2.1.2/topics/fcm) selecionando somente os 40 termos mais comuns que selecionamos no passo anterior.

```{r}
topfeatfcm <- fcm(dfm_dou_2021) %>% 
  fcm_select(., top_feat)
```

Por fim, criamos nosso gráfico de redes utilizando a função [*textplot_network*](https://www.rdocumentation.org/packages/quanteda/versions/2.1.2/topics/textplot_network) direcionada a redes de recursos coocorrentes. Lembre que o *set.seed* é para tornar replicável, enquanto as opções do gráfico dizem respeito à proporção de coocorrências (*min_freq*), a opacidade das ligações (*edge_alpha*) e o tamanho dos principais recursos coocorrentes (*edge_size*).

```{r}
set.seed(1234)
textplot_network(topfeatfcm, min_freq = 0.1, edge_alpha = 0.2, edge_size = 2)
```

### Topic models

O *quanteda* oferece boas ferrametnas para *topic models*. Aqui retomamos os dados do DOU utilizando apenas o subconjunto de 2021. Assim como fizemos anteriormente, aplicaremos a função `dfm_trim()` para realizar "cortes" nos nossos recursos. 

```{r}
quant_dfm <- dfm_trim(dfm_dou_2021, min_termfreq = 3, min_docfreq = 5)

quant_dfm
```

Agora podemos estimar um [*Structural Topic Model*](https://www.rdocumentation.org/packages/stm/versions/1.3.6/topics/stm) com o pacote `stm` e a função `stm()`. Vamos utilizar um exemplo de 10 tópicos, representado na opção `K` da função. Utilizamos o valor `FALSE` na opção `verbose` para que não imprima na tela as iterações realizadas durante os cálculos para modelagem realizados pela a função - a opção padrão é `TRUE`, a utilize se quiser observar cada um dos passos no console do seu RStudio.

```{r fig.width = 7, fig.height = 5}
my_lda_fit15 <- stm(quant_dfm, K = 15, verbose = FALSE)
plot(my_lda_fit15, main = "Principais Tópicos", xlab = "Proporções esperadas dos tópicos")    

```

## Terminamos(?)

Bem, espero que durante o curso tenham aproveitado bastante o conteúdo. O que vimos aqui fornece um bom conjunto de ferramentas para começar a explorar o mundo da análise quantitativa de texto e agora espero que sigam nessa explorando esse universo cada vez mais. Aqui nós vimos muito mais o aspecto de execução dos métodos, mas mais importante agora é identificar quais são os métodos de interesse e partir para leituras que auxiliem a entender melhor como utilizá-los.