---
layout: single
title:  "[7] Funções recursivas"
toc: true
lesson: 7
---

Nesta aula exploramos o mecanismo de recursão da linguagem para implementarmos o conceito de loops.


## Conceitos básicos

Como já vimos em várias ocasiões, é possível definir funções a partir de outras funções, sejam elas da biblioteca básica do Haskell ou implementadas por você mesmo. Por exemplo, a função *fatorial* pode ser facilmente implementada usando a função *product*, que calcula o produto dos valores de uma lista de inteiros:

```haskell
fatorial :: Int -> Int
fatorial n = product [1 .. n]
```

Entretanto, na maioria das linguagens é possível definir funções a partir delas mesmas. Por exemplo, a função *fatorial* pode ser reescrita utilizando essa idéia:

```haskell
fatorial :: Int -> Int
fatorial 0 = 1
fatorial n = n * fatorial (n - 1)
```

O primeiro padrão diz que sempre que a função *fatorial* for chamada com o argumento 0 a função deve retornar 1. Isso é chamado de *caso base*. Nas demais situações, a função *fatorial* vai multiplicar *n* pelo fatorial do seu antecessor. Este é chamado de *passo recursivo*.

Como pode ser observador, apesar de a função *fatorial* ser definida em termos dela mesma, isso não vai ocasionar um loop infinito, pois a cada passo do algoritmo o valor de *n* está sendo reduzido de uma unidade, e em algum momento ele chegará a 0, fazendo com que a função não mais seja invocada recursivamente. Neste momento a recursão para e as multiplicações são executadas.


## Recursão em listas

A recursão não é limitada a funções sobre inteiros, elas também podem ser definidas para trabalhar com listas. Por exemplo, a função *product* mencionada anteriormente, pode ser facilmente definida como uma função recursiva. Embora essa não seja a implementação padrão da biblioteca, ela é bastante fácil de entender:

```haskell
product :: Num a => [a] -> a
product [] = 1
product (x:xs) = x * product xs
```

Ou seja, a multiplicação de uma lista pode ser traduzida como a multiplicação do primeiro elemento da lista pelo produto do restante da lista. Um outro exemplo bastante intuitivo é a função *length*, que calcula o tamanho de uma lista:

```haskell
length :: [a] -> Int
length [] = 0
length (_:xs) = 1 + length xs
```

### O algoritmo *insertion sort* utilizando funções recursivas

Iniciemos implementando uma função chamada *inserir*, que é capaz de inserir um elemento na posição correta em uma lista ordenada.

```haskell
inserir :: Ord a => a -> [a] -> [a]
inserir x [] = [x]
inserir x (y:ys) 
    | x <= y = x : y : ys
    | otherwise = y : inserir x ys
```

Inserir um elemento em uma lista vazia vai gerar o que chamamos de *singleton*, uma lista com apenas um elemento. Caso a lista onde o novo item será inserido não seja uma lista vazia, o algoritmo precisa encontrar a posição onde colocá-lo. Caso ele seja menor que o primeiro elemento da lista (lembre-se que a lista precisa estar ordenada previamente), seu lugar é na cabeça da lista, o que é alcançado fazendo `x : y : ys`. Caso ele não seja menor que a cabeça da lista, a função é invocada recursivamente, mas agora tentando inserir o *x* na cauda da lista. Veja um exemplo de execução:

> inserir 3 [1,2,3,4,5]  
> = 1 : inserir 3 [2,4,5]  
> = 1 : 2 : inserir 3 [4,5]  
> = 1 : 2 : 3 : [4,5]  
> = [1,2,3,4,5]

Agora que temos a função *inserir* podemos implementar a função *iSort*. Nesta implementação o problema é reduzido a ordenar uma lista vazia, e depois disso inserir os elementos na ordem correta a passo recursivo. Segue a implementação:

```haskell
iSort :: Ord a => [a] -> [a]
iSort [] = []
iSort (x:xs) = inserir x (iSort xs)
```

Segue um exemplo de execução:

> iSort [3,2,1,4]  
> = inserir 3 (iSort [2,1,4])  
> = inserir 3 (inserir 2 (iSort [1,4]))  
> = inserir 3 (inserir 2 (inserir 1 (iSort [4])))  
> = inserir 3 (inserir 2 (inserir 1 (inserir 4 (iSort []))))  
> = inserir 3 (inserir 2 (inserir 1 (inserir 4 [])))  
> = inserir 3 (inserir 2 (inserir 1 (4 : [])))  
> = inserir 3 (inserir 2 (1 : (4 : [])))  
> = inserir 3 (1 : inserir 2 (4 : []))  
> = inserir 3 (1 : (2 : (4 : [])))  
> = 1 : inserir 3 (2 : (4 : []))  
> = 1 : 2 : inserir 3 (4 : [])  
> = 1 : 2 : 3 : (4 : [])  
> = [1,2,3,4]  


## Recursão múltipla

Funções também podem ser definidas usando recursão múltipla, isto é, o passo recursivo pode invocar a função recursivamente mais de uma vez. Tome come exemplo a função que calcula o número de [Fibonacci](https://pt.wikipedia.org/wiki/Sequ%C3%AAncia_de_Fibonacci):

```haskell
fib :: Int -> Int
fib 0 = 0
fib 1 = 1
fib n = fib (n - 1) + fib (n - 2)
```


## Recursão mútua

Algumas funções possuem uma relação muito íntima, invocando uma à outra repetidas vezes, criando o que chamamos de recursão mútua. Por exemplo, considere as funções *par* e *impar*, cujas implementações são mais eficientes testando o resto após uma divisão por 2, mas podem ser definidas uma em função da outra, em um arranjo curioso:

```haskell
par :: Int -> Bool
par 0 = True
par n = impar (n - 1)

impar :: Int -> Bool
impar 0 = False
impar n = par (n - 1)
```

Traduzindo, o número 0 é par e não é ímpar. Qualquer outro número é classificado como par se seu antecessor é ímpar, e um número é classificado como ímpar se seu antecessor é par.