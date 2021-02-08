# Tutorial 9: Capturando as publicações do DOU

## Abrindo o RSelenium e acessando a busca do DOU

Como vimos no Tutorial 8, para acessar sites dinâmicos utilizaremos o *RSelenium*. Antes de abrir nosso navegador remoto, vamos carregar os pacotes que utilizaremos no Tutorial:

```{r}
library(RSelenium)
library(rvest)
library(stringr)
library(dplyr)
```

Após carregar nossos pacotes, abrimos uma chamada no *RSelenium* para criar um Driver remoto do Chrome com a função *rsDriver* e criamos um cliente para conseguirmos navegar pelas páginas que precisamos.

```{r}
rD <- rsDriver(browser = c("chrome"), chromever = "88.0.4324.96",
               port = 4567L)

cliente <- rD$client
```

Como já vimos no Tutorial 8, vamos criar um *url_base* para realizarmos a substituição pelas páginas que temos interesse e realizar a raspagem posteriormente com o *rvest*. É possível utilizar o RSelenium para preencher automaticamente todos os campos da caixa de busca e pedir para mudar de página, mas por enquanto adotaremos a abordagem que já trabalhamos. Faremos isso em outro momento.

```{r}
url_base <-  "https://www.in.gov.br/consulta/-/buscar/dou?q=quilombo&s=todos&exactDate=personalizado&sortType=0&delta=20&currentPage=ATUAL&newPage=PROXIMA&score=0&id=300156803&displayDate=1611198000000&publishFrom=01%2F01%2F2020&publishTo=04%2F02%2F2021"
```

Com o *url_base* estabelecido, nosso próximo passo é utilizar a função *gsub* para acessarmos as páginas de interesse. Vamos, por enquanto, ficar na primeira página do sistema de buscas. Lembrando que a segunda página é a "currentPage" 1, iniciaremos com "currentPage" igual a 0 e "newPage" igual a 1. Vamos substituir!

```{r}
i <- 0
url_pesquisa <- gsub("ATUAL", i, url_base)
url_pesquisa <- gsub("PROXIMA", i+1, url_pesquisa)
print(url_pesquisa)
```

Após estabelecer qual o url de interesse, o próximo passo é acessar nossa *url_pesquisa* utilizando a função  *$navigate* no objeto *cliente*. Com isso, a conexão com o Driver remoto do Chrome que estabelecemos carregará nossa página para controle remoto utilizando o R.

```{r}
cliente$navigate(url_pesquisa)
```

Pronto! Agora nos próximos passos utilizaremos os "xpaths" para capturarmos os títulos e links das publicações do DOU. 

Dica: para extrair os xpaths, utilize seu Google Chrome e não nossa versão remota. Aqui ele não indica "automaticamente" onde estão os nós que queremos quando inspecionamos os objetos.

## Capturando os títulos e links das publicações do DOU

Nessa etapa utilizaremos os argumentos já conhecidos e aplicados nas páginas estáticas, como o da Folha de São Paulo. Por a abrirmos com o *RSelenium* conseguimos capturar seu conteúdo, diferentemente do que aconteceu no Tutorial 8. Vamos aplicar o mesmo código que utilizamos anteriormente. No entanto, nossa página dinâmica está em uma lista acessada a partir do *cliente* com a função *getPageSource()*. Por ser o primeiro elemento de uma lista, identificamos com o argumento "[[1]]" (se lembram que utilizamos dois colchetes para objetos dentro de uma lista?). Em seguida combinaremos os dois *vetores* em um *data_frame* com os dados que utilizaremos em nossa pesquisa:

```{r}
pagina <- read_html(cliente$getPageSource()[[1]])

titulo <-  pagina %>% 
  html_nodes(xpath = "//*[@class='title-marker']//a") %>% 
  html_text()
  
link <-  pagina %>% 
  html_nodes(xpath = "//*[@class='title-marker']//a") %>% 
  html_attr(name = "href")

dados_tabela <- data.frame(titulo, link)
```


Diferentemente do que observamos em nossa tentativa sem o *RSelenium*, agora conseguimos capturar os elementos de nossa página que está aberta remotamente e criamos um novo data_frame com os dados que precisamos para acessar as publicações do DOU que estão na primeira página de resultados da nossa busca por "quilombo".

Agora faremos um loop para capturar as páginas de todas as páginas do nosso resultado da busca. Criaremos um *data_frame* vazio para empilharmos as tabelas com títulos e links. Depois, aplicaremos um for para um vetor que vai de 0 (página 1) a 23 (página 24) no código que utiliamos acima - exatamente o mesmo. No fim, empilhamos as tabelas criadas dentro do nosso novo *data_frame*.

```{r}
dados_tabela <- data.frame()

for (i in 0:23) {
  
  print(i)
  
  url_pesquisa <- gsub("ATUAL", i, url_base)
  
  url_pesquisa <- gsub("PROXIMA", i+1, url_pesquisa)
  
  cliente$navigate(url_pesquisa)
  
  pagina <- read_html(cliente$getPageSource()[[1]])
  
  secao <- pagina %>% 
    html_nodes(xpath = "//*[@class='resultado']//p[1]") %>% 
    html_text()
  
  titulo <-  pagina %>% 
    html_nodes(xpath = "//*[@class='title-marker']//a") %>% 
    html_text()
  
  link <-  pagina %>% 
    html_nodes(xpath = "//*[@class='title-marker']//a") %>% 
    html_attr(name = "href")
  
  tabela <- as.data.frame(cbind(titulo, link))
  
  dados_tabela <- rbind(dados_tabela, tabela)
  
}
```

Isso parece familiar, não? É o mesmo procedimento que utlizamos para capturar os títulos e links da nossa pesquisa no site da Folha de São Paulo. Somente alteramos os "xpaths" para nossos nodes. Só que, para isso, precisamos do *RSelenium* e do nosso driver remoto do Chrome.

Nosso próximo passo é observar a estrutura dos textos para extrair as informações que precisamos. Como elas são estáticas, não precisaremos acessar pelo navegador remoto criado com o *RSelenium* e o *chromedriver*, então  encerraremos a sessão porque não vamos mais utilizá-la por aqui.


```{r}
cliente$close()
rD$server$stop()
```



## Capturando o texto de uma publicação do DOU

Estamos novamente com uma página estática, voltamos à nossa zona de conforto - sim, sabemos que nem tanto rs. Vamos trabalhar com a primeira página dos resultados. Aqui temos uma "pegadinha". Observem o que temos na primeira linha da nossa coluna de links e tente identificar e depois siga para a próxima etapa.

```{r}
url_pesquisa <- dados_tabela$link[1]
print(url_pesquisa)
```

Alguma coisa estranha? Sim, nós capturamos somente o "final" dos endereços que utilizaremos. Assim, recorreremos à função *paste* para colar esse final ao endereço base utilizado pela [Imprensa Nacional](https://www.in.gov.br). Vejamos como fica:

```{r}
url_pesquisa <- paste("https://www.in.gov.br", url_pesquisa, sep = "")
print(url_pesquisa)
```

Agora temos nosso endereço completo. Vamos [acessá-lo e inspecionar sua estrutura ](https://www.in.gov.br/web/dou/-/aviso-de-retificacao-300125659) procurando os nodes realitvos a data, título, seção de publicação, título e do texto em si. 

Agora que fez isso, vamos dar uma olhada na alternativa que adotei para esses caminhos. Aqui já faremos tudo em uma única etapa utilizando o *pipe*, já considerando as possíveis correções de símbolos que algum dos textos precise:

```{r}
pagina <- read_html(url_pesquisa)
  
data <- pagina %>% 
  html_nodes(xpath = "//*[@id='materia']//p[1]/span[2]") %>% 
  html_text()
  
edicao <- pagina %>% 
  html_nodes(xpath = "//*[@id='materia']//p[1]/span[5]") %>% 
  html_text()
  
secao_pagina <- pagina %>% 
  html_nodes(xpath = "//*[@id='materia']//p[1]/span[7]") %>% 
  html_text() %>% 
  str_squish() 
  
titulo <- pagina %>% 
  html_nodes(xpath = "//*[@id='materia']//div[4]/p[1]") %>% 
  html_text()

texto <- pagina %>% 
  html_nodes(xpath = "//*[@id='materia']/div/div[4]") %>% 
  html_text() %>% 
  str_squish()
```

Em seguida faremos nosso data_frame juntando todas essas informações e vamos observá-la com a função *View*:

```{r}
tabela <- data.frame(titulo, data, edicao, secao_pagina, texto, url_pesquisa)
View(tabela)
```

Vamos agora aplicar um loop e montar essa tabela com todas as publicações da nossa primeira página de resultados. Primeiro criamos um *data_frame* vazio para empilharmos nossas tabelas geradas em cada página. Para percorrermos o loop, utilizaremos o vetor *$link* dentro do *data_frame* "dados_tabela" e passarmos por todos nossos links. Os elementos que extraíremos de cada página já estão estabelecidos acima, então não sofrem alterações. Por fim, geramos um *data_frame* provisório com a tabela das 6 variáveis em cada página e vamos empilhá-las em nosso *data_frame* principal, o "dados_dou". 

```{r}
dados_dou <- data.frame()

for (i in dados_tabela$link) {
  
  url_pesquisa <- paste("https://www.in.gov.br", i, sep = "")
  
  print(url_pesquisa)
  
  pagina <- read_html(url_pesquisa)
  
  data <- pagina %>% 
    html_nodes(xpath = "//*[@id='materia']//p[1]/span[2]") %>% 
    html_text()
  
  edicao <- pagina %>% 
    html_nodes(xpath = "//*[@id='materia']//p[1]/span[5]") %>% 
    html_text()
  
  secao_pagina <- pagina %>% 
    html_nodes(xpath = "//*[@id='materia']//p[1]/span[7]") %>% 
    html_text() %>% 
    str_squish() 
  
  titulo <- pagina %>% 
    html_nodes(xpath = "//*[@id='materia']//div[4]/p[1]") %>% 
    html_text()
  
  texto <- pagina %>% 
    html_nodes(xpath = "//*[@id='materia']/div/div[4]") %>% 
    html_text() %>% 
    str_squish()
  
  tabela <- data.frame(titulo, data, edicao, secao_pagina, texto, url_pesquisa)

  dados_dou <- rbind(dados_dou, tabela)
    
}
```

Mais uma vez, isso parece familiar, não? A lógica que utilizamos na Folha é a mesma! O que precisamos, quando fazmos nossas raspagens, é identificar as peculiaridades de cada site e automatizar o processo em sequência. E assim, temos as publicações do DOU para a palavra "quilombo" desde o início de 2020.

No fim da semana trabalharemos com a manipulação de texto e retomaremos o banco produzido por aqui.