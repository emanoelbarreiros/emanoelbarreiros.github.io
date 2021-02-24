---
layout: single
title:  "[8] Funções de alta ordem"
toc: true
lesson: 8
---

Neste post estudamos as funções de alta ordem, que permitem a modularização de comportamentos recorrentes em funções que podem ser reutilizadas posteriormente.


## Conceitos básicos

Como vimos anteriormente, funções com múltiplos argumentos são possíveis em Haskell devido ao *currying*, isto é, os argumentos são aplicados um por vez, explorando o fato de que funções podem retornar outras funções como resultado. Por exemplo, a definição da função *soma*

```haskell
soma :: Int -> Int -> Int
soma x y = x + y
```
 
que é equivalente a

```haskell
soma = \x -> (\y -> x + y)
```

que codifica uma função que recebe o parâmetro inteiro *x* e retorna uma função, que por sua vez recebe um parâmetro inteiro *y* e retorna o valor *x + y*. Devido a esta funcionalidade da linguagem é possível passar funções como argumentos para outras funções. Por exemplo, podemos definir uma função que recebe uma função e um valor, e retorna o resultado da aplicação da função duas vezes:

```haskell
duasVezes :: (a -> a) -> a -> a
duasVezes f x = f (f x)
```

Exemplo de aplicação:

> \>duasVezes (+1) 3  
> 5  
>  
> \> duasVezes reverse [1,2,3]  
> [1,2,3]

Formalmente, uma função que recebe outras funções como parâmetro é o que chamamos de *função de alta ordem*. Em contraste, quando falamos de funções que podem ser tratadas como valor na linguagem, damos a estas o nome de *funções de primeira classe*. O fato de falarmos muito do *currying* no Haskell pode confundir um pouco os conceitos de funções e alta ordem e de primeira classe, mas todo os três termos dizem respeito a coisas diferentes.


## Processamento de listas

### A função *map*
O Prelude nos dá uma série de funções de altar ordem para tratar listas. Como primeiro exemplo temos a função *map*, que é capaz de aplicar uma outra função a todos os elementos de uma lista recebida como argumento:

```haskell
map :: (a -> b) -> [a] -> [b]
map f xs = [f x | x <- xs]
```

Com a função *map* podemos, por exemplo, incrementar em uma unidade todos os valores em uma lista de inteiros:

> \> map (+1) [1,2,3]  
> [2,3,4]



### A função *filter*

Outra função muito útil da biblioteca padrão é a *filter*. Ela é capaz de selecionar elementos de uma lista a partir da avaliação de um predicado, que caso avaliado para *True* confirma a inclusão do valor original da lista no resultado. Quando o predicado é avaliado para *False* o elemento não será incluído no resultado. A função *filter* pode ser definida da seguinte forma:

```haskell
filter :: (a -> Bool) -> [a] -> [a]
filter p xs = [x | x <- xs, p x]
```

A função *p* funciona como uma [guarda](http://localhost:4000/funcional/haskell-5#guardas), ou seja, `filter p xs` retornará uma lista de todos os valores *x*, tal que *x* é um elemento de *xs* e o valor `p x` é *True*. Exemplos

> \> filter even [1 .. 10]  
> [2, 4, 6, 8, 10]  
>   
> \> filter (> 5) [1 ..10]  
> [6, 7, 8, 9, 10]
>   
> filter (/= ' ') "abc def ghi"  
> "abcdefghi"


### Juntando *map* e *filter*

As funções *map* e *filter* podem ser usadas juntas, e é até comum vermos isso no dia a dia. Em geral, usa-se a função filter para selecionar alguns elementos de uma lista e em seguida a função map para modificá-los de alguma forma. Tome como exemplo a função abaixo, que a partir de uma lista retorna a soma dos elementos pares elevados ao quadrado:

```haskell
somaQuadPares :: [Int] -> Int
somaQuadPares ns = sum (map (^2) (filter even ns))
```

Mais alguns exemplos de uso da funcionalidade de funções de alta ordem:

* Decidir se todos os elementos em uma lista satisfazem um predicado:
> \> all even [2,4,6,8]  
> True
* Decidir de pelo menos um elemento de uma lista satisfaz um predicado:
> \> any odd [2,4,6,8]  
> False
* Selecionar elementos de uma lista enquanto eles satisfazem um predicado:
> \> takeWhile even [2,4,6,7,8]  
> [2,4,6]
* Remover elementos de uma lista enquanto eles satisfazem um predicado:
> \> dropWhile odd [1,3,5,6,7]  
> [6,7]


## A função *foldr*

Um idioma é observado recorrentemente quando queremos processar uma lista a partir de uma operação a ser aplicada aos seus elementos na forma:

```haskell
f [] = v
f (x:xs) = x # f xs
```

Isto é, a função mapeia a lista vazia para um valor *v*, e qualquer lista não vazia para um operador *#* aplicado à cabeça da lista e ao resultado da aplicação recursiva de *f* à cauda da lista. Seguem alguns exemplos:

```haskell
soma [] = 0
soma (x:xs) = x + soma xs

produto [] = 1
produto (x:xs) = x * produto xs

ou [] = False
ou (x:xs) = x || ou xs

e [] = True
e (x:xs) = x && e xs
```

A função de alta ordem *foldr* (abreviatura para *fold right*) encapsula este padrão de recursão definindo funções sobre listas, com um operador *#* e o valor *v* como argumentos. Por exemplo, as funções escritas acima podem ser redefinidas usando *foldr*:

```haskell
soma :: Num a => [a] -> a
soma = foldr (+) 0
-- alternativamente: soma xs = foldr (+) 0 xs

produto :: Num a => [a] -> a
produto = foldr (+) 1
-- alternativamente: produto xs = foldr (+) 1 xs

ou :: [Bool] -> Bool
ou = foldr (||) False
-- alternativamente: ou xs = foldr (||) False xs

e :: [Bool] -> Bool
e = foldr (&&) True
-- alternativamente: e xs = foldr (&&) True xs
```

Como será que podemos definir a função *foldr* recursivamente? Pare um pouco pra pensar e tente encontrar uma definição. Depois que você pensar um pouco compare a sua solução com a [minha](https://gist.github.com/emanoelbarreiros/5ce4e8258529961a2d9cf99cf8543f2f).


Embora *foldr* encapsule um padrão recursivo simples, ela pode ser usada para definir outras funções que à primeira vista não estariam relacionadas. Um exemplo é a função *length*, que inicialmente pode ser definida como

```haskell
length :: [a] -> Int
length [] = 0
length (_:xs) = 1 + length xs
```

Mas pode ser reescrita utilizando-se a função *foldr*:

```haskell
length :: [a] -> Int
length = foldr (\_ n -> 1 + n) 0
```

Um outro exemplo interessante é com a função *reverse*, que inverte uma lista. Ela também pode ser implementada usando-se *foldr*. Inicialmente vamos ver como poderia ser sua implementação sem utilizar *foldr*:
```haskell
reverse :: [a] -> [a]
reverse [] = []
reverse (x:xs) = reverse xs ++ [x]
```

Aplicando *reverse* sobre a lista `1 : (2 : (3 : []))`, teremos:
>  (([] ++ [3]) ++ [2]) ++ [1]

Utilizar *foldr* para implementar a função *reverse* não é muito intuitivo. Primeiro precisamos definir uma função auxiliar. Aqui o nome dela é uma brincadeira com o operador *cons*, cujo símbolo é o caractere ':':
```haskell
snoc x xs = xs ++ [x]
```

Ela tem o nome *cons* invertido, pois faz o inverso do que o *cons* faz, ele adiciona um elemento no fim de uma lista, ao invés da cabeça. Podemos então fazer a primeira reescrita da função *reverse*:
```haskell
reverse :: [a] -> [a]
reverse [] = []
reverse (x:xs) = snoc x (reverse xs)
```

Reescrever a função *reverse* usando *foldr* agora é um pouco mais fácil:
```haskell
reverse :: [a] -> [a]
reverse = foldr snoc []
```

De maneira geral, considerando *#* um operador genérico, podemos entender a função *foldr* como:

```haskell
foldr (#) v [x0, x1, ..., xn] = x0 # (x1 # (... (xn # v) ...))
```

## A função *foldl*

Também é possível definir funções recursivas a partir de operadores associativos à esquerda. Podemos por exemplo, redefinir a função *soma* para que ela utilize uma função auxiliar que recebe um argumento extra que acumula o resultado parcial da soma:

```haskell
soma :: Num a => [a] -> a
soma = soma' 0
       where
            soma' v []  = v
            soma' v (x:xs) = soma' (v+x) xs
```

```
   soma [1,2,3]
=  soma' 0  [1,2,3] --> aplicando soma'
=  soma' (0 + 1) [2,3] --> aplicando soma'
=  soma' ((0 + 1) + 2) [3] --> aplicando soma'
=  soma' (((0 + 1) + 2) + 3) [] --> aplicando soma'
=  6 -> aplicando +
```

A forma como os parênteses estão organizados mostra que agora a soma está associada à esquerda. No caso específico do soma não faz diferença, pois a soma é [associativa](https://pt.wikipedia.org/wiki/Associatividade). Com a subtração, por exemplo, aplicar *foldl* e *foldr* produz resultados diferentes.

De maneira geral, a função *foldl* implementa o padrão recursivo a seguir:
```haskell
f v [] = v
f v (x:xs) = f (v # x) xs
```

Traduzindo, a função mapeia a lista vazia para o valor acumulado *v*, e qualquer lista não-vazia para o resultado do processamento recursivo da cauda usando um novo valor acumulado, resultado da aplicação do operador *#* ao valor acumulado atual e a cabeça da lista. Para as funções *soma*, *produto* *ou* e *e*, não faz diferença usar o *foldl* ou *foldr*, pois as operações que elas implementam são associativas, mas para as funções *length*  e *reverse* precisamos fazer adaptações:
```haskell
length :: [a] -> Int
length = foldl (\n _ -> n + 1) 0

reverse :: [a] -> [a]
reverse = foldl (\xs x -> x : xs) []
```

O que resulta na seguinte avaliação 
> length [1,2,3] = ((0 + 1) + 1) + 1 = 3  
> reverse [1,2,3] = 3 : (2 : (1 : []))

A função *foldl* propriamente dita pode ser especificada da seguinte forma:
```haskell
foldl :: (a -> b -> a) -> a -> [b] -> a
foldl f v [] = v
foldl f v (x:xs) = foldl f (f v x) xs
```

Na prática, entretanto, é mais fácil imaginar o *foldl* não recursivamente. Considerando *#* um operador genérico, *foldl* tem a seguinte semântica:
```haskell
foldl (#) v [x0, x1, ..., xn] = (... ((v # x0) # x1) ..) # xn
```

## Comparando *foldr* e *foldl*

As diferenças entre *foldr* e *foldl* à primeira vista podem ser pequenas, mas são dramáticas. Para facilitar, vamos nos ater às simplficações mostradas no fim de cada seção:
```haskell
foldr (#) v [x0, x1, ..., xn] = x0 # (x1 # (... (xn # v) ...))

foldl (#) v [x0, x1, ..., xn] = (... ((v # x0) # x1) ..) # xn
```

Primeiro perceba a ordem de avaliação das operações. Com o *foldr* a avaliação final da expressão resultante da aplicação começa a ser avaliada em `(xn # v)`, ou seja, com o último elemento da lista, e vai seguindo para fora, operando o resultado desta sub-expressão com x<sub>n-1</sub>, x<sub>n-2</sub>, até x<sub>0</sub>. 

Por outro lado, a avaliação da expressão gerada por *foldl* vai iniciar a avaliação com `v # x0`, ou seja, com o primeiro elemento da lista, e vai seguindo para fora operando este resultado com x<sub>1</sub>, x<sub>2</sub>, até x<sub>n</sub>.

Segundo, perceba também que a posição dos operandos nas operações é diferente. Com *foldr* o operando *v* é aplicado do lado direito do operador, enquanto que em *foldl* o operando *v* é colocado do lado esquerdo da operação.


## O operador de composição

O operador de composição . retorna uma nova função, que é o resultado da composição das duas funções que ele recebe como argumentos. Ele pode ser definido da seguinte forma:
```haskell
(.) :: (b -> c) -> (a -> b) -> (a -> c)
f . g = \x -> f (g x)
```

Ou seja, `f . g`, lido como *f de g*, recebe um argumento *x*, aplica primeiramente *g* a este argumento, e ao resultado desta aplicação é aplicada a função *f*. O operador de composição pode ser usado para simplificar a aplicação de funções aninhandas. Por exemplo, as funções
```haskell
impar n = not (par n)
duasVezes f x = f (f x)
somaQuadradosPares ns = sum (map (^2) (filter par ns))
```

podem ser reescritas como

```haskell
impar = not . par
duasVezes = f . f
somaQuadradosPares ns = sum . map (^2) . filter par
```

Um fato interessante, é que a definição de *somaQuadradosPares* se apoia no fato de que a composição de funções é associativa, isto é, `f . (g . h) = (f . g) . h` para quaisquer funções *f*, *g*, e *h* com os tipos apropriados, logo, não é necessário colocar os parênteses, a associatividade garante isso pra a gente.

Vou deixar aqui só mais um fato, que pode esquentar sua cabeça por alguns minutos. Uma função que ninguém dá nada por ela, mas quebra um galho danado é a função *id*, abreviação para identidade. Como o próprio nome sugere, ela simplesmente retorna o que recebe como parâmetro:

```haskell
id :: a -> a
id = \x -> x
```

Ela possui uma propriedade interessante: `f . id = f` e `id . f = f` para qualquer função *f*. Além de ela ser útil quando analizamos programas, ela pode ser usada para criar uma função capaz de fazer a composição de funções dispostas em uma lista:
```haskell
compor :: [a -> a] -> (a -> a)
compor = foldr (.) id
```

Vamos tentar entender um pouco melhor o que foi dito aqui, porque tem algumas coisas acontecendo por baixo dos panos. Em um cenário hipotético podemos ter a seguinte aplicação:
> \> novaFuncao = compor [f, g, h]

Note que estamos fazendo um *foldr*, um fold para a direita, dê uma lembrada no que isso significa:
```haskell
foldr (#) v [x0, x1, ..., xn] = x0 # (x1 # (... (xn # v) ...))
```

Como estamos fazendo um *foldr*, temos o equivalente a isso aqui:
> \> novaFuncao = f . (g . (h . id))

Como `h . id` é igual a `id . h`, qualquer um dos casos é igual a `h`, ou seja, temos em *novaFuncao* a composição das três funções *f*, *g*, e *h*:
> \> novaFuncao = f . (g . h)

Se usássemos um *foldl* o resultado seria diferente? Converse com o seu professor sobre isso.