---
layout: single
title:  "[2] Tipos e Classes (Parte 1 de 2)"
toc: true
---

Este artigo fala sobre tipos e classes, conceitos 

## Tipo

Um tipo é uma coleção de valores relacionados. Você deve ter percebido que essa definição também se aplica a conjuntos, e é exatamente isso que um tipo significa. O tipo `Bool`, por exemplo, só tem dois valores possíveis, `True` e `False`. A função `not`, negação lógica, que tem o tipo `Bool -> Bool` faz o mapeamento entre um valor `Bool` e outro valor do mesmo tipo. Usa-se a notação `v :: T` para denotar que `v` tem o tipo `T` . Exemplo:

```haskell
False :: Bool
True :: Bool
not :: Bool -> Bool
```

Em Haskell todas as expressões tem um tipo, que é calculado antes de executar a expressão através de um mecanismo chamado *inferência de tipo*. Este cálculo é realizado através da aplicação de regras de tipo para aplicação de funções. Em outras palavras, se uma `f` é uma função que mapeia argumentos do tipo `A` para resultados do tipo `B`, e se `e` é uma expressão do tipo `A`, então aplicar `f` a `e` resultará em um valor do tipo `B`.

Exemplos simples são a disjunção e a conjunção lógicas. Duas possibilidades para a definição das funções `ou` e `e` é exibida abaixo:

```haskell
ou :: Bool -> Bool -> Bool
ou True _ = True
ou False a = a

e :: Bool -> Bool -> Bool
e False _ = False
e True a = a
```

A definição da função `ou` traz para nós as seguintes informações:
* a função `ou` deve receber dois valores Bool e retornar um valor Bool;
* quando a função `ou` receber o valor `True` como o primeiro valor, o segundo valor recebido como argumento não importa, o retorno sempre será `True`;
* quando a função `ou` receber o valor False como primeiro argumento, o resultado da função será igual ao valor recebido como segundo argumento.

A definição da função `e` traz para nós as seguintes informações:
* a função `e` deve receber dois valores Bool e retornar um valor Bool;
* quando a função `e` receber o valor `False` como o primeiro valor, o segundo valor recebido como argumento não importa, o retorno sempre será `False`;
* quando a função `e` receber o valor `True` como primeiro argumento, o resultado da função será igual ao valor recebido como segundo argumento.

o Haskell sabe então que a expressão `e True (ou False True)` só pode ser do tipo `True` porque a função `ou` mapeia dois valores `Bool` em um valor `Bool`, então o tipo da expressão `(ou False True)` será `True`. Neste momento, o interpretador sabe que a aplicação da função `ou` é válida, pois ela está recebendo dois argumentos do tipo correto, ele agora passa a avaliar a segunda parte da expressão, que é `e True val`. Assuma que `val` é o resultado da avaliação da expressão anterior, e que o Haskell sabe que ela é do tipo `Bool` (`val :: Bool`). A função `e` precia de dois valores booleanos para operar, e o Haskell verifica que isso confere, pois `True :: Bool` e `val :: Bool`, logo a aplicação da função `e` é válida, e o valor esperado ao fim da sua avaliação é `Bool`.

Ao fim desta avaliação, o Haskell verificou que a expressão é válida do ponto de vista dos tipos envolvidos. Um programa real será composto de várias outras expressões, e a verificação de tipos será realizada desta forma para cada uma das aplicações das diversas funções envolvidas.

Você pode verificar o tipo de qualquer expressão utilizando o comando `:type` no Hugs. No nosso caso, seria algo assim:

> *Main> :type (e True (ou False True))  
> (e True (ou False True)) :: Bool

Neste caso o prompt não tem mais a palavra Prelude pois carregamos um módulo utilizando, o comando `:load`, contendo a nossa função.


## Tipos básicos

Haskell nos dá alguns tipos básicos. Os mais comuns são listados abaixo.

* *Bool* - **Valores lógicos**
Este tipo possui apenas os valores `True` e `False`.

* *Char* - **Caracteres únicos**
Contém todos os caracteres únicos que são possíveis de se representar em um computador, como 'a', 'A', '6' e '_'. Também possui os caracteres de controle '\n' (quebra de linha) e '\t' (tabulação). Caracteres únicos devem ser envolvidos pelas aspas simples.

* *String* - **Cadeia de caracteres**
Contém todas as cadeias de caracteres que podem ser representadas pela linguagem, incluindo a string vazia, "". Toda String deve ser envolvidas por aspas duplas.

* *Int* - **Inteiros de precisão fixa**
Contém todos os inteiros com uma precisão fixa. A precisão é definida por um espaço de memória previamente especificado para seu armazenamento. No Hugs os valores Int precisam estar dentro do intervalo [-2<sup>31</sup>;2<sup>31</sup> - 1]. Esta limitação é puramente voltada para fins de desempenho.

* *Integer* - **Inteiros de precisão arbitrária**
Este tipo contém *todos* os inteiros, utilizando a quantidade de memória necessária para representá-los. Além da quantidade de memória utilizada para armazenar valores deste tipo, os computadores também possuem instruções dedicadas ao processamento de inteiros de precisão fixa. Por outro lado, a representação e manipulação de inteiros de precisão arbtirária precisará de tratamentos de software mais elaborados, o que impacta diretamente o desempenho. Utilize o tipo Integer só quando for realmente necessário. Uma dica, quase nunca é necessário.

* *Float* - **Números de ponto flutuante de precisão simples**
Este tipo contém os números com casas decimais. Semelhante aos Int, possuem um espaço de memória pré-definido para seu armazenamento.


## Listas e seus tipos

Uma lista é uma sequência de elementos do mesmo tipo. Seus elementos são envolvidos por colchetes e separados por vírgulas. Escrevemos `[T]` para representar uma lista cujos elementos sejam do tipo *T*. Por exemplo

> [False, True, False]            :: [Bool]  
> ['a', 'c', '3', '\n']           :: [Char]  
> ["UPE", "Garanhuns", "Haskell"] :: [String]

Podemos ter listas vazias, representando-as como []. Haskell também não nos limita quanto à quantidade de elementos que uma lista pode ter. Devido à avaliação preguiçosa, listas infinitas são naturalmente manipuladas pela linguagem.

## Tuplas

Uma tupla é uma sequencia finita de elementos, não necessariamente do mesmo tipo. É interessante ressaltar que nas tuplas não há a limitação quanto ao tipo dos elementos na tupla que existe nas listas. Seus elementos são envolvidos por parênteses, e separadas por vírgulas. A quantidade de elementos em uma tupla é chamada de *aridade*. Tuplas com apenas um elemento não são permitidas pois gerariam uma ambiguidade com a utilização de parênteses para modificação da prioridade de execução de expressões.

## Tipos de funções

Uma função é algo que mapeia argumentos para resultados. Esta definição é bastante ampla, permitindo que funções recebam uma quantidade arbitrária de argumentos, de quaisquer tipos, e retorne um outro valor, de qualquer outro tipo. Em Haskell, utilizamos a notação da seta (`->`)para definir o tipo de funções. Você pode achar estranho à primeira vista, mas é bastante apropriado se você lembrar que funções mapeiam valores de entrada em um valor de saída. Essa imagem deve ser familiar para você:

![Função](/assets/funcao.jpg)

[Fonte: [Mundo Educação](https://mundoeducacao.uol.com.br/matematica/dominio-contradominio-imagem-uma-funcao.htm)]

Agora, pra você, a notação que Haskell utiliza faz todo o sentido também :wink:


### Funções currificadas

Funções com múltiplis argumentos também podem ser tratadas de outra forma, menos óbvia, explorando o fato de que funções podem retornar outras funções. Considere o exemplo:

```haskell
soma :: (Int, Int) -> Int
soma (x, y) = x + y

soma2 :: Int -> (Int -> Int)
soma2 x y = x + y

inc :: Int -> Int
inc = soma2 1
```
A definição do tipo de `soma` declara que ela recebe uma tupla de inteiros de aridade 2, e retorna um inteiro. Por outro lado, a definição da função `soma2` declara que ela recebe um inteiro e devolve uma função, que por sua vez recebe um inteiro e retorna o resultado `x + y`. Nos aproveitando deste comportamento, podemos definir a função `inc`, que nada mais é do que a função `soma` aplicada ao argumento 1, que é uma nova função diferente de `soma`. Ao aplicar esta nova função a um segundo valor, teremos o resultado da soma de 1 mais esse valor, que é justamente o que caracteriza o incremento.

O que é mais interessante é que esse é o mecanismo que a linguagem utiliza para ser capaz de representar e operar funções com múltiplos parâmetros. Em outras palavras, em Haskell, todas as funções tem apenas um parâmetro, mas a linguagem nos fornece a funcionalidade de funções com múltiplos parâmetros através da currificação de funções com apenas um parâmetro. Considere este outro exemplo para tornar esse conceito um pouco mais claro:

```haskell
mult :: Int -> (Int -> (Int -> Int))
mult x y z = x * y * z
```

Ao fazermos a aplicação `mult 3 4 5`, na realidade o Haskell está fazendo por baixo dos panos isso aqui:

> ((mult 3) 4) 5

Traduzindo, `mult` recebe um parâmetro `Int` e retorna uma função `Int -> (Int -> Int)`. Colocando de outra forma, `mult 3` retorna uma função equivalente a esta definição:

```haskell
mult2 y z = 3 * y * z
```

Ao aplicamos o segundo valor a `mult2`, que no nosso exemplo é `mult2 4`, o que teremos como retorno é uma função equivalente a:

```haskell
mult3 z = 3 * 4 * z
```

Por último, somente ao aplicamos o terceiro valor a `mult3`, teremos o resultado final da multiplicação. Como conveniência, para evitar o uso excessivo de parênteses, o operador `->` é associativo à direita, isto é, `Int -> Int -> Int -> Int` é equivalente à expressão `Int -> (Int -> (Int -> Int))`. Por outro lado, a aplicação de funções é associativa à esquerda, o que faz com que `mult 3 4 5` seja equivalente a `(mult 3) 4) 5`.

Dessa forma fica claro como o Haskell utiliza o conceito de currificação para tornar possível a implementação de funções com múltiplos parâmetros.