# Tutorial 8: Coleta de dados no site do Diário Oficial da União, primeira tentativa

## Apresentação do problema

Aprendemos a coletar dados de páginas de busca a partir de atributos e conteúdos das "tags" nos "xpaths" e como automatizar a captura de todas as páginas do resultado. Nossos pŕoximos passos estão relacionados com os mesmos procedimentos para o site do Diário Oficial da União (DOU). Veremos que algumas coisas podem se alterar nos próximos tutoriais, uma vez que lidamos com páginas estáticas e nem sempre é a realidade que encontramos - muitas vezes elas são dinâmicas e requerem outras ferramentas.

Vamos ver o que acontece quando seguimos os mesmos passos aplicados na raspagem da Folha de São Paulo. Num primeiro passo, vamos observar a segunda página do portal de buscas do DOU para o termo "quilombo" a partir do início do ano de 2020:

 

```{r}
url <-  "https://www.in.gov.br/consulta/-/buscar/dou?q=quilombo&s=todos&exactDate=personalizado&sortType=0&delta=20&currentPage=1&newPage=2&score=0&id=300156803&displayDate=1611198000000&publishFrom=01%2F01%2F2020&publishTo=04%2F02%2F2021"
```

Vamos olhar os parâmetros que estão no nosso link. O argumento "q" diz respeito ao termo que requeremos e seus parâmetros se alteram quando mudamos a opção "onde pesquisar". Já o "s" trata da opção "jornal", se em todos ou em algum específico. O "exact_Date" fala do período que procuramos, no caso é personalizado e o período aparece no final da url. O argumento "sortType" trata a forma de ordenamento, se 0 é por data e se 1 é por relevância, enquanto o "delta" é o número de resultados por página. O que nos interessa, na verdade, estão nos argumentos "currentPage" e "newPage", página atual e seguinte da nossa busca e as quais vamos substituir. 

## Padrões e *gsub*

Uma vez que os argumentos relacionados ao número da página estão no meio do url, vamos criar padrões para realizar a substituição no texto com a função *gsub*. Por agora, vamos utilizar os termos "ATUAL" e "PROXIMA" respectivamente:

```{r}
url_base <-  "https://www.in.gov.br/consulta/-/buscar/dou?q=quilombo&s=todos&exactDate=personalizado&sortType=0&delta=20&currentPage=ATUAL&newPage=PROXIMA&score=0&id=300156803&displayDate=1611198000000&publishFrom=01%2F01%2F2020&publishTo=04%2F02%2F2021"
```

Como a segunda página é indicada pelo número 1, começamos pelo 0 para indicar a primeira página. Para substituir o termo "PROXIMA" já realizamos no novo objeto com essa alteração somando 1 ao valor aplicado na página atual:


```{r}
i <- 0
url_pesquisa <- gsub("ATUAL", i, url_base)
url_pesquisa <- gsub("PROXIMA", i+1, url_pesquisa)
print(url_pesquisa)
```

Vimos que agora temos o resultado para a primeira página da pesquisa. Vamos abri-lá no navegador e explorar os "xpaths" dos títulos e links.

## Coletando o conteúdo e o atributo dos links da primeira página

Tente explorar o código do *html* e veja se nota alguma diferença para quando fizemos o procedimento na Folha de São Paulo. Se não notou, sem problemas, é uma diferença sutil e falaremos disso daqui a pouco. 

Como já avançamos um pouco na manipulação de dados, vamos utilizar o dplyr e os pipes para extrair nossos objetos em menos etapas:


```{r}
library(rvest)
library(dplyr)

pagina <- read_html(url_pesquisa)
  
titulo <-  pagina %>% 
  html_nodes(xpath = "//*[@class='title-marker']//a") %>% 
  html_text()
  
link <-  pagina %>% 
  html_nodes(xpath = "//*[@class='title-marker']//a") %>% 
  html_attr(name = "href")
  
tabela <- as.data.frame(cbind(secao, titulo, link))
```

Alguma coisa parece errada, não? Incluímos os "xpaths" e mesmo assim nossa tabela não tem um único resultado. Mas por que aconteceu isso?

Algumas páginas possuem [páginas dinâmicas](https://pt.wikipedia.org/wiki/P%C3%A1gina_din%C3%A2mica#:~:text=Uma%20p%C3%A1gina%20din%C3%A2mica%20em%20geral,aplica%C3%A7%C3%B5es%20para%20intranet%20e%20extranet.), como a do sistema de buscas do DOU. Diferentemente da Folha de São Paulo que possuía uma estrutura estática, existe aqui uma flexibiçlização no conteúdo sempre que abrimos (perceberam que não falamos de alguns dos parâmetros do link? Eles sempre se alteram).

Para solucionar isso, acessaremos de forma remota um navegador que "abrirá" cada uma das páginas dentro do R e permitirá a realização da raspagem utilizando o próprio pacote *rvest*. Mas faremos isso na próxima aula e meia quando acessaremos o DOU com o *RSelenium* e veremos um modelo ainda mais complexo com o Estadão. 

## Preparando o *RSelenium*

Antes de utilizar o RSelenium para raspar páginas precisamos nos preparar. Manteremos a utilização do Google Chrome, mas, para isso, precisamos do *chromedriver* e do *java* atualizado ["Instalando o JRE/JDK padrão" caso use ubuntu](https://www.digitalocean.com/community/tutorials/how-to-install-java-with-apt-on-ubuntu-20-04-pt).

### chromedriver

Para acessarmos o navegador pelo R, precisamos instalar o [chromedriver](https://sites.google.com/a/chromium.org/chromedriver/), que, de forma simples, é uma ferramenta que permite ao Selenium controlar o navegador Chrome. Recomendamos a utilização da última versão estável (stable) para sua versão do Chrome. Para identificar qual a sua versão do navegador, clique nos três pontinhos na extremidade à direita da barra de navegação, depois em "Ajuda" e "Sobre o Google Chrome". A versão instalada na minha máquina, por exemplo, é a 88.0.4324.96, portanto, uso a versão 88.0.4324.96 do driver. Use a mesma da sua versão do Chrome.

Para não precisarmos de um *Docker* que traria muito mais dificuldade para nossa tarefa, utilizamos a função *rsDriver* do RSelenium. Os argumentos que precisamos são o *browser* (o navegador utilizado, aqui o Google Chrome), *chromever* (versão do chromedriver utilizada) e [*port*](https://www.lifewire.com/port-numbers-on-computer-networks-817939), na qual utilizaremos aqui a padrão da função *4567L*.
 
Na sequência, vinculamos um *client* ao servidor para que possamos [abrir e fazer as requisições em linguagem R ao servidor que acessamos](https://www.pawangaria.com/post/automation/what-is-selenium-webdriver/). 

```{r}
rD <- rsDriver(browser = c("chrome"), chromever = "88.0.4324.96",
               port = 4567L)
cliente <- rD$client
```

Vamos testar se está tudo funcionando bem acessando nossa *url_pesquisa* utilizando a função *$navigate*. Caso uma janela do Chrome se abra dentro do R é sinal que conseguimos controlá-la remotamente.

```{r}
cliente$navigate(url_pesquisa)
```

Para encerrar a conexão e fechar o navegador remoto, utilize a função *$close* para o cliente e encerrar a sessão com a função *$server$stop*:

```{r}
cliente$close()
rD$server$stop()
```

Caso tenha algum problema e não consiga acessar a página antes do final da aula, envie um e-mail com um print que mostre a mensagem de erro apresentada no console do R o quanto antes. Precisamos do *RSelenium* funcionando no início da pŕoxima aula.
