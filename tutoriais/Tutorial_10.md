# Tutorial 8: Inícido da coleta de dados no sistema de busca do Estadão

## Apresentação do problema

Agora já temos uma experiência com a raspagem de dados da Folha de São Paulo e do DOU. Agora vamos aprender algumas novas ferramentas do *RSelenium* trabalhando com o sistema de busca do Estadão. Aqui aprenderemos a preencher alguns formulários e opções da pesquisa de forma remota utilizando o próprio R. Assim como o DOU, a página de pesquisa do Estadão é dinâmica, mas também utilizaremos algumas das ferramentas que aprendemos com o *rvest*.

Primeiro vamos carregar os pacotes que utilizaremos no tutorial e abrir nossa conexão remota com o Chrome usando a função *rsDriver* e criar nosso cliente para conexão:

```{r}
library(RSelenium)
library(rvest)
library(dplyr)

rD <- rsDriver(browser = c("chrome"), chromever = "88.0.4324.96",
               port = 4567L)
cliente <- rD$client
```

## Login 

Diferentemente do que fizemos nos sites da Folha e do DOU, não conseguimos abrir as notícias do Estadão sem login de assinante. Dessa forma, vamos acessá-la com nosso cliente e "aceitar" os cookies com um código de R. Primeiro identificamos qual o "xpath" do botão e utilizamos a função *$findElement*; em um segundo momento "clicamos" no botão usando a função *$clickElement*:

```{r}
url_login <- "https://acesso.estadao.com.br/login"

cliente$navigate(url_login)

botao_cookies <- cliente$findElement(using = "xpath", "//*[@id='site-container']/div[5]/div/div/div[2]/button")
botao_cookies$clickElement()
```

Bacana, não? Com o RSelenium temos diversas opções para navegar pela página e esta foi a primeira que veremos nesse tutorial. A segunda vem na sequência, quando preencheremos os campos de e-mail e senha do formulário encontrando os "xpaths" desses elementos e aplicando a *$findElement*; na sequência, inserimos nossas informações de login em cada um deles usando a função *$sendKeysToElement* a partir de uma lista que inserimos no argumento. Por fim, identificamos o botão e clicamos nele como no chunk anterior:

```{r}
email <- cliente$findElement(using = "xpath", "//*[@id='email_login']")
email$sendKeysToElement(list("meireles.t@hotmail.com"))

senha <- cliente$findElement(using = "xpath", "//*[@id='senha']")
senha$sendKeysToElement(list("cebraplabafro"))

login <- cliente$findElement("xpath", "//*[@id='btn-login']")
login$clickElement()
```

Agora estamos logados no site do Estadão com uma assinatura digital (não sei o quanto meu login e senha funcionarão ao mesmo tempo para todos rs).

## Definindo os parâmetros da busca

Lembram que no Tutorial 9 comentei que era também era possível preencher os parâmetros que queremos a partir da página padrão de busca do DOU? Falamos em explorar isso em outro momento - que é agora. Vamos abrir o sistema de buscas no nosso Driver do navegador:

```{r}
url_pesquisa <- "https://busca.estadao.com.br/"

cliente$navigate(url_pesquisa)
```

Observando rapidamente a página, vemos que podemos (1) inserir os termos de busca; (2) definir o período e (3) escolher o tipo de conteúdo. Continuaremos explorando as funções *$findElement*, *$sendKeysToElement* e *$clickElement* para realizar nossa busca de forma remota.

No entanto, para tornar objetos "clicáveis" precisamos que o navegador os esteja exibindo. Dessa forma, por exemplo, para permitir que cliquemos no filtro por data, precisamos descer nosso cursor pela página de busca. Como nosso objetivo é automatizar todo o processo, precisamos de uma função que faça isso por nós para todos os termos que temos interesse de pesquisar. Para isso utilizamos a função *mouseMoveToLocation* aplicada a um objeto próximo aos botões que reremos clicar. Utilizaremos o objeto de texto com os diferentes nomes de editoriais antes de acessar o período e o tipo de conteúdo:

```{r}
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

Pronto! Conseguimos preencher nossa pesquisa com os parâmetros "quilombo" para o "último ano" selecionando apenas "notícias". O próximo passo é "rolar" a página para clicarmos em "carregar mais". É importante entender que temos 10 resultados por página e 89 resultados, logo precisaremos descer e clicar por 8 vezes.

Utilizaremos um *loop* para clicarmos as 8 vezes no botão, indo de 1 a 8. Teremos uma nova função aqui, a *Sys.sleep* que gera um "delay" para que tenhamos a rolagem da página. Criamos tempo de espera de 3 segundos entre o comando para mover nossa página e outro de 2 segundos após clicar e esperar pelo carregamento.

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

## Capturando títulos e links

Vamos acessar agora o "html" com a função *read_html* aplicada à página como está após carregarmos todas as notícias para a palavra quilombo no último ano. Os passos já são conhecidos. Extrairemos os títulos e os links utilizando os nós e os atributos para construirmos um *data_frame* com as duas informações.

```{r}
pagina <- read_html(cliente$getPageSource()[[1]])

titulos <- pagina %>% 
  html_nodes(xpath = "//a/h3[@class='third']") %>% 
  html_text()

links <- pagina %>% 
  html_nodes(xpath = "//a[@class='link-title']") %>% 
  html_attr(name = "href")

dados_links <- data.frame(titulos, links)
```

E conseguimos, mais uma vez, coletar as informações que queremos para automatizar a coleta das notícias no início da pŕoxima aula.