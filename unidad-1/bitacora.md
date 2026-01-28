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

### Actividad 06 - Explicación del sistema físico interactivo (micro:bit + p5.js)

En esta actividad hice un sistema físico interactivo donde el micro:bit funciona como el control físico (input) y p5.js muestra el resultado en pantalla (output). Ambos se conectan por USB serial, que es como un “canal” para enviar letras entre el micro:bit y el computador en tiempo real.

Empezaré explicando el codigo del micro:bit 

1) Parte del micro:bit (input y envío de datos)

El obejtivo del micro:bit en este sistema fisico interactivo es leer un botón y mandar un mensaje por serial para que p5.js lo reciba.

Las primera tres lineas de código:

```python
from microbit import *
uart.init(baudrate=115200)
while True:
```
Es basicamente un ciclo donde se importan las funciones del micro:bit como botones y comunicación serial, luego se enciende la comunicación por el serial, y se determina si recibe algo o no recibe nada, repitiendose esto todo el tiempo.

Con esta primera parte podemos responder de nuevo la pregunta de la actividad 04, ¿Porque no "funcionaba bien" cuando la orden era was_pressed()?

was_pressed() dectecta el botón solo una vez, lo que hacia que el micro:bit mandara la información solo por un instante lo que hacia que en el p5.js eso se viera como un solo frame.

Por lo cual se cambio a is_pressed() que puede funcionar mas como un estado, diciendome que mientras sostengo el botón sigue devolviendo true todo el tiempo.

Algo así:

```python
if button_a.is_pressed():
    uart.write('A')
else:
    uart.write('N')
sleep(100)
```
Que dice que si oprimo el botón envio A y si no lo aprieto envio N constantemente. 

Esto hace que p5.js siempre este recibiendo algo y el resultado se vuelva mas estable.

-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------
Ahora explicare la parte del p5.js

2) Parte de p5.js (recibir datos y mostrar output)

Ya se que p5.js está conectado al micro:bit, donde este se encarga de crear la parte visual y de leer lo que manda el micro:bit.

Este consta de varias capas para programar:

<img width="158" height="165" alt="image" src="https://github.com/user-attachments/assets/62916176-6754-41b1-8322-a2bb8137e315" />

Siguiendo esta idea nuestro primer paso para que p5.js pueda comunicarse con el microbit es agregar en el codigo html, la libreria que necesitamos para la comunicación:

<img width="813" height="45" alt="image" src="https://github.com/user-attachments/assets/de6dc338-7311-4edb-84eb-f09e33d2bf0f" />

Ya con esto hecho, lo siguiente que hacemos es ya pasar a la parte del código en js, donde siempre tendremos que setear nuestras variables globales:

```javascript
let port;
let connectBtn;
```
En este caso seteamos dos:
Port;: que se encarga de guardar la conexión con el serial
ConnectBtn;: siendo esta la "creación" del botón como tal.

    Luego de setear las variables globales, tenemos que setear nuestro canva y el botón para poder conectar con el micro:bit
    
```javascript
function setup() {
  createCanvas(400, 400);
  port = createSerial();
  connectBtn = createButton('Connect to micro:bit');
  connectBtn.mousePressed(connectBtnClick);
}
```
En esta parte del codigo estamos creando tres cosas:

Se crea el canva donde se dibuja
Creamos el puerto serial que nos permite conexión
Se crea un botón que al momento al que le damos click ejecuta a connectBtnClick()

Luego de crear nuestro canva y el botón, seguimos a Draw donde se lee el serial en cada frame y se dibuja encima del canva


```javascript
if (port.availableBytes() > 0) {
  let dataRx = port.read(1);
  if (dataRx == "A") {
    fill("red");
  } else if (dataRx == "N") {
    fill("green");
  }
}

```

En esta parte del código availableBytes() se encarga de revisar si llego información, read(1) lee la "letra" o la información que llega, en este caso si se recibe "A" cambia el color a "red" y si llega N vuelve a "green".
Lo que hace este codigo es que permine a p5.js no "parpadear" porque constantemenete se le esta mandando información sea "A" o "N"

Lo siguiente seria como se conecta y se desconecta el puerto

```javascript
function connectBtnClick() {
  if (!port.opened()) {
    port.open('MicroPython', 115200);
  } else {
    port.close();
  }
}
```

Que basicamente lo que se entiende en esta función es: 

Si el puerto esta abierto significa que NO esta conectado y aparece Micropython y si esta cerrado esta Conectado.

Conclusión: ¿por qué esto es un sistema físico interactivo?

Porque hay un flujo claro:

Input físico: presionar el botón del micro:bit.

Comunicación: el micro:bit envía letras por serial.

Proceso: p5.js interpreta esas letras.

Output visual: se actualiza el dibujo en pantalla en tiempo real.

Lo más importante que aprendí es que en sistemas interactivos no basta con enviar un mensaje una vez: muchas veces se necesita enviar un “estado” continuamente para que el output se mantenga estable.
