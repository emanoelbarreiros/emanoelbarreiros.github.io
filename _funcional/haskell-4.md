---
layout: single
title:  "[5] Definição de funções"
toc: true
lesson: 5
---

Nesta postagem vamos aprender sobre as diferentes formas para definição de funções. Uma coisa para termos em mente antes de vermos os detalhes sobre definições de funções, é que é muito mais produtivo construirmos novas funções a partir de funções pré-existentes. Essas funções pré-existentes podem ter sido desenvolvidas por nós ou estar disponíveis nas biliotecas.

O Haskell Language Report é a fonte oficial de informações sobre a linguagem: [https://www.haskell.org/onlinereport/haskell2010/]([https://www.haskell.org/onlinereport/haskell2010/)

As biliotecas podem ser encontradas aqui: [https://www.haskell.org/onlinereport/haskell2010/haskellpa2.html#x20-192000II](https://www.haskell.org/onlinereport/haskell2010/haskellpa2.html#x20-192000II)


## Expressões diretas

A forma mais simples de definir uma função é fazê-lo a partir de uma expressão simples, que resulta em um valor que será retornado para o código que invocou a função. Exemplo:

```haskell 
dobro :: Int -> Int
dobro x = 2 * x
```

O nome da função é o primeiro nome a ser colocado na linha, seguido pelos parâmetros, todos separados por espaços. Lembrar das [regras de nomes](https://emanoelbarreiros.github.io/funcional/haskell-1#regras-para-nomes).

Depois temos um sinal e igualdade, e logo depois o código da função. Esta função pode agora ser invocada em outros pontos do programa.


## Expressões condicionais

Outra forma é utilizar condicionais, que permitem que sejam definidos caminhos alternativos para a execução do programa, a partir da avaliação de uma condição. Veja abaixo o exemplo de uma possível implementação da função `signum`, que nos retorna o sinal de um número:

```haskell
signum :: Int -> Int
signum n = if n < 0 then -1 else
              if n == 0 then 0 else 1
```

**IMPORTANTE:** Diferente da maioria das linguagens, blocos condicionais em Haskell exigem sempre o `else`. 


## Equações com guardas

Uma alternativa à utilização de `if-else` é a definição de funções através de *guardas*, definidas usando-se o caractere `|`. Caso a função possua mais de uma guarda, as condições serão checadas uma a uma, de cima para baixo. A primeira expressão cuja condições for avaliada para verdadeiro será selecionada, e a expressão definida por ela será executada. 

**IMPORTANTE:** Todas as expressões definidas nas diversas guardas devem ter o mesmo tipo, para que o tipo coreto seja definido para a função.

Veja como a função `signum` poderia ser definida usando guardas:

```haskell
signum :: Int -> Int
signum n | n < 0 = 0
         | n == 0 = 0
         | otherwise = -1
```

Perceba que em cada guarda temos uma condição que deve ser satisfeita para que aquele trecho de código seja selecionado. Na primeira guarda, por exemplo, a funções retornará 0 caso a condição `n < 0` seja satisfeita. Na última guarda o termo `otherwise` funciona como um *caso contrário*, ou seja, caso nenhuma das condições anteriores seja satisfeita, a função selecionará a expressão marcada com o `otherwise`.


## Casamento de padrões

Em alguns casos, as funções possuem definições muito simples, que podem ser especificadas utilizando-se casamento de padrões. Semelhante à definição usando guardas, a função fará a comparação dos argumentos recebidos com os padrões especificados, um a um, de cima para baixo, e selecionará o primeiro padrão que casa corretamente com os valores recebidos. Veja abaixo o exemplo da função `not`, que inverte um valor lógico recebido como parâmetro:

```haskell
not :: Bool -> Bool
not True = False
not False = True
```

Abaixo mais um exemplo, agora com a função lógica *and*, definida através do operador `&&`:

```haskell
(&&) :: Bool -> Bool -> Bool
True && True = True
True && False = False
False && True = False
Fals && False = False
```

Caso a função receba mais de um parâmetro, como é o caso da função `&&`, os valores recebidos como parâmetro serão casados na mesma ordem em que os valores aparecem nas equações definidas nos padrões. É possível também definir a função da seguinte forma:

```haskell
(&&) :: Bool -> Bool -> Bool
False && _ = False
True && b = b
```

Na definição acima, o caractere `_` (underscore), significa que não nos interessa o que chega como segundo parâmetro, caso o primeiro seja `False`, o resultado deve ser `False`. No segundo padrão, caso o primeiro valor seja `True`, o resultado será igual ao valor do segundo parâmetro.


## Padrões com tuplas

A definição de funções a partir do recebimento de tuplas já define um padrão, que será casado com qualquer tupla com a mesma aridade e cujos componentes casem com os padrões especificados pela função. Tome como exemplo as funções `fst` e `snd`, que retornam o primeiro e segundo valores de um par, respectivamente:

```haskell
fst :: (a,b) -> a
fst (x,_) = x

snd :: (a,b) -> b
snd (_,y) = y
```


## Padrões com listas

É possível também definir uma lista de padrões, que será casada com o valor de lista passado como parâmetro. Veja o exemplo abaixo, que tenta fazer o casamento de uma lista de inteiros que tem exatamente 3 elementos. Para qualquer outra lista de inteiros o retorno será `False`:

```haskell
teste1 :: [Int] -> Bool
teste1 [_,_,_] = True
teste1 _ = False
```

É possível modificar esse padrão para aceitar apenas listas de inteiros que tenham como primeiro elemento o valor 1:


```haskell
teste2 :: [Int] -> Bool
teste2 [1,_,_] = True
teste2 _ = False
```

### Listas não são primitivos da linguagem

Ao contrário do que podemos pensar, listas não são tidas como um primitivo da linguagem. Na realidade, elas são construídas com a aplicação repetida do operador `:`, chamado de *cons*, que constrói a lista completa a partir de uma lista vazia.

A lista `[1, 2, 3]`, por exemplo, é construída por `1 : (2 : (3 : []))`. Para evitar o uso excessivo de parênteses, institui-se que o operador *cons* é associativo à direita, ou seja, a expressão `1 : 2 : 3 : []`.

Este mesmo operador pode ser usado para fazer o casamento de padrões de listas, utilizando a construção `(x:xs)`. Esta expressão vai casar com uma lista de qualquer tamanho, o `x` representará o item na cabeça da lista e `xs` a cauda da lista. Você pode usar quaisquer identificadores válidos para representar a cabeça e cauda da lista, mas é bastante comum ver `x` e `xs` sendo usados para este fim.

Podemos agora criar uma função que recebe uma lista de qualquer tamanho e que tenha o valor 1 como o primeiro elemento:

```haskell
teste3 :: [Int] -> Bool
teste3 (1:_) = True
teste3 _ = False
```


## Expressões lambda

Expresões lambda constituem-se como mais uma alternativa para a definição de funções. Até agora definimos funções a partir de equações, com um nome associado a elas. Diferentemente, ao utilizarmos expressões lambda para criar funções, na realidade estamos criando uma função sem nome, ou anônima. Por exemplo, uma função que recebe um parâmetro e retorna o valor recebido multiplicado por 10, pode ser escrita da seguinte forma:

```haskell
\x -> x * 10
```

A contrabarra que inicia a definição da função é apenas uma alusão a letra grega \\(\lambda\\) (lambda). O que segue é a especificação dos parâmetros. A seta `->` delimita o início do corpo da função. Além de poder ser usada em qualquer lugar e da mesma forma qualquer outra função, elas também são úteis para formalizar o significar de funções currificadas. Por exemplo, a definição

```haskell
soma :: Int -> Int -> Int
soma x y = x + Y
```

Pode ser entendida como

```haskell
soma :: Int -> Int -> Int
soma = \x -> (\y -> x + y)
```

Esta última forma deixa claro que a função soma recebe um parâmetro `x` e retorna uma função, que por sua vez recebe um outro parâmetro chamado `y` e retorna o valor `x + y`.

Expressões lambda também podem ser usadas quando precisamos criar uma função muito simples que não será usada em nenhum outro lugar no programa. Por exemplo, queremos definir uma função chamada *impares*, que retorna os *n* primeiros números ímpares. Ela pode ser definida assim:

```haskell
impares :: Int -> [Int]
impares n = map f [0 .. n-1]
            where f x = x * 2 + 1
```

Como a função `f` só será usada neste contexto, podemos redefinir a função desta forma:

```haskell
impares :: Int -> [Int]
impares n = map (\x -> x * 2 + 1) [0 .. n-1]
```


## Seções

Funções como `+`, que são escritas entre seus dois argumentos são chamados de operadores. Qualquer função com dois parâmetros pode ser convertida para um operador envolvendo seu nome com crases, como \`div\`, que vimos anteriormente. O contrário também pode ser feito, pois afinal de contas todos os operadores são funções. Esse efeito é alcançado envolvendo o operador em parênteses e colocando-o à frente dos seus argumentos, gerando uma aplicação currificada: `(+) 1 2`. Também é possível incluir um dos argumentos dentro dos parênteses, isto é, `(1 +) 2 ` e `(+ 1) 2` também são válidos. Enquanto `(+)` vai "gerar" uma função que esperar dois argumentos, `(+1)` ou `(1+)` vai gerar uma função que espera apenas um argumento, pois o primeiro já foi aplicado.

Em geral, se **@** é um operador, expresões como (**@**), (x **@**) e (**@** y) pra argumentos *x* e *y*, são chamadas de *seções*, cujo significado pode ser espeficado como funções lambda da seguinte forma:

```haskell
(@) = \x -> (\y -> x + y)

(x @) = \y -> x @ y

(@ y) = \x -> x @ y
```

As seções tem três aplicações básicas. Primeiro, elas podem ser usadas para criar funções simples, mas bastante úteis e compactas:
* (+) é a função de adição: `\x -> (\y -> x + y)`
* (1+) é a função de incremento: `\x -> x + 1`
* (1/) é a função que calcula o inverso de um número: `\x -> 1 / x`
* (*2) é a função de duplicação: `\x -> 2 * x`
* (/2) é a função metade: `\x -> x / 2`

Segundo, seções são importantes para definir o tipo de operadores, pois um operador em si não é uma expresso válida em Haskell. O tipo do operador **+** para inteiros é da forma:
```haskell
(+) :: Int -> Int -> Int
```

Terceiro, seções também são úteis quando desejamos passar operadores como parâmetros. Por exemplo, a função *sum*, da bilioteca padrão, que calcula a soma de todos os elementos em uma lista, pode ser definida a partir da função *foldl*:

```haskell
sum :: [Int] -> Int
sum = foldl (+) 0
```
