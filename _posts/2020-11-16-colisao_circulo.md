---
layout: single
title:  "Colisões entre círculos"
toc: true
comments: true
categories: [game]
tags: [python, pyxel]
published: true
---

Neste post vamos investigar o conceito de colisões. Para visualizar o que vamos implementar, utilizarei o Pyxel.

![Colisão entre círculos](/assets/colisao_circ-circ.gif)

## Colisão

No contexto de jogos digitais, as regras de colisão definem os limites dos mapas, por onde os personagens podem se mover (plataformas, paredes, etc.). Se o jogo possuir o ideia de tiro, é necessário também detectar quando personagens e o terreno são atingidos pelos tiros.

Nesse sentido, o controle de colisão tem um papel muito importante dentro de um jogo. A maioria dos engines possuem funcionalidades especificas para o tratamento de colisões, facilitando a vida do programador. No caso do Pyxel não temos tais facilidades, temos que implementar todo o código responsável por detectar as colisões entre os elementos do jogo.

## Colisão entre círculos

A colisão entre círculos é a mais simples, pois um círculo é uma figura uniforme, tendo uma distância única em relação ao centro, o raio. Desta forma, para detectar se dois círculos colidiram basta medir a distância entre os dois elementos para os quais desejamos saber se houve colisão e comparar com a soma de seus raios. Se essa distância é menor que a soma dos raios das duas figuras temos uma colisão. Vamos ver como podermos implementar isso usando Pyxel. Ele não trás nenhuma funcionalidade para ajudar nessa tarefa, vamos usá-lo apenas para ter algo visual para observar.

## O código

O código abaixo implementa a detecção de colisão entre dois círculos. Neste exemplo, um dos círculos está fixo e o usuário pode mover um segundo círculo com o mouse.

```python
import pyxel
import math

class Circulos:
    def __init__(self):
        pyxel.init(200, 200)
        self.x = 50
        self.y = 50
        self.raio1 = 20
        self.raio2 = 10
        self.colidiu = False
        pyxel.run(self.update, self.draw)

    def update(self):
        self.colidiu = self.detectar_colisao(self.x, self.y, self.raio1, pyxel.mouse_x, pyxel.mouse_y, self.raio2)

    def draw(self):
        pyxel.cls(6)
        pyxel.circ(self.x, self.y, self.raio1, 5)

        cor = 15
        if self.colidiu:
            cor = 8

        pyxel.circ(pyxel.mouse_x, pyxel.mouse_y, self.raio2, cor)


    def detectar_colisao(self, x1, y1, r1, x2, y2, r2):
        distancia = math.sqrt((x1 - x2)**2 + (y1 - y2)**2)
        if distancia <= r1 + r2:
            return True
        else:
            return False

Circulos()
```

No construtor da classe `Circulos` temos a definição dos atributos `x` e `y`, que compõem a coordenada do círculo estático mencionado anteriormente. O atributo `raio1` é o raio deste círculo estático, e `raio2` é o raio do círculo que você pode mover pela tela. Criamos mais um atributo para armazenar a informação e que os dois elementos estão em situação de colisão entre si. Essa não é a melhor estratégia se você tiver vários elementos para verificar a colisão, mas como temos apenas dois elementos é suficiente.

A função `draw` vai pintar o círculo menor com uma cor diferente caso eles estejam em situação de colisão. A linha 19 pinta o círculo estático. Define-se a cor 15 de antemão, e caso exista a colisão ela é alterada para 8. A linha 25 pinta o círculo com a cor correta de acordo com o estado do elemento.

A função `update` faz apenas delegar a detecção da colisão para a função `detectar_colisao`, que como o próprio nome diz, fará o cálculo da distância entre os dois centros dos círculos. A função precisa saber onde está o primeiro círculo (`self.x` e `self.y`) e seu raio (`self.raio1`), bem como a posição do mouse (`pyxel.mouse_x` e `pyxel.mouse_y`) e o raio do círculo móvel (`self.raio2`). O cálculo é feito utilizando-se o Teorema de Pitágoras, onde a distância entre os dois pontos é hipotenusa (linha 29). Se a distância for menor ou igual à soma dos raios, as figuras colidiram (linha 30).
