# Tutorial Pré-Curso I: data_frames

# Pacotes no R

Muitas ações que precisamos em atividades diversas executadas não fazem parte da biblioteca básica do R, mas outros desenvolvedores já desenvolveram funções para isso. Em muitos casos, diversas funções são compiladas em novas bibliotecas direcionadas para atividades bem específicas. Essas bibliotecas (ou pacotes de funções) são disponibilziadas pela comunidade de R e, após aprovação, vão para um diretório com bibliotecas já testadas, o [CRAN](https://cran.r-project.org/web/packages/policies.html).

Nesse primeiro momento, vamos instalar uma de uma biblioteca chamada *tidyverse*, falaremos um pouco mais dela depois. Assim, nossa primeira ação é realizar o processo de instalação de uma biblioteca.Para isso, basta executarmos o comando abaixo:

```{r}
install.packages("tidyverse")
```

No entanto, mesmo com a biblioteca instalada as funções não ficam disponíveis automaticamente. É necessário carregar a biblioteca para torná-las disponíveis. Assim, vamos executar o comando para tornar as funções da biblioteca *tidyverse* disponíveis. Basta executar o comando abaixo:

```{r}
library(tidyverse)
```

Excelente! Já temos boa parte das funções que precisamos disponíveis para o nosso primeiro tutorial. Vamos utilizá-las logo mais.

# Tidyverse

 O _tidyverse_ é uma compilação de diversas bibliotecas que, grosso modo, compõem uma linguagem "alternativa" dentro do R. Os pacotes mais conhecidos são o _dplyr_ e o _ggplot2_, também utilizaremos o _rvest_ para capturar dados das páginas da internet.

Diversas funções que fazem parte do _tidyverse_ serão utilizadas ao longo da semana e serão destacadas quando surgirem. Uma outra forma de checar se ele está instalado é utilizando um processo um pouco mais complexo: pediremos para que o R cheque se o pacote já está instalado e, caso não esteja, realize a instalação. Façamos isso pro *dplyr*.

```{r}
if (!require("dplyr")) install.packages("dplyr"); library(dplyr)
```

Dica: se você "chamar" o pacote _tidyverse_, não precisará chamar *dplyr*, pois a função do _tidyverse_ é carregar todos os pacotes que o compõem.

# Os *data_frames*

Uma característica distintiva da linguagem de programação R é ter sido desenvolvida para a análise de dados. E quando pensamos em análise de dados, a protagonista do show é a *base de dados* ou, como vamos conhecer a partir de agora, *data_frame*. Começaremos daqui antes de seguir para aspectos mais básicos da linguagem, como quais são os tipos de objetos mais comuns.

Por esta razão, em vez de aprender como fazer aritmética, elaborar funções ou executar loops para repetir tarefas e outros aspectos básicos da linguagem, vamos olhar antes para o R como um software concorrente dos demais utilizados para análise de dados em ciências sociais, como SPSS, Stata, SAS e companhia.

As principais características de um data frame são: (1) cada coluna representa uma variável (ou característica) de um conjunto de observações; (2) cada linha representa uma observação e contém os valores de cada variável para tal observação. Vejamos um exemplo:

| Candidato | Partido | Votos | 
| --------- |:-------:| -----:|
| Beatriz   | PMDB    |   350 | 
| Danilo    | SOL     |  1598 | 
| Pedro     | PTB     |   784 | 
| Davi      | PSD     |   580 | 
| Mateus    | PV      |   2   | 

Note que em uma linha os elementos são de tipos de diferentes: na primeira coluna há uma nome (texto), na segunda uma sigla de partido (texto, mas limitado a um conjunto de siglas) e na terceira votos (número inteiros). 

Por outro lado, em cada coluna há somente elementos de um tipo. Por exemplo, há apenas números inteiros na coluna votos. Colunas são variáveis e por isso aceitam registros de um único tipo. Se você já fez um curso de estatísticas básica ou de métodos quantitativos deve se lembrar que as variáveis são classificadas da seguinte maneira:

1- Discretas

  - Nominais, que são categorias (normalmente texto) não ordenadas
  
  - Ordinais, que são categorias (normalmente texto) ordenadas
  
  - Inteiros, ou seja, o conjunto dos números inteiros

2- Contínuas, números que podem assumir valores não inteiros

Se destacamos uma coluna do nosso data frame, temos um *vetor*. Por exemplo, a variável "Votos" pode ser presentado da seguinte maneira: {350, 1598, 784, 580, 2}. Um data frame é um conjunto de variáveis (vetores) dispostos na vertical e combinados um ao lado do outro.

Data frame e vetores são *objetos* na linguagem R.

Vamos ver como o R representa vetores e data frames na tela. Antes disso, é preciso "abrir" um data frame.

## Pausa para pacotes

Uma das características mais atrativas da linguagem R é o desenvolvimento de *pacotes* pela comunidade de usuários. Pacotes são conjuntos de funções (os "comandos") e, por vezes, guardam também dados em diversos formatos.

Vamos carregar um pacote chamado *datasets*, que contém diversos conjuntos de dados úteis para fins didáticos. Para carregar um pacote (e, portanto, tornar as funções disponíveis para uso naquela sessão de R) usamos a função *library*:

```{r}
library(datasets)
```

## De volta aos dataframes

Com a função *data* podemos carregar um conjunto de dados disponível na sua sessão de R. Utilizaremos a base *mtcars*, que contém dados da revista *Motor Trend US* sobre características (variáveis) de 32 automóveis (esse é um dos conjuntos de dados mais populares em cursos introdutórios de R).

```{r}
data(mtcars)
```

Pronto! Logo mais veremos como abrir conjuntos de dados de outras fontes (arquivos de texto, outros softwares, etc), mas já podemos começar a trabalhar com *data frames*.

Antes de avançar, vamos usar o *help* (documentação aqui, mas é como a "ajuda" de outros softwares) do R para descobrir o que há no *data_frame* chamado *mtcars*:

```{r}
?mtcars
```

Você pode ler com calma antes de avançar.

Se quiseremos olhar para os dados que acabamos de carregar utilizamos a função *View* (com V maiúsculo mesmo). 

Dica: Aqui cabe indicar que letras maiúsculas são pouco usuais nos nomes de funções em R e, como dica, adote sempre a utilização de letras minúsculas aos nomes dos objetos para facilitar a digitação.

```{r}
View(mtcars)
```

Com apenas 32 observações, não fica tão difícil "olhar" para os dados. Mas e se estívessemos trabalhando com, por exemplo, o total de candidatos vereadores no Brasil em 2020? Ou com todas as matérias publicadas no DOU para determinado tema? Seria o comando *View* seria útil?

## Do editor de planilhas ao R - parte 1

A partir desse ponto no curso vamos resistir à tentação de "olhar" para os dados. O hábito de quem trabalha com editores de planilha como MS Excel ou Libre Office, ou ainda com alguns softwares de análise de dados como SPSS e STATA, é trabalhar "olhando" para os dados, ou seja, para os valores de cada célula de uma base dados.

Você perceberá em pouco tempo que isso não é necessário. Na verdade, é contraproducente.

## Head no lugar de View

Por exemplo, podemos substituir a função *View* pela função *head*. Veja o resultado:

```{r}
head(mtcars)
```

Com apenas as 6 primeiras linhas do *data_frame* temos noção de todo o conjunto. Sabemos rapidamente que os nomes dos carros são o nome de cada uma das linhas, e que o nome das colunas indicam qual característica está armazenada coluna (lembre-se da documentação de *mtcars* que você acabou de ler).

Alternativamente, podemos usar a função *str* (abreviação de "structure"):

```{r}
str(mtcars)
```

Com *str* sabemos qual é a lista de variáveis (colunas) no *data_frame*, de qual tipo são -- no caso, todas são numéricas e vamos falar sobre esse tema mais tarde -- e os primeiros valores de cada uma, além do número total de observações e variáveis mostrados no topo do *output*.

Há outras maneiras de obter o número linhas e colunas de um *data_frame*:

```{r}
nrow(mtcars)
ncol(mtcars)
dim(mtcars)
```

*nrow* retorna o número de linhas; *ncol*, o de coluna; *dim* as dimensões (na ordem linha e depois coluna) do objeto.

*names*, por sua vez, retorna os nomes das variáveis do *data_frame*:

```{r}
names(mtcars)
```

## Argumentos ou parâmetros das funções

Note que em todas as funções que utilizamos até agora, *mtcars* está dentro do parênteses que segue o nome da função. Essa *sintaxe* é característica das funções de R. O que vai entre parêntesis são os *argumentos* ou *parâmetros* da função, ou seja, os "inputs" que serão transformados.

Uma função pode receber mais de um argumento. Pode também haver argumentos não obrigatórios, ou seja, para os quais não é necessário informar nada se você não quiser alterar os valores pré-definidos. Por exemplo, a função *head* contém o argumento *n*, que se refere ao número de linhas a serem *impressas* na tela, pré-estabelecido em 6. Para alterar o parâmetro *n* para 10, por exemplo, basta fazer:

Dicas: Você pode conhecer os argumentos da função na documentação do R usando *?* antes do nome da função; quando utilizando o RStudio, ao apertar "tab" dentro de uma função ele também te indicará possíveis argumentos.

```{r}
head(x = mtcars, n = 10)
```

*x* é o argumento que já havíamos utilizado anteriormente e indica em que objeto a função *head* será aplicada. 

Dicas: você pode omitir tanto "x =" quanto "n =" se você já conhecer a ordem de cada argumento no uso da função. Veja que neste caso estamos utilizando o símbolo "=" sem fazer a atribuição de dados a um objeto, mas para atribuir valores (ou objetos) aos argumento de uma função. Lembra-se que no tutorial anterior falamos sobre isso? Retomamos que para não haver confusão é preferível usar o símbolo "<-" para atribuição e o "=" para as demais situações.

## Pausa para um comentário

Quando editamos um script, podemos fazer comentários no meio do código. Basta usar # e tudo que seguir até o final da linha não será interpertado pelo R como código. Por exemplo:

```{r}
# Imprime o nome das variaveis do data frame mtcars
names(mtcars)

names(mtcars) # Repetindo o comando acima com comentario em outro lugar
```

Comentários são extremamente úteis para documentar seu código. Documentar é parte de programar e você deve pensar nas pessoas com as quais vai compartilhar o código e no fato de que com certeza não se lembrará do que fez em pouco tempo (garanto, você vai esquecer). Eles são essenciais para replicabilidade ou atualização do código.

## Construindo vetores e data frames

Vamos esquecer o *data_frame* com o qual estávamos trabalhando até agora. Para remover um objeto do *Workspace* fazemos:

```{r}
rm(mtcars)
```

Vamos seguir este tutorial de forma menos entediante. Vamos montar um banco de dados de notícias.

Escolha 2 jornais ou portais de notícias diferentes. Vá em cada um deles e colete 2 notícias quaisquer. Em casa notícia, colete as seguintes informações:

- Nome do jornal ou portal (pode abreviar)
- Data da notícia (não precisa coletar a hora)
- Título
- Autor(a)
- Número de palavras no texto (use o MS Word ou Libre Office se precisar - chute se tiver preguiça, mas não perca tempo contando)
- Marque 1 se a notícia for sobre política e 0 caso contrário
- Marque TRUE se a notícia contiver vídeo e FALSE caso contrário

Se quiser ganhar tempo, pode apenas copiar os exemplos do tutorial e repetir a tarefa depois da aula.

Insira as informações nos vetores em ordem de coleta das notícias.

Com cada informação, vamos construir um vetor. Vejam meus exemplos. Começando com o nome do jornal ou portal:

```{r}
jornal <- c("The Guardian", "The Guardian", "Folha de São Paulo", 
            "Folha de São Paulo")
```

Opa, calma! Temos um monte de coisas novas aqui!

A primeira delas é: se você criou corretamente o vetor *jornal*, então nada aconteceu na sua tela, ou seja, nenhum output foi impresso. Isso se deve ao fato de que criamos um novo *objeto*, chamado *jornal* e *atribuímos* a ele os valores coletados sobre os nomes dos veículos nos quais coletamos as notícias. O símbolo de *atribuição* em R é *<-*. Note que o símbolo lembra uma seta para a esquerda, indicando que o conteúdo do vetor será armazenado no objeto _jornal_.

Objetos não são nada mais do que um nome usado para armazenar dados na memória RAM (temporária) do seu computador. No exemplo acima, *jornal* é o objeto e o vetor é a informação armazenada. Uma vez criado, o objeto está disponível para ser usado novamente, pois ele ficará disponível no *workspace*. Veremos adiante que podemos criar um *data_frame* a partir de vários vetores armazenados na memória. Especificamente no RStudio, os objetos ficam disponíveis no painel *Global Environment* (que provavelmente está no canto direito superior da sua tela).

Posso usar o símbolo *=* no lugar de *<-*? Sim. Funciona. Mas nem sempre funciona e esta substituição é uma fonte grande de confusão. Quando entendermos um pouco mais da sintaxe da linguagem R ficará claro. Por enquanto, quando for atribuir um nome a um objeto recém criado, use *<-*.

Se o objetivo fosse criar o vetor sem "guardá-lo" em um objeto, bastaria repetir a parte do código acima que começa após o símbolo de atribuição. *c* (do inglês "concatenate") é a função do R que combina valores de texto, número ou lógicos (lembram do último tutorial?) em um vetor. É um função muito utilizada ao programar em R.

```{r}
c("The Guardian", "The Guardian", "Folha de São Paulo",
  "Folha de São Paulo")
```

Note que há uma quebra de linha no código. Não há problema algum. Uma vez que o parêntese que indica o fim do vetor não foi encontrado, o R entende o que estiver na próxima como continuidade do código (e, portanto, do vetor). Dica: quebras de linha ajudam a vizualizar o código e com o tempo você também usará.

Vamos seguir com nossa tarefa de criar vetores. Já temos o vetor jornal (que você pode observar no workspace). Os demais, na minha coleta de dados, estão a seguir:

```{r}
# Data
# Obs: ha uma funcao em R com nome "data".
# Evite dar a objetos nome de funcoes existentes
data_noticia <- c("10/03/2017", "10/03/2017", "10/03/2017", "10/03/2017")

# Titulo
titulo <- c("'Trump lies all the time': Bernie Sanders indicts president's assault on democracy",
            "BBC interviewee interrupted by his children live on air – video",
            "Bolsista negra é hostilizada em atividade no campus da FGV de SP",
            "Meninas podem ser o que quiserem, inclusive matemáticas")

# Autor
autor <- c("Ed Pilkington ", NA, "Joana Cunha; Jairo Marques", "Marcelo Viana")

# Numero de caracteres
n_caracteres <- c(5873, 207, 1358, 3644)

# Conteudo sobre politica
politica <- c(1, 0, 0, 0)

# Contem video
video <- c(TRUE, TRUE, FALSE, FALSE)
```

Para onde vão os objetos de R criados? Para o workspace. Se quisermos uma fotografia do nosso workspace, usamos a função *ls*, com parêntese vazio (ou seja, sem argumentos além dos pré-estabelecidos):

```{r}
ls()
```

Lembram da remoção de objetos? Podemos utilizar a função *ls* pedindo para que o R remova todos os objetos que estão no *Workspace*:

```{r}
# rm(list=ls()) # Não o utilizaremos agora porque não queremos limpar nosso workspace
```

Vimos um novo argumento que diz respeito a um outro tipo de objeto, *list*. Listas são úteis, principalmente, para organizarmos nosso ambiente de trabalho. No entanto não falaremos delas nesse momento.

## Detalhes importantes nos vetores acima

Retomando alguns detalhes importantes a serem notados no exemplo acima:

- O formato da data foi arbitrariamente escolhido. Por enquanto, o R entende apenas como texto o que foi inserido. Datas são especialmente chatas de se trabalhar e há funções específicas para tanto.
- Os textos foram inseridos entre aspas. Os números, não. Se números forem inseridos com aspas o R os entenderá como texto também.
- Além de textos e números, temos no vetor *vídeo* valores lógicos, TRUE e FALSE. *logical* é um tipo de dado do R (e é particularmente importante).
- O texto do primeiro título que coletei contém aspas. Como colocar aspas dentro de aspas sem fazer confusão? Se você delimitou o texto com aspas duplas, use aspas simples no texto e vice-versa.
- O que são os *NA* no meio do vetor *autor*? Quando coletei as notícias, não consegui identificar o autor(a) de algumas (eram notícias da redação). *NA* é o símbolo do R para *missing values* e lidar com eles é uma das grandes chatices em R.

## Criando um data frame com vetores:

Como vimos acima, *data_frames* são um conjunto de vetores na vertical. Se você introduziu os valores em cada vetor na ordem correta de coleta dos dados, então eles podem ser *pareados* e *combinados*. No meu exemplo, a primeira posição de cada vetor contém as informações sobre a primeira notícia, a segunda posição sobre a segunda notícia e assim por diante. 

Obviamente, se estiverem pareados, os vetores devem ter o mesmo comprimento. Há uma função bastante útil para checar o comprimento:

```{r}
length(jornal)
```

Vamos criar com os vetores que construímos um *data_frame* com o nome *dados*. Vamos produzí-lo, discutir a função *data.frame* e depois examiná-lo:

```{r}
dados <- data.frame(jornal, data_noticia, titulo, autor, n_caracteres, politica, video)
```

A função *data.frame* cria um *data_frame* a partir de vetores ou matrizes (que ainda não vimos). Quando criamos a partir de vetores, automaticamente os nomes das variáveis (colunas) no novo objeto serão os nomes dos vetores. Mas poderíamos querer nomes novos (por exemplo, em inglês). Bastaria fazer:

```{r}
dados <- data.frame(newspaper = jornal, 
                    date = data_noticia, 
                    title = titulo, 
                    author = autor, 
                    n_char = n_caracteres, 
                    politics = politica,
                    video)
```

Usando as funções que aprendemos nessa aula:

```{r}
# 6 primeiras (e unicas, no caso) linhas
head(dados)

# Estrutura do data frame
str(dados)

# Nome das variaveis
names(dados)

# Dimensoes do data frame
dim(dados)
```

## Tipos de dados em R e vetores

Usando o que aprendemos sobre vetores, vamos examinar com cuidado os tipos de dados que podem ser armazenados em vetores:
*doubles*, *integers*, *characters*, *logicals*, *complex*, e *raw*. Neste tutorial, vamos examinar os 3 mais comumente usados na análise de dados: *doubles*, *characters*, *logicals*.

### Doubles

*doubles* são utilizados para guardar números. Por exemplo, o vetor *n_caracteres*, que indica o número de caracteres em cada texto, é do tipo *double*. Vamos repetir o comando que cria este vetor:

```{r}
n_caracteres <- c(5873, 6301, 358, 3644, 4086, 3454)
```

Com a função *typeof* você consegue descobrir o tipo de cada vetor:

```{r}
typeof(n_caracteres)
```

É possível fazer operações com vetores númericos (_integers_ também são vetores numéricos, mas vamos esquecer deles por um segundo e fazer *double* sinônimo de numérico). Por exemplo, podemos somar 1 a todos os seus elementos, dobrar o valor de cada elemento ou somar todos:

```{r}
n_caracteres + 1
n_caracteres * 2
sum(n_caracteres)
```

Note que as duas primeiras funções retornam vetores de tamanho igual ao original, enquanto a aplicação da função *sum* a um vetor retorna apenas um número, ou melhor, um *vetor atômico* (que contém um único elemento). Tal como aplicamos as operações matemáticas e a função *sum*, podemos aplicar diversas outras operações matemáticas e funções que descrevem de forma sintética os dados (média, desvio padrão, etc) a vetores numéricos. Veremos operações e funções com calma num futuro breve.

### Logicals

O vetor *politica* também são do tipo *double*. Mesmo registrando apenas a presença e ausência de uma característica, os valores inseridos são números. Mas e o vetor *video*? Vejamos:

```{r}
typeof(politica)
typeof(video)
```

Em vez de armazenarmos a sim/não, presença/ausência, etc com os números 0 e 1, podemos em R usar o tipo *logical*, cujos valores são TRUE e FALSE (ou T e F maiúsculos, para poupar tempo e dígitos). Diversas operações e resultados no R usam vetores lógicos (por exemplo, "filtrar uma base dados") e os utilizaremos com frequência.

O que acontece se fizermos operações matemáticas com vetores lógicos?

```{r}
video + 1
sum(video)
```
Automaticamente, o R transforma FALSE em 0 e TRUE em 1. Por exemplo, ao somar 2 FALSE e 2 TRUE obtemos o valor 2, que é o total de notícias que contêm vídeos.

### Character

Finalmente, vetores que contêm texto são do tipo *character*. O nome dos jornais, data, título e autor são informações que foram inseridas como texto (lembre-se, o R não sabe por enquanto que no vetor *data_noticia* há uma data). Operações matemáticas não valem pare estes vetores.

## Tipo e classe

Veremos em momento oportuno que os objetos em R também tem um atributo denominado *class*. A classe diz respeito às características dos objetos enquanto tipo diz respeito ao tipo de dado armazenado. No futuro ignoraremos o tipo e daremos mais atenção à classe, mas é sempre bom saber distinguir os tipos de dados que podemos inserir na memória do computador usando R.

## Abrindo dados de fontes externas

R é uma linguagem extremamente flexível quanto ao formato de dados que podem ser importados. Comumente, utilizaremos dados provenientes de arquivos de texto (.txt, .csv, .tab, etc) ou de arquivos de excel transformados em arquivos de texto. Mas estas estão longe de serem as únicas possibilidades.

Com o pacote *foreign* abriremos arquivos produzidos por outros softwares -- como Stata e SPSS. Podemos também conectar o R a sistemas de gerenciamento de bancos de dados relacionais (SGBD), como o SQL Server. Finalmente, podemos receber via web arquivos em formato XML ou Json e transformá-los em *data frames* em poucos passos.

Vamos começar com o exemplo simples. Baixe os dados com resultados eleitorais (votação nominal por município e zona) do Acre em 2014 disponíveis [aqui](https://raw.githubusercontent.com/leobarone/FLS6397/master/data/votacao_candidato_munzona_2014_AC.txt) (clique em "Save link as" e faça o download ou vá [aqui](https://github.com/leobarone/FLS6397/blob/master/data/votacao_candidato_munzona_2014_AC.txt) e clique em download). Estes dados são provenientes do [Repositório de Dados Eleitorais](http://www.tse.jus.br/eleicoes/estatisticas/repositorio-de-dados-eleitorais) do TSE. O Tribunal disponibiliza para cada ano eleitoral arquivos por UF em formato de texto (.txt) e numa pasta comprimida em formato .zip.

Obs: se você tiver algum problema com o download, encoding, ou outro, pode ir direto ao Repositório e fazer o download de lá.

Diferentemente de outros softwares, no RStudio a "interface Windows", ou "menus e janelas", não oferece a possibilidade executar funções. Porém, na aba *Global Environment* há um botão muito útil chamado "Import Dataset", que serve para importar para o *workspace* arquivos em qualquer formato de texto ("From CSV"), MS Excel, Stata, SAS e SPSS.

Use o botão e a janela para chegar ao arquivo e abrí-lo.

Observe duas mudanças na sua tela. Agora você encontrará o objeto *votacao\_candidato\_munzona\_2014\_AC* na aba *Global Environment* e o comando de importação dos dados no console, seguido do comando _View_ que provocou a abertura dos dados no RStudio.

Um detalhe importante: agora temos dois _data frames_ (_dados_ e _votacao\_candidato\_munzona\_2014\_AC_ ) em nosso _workspace_ convivendo com os vetores que criamos. Veja:

```{r}
ls()
```

Em outros softwares, como Stata e SPSS, costumamos trabalhar com um único objeto, que é o banco de dados. Apenas um *data_frame* por vez vai para a memória do software. No R, entretanto, temos um "espaço" no qual vários objetos coexistem.

Veja que não precisamos usar o botão de importação do RStudio para abrir o arquivo de dados. Podemos escrever a função *read.table* e todos os seus argumentos. Repetindo o comando:

```{r}
# OBS 1: insira o endereco do arquivo no computador que estiver 
# utilizando no lugar do endereco de arquivo abaixo
#
# OBS 2: Windows e R tem um problema serio relacionados as barras nos enderecos.
# Tente usar \\ (duas barras invertidas) no lugar de uma barra
# Alternativamente, copie o endereco usado pelo RStudio via botao de importacao
votacao_candidato_munzona_2014_AC <- read.table(
  "~/FLS6397/data/votacao_candidato_munzona_2014_AC.txt", 
  sep = ";", fileEncoding="latin1")

# Uma opção também é abrir o arquivo diretamente do repositório sem a necessidade 
# de download

votacao_candidato_munzona_2014_AC <- read.table(
  "https://raw.githubusercontent.com/thiagomeireles/cebrap_afro_2021/main/data/votacao_candidato_munzona_2014_AC.txt",
  sep = ";",
  fileEncoding="latin1")
```

No curso, vamos evitar usar o botão. Ao sair do curso, use-o sem problemas. Mas, como estamos em treinamento, precisamos dominar bem a função *read.table* e seus argumentos.

Os mais importantes são os três primeiros, *file*, *header* e *sep*. O primeiro deles, *file* é obviamente o endereço do arquivo de dados. *header*, que recebe valores lógicos, indica se a primeira linha contém os nomes das variáveis (T) ou é a primeira linha dos dados (F). *sep* indica qual é o símbolo utilizado para separar as colunas em uma linha (por exemplo, o TSE costuma usar ponto-e-vígula -- além deste, vírgula e tab são separadores bastante encontrados em arquivos de texto).

Alguns outros argumentos da função *read.table* a se notar: *dec* indica qual é o separador de decimais (ponto ou vírgula); *quote*, se os dados em cada "célula" estão contidos entre aspas; *stringsAsFactors* indica se textos devem ser vistos como textos ou variáveis categóricas (*factors*, que é uma classe de objetos que vimos no tutorial seguinte); e *fileEncoding*, que indica qual *encoding*, ou seja, qual forma de codificar bites em texto e números, foi usado ao salvar o arquivo (este é argumento particularmente importante para evitar bagunça de caracteres quando trocamos arquivos entre diferentes sistemas operacionais -- IOS, Linux e Windows).

## Breve exerício:

1- Quantas linhas e quantas colunas têm o *data_frame* _votacao\_candidato\_munzona\_2014\_AC_? Use as funções que você aprendeu no tutorial.

2- Observe as 4 primeiras linhas do *data_frame* com o comando _head_ (só as 4 primeiras).

3- Use o comando _str_ para examinar o *data_frame*.

## Cadê os nomes das variáveis?

Nem sempre teremos os nomes das variáveis na primeira linha do arquivo (e por isso utilizamos _header_ = F). Quando isso ocorre o R preenche automaticamente os nomes com V1, V2, ..., Vn, onde n é o total de colunas. Veja:

```{r}
names(votacao_candidato_munzona_2014_AC)
```

Onde obter essa informação? No caso do TSE, no arquivo LEIAME. Em geral, precisamos de um arquivo que acompanhe o arquivo de dados para compreender a base de dados. Ou teremos que advinhar.

Nos próximos tutoriais aprenderemos a transformar o *data_frame* e alterar os nomes das variáveis.

## Colunas como vetores

Uma vez que importamos os dados, podemos trabalhar com as colunas do *data_frame* como se fossem vetores. Por exemplo, vamos tirar a média (com a função *mean*) da coluna V29, que é a coluna "votos" -- no caso, de cada candidato em cada zona eleitoral. Veja:

```{r, error=TRUE}
mean(V29)
```

Opa! O vetor V29 não foi encontrado! Isso ocorre porque V29 não é um objeto no nosso *workspace*. Como podemos ter mais de um *data_frame*, se houvesse mais de um *data_frame* com a variável com nome V29 haveria confusão. O vetor com o qual queremos trabalhar está dentro do objeto _votacao\_candidato\_munzona\_2014\_AC_. Indicamos sua localização colocando primeiro o nome do *data_frame*, seguido do símbolo $ e depois do nome da coluna:

```{r, error=TRUE}
mean(votacao_candidato_munzona_2014_AC$V29)
```

## Paramos por aqui

Fizemos neste tutorial uma rápida introdução a vetores e *data_frames*. Vamos agora dar inúmeros passos para trás e aprender os fundamentos da linguagem R. De um forma ou de outra, tenha certeza de que você já começou a se acostumar com a linguagem, sua sintaxe, léxico e peculiaridades.
