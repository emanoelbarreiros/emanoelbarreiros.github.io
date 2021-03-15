---
layout: single
title:  "[10] Programando interatividade"
toc: true
lesson: 10
---

Nesta aula vamos começar a entender como programas em Haskell podem ser feitos interativos. Mas como programas em Haskell podem ser interativos, se interatividade necessita, por definição de interação com o mundo exterior? Lembre-se que Haskell é uma linguagem pura, isto é, por natureza ela não produz efeitos colaterais. Mas interagir com um programa quebraria essa regra. Como Haskell consegue atingir esse objetivo sem perder a sua pureza? É possível, agradeça à genialidade dos projetistas da linguagem. Vejamos como isso é feito mais adiante.


## O problema

A maioria dos programas de *antigamente* fazia o que chamamos de processamento em lotes (ou execução de rotinas *batch*), sem a interação com seus usuários, para otimizar o tempo que o computador passava processando dados. Todos sabemos que bugs só existem porque temos usuários.

Como exemplo, um sistema de folha de pagamento é um sistema *batch*, pois em algum momento uma rotina é iniciada, tem como entrada os dados dos funcionários de uma empresa e produz uma série de informações relevantes para esta empresa, relacionadas ao pagamento de seus funcionários.

Ainda hoje uma parte do nosso sistema bancário é implementado como rotinas *batch*. Por que você acha que uma transferência bancária via *DOC* só cai de um dia para o outro? A mesma coisa acontece com o pagamento de boletos, cujos pagamentos só são detectados pelo emissor pelo menos um dia após o pagamento. Tudas são rotinas *batch*.

A maioria dos programas com os quais vocês tiveram contato até agora (com exceção dos exercícios propostos no URI Online Judge) são implementados como programas *batch*, não tem interação com o usuário. Em outras palavras, eles iniciam, processam e retornam sem nenhuma interação com o usuário, seja para obter dados pelo teclado ou escrever algo na tela.

É muito mais comum hoje em dia que programas sejam interativos, assim eles podem ser mais úteis para nós. Mas como esses programas podem ser implementados se por natureza eles precisam obter mais dados e produzir efeitos colaterais enquanto o programa está executando? Solução encontrada por Haskell envolve um novo tipo e algumas operações primitivas.


## Ações

Em Haskell, um programa interativo é tomado como uma função pura, que utiliza o estado atual do mundo como entrada e produz uma versão modificada desse mundo como resultado, onde cada mudança em relação ao mundo atual é resultado de um efeito colateral executado durante a execução do programa. Tomando o tipo hipotético `Mundo`, cujos valores representam os possíveis estados do mundo, a programação interativa em Haskell pode ser vista como uma função de tipo `Mundo -> Mundo`, que abreviamos para `IO`:
```haskell
type IO = Mundo -> Mundo
```

De maneira mais genérica, é possível que além de modificar o mundo, um programa também retorne um valor. Logo, alteramos o tipo ligeiramente para refletir essa possibilidade:
```haskell
type IO a = (a, Mundo)
```

Um exemplo desta possibilidade é um programa que lê um caractere do teclado e o devolve para o usuário. Expressões do tipo `IO` são chamadas de *ações*. Por exemplo, o tipo `IO Char` é a ação que retorna um caractere, enquanto `IO ()` não retorna nada, representada pela tupla vazia.


## Ações básicas

Começamos com a ação `getChar :: IO Char`, que é capaz de obter um caractere a partir do teclado, imprime-o na tela, e retorna-o como resultado. Caso não exista um caractere no buffer para ser lido, a função aguarda até que um esteja disponível. A função irmã da *getChar* é a função `putChar :: IO ()`, que imprime um caractere na tela e não possui retorno.

A última ação básica é a `return :: a -> IO a`, que simplesmente retorna o valor passado a ela como argumento. A função `return` é responsável por criar uma ponte entre expressões puras,. sem efeitos colaterais, e ações com efeitos colaterais.


## Sequenciamento de operações

Em Haskell, uma sequencia de ações pode ser combinadas em uma única ação usando a notação `do`:
```haskell
do v1 <- a1
   v2 <- a2
   .
   .
   .
   vn <- an
   return (f v1 v2 ... vn)
```

Da maneira como imaginamos, as expressões escritas serão executadas linha a linha, de cima pra baixo. Este padrão é geralmente encontrado, com a semântica de que teremos nas linhas iniciais do bloco as ações necessárias para obter os valores necessárias para o processamento, depois executar a fução `f`, capaz de transformar os valores obtidos em um resultado a ser retornado para o usuário.

IMPORTANTE: 

* A regra do alinhamento precisa ser mantida, todas as linhas do bloco precisam iniciar na mesma coluna;
* Da mesma forma como fazemos nas compreensões de listas, expressões da forma `v1 <- a1` são chamadas de geradores, porque eles geram os valores a serem atribuídos às "variáveis";
* Se o valor produzir pelo gerador `vi <- ai` for desnecessário para o programa, a linha pode ser abreviada para `ai`, que tem o mesmo significado que `_ <- ai`. Por exemplo, uma ação que lê três caracteres, desprezando o segundo, e retorna uma tupla formada pelo primeiro e terceiro pode ser escrita assim:
```haskell
acao :: IO (Char,Char)
acao = do x <- getChar
            getChar
            y <- getChar
            return (x,y)
```

## Primitivas derivadas

Agora que temos as três primitivas básicas e a notação `do` podemos construir mais ações utilitárias. O código de *getLine* e *putStrLn* mostrado aqui, inclusive, é o mesmo encontrado na implementação do Prelude. Então vamos lá, a primeira é *getLine*:
```haskell
getLine :: IO String
getLine = do x <- getChar
             if x == '\n' then
                return []
             else
                do xs <- getLine
                   return (x:xs)
```

Observe a recursividade na definição de *getLine*. Não siga adiante sem antes entender bem a função. As próximas são a ações *putStr* e *putStrLn*, que respectivamente escrevem uma string na tela e uma string mais a quebra de linha:
```haskell
putStr :: String -> IO ()
putStr [] = return ()
putStr (x:xs) = do putChar x
                   putStr xs

putStrLn :: String -> IO ()
putStrLn xs = do putStr xs
                 putStr '\n'
```


## Colocando os conceitos em prática

Agora vamos ver os conceitos sendo aplicados na prática com um exemplo um pouco mais extenso, um simples jogo de forca. 
<!--O segundo, chamado jogo da vida, que na realidade não é um jogo, e sim uma simulação de um sisteam biológico extremamente simples. Tipo, simples mesmo, mas ainda assim fascinante, pois conseguimos comportamenteos bem complexos a partir de regras muito simples. Leia um pouco sobre o jogo da vida: [https://pt.wikipedia.org/wiki/Jogo_da_vida](https://pt.wikipedia.org/wiki/Jogo_da_vida). Eu também tenho uma implementação dele em Python, usando o [Processing](https://processing.org/): [https://github.com/emanoelbarreiros/aulasprocessing/tree/master/sketches/desafio4](https://github.com/emanoelbarreiros/aulasprocessing/tree/master/sketches/desafio4).
-->

### Forca

No início do jogo um dos jogadores define a palavra secreta, enquanto o outro deve descobrir qual é essa palavra. Para tornar as coisas um pouco mais difíceis, precisamos encontrar um jeitor de esconder os caracteres digitados pelo primeiro jogador, ou seja, não podemos usar a função *getLine* padrão.

Nosso algoritmo principal é:
```haskell
main :: IO ()
main = do hSetBuffering stdout NoBuffering
          putStrLn "Escreva uma palavra: "
          palavra <- obterLinhaSecreta
          putStrLn "Tente adivinhar: "
          jogar palavra
```

OBS: A linha `hSetBuffering stdout NoBuffering` faz com que sempre que você tente escrever na tela o valor será exibido, [caso contrário a String só aparecerá quando aparecer um '\n' no buffer](https://stackoverflow.com/questions/43830057/haskell-io-output-not-printed-until-program-exits#comment74697627_43830057).

A função `obterLinhaSecreta` agora será responsável por obter do teclado uma string, com a particularidade de não exibí-la na tela durante sua digitação:
```haskell
obterLinhaSecreta :: IO String 
obterLinhaSecreta = do x <- obterChar
                       if x == '\n' then 
                           do putChar x
                              return []
                       else 
                           do putChar '-'
                              xs <- obterLinhaSecreta
                              return (x:xs)
```

A função `obterChar`, usada pela função anterior, obtém um Char sem exibí-lo na tela:
```haskell
obterChar:: IO Char 
obterChar = do hSetEcho stdin False
               x <- getChar
               hSetEcho stdin True
               return x
```

As instruções `hSetEcho stdin False` e `hSetEcho stdin True` desligam e ligam a saída padrão do caractere digitado, respectivamente.

OBS: `hSetEcho` e `hSetBuffering` estão disponíveis no módulo System.IO. Para utilizá-lo inclua a declaração `import System.IO` no início do seu script.

Precisamos agora implementar a função `main`, que será responsável por implementar o nosso [*game loop*](https://emanoelbarreiros.github.io/game/snake/snake-1-pt/#o-game-loop). 
```haskell
jogar :: String -> IO ()
jogar palavra = do putStr "? "
                   tentativa <- getLine
    
                   if tentativa == palavra then
                       putStrLn "Muito bem!"
                   else do putStrLn (encontrar palavra tentativa)
                           jogar palavra
```

O jogador continuará tentando até que consiga descobrir a palavra secreta. Caso não acerte a palavra, exibimos apenas as letras que ele acertou, nas posições corretas:
```haskell
encontrar :: String -> String -> String 
encontrar xs ys = [if x `elem` ys then x else '-' | x <- xs]
```

O jogo está pronto, e pode agora ser jogado:
> \> ./forca  
> Escreva uma palavra:  
> -------  
> ? java  
> -a-----  
> ? python  
> h------  
> ? lua  
> -a---ll  
> ? haskell  
> Muito bem!
