# Tutorial 1: Manipulação de dados com o r *base*

## Manipulação de dados com a gramática básica do R

Depois de realizar os tutoriais pré-curso, você já possui diversas ferramentas para programar em R. A combinação de vetores, *data\_frames*, loops, condicionais e funções permitem a realização de diversas tarefas.

No entanto, ainda não vimos como manipular variáveis e observações em um *data\_frame*, algo essencial ao trabalhar com dados. Quando iniciarmos a coleta de dados atomatizada com *webscraping* ou acesso a APIs, precisaremos organizar nossos dados, não? Essa é uma ferramenta que lembra outros softwares utilizados para análise de dados como Stata, SPPS e SAS.

Nos tutoriais anteriores, falamos sobre diferentes "gramáticas" ou adaptações da linguagem, especialmente *dplyr* e *data.table*. Apesar da gramática do *data.table* também ser útil em muitos casos, focaremos na gramática básica e no *dplyr*, uma vez que hoje os objetos do tipo *DT* (data.table) lidam bem com essa possibilidade.

Apesar da "gramática" básica ser essencial no processo de aprendizagem de R, ela é pouco elegante e muitas vezes constitui uma barreira de aprendizado. Em muitos aspectos ela pode ser mais confusa e necessitar de muito mais código para realizar menos tarefas quando comparada a outros softwares de análise de dados ou ao *dplyr* e as demais ferramentas do *tidyverse*. Mas ainda precisamos como funciona o *base* da linguagem para não limitarmos nossa capacidade de aprendizado das outras gramáticas. Após isso, no próximo tutorial abordaremos como realizar as mesmas tarefas utilizando a *dplyr*.

## Variáveis e data frames

Como esta atividade é para fins didáticos, utilizaremos um banco de dados falso utilizado em outros cursos por necessitarmos, nesse momento, apenas do contato com as ferramentas de manipulação de dados. Quando iniciarmos o processo de coleta de dados, aplicaremos aos bancos que criarmos.

É um banco "fake" utilizado em outros cursos que lecionamos com o Leonardo Barone e está em inglês.

Ao utilizar o R, vimos que temos várias possibilidades e abertura de dados, como os pacotes *readr*, *data.table* e *haven*. Aqui utilizaremos a função *read\_delim* do pacote *readr*:

```{r}
library(readr)
url_fake_data <- "https://raw.githubusercontent.com/thiagomeireles/cebrap_afro_2021/main/data/fake_data.csv"
fake <- read_delim(url_fake_data, delim = ";", col_names = T)
```

A descrição das variáveis do banco de dados está abaixo:

"Fakeland is a very stable democracy that helds presidential elections every 4 years. We are going to work with the fake dataset of Fakeland individual citizens that contains information about their basic fake characteristics and fake political opinions/positions. The variables that our fake dataset are:

- _age_: age
- _sex_: sex
- _educ_: educational level
- _income_: montly income measured in fake money (FM\$)
- _savings_: total fake money (FM\$) in savings account
- _marrige_: marriage status (yes = married)
- _kids_: number of children
- _party_: party affiliation
- _turnout_: intention to vote in the next election
- _vote\_history_: numbers of presidential elections that turned out since 2002 elections
- _economy_: opinion about the national economy performance
- _incumbent_: opinion about the incumbent president performance
- _candidate_: candidate of preference"

### Exercício

Vamos utilizar as funções apresentadas nos outros tutoriais (*head*, *dim*, *names*, *str*, etc.) para conhecer os dados. Quantas linhas e colunas possui nosso *data\_frame*? E quantas colunas? Qual a forma de armazenamento de dados (tipo de dados e classes dos vetores/colunas)?


## Data frame como conjunto de vetores

Já vimos como construir um *data\_frame* a partir de vetores de mesmo tamanho e "pareados", ou seja, com a posição da informação representando cada observação. Para trabalhar com um vetor dentro de um *data\_frame* utilizamos o símbolo "$" para separar o nome do *data\_frame* do nome da variável. Por exemplo, para imprimir as observações do vetor "age" no *data\_frame* "fake" escrevemos "fake\$age":

```{r}
print(fake$age)
```
Podemos ainda criar uma cópia como vetor que não seja variável do *data\_frame*:

```{r}
idade <- fake$age
print(idade)
```
Mas qual a necessidade de utilizar o "$" com o nome do *data\_frame* e não utilizarmos apenas a indicação de "age"? Por enquanto, trabalhamos com *Workspaces* bastante limpos, mas normalmente temos mais de um *data\_frame* e mais de um pode tr uma variável com nome "age". É como se indicássemos um endereço composto pelo *data\_frame* + nome da variável para evitar ambiguidades dentro do nosso espaço de trabalho. Isso pode parecer um pouco estranho para quem trabalha com SPPS, Stata ou SAS, mas faz todo sentido dentro do R.

Na sequência, vamos observar outros exemplos simples de como usar variáveis de um *data\_frame*, algumas utilizaremos no futuro e você pode utilizá-los para ambientação à linguagem.

Gráfico de distribuição de uma variável contínua:

```{r}
plot(density(fake$age), main = "Distribuição de Idade", xlab = "Idade", ylab = "Densidade")
```

Gráfico de dispersão de duas variáveis contínuas:

```{r}
plot(fake$age, fake$savings, main = "Idade x Poupança", xlab = "Idade", ylab = "Poupança")
```

Mesmo não conhecendo os argumentos das funções é intuitivo identificar que *main* é utilizado para atribuir o título, *xlab* e *ylab* os rótulos dos eixos do gráfico. E por que estão entre aspas? Porque são atributos do tipo character.

Tabela de uma variável categórica (contagem):

```{r}
table(fake$party)
```

Tabela de duas entradas para duas variávels categóricas (contagem):

```{r}
table(fake$party, fake$candidate)
```

Se você já trabalhou com outras ferramentas de análise de dados, a utilização do "endereço" completo da variável pode parecer um pouco irritante. Mas você vai se acostumar, acredite.

## Dimensões em um data frame

Como já conversamos, um *data\_frame* é um tipo específico de matriz e, por isso, tem duas dimensões: linha e coluna. Se quisermos selecionar elementos específicos de um *data\_frame*, utilizamos colchetes separados por uma vírgula e inserimos (1) a seleção de linhas antes da vírgula e (2) a seleção das colunas depois da vírgula, isto é, [linhas, colunas]. Vamos observar alguns exemplos de seleção de linhas: 

Quinta linha:

```{r}
fake[5, ]
```

Quinta e a oitava linhas:

```{r}
fake[c(5,8), ]
```

As linhas 4 a 10:

```{r}
fake[4:10,]
```

Agora alguns exemplos de colunas, começando pela segunda coluna:

```{r}
fake[, 2]
```

Note que o resultado é semelhante ao de:

```{r}
fake$sex
```

No entanto, no primeiro caso estamos produzindo um _data frame_ de uma única coluna, enquanto no segundo estamos produzinho um vetor. Exceto pela classe, são idênticos.

Segunda e sétima colunas:

```{r}
fake[, c(2,7)]
```

Três primeiras colunas:

```{r}
fake[, 1:3]
```

São ferramentas que vimos rapidamente nos outros tutoriais, mas agora aplicadas a um *data\_frame*.

### Exercício

Qual é a idade do 17o. indivíduo? Qual é o candidato de preferência do 25o. indivíduo?

## Seleção de colunas com nomes das variáveis

No nosso *data\_frame* "fake" as linhas não possuem nomes (sim, elas podem ter). No entanto, as colunas **sempre** têm. Como, via de regra, trabalhamos com um número muito maior de linhas do que de colunas, o nome das últimas costumam ser muito mais úteis. Isso nos oferece outra possibilidade para seleção das colunas, utilizando seus nomes no lugar das posições para selecioná-las:

```{r}
fake[, c("age", "income", "party")]
```

No entanto, diferente de ferramentas de seleção de outros softwares, ao utilizar os nomes o operador ":" não é válido, ele se aplica somente a sequências de números inteiros.

```{r, error = T}
fake[, "age":"sex"]
```

Em um exercício de aplicação dessas ferramentas, vamos pensar que nossos dados "fake" dizem respeito aos resultados eleitoriais do estado do Rio Grande do Sul nas eleições de 2020 retirados do Repositório de Dados Eleitorais do TSE (e faremos esse exercício depois!). Existem um número grande de colunas que não utilizaremos (como horário da extração dos dados) e, para "facilitar" o trabalho do nosso computador, liberamos memória ao trabalhar com um *data\_frame* menor depois de fazer uma seleção de colunas. Podemos tanto utilizar a posição ou os nomes das colunas para gerarmos um novo *data\_frame* (ou sobrescrevemos o atual). Vamos ver como ficaria em "fake":

```{r}
new_fake <- fake[, c("age", "income", "party", "candidate")]
```

E se quiséssemos todas as colunas exceto um número pequeno? Vamos utilizar a função *setdiff* para remover as colunas "turnout" e "vote_history" a partir de um vetor com todos os nomes de colunas (gerado a partir da função *names*) e um vetor com as colunas que queremos excluir. Vamos observar o resultado criando o *data\_frame* "new\_fake2":

```{r}
selecao_colunas <- setdiff(names(fake), c("turnout", "vote_history"))
print(selecao_colunas)
new_fake2 <- fake[,selecao_colunas]
```

## Selecionando linhas com o operadores relacionais

Fizemos acima uma seleção de colunas utilizando os nomes das colunas, mas isso nem sempre é tão simples. Grandes bancos de dados (que possuem muitas colunas), como Censos (populacional ou escolar) ou PNADCs, normalmente não possuem os nomes das colunas e em muitos casos o número de colunas ultrapassa as centenas. Isso pode parecer um pouco assustador, mas podemos trabalhar com *data\_frames* que contenham somente as variáveis que temos interesse e renomeá-las antes ou depois da seleção para tornar o trabalho mais simples - veremos daqui a pouco como fazer isso.

Mas, mais assustador do que a seleção de colunas, pode ser a seleção de linhas, muito mais numeros e em muitos casos ultrapassando as centenas de milhares ou mesmo alcançando os milhões. Aqui entram os operadores relacionais, eles são fundamentais nesse processo. Vamos entender isso com pequenos passos até chegar ao resultado final.

Vamos supor, por exemplo, que queremos selecionar apenas os indivíduos que pretendem votar na próxima eleição (variável "turnout"). É possível gerar um vetor lógico que represente essa seleção:

```{r}
fake$turnout == "Yes"
```

Vamos guardar esse vetor lógico em um objeto denominado "selecao\_linhas"

```{r}
selecao_linhas <- fake$turnout == "Yes"
print(selecao_linhas)
```

Podemos, agora, inserir esse vetor lógico na posição das linhas dentro dos colchetes para gerar um novo conjunto de dados que atenda à condição esperada, ou seja, da intenção de votar:

```{r}
fake_will_vote <- fake[selecao_linhas, ]
```

Basicamente, podemos fazer a seleção de linhas (ou de colunas) utilizando sua posição, seus nomes ou um vetor lógico do mesmo tamanho das linhas (ou colunas). Não precisamos seguir todos os passos acima. Vamos observar um exemplo de geração de um novo *data\_frame* com os indivíduos que se identificam como "Independent" explicando qual o vetor lógico que queremos no próprio processo de seleção:

```{r}
fake_independents <- fake[fake$party == "Independent", ]
```

E sim, é posível combinar condições com operadores lógicos ("ou", "e" e "não") para seleções mais complexas:

```{r}
fake_married_no_college_yong <- fake[fake$marriage == "Yes" & 
                                       fake$age <= 30 & 
                                       !(fake$educ == "College Degree or more"), ]
```

Vamos tentar traduzir para o português nossa seleção no último exemplo. Esse é sempre um exercício muito útil para entendermos o que fazemos.

Selecionamos (1) pessoas quasadas que (2) possuam até 30 anos e (3) não tenham curso superior. Como utilizamos o operador lógico "e", são condições que se "somam" e dependem uma das outras.

### Exercício

Produza um novo *data\_frame* com apenas 4 variáveis -- "age", "income", "economy" e "candidate" -- e que contenha apenas eleitores homens, ricos ("income" maior que FM\$ 3 mil, que é dinheiro pra caramba em Fakeland) e inclinados a votar no candidato "Trampi".

Quais as dimensões do novo *data\_frame*? Qual é a idade média dos eleitores no novo *data\_frame*? Qual é a soma da renda no novo *data\_frame*?

## Função subset

Ainda temos uma maneira alternativa de fazer a seleção de linhas utilizando a função *subset*. Vamos repetir o exemplo dos indivíduos que se identificam como "Independent" com a nova função:

```{r}
fake_independents <- subset(fake, party == "Independent")
```

Obtemos o mesmo resultado e você pode achar que é uma forma mais elegante de código. Veremos, ainda hoje, outra forma ainda mais simples utilizando o pacote *dplyr*.

## Criando uma nova coluna

A criação de colunas em *data\_frames* é algo trivial. Como exemplo, podemos criar uma coluna "vazia" somente com "missing values", respresentados no R por "NA"

```{r}
fake$vazia <- NA
```

Podemos criar uma coluna a partir de outra(s). Vamos criar duas novas colunas como exemplo. A primeira é "poupança", que é a coluna "savings" convertida para real (a cotação de um FM\$, o fake money, é de R\$ 17 ). A segunda é a coluna "savings\_year", que é a divisão de "savings" por todos os anos do indivíduo a partir dos 18:

```{r}
fake$poupanca <- fake$savings / 17
fake$savings_year <- fake$savings / (fake$age - 18)
```

É possível realizar qualquer operação com vetores que vimos nos tutoriais anteriores para criação de novas variáveis, desde que os vetores tenham sempre o mesmo tamanho. Isso não é um problema em um *data\_frame*.

Se quisermos simplesmente substituir o conteúdo de uma variável, e não gerar uma nova, o procedimento é o mesmo. A única diferença é que atribuiremos o resultado da operação entre vetores à variável existente. Vamos, como exemplo, transformar a variável "age" em medida de meses:

```{r}
fake$age <- fake$age  * 12
```

Nos próximos passados, vamos descobrir como substituir valores em uma variável e, depois, como recodificamos variáveis.

## Substituindo valores em um variável

Agora vamos aprender a substituir valores em uma variável iniciando com a "tradução" para o português da variável "party". Inicialmente vamos alterar cada categoria individualmente e sem nenhuma função que auxlilie a substituição de valores.

Vamos começar com uma tabela de contagem da variável "party":

```{r}
table(fake$party)
```

Agora, observe o resultado do código abaixo:

```{r}
fake$party[fake$party == "Independent"]
```

O que obtivemos é um subconjunto APENAS da variável "party" e não de todo *data\_frame*. Veja que não utilizamos a vírgula dentro dos colchetes. Se atribuíssemos algum valor para essa seleção, "Independentes" por exemplo, realizaríamos a substituição desses valores:

```{r}
fake$party[fake$party == "Independent"] <- "Independente"
```

Importante: a seleção do vetor (colchetes) está à esquerda do símbolo de atribuição.

Observe o resultado na tabela:

```{r}
table(fake$party)
```

### Exercício

Traduza para o português as demais categorias da variável "party".

## Substituição com o comando replace

Podemos utilizar a função *replace* para obter o mesmo resultado de substituição de valores em uma mesma variável. Vamos traduzir para o português a variável "sex":

```{r}
fake$sex <- replace(fake$sex, fake$sex == "Female", "Mulher")
fake$sex <- replace(fake$sex, fake$sex == "Male", "Homem")
table(fake$sex)
```

Mais elegante, não?

## Recodificando uma variável

Além de subtituir valores, podemos recodificar variáveis. Como exemplo, imaginemos que não interessa trabalhar a renda ("income") como variável contínua. Mas, a partir dela, podemos construir uma variável indicando quem é rico ("rich"). Vamos estabelecer que um indivíduo é "rich" se a renda for superior a FM\$ 3 mil e "not rich" caso seja até 3 mil. Operaremos da seguinte forma: (1) criaremos uma variável com "missing values"; (2) substituiremos o valor para os indivíduos ricos; (3) substituiremos os valores para os não-ricos. Perceba que a seleção da variável "rich" para a substituição se dá a partir da variável "income":

```{r}
fake$rich <- NA
fake$rich[fake$income > 3000] <- "rich"
fake$rich[fake$income <= 3000] <- "not rich"
table(fake$rich)
```

### Exercício

Utilize o que você aprendeu sobre transformações de variáveis neste tutorial e o sobre fatores ("factors") no tutorial 2 para transformar a variável "rich" em fatores.

### Exercício (mais um)

Crie a variável "kids2" que indica se o indivíduo tem algum filho (TRUE) ou nenhum (FALSE). Dica: essa é uma variável de texto, e não numérica.

## Recodificando uma variável contínua com a função cut

Uma opção para a recodificação de variáveis contínuas é a função *cut*. Vamos repetir o exemplo e criar a variável "rich2" usando a nova abordagem:

```{r}
fake$rich2 <- cut(fake$income, 
                  breaks = c(-Inf, 3000, Inf), 
                  labels = c("não rico", "rico"))
table(fake$rich2)
```

Algumas observações importantes:
- se a variável tiver duas categorias, precisamos de três "break points";
- "-Inf" e "Inf" são símbolos para menos e mais infinito, respectivamente;
- o R não inclui o primeiro "break point" na primeira categoria como padrão, para isso é preciso alterar o argumento "include.lowest" para "TRUE";
- também por padrão, os intervalos são fechados à direita e abertos à esquerda, ou seja, incluem o valor superior que delimita o intervalo, mas não o inferior. No exemplo, se uma pessoa ganha 3 mil FM\$ ela fica na primeira categoria. Isso muda se o argumento "rigth" for alterado para "FALSE".

### Exercício

Crie a variável "poupador", gerada a partir de savings\_year (que criamos anteriormente, antes de transformar "age" em meses), e que separa os indivíduos que poupam muito (mais de FM/$ 1000 por ano) dos que poupam pouco. Use a função _cut_.

## Recodificando uma variável contínua com a função recode

Assim como temos a função *cut* para variáveis contínuas, a função *recode* do pacote *dplyr* é utilizada para variáveis categóricas, sejam texto ou fatores. E seu uso é simples e intuitivo. vamos utilizar a variável "educ" como exemplo:

```{r}
library(dplyr)
fake$college <- recode(fake$educ, 
                       "No High School Degree" = "No College",
                       "High School Degree" = "No College",
                       "College Incomplete" = "No College",
                       "College Degree or more" = "College")
table(fake$college)
```

Podemos comparar as mudanças com uma tabela de 2 entradas:

```{r}
table(fake$college, fake$educ)
```

### Exercício

Crie a variável "economia", que os indivíduos que avaliam a economia (variável "economy") como "Good" ou melhor recebem o valor "positivo" e os demais recebem "negativo".

## Ordenar linhas e remover linhas duplicadas:

Agora vamos aprender a ordenar linhas em um banco de dados e a remover entradas duplicadas.

Com a função *order* podemos gerar um vetor que indica qual a posição que cada linha deve recever no ordenamento desejado. Vamos ordenar nosso "fake" pela renda.

```{r}
ordem <- order(fake$income)
print(ordem)
```
Ela indica a posição atual de cada linha no nosso *data\_frame* pela ordem que queremos.

Se aplicarmos um vetor numérico com um novo ordenaemnto à parte destinada às linhas no colchetes, receberemos o *data\_frame* ordenado:

```{r}
fake_ordenado <- fake[ordem, ]
head(fake_ordenado)
```

Podemos aplicar a função *order* diretamente dentro dos colchetes:


```{r}
fake_ordenado <- fake[order(fake$income), ]
```


Para encerrar, vamos duplicar de forma proposital parte dos nossos dados (as 10 primeiras linhas) usando o comando *rbind*, que "empilha" dois *data\_frames*:

```{r}
fake_duplicado <- rbind(fake, fake[1:10, ])
```

Vamos ordenar para ver algumas duplicidades:

```{r}
fake_duplicado[order(fake_duplicado$income), ]
```

E agora removemos as observações duplicadas com a função *duplicated*:

```{r}
fake_novo <- fake_duplicado[!duplicated(fake_duplicado),]
```

Note que precisamos da exclamação (operador lógico "não") para ficar com todas as linhas **não** duplicadas.

## Renomeando variáveis

Em breve veremos como renomear variáveis de uma maneira bem mais simples. Mas também devemos aprender o jeito trabalhoso de renomear um variável.

Podemos observar os nomes das variáveis de um *data\_frame* usando a função *names*:

```{r}
names(fake)
```

Os nomes das colunas são um vetor. Para renomear as variáveis, basta substituir o vetor de nomes por outro. Por exemplo, vamos manter todas as variáveis com o mesmo nome, exceto as três primeiras:

```{r}
names(fake) <- c("idade", "sexo", "educacao", "income", "savings", "marriage", "kids", "party", "turnout", "vote_history", "economy", "incumbent", "candidate", "vazia", "poupanca", "savings_year", "rich", "rich2", "college")
head(fake)
```

Você não precisa de um vetor com todos os nomes sempre que precisar alterar algum. Basta conhecer a posição da variável que quer alterar. Veja um exemplo com "marriage", que está na sexta posição:

```{r}
names(fake)[6] <- "casado"
```
Simples, mas veremos que não é nada produtivo.

## Listas no R

Falamos de dois dos principais tipos de objetos do R, os *vetores* e os *data\_frames*. Um terceiro tipo muito importante são as *listas*. Elas são coleções de elementos que não necessariamente são da mesma classe, ou seja, pode conter *vetores*, *data\_frames* ou outros tipos de objetos. São muito úteis para organização do que utilizamos em nosso trabalho, permitindo que deixemos esses objetos "guardados" para acessarmos somente quando precisarmos com uma extração. Para criar uma lista, bata utilizar o comando *list()*

```{r}
lista <- list()
```

Podemos ainda criar a lista já com objetos determinados:

```{r}
lista <- list(8, "banana", c(T, T, F, T, F), 1i, c(seq(8, 76, by = 2.5)))
print(lista)
```

Perceba que cada objeto da lista está identificado entre duplas chaves ("[[]]"). Para acessá-lo ou extraí-lo, basta utilizar a posição do objeto ou o seu nome.

```{r}
lista[["pi"]] <- 3.14

lista[["pi"]]
lista[[6]]
```
Quando utilizamos múltiplos *data\_frames* pode ser conveniente armazená-los em listas e somente chamá-los quando necessário. Para transformar o que está dentro de uma lista em *vetor* ou *data\_frame*, utilizamos os comandos *as.vector* e *as.data.frame*:

```{r}
vetor <- as.vector(lista[[3]])
df <- as.data.frame(lista[[3]])
```