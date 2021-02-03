#Tutorial 4: Introdução ao *rvest* e revisão

## Pacotes *rvest*

Nesse primeiro momento, precisamos de uma biblioteca chamada *rvest*. Ela possui funções utilizadas para facilitar o download e a manipulação de conteúdo proveniente de *html* e *xml*. Assim, nossa primeira ação é realizar o processo de instalação de uma biblioteca. Para isso, basta executarmos o comando abaixo:

```{r}
install.packages("rvest")
```

No entanto, mesmo com a biblioteca instalada as funções não ficam disponíveis automaticamente. É necessário carregar a biblioteca para torná-las disponíveis. Assim, vamos executar o comando para tornar as funções da biblioteca *rvest* disponíveis. Basta executar o comando abaixo:

```{r}
library(rvest)
```

O pacote _rvest_ faz parte de um "universo" de pacotes camado _tidyverse_. O _tidyverse_ é uma compilação de diversas bibliotecas que, grosso modo, compõem uma linguagem "alternativa" dentro do R. Os pacotes mais conhecidos são o _dplyr_ e o _ggplot2_.

Diversas funções que fazem parte do _tidyverse_ serão utilizadas ao longo da semana e serão destacadas quando surgirem. No entanto, já vamos realizar a instalação do pacote. Aqui, no entanto, faremos um processo um pouco mais complexo: pediremos para que o R cheque se o pacote já está instalado e, caso não esteja, realize a instalação.

```{r}
if (!require("tidyverse")) install.packages("tidyverse"); library(tidyverse)
```

Dica: se você "chamar" o pacote *tidyverse*, não precisará chamar *rvest*, pois a função do *tidyverse* é carregar todos os pacotes que o compõem.

## Capturando o conteúdo de uma página com *rvest*

Para acessar o conteúdo de uma página, precisamos de funções que façam algo semelhante a um navegador de internet, ou seja, que se comuniquem com o servidor da página e receba o seu conteúdo. Para capturar uma página, ou melhor, o código HTML no qual a página está escrita, utilizamos a função *read\_html*, do pacote *rvest*. Vamos usar um exemplo com a Wikipedia.

```{r}
url <- "https://pt.wikipedia.org/wiki/Lista_de_pa%C3%ADses_e_territ%C3%B3rios_por_%C3%A1rea"
pagina <- read_html(url)
print(pagina)
```

O resultado é um documento "xml_document" que contém o código html que podemos inspecionar usando o navegador. Vamos entender no próximo tutorial o que é um documento XML, por que páginas em HTML são documentos XML e como navegar por eles. Por enquanto, basta saber que ao utilizarmos *read\_html*, capturamos o conteúdo de uma página e o armezenamos em um objeto bastante específico.

Neste tutorial, trabalharemos com um objeto bastante específico em páginas de internet: as tabelas. No caso da página que abrimos, temos a lista dos países de maior área no mundo divididos por categorias. Abrindo a página na [Wikipedia](https://pt.wikipedia.org/wiki/Lista_de_pa%C3%ADses_e_territ%C3%B3rios_por_%C3%A1rea), podemos ver que esses dados estão dentro de uma tabela quando clicamos com o botão direito sobre os dados e inspecionamos o conteúdo.

O *rvest_*possui uma função específica para ler as tabelas e fazer o download para nosso ambiente global, a *html\_table*. Ela lê um "xml_document" e extrai TODAS as tabelas escritas em HTML da url e retorna uma lista (lembra delas do tutorial pré curso? Voltaremos a elas mais adiante).

```{r}
tabelas_wiki <- html_table(pagina)
class(tabelas_wiki)
str(tabelas_wiki)
```

Lembre-se que utilizando as funções *class* e *str* conseguimos identificar o tipo de objeto gerado e como ele foi estruturado. Geramos em uma lista com 7 tabelas. Voltaremos a elas mais tarde.

## Atividade inicial - Pesquisa no site da Folha de São Paulo

### For loop e links com numeração de página

Vamos começar visitando o site da Folha de São Paulo e entrar na ferramenta de pesquisa de proposições. Clique [aqui](https://search.folha.uol.com.br/) para acessar a página.

Ao acessar o site, podemos elaborar uma pesquisa que retorne um númedo de respostas que tornaria ineficiente a coleta manual, como, por exemplo, a palavra "quilombo".

Os 1555 resultados estão divididos em 63 páginas com 25 observações cada. Em cada um dos resultados, existem 4 tipos informações básicas: seção do jornal, título da matéria, data da publicação e o hyperlink.

Podemos prosseguir, clicando nos botões de navegação ao final da página, para as demais páginas da pesquisa. Por exemplo, podemos ir para a página 2 clicando uma vez na seta indicando à direita.

OBS: Há uma razão importante para começarmos nosso teste com a segunda página da busca. Em diversos servidores web, como este da Folha, o link (endereço url) da primeira página é "diferente" dos demais. Em geral, os links são semelhantes da segunda página em diante.

Nossa primeira tarefa consiste em capturar estas informações. Vamos, no decorrer da atividade aprender bastante sobre R, objetos, estruturas de dados, loops, captura de tabelas em HTML e manipulação de dados.

Vamos armazenar a URL em um objeto ("url_base", mas você pode dar qualquer nome que quiser).

```{r}
url_base <- "https://search.folha.uol.com.br/search?q=quilombo&site=todos&periodo=todos&results_count=1555&search_time=0%2C117&url=https%3A%2F%2Fsearch.folha.uol.com.br%2Fsearch%3Fq%3Dquilombo%26site%3Dtodos%26periodo%3Dtodos&sr=26"
```

Note que a estrutura do endereço URL é relativamente simples: "q" é o primeiro parâmetro e informa o texto buscado. "site=jornal", informa que a busca é feita em todas as páginas com publicações realizadas na Edição Impressa. Já "periodo" é autoexplicativo, "results_count" é o total de resultados e "search_time" o tempo que demorou para realização da busca. "url" é o próprio endereço base do site de buscas da Folha. E, muito importante, "sr" é o número do primeiro resultado da página, que, no caso da Folha, pula de 25 em 25. Nesse primeiro momento vamos capturar apenas a página 2, mas depois aprenderemos a capturar todas as páginas usando loops.

Podemos ver que há muitas páginas de resultado para a palavra-chave que utilizamos. Nosso desafio é conseguir "passar" de forma eficiente por elas, ou seja, acessar as 53 páginas de resultado e "raspar" o seu conteúdo. Para isso, usaremos uma função essencial na programação, o "for loop".

Loops são processos iterativos e são extremamente úteis para instruir o computador a repetir uma tarefa por um número finito de vezes. Por exemplo, vamos começar "imprimindo" na tela os números de 1 a 9:

```{r}
for (i in 1:9) {
  print(i)
}
```

Simples, não? Vamos ler esta instrução da seguinte maneira: "para cada número i no conjunto que vai de 1 até 9 (essa é a parte no parênteses) imprimir o número i (instrução entre chaves)". E se quisermos imprimir o número i multiplicado por 7 (o que nos dá a tabuada do 7!!!), como devemos fazer?

```{r}
for (i in 1:9) {
  print(i * 7)
}
```

Tente agora construir um exemplo de loop que imprima na tela os números de 3 a 15 multiplicados por 10 como exerício.

```{r, echo=FALSE, eval=FALSE}
for (i in 3:15) {
  print(i * 10)
}
```

### Substituição com *gsub*

Cumprimos uma etapa importante: observamos o funcionamento dos loops de forma intuitiva. O próximo passo é fazer com que ele "passe" pelas páginas que contém a informação que nos interessa. Assim, devemos instruir o programa a passar pelas páginas de 1 a 177, substituindo apenas o número da página atual -- "currentPage" -- no endereço URL que guardamos no objeto url_base.

Para isso precisamos, no entando, de uma função que permita substituir no texto básico do URL ("url_base") o número da página. Ainda que existam diferentes formas para se realizar essa tarefa, aqui utilizaremos a função *gsub*, a qual já conhecemos dos tutoriais anteriores. Ela é uma função básica da linguagem e permite a substituição de um pedaço de um objeto de texto por outro a partir de um critério especificado. 

Os argumentos (o que vai entre os parenteses) da função são, em ordem, o termo a ser substituído, o termo a ser colocado no lugar e o objeto no qual a substituição ocorrerá. Na prática, ela funciona da seguinte forma:

```{r}
o_que_procuro_para_susbtituir <- "palavra"
o_que_quero_substituir_por <- "batata"
meu_texto <- "quero substituir essa palavra"

texto_final <- gsub(o_que_procuro_para_susbtituir, o_que_quero_substituir_por, meu_texto)

print(texto_final)
```

Agora que sabemos substituir partes de textos e fazer loops, podemos mudar o número da página do nosso endereço de pesquisa.

Já observamos que na URL, o que varia de uma página para outra é o número do primeiro resultado da página na expressão "sr". De outra forma, vamos da "sr=1" para "currentPage=26" e assim por diante. Sabemos que, em nossa pesquisa, o "1" é o primeiro valor da primeira página e o "26" da segunda, e assim se segue. As páginas possuem peculiaridades e isso torna importante conhecer a página que queremos "raspar". No fim, a captura de informações em páginas de internet é quase uma atividade "artesanal". 

Posto isso, vamos substituir na URL da página 2 de nossa busca o número por uma expressão que "guarde o lugar" do número da página. Esse algo é um "placeholder" e pode ser qualquer texto. No caso, usaremos "REFERENCIA". Veja abaixo onde "REFERENCIA" foi introduzido no endereço URL. 

Obs: Lembremos que ao colocar na URL, não devemos usar as aspas. Ainda assim devemos usar aspas ao escrever "REFERENCIA" como argumento de uma função, pois queremos dizer que procuramos a palavra "REFERENCIA" e não o objeto chamado REFERENCIA.

```{r}
url_base <- "https://search.folha.uol.com.br/search?q=quilombo&site=todos&periodo=todos&results_count=1555&search_time=0%2C117&url=https%3A%2F%2Fsearch.folha.uol.com.br%2Fsearch%3Fq%3Dquilombo%26site%3Dtodos%26periodo%3Dtodos&sr=REFERENCIA"
```

Por exemplo, se quisermos gerar o link da página 6, podemos escrever:

```{r}
url <- gsub("REFERENCIA", "51", url_base)

print(url)
```
Ou, em vez de usar um número diretamente na substituição, podemos usar um vetor que represente um número -- por exemplo a variável i, que já usamos no loop anteriormente.

```{r}
i <- 51
url <- gsub("REFERENCIA", i, url_base)

print(url)
```

Agora que temos o código substituindo funcionando, vamos implementar o loop para que as URLs das páginas sejam geradas automaticamente. Por exemplo, se quisermos "imprimir" na tela as páginas 0 a 5, podemos usar o seguinte código:

```{r}
url_base <- "https://search.folha.uol.com.br/search?q=quilombo&site=todos&periodo=todos&results_count=1555&search_time=0%2C117&url=https%3A%2F%2Fsearch.folha.uol.com.br%2Fsearch%3Fq%3Dquilombo%26site%3Dtodos%26periodo%3Dtodos&sr=REFERENCIA"

for(i in 0:5){
  url <- gsub("REFERENCIA", 1+i*25, url_base)
  print(url)
}
```

Note que aqui a função é de substituição obedece a particularidade da nossa página. Temos a expressão "1+i*25" como o que vai substituir, uma vez que 1 é o primeiro restulado, i representa o avanço da página e 25 o número de resultados em cada uma. Mas por que começamos um 0? Porque na primeira página não adicionamos os 25 resultados como referência ao primeiro valor.


### Listas

Um detalhe fundamental do resultado das funções do pacote *rvest* é que o resultado vem em lista. Por que uma lista? Porque pode haver mais de uma tabela página e cada tabela ocupará uma posição na lista, como vimos aqui, ou nas outras estruturas que veremos a seguir. Para o R, uma lista pode combinar objetos de diversas classes: vetores, data frames, matrizes, etc.

Ao rasparmos as página da Wikipedia, a função _html\_table_ retornornou sete tabelas e não apenas as listas dos países, que é o que queríamos.

Como acessar objetos em uma lista? Podemos ulitizar colchetes. Porém, se utilizarmos apenas um colchete, estamos obtendo uma sublista. Por exemplo, vamos criar diferentes objetos e combiná-los em uma lista:

```{r}
# Objetos variados
matriz <- matrix(c(1:6), nrow=2)
vetor.inteiros <- c(42:1)
vetor.texto <- c("a", "b", "c", "d", "e")
vetor.logico <- c(T, F, T, T, T, T, T, T, F)
texto <- "meu texto aleatorio"
resposta <- 42

# Lista
minha.lista <- list(matriz, vetor.inteiros, vetor.texto, vetor.logico, texto, resposta)
print(minha.lista)
```

Para produzirmos uma sublista, usamos um colchete (mesmo que a lista só tenha um elemento!):

```{r}
print(minha.lista[1:3])
class(minha.lista[1:3])
print(minha.lista[4])
class(minha.lista[4])
```

Se quisermos usar o objeto de uma lista, ou seja, extraí-lo da lista, devemos usar dois colchetes:

```{r}
print(minha.lista[[4]])
class(minha.lista[[4]])
```

Ao obtermos uma lista de tabelas de uma página (nem sempre vai parecer que todos os elementos são tabelas, mas são, pelo menos para um computador que "lê" HTML), devemos utilizar dois colchetes para extrair a tabela que queremos. 

Se lembra da lista com os países de maior área no mundo extraídos da Wikipedia? Podemos extrair, por exemplo, os países com mais de um milhão de metros quadrados de área em um *dataframe_* - o que normalmente manipulamos como base de dados em outros softwares. Para isso, criamos um novo objeto contendo apenas o conteúdo _[[1]]_ da nossa lista _tabelas_wiki_:

```{r}
gigantes <- tabelas_wiki[[1]]
head(gigantes)
```

Bem mais bonito, não? No entanto, os dados estão bagunçados. Retomaremos adiante.

## Data\_Frames

Excelente, não? Mas e aí? Cadê os dados? O problema é que até agora ainda não fizemos nada com os dados, ou seja, ainda não guardamos eles em novos objetos para depois podermos utilizá-los na análise. Da mesma forma, vimos que as informações estão um pouco bagunçadas, não?

Neste último passo, vamos fazer o seguinte: precisamos de uma estrutura que armazene as informações, então criamos um data frame vazio (chamado "dados") e, para cada iteração no nosso loop (ou seja, para cada "i"), vamos inserir a tabela da página i como novas linhas no nosso data frame. A função nova que precisamos se chama _bind\_rows_, que é parte do pacote _dplyr_, protagonista do _tidyverse_. Ela serve para unir diferentes data frames (ou vetores ou matrizes), colocando suas linhas uma debaixo da outra. Vejamos um exemplo antes de avançar:

```{r}
# Criando 2 data frames separados
meus.dados1 <- data_frame("id" = 1:10, "Experimento" = rep(c("Tratamento"), 10))
print(meus.dados1)

meus.dados2 <- data_frame("id" = 11:20, "Experimento" = rep(c("Controle"), 10))
print(meus.dados2)

# Combinando os dois data.frames
meus.dados.completos <- bind_rows(meus.dados1, meus.dados2)
print(meus.dados.completos)
```

## Captura das tabelas com armazenamento em data frames

Pronto. Podemos agora criar um data frame vazio ("dados") e preenchê-lo com os dados capturados em cada iteração. O resultado final será um objeto com todas as tabelas de todas as páginas capturadas, que é o nosso objetivo central. 

Vamos trabalhar apenas com a lista de tabelas da Wikipedia. 

Obs: vamos inserir um "contador" das tabelas capturadas com "print(i)". Isso será muito útil quando quisermos capturar um número grande de páginas fazedndo as substituições nas URL de referência, pois o contador nos dirá em qual iteração (sic, é sem "n" mesmo) do loop estamos.

```{r}
dados <- data.frame()

for (i in c(1,3,4,5)) {

  print(i)
  
  tabela <- tabelas_wiki[[i]]
  
  colnames(tabela)[3] <- "Área (Km)2"
  
  dados <- bind_rows(dados, tabela)
}
```

Note que não incluímos a segunda tabela, uma vez que a "variável" de área é uma lista e isso impediria que o loop fosse concluído. Quando formos para as páginas de internet, temos que nos atentar à estrutura que construímos o loop para evitar problemas como esse. Por agora não lidaremos com isso, mas veremos como lidar com diferentes estruturas nos próximos tutoriais porque não queremos perder dados.

No fim, ficamos com 4 variáveis  do tipo "character", mas precisamos checar se estão estruturadas. As 6 observações apresentam o resultado adequado, o que nos dá uma boa dica que que tudo ocorreu bem até a última página capturada.

Vamos observar o resultado utilizando a função "str" (abreviação de structure), que retorna a estrutura do data frame, e "tail", que é como a função "head", mas retorna as 6 últimas em vez das 6 primeiras observações.

```{r}
# Estrutura do data frame
str(dados)

# 6 primeiras observações
head(dados)

# 6 últimas observações
tail(dados)
```

Com a função *View* podemos ver os dados em uma planilha.

```{r}
View(dados)
```