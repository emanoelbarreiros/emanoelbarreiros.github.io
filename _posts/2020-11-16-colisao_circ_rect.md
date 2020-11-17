---
layout: single
title:  "Colisões entre retângulos e círculos"
toc: true
comments: true
categories: [game]
tags: [python, pyxel]
published: true
---

Neste post vamos expandir o conceito de colisões, trabalhando agora com a colisão entre retângulos e círculos. Para visualizar o que vamos implementar, utilizarei o Pyxel.

![Colisão entre retângulo e círculo](/assets/colisao_circ_rect.gif)

Se você ainda não leu nada sobre colisões, dê uma olhada rápida aqui: [post inicial sobre colisões]({% post_url 2020-11-16-colisao_circulo %}). Se você ainda não viu o post sobre colisão entre retângulos, dá uma olhada aqui: [post inicial sobre colisões entre retângulos]({% post_url 2020-11-16-colisao_retangulos %}).

## Colisão entre círculo e retângulo

A estratégia para calcular se existe uma colisão entre círculo e retângulo é uma junção do código que vimos para detectar a colisão entre círculos e entre retângulos. Precisamos agora fazer os seguintes testes:
1. Se o círculo está à DIREITA do retângulo, checamos a borda DIREITA;
2. Se o círculo está à ESQUERDA do retângulo, checamos a borda ESQUERDA;
3. Se o círculo está ACIMA do retângulo, checamos a borda SUPERIOR;
4. Se o círculo está ABAIXO do retângulo, checamos a borda INFERIOR.

Utilizando o código abaixo, teremos no ponto `(x_ponto, y_ponto)` o ponto mais próximo do círculo que pertence ao retângulo, seja ele na borda ou em seu interior. **IMPORTANTE: o ponto `(x1, y1)`, que representa a posição do retângulo, é o seu canto superior esquerdo, e o ponto `(x2, y2)` é o centro do círculo.**

```python
x_ponto = x2
y_ponto = y2
if x2 < x1:
    x_ponto = x1
elif x2 > x1 + l1:
    x_ponto = x1 + l1

if y2 < y1:
    y_ponto = y1
elif y2 > y1 + a1:
    y_ponto = y1 + a1
```

A linha 3 só vai ser avaliada para True se o centro do círculo estiver à esquerda do retângulo, e se isso acontecer, o valor para `x_ponto` vai ser o `x` do próprio retângulo, ou seja, o `x` do seu canto superior esquerdo. Por outro lado, se isso não for verdade ele checa a outra condição (linha 5), que quer saber se o centro do círculo está à direita do retângulo. Nesse caso, o `x_ponto` vai assumir o valor do `x` do canto superior esquerdo do retângulo mais a sua largura. De forma análoga a mesma checagem é feita agora para a outra dimensão, a Y.

A boa sacada desse código (não é meu, então não estou me gabando :wink:) é que ele inicia definindo como `x_ponto` e `y_ponto` as respectivas coordenadas do centro do círculo, e só vai mudar esse valor de acordo com as checagens dos `if`s nas linhas 3-11. Caso ele não esteja nem à direita nem à esquerda, do retângulo, ele só pode estar entre a borda esquerda e a borda direita, e nesse caso o valor de `x_ponto` permanece como o valor inicial `x2` (o X do centro do círculo). Se ele não estiver nem acima nem abaixo do retângulo, ele tem que estar entre as bordas superior e inferior, seguindo uma análise análoga ao que foi feita para o eixo X. Tudo isso fica muito claro quando você vê o código executando no Pyxel.

Depois de identificar qual ponto no retângulo está mais próximo do círculo, é só calcular a distância desse ponto para o centro do círculo, e se ela for menor que o seu raio, temos uma colisão:

```python
dist_x = x2 - x_ponto
dist_y = y2 - y_ponto
distancia = math.sqrt(dist_x**2 + dist_y**2)

if distancia <= r:
    return True, x_ponto, y_ponto

return False, x_ponto, y_ponto
```

Eu aproveito para retornar `x_ponto` e `y_ponto` porque posso pintá-lo também, e aí fica muito mais intuitivo para entender o código.

## Código completo

Esse é o código completo do programa. Veja como os trechos que mostrei anteriormente se integram.

```python
import pyxel
import math

class Colisao:
    def __init__(self):
        pyxel.init(200, 200)
        self.x_rect = 30
        self.y_rect = 30
        self.largura = 40
        self.altura = 40
        self.raio = 10
        self.colidiu = False
        self.x_prox = -1
        self.y_prox = -1
        pyxel.run(self.update, self.draw)

    def update(self):
        self.colidiu, self.x_prox, self.y_prox = self.detectar_colisao(self.x_rect, self.y_rect, self.largura, self.largura, pyxel.mouse_x, pyxel.mouse_y, self.raio)

    def draw(self):
        pyxel.cls(6)
        pyxel.rect(self.x_rect, self.y_rect, self.largura, self.altura, 5)

        cor = 15
        cor_ponto = 8
        if self.colidiu:
            cor = 8
            cor_ponto = 10

        pyxel.circ(pyxel.mouse_x, pyxel.mouse_y, self.raio, cor)
        pyxel.circ(self.x_prox, self.y_prox, 1, cor_ponto)


    def detectar_colisao(self, x1, y1, l1, a1, x2, y2, r):
        x_ponto = x2
        y_ponto = y2
        if x2 < x1:
            x_ponto = x1
        elif x2 > x1 + l1:
            x_ponto = x1 + l1

        if y2 < y1:
            y_ponto = y1
        elif y2 > y1 + a1:
            y_ponto = y1 + a1

        dist_x = x2 - x_ponto
        dist_y = y2 - y_ponto
        distancia = math.sqrt(dist_x**2 + dist_y**2)

        if distancia <= r:
            return True, x_ponto, y_ponto

        return False, x_ponto, y_ponto

Colisao()
```

Eu precisei criar mais dois atributos para armazenar a coordenada do ponto do retângulo mais próximo ao círculo: `self.x_prox` e `self.y_prox`. Essas variáveis são atribuídas na linha 18, com o retorno da função `detectar_colisao`.

Nas linhas 24 a 28 estou apenas decidindo de qual cor vou mandar pintar os elementos visuais mais importantes. Inicio definindo a cor do círculo para 15 (bege, linha 24), que vai ser a cor quando o círculo não está colidindo. Também defino a cor padrão do ponto para 8 (um tom de vermelho, linha 25). Caso haja a colisão eu troco a cor do círculo para 8 (um tom de vermelho, linha 27) e a cor do ponto para 10 (um amarelo claro, linha 28), depois disso é só pintar o círculo e o ponto.
