# Unidad 1

## Bitácora de proceso de aprendizaje
### Actividad 01
¿Qué es un sistema físico interactivo?

Un sistema físico interactivo es una experiencia donde las personas pueden interactuar con un espacio o con objetos reales y estos responden usando tecnología. No es solo ver una pantalla o un video, sino que el cuerpo del usuario participa: moverse, tocar, caminar, jugar o explorar hace que el sistema cambie o reaccione.
Después de ver los videos, entendí que estos sistemas mezclan el mundo físico con el digital. Usan sensores, cámaras, proyecciones, sonido, realidad virtual o seguimiento de movimiento para que lo que hace una persona tenga una respuesta inmediata. Por ejemplo, en experiencias como Moment Factory o The VOID, el espacio, las luces y los sonidos reaccionan a las personas que están dentro.
Lo importante es que el usuario no es solo espectador, sino que se vuelve parte de la experiencia. El sistema depende de lo que la persona haga, y por eso cada experiencia puede ser diferente. También se siente más real e inmersivo porque el cuerpo está involucrado, no solo la vista.

En pocas palabras, un sistema físico interactivo es una forma de usar la tecnología para crear experiencias vivas, donde el espacio, la tecnología y las personas trabajan juntos.


¿Cómo podría aplicar lo que he visto en mi perfil profesional?

Desde mi énfasis en Animación, puedo aplicar lo que he visto creando animaciones que respondan a las acciones de las personas. En los sistemas físicos interactivos, la animación sirve para que los espacios, objetos o personajes se muevan y cambien cuando el usuario camina, toca algo o interactúa con el entorno.

Teniendo en cuenta mi enfasis en animación podría trabajar en:

Animaciones que reaccionen al movimiento del usuario.
Personajes o elementos visuales que cobren vida en experiencias interactivas.
Animaciones para proyecciones o espacios inmersivos.
Experiencias donde la animación ayude a contar una historia de forma más activa.

Esto me ayuda a entender que la animación no es solo para ver, sino también para interactuar y experimentar, y que puede usarse en proyectos de entretenimiento digital más inmersivos.

### Actividad 02

¿Qué es el diseño y el arte generativo?

El diseño o arte generativo es una forma de crear usando reglas y procesos, en lugar de hacer todo manualmente. El diseñador define un input (datos, reglas o valores), luego un proceso (cómo se transforman esos datos) y el sistema genera un output (el resultado visual).

Esto significa que una misma idea puede producir muchos resultados diferentes, dependiendo de los valores que cambien. El diseñador no controla cada detalle, sino que diseña el sistema que crea las formas, colores o movimientos.

### Actividad 04

El programa no funcionaba bien con was_pressed() porque esa función solo detecta el botón una sola vez, justo cuando se presiona. Entonces el micro:bit enviaba el mensaje solo por un momento. En p5.js ese mensaje se leía en un solo frame y en el siguiente ya no había información, por eso el color cambiaba muy rápido y volvía al estado normal.

En cambio, con is_pressed() el micro:bit detecta si el botón está presionado todo el tiempo. Mientras el botón está apretado, el micro:bit sigue enviando el mensaje constantemente. Así p5.js recibe información en cada frame y el color se mantiene estable.

## Bitácora de aplicación 

### Actividad 05

Código para el micro:bit 

```python

from microbit import *

uart.init(baudrate=115200)

while True:
    if button_a.is_pressed():
        uart.write('L')
    elif button_b.is_pressed():
        uart.write('R')
    sleep(100)

```
CÓDIGO PARA P5.JS

Me asegure primero que en el html estuviera la libreria necesaria

<script src="https://unpkg.com/@gohai/p5.webserial@^1/libraries/p5.webserial.js"></script>

Ahora el código en js

```javascript
let port;
let connectBtn;
let x;

function setup() {
  createCanvas(400, 400);
  x = width / 2;

  port = createSerial();
  connectBtn = createButton('Connect to micro:bit');
  connectBtn.position(120, 350);
  connectBtn.mousePressed(connectBtnClick);
}

function draw() {
  background(220);

  if (port.availableBytes() > 0) {
    let data = port.read(1);

    if (data == 'L') {
      x -= 5;
    } 
    else if (data == 'R') {
      x += 5;
    }
  }

  x = constrain(x, 25, width - 25);

  ellipse(x, height / 2, 50, 50);

  if (!port.opened()) {
    connectBtn.html('Connect to micro:bit');
  } else {
    connectBtn.html('Disconnect');
  }
}

function connectBtnClick() {
  if (!port.opened()) {
    port.open('MicroPython', 115200);
  } else {
    port.close();
  }
}
```

El sistema físico interactivo funciona usando los botones del micro:bit como entrada física. Cuando se presiona el botón A, el micro:bit envía una señal para mover el círculo a la izquierda, y cuando se presiona el botón B, envía una señal para moverlo a la derecha.

p5.js recibe estos datos por comunicación serial y cambia la posición del círculo en la pantalla en tiempo real. De esta forma, el movimiento del objeto digital depende directamente de la interacción física con el micro:bit.


## Bitácora de reflexión

