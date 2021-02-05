# [Cebrap.lab Afro] Curso de manipulação, raspagem e mineração de dados em R

## Informações básicas

### Instrutor: 
	
[Thiago Meireles](https://thiagomeireles.github.io/)

### Data, Hora e Local

De 01 a 19 de fevereiro, das 9 às 12 horas das segundas, quartas e sextas-feira.

Os encontros serão via Zoom e utilizarão material disponível no repositório do curso..

## Apresentação

O laboratório apresenta as principais ferramentas de captura de dados na Internet e análise quantitativa de texto utilizando R. Além de ser um software livre voltado para estatística computacional e análise de dados, R é uma linguagem focada na aplicação de funções que, entre outras possibilidades, permite a captura de dados de forma automatizada na internet. A partir de informações disponíveis em portais de notícias, apresentaremos esse processo de raspagem de dados de páginas web (especialmente de tabelas e de páginas construídas em html) e construção de bases de dados com textos de Internet tratados como informações quantitativas, o que permitirá introduzir algumas das práticas de mineração de texto. Faremos um exercício empírico partindo de uma questão de pesquisa que conduzirá a experimentação, de forma a capacitar os participantes com ferramentas e procedimentos que depois poderão ser usadas para a construção de suas próprias bases de dados. 

Antes, porém, de entrarmos no mundo da manipulação e análise de texto, passaremos por aspectos básicos da linguagem e as formas de manipulação de seus objetos.

Esse repositório será alimentado ao longo do curso com roteiros de aula e tutoriais atualizados tentado atender as particularidades da turma.

### Dinâmica dos encontros

As aulas terão conteúdo expositivo sobre conceitos e ferramentas básicas utilizados durante o curso, mas a maior parte do tempo será dedicada à realização de tutoriais assistidos. Trabalharemos em dupla, cada um em seu computador. O professor acompanhará o andamento de cada dupla, tirando as dúvidas (sim, elas surgirão o tempo todo).

## Requisitos

### Preparação
A participação no curso requer uma exposição prévia à linguagem R e ao ambiente de tabalho do RStudio.

Caso não possua contato com a linguagem ou não a utilize há algum tempo, a realização dos tutoriais pré-curso são essenciais.

Ainda que tenha conhecimento básico das estruturas da linguagem, é fortemente recomendado que tambem o façam.

O tempo estimado para esses tutoriais é de *aproximadamente 4 horas*.

### Equipamento

Como a maior parte do curso é baseada em tutoriais em que vocês aprenderão "colocando a mão na massa", é necessário acesso a um computador com internet durante as aulas.

### Softwares

Foram preparados dois roteiros para antes do curso, um primeiro para [Instalação do R](https://github.com/thiagomeireles/cebrap_afro_2021/blob/main/roteiros/00_instalacao.md) com as instruções para todos os softwares necessários.

### Pré-Curso

Além do roteiro para instalação, é importante a realização do [Roteiro do Básico em R](https://github.com/thiagomeireles/cebrap_afro_2021/blob/main/roteiros/01_basico.md). Nele serão apresentadas algumas funções e conceitos básicos utilizados em aula.

## Objetivos

Os participantes, ao fim do curso, serão capazes de:
- Manipular bases de dados utilizando as abordagens *base* e *dplyr*;
- Coletar dados de sites de estrutura mais simples, como jornais;
- Coletar dados do Diário Oficial da União;
- Coletar dados do *Twitter* via API;
- Realizar tarefas relacionadas a mineração de texto a partir de diferentes abordagens;
- Produzir gráficos e grafos mais simples a partir dos dados coletados;
- Entender e aplicar conceitos básicos de *text mining*.

## Roteiros e tutoriais

### Roteiros

Todas os dias de curso terão roteiros a cumprir. Pouco antes de cada encontro, as linhas abaixo serão preenchidas com links com as descrições do que esperamos em cada dia de curso e como o faremos. Serão indicadas leituras para melhor compreensão do conteúdo, mas em boa parte do curso elas são cobertas de forma satisfatória pelos tutoriais.

[01/02/2021](https://github.com/thiagomeireles/cebrap_afro_2021/blob/main/roteiros/dia_01.md) - Manipulação de dados em R: base e *dplyr*

[03/02/2021](https://github.com/thiagomeireles/cebrap_afro_2021/blob/main/roteiros/dia_02.md) - O básico da raspagem de dados: Introdução à raspagem em *html* utilizando um portal de notícias

[05/02/2021](https://github.com/thiagomeireles/cebrap_afro_2021/blob/main/roteiros/dia_03.md) - Desafios de raspagem de dados: Automatização da raspagem em *html* e novos desafios

[08/02/2021] - Raspagem por termos: Diário Oficial da União

[10/02/2021] - Twitter: utilizando o *rtweet* para acessar a API

[12/02/2021] - Introdução à manipulação de textos como dados: pacotes *string*, *tm* e *tidytext*

[19/02/2021] - Introdução à análise quantitativa de texto: conhecendo o pacote *quanteda*

[22/02/2021] - Modelagem básica: apresentação de métodos de análise quantitativa de texto


### Tutoriais

Os links para os tutoriais serão disponibilizados abaixo antes de cada aula.

[Tutorial 1](https://github.com/thiagomeireles/cebrap_afro_2021/blob/main/tutoriais/Tutorial_1.md) - Manipulação de dados com o R *base*

[Tutorial 2](https://github.com/thiagomeireles/cebrap_afro_2021/blob/main/tutoriais/Tutorial_2.md) - Manipulação de dados em R com o *dplyr*

[Tutorial 3](https://github.com/thiagomeireles/cebrap_afro_2021/blob/main/tutoriais/Tutorial_3.md) - Bases de dados relacionais

[Tutorial 4](https://github.com/thiagomeireles/cebrap_afro_2021/blob/main/tutoriais/Tutorial_4.md) - Introdução ao rvest e revisão

[Tutorial 5](https://github.com/thiagomeireles/cebrap_afro_2021/blob/main/tutoriais/Tutorial_5.md) - html com o sistema de busca da Folha de São Paulo

[Tutorial 6](https://github.com/thiagomeireles/cebrap_afro_2021/blob/main/tutoriais/Tutorial_6.md) - Automatizando a raspagem na Folha de São Paulo

[Tutorial 7](https://github.com/thiagomeireles/cebrap_afro_2021/blob/main/tutoriais/Tutorial_7.md) - Raspando notícias da Folha

[Tutorial 8](https://github.com/thiagomeireles/cebrap_afro_2021/blob/main/tutoriais/Tutorial_8.md) - Coleta de dados no site do Diário Oficial da União, primeira tentativa

## Referências

*Incompletas, algumas ainda a incluir*

- Grolemund, Garrett (2014). Hands-On Programming with R. Ed: O'Reilly Media. Disponível gratuitamente [aqui](https://rstudio-education.github.io/hopr/)
- Wichkam, Hadley e Grolemund, Garrett (2016). R for Data Science. Ed: O'Reilly Media. Disponível gratuitamente [aqui](http://r4ds.had.co.nz/data-visualisation.html)
- Wichkam, Hadley (2014). Advanced R. Ed: Chapman and Hall/CRC. Disponível gratuitamente [aqui](http://adv-r.had.co.nz/)
- Gillespie, Colin e Lovelace, Robin (2016). Efficient R programming. Ed: O'Reilly Media. Disponível gratuitamente [aqui](https://csgillespie.github.io/efficientR/)

