# Tutorial 12: Twitter

Este tutorial é um guia sobre como obter dados diretamente da API do Twitter.

## Twitter Apps

Diferente do que fizemos até agora, o Twitter possui uma maior facilidade para capturar dados devido à possibilidade de acesso via API. Em outras palavras, não precisamos capturar o código de HTML da página do Twitter como fizemos até agora. Para obtermos os dados sobre tweets, usários ou outros aspectos, basta enviarmos uma *request* específica pela API para recevermos de volta dos dados que queremos.

No entanto, como conversamos, antes de enviar as requisições de dados precisamos informar quem somos e conectarmos à API. Para isso, é preciso possuir uma conta no Twitter. A partir dela, precisamos "criar um app" para estabelecermos a conexão a partir de uma [conta de desenvolvedor](https://apps.twitter.com/) e selecionar "Create New App" nos "Standalone Apps".

É necessário preencher os dados desse app, incluindo um "website" que pode ser uma página pessoal ou seu github. Para um bom funcionamento da conexão preencha o campo "Callback URL" com "http://127.0.0.1:1410".

Ao retonrar para a página inicial do seu Twitter Apps, selecione seu app e clique na engrenagem de configurações. Na nova página, teremos uma opção de "Keys and tokens", logo abaixo do nome do app, que precisamos acessar. Clique em "regenerate" para a opção "API key & secret" e para a opção "Access token & secret". Precisaremos dessas quatro informações para estabelecer uma conexão com o Twitter pelo R.

Pacote rtweet - preparando o ambiente

Algumas das funções que vamos utilizar nesta atividade não estão na biblioteca básica do R e do *dplyr*. Outras fazem parte de pacotes que são novos para nós. O primeiro pacote é o próprio *rtweet* com o qual nos conectamos e fazemos as solicitações ao Twitter. Além dele, utilizaremos o pacote [*sf*](https://www.rdocumentation.org/packages/sf/versions/0.9-7) para manipulação de dados de mapas e a geração de gráficos. Caso utilize Linux precisará instalar algumas dependências via terminal detalhados [neste link](https://github.com/r-spatial/sf#linux). Caso utilize o MacOS, também precisará, mas basta seguir [estes passos](https://github.com/r-spatial/sf#macos)

Para requisitar os *shapefiles* (arquivos com informações geográficas para a produção de mapas, de forma grosseira) desenvolvidos pelo IBGE utilizaremos o pacote [*geobr*](https://www.rdocumentation.org/packages/geobr/versions/1.0) desenvolvido pelo IPEA que torna nossa missão bastante mais fácil. Para a produção de gráficos, utilizaremos três outros pacotes. O primeiro é o [*ggplot2*](https://www.rdocumentation.org/packages/ggplot2/versions/3.3.3), parte do pacote *tidyverse*, que permite a produçao de gráficos de forma intuitiva e com diversas opções de edição. E, por fim, utilizaremos os pacotes [*igraph*](https://www.rdocumentation.org/packages/igraph/versions/0.3.1ggra) e [*ggraph*](https://www.rdocumentation.org/packages/ggraph/versions/2.0.4) que nos oferecem opções para produção de melhores gráficos de análise de redes.

Vamos instalar as bibliotecas desses pacotes:

```{r}
install.packages("rtweet")
install.packages("sf")
install.packages("geobr")
install.packages("ggplot2")
install.packages("igraph")
install.packages("ggraph")
```

Com todas as bibliotecas instaladas, precisamos carregar as funções disponíveis carregando as bibliotecas:

```{r}
library(rtweet)
library(dplyr)
library(sf)
library(geobr)
library(ggplot2)
library(igraph)
library(ggraph)
```

## Estabelecendo uma conexão com o Twitter pelo R

Para conectar-se ao Twitter via R, utilizaremos a função *create_token*. Ela recebe como parâmetros o nome do app e as "chaves" que você mencionamos na página do App que você criou: "Consumer Key (API Key)", "Consumer Secret (API Secret)", "Access Token" e "Access Token Secret". É conveniente armazenar ambas em objetos de texto, como abaixo:

```{r}
## nome do app que criou
appname <- "scrap_cebrap"
## api key (substitua pela sua)
api_key <- "ekBWZTNFMTEPvy4JRDuEsrO6Y"
## api secret (substitua pela sua)
api_secret_key <- "2A6mMwifPtCVcKqdvPGlQ1K0IHCtLgKJG9b1xFge4bYWm8TmPn"
## token api key (substitua pela sua)
access_token <- "28703537-aykSMztVZ1epTFgopEEhGHPtnSFJ7IvUsdjwNzDWW"
#token api key secret (substitua pela sua)
access_token_secret <- "35fgbt0sdHwxPEJbs9P89rW6oRsnqy1T6CaxLLq3LarEa"
```

Agora utilizaremos a função *create_token* para nos conectarmos ao Twitter pelo R:

```{r}
twitter_token <- create_token(
  app = appname,
  consumer_key = api_key,
  consumer_secret = api_secret_key,
  access_token = access_token,
  access_secret = access_token_secret)
```

Uma vez criado o token, não é preciso repetir a operação nas próximas vezes que iniciar umas sessão do R. Basta utilizar a função *get_token* que ela é recuperada:

```{r}
get_token()
```

## Procurando tweets com #hashtags e termos de busca

Um dos possíveis interesses é procurar hashtags ou termos específicos. Para isso, utilizamos a função *search_tweets* que realiza a tarefa de forma simples. Em uma primeira tentativa, vamos buscar os 200 últimos tweets do termo "quilombo". Por padrão, o argumento *type* tem "recent" como atributo, mas também podemos aplicar recent "mixed" e "popular". Não vamos alterar durante o tutorial.

```{r}
tweets_quilombo <- search_tweets("quilombo",
                                 n = 200)
```

O resultado da busca é um *data_frame* com 90 colunas sobre os 200 tweets. Vamos utilizar a função *str* para explorar o que recebemos:

```{r}
class(tweets_quilombo)

str(tweets_quilombo)
```

Se examinamos com cuidado, vemos que o *data_frame* contém elementos bastante específicos, com diversas listas.

Vamos utilizar a função *names* para obeservar o nome das variáveis que recebemos:

```{r}
names(tweets_quilombo)
```


Agora sabemos que temos diversas informações que nos interessam, tal como texto, data e hora, se é retweet, contagem de retweets, o "nome de tela" do usuário, o id do usuário, número de favoritações ("likes"), etc. Assim, temos que o resultado retornado porssui NOVENTA variáveis (é bastante) com diversos tipos de informação e metadados. Vamos observar os textos dos primeiros tweets recebidos com a função *head*:

```{r}
head(tweets_quilombo$text)
```

Ao olhar a coluna "text" claramente obtivemos resultados em espanhol que possivelmente não são do Brasil. Podemos estabelecer essas duas condições e também retirar os retweets, mantendo somente quem publicou o texto. Primeiro selecionamos a língua no parâmetro *lang*. Vamos pular um argumento e ir para o *include_rts* que, com a opção *FALSE* exclui os retweets.

Por fim, definimos de onde queremos os tweets com a função *lookup_coords* no argumento *geocode*, além de indicar uma *apikey*. Você já deve suspeitar, pelo que vimos hoje, que é a chave para acessar uma API - aqui a do Google Maps. Ela precisa de muitos passos e, por hora, vamos utilizar um app que eu criei, mas caso tenham interesse é necessário acessar este [link](https://cloud.google.com/maps-platform?hl=pt-br), criar seu app com as funções de georreferenciamento e incluir um método de pagamento para ativar seu app (eles não cobram automaticamente).

```{r}
tweets_quilombo <- search_tweets("quilombo",
                                 n = 100, 
                                 lang = "pt",
                                 geocode = lookup_coords("brasil", apikey = "AIzaSyAMghXSOGW8RxCPY4FOOAYilA7OMKOpaNw"),
                                 include_rts = F)
```

Podemos, ainda, estabelecer uma pesquisa por múltiplos termos acrescentando um *OR* entre os termos no nosso vetor character no começo da função. Aqui ampliaremos nossa busca para 15 mil tweets:

```{r}
tweets_multiplos <- search_tweets("quilombo OR quilobolas OR racismo",
                                  n = 15000,
                                  lang = "pt",
                                  geocode = lookup_coords("brasil", apikey = "AIzaSyAMghXSOGW8RxCPY4FOOAYilA7OMKOpaNw"),
                                  include_rts = F)
```

Muito bacana, não? Conseguimos buscar um grande número de tweets em apenas poucos segundos.

## Criando um mapa de pontos dos tweets

Uma opção bem bacana que temos é o georreferenciamento dos tweets. Como veremos, nem sempre temos essa informação porque o usuário precisa compartilhar sua localização com o Twitter. Mesmo assim vale a pena explorar essa ferramenta. 

Nosso primeiro passo é obter as coordenadas de cada tweet com a função *lat_lng*. Aplicaremos ao nosso objeto com diversos termos que nos retornará um objeto com mais duas variáveis no final, vamos observá-la com a função *head*.

```{r}
geo_tweet <- lat_lng(tweets_multiplos)

head(geo_tweet[,91:92])
```

Agora possuímos as informações de latitude e longitude quando disponíveis (possivelmente com missing values nas primeiras linhas, rs). Nosso próximo passo é importar um shapefile do Brasil para produção de nosso mapa. Utilizaremos a função *read_state* para pegar o shapefile produzido pelo IGBE em 2019. Depois vamos observar a classe do objeto que criamos:

```{r}
shp_br <- read_state(year = 2019)

class(shp_br)
```

Você percebeu que foi realizado um "download" do shapefile que é um objeto da classe *sf* (simple features) que permite a nós trabalharmos com shapefiles como se fossem *data_frames*. Para plotar esse mapa, utilizaremos o *ggplot2* e "somamos" a função *geom_sf*. Utilizarmos o *pipe* para indicar onde buscamos nossos dados. Note que a partir do momento em que chamamos o ggplot, não usamos mais o *pipe* e sim o símbolo de soma (+):

```{r}
shp_br %>% 
  ggplot() +
  geom_sf()
```

Vamos criar um objeto para armazenar os mapas dos tweets e o deixaremos mais "bonitão". Vamos fazer o processo já baixando o shapefile e acrescentando novos elementos após o *pipe*. Primeiro vamos acrescentar um tema mais limpo, preto e branco com a função *theme_bw* (black and white). Em um segundo passo novo, plotaremos os pontos do georreferenciamento utilizando como *data* o objeto "geo_tweet" e, como elementos da *aesthetics* a longitude e a latitude, definindo o tamanho de cada ponto (*size*), seu formato (*shape*) e sua cor (*fill*). Por fim, acrescentaremos os rótulos com a função *labs*, nas quais os argumentos são auto-explicativos. Não se preocupe com eles por hora, para uma melhor compreensão dê uma olhada no [Capítulo 3 do *R for Data Science*](https://r4ds.had.co.nz/data-visualisation.html).

```{r}
mapa_tweets <- read_state(year = 2019) %>% 
  ggplot() +
  geom_sf() +
  theme_bw() +
  geom_point(data = geo_tweet, aes(x = lng, y = lat), size = 2, 
shape = 23, fill = "darkred") +
  labs(x = NULL,
       y = NULL,
       title = "Distribuição geográfica de uma amostra de tweets",
       subtitle = "Termos 'quilombo', 'quilombola' e 'racismo'",
       caption = "Fonte: Dados coletados da API do Twitter via rtweet")

print(mapa_tweets)
```

Alguns pontos vieram com erro, não? Eles ficaram fora dos limites do mapa brasileiro e, talvez, na próxima *request* à API devemos utilizar "brazil" com z. rs

## Capturando dados de um usuário

Além de obtermos os tweets de uma busca por hashtag ou termo-chave, podemos obter informações dos usuários, como sua timeline, "amigos" (quem eles seguem) e seguidores. Vamos utilizar a *\@conectas* como exemplo e primeiro capturar as últimas mil publicações que estão em sua timeline com a função *get_timeline*:

```{r}
conectas <- get_timeline("conectas", n = 1000)
```

O objeto retornado possui as mesmas 90 variáveis que observamos anteriormente, mas agora são os tweets apenas do usuário *\@conectas*. A partir desse objeto, o pacote *rtweet* permite fazer um gráfico de série temporal para obervar a atividade da conta. Vamos integrar aqui com o *ggplot2* e criar um gráfico para a cada 7 dias da conta:

```{r}
ts_plot(conectas, by = "7 days") +
  theme_bw() +
  labs(
    y = NULL, x = NULL,
    title = "Frequências dos tweets da @conectas entre 08/06/2020 e 08/02/2021",
    subtitle = "Tweets agregados a cada 7 dias",
    caption = "Fonte: Dados coletados da API do Twitter via rtweet"
  )
```

Muito bacana, não? Funções simples para produzir resultados tangíveis são ótimas ferramentas.

Em nossos próximos passos, buscaremos os amigos e os seguidores da *\@conectas* utilizando as funções *get_friends* e *get_followers*. Para a primeira função, o limite de resultados é de 5 mil e para a segunda de 75 mil. Para esperar o tempo para uma nova requisição, utilizamos o parâmetro *TRUE* em um argumento chamado *retryonratelimit* que "espera" para continuar a requisição quando atingimos o limite antes de seguir para a próxima linha de código. Em ambos os casos, o número de resultados pré-estabelecido é de 5 mil, mas para pegarmos todos os resultados podemos utilizar o argumento *Inf* que indica Infinito para valores positivos.

```{r}
amigos_conectas <- get_friends("conectas",
                               n = Inf,
                               retryonratelimit = TRUE) #usado no caso de ultrapassar os 5k


seguidores_conectas <- get_followers("conectas",
                                     n = Inf,
                                     retryonratelimit = TRUE)
```

Assim, a partir de duas funções, pegamos todos os amigos e seguidores da *\@conectas*. Com mais uma função podemos pegar as últimas favoritações, os likes, da *\@conectas*, a *get_favorites* utilizando o "n = 1000" : 

```{r}
conectas_fav <- get_favorites("conectas",
                              n = 1000)
```

Você deve ter percebido que o número é maior do que o requisitado, isso está associado às threads publicadas.

## Observando as conexões com mapas de redes

Uma coisa muito bacana que podemos fazer é observar as relações entre diversos usuários com mapas de redes. Vamos fazer uma pequena amostra nesse tutorial utilizando 10 usuários: *\@coalizaonegra*, *\@conaquilombos*, *\@conectas*, *\@damaresalves*, *\@davidmirandario*, *\@bolsonarosp*, *\@fopir_*, *\@palmaresgovbr*, *\@odara_instituto* e *\@jairbolsonaro*.

O primeiro passo é buscar quem são os amigos desses usuários com a já vista função *get_friends*:

```{r}
amigos <- get_friends(c("coalizaonegra",
                     "conaquilombos",
                     "conectas",
                     "damaresalves",
                     "davidmirandario",
                     "bolsonarosp",
                     "fopir_",
                     "palmaresgovbr",
                     "odara_instituto",
                     "jairbolsonaro"),
                   retryonratelimit = TRUE)
```

Antes de prosseguir, vamos buscar os \@s dos usuários que comporão nossa rede, uma vez que só obtemos o seu id. Faremos isso em dois passos. No primeiro, vamos selecionar apenas observações únicas do "user_id". Em um segundo momento utilizaremos a função *lookup_users* para buscar as informações desses usuários. Como o resultado contém todas as 90 variáveis contendo o último tweet de cada um dos usuários, vamos selecionar apenas duas variáveis de interesse para inserir os dados com os nomes de usuários no nosso *data_frame* "amigos": "user_id" e "screen_name":

```{r}
usernames <- unique(amigos$user_id)
users     <- lookup_users(usernames) %>% 
  select(user_id, screen_name)
```

Agora faremos um longo *pipe* que seguirá os seguintes passos: (1) unir os DFs amigos e users com a função *left_join*; (2) agrupar os dados pelo nome dos usuários (screen_name) com a função *group_by*; (3) contar o número de observações de cada usuário com a função *mutate*; (4) desagrupar para continuar a nossa manipulação de dados com a função *ungroup*; (5) selecionar todas as variáveis menos a "user_id" com a função *select*; (6) selecionar apenas os usuários que estejam nos amigos de pelo menos 4 dos nossos 10 usuários com a função *filter*; (7) preparar um gráfico a partir do data frame com a função *graph_from_data_frame*; e (8) criar o objeto ggraph utilizando o layout "fr" (para ver os layouts possíveis acesse [esse link](https://d33wubrfki0l68.cloudfront.net/756daa22f3690a4073ce41607ef98bf5536035a4/1f06e/post/2017-02-06-ggraph-introduction-layouts_files/figure-html/unnamed-chunk-12.gif)). Utilizamos a função *set.seed* antes de tudo isso para que o gráfico gerado seja o mesmo para todos - ele define um padrão para ações aleatórias, no caso "12345".

```{r}
set.seed(12345)
graph_data <- amigos %>%
  left_join(users) %>% 
  group_by(screen_name) %>% 
  mutate(n = n()) %>% 
  select(-user_id) %>% 
  ungroup() %>% 
  filter(n > 4) %>%
  graph_from_data_frame() %>%
  ggraph(layout = "fr")
```

Antes de gerar nosso gráfico, definiremos as cores em que aparecerão os usuários pesquisados e os conectados criando o objeto "mcolor" que diferencia a cor entre quem está nos "users" (nossos 10 usuários) e quem não está (os amigos). 

```{r}
mcolor <- graph_data$data %>%
  mutate(mcolor = if_else(name %in% amigos$user, 
                          "blue4", "blueviolet")) %>%
  select(mcolor)
```

O próximo passo é a produção do nosso gráfico de redes. Começamos somando o que faremos no gráfico ao objeto "graph_data", que tem todas as informações que buscaremos para plotar. No primeiro passo, criaremos linhas entre os nós no início (nossos usuários) até o fim (amigos) com a função *geom_edge_link* - não se preocupem com as aesthetics por agora. Um segundo passo é criar pontos nos nós com a função *geom_node_point* e, por fim, acrescentar os rótulos nos nós com a função *geom_node_text*. Escolhemos o tema e removemos a legenda e...

```{r}
graph_data +
  geom_edge_link(aes(edge_alpha = n, edge_width = n, colour = factor(amigos$user)), edge_colour = "cyan2") +
  geom_node_point(size = 1) +
  geom_node_text(aes(label = name), repel = TRUE,
                 point.padding = unit(0.2, "lines"), colour=mcolor$mcolor) +
  theme_void() + 
  theme(legend.position="none")
```

Produzimos nosso primeiro gráfico de redes. Começamos a ver alguns produtos dos dados que coletamos a partir de hoje, nas próximas aulas isso vai se tornar mais comum. 
