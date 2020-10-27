---
layout: single
title:  "Implementando aceleração [parte 2 - estilingue]"
toc: true
comments: true
categories: [game]
tags: [python, pyxel]
published: true
---

Neste post vamos incrementar um pouco mais implementação de gravidade adicionando uma forma de definir a velocidade inicial de um corpo. Neste exemplo implementar um estilingue (de onde eu venho os maloqueiros chamam de badoque). Este programa é uma evolução do programa desenvolvido no [post sobre gravidade]({% post_url 2020-10-26-gravidade %}).

## Iniciando

Neste programa a bola vai poder viajar em qualquer direção, não apenas cair, uma vez que vamos ser capazes de dar uma velocidade inicial para a bola. A direção e a velocidade inicial do corpo serão definidas pelo ponto onde iniciamos o movimento.

O movimento inicia quando o usuário clica, mantém o clique e arrasta. Nesse momento o programa registra o ponto do clique e mostra ao usuário uma linha que liga o ponto atual do mouse e o ponto onde iniciou-se o movimento. Dê uma olhada abaixo para entender melhor o que vamos alcançar com este post.

![Estilingue funcionando](/assets/estilingue.gif)

## Modificando a classe Bola

Agora que nossa bolinha vai poder se mover em qualquer direção precisamos adicionar a velocidade X também:

```python
class Bola:
    def __init__(self, x, y, vel_x, vel_y):
        self.x = x
        self.y = y
        self.raio = 10
        self.velocidade_x = vel_x
        self.velocidade_y = vel_y
```

## Coeficiente elástico

Você deve ter percebido também que dessa vez a bola não fica quicando indefinidamente, depois de um tempo ela para. No caso das colisões, introduzimos uma variável responsável por controlar quanto da velocidade será mantida após uma colisão. Para colisões que tentam simular o mundo real sempre temos uma perda de energia, então o valor para essa variável (`coeficiente_restituicao`) será menor que 1. Veja como ficou o método `__init__`:

```python
def __init__(self):
    pyxel.init(200, 200)
    pyxel.mouse(True)
    self.bolas = []
    self.ponto_clique = None
    self.gravidade = 1
    self.coeficiente_restituicao = 0.9
    pyxel.run(self.update, self.draw)
```

## Tratando a interação com o usuário

Precisamos agora tratar a interação com o usuário. Como mencionado anteriormente, o usuário clica na tela, mantém o botão esquerdo do mouse pressionado e arrasta o mouse. Quando o botão é liberado a bola é lançada na direção apontada pela linha exibida na tela. Para isso foi criada uma função chamada `tratar_interacao_usuario`.

```python
def tratar_interacao_usuario(self):
    if pyxel.btnp(pyxel.MOUSE_LEFT_BUTTON):
        self.ponto_clique = (pyxel.mouse_x, pyxel.mouse_y)

    if pyxel.btnr(pyxel.MOUSE_LEFT_BUTTON):
        delta_x = self.ponto_clique[0] - pyxel.mouse_x
        delta_y = self.ponto_clique[1] - pyxel.mouse_y
        bola = Bola(pyxel.mouse_x, pyxel.mouse_y, delta_x//3, delta_y//3)
        self.bolas.append(bola)
        self.ponto_clique = None    
```

A linha 2 detecta quando o botão esquerdo do mouse é pressionado. Nesse momento essa posição é registrada na variável `ponto_clique` através de uma tupla. A linha 5 por sua vez identifica quando o botão é liberado. Nesse momento calculamos a distância, nos dois eixos, entre o ponto onde iniciou-se o movimento e o ponto onde o botão foi liberado. Esses valores são armazenados em `delta_x` e `delta_y`. É criada uma nova bola na posição atual do mouse e as velocidades para os eixos X e Y são passadas também para o construto. É feita uma divisão por 3 para que o valor não seja tão alto. Depois disso a bola é adicionada à lista de bolas (linha 9) e o ponto do clique é apagado (linha 10).

## O método `update`

Vejamos agora como ficou o método `update`.

```python
def update(self):
    self.tratar_interacao_usuario()

    for bola in self.bolas:
        bola.x += bola.velocidade_x
        bola.velocidade_y += self.gravidade
        bola.y += bola.velocidade_y            

        if bola.x + bola.raio >= pyxel.width:
            bola.x = pyxel.width - bola.raio - 1
            bola.velocidade_x = int(-bola.velocidade_x * self.coeficiente_restituicao)

        if bola.x - bola.raio <= 0:
            bola.x = bola.raio + 1
            bola.velocidade_x = int(-bola.velocidade_x * self.coeficiente_restituicao)

        if bola.y + bola.raio >= pyxel.height:
            bola.y = pyxel.height - bola.raio - 1
            bola.velocidade_y = int(-bola.velocidade_y * self.coeficiente_restituicao)            

        if bola.y - bola.raio <= 0:
            bola.y = bola.raio + 1
            bola.velocidade_y = int(-bola.velocidade_y * self.coeficiente_restituicao)

        if abs(bola.y + bola.raio - pyxel.height) < 2:
            bola.velocidade_x *= 0.9
```

A primeira linha do método invoca o comportamento que acabamos de implementar para interação do usuário. O restante do método é responsável por tratar a colisão das bolas com as bordas laterais, superior e inferior da janela. Queremos dar a sensação de que a bola está colidindo com as bordas da janela.

As linhas 5-7 são bem parecidas com o exemplo da gravidade, estamos apenas aqui também atualizando a posição X de acordo com a velocidade X, o que não acontecia no exemplo anterior. Tomemos as linhas 9-11 como exemplo para o tratamento da colisão. A detecção da colisão é feita pelo cabeçalho do `if`: `if bola.x + bola.raio >= pyxel.width:`. Precisamos fazer a soma da posição X da bola com seu raio porque a coordenada controlada pelo sistema é do centro do círculo, e todos os pontos dele devem permanecer dentro da janela. Caso a expressão desse cabeçalho seja avaliada para `True` a bola está em uma posição não permitida, então precisamos faze-la retornar a uma posição válida (linha 10). Subtraímos 1 para jogar a bola 1 pixel para a esquerda, pois percebi que se isso não for feito a cada colisão o sistema perdia ainda mais energia do que foi especificado pelo coeficiente de restituição. Na linha 11 fazemos a inversão da velocidade e a multiplicamos pelo coeficiente de restituição, o que faz com que a bole perca 10% de sua velocidade a cada colisão.

As linhas 25-26 também são interessantes, elas adicionam um efeito de atrito entre as bolas e o piso. Se for detectado que a bola está encostando no piso, ela também perde 10% de sua velocidade no eixo X (movimento lateral).

## O método `draw`

Este método `draw` traz as modificações necessárias para produzir o efeito desejado.

```python
def draw(self):
    pyxel.cls(1)
    if self.ponto_clique is not None:
        pyxel.line(self.ponto_clique[0], self.ponto_clique[1], pyxel.mouse_x, pyxel.mouse_y, 7)
        pyxel.circ(pyxel.mouse_x, pyxel.mouse_y, 10, 10)
        pyxel.circb(pyxel.mouse_x, pyxel.mouse_y, 10, 9)

    for bola in self.bolas:
        pyxel.circ(bola.x, bola.y, bola.raio, 10)
        pyxel.circb(bola.x, bola.y, bola.raio, 9)
```
O `if` da linha 3 só terá seu corpo executado caso o usuário esteja manipulando uma bola, sinalizado pela variável `ponto_clique` diferente de `None`. Quando isso acontecer desenhamos uma linha entre o ponto do clique e a posição atual do mouse. Depois disso desenhamos um círculo amarelo e um contorno laranja na posição do mouse.

Linhas 8-10 pintam as outras bolas que já existem. Da mesma forma que anteriormente, desenham um círculo amarelo e um contorno laranja sobre a posição onde se encontra cada bola.

## Resumo

Neste post exploramos uma implementação simples do efeito de um estilingue. Incrementamos um pouco o que foi implementado no [post sobre gravidade]({% post_url 2020-10-26-gravidade %}).

Este é o resultado completo da implementação:
```python
import pyxel

class Bola:
    def __init__(self, x, y, vel_x, vel_y):
        self.x = x
        self.y = y
        self.raio = 10
        self.velocidade_x = vel_x
        self.velocidade_y = vel_y

class Jogo:
    def __init__(self):
        pyxel.init(200, 200)
        pyxel.mouse(True)
        self.bolas = []
        self.ponto_clique = None
        self.gravidade = 1
        self.coeficiente_restituicao = 0.9
        pyxel.run(self.update, self.draw)     

    def tratar_interacao_usuario(self):
        if pyxel.btnp(pyxel.MOUSE_LEFT_BUTTON):
            self.ponto_clique = (pyxel.mouse_x, pyxel.mouse_y)

        if pyxel.btnr(pyxel.MOUSE_LEFT_BUTTON):
            delta_x = self.ponto_clique[0] - pyxel.mouse_x
            delta_y = self.ponto_clique[1] - pyxel.mouse_y
            bola = Bola(pyxel.mouse_x, pyxel.mouse_y, delta_x//3, delta_y//3)
            self.bolas.append(bola)
            self.ponto_clique = None   

    def update(self):
        self.tratar_interacao_usuario()

        for bola in self.bolas:
            bola.x += bola.velocidade_x
            bola.velocidade_y += self.gravidade
            bola.y += bola.velocidade_y            

            if bola.x + bola.raio >= pyxel.width:
                bola.x = pyxel.width - bola.raio - 1
                bola.velocidade_x = int(-bola.velocidade_x * self.coeficiente_restituicao)

            if bola.x - bola.raio <= 0:
                bola.x = bola.raio + 1
                bola.velocidade_x = int(-bola.velocidade_x * self.coeficiente_restituicao)

            if bola.y + bola.raio >= pyxel.height:
                bola.y = pyxel.height - bola.raio - 1
                bola.velocidade_y = int(-bola.velocidade_y * self.coeficiente_restituicao)            

            if bola.y - bola.raio <= 0:
                bola.y = bola.raio + 1
                bola.velocidade_y = int(-bola.velocidade_y * self.coeficiente_restituicao)

            if abs(bola.y + bola.raio - pyxel.height) < 2:
                bola.velocidade_x *= 0.9


    def draw(self):
        pyxel.cls(1)
        if self.ponto_clique is not None:
            pyxel.line(self.ponto_clique[0], self.ponto_clique[1], pyxel.mouse_x, pyxel.mouse_y, 7)
            pyxel.circ(pyxel.mouse_x, pyxel.mouse_y, 10, 10)
            pyxel.circb(pyxel.mouse_x, pyxel.mouse_y, 10, 9)

        for bola in self.bolas:
            pyxel.circ(bola.x, bola.y, bola.raio, 10)
            pyxel.circb(bola.x, bola.y, bola.raio, 9)

Jogo()
```
