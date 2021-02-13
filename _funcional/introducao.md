---
layout: single
title:  "Introdução à Programação Funcional"
toc: true
tags: [haskell]
---

Neste post vamos iniciar o estudo do paradigma funcional. Como ferramenta, usaremos a linguagem de programação [Haskell](https://www.haskell.org/).


## O paradigma funcional

Como o próprio nome diz, a programação funcional gira em torno de funções. Mas o que vem a ser funções? Embora estejamos acostumados com o termo 'função' das linguagens imperativas e orientadas a objeto (nesse último costumamos usar o termo *método*, mas chamar de função não está de todo errado), no paradigma funcional elas tem um significado mais próximo do seu significado original, as funções matemáticas. O paradigma funcional traz a programação um pouco mais para perto do formalismo matemático e a maior parte das implicações que isso traz consigo.

Em outras palavras, a *programação funcional* é um *estilo* de programação que consiste na aplicação de funções a argumentos, produzindo assim resultados. Uma linguagem de programação funcional é então aquela linguagem que suporta e encoraja o estilo funcional.

Dê uma olhadinha aqui, depois volte -> [link para o youtube](https://www.youtube.com/watch?v=LnX3B9oaKzw)

[![O paradigma funcional](https://img.youtube.com/vi/LnX3B9oaKzw/0.jpg)](https://www.youtube.com/watch?v=LnX3B9oaKzw)

Como um exemplo, vamos considerar que precisemos computar a soma de inteiros entre 1 e outro número *n*. Na maioria dos casos desenvolveríamos um algoritmo iterativo, provavelmente utilizando um laço de repetição. Declararíamos uma variável para acumular o valor atual da soma e a cada iteração adicionaríamos mais um número ao total. O laço executaria até que uma variável de controle alcançasse o número *n* desejado. Ao final teríamos nosso resultado. Segue abaixo uma possibilidade de implementação, utilizando a linguagem Java:

```python
def soma(n):
  total = 0
  for i in range(1, n + 1):
    total += i

  return total
```

Fomos um pouco desleixados no código acima, não estamos fazendo nenhuma verificação sobre os valores possíveis para *n*, mas ela vai funcionar na maioria dos casos. Na própria linguagem Python podem existir formas mais diretas de escrever esse algoritmo, mas isso não é importante no momento.

No código acima, temos a variável *total* sendo atualizada a cada rodada do laço, e sendo retornada após o cálculo. Se passarmos como parâmetro o valor 5, o laço deve rodar 5 vezes. Esse *idioma* ([o que é um idioma?](https://stackoverflow.com/questions/302459/what-is-a-programming-idiom)) é bastante comum em linguagens imperativas. Sabemos que Python é uma mistura de imperativo, com orientado a objetos, com funcional (é uma mistura danada...), mas vamos nos ater à natureza imperativa desse código especificamente.

Em Haskell poderíamos atingir esse objetivo utilizando duas funções, uma chamada [ .. ] e outra chamada *sum*. A primeira é usada para produzir uma lista a partir de um intervalo e a outra é capaz de somar os itens de uma lista. Dessa forma, o código em Haskell para resolver esse problema se resume a:

```haskell
soma n = sum [1 .. n]
```

Conseguiríamos chegar em algo parecido com isso Python, mas observe que em linguagens funcionais essa forma de pensar permeará todo o seu programa. Em outras palavras, programar em uma linguagem funcional é aprender a pensar de uma forma diferente. Linguagens pertencentes a este paradigma nos impõem algumas restrições, que alguns contextos são encarados como limitações, mas na realidade podem nos ajudar e nos forçam a pensar de maneira diferente.

Pare agora, assista a esse vídeo e depois volte -> [link para o youtube](https://www.youtube.com/watch?v=sqV3pL5x8PI)

[![Vídeo comparando paradigma funcional e imperativo](https://img.youtube.com/vi/sqV3pL5x8PI/0.jpg)](https://www.youtube.com/watch?v=sqV3pL5x8PI)

Bem vindo ao paradigma funcional. Espero que goste.

## A linguagem de programação Haskell

Haskell é uma linguagem de programação puramente funcional, nomeada em homenagem ao matemático Haskell Curry, famoso por seu trabalho na área de Lógica. Como curiosidade, Haskell Curry é o inventor do [*lambda calculus*](https://www.youtube.com/watch?v=eis11j_iGMs) (cuidado, esse vídeo pode deixar você sem sono).

Algumas características da linguagem Haskell:
+ **Programas concisos**: Devido ao alto nível de abstração, o que faz com que a linguagem faça muitas coisas por trás dos panos, programas em Haskell são muito mais compactos que programas que fazem a mesma coisa em linguagens de outros paradigmas.
+ **Podoroso sistems de tipos**: A maioria das linguagens modernas incluem algum *sistema de tipos* para detectar erros de compatibilidade de operações, por exemplo. O sistema de tipos do Haskell é muito robusto, e apesar de precisar de pouca informação por parte dos programadores, é capaz de detectar uma série de erros de incompatibilidade de tipos antes mesmo da execução do programa. O sistema de tipos de Haskell permite funções polimórficas e sobrecarga.
+ **Compreensão de listas**: Listas são largamente utilizadas em programação. Em Haskell as listas são incluídas como um conceito básico da linguagem, e adiciona uma notação simples mas poderosa para a construção de novas listas a partir da seleção e filtragem de elementos a partir de listas existentes. Em alguns casos o uso de compreensão de listas permite a definição concisa de funções sem a necessidade da definição de utilizar-se o mecanismo de recursão.
+ **Funções de alta ordem**: Haskell é uma linguagem na qual funções são consideradas cidadãs de primeira classe, ou seja, funções também são consideradas valor na linguagem. Isso permite que funções seja passadas como parâmetro, ou até retornadas por outras funções.
+ **Efeitos monádicos**: Embora as funções em Haskell sejam puras, o que não permite que funções tenham efeitos colaterais, muitos programas precisam produzir alguma forma de efeito colateral, na maioria das vezes relacionado a I/O. Haskell fornece o framework de mônadas para tratar desses efeitos colaterais sem comprometer a pureza das funções da linguagem.
+ **Avaliação preguiçosa**: Programas em Haskell são executados usando uma técnica chamada de "avaliação preguiçosa". Ela permite que a computação de um determinado trecho só será realizada quando realmente for necessária. Com isso é possível, por exemplo, ter funções que trabalhem com listas infinitas.


## Mais um pouco da linguagem Haskell

Relembrando um pouco a função *soma* que vimos anteriormente, ela também pode ser escrita na forma:

```haskell
soma [] = 0
soma (x:xs) = x + soma xs
```

A linha 1 informa que a soma de uma lista vazia é 0, e a segunda diz que a soma de uma lista não-vazia, composta pelo primeiro valor (`x`) e pelos demais valores (`xs`), é definida pela adição de `x` à soma de `xs`. O resultado da chamada `soma [1,2,3]` é dado por:

```
   soma [1,2,3]
=  1 + soma [2,3] -> aplicando soma
=  1 + (2 + soma[3]) -> aplicando soma
=  1 + (2 + (3 + soma[])) -> aplicando soma
=  1 + (2 + (3 + 0)) -> aplicando soma
=  6 -> aplicando +
```

A função `soma` é recursiva, e em sua definição temos o caso base e o passo recursivo. A cada aplicação da função soma reduzimos o tamanho do problema remanescente, até que que reste apeas a lista vazia, cuja soma é 0. Em Haskell, toda função tem um tipo, mesmo que não estejamos definindo explícitamente. No caso da nossa função soma, ela tem o tipo `Num a => [a] -> a`. Esta notação define que dado um valor `a` numérico (Haskell tem alguns tipos numéricos), a função `soma` é uma função que mapea uma lista de valores deste tipo em um único valor. São tipos numéricos em Haskell os inteiros e os pontos flutuantes, então nossa função consegue realizar seu cálculo com qualquer um deles.

Como dito anteriormente, tipos permitem que algumas verificações sejam realizadas antes da execução do código. Para cada aplicação função no programa, o compilador realiza uma checagem para garantir que os argumentos passados para as funções sejam compatíveis com seus tipos. Por exemplo, não seríamos capazes de aplicar a funço `soma` a uma lista de strings.


### Um exemplo mais interessante

Vamos definir a função `quicksort`, que realiza a ordenação de uma lista de inteiros. Ela pode ser definida da seguinte forma:

```haskell
quicksort [] = []
quicksort (x:xs) = quicksort menores ++ [x] ++ quicksort maiores
                   where
                      menores = [a | a <- xs, a <= x ]
                      maiores = [b | b <- xs, b > x ]
```

O operador `++` recebe duas listas e retorna concatenação das duas em uma terceira lista. A palavra chave `where` introduz definições locais, no caso uma lista chamada `menores` que vai conter os valores em `xs` menores ou iguais ao pivô (`x`) e uma outra lista chamada `maiores`, que conterá os valores em `xs` maiores que o pivô. Podemos ler a definição da lista `menores` como *uma lista contendo os valores a, tal que a, em xs, é menor ou igual a x*. A lista maiores tem uma leitura análoga. Ao fim da execução, a função `quicksort` terá produzido uma nova lista, a versão ordenada da lista inicial.

Interessantemente, o tipo da função `quicksort` é `Ord a => [a] -> [a]`, isto é, dado um valor `a`, ordenável, ela recebe uma lista destes valores e retorna uma lista deste mesmo tipo. Na linguagem Haskell, muitos tipos são ordenáveis, por exemplo, caracteres e strings. Desta forma, a nossa função `quicksort` é capaz de ordenar listas de qualquer tipo ordenável, o que inclui uma lista de catacteres ou de strings.
