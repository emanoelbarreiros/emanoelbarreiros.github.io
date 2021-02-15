---
layout: single
title:  "[4] Tipos e Classes (Parte 2 de 2)"
toc: true
lesson: 4
---

Dando continuidade ao conceito de tipos de classes, vamos ver os tipos polimórficos, tipos sobrecarregados, o conceito de classes e algumas classes básicas.

## Tipos polimórficos

A função `length` calcula o tamanho de qualquer lista, não importando o tipo dos elementos contidos nela. A idéia de que o comprimento pode ser calculado para listas de qualquer tipo é formalizado pela inclusão de uma *variável de tipo*. Variáveis de tipo devem começar com um caracter minúsculo. Por convenção utilizam-se os caracteres *a*, *b*, *c* e assim por diante. O tipo da função `length` é definido da seguinte forma:

```haskell
length :: [a] -> Int
```

Dizemos então que qualquer tipo que contenha uma variável de tipo é polimórfico, e qualquer função que tenha um tipo polimórfico, é polimórfica. Mais exemplos:

```haskell
fst :: (a, b) -> a
head :: [a] -> a
take :: Int -> [a] -> [a]
zip :: [a] -> [b] -> [(a,b)]
id :: a -> a
```

## Tipos sobrecarregados

O operador + pode ser usado para calcular a soma de quaisquer dois números do mesmo tipo numérico, sejam eles inteiros ou pontos flutuantes. O formalismo de que o operador + pode ser usado com tipos numéricos é denotado pela *restrição de classe*. As restrições de tipo são escritas na forma `C a`, onde `C` é o nome da classe e `a` é uma variável de tipo. Para o operador +, o tipo é definido da seguinte forma:

```haskell
(+) :: Num a => a -> a-> a
```

Isto é, para qualquer tipo `a` da classe `Num` de tipos numéricos, a função `(+)` tem o tipo `a -> a -> a`. Envolvendo o + em parênteses convertemos o operador para a forma currificada da função. Um tipo que contém pelo menos uma restrição de classes é chamado de *sobrecarregado*.

## Classes básicas

Lembre que tipo é um conjunto de valores relacionados. Construindo sobre esta definição, uma *classe* é uma coleção de tipos que suportam certas operações sobrecarregadas, chamadas de *métodos*. Vamos detalhar agora algumas classes básicas disponíveis na linguagem.

### *Eq* - tipos de igualdade

Esta classe contém os tipos cujos valores  podem ser comparados quanto à igualdade e desigualdades usando os métodos abaixo:

```haskell
(==) Eq a => a -> a -> Bool
(==) :: a -> a -> Bool

(/=) Eq a => a -> a -> Bool
(/=) :: a -> a -> Bool
```

Todos os tipos básicos *Bool*, *Char*, *String*, *Int*, *Integer* e *Float* são instâncias da classe *Eq*, bem como listas e tuplas, desde que os tipos de seus elementos também sejam instâncias da classe *Eq*. Exemplos:

> Prelude> False == False  
> True  
> Prelude> 'a' == 'b'  
> False  
> Prelude> "abc" == "abc"  
> True  
> Prelude> [1,2] == [1,2,3]  
> False  
> Prelude> ('a', 2) == ('a', 2)  
> True  


### *Ord* - tipos ordenáveis

Esta classe contém os tipos que são instâncias da classe *Eq*, mas adicionam a restrição que todos os seus valores sejam linearmente ordenáveis, e consequentemente podem ser comparados utilizando-se os seguintes métodos:

```haskell
(<) :: a → a → Bool
(<=) :: a → a → Bool
(>) :: a → a → Bool
(>=) :: a → a → Bool
min :: a → a → a
max :: a → a → a
```

A restrição de classe `Ord a =>` está sendo omitida por simplicidade, mas formalmente ela faz parte da definição do tipo dos métodos. Todos os tipos básicos *Bool*, *Char*, *String*, *Int*, *Integer* e *Float* são instâncias da classe *Ord*, bem como listas e tuplas, desde que os tipos de seus elementos também sejam instâncias da classe *Ord*. Exemplos:

> Prelude> False < False  
> True  
> Prelude> min 'a' 'b'  
> 'a'  
> Prelude> "UFPE" < "UPE"  
> True  
> Prelude> [1,3] < [1,2,3]  
> False  
> Prelude> ('a', 2) < ('a', 3)  
> True  

Observe que para *String*s, listas e tuplas, a ordem é definida a partir do primeiro elemento. Caso os primeiros elementos de ambas as coleções sejam iguais, passa-se a comparar o segundo elemento, e assim por diante.


### *Show* - tipos exibíveis

Esta classe contém os tipos cujos valores que podem ser convertidos para String usando o seguinte método:

```haskell
show :: a -> String
```

Todos os tipos básicos *Bool*, *Char*, *String*, *Int*, *Integer* e *Float* são instâncias da classe *Show*, bem como listas e tuplas, desde que os tipos de seus elementos também sejam instâncias da classe *Show*.


### *Read* - tipos que podem ser lidos

Funciona em conjunto com a classe Show, e contém os tipos cujos valores podem ser convertidos a partir de Strings, usando o seguinte método:

```haskell
read :: String -> a
```

Todos os tipos básicos *Bool*, *Char*, *String*, *Int*, *Integer* e *Float* são instâncias da classe *Read*, bem como listas e tuplas, desde que os tipos de seus elementos também sejam instâncias da classe *Read*. Exemplos:

> Prelude> read "False" :: Bool  
> False  
> Prelude> read "’a’" :: Char  
>’a’  
> Prelude> read "123" :: Int  
> 123  
> Prelude> read "[1,2,3]" :: [ Int ]  
> [1, 2, 3]  
> Prelude> read "(’a’,False)" :: ( Char , Bool )  
> (’a’, False )  

O uso de :: nestes exemplos serve para determinar o tipo do resultado. Na prática, na maioria dos casos seu uso não é necessário, pois o tipo do resultado pode ser inferido pelo Haskell de acordo com o uso da expressão em expressões maiores. Por exemplo, a expressão `not (read "False")` não precisa usar operador :: pois como a expressão será aplicada a uma outra função, esta que recebe um valor *Bool*, o resultado de `read "False"` só pode ser *Bool*. Entretanto, o resultado da função `read` é indefinido se o valor a ser convertido não for compatível com o tipo esperado. Por exemplo, `not (read "olá")` resultará em um erro, pois a String "olá" não pode ser convertida para um valor *Bool*.


### *Num* - tipos numéricos

Esta classe contém os tipos que são instâncias das classes *Eq* e *Show*, mas que também sejam numéricos e, como tal, possam ser manipulados utilizando os seguintes métodos:

```haskell
(+) :: a -> a -> a
(−) :: a -> a -> a
(∗) :: a -> a -> a
negate :: a -> a
abs :: a -> a
signum :: a -> a
```

O método `negate`retorna o simétrico de um número, `abs` retorna o valor absoluto de um número e o `signum` o seu sinal:

> Prelude> 1 + 2  
> 3  
> Prelude> 1.1 + 2.2  
> 3.3  
> Prelude> negate 3.3  
> −3.3  
> Prelude> abs (−3)  
> 3  
> Prelude> signum (−3)  
> −1

### *Integral* - tipos integrais

Esta classe contém os tipos que são instâncias da classe *Num*, mas cujos valores são inteiros, e como tal, suportam as função de divisão e resto:

```haskell
div :: a -> a-> a
mod :: a -> a-> a
```

Na prática esses métodos são usados utilizand-se a notação infixa, envolvendo o nome deles em acentos graves:

> Prelude> 7 \`div\` 2  
> 3  
> Prelude> 7 \`mod\` 2  
> 1

### *Fractional* - tipos fracionários

Esta classe contém os tipos que são instâncias da classe *Num*, mas que cujos valores não são *Integral* (não inteiros) e, como tal, suportam as operações de divisão fracrionária e recípro:

```haskell
(/) :: a -> a _->_ a
recip :: a -> a
```

O tipo básico *Float* é instância da classe *Fractional*.