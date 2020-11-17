---
layout: single
title:  "Colisões entre retângulos"
toc: true
comments: true
categories: [game]
tags: [python, pyxel]
published: true
---

Neste post vamos expandir o conceito de colisões, trabalhando agora com a colisão entre retângulos. Para visualizar o que vamos implementar, utilizarei o Pyxel.

![Colisão entre retângulos](/assets/colisao_rect_rect.gif)

Se você ainda não leu nada sobre colisões, dê uma olhada rápida aqui: [post inicial sobre colisões]({% post_url 2020-11-16-colisao_circulo %}).

## Colisão entre retângulos

A colisão entre retângulos é ligeiramente mais complexa do que a colisão entre círculos, mas nada demais. Considerando o cenário onde não há rotação dos retângulos, precisamos verificar a distância entre as bordas de cada um dos retângulos.

Para o detalhamento que vem a seguir considere que temos um retângulo chamado `r1` (cor azul) e outro chamado `r2` (cor bege quando sem colisão, vermelho quando em colisão). Precisamos fazer basicamente quatro perguntas para saber se existe uma colisão entre eles dois:
1. A borda DIREITA de `r1` está à DIREITA da borda ESQUERDA de `r2`?
2. A borda ESQUERDA de `r1` está à ESQUERDA da borda direita de `r2`?
3. A borda INFERIOR de `r1` está ABAIXO da borda SUPERIOR de `r2`?
4. A borda SUPERIOR de `r1` está ACIMA da borda INFERIOR de `r2`?

Se a resposta para todas essas perguntas for sim, esses dois retângulos estarão se sobrepondo, ou seja, interpretaremos isso como uma colisão. Pode ser um pouco difícil de visualizar essas condições, então vamos tentar ver alguns exemplos visuais.

### Exemplo 1

![Colisão](/assets/sim.png)

1. A borda DIREITA de `r1` está à DIREITA da borda ESQUERDA de `r2`? **SIM**
2. A borda ESQUERDA de `r1` está à ESQUERDA da borda direita de `r2`? **SIM**
3. A borda INFERIOR de `r1` está ABAIXO da borda SUPERIOR de `r2`? **SIM**
4. A borda SUPERIOR de `r1` está ACIMA da borda INFERIOR de `r2`? **SIM**

### Exemplo 2

![Colisão](/assets/nao_1.png)

1. A borda DIREITA de `r1` está à DIREITA da borda ESQUERDA de `r2`? **SIM**
2. A borda ESQUERDA de `r1` está à ESQUERDA da borda direita de `r2`? **SIM**
3. A borda INFERIOR de `r1` está ABAIXO da borda SUPERIOR de `r2`? **SIM**
4. A borda SUPERIOR de `r1` está ACIMA da borda INFERIOR de `r2`? **NÃO**

### Exemplo 3

![Colisão](/assets/nao_2.png)

1. A borda DIREITA de `r1` está à DIREITA da borda ESQUERDA de `r2`? **SIM**
2. A borda ESQUERDA de `r1` está à ESQUERDA da borda direita de `r2`? **SIM**
3. A borda INFERIOR de `r1` está ABAIXO da borda SUPERIOR de `r2`? **NÃO**
4. A borda SUPERIOR de `r1` está ACIMA da borda INFERIOR de `r2`? **SIM**

### Exemplo 3

![Colisão](/assets/nao_3.png)

1. A borda DIREITA de `r1` está à DIREITA da borda ESQUERDA de `r2`? **SIM**
2. A borda ESQUERDA de `r1` está à ESQUERDA da borda direita de `r2`? **NÃO**
3. A borda INFERIOR de `r1` está ABAIXO da borda SUPERIOR de `r2`? **SIM**
4. A borda SUPERIOR de `r1` está ACIMA da borda INFERIOR de `r2`? **SIM**

Como foi possível ver, não satisfazer a qualquer uma das condições significa que os dois retângulos não estão em colisão.

## Código

O código para tornar essa checagem é simples, basta fazermos uma função que receba as coordenadas de cada um dos retângulos, suas larguras e alturas. Vale salientar que estamos sempre considerando como a coordenada do retângulo o seu canto superior esquerdo. Caso você esteja considerando o centro do retângulo, precisará ajustar o algoritmo. A única diferença relevante aqui em relação à colisão entre círculos é a função `detectar_colisao` (linhas 28-33).

Na definição da função `detectar_colisao` temos os parâmetros `x1`, `y1`, `l1` e `a1`, correspondentes à coordenada X, coordenada Y, largura e altura do retângulo 1, respectivamente, e `x2`, `y2`, `l2` e `a2`, correspondentes à coordenada X, coordenada Y, largura e altura do retângulo 2, respectivamente.

```python
import pyxel

class Retangulos:
    def __init__(self):
        pyxel.init(200, 200)
        self.x = 30
        self.y = 30
        self.largura = 40
        self.altura = 20
        self.colidiu = False
        pyxel.run(self.update, self.draw)

    def update(self):
        self.colidiu = self.detectar_colisao(self.x, self.y, self.largura, self.largura, pyxel.mouse_x, pyxel.mouse_y, self.largura, self.altura)

    def draw(self):
        pyxel.cls(6)
        pyxel.rect(self.x, self.y, self.largura, self.largura, 5)

        cor = 15
        if self.colidiu:
            cor = 8

        pyxel.rect(pyxel.mouse_x, pyxel.mouse_y, self.largura, self.altura, cor)


    def detectar_colisao(self, x1, y1, l1, a1, x2, y2, l2, a2):
        if x1 < x2 + l2 and x1 + l1 > x2 and y1 < y2 + a2 and y1 + a1 > y2:
            return True
        else:
            return False

Retangulos()
```
