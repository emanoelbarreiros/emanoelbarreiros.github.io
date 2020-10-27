---
layout: single
title:  "Implementando aceleração [parte 1 - gravidade]"
toc: true
comments: true
categories: [game]
tags: [python, pyxel]
published: true
---

Neste post vamos implementar o conceito de gravidade, muito comum é uma grande variedade de jogos.

## O conceito de gravidade

O conceito de gravidade é muito simples, ele é transferido da vida real diretamente para o nosso jogo. Precisamos fazer apenas algumas mudanças conceituais em termos de unidades, que explico melhor abaixo. De forma bastante simplificada, no mundo real, a força gravitacional exercida por um corpo de grande massa em um corpo menor cria uma aceleração em direção ao corpo de maior massa.

No mundo real, percebemos isso como "as coisas caem". Dessa forma, em um jogo que emprega esta mecânica, precisamos implementar algo de forma que esse comportamento seja reproduzido. Precisamos entender bem o que é aceleração para que a conversão do conceito no mundo real seja mais tranquila para o nosso mundo simulado. A unidade usada tradicionalmente de acordo com o sistema métrico universal é de m/s^2 (metros por segundo ao quadrado). Por exemplo, se temos a nossa gravidade terrestre de aproximadamente 10 m/s^2, isso quer dizer que a velocidade do corpo aumenta 10 m/s a cada segundo (10 metros por segundo por segundo, por isso a unidade é m/s^2). Então, de forma bastante simples, a velocidade é a taxa com que a posição do corpo muda em relação ao tempo, e a aceleração é a taxa com a qual a velocidade do corpo muda em relação ao tempo. Aceleração muda a velocidade, velocidade muda a posição.

No nosso contexto simulado precisamos definir algo parecido, então obviamente temos que ter uma posição para o corpo (um personagem controlado pelo jogador, um NPC, qualquer elemento do jogo), uma velocidade de movimento e a aceleração. Nossas unidades aqui precisam mudar, para tornar nossa vida mais fácil. Em um jogo 2D, a posição do corpo é dividida em duas variáveis, a posição no eixo X e a posição no eixo Y em pixels. Geralmente a distância do corpo em relação à origem no plano cartesiano imposto pelo seu *engine*. No caso do Pyxel, a origem fica no canto superior esquerdo da janela. Tanto a velocidade quanto a aceleração também precisam ter duas componentes, uma para o eixo X e uma para o eixo Y.

A unidade de medida mais simples que podemos adotar aqui para a velocidade é de pixels por frame. Se definirmos a velocidade de um corpo como sendo 2 pixels/frame no eixo Y, isso significa que a cada frame a posição do corpo deve mudar 2 pixels (se a velocidade se mantiver constante). No caso do Pyxel, o corpo vai se mover para baixo 2 pixels a cada frame, e um total de 60 pixels em um segundo, pois a taxa de atualização de quadros padrão do Pyxel é de 30 frames por ssegundo (fps). A aceleração, por sua vez, é a taxa de mudança da velocidade, e no nosso caso é medida em pixels/frame^2. Por exemplo, se nossa aceleração é de 2 pixels/frame^2, a velocidade do corpo vai aumentar 2 pixels/frame a cada frame.

## Código
Vamos ver um exemplo bem simples de como podemos implementar esses conceitos no código abaixo. Neste programa simples, ao clicar na tela um círculo surge no ponto do clique e fica quicando na base da janela indefinidamente.

Criamos no código abaixo a classe Bola. Ela vai ser responsável por armazenar todas as informações referente a cada bola que possamos ter ao mesmo tempo no nosso jogo. Lembre do conceito de classes e como elas servem para definirmos o esqueleto de um objeto. Objetos são entidades com vida no nosso programa, ou seja, cada Bola criada vai ter seus próprios valores para `x`, `y`, `raio` e `velocidade`. Neste exemplo, nossas bolas podem mover-se apenas no eixo Y, por isso temos apenas uma variável para armazenar o valor da velocidade.

```python
class Bola:
    def __init__(self, x, y):
        self.x = x
        self.y = y
        self.raio = 10
        self.velocidade = 0
```

A classe `Jogo` (abaixo) implementa a lógica do nosso programa. Na linha 4, dentro da função `__init__` definimos a nossa gravidade como sendo 1 pixel/frame^2. Na linha 3 definimos a nossa lista de bolas, pois vamos querer ter várias bolas ao mesmo tempo no programa. A linha 6 informa ao Pyxel que queremos ver o ponteiro do mouse.

Já na função `update`, linha 10, nós verificamos se o botão esquerdo do mouse foi liberado (essa função retorna `True` apenas no frame seguinte ao momento em que o botão é liberado). Neste momento é criada uma nova Bola (linha 11) na posição onde se encontra o mouse e esta bola é adicionada à nossa lista de bolas (linha 12).

Entre as linhas 14-19 temos um laço `for` que vai iterar por todas as bolas que temos na lista. Para cada bola atualizamos a sua velocidade baseada na aceleração (linha 15), e atualizamos sua posição baseado na velocidade (linha 16). Nas linhas 17-19 verificamos se ela está tentando sair da tela pela borda inferior (linha 17). Se verificarmos que este é o caso, atualizamos sua posição para que ela não pareça estar fora dos limites da tela (linha 18) e invertemos sua velocidade para que ela agora passe a subir, com a velocidade negativa (linha 19), dando assim a impressão de que ela bateu no chão.

Nas linhas 21-24 temos a função `draw`, que também vai ser executada a cada frame. A linha 22 pinta toda a tela com a cor 1 e as linhas 23 e 24 são responsáveis por pintar as bolas, uma a uma, com a cor 8.

```python
class Jogo:
    def __init__(self):
        pyxel.init(200, 200)
        self.gravidade = 1
        self.bolas = []
        pyxel.mouse(True)
        pyxel.run(self.update, self.draw)

    def update(self):
        if pyxel.btnr(pyxel.MOUSE_LEFT_BUTTON):
            bola = Bola(pyxel.mouse_x, pyxel.mouse_y)
            self.bolas.append(bola)

        for bola in self.bolas:
            bola.velocidade += self.gravidade
            bola.y += bola.velocidade
            if bola.y + bola.raio >= pyxel.height:
                bola.y = pyxel.height - bola.raio - 1
                bola.velocidade = -bola.velocidade

    def draw(self):
        pyxel.cls(1)
        for bola in self.bolas:
            pyxel.circ(bola.x, bola.y, bola.raio, 8)
```


## Resumo

Neste post vimos o conceito físico super simplificado de velocidade e aceleração, bem como implementá-los de maneira muito simples em nossos joguinhos. Esse exemplo pode ser enriquecido com mais funcionalidades a depender do jogo.

Este é o código completo para este exemplo:

```python
import pyxel

class Bola:
    def __init__(self, x, y):
        self.x = x
        self.y = y
        self.raio = 10
        self.velocidade = 0

class Jogo:
    def __init__(self):
        pyxel.init(200, 200)
        self.gravidade = 1
        self.bolas = []
        pyxel.mouse(True)
        pyxel.run(self.update, self.draw)

    def update(self):
        if pyxel.btnr(pyxel.MOUSE_LEFT_BUTTON):
            bola = Bola(pyxel.mouse_x, pyxel.mouse_y)
            self.bolas.append(bola)

        for bola in self.bolas:
            bola.velocidade += self.gravidade
            bola.y += bola.velocidade
            if bola.y + bola.raio >= pyxel.height:
                bola.y = pyxel.height - bola.raio - 1
                bola.velocidade = -bola.velocidade

    def draw(self):
        pyxel.cls(1)
        for bola in self.bolas:
            pyxel.circ(bola.x, bola.y, bola.raio, 8)

Jogo()
```
