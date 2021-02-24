# Tutorial 17: Análise de sentimentos em PT-BR com *quanteda* e *lexiconPT*

## Carregando os pacotes

Hoje vamos utilizar uma função para chamar nossos pacotes que passa por algumas etapas, vamos tentar entendê-las. Estamos construindo a função *inst_load* que acessar os `req_pack`, ou seja, os pacotes que queremos carregar. Em um primeiro momento, ela cria um novo vetor chamado `remain_pack` no caso de não encontrar o pacote entre os instalados. Caso ele crie esse vetor, com ele possuindo alguma linha, mandar inslar o pacote. Por fim, a última etapa acessa o pacote no vetor `req_pack` para carregá-lo. 

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

Com isso, podemos criar um vetor com todos os pacotes que queremos carregar, aqui entre aspas por se tratar de um vetor character

```{r}
packages_to_load <- c("quanteda", "dplyr", "readr", "stringr", "lexiconPT", "tibble", "ggplot2")
```

Tendo esse vetor, apenas pedimos para que a função *inst_load* que criamos o acesse. Você verá que, caso o pacote não esteja instalado, o R fará isso. Se estiverem todos instalados, o R apenas carregará os pacotes. Bem útil, não?

```{r}
inst_load(packages_to_load)
```

## Abrindo os dados

### Dados DOU

Aqui acessaremos mais uma vez os dados que capturamos no DOU com a função *read_csv* do pacote *readr*. Dentro dela já especificamos a leitura da variável de `data` como tal e em seguida usamos o *mutate* do *dplyr* para criar uma variável de ano a partir da variável de data.

```{r, message=FALSE, warning=FALSE}
dados_dou <- read_csv("https://raw.githubusercontent.com/thiagomeireles/cebrap_afro_2021/main/data/dados_dou.csv",
                      col_types = cols(data = col_date(format = "%d/%m/%Y"))) %>% 
  mutate(ano = format(data, "%Y"))
```

Em seguida vamos criar nosso corpus da mesma forma que o tutorial passado, atribuindo as informações (metadados) com a função *docvars*.

```{r, message=FALSE, warning=FALSE}
corpus_dou <- corpus(dados_dou$texto)

docvars(corpus_dou, "titulo") <- dados_dou$titulo
docvars(corpus_dou, "data")   <- dados_dou$data
docvars(corpus_dou, "edicao") <- dados_dou$edicao
docvars(corpus_dou, "ano")    <- dados_dou$ano

corpus_dou_2021 <- corpus_subset(corpus_dou, ano == 2021)
```

### Dicionário de sentimentos em Português

Vamos utilizar o pacote *lexiconPT* para acessar a versão 3.0 da OpLexicon, um dicionário de sentimento para português desenvolvido por pesquisadores da PUCRS. Ela é uma atualização da versão 2.1 para a polaridade de alguns adjetivos. Se quiserem conhecer mais o projeto, acessem esse [link](https://www.inf.pucrs.br/linatural/wordpress/recursos-e-ferramentas/oplexicon/). O pacote também conta com o dicionário [SentiLex](http://b2find.eudat.eu/dataset/b6bd16c2-a8ab-598f-be41-1e7aeecd60d3) que não vamos utilizar no tutorial.

```{r}
lexicon <- oplexicon_v3.0

head(lexicon)
```

Vemos que carregamos um *data.frame* com as informações do dicionário, mas precisamos criar um dicionário para utilizarmos no *quanteda* utilizando a variável "polarity" que possui três valores: positivo (1), neutro (0) e negativo (-1). São as três categorias que criaremos no dicionário, vamos lá:

```{r}
dicionario_lex <- dictionary(list(positivo = lexicon$term[lexicon$polarity==1],
                                  negativo = lexicon$term[lexicon$polarity==-1],
                                  neutro   = lexicon$term[lexicon$polarity==0]))
```

## Iniciando a análise


Antes de criar nossa Matriz Documento-Recurso, vamos recriar nosso vetor de *stopwords*:

```{r}
stopwords_pt <- c(stopwords("pt"), "nº", "n", "º",  "conceição", "dias", "cnpj", "josé", "santa", "art", "processo", "portaria", "+", "r", "$", "1°", "0001-04","1524422177k660001", "ug", "°", "diretor")
```

Para criar nossa matriz, vamos gerar um objeto tokenizado anterior. Em sequência criamos nossa matriz indicando nosso dicionário e agrupando por data utilizando o *pipe*.

```{r}
tokens_publicacoes <- tokens(corpus_dou_2021,
                             stem = FALSE, 
                             remove_punct = TRUE,
                             remove_numbers = TRUE)

dfm_publicacoes <- dfm(tokens_publicacoes,
                       remove = stopwords_pt,
                       dictionary = dicionario_lex) %>% 
  dfm_group("data")
```

Agora que temos nosso dicionário aplicado à nossa Matriz Documento-Recurso podemos produzir nossos gráficos com as análises do sentimento dos textos que analisamos.

Nesse primeiro, criaremos um gráfico de linhas que vê a frequência de termos com sentimentos positivos, neutros e negativos por data. As opções de edição são específicas da função [*matplot*](https://www.rdocumentation.org/packages/graphics/versions/3.6.2/topics/matplot), aqui estamos especificando o tipo como linha, utilizando as duas primeiras colunas e especificando os rótulos dos eixos. Chamamos o gráfico com a função *grid* e inserimos a legenda no fim do processo.

```{r}
# Copie e cole as linhas do gráfico no console ou rode todo o chunk de uma vez
matplot(x = dfm_publicacoes$data, y = dfm_publicacoes, type = "l", lty = 1, col = 1:2,
        ylab = "Frequência", xlab = "") 
grid()
legend("topleft", col = 1:2, legend = colnames(dfm_publicacoes), lty = 1, bg = "white")
```

Faremos algo similar nesse segundo gráfico, mas com o "saldo" do sentimento para cada data, ou seja, a diferença entre a pontuação do positivo com o negativo. No comando *abline* criamos uma linha marcando o valor 0 no eixo y.

```{r}
# Copie e cole as linhas do gráfico no console ou rode todo o chunk de uma vez
plot(dfm_publicacoes$data, dfm_publicacoes[,"positivo"] - dfm_publicacoes[,"negativo"], 
     type = "l", ylab = "Sentimento", xlab = "")
grid()
abline(h = 0, lty = 2)
```
Para o terceiro gráfico, criaremos um objeto com os resultados do estimador Kernel Nadaraya-Watson (método de regressão não paramétrica).

```{r}
dat_smooth <- ksmooth(x = dfm_publicacoes$data, 
                      y = dfm_publicacoes[,"positivo"] - dfm_publicacoes[,"negativo"],
                      kernel = "normal", bandwidth = 30)
```

E em seguida faremos o gráfico com a linha ao longo das datas usando procedimentos semelhantes aos anteriores:

```{r}
# Copie e cole as linhas do gráfico no console ou rode todo o chunk de uma vez
plot(dat_smooth$x, dat_smooth$y, type = "l", ylab = "Sentimento", xlab = "")
grid()
abline(h = 0, lty = 2)
```

Migrando para as alternativas do *ggplot2*, vamos preparar um documento para identificar o efeito que os termos mais recorrentes possuem. Primeiro vamos criar uma nova matriz para a análise de sentimento, mas sem identificar os valores do dicionário:

```{r}
dfm_sentimento <- dfm(tokens_publicacoes,
                      remove = stopwords_pt,
                      stem = FALSE, 
                      remove_punct = TRUE,
                      remove_numbers = TRUE)
```

Em seguida, vamos observar um número razoável dos termos mais recorrentes, digamos 500. Em seguida, vamos converter o resultado em um *data.frame* com a função *as.data.frame*, renomear a varíavel que indica o número de aparições com a função *rename* e transformar o nome das linhas (termos) em uma variável com a função *rownames_to_column*. No próximo passo, faremos um *join* com o dicionário, incluindo todas as informações do `lexicon`, em seguida remover os termos que não são analisados pelo dicionário com a função *na.omit* (ela remove todas as linhas com missing values). No passo seguinte, utilizamos a função *mutate* para criar novas variáveis, o `saldo` (produto do sentido da polaridade pelo número de ocorrências) e uma variável `polaridade` que indica se é neutra, positiva ou negativa. Por fim, vamos selecionar apenas os termos que possuem polaridade positiva ou negativa com a função *filter* e, em seguida, apenas o top 30 de ocorrências com a função *slice_max*:

```{r}
top_feat <- topfeatures(dfm_sentimento, 500) %>% 
  as.data.frame() %>% 
  rename(., n = `.`) %>% 
  rownames_to_column(var = "term") %>% 
  left_join(lexicon, by = "term") %>% 
  na.omit() %>% 
  mutate(saldo = polarity*n,
         polaridade = NA,
         polaridade = ifelse(polarity==0, "neutra", polaridade),
         polaridade = ifelse(polarity==-1, "negativa", polaridade),
         polaridade = ifelse(polarity==1, "positiva", polaridade)) %>% 
  filter(polaridade!="neutra") %>% 
  slice_max(n, n = 30) 
```

Agora produziremos dois gráficos a partir desse *data.frame*. No primeiro veremos a "contribuição para o sentimento" vendo o número de ocorrências de cada um dos 30 termos. Ao definir a aesthetics, no eixo x teremos os termos ordenados de forma crescente; no eixo y o número de ocorrências e com *fill* diferenciaremos os positivos dos negativos. O que temos de novo é o *geom_bar*, função para produção de um gráfico de barras no ggplot expliticanto a `stat` como `identity` para dizer que estamos fornecendo os valores do eixo y. Além disos, no *theme* estamos apenas "girando" o label para a posição vertical.

```{r}
top_feat %>% 
  ggplot(aes(reorder(term, n), n, fill = polaridade)) + 
  geom_bar(stat = "identity") +
  theme_bw() +
  theme(axis.text.x = element_text(angle = 90, hjust = 1)) +
  ylab("Contribuição para o sentimento") +
  xlab("Termos")
```

Por fim, faremos um gráfico que mostra o saldo de cada um desses termos, também os ordenando no eixo x de forma crescente. No eixo y teremos o saldo e preencheremos com *fill* para diferenciar os positivos e negativos. 

```{r}
top_feat %>% 
  filter(saldo!=0) %>% 
  ggplot(aes(reorder(term, saldo), saldo, fill = polaridade)) + 
  geom_bar(stat = "identity") +
  theme_bw() +
  theme(axis.text.x = element_text(angle = 90, hjust = 1)) +
  ylab("Saldos para o sentimento") +
  xlab("Termos")
```

Bastante coisa bacana, não? Grande parte desses resultados parece mais descritivo, mas o trabalho pela análise de sentimentos é bastante interessante e traz dimensões interessantes para a análise de notícias e tweets, por exemplo. Por 