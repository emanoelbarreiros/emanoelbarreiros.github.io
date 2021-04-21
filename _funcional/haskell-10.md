---
layout: single
title:  "[11] Testes com Hspec"
toc: true
lesson: 11
---

Neste post vamos discutir um pouco sobre a implementação de testes unitários em Haskell, utilizando o [Hspec](https://hackage.haskell.org/package/hspec) e [QuickCheck](https://hackage.haskell.org/package/QuickCheck).

O código comentado neste post pode ser encontrado no [Github](https://github.com/emanoelbarreiros/validarcpf).

Um excelente tutorial sobre o QuickCheck pode ser encontrado [aqui](https://begriffs.com/posts/2017-01-14-design-use-quickcheck.html).


## TDD (Test Driven Development)
TDD, do Inglês Test Driven Development é uma técnica de programação focada no desenvolvimento iterativo de software, promovendo a criação de testes antes do desenvolvimento das funcionalidades do sistema. De acordo com a técnica, o desenvolvimento é realizado em ciclos de evolução, iniciando com a escrita de testes que determinam como o código a ser desenvolvido deve se comportar. Uma vez com os testes prontos, o trabalho do desenvolvedor é escrever o código da funcionalidade, que deve ser capaz de passar em todos os testes especificados.

A abordagem orientada a testes dá uma ênfase maior à fase de projeto, faz com que os desenvolvedores pensem com bastante detalhe no que o código prestes a ser escrito deve ser capaz de fazer. Além disso, a existência de testes unitários com uma boa cobertura do código fonte aumenta a confiança dos desenvolvedores quanto a evolução do software, pois após qualquer manutenção, seja ela corretiva ou evolutiva, o sistema deve ser capaz de passar nos testes pré-existentes.

Este artigo fala bem sobre o TDD: [clicar para ir ao site devmedia](https://www.devmedia.com.br/test-driven-development-tdd-simples-e-pratico/18533#:~:text=TDD%20%C3%A9%20o%20Desenvolvimento%20Orientado,do%20nosso%20c%C3%B3digo%20de%20produ%C3%A7%C3%A3o!)

## Testes em Haskell
Haskell possui alguns frameworks apropriados ao desenvolvimento de testes unitários. Os mais conhecidos são o [Hspec](https://hspec.github.io/), [QuickCheck](http://www.cse.chalmers.se/~rjmh/QuickCheck/manual.html) e [HUnit](https://github.com/hspec/HUnit#readme).

O Hspec é capaz de integrar tanto com o HUnit quanto com o QuickCheck, então vamos escolhe-lo para desenvolver nossos testes. Ele implementa o que se chama de EDSL, ou [Embedded Domain-Specific Language](https://wiki.haskell.org/Embedded_domain_specific_language), e permite que escrevamos os testes de maneira bastante natural. Um [fluxo de desenvolvimento alinhado com o que foi falado sobre TDD](https://hspec.github.io/writing-specs.html) pode ser encontrado na documentação do Hspec.

### Escrevendo testes com o Hspec
Através da EDSL implementada pelo Hspec utilizamos as construções `describe` e `it` para organizar os *specs*. Um *spec* é composto por um ou mais `describe`s, podendo inclusive ser aninhados. Observe o exemplo abaixo, extraído da própria documentação do Hspec:
```haskell
-- Arquivo test/ValidadorSpec.hs

module ValidadorSpec (spec) where

-- utilizaremos todos esses imports mais adiante
import Test.Hspec
import Test.Hspec.QuickCheck
import Test.QuickCheck
import qualified Data.Text as Text
import qualified Data.Text.IO as Text
import Data.Text (Text,unpack)
import Control.Monad
import Text.Printf
import Validador

main :: IO ()
main = hspec $ do
  describe "Prelude" $ do
    describe "read" $ do
      it "can parse integers" $ do
        read "10" `shouldBe` (10 :: Int)

    describe "head" $ do
      it "returns the first element of a list" $ do
        head [23 ..] `shouldBe` 23
```

As construções `describe` e `it` de fato servem para organizar, tanto no código fonte, quanto na saída do resultado dos testes, que está sendo testado. As linhas 2-6 especificam o teste da função `read`. Perceba como a leitura do teste é muito simples, facilitando o entendimento do objetivo daquele teste.

A cláusula `describe` informa qual função está sendo testada. A cláusula `it` dá uma descrição do que está sendo testado, e logo após temos as [*expectations*](https://hspec.github.io/expectations.html), que são as implementações dos comportamentos esperados. Geralmente elas iniciam com *should* e nada mais são do que funções que fazem a asserção dos valores retornados pelas funções invocadas.

A *expectation* mais comum é a `shouldBe` que vai fazer a comparação por igualdade. Em outras palavras, ela vai verificar se o valor retornado pela execução da função (lado esquerdo) é igual ao valor especificado do lado direito.


## Exemplo: validador de CPF

Implementaremos um validador de CPF simples para mostrar como funciona o *Hspec*. Para criação do projeto, iniciamos com o comando *stack*:

> \> stack new validadorcpf

### Configurando a descoberta de casos de teste

Após a criação do projeto você terá um diretório *test* na raiz. Por padrão será criado um arquivo chamado *Spec.hs* e ele estará configurado no arquivo *package.yaml* como o arquivo principal da sua suíte de testes. A maneira mais fácil de incluir novos arquivos com especificação de testes é iniciar modificando o conteúdo do arquivo *Spec.hs*, alterando o seu conteúdo para que ele tenha uma única linha, como mostrado abaixo:

```haskell
{-# OPTIONS_GHC -F -pgmF hspec-discover #-}
```
Você precisará adicionar algumas dependências no *package.yaml*. Adicione as dependências ao *hspec* e *QuickCheck* na seção *depedencies* sob a seção *tests*. Aproveite e adicione logo a dependência a *text*. Utilizaremos mais tarde.


## Criando nossos casos de testes

Vamos agora iniciar a criação do nosso Spec e logo depois falaremos do código que implementa a validação do CPF.

### Primeira versão do spec

O algoritmo de validação de CPF é simples, mas requer o cálculo de alguns passos. O primeiro deles é validar o primeiro dígito, para só depois validar o segundo dígito verificador. Veja detalhes do algoritmo de validação [aqui](https://dicasdeprogramacao.com.br/algoritmo-para-validar-cpf/).

Prevejo que teremos uma função para validação do primeiro dígito e uma para o segundo dígito. Podemos então ter uma terceira função para decidir se o número de CPF inteiro é válido a partir da invocação dessas duas outras funções.

Implementaremos toda a lógica do nosso validador em apenas um arquivo. Na pasta *src* devemos criar um novo arquivo fonte, chamado *Validador.hs*, com o seguine conteúdo:
```haskell
-- Arquivo src/Validador.hs
module Validador where

validarPrimeiroDigito :: String -> Bool
validarPrimeiroDigito s = undefined

validarSegundoDigito :: String -> Bool
validarSegundoDigito s = undefined

valido :: String -> Bool
valido s = validarPrimeiroDigito s && validarSegundoDigito s
```

Definimos nossas funções retornando o valor `undefined`. Ele é útil em situações como essa, que ainda não implementamos o código da função mas queremos que o projeto seja compilado. Quando o valor é avaliado em tempo de execução uma exceção é lançada, mas graças à *laziness* do Haskell, isso só ocorre quando esse valor é avaliado em tempo de execução. Existe algo análogo para os testes. Caso deseje, você pode deixar as especificações de testes como pendentes até que você tenha a oportunidade de implementá-los. Veja mais [aqui](https://hspec.github.io/writing-specs.html#using-pending-and-pendingwith).

Ao criar novos arquivos de teste, devemos nomeá-los como *NOME_DO_MODULOSpec.hs*, onde *NOME_DO_MODULO* é o nome do módulo ao qual este arquivo de teste se destina. No nosso caso, criaremos um módulo de testes de nome *ValidadorSpec.hs*. Todos os módulos de teste devem ser localizados dentro do diretório *tests* ou em algum subdiretório. Caso contrário, a descoberta automática dos módulos de teste não funcionará como esperado.

Criamos o arquivo com o conteúdo abaixo:
```haskell
-- Arquivo test/ValidadorSpec.hs
module ValidadorSpec (spec) where

import Test.Hspec
import Test.Hspec.QuickCheck
import Test.QuickCheck
import qualified Data.Text as Text
import qualified Data.Text.IO as Text
import Data.Text (Text,unpack)
import Control.Monad
import Text.Printf
import Validador

spec :: Spec
spec = do
    describe "validarPrimeiroDigito" $ do
      context "quando passado um CPF com o primeiro digito valido" $ do
        it "valida o CPF" $ do
          validarPrimeiroDigito "51171591870" `shouldBe` True
      context "quando passado um CPF com o primeiro digito invalido" $ do
        it "invalida o CPF" $ do
          validarPrimeiroDigito "51171591800" `shouldBe` False

    describe "validarSegundoDigito" $ do
      context "quando passado um CPF com o segundo digito valido" $ do
        it "valida o CPF" $ do
          validarSegundoDigito "51171591870" `shouldBe` True
      context "quando passado um CPF com o segundo digito invalido" $ do
        it "invalida o CPF" $ do
          validarSegundoDigito "51171591879" `shouldBe` False

    describe "valido" $ do
      context "quando passado um CPF valido" $ do
        it "valida o CPF" $ do
          valido "51171591870" `shouldBe` True
      context "quando passado um CPF com primeiro digito invalido e segundo valido" $ do
        it "invalida o CPF" $ do
          valido "51171591890" `shouldBe` False
      context "quando passado um CPF com primeiro digito valido e segundo invalido" $ do
        it "invalida o CPF" $ do
          valido "51171591879" `shouldBe` False
      context "quando passado um CPF com ambos os digitos invalidos" $ do
        it "invalida o CPF" $ do
          valido "51171591899" `shouldBe` False
```

Vamos detalhar o conteúdo deste arquivo. Devemos obrigatoriamente expor uma função chamada *spec*, do tipo *Spec* (linha 1). Precisamos também incluir as importações aos módulos do Hspec e Validador (linhas 3 e 4). As linhas 8-14 implementam o primeiro conjunto de testes. A linha 8 informa que o teste diz respeito à função *validarPrimeiroDigito*, linha 9 descreve o contexto e a linha 10 detalha o resultado esperado. A linha 11 implementa de fato a asserção, determinando que a chamada à função `validarPrimeiroDigito` que recebe o valor `"51171591870"` *deveria ter o retorno* `True`. A asserção é realizada pela função `shouldBe`, usada na forma infixa para melhorar a legibilidade. Observe que esse *idioma* se repete para todos os outros casos de teste.

Assim que rodarmos a suíte de testes seremos presenteados com aquele erro de execução que eu comentei anteriormente. Por sinal, faça assim para rodar os testes, dentro do diretório do seu projeto:

> \> stack test

Você deve ter observado que ao executar os testes receberemos uma exceção chamada *ErrorCall* pois nosssas funções ainda não foram definidas completamente. Vamos ver isso agora.


### Adicionando código do validador

Receberemos o CPF como uma String, mas precisaremos converter cada caractere para sua representação inteira de forma que possamos implementar o [algoritmo de validação](https://dicasdeprogramacao.com.br/algoritmo-para-validar-cpf/).

Uma maneira simples e fazer isso é implementar uma função que receba uma String e devolva uma lista de inteiros. Podemos utilizar a função `digitToInt` disponível no módulo *Data.Char* e fazer assim:
```haskell
-- Arquivo src/Validador.hs
import Data.Char --adicione essa linha ao topo do seu arquivo

digitos :: String -> [Int]
digitos = map digitToInt
```

Podemos agora utilizar essa função no nosso algoritmo de validação. Eu o implementei dessa forma:
```haskell
-- Arquivo src/Validador.hs
validarPrimeiroDigito :: String -> Bool
validarPrimeiroDigito s = sum (zipWith (*) (digitos (take 9 s)) [10, 9..2]) * 10 `mod` 11 == digitToInt (s !! 9)

validarSegundoDigito :: String -> Bool
validarSegundoDigito s = sum (zipWith (*) (digitos (take 10 s)) [11, 10..2]) * 10 `mod` 11 == digitToInt (s !! 10
```
A função `zipWith` é capaz de fazer o zip de duas listas e depois aplica uma função semelhante ao `map`.A função `valido` já está implementada, não precisamos tocar nela, mas agora temos uma nova função para testar também, a função `digitos`. Vamos adicionar um caso de teste no arquivo *ValidadorSpec.hs* para isso.
```haskell
-- Arquivo test/ValidadorSpec.hs
describe "digitos" $ do
  context "quando recebe uma string" $ do
    it "retorna a representação [Int] da string" $ do
      digitos "123456" `shouldBe` [1,2,3,4,5,6]
```

Excelente. Temos testes funcionando como esperávamos, mas visualizo uma oportunidade de refatorarmos o código, especificamente as funções `validarPrimeiroDigito` e `validarSegundoDigito`. Veja que elas em praticamente o mesmo código. Vamos fazer agora uma nova versão, extraindo o comportamento comum e utilizando esse novo comportamento para as nossas funções de validação. Criarei uma função `validarDigito` que servirá para validar qualquer um dos dígitos verificadores e atualizarei as funções `validarPrimeiroDigito` e `validarSegundoDigito`.
```haskell
validarDigito :: String -> Int -> Bool 
validarDigito s d
    | d >= 10 && d <= 11 = sum (zipWith (*) (digitos (take (d-1) s)) [d, (d-1)..2]) * 10 `mod` 11 == digitToInt (s !! (d-1))
    | otherwise          = False

validarPrimeiroDigito :: String -> Bool
validarPrimeiroDigito s = validarDigito s 10

validarSegundoDigito :: String -> Bool
validarSegundoDigito s = validarDigito s 11
```

Bem melhor, pois se precisarmos corrigir algum bug no algoritmo de validação faremos isso em apenas um lugar. Rodando os testes vemos que tudo ainda continua funcionando. Vamos tentar incrementar um pouco os testes agora. Vamos adicionar mais um teste para ver se nossa função funciona como gostaríamos, rejeitando validações de quaisquer dígitos com exceção do dígito 10 e 11. Para isso precisamos utilizar a integração com o QuickCheck.


## Incrementando os testes

O QuickCheck permite que nós geremos testes aleatórios para avaliar nossas funções. A forma como o QuickCheck alcança esse objetivo é através da crição, por parte do programador, de propriedades, que por sua vez são verificadas pelo QuickCheck. Leia um pouco sobre o QuickCheck [aqui](http://www.cse.chalmers.se/~rjmh/QuickCheck/manual.html). Vamos utilizar a [integração do Hspec com o QuickCheck](https://hspec.github.io/quickcheck.html).

Adicionaremos três novos casos de teste para a função `validarDigito`, confirmando que a função falha para quaisquer dígitos diferentes de 10 e 11. Iniciamos com a validação do décimo e décimo-primeiro dígitos, que são validados corretamente. Depois disso criamos uma propriedade e passamos uma função que retorna um Bool com o resultado da avaliação.
```haskell
-- Arquivo ValidadorSpec.hs

describe "validarDigito" $ do
  context "quando solicita validacao do 10o digito" $ do
    it "valida o digito" $ do
      validarDigito "51171591870" 10 `shouldBe` True
  context "quando solicita validacao do 11o digito" $ do
    it "valida o digito" $ do
      validarDigito "63683260505" 11 `shouldBe` True
  context "quando solicita um digito invalido" $ do
    prop "invalida o CPF" $ do
      \x -> x /= 10 && x /= 11 ==> validarDigito "51171591899" x `shouldBe` False
```

Na linha 12, o código que vemos antes do operador entre o operador `->` e o operador `==>` é o que se chama de [condição](http://www.cse.chalmers.se/~rjmh/QuickCheck/manual_body.html#6). O código `x /= 10 && x /= 11` diz ao QuickCheck que só devem ser gerados valores diferentes de 10 e 11, ou seja, valores que satisfazem essa condição são passadas para a validação. Por padrão o QuickCheck gera 100 valores aleatórios e confronta com a nossa função. Tente executar o teste e veja o resultado.

## Usando uma massa de dados

Agora que nossas funções estão com testes mais básicos implementados, podemos tentar usar uma massa de testes para ver se tudo ainda funciona. Temos um arquivo chamado [exemplos.txt](https://github.com/emanoelbarreiros/validarcpf/blob/master/test/exemplos.txt) no mesmo diretório onde se encontra o arquivo *ValidadorSpec.hs*, ele contém 500 CPFs válidos, gerados por um serviço que encontrei na web. Criamos agora uma função que é capaz de ler esses CPFs e produzir uma lista com eles, para que possamos passar para a nossa função que fará a validação da massa. Adicione essa função ao arquivo *ValidadorSpec.hs*. 
```haskell
-- Arquivo ValidadorSpec.hs

lerExemplos :: IO [Text]
lerExemplos = Text.lines <$> Text.readFile "test/exemplos.txt"
```

Adicione agora esse trecho à função *spec*:
```haskell
-- Arquivo ValidadorSpec.hs

exemplos <- runIO lerExemplos
  forM_ exemplos $ \cpf ->
    describe "valido" $ do
      context "quando alimentado com exemplos válidos" $ do
        it (printf "exemplo %s" cpf) $ do
          valido (unpack cpf) `shouldBe` True
```

Na linha 1 utilizamos a função `runIO`, que é capaz de executar uma função que faz IO, nossa função `lerExemplos`. Temos agora nossa lista de CPFs em `exemplos`. A linha 2 utiliza a função `forM_`, que basicamente executa um map na lista `exemplos`, utilizando a função lambda que informamos depois do operador `->`. A função `forM_` é capaz de executar uma operação monádica (a descrição do nosso teste) sobre a lista e ignora o resultado (observe o caractere underscore no fim do nome da função). Leia um pouco mais sobre a função `forM_` [aqui](https://hackage.haskell.org/package/base-4.15.0.0/docs/Control-Monad.html#v:forM_).

Perceba que daí pra baixo é igual ao que já tínhamos anteriormente, com uma única diferença, o texto exibido pela instrução `it` agora é diferente para cada exemplo que temos em nossa lista. Tente executar agora os testes e veja o que acontece.

Nossa! Temos muitos exemplos que não estão passando nos nossos testes. Obervando o algoritmo de validação do CPF, podemos perceber que faltou implementar um detalhe. É possível que ao calcularmos o valor do dígito verificador o valor encontrado seja 10. Quando isso acontecer devemos considerar que o dígito verificador deva ter o valor 0.

Vamos corrigir esse bug. Altere o código da função `validarDigito` e crie uma nova função chamada `normalizar`, que será responsável por fazer a *correção* do dígito verificador encontrado pelo algoritmo de validação.
```haskell
-- Arquivo Validador.hs

validarDigito :: String -> Int -> Bool 
validarDigito s d
  | d >= 10 && d <= 11 = normalizar (sum (zipWith (*) (digitos (take (d-1) s)) [d, (d-1)..2]) * 10 `mod` 11) == digitToInt (s !! (d-1))
  | otherwise          = False

normalizar :: Int -> Int
normalizar = flip mod 10
```

Agora, antes de comparar o valor calculado pelo algoritmo de validação, passamos esse valor pela função `normalizar`, que só faz obter o resto da divisão desse valor por 10. Isso resolve nosso problema. Faça um exercício mental e tente descobrir porque :)

Explicando um pouco a linha 7, utilizamos a função `flip`. Ela é bem útil, pois ela recebe uma função e os dois argumentos que essa função deve receber, mas na ordem inversa. A função `flip` então aplica esses valores na ordem correta. É o que precisamos para poder usar a função `mod` da forma correta, pois desejamos utilizar o *currying* para definir a função normalizar. Em outras palavras, deixamos a `flip` parcialmente aplicada, fornecemos apenas a função `mod` e seu primeiro parâmetro. O segundo parâmetro será passado quando a função `normalizar` for invocada. Agora sim, rodamos os casos de teste e temos todos os testes passando como gostaríamos.

## Conclusão 
Neste post exploramos apenas um pouco do que é possível fazer com os módulos de testes Hspec e QuickCheck para a linguagem Haskell. São ferramentas muito poderosas, mas para que sejam usadas da forma correta o programador precisa ter a cultura do desenvolvimento com testes. Essa é uma habilidade muito valorizada no mercado de trabalho, e sugiro que você sempre tente desenvolver com testes em mente.

O código completo destes exemplos pode ser encontrado neste repositório no Github: [https://github.com/emanoelbarreiros/validarcpf](https://github.com/emanoelbarreiros/validarcpf).