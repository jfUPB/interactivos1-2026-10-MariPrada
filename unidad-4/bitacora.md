# Unidad 4

## Bitácora de proceso de aprendizaje


## Bitácora de aplicación 

Hice la conexion entre el repositorio base y el servidor


<img width="586" height="390" alt="image" src="https://github.com/user-attachments/assets/74cca00f-4af8-45a7-91c3-c2f0e21e3b16" />
<img width="1916" height="932" alt="image" src="https://github.com/user-attachments/assets/e74fe3f3-da72-4009-b615-4b0daf43e49e" />

COMO CONECTO EL SERVIDOR 

- crear carpeta con mkdir dentro de user 
- copiar el repositorio en el sistema
- entrar a la carpeta en la terminal de gitbash con " cd + nombre de carpeta
- instalar modulos con node (npm install)
- node bridgeServer.js para abrir el servidor y para conectar al microbit (node bridgeServer.js --device microbit"nombredeladaptador")

- en VS code instalar la extension de p5.vscode
- zona inferior derecha, Go Live
- conectarse a pagina al serer

- para conectarse con el microbit:
- node bridgeServer.js --device microbit
- conectarse al server en la pagina (DARLE CONECTAR EN LA PAGINA)




mkdir = crear carpeta
clone = copiar
cd = change directory + donde se quiera entrar según la posición actual
ls = MOSTRAR CONTENIDO de la posición actual
clear = limpiar consola
npm = node package manager
Ctrl + c = apagar servidor 

<details>
  <summary><b>MicrobitASCIIAdapter2.js</b></summary>
  
  ``` js
  
  const { SerialPort } = require("serialport");
const BaseAdapter = require("./BaseAdapter");

class ParseError extends Error {}

function parseFrame(line) {
  const trimmed = line.trim();

  if (!trimmed.startsWith("$")) {
    throw new ParseError("Frame does not start with $");
  }

  const body = trimmed.slice(1);
  const parts = body.split("|");

  if (parts.length !== 6) {
    throw new ParseError(`Expected 6 fields, got ${parts.length}`);
  }

  const data = {};

  for (const part of parts) {
    const [key, value] = part.split(":");
    if (key === undefined || value === undefined) {
      throw new ParseError(`Malformed field: ${part}`);
    }
    data[key] = value;
  }

  if (
    data.T === undefined ||
    data.X === undefined ||
    data.Y === undefined ||
    data.A === undefined ||
    data.B === undefined ||
    data.CHK === undefined
  ) {
    throw new ParseError("Missing required fields");
  }

  const t = Number(data.T);
  const x = Number(data.X);
  const y = Number(data.Y);
  const a = Number(data.A);
  const b = Number(data.B);
  const chk = Number(data.CHK);

  if (![t, x, y, a, b, chk].every(Number.isFinite)) {
    throw new ParseError("Invalid numeric data");
  }

  if (x < -2048 || x > 2047 || y < -2048 || y > 2047) {
    throw new ParseError("Out of expected range");
  }

  if (![0, 1].includes(a) || ![0, 1].includes(b)) {
    throw new ParseError("Invalid button data");
  }

  const calculatedChk = Math.abs(x) + Math.abs(y) + a + b;

  if (calculatedChk !== chk) {
    throw new ParseError(
      `Corrupt frame: CHK=${chk}, expected=${calculatedChk}`
    );
  }

  return {
    x: x | 0,
    y: y | 0,
    btnA: a === 1,
    btnB: b === 1,
  };
}


class MicrobitAsciiAdapter2 extends BaseAdapter {
  constructor({ path, baud = 115200, verbose = false } = {}) {
    super();
    this.path = path;
    this.baud = baud;
    this.port = null;
    this.buf = "";
    this.verbose = verbose;
  }

  async connect() {
    if (this.connected) return;
    if (!this.path) throw new Error("serialPort is required for microbit device mode");

    this.port = new SerialPort({
      path: this.path,
      baudRate: this.baud,
      autoOpen: false,
    });

    await new Promise((resolve, reject) => {
      this.port.open((err) => (err ? reject(err) : resolve()));
    });

    this.connected = true;
    this.onConnected?.(`serial open ${this.path} @${this.baud}`);

    this.port.on("data", (chunk) => this._onChunk(chunk));
    this.port.on("error", (err) => this._fail(err));
    this.port.on("close", () => this._closed());
  }

  async disconnect() {
    if (!this.connected) return;
    this.connected = false;

    if (this.port && this.port.isOpen) {
      await new Promise((resolve, reject) => {
        this.port.close((err) => {
          if (err) reject(err);
          else resolve();
        });
      });
    }
    this.port = null;
    this.buf = "";
    this.onDisconnected?.("serial closed");
  }

  getConnectionDetail() {
    return `serial open ${this.path}`;
  }

  _onChunk(chunk) {
    this.buf += chunk.toString("utf8");

    let idx;
    while ((idx = this.buf.indexOf("\n")) >= 0) {
      const line = this.buf.slice(0, idx).trim();
      this.buf = this.buf.slice(idx + 1);

      if (!line) continue;

      try {
        const parsed = parseCsvLine(line);
        this.onData?.(parsed);
      } catch (e) {
        if (e instanceof ParseError) {
          if (this.verbose) console.log("Bad data:", e.message, "raw:", line);
        } else {
          this._fail(e);
        }
      }
    }

    if (this.buf.length > 4096) this.buf = "";
  }

  _fail(err) {
    this.onError?.(String(err?.message || err));
    this.disconnect();
  }

  _closed() {
    if (!this.connected) return;
    this.connected = false;
    this.port = null;
    this.buf = "";
    this.onDisconnected?.("serial closed (event)");
  }

  async writeLine(line) {
    if (!this.port || !this.port.isOpen) return;
    await new Promise((resolve, reject) => {
      this.port.write(line, (err) => (err ? reject(err) : resolve()));
    });
  }

  async handleCommand(cmd) {
    if (cmd?.cmd === "setLed") {
      const x = Math.max(0, Math.min(4, Math.trunc(cmd.x)));
      const y = Math.max(0, Math.min(4, Math.trunc(cmd.y)));
      const v = Math.max(0, Math.min(9, Math.trunc(cmd.value)));
      await this.writeLine(`LED,${x},${y},${v}\n`);
    }
  }
}

module.exports = MicrobitAsciiAdapter2;
```
</details>

en `bridgeServer.js` AGREGAR mi nuevo adaptador
``` js
const MicrobitAsciiAdapter2 = require("./adapters/MicrobitAsciiAdapter2");
```


y añadir nuevo if en `createAdapter()` con el nuevo adapter

``` js
if (DEVICE === "microbitV2") {
    const path = SERIAL_PATH ?? await findMicrobitPort();
    if (!path) {
      log.error("micro:bit not found. Use --serialPort to specify manually.");
      process.exit(1);
    }
    log.info(`micro:bit found at ${path}`);
    return new MicrobitAsciiAdapter2({ path, baud: BAUD, verbose: VERBOSE });
  }
```

<details>
<summary><b>Sketch.js</b></summary>

``` js
const EVENTS = {
    CONNECT: "CONNECT",
    DISCONNECT: "DISCONNECT",
    DATA: "DATA",
    KEY_PRESSED: "KEY_PRESSED",
    KEY_RELEASED: "KEY_RELEASED",
};

class PainterTask extends FSMTask {
    constructor() {
        super();

        this.rxData = {
            x: 0,
            y: 0,
            btnA: false,
            btnB: false,
            ready: false
        };

        this.circleResolution = 2;
        this.radius = 10;
        this.shouldFill = false;
        this.shouldDraw = false;

        this.transitionTo(this.estado_esperando);
    }

    estado_esperando = (ev) => {
        if (ev.type === "ENTRY") {
            cursor();
            console.log("Waiting for connection...");
        } else if (ev.type === EVENTS.CONNECT) {
            this.transitionTo(this.estado_corriendo);
        }
    };

    estado_corriendo = (ev) => {
        if (ev.type === "ENTRY") {
            noCursor();
            background(255);
            strokeWeight(2);
            stroke(0, 25);
            noFill();

            console.log("Microbit ready to draw");

            this.rxData = {
                x: 0,
                y: 0,
                btnA: false,
                btnB: false,
                ready: false
            };

            this.circleResolution = 2;
            this.radius = 10;
            this.shouldFill = false;
            this.shouldDraw = false;
        }

        else if (ev.type === EVENTS.DISCONNECT) {
            this.transitionTo(this.estado_esperando);
        }

        else if (ev.type === EVENTS.DATA) {
            this.updateLogic(ev.payload);
        }

        else if (ev.type === EVENTS.KEY_PRESSED) {
            this.handleKeys?.(ev.keyCode, ev.key);
        }

        else if (ev.type === EVENTS.KEY_RELEASED) {
            this.handleKeyRelease?.(ev.keyCode, ev.key);
        }

        else if (ev.type === "EXIT") {
            cursor();
        }
    };

    updateLogic(data) {
        this.rxData.ready = true;
        this.rxData.x = data.x;
        this.rxData.y = data.y;
        this.rxData.btnA = data.btnA;
        this.rxData.btnB = data.btnB;

        // Y del acelerómetro -> resolución del polígono
        this.circleResolution = int(map(data.y, -2048, 2047, 2, 10));
        this.circleResolution = constrain(this.circleResolution, 2, 10);

        // X del acelerómetro -> radio
        this.radius = map(data.x, -2048, 2047, -360, 360);

        // Botón B -> fill
        this.shouldFill = data.btnB;

        // Botón A -> dibujar
        this.shouldDraw = data.btnA;
    }
}

let painter;
let bridge;
let connectBtn;
const renderer = new Map();

function setup() {
    createCanvas(720, 720);
    noFill();
    background(255);
    strokeWeight(2);
    stroke(0, 25);

    painter = new PainterTask();
    bridge = new BridgeClient();

    bridge.onConnect(() => {
        connectBtn.html("Disconnect");
        painter.postEvent({ type: EVENTS.CONNECT });
    });

    bridge.onDisconnect(() => {
        connectBtn.html("Connect");
        painter.postEvent({ type: EVENTS.DISCONNECT });
    });

    bridge.onStatus((s) => {
        console.log("BRIDGE STATUS:", s.state, s.detail ?? "");
    });

    bridge.onData((data) => {
        painter.postEvent({
            type: EVENTS.DATA,
            payload: {
                x: data.x,
                y: data.y,
                btnA: data.btnA,
                btnB: data.btnB
            }
        });
    });

    connectBtn = createButton("Connect");
    connectBtn.position(10, 10);
    connectBtn.mousePressed(() => {
        if (bridge.isOpen) bridge.close();
        else bridge.open();
    });

    renderer.set(painter.estado_corriendo, drawRunning);
}

function draw() {
    painter.update();
    renderer.get(painter.state)?.();
}

function drawRunning() {
    let task = painter;

    if (!task.rxData.ready) return;
    if (!task.shouldDraw) return;

    push();
    translate(width / 2, height / 2);

    let angle = TAU / task.circleResolution;

    if (task.shouldFill) {
        fill(34, 45, 122, 50);
    } else {
        noFill();
    }

    beginShape();
    for (let i = 0; i <= task.circleResolution; i++) {
        let x = cos(angle * i) * task.radius;
        let y = sin(angle * i) * task.radius;
        vertex(x, y);
    }
    endShape();

    pop();
}

function windowResized() {
    resizeCanvas(windowWidth, windowHeight);
}
```
</details>

<details>
  <summary><b>Microbit</b></summary>

  ``` py
from microbit import *

uart.init(115200)
display.set_pixel(0,0,9)

startTime = running_time()

while True:
    t = running_time()
    xValue = accelerometer.get_x()
    yValue = accelerometer.get_y()
    aState = 1 if button_a.is_pressed() else 0
    bState = 1 if button_b.is_pressed() else 0
    checksum = abs(xValue) + abs(yValue) + aState + bState
    
    data = "$T:{}|X:{}|Y:{}|A:{}|B:{}|CHK:{}\n".format(t, xValue, yValue, aState, bState, checksum)
    
    uart.write(data)
    
    sleep(100) # Envia datos a 10 Hz
```
</details>


## Bitácora de reflexión
