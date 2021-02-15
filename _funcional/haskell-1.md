---
layout: single
title:  "[2] Começando com Haskell"
toc: true
tags: [haskell]
lesson: 2
---

Neste post daremos os primeiros passos com a linguagem Haskell.

## Instalação

O primeiro passo é instalar tudo que é necessário para fazer o Haskell funcionar. Para isso, dê uma olhada em: [https://www.haskell.org/downloads/](https://www.haskell.org/downloads/)

Depois de realizar a instalação de tudo que for necessário, retorne e continue.


## O Hugs

É possível compilar programas escritos em Haskell para código de máquina nativo, mas muitas vezes é mais conveniente rodar o código interativamente. Para isso vamos utilizar o Hugs, que é um interpretador interativo.

Dependendo da sua instalação você vai iniciar o Hugs de forma diferente. Se você instalou a *Stack*, inicie o Hugs digitando `stack ghci` no seu terminal (prompt de comando se estiver em um ambiente Windows). Se instalou a plataforma completa, digite apenas o comando `ghci` no terminal.

Você terá prompt capaz de executar código Haskell e carregar arquivos com definições de funções, que poderão ser invocadas posteriormente. Você verá o nome `Prelude` no prompt, o que significa que já temos importado o módulo Prelude, que sempre está disponível.

Temos à disposição as operações matemáticas de soma, subtração, multiplicação, divisão e potenciação. Elas funcionam como esperamos


> Prelude> 2 + 3  
> 5
> 
> Prelude> 3 / 2  
> 1.5
> 
> Prelude> 3 `div` 2  
> 1
> 
> Prelude> 3 ^ 2  
> 9


Uma coisa interessante é que `div` é uma função que recebe dois argumentos e realiza a divisão inteira do primeiro argumento pelo segundo. Entretanto, ela está sendo usada como um operador. Aproveitando, a tabela abaixo mostra a precedência de alguns operadores em Haskell:

| Nível de precedência | Op. associativos à esquerda | Op. não associativos | Op. associativos à direita |
|----------------------|-----------------------------|----------------------|----------------------------|
| 0                    |                             |                      | $, $!, ‘seq‘               | 
| 1                    | \>\>, \>\>=                 |                      |                            |
| 2                    |                             |                      | \|\|                       | 
| 3                    |                             |                      | &&                         |
| 4                    |                             | ==, /=, <, <=, >, >=, `elem`, `notElem` |         |
| 5                    |                             |                      | :, ++                      |
| 6                    | +, -                        |                      |                            | 
| 7                    | ⋆, /, `div`, `mod`, `rem`, `quot` |                |                            | 
| 8                    |                             |                      | ^, ^^, **                  |
| 9                    | !!                          |                      | .                          | 



## Algumas funções disponíveis no Prelude

O Prelude é o módulo básico do Haskell, trazendo algumas funções básicas. Segue a definição de algumas delas:

* `head`: Obtém o primeiro elemento de uma lista
> Prelude> *head* [1,2,3,4,5]  
> 1

* `tail`: Obtém a cauda da lista, i.e., a lista sem o seu primeiro elemento
> Prelude> *tail* [1,2,3,4,5]  
> [2,3,4,5]

* `!!` (operador, utiliza notação infixa): seleciona o n-ésimo elemento de uma lista (iniciando em 0)
> Prelude> [1,2,3,4,5] !! 2  
> 3

* `drop`: remove os *n* primeiros elementos de uma lista
> Prelude> drop 2 [1,2,3,4,5]  
> [3,4,5]

* `take`: Seleciona os *n* primeiros elementos de uma lista
> Prelude> take 2 [1,2,3,4,5]  
> [1,2]

* `length`: Calcula a quantidade de elementos em uma lista
> Prelude> length [1,2,3,4,5]  
> 5

* `sum`: Calcula a soma de todos os elementos de uma lista de números
> Prelude> sum [1,2,3,4,5]  
> 15

* `product`: Calcula o produto de todos os elementos de uma lista de números
> Prelude> product [1,2,3,4,5]  
> 120

* `++` (operador, utiliza notação infixa): Concatena duas listas
> Prelude> [1,2,3,4,5] ++ [6,7,8,9,10]  
> [1,2,3,4,5,6,7,8,9,10]

* `reverse`: Inverte uma lista
> Prelude> reverse [1,2,3,4,5]  
> [5,4,3,2,1]


## Aplicação de funções

Como você já deve ter percebido, a aplicação das funções por padrão utiliza a forma prefixa, o que significa que você deve escrever o nome da função, seguido por um espaço e os argumentos da função, separados por espaços. Uma coisa importante para lembrar, a **aplicação de funções tem a precedência 10**, ou seja, maior do que qualquer operador. Isso pode criar algumas confusões mentais. Por exemplo, considere a função abaixo:

```haskell
menor x y = x <= y then x else y
```

O resultado da aplicação `menor 5 1 + 6` é 7, pois a ordem de execução é primeiro `menor 5 1`, cujo resultado é 1, e só depois a soma `1 + 6` é executada.


## Definindo suas funções

Como visto no último exemplo, é possível, além de usar as funções definidas no Prelude (o módulo básico do Haskell), definir nossas funções em arquivos de código fonte e carregá-los. Para fazer isso, é necessário fazer salvar seu código em arquivos com a extensão .hs e carregá-los no Hugs utilizando  o comando `:load`:

>Prelude> :load teste.hs 


A qualquer momento, se houver alguma alteração no código fonte do arquivo carregado, você pode recarregá-lo usando o comando `:reload`. Comandos mais usados do Hugs:

| Comando      | Significado   |
|--------------|---------------|
|*:load nome*  |carregar script *nome*|
|*:reload*     |recarregar script atual|
|*:type expre* |mostra o tipo de *expre*|
|*:?*          |mostra todos os comandos|
|*:quit*       |sair do Hugs |

### Regras para nomes

Nomes de funções devem iniciar com uma letra minúscula, seguida ou não por uma combinação de letras (maiúsculas ou minúsculas), dígitos, e *underscores*. Pode ainda ser finalizado (apenas finalizado) com um caractere de aspa simples. 

As palavras abaixo possuem algum significado na linguagem, e por isso são reservadas, não podendo ser usadas para nomear funções:

> **case class data default deriving do else**  
> **if import in infix infixl infixr instance**  
> **let module newtype of then type where**  

Por converção, os parâmetros que representem listas nas funções terminam com `s`. Por exemplo, uma lista de números geralmente é chamada de `ns`, uma lista arbitrária de valores pode ser chamadas `xs`.

### A regra do layout

Em um script, cada definição deve iniciar exatamente na coluna. Isto permite que definições relacionadas a um bloco sejam identificadas eja um exemplo abaixo:

```haskell
a = b + c
    where
        b = 1
        c = 2
d = a * 2
```

Importante salientar que os valores representados por `b` e `c` só são visíveis pela expressão `a = b + c`. Dizemos que `b` e `c` tem um escopo local à expressão.

A sintaxe abaixo também é permitida para dar um pouco mais de clareza, se for necessário:

```haskell
a = b + c
    where
        {b = 1;
         c = 2}
d = a * 2
```


## Comentários

Comentários iniciam com `--`. Qualquer coisa após os `--` será ignorada. Este é o comentário de linha. Haskell também possui comentários de bloco, e eles são definidos assim:

```haskell
-- Bloco comentado
{-
sinal n = if n < 0 then -1 else
    if n == 0 then 0 else 1
-}
```