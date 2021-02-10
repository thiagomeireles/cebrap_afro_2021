# Tutorial 11: Coleta das notícias no sistema de busca do Estadão

## Apresentação do problema

No último tutorial aprendemos a controlar o nosso navegador Chrome de forma remota utilizando o *RSelenium* e capturamos todos os links das notícias relacionadas ao termo "quilombo" para o último ano. Hoje daremos o passo seguinte que é a captura do conteúdo dessas notícias de forma automatizada. Para isso, vamos repassar rapidamente pelos passos utilizados no último tutorial.

1. Carregamos os pacotes, criamos nosso Driver do Chrome virtual e abrimos nossa conexão com o cliente:

```{r}
library(RSelenium)
library(rvest)
library(dplyr)

rD <- rsDriver(browser = c("chrome"), chromever = "88.0.4324.96",
               port = 4567L)
cliente <- rD$client
```

2. Abrimos a página de login do Estadão e aceitamos os cookies:

```{r}
url_login <- "https://acesso.estadao.com.br/login"

cliente$navigate(url_login)

botao_cookies <- cliente$findElement(using = "xpath", "//*[@id='site-container']/div[5]/div/div[2]/button")
botao_cookies$clickElement()
```

3. Preenchemos as informações de login e agora estamos conectados como asssinantes:

```{r}
email <- cliente$findElement(using = "xpath", "//*[@id='email_login']")
email$sendKeysToElement(list("meireles.t@hotmail.com"))

senha <- cliente$findElement(using = "xpath", "//*[@id='senha']")
senha$sendKeysToElement(list("cebraplabafro"))

login <- cliente$findElement("xpath", "//*[@id='btn-login']")
login$clickElement()
```

4. Abrimos o sistema de buscas e preenchemos para o termo "quilombo", selecionando o último ano e apenas as notícias:

```{r}
url_pesquisa <- "https://busca.estadao.com.br/"

cliente$navigate(url_pesquisa)

termo_busca <- cliente$findElement(using = "xpath", "//section[3]//div[2]/input[3]")
termo_busca$sendKeysToElement(list("quilombo"))

descer <- cliente$findElement("xpath", "/html/body/section[4]/div/section[1]/div/section[1]/h5")
cliente$mouseMoveToLocation(webElement = descer)

periodo <- cliente$findElement("xpath", "//section/form/section/div/nav/a")
periodo$clickElement()

definir_periodo <- cliente$findElement("xpath", "/html/body/section[3]/div/section/form/section/div/nav/ul/li[6]/button")
definir_periodo$sendKeysToElement(list("No último ano"))

tipo_conteudo <- cliente$findElement("xpath", "/html/body/section[3]/div/section/form/div[3]/nav/ul/li[2]/button")
tipo_conteudo$clickElement()
```

5. Descemos e clicamos em "carregar mais" diversas vezes para que todos os resultados fossem abertos para capturar os links:

```{r}
for (i in 1:8) {
  print(i)
  posicao <- cliente$findElement("xpath", "//*[@id='tinfi-0-3']")
  cliente$mouseMoveToLocation(webElement = posicao)
  Sys.sleep(3)
  mudar <- cliente$findElement("xpath", "/html//div/a[@class='go more-list-news btn-mais fn brd-e']")
  mudar$clickElement()  
  Sys.sleep(2)
}
```

6. Acessamos a página com as ferramentas do *rvest* para extrair os títulos e os links das matérias

```{r}
pagina <- read_html(cliente$getPageSource()[[1]])

titulos <- pagina %>% 
  html_nodes(xpath = "//a/h3[@class='third']") %>% 
  html_text()

links <- pagina %>% 
  html_nodes(xpath = "//a[@class='link-title']") %>% 
  html_attr(name = "href")

secao <- pagina %>% 
  html_nodes(xpath = "//h4[@class='cor-e']") %>% 
  html_text()

dados_links <- data.frame(titulos, links, secao)
```

Após relembrarmos rapidamente (e rodarmos o código para voltarmos ao estágio em que paramos), nosso pŕoximo e último passo na captura aqui é acessar os links e capturar seu conteúdo.

## Capturando as notícias do Estadão

Nesse ponto retomamos a importância de estarmos logados no Estadão, uma vez que para acessar seu conteúdo e inspecionar os elementos que queremos precisamos do login. Vamos trabalhar com o primeiro resultado da nossa pesquisa.

```{r}
url_noticia <- dados_links$links[1]
print(url_noticia)
```
Agora que identificamos, abrimos nossa página para inspecionar e buscar os elementos que nos interessam.

```{r}
cliente$navigate(url_noticia)
```

Você perceberá uma diferença entre inspecionar uma página pelo nosso navegador remoto e pelo navegador em nosso computador. Diferentemente do que fizemos em nossos navegadores, o que controlamos pelo R não indica automaticamente qual o *node* que está relacionado ao elemento que clicamos. Isso ocorre porque ele é controlado de forma automática pelo R e as entradas manuais não são "naturais". Dessa forma, para explorar o conteúdo de nossa notícia, acesse o link do objeto "url_noticia" para realizar a inspeção. 

```{r}
pagina <- read_html(url_noticia)
```

Com a nossa notícia identificada pelo R como uma página estática, vamos identificar os *xpaths* para alguns elementos: título, subtítulo, autores, data e hora, além do texto da notícia. Após identificar quais os nós, já realizaremos a extração do texto usando um *pipe* do dplyr:

```{r}
titulo <- pagina %>% 
  html_nodes(xpath = "//h1[@class='n--noticia__title']") %>% 
  html_text()

subtitulo <- pagina %>% 
  html_nodes(xpath = "//h2[@class='n--noticia__subtitle']") %>% 
  html_text()

autores <- pagina %>% 
  html_nodes(xpath = "//*[@class='n--noticia__state']/p[1]/span") %>% 
  html_text()

data_hora <- pagina %>% 
  html_nodes(xpath = "//*[@class='n--noticia__state']/p[2]") %>% 
  html_text()

texto <- pagina %>% 
  html_nodes(xpath = "//div[3]/p/span") %>% 
  html_text()
```

Você deve ter percebido que, assim como as notícias da Folha, o texto do Estadão é um vetor de cinco valores. Vamos observar se está tudo correto imprimindo o conteúdo:

```{r}
print(texto)
```

Agora sabemos que somente estão separados parágrafos/linhas e já vimos como lidar com isso. Vamos aplicar a função *paste* para tornar o objeto um vetor atômico, assim como os demais que extraímos da página. Utilizaremos um espaço como separador no argumento *collapse*:

```{r}
texto <- paste(texto, collapse = " ")
```

O próximo passo é gerar uma tabela que agregue toda essa informação. Utilizaremos a função *data.frame* para isso:

```{r}
tabela <- data.frame(titulo, subtitulo, autores, data_hora, texto, url_pesquisa)
```

Pronto, criamos nossa tablea com os 6 valores extraídos da nossa pesquisa. O próximo passso é realizar um *loop* para extrair o conteúdo de todas as notícias. Já vimos isso bastante, então vamos aplicá-lo utilizando os vetores *$links* do *data_frame* "dados_links". Agruparemos cada tabela a um novo *data_frame* que criaremos antes do loop, o "dados_estadao". 

No entanto aqui temos algumas dificuldades adicionais. A primeira é que o Estadão, diferentemente da Folha, possui estruturas da página completamente diferentes para cada caderno. Nesse sentido, vale a pena "extrair" todos os links das pesquisas para todos os termos ou todo o período de interesse e depois realizar a raspagem apropriada para cada seção (você deve ter percebido que esse objeto foi acrescentado quando pegamos o link). A segunda dificuldade está relacionada à ausência de subtítulos em algumas das matérias, e, dessa forma, quando não existir substituiremos por *NA* para indicar que não existem e evitar erros. Por fim, em algumas das páginas o node para autor e data e hora é diferente, assim, quando não o encontrarmos, diremos para que ele procure pelo segundo. Faremos a raspagem, dessa forma, somente para a seção Política por enquanto, à qual pertence a página que raspamos para conhecer os *xpaths*.

```{r}
dados_estadao <- data.frame()

for(i in subset(dados_links, secao=="Política")$links){
  
  print(i)
  
  link <- i

  pagina <- read_html(link)
  
  titulo <- pagina %>% 
    html_nodes(xpath = "//div[1]/section/article/h1") %>% 
    html_text()
  
  subtitulo <- pagina %>% 
    html_nodes(xpath = "//h2[@class='n--noticia__subtitle']") %>% 
    html_text()
  
  subtitulo[length(subtitulo) == 0] <- NA_character_

  autores <- pagina %>% 
    html_nodes(xpath = "//*[@class='n--noticia__state']/p[1]/span") %>% 
    html_text()
  
  autores[length(autores) == 0] <- pagina %>% 
    html_nodes(xpath = "//div[@class='n--noticia__state-title']") %>% 
    html_text()

  data_hora <- pagina %>% 
    html_nodes(xpath = "//*[@class='n--noticia__state']/p[2]") %>% 
    html_text()
  
  data_hora[length(data_hora) == 0] <- pagina %>% 
    html_nodes(xpath = "//div[@class='n--noticia__state-desc']/p") %>% 
    html_text()
  
  texto <- pagina %>% 
    html_nodes(xpath = "//div[3]/p") %>% 
    html_text()

  texto <- paste(texto, collapse = " ")
  
  tabela <- data.frame(titulo, subtitulo, autores, data_hora, texto, link)
  
  dados_estadao <- rbind(dados_estadao, tabela)

}
```

Perceba que as alterações nas classes de autores e data e hora são muito sutis, mas fazem toda a diferença na hora de fazermos a raspagem. Caso queira observar a diferença, "cancele" o código da segunda tentativa de captura de autores ou data e hora e terá impresso o link de acesso no console. 

Como já disse algumas vezes, automatizamos o processso para ganho em escala de um trabalho que possui um componente "artesanal" muito forte. Espero que percebam isso para lidar com essas pequenas dificuldades que aparecem (e são muito mais rápidas de resolver do que um copia e cola de cada página).
