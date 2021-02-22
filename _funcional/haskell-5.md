---
layout: single
title:  "[6] Compreensão de listas"
toc: true
lesson: 6
---

Neste post vamos explorar o conceito de compreensão de listas.

## Conceitos básicos

Na Matemática, a notação de compreensão de listas pode ser usada para construir novos conjuntos a partir de outros conjuntos pré-existentes. Por exemplo, a compreensão { x<sup>2</sup> \| x &#8712; {1 .. 5} } produz o conjunto {1, 4, 9, 16, 25}. Em Haskell temos uma notação muito similar. A compreensão acima pode ser expressada como:

```haskell
[x ^ 2 | x <- [1 .. 5]]
```

O símbolo `|` é lido como *tal que*, `<-` é lido como *tirado de* e a expressão `x <- [1 .. 5]` é chamada de gerador. É possível ter mais de um gerador, separados por vírgulas. Por exemplo, todas as combinações possíveis entre elementos do conjunto [1,2,3] com os elementos do conjunto [4,5] pode ser expressa como:

```haskell
[(x,y) | x <- [1,2,3], y <- [4,5]]
```
Resultado:
> [(1,4), (1,5), (2,4), (2,5), (3,4), (3,5)]

Mudar a ordem dos geradores influencia a ordem em que os itens são montados:

```haskell
[(x,y) | y <- [4,5], x <- [1,2,3]]
```
Resultado:
> [(1,4), (2,4), (3,4), (1,5), (2,5), (3,5)]


## Guardas

Algumas vezes podemos querer filtrar alguns valores da lista original quando estamos executando a compreensão. Para fazer isso, utilizamos o que se chama de *guarda*. Ela é nada mais do que uma expressão booleana, que é avaliada para cada valor gerado pelo gerador. Vejamos um exemplo:

```haskell
fatores :: Int -> [Int]
fatores n = [x | x <- [1 .. n], n `mod` x == 0]
```

Após o gerador vemos a expressão `n \`mod\` x == 0` é avaliada para os valores entre 1 e *n*, e só será incluído se *x* divide *n* perfeitamente (resto 0).

> \> fatores 15  
> [1, 3, 5, 15]

Podemos utilizar a função *fatores* para implementar uma função simples que vai determinar se um número é primo:

```haskell
primo :: Int -> Bool
primo n = fatores n == [1,n]
```

De posse da função *primo*, podemos agora construir, usando compreensão de listas, uma list com todos os primos entre 2 e um valor n arbitrário:

```haskell
primos :: Int -> [Int]
primos n = [x | x <- [2 .. n], primo x]
```

Podemos implementar um outro exemplo interessante, uma função que encontra elementos em uma tabela a partir de uma chave. Imagine que tenhamos uma lista com tuplas `(k,v)`, onde `k` representa a chave e `v` o valor referenciado pela chave. A função `buscar` filtra todos os elementos cuja chave informada na invocação é igual à chave na tabela:

```haskell
buscar :: Eq a => a -> [(a,b)] -> [b]
buscar k xs = [v | (k', v) <- xs, k == k']
```

> \> buscar 'a' [('a', 1), ('b', 2), ('c', 3), ('a', 4)]  
> [1, 4]


## A função *zip*

A função *zip* faz uma lista a partir do pareamente de elementos de duas listas distintas. Ela finaliza o processo de pareamento quando uma ou ambas as listas não possuem mais elementos para parear. Por exemplo:

> \> zip ['a', 'b', 'c'] [1, 2, 3, 4, 5]  
> [('a', 1), ('b', 2), ('c', 3)]

A função *zip* é comumente usada quando trabalhamos com compreensão de listas. Por exemplo, imagine que definamos uma funço chamada *pares*, que faz o casamento de um número com seu sucessor em uma lista:

```haskell
pares :: [a] -> [(a,a)]
pares xs = zip xs (tail xs)
```

Exemplo de aplicação:

> \> pares [1,2,3,4]  
> [(1,2), (2,3), (3,4)]

Agora, podemos usar a função *pares* para decidir se uma lista de números está ordenada:
 
```haskell
ordenada :: Ord a => [a] -> Bool
ordenada xs = and [x <= y | (x,y) <- pares xs]
```

Exemplo de aplicação:

> \> ordenada [1,2,3,4]  
> True
> \> ordenada [2,1,3,4]  
> False

Usando a função *zip* também podemos construir uma função que encontra todas as ocorrências de um valor em uma lista e retorna as posições onde eles se encontram:

```haskell
posicoes :: Eq a => a -> [a] -> [Int]
posicoes x xs = [i | (z, i) <- zip xs [0 ..], x == z]
```

Observe que utilizamos a construção `[0 ..]`, o que geraria uma lista infinita. Obviamente essa lista infinita não é gerada. O Haskell só irá gerar a quantidade de elementos que ele precisa, pois a função *zip* encerra o processamento quando uma das listas passadas como parâmetro é esgotada. Isso é possível devido à avaliação preguiçosa (*lazy evaluation*) da linguagem.