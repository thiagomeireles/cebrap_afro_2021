Tutorial 7: Raspando notícias da Folha

Nos atividades anteriores utilizamos as ferramentas básicas de captura de dados disponíveis na biblioteca *rvest*. Primeiro, aprendemos a capturar várias páginas contendo tabelas em formato HTML de uma só vez. Depois, aprendemos como um documento XML está estruturado e que podemos a extrair com precisão os conteúdos de tags e os valores dos atributos das tags de páginas escritas em HTML.

Neste tutorial vamos colocar tudo em prática e construir um banco de dados de notícias. O nosso exemplo será o conjunto de notícias (275 na data da construção deste tutorial) publicadas nas Edições Impressas da Folha de São Paulo. 

Clique no [link](https://search.folha.uol.com.br/search?q=quilombo&site=jornal&ed=15%2F10%2F2015&periodo=todos&results_count=275&search_time=1%2C150&url=https%3A%2F%2Fsearch.folha.uol.com.br%2Fsearch%3Fq%3Dquilombo%26site%3Djornal%26periodo%3Dtodos&sr=) caso não lembre como está a estruturada a busca no sistema da Folha.

# Raspando uma notícia no site da Folha de São Paulo

Antes de começar, vamos chamar a biblioteca _rvest_ para tornar suas funções disponíveis em nossa sessão do R. Na carona, vamos chamar também a biblioteca _dplyr_.

```{r}
library(rvest)
library(dplyr)
library(stringr)
```

Nossa primeira tarefa será escolher uma única notícia (a primeira da busca, por exemplo), e extrair dela 4 informações de interesse: o título da notícia; o subtítulo da notícia; a data e hora da notícia; e o texto da notícia.

O primeiro passo é criar um objeto com endereço URL da notícia e outro que contenha o código HTML da página:

```{r}
url_noticia <- "https://www1.folha.uol.com.br/equilibrioesaude/2021/01/imagens-de-sapatos-destacam-ausencia-deixada-pelas-200-mil-vitimas-de-covid-19-no-brasil.shtml"
pagina <- read_html(url_noticia)
```

Pronto! Agora temos um objeto adequadamente identificado como XML pelo R. Com o objeto XML preparado e representando a página com a qual estamos trabalhando, vamos à caça das informações que queremos.

Volte para a página da notícia. Procure o título da notícia e examine-o, inspencionando o código clicando com o botão direito do mouse e selecionando "Inspecionar". Note o que encontramos:

```{xml}
<h1 class="c-content-head__title" itemprop="headline">== $0"Imagens de sapatos destacam ausência deixada pelas 200 mil vítimas de Covid-19 no Brasil"</h1>
```

Tente sozinh@ e por aproximadamente 1~2 minutos construir o "xpath" (caminho em XML) que nos levaria a este elemento antes de avançar. (Tente sozinh@ antes de copiar a resposta abaixo!)
  
A resposta é:

```{xml}
//h1[@class = 'c-content-head__title']
```

Usando agora as funções _html\_nodes_ e _html\_text_, como vimos no tutorial anterior, vamos capturar o título da notícia:

```{r}
node_titulo <- html_nodes(pagina, xpath = "//h1[@class = 'c-content-head__title']")
titulo <- html_text(node_titulo) %>% 
    str_squish()
print(titulo)
```

Simples, não? Repita agora o mesmo procedimento para o subtítulo (tente sozinh@ antes de copiar a resposta abaixo!):

```{r}
node_subtitulo <- html_nodes(pagina, xpath = "//h2[@class = 'c-content-head__subtitle']")
subtitulo <- html_text(node_subtitulo) %>% 
    str_squish()
print(subtitulo)
```


```{r}
node_datahora <- html_nodes(pagina, xpath = "//div[5]//time")
datahora <- html_text(node_datahora) %>% 
  str_squish()
print(datahora)
```


Finalmente, peguemos o texto. Note que o texto está dividido em vários parágrafos cujo conteúdo está inseridos em tags "p", todas filhas da tag "article". Se escolhemos o xpath sem especificar a tag "p" ao final, como abaixo, capturamos um monte de "sujeira", como os botões de twitter e facebook. Por esta razão, vamos especificar a tag "p" ao final do xpath. Neste caso, recebemos um vetor contendo cada um dos parágrafos do texto. Precisaríamos "juntar" (concatenar) todos os parágrafos para formar um texto único. Faremos isso a seguir.

```{r}
node_texto <- html_nodes(pagina, xpath = "//article[@class = 'c-news']//div[5]//p")
texto <- html_text(node_texto) %>% 
  str_squish()
print(texto)
```

A função _paste_ (que é igual _paste0_, mas tem por padrão deixar espaço entre os textos "colados") permite juntarmos todos os textos de um vetor do tipo "character" em um só utilizando o argumento "collapse". Vamos, assim, juntar todos os parágrafos com _paste_ separando-os por um espaço simples depois de filtrar as entradas "vazias" do vetor:

```{r}
texto <- texto[texto!=""]
texto <- paste(texto, collapse = " ")
print(texto)
```

### Tarefa 1: Tente raspar a notícia seguinte na busca da Folha de São Paulo

Tente agora raspar a notícia seguinte usando a mesma estratégia. É fundamental notar que variamos a notícia, mas as informações continuam tendo o mesmo caminho. Essa é a propriedade fundamental do portal raspado que nos permite obter todas as notícias sem nos preocuparmos em abrir uma por uma. O link para a próxima notícia está no objeto "url" abaixo:

```{r}
url_noticia <- "https://www1.folha.uol.com.br/cotidiano/2021/01/nova-camara-de-sp-comeca-com-maior-oposicao-a-covas-e-tensao-entre-pt-e-psol.shtml"
```

Obs: agora que criarão seus próprios *scripts* com os códigos para raspagem dessa página, não se esqueçam de documentar o que estão fazendo. Isso é muito importante para qualquer pessoa que vá replicar o que fizeram, mas, também, para que consiga entender o que fez mesmo depois de muito tempo. Como muitos pacotes sofrem alterações, inclusive com substituição de funções, isso auxilia a resolver problemas que possam aparecer. Parte importante na programação é deixar claro para você e para os outros o que e o porquê está fazendo cada etapa.

# Um código, duas etapas: raspando todas as notícias sobre quilombo na Folha de São Paulo

Vamos fazer um breve roteiro do que precisamos fazer para criar um banco de dados que contenha todos os títulos, subtítulos, data e hora e texto de todas as notícias sobre quilombos da Folha de São Paulo.

### Etapa 1
* Passo 1: conhecer a página de busca (e compreender como podemos "passar" de uma página para outra)
* Passo 2: raspar (em loop!) as páginas de busca para obter todos os links de notícia

Esta é a primeira etapa da captura. Em primeiro lugar temos que buscar todos os URLs que contêm as notícias buscadas. Em outras palavras, começamos obtendo "em loop" os links das notícias e, só depois de termos os links, obtemos o conteúdo destes links. Nossos passos seguintes, portanto, são:

### Etapa 2
* Passo 3: conhecer a página da notícia (e ser capaz de obter nela as informações desejadas). Já fizemos isso acima!
* Passo 4: raspar (em um novo loop!) o conteúdo dos links capturados no Passo 2.

Vamos construir o código da primeira etapa da captura e, uma vez resolvida a primeira etapa, faremos o código da segunda.

### Código da etapa 1

AVISO: Esse código é o que fizemos no tutorial anterior. Comecemos criando o URL base sem a numeração de notícias:

```{r}
url_base <-  "https://search.folha.uol.com.br/search?q=quilombo&site=jornal&ed=15%2F10%2F2015&periodo=todos&results_count=275&search_time=1%2C150&url=https%3A%2F%2Fsearch.folha.uol.com.br%2Fsearch%3Fq%3Dquilombo%26site%3Djornal%26periodo%3Dtodos&sr="
```

Em segundo lugar, vamos observar o URL da página de busca (poderíamos buscar termos chave, mas, neste caso, vamos pegar todas as notícias relacionadas a quilombos). Na página 2 da busca vemos que o final é "sr=26". Na página 3 o final é "sr=51". Há um padrão: as buscas são realizadas de 25 em 25. De fato, a 27a. é última página da busca. Para "passarmos" de página em página, portanto, temos que ter um "loop" que conte não mais de 1 até 25, mas na seguinte sequência numérica: {1, 26, 51, 76, ..., 226, 251}.

Precisamos, então, que "i" seja recalculado dentro do loop para coincidir com a numeração da primeira notícia de cada página. Parece difícil, mas é extremamente simples. Veja o loop abaixo, que imprime a sequência desejada multiplicando (i - 1) por 25 e somando 1 ao final:

```{r}
for (i in 1:11){
  i <- (i - 1) * 25 + 1
  print(i)
}
```

Precisamos agora é incluir nas "instruções do loop". Assim, construímos o url de cada página do resultado da busca:

```{r}
url_pesquisa <- paste(url_base, 1, sep = "")
```

A seguir, capturamos o código HTML da página:

```{r}
pagina <- read_html(url_pesquisa)
```

Escolhemos apenas os "nodes" que nos interessam:

```{r}
nodes_titulos <- html_nodes(pagina, xpath = "//ol/li/div/div/a/h2[@class = 'c-headline__title']")
nodes_links <- html_nodes(pagina, xpath = "//*[@id='view-view']/div/div/a")
```

Extraímos os títulos e os links com as funções apropriadas:

```{r}
titulos <- html_text(nodes_titulos) %>% 
  str_squish()

links   <- html_attr(nodes_links, name = "href")
links   <- unique(links)
```

Combinamos os dois vetores em um data frame:

```{r}
tabela_titulos <- data.frame(titulos, links)
```

Falta "empilhar" o que produziremos em cada iteração do loop de uma forma que facilite a visualização. Criamos um objeto vazio antes do loop. 

Usaremos a função _bind\_rows_ (ou _rbind_ se estiver com problemas com o _dplyr_) para combinar data frames. A cada página agora, teremos 25 resultados em uma tabela com duas variáveis. O que queremos é a junção dos 25 resultados de cada uma das 11 páginas. Vamos também chamar a biblioteca _dplyr_ para usar sua função _bind\_rows_. 

```{r}
library(dplyr)
dados_pesquisa <- data.frame()
dados_pesquisa <- bind_rows(dados_pesquisa, tabela_titulos)
```

Chegou o momento de colocar dentro loop tudo o que queremos que execute em cada uma das vezes que ele ocorrer. Ou seja, que imprima na tela a página que está executando, que a URL da página de resultados seja construída com a função paste, para todas elas o código HTML seja examinado, lido no R e transformado em objeto XML, colete todos os links e todos os títulos e que "empilhe". Lembrando que não podemos esquecer de definir a URL que estamos usando e criar um data frame vazio para colocar todos os links e títulos coletados antes de iniciar o loop.

```{r}
url_base <-  "https://search.folha.uol.com.br/search?q=quilombo&site=jornal&ed=15%2F10%2F2015&periodo=todos&results_count=275&search_time=1%2C150&url=https%3A%2F%2Fsearch.folha.uol.com.br%2Fsearch%3Fq%3Dquilombo%26site%3Djornal%26periodo%3Dtodos&sr="

dados_pesquisa <- data_frame()

for (i in 1:11){
  
  print(i)

  x <- (i - 1) * 25 + 1
  
  url_pesquisa <- paste(url_base, x, sep = "")
  
  pagina <- read_html(url_pesquisa)
  
  nodes_titulos <- html_nodes(pagina, xpath = "//ol/li/div/div/a/h2[@class = 'c-headline__title']")
  nodes_links <- html_nodes(pagina, xpath = "//*[@id='view-view']/div/div/a")

  titulos <- html_text(nodes_titulos) %>% 
    str_squish()

  titulos <- as.data.frame(titulos) %>% 
  mutate(check = !duplicated(titulos)) %>% 
  filter(titulos=="Mônica Bergamo" | check != F,
         titulos!="Festival Valongo revê rumos e prioriza ativismo")
  
  titulos <- as.vector(titulos$titulos)
  
  links   <- html_attr(nodes_links, name = "href")
  links <- unique(links)
  links <- links[links!="http://www1.folha.uol.com.br/fsp/campinas/cm04109812.htm"]

  tabela_titulos <- data.frame(titulos, links)
  
  dados_pesquisa <- bind_rows(dados_pesquisa, tabela_titulos)

}
```

Agora todos os títulos e links de todos os resultados de site do Folha de São Paulo estão em um único banco de dados. Bom trabalho!

### Código da etapa 2

No começo do tutorial resolvemos a captura do título, sutbtítulo, data e hora, e texto para uma única notícia no portal da Folha de São Paulo. Nos resta agora capturar, em loop, o conteúdo de cada uma das páginas cujos links estão guardados no vetor "links_Folha de São Paulo".

Vamos rever o procedimento, para uma URL qualquer, da captura do título, data e hora e texto (vamos deixar o link para o relatório de pesquisa de lado por enquanto, posto que algumas notícias não contêm o link e esta pequena ausência interromperia o funcionamento do código).

```{r}
url_noticia <- "https://www1.folha.uol.com.br/equilibrioesaude/2021/01/imagens-de-sapatos-destacam-ausencia-deixada-pelas-200-mil-vitimas-de-covid-19-no-brasil.shtml"

pagina <- read_html(url_noticia)

node_titulo <- html_nodes(pagina, xpath = "//h1[@class = 'c-content-head__title']")
titulo <- html_text(node_titulo) %>% 
    str_squish()

node_subtitulo <- html_nodes(pagina, xpath = "//h2[@class = 'c-content-head__subtitle']")
subtitulo <- html_text(node_subtitulo) %>% 
    str_squish()

node_datahora <- html_nodes(pagina, xpath = "//div[5]//time")
datahora <- html_text(node_datahora) %>% 
  str_squish()

node_texto <- html_nodes(pagina, xpath = "//article[@class = 'c-news']//div[5]//p")
texto <- html_text(node_texto) %>% 
  str_squish()
texto <- texto[texto!=""]
texto <- paste(texto, collapse = " ")
```

Para fazermos a captura de todos os links em "loop" deve ter o seguinte aspecto, como se vê no código abaixo que imprime todos os 670 links cujo conteúdo queremos capturar. Note que a forma de utilizar o loop é ligeiramente diferente da que havíamos visto até então. No lugar de uma variável "i" que "percorre" um vetor numérico (1:27, por exemplo), temos uma variável "link" que recebe, a cada iteração, um endereço URL da notícia, em ordem.

Esses endereços estão armazenados na variável "links" do banco de dados que criamos na Etapa 1, "dados\_pesquisa". Assim, na primeira iteração temos que "link" será igual "dados_pesquisa\$links[1]", na segunda "dados_pesquisa\$link[2]" e assim por diante até a última posição do vetor "links_Folha de São Paulo" -- no nosso caso a posição 669.

```{r, echo=FALSE}
for (link in dados_pesquisa$links){
  print(link)
}
```

Combinando os dois código, e criando um data frame "dados\_noticias" que é vazio antes do loop temos o código completo da captura. Tal como quando trabalhamos com tabelas, utilizando a função "bind_rows" para combinar o data frame que resultou da iteração anterior com a linha que combina o conteúdo armazenado em "titulo", "datahora" e "texto". Faremos isso para as 100 primeiras notícias por agora, voltaremos mais tarde para buscar de todos.

Obs: O artigo de opinião da oitava entrada tem divergência na estrutura do xpath. Vamos ignorá-la por agora.

```{r}
dados_noticias <- data.frame()

for (link in dados_pesquisa$links[c(1:7,9:100)]){
  
  print(link)
  
  pagina <- read_html(link)
  node_titulo <- html_nodes(pagina, xpath = "//h1[@class = 'c-content-head__title']")
  titulo <- html_text(node_titulo) %>%
    str_squish()
  
  node_subtitulo <- html_nodes(pagina, xpath = "//h2[@class = 'c-content-head__subtitle']")
  subtitulo <- html_text(node_subtitulo) %>% 
    str_squish()
  
  node_datahora <- html_nodes(pagina, xpath = "//div[5]//time")
  datahora <- html_text(node_datahora) %>% 
    str_squish()
  
  node_texto <- html_nodes(pagina, xpath = "//article[@class = 'c-news']//div[5]//p")
  texto <- html_text(node_texto) %>% 
    str_squish()
  texto <- texto[texto!=""]
  texto <- paste(texto, collapse = " ")
  
  tabela_noticia <- data_frame(titulo, subtitulo, datahora, texto)
  
  dados_noticias <- bind_rows(dados_noticias, tabela_noticia)

}
```

O resultado do código é um data frame ("dados\_noticias") que contém 4 variáveis em suas colunas: "titulo", "datahora", "link" e "texto". A partir de agora você poderia, por exemplo, usar as ferramentas de "text mining" para criar uma nuvem de palavras ("wordcloud"), fazer a contagem de termos, examinar a semelhança da linguagem usada pelo instituto Folha de São Paulo com a usada por outros institutos de opinião pública, fazer análise de sentimentos, etc.

### Tarefa 2: Comentando o código

Antes disso, sua tarefa é a seguinte: executar ambas as etapas do código e comentá-lo por completo (use # para inserir linhas de comentário). Comentar o código alheio é uma excelente maneira de ver se você conseguiu compreendê-lo por completo e serve para você voltar ao código no futuro quando for usá-lo de modelo para seus próprios projetos em R.

Dica: Compare o número de obsevações dos dataframes _dados_noticias_ e _dados_pesquisa_. O que ocorreu?

```{r}
url_base <-  "https://search.folha.uol.com.br/search?q=quilombo&site=jornal&ed=15%2F10%2F2015&periodo=todos&results_count=275&search_time=1%2C150&url=https%3A%2F%2Fsearch.folha.uol.com.br%2Fsearch%3Fq%3Dquilombo%26site%3Djornal%26periodo%3Dtodos&sr="

dados_pesquisa <- data_frame()

for (i in 1:11){
  
  print(i)
  
  i <- (i - 1) * 25 + 1
  
  url_pesquisa <- paste(url_base, i, sep = "")
  
  pagina <- read_html(url_pesquisa)
  
  nodes_titulos <- html_nodes(pagina, xpath = "//ol/li/div/div/a/h2[@class = 'c-headline__title']")
  nodes_links <- html_nodes(pagina, xpath = "//*[@id='view-view']/div/div/a")
  
  titulos <- html_text(nodes_titulos) %>% 
    str_squish()

  titulos <- as.data.frame(titulos) %>% 
  mutate(check = !duplicated(titulos)) %>% 
  filter(titulos=="Mônica Bergamo" | check != F,
         titulos!="Festival Valongo revê rumos e prioriza ativismo")
  
  titulos <- as.vector(titulos$titulos)
  
  links   <- html_attr(nodes_links, name = "href")
  links <- unique(links)
  links <- links[links!="http://www1.folha.uol.com.br/fsp/campinas/cm04109812.htm"]

  tabela_titulos <- data.frame(titulos, links)
  
  dados_pesquisa <- bind_rows(dados_pesquisa, tabela_titulos)
}

dados_noticias <- data.frame()

for (link in dados_pesquisa$links[-8]){
  
print(link)
  
  pagina <- read_html(link)
  node_titulo <- html_nodes(pagina, xpath = "//h1[@class = 'c-content-head__title']")
  titulo <- html_text(node_titulo) %>%
    str_squish()
  
  node_subtitulo <- html_nodes(pagina, xpath = "//h2[@class = 'c-content-head__subtitle']")
  subtitulo <- html_text(node_subtitulo) %>% 
    str_squish()
  
  node_datahora <- html_nodes(pagina, xpath = "//div[5]//time")
  datahora <- html_text(node_datahora) %>% 
    str_squish()
  
  node_texto <- html_nodes(pagina, xpath = "//article[@class = 'c-news']//div[5]//p")
  texto <- html_text(node_texto) %>% 
    str_squish()
  texto <- texto[texto!=""]
  texto <- paste(texto, collapse = " ")
  
  tabela_noticia <- data_frame(titulo, subtitulo, datahora, texto)
  
  dados_noticias <- bind_rows(dados_noticias, tabela_noticia)

  
}
```

Vocês devem perceber que o número de observações do nosso *data_frame* "dados_noticias" é muito menor do que no "dados_pesquisa". Ok, vimos que uma das páginas foi excluída do loop por conflito no "nó", identificamos e poderíamos acrescentar com a correção do nó do título. Mas e as outras quase 200?

Se observamos o período das notícias capturadas, ele é recente (a partir de abril de 2018), então para notícias anteriores a esse período precisamos identificar os nós na página e realizar o mesmo processo. Como já foi frisado, é um processo automatizado com alguns aspectos "artesanais". Mesmo assim o ganho de tempo na execução da tarefa é gigantesco, não?