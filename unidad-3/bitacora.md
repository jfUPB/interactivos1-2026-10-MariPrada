# Unidad 3

## Bitácora de proceso de aprendizaje


## Bitácora de aplicación 
###actividad 04
``` javaScript

const TIMER_LIMITS = { min: 15, max: 25, defaultValue: 20 };

const EVENTS = {
  DEC: "A",
  INC: "B",
  START: "S",
  TICK: "Timeout",
};

const UI = {
  dialSize: 250,
  ringWeight: 20,
  bigText: 100,
  configText: 120,
  helpText: 18,
};

class Temporizador extends FSMTask {
  constructor(minValue, maxValue, defaultValue) {
    super();
    this.minValue = minValue;
    this.maxValue = maxValue;
    this.defaultValue = defaultValue;

    this.configValue = defaultValue;
    this.totalSeconds = defaultValue;
    this.remainingSeconds = defaultValue;

    this.myTimer = this.addTimer(EVENTS.TICK, 1000);

    // --- NUEVO: buffer para detectar A-B-A ---
    this.combo = []; // guarda últimas 3 teclas A/B

    this.transitionTo(this.estado_config);
  }

  get currentState() {
    return this.state;
  }

  // --- NUEVO: registra A/B y detecta A-B-A ---
  pushCombo(ev) {
    if (ev !== EVENTS.DEC && ev !== EVENTS.INC) return false;
    this.combo.push(ev);
    if (this.combo.length > 3) this.combo.shift();
    return this.combo.length === 3 &&
      this.combo[0] === EVENTS.DEC &&
      this.combo[1] === EVENTS.INC &&
      this.combo[2] === EVENTS.DEC;
  }

  estado_config = (ev) => {
    if (ev === ENTRY) {
      this.configValue = this.defaultValue;
      this.combo = [];
    } else if (ev === EVENTS.DEC) {
      if (this.configValue > this.minValue) this.configValue--;
    } else if (ev === EVENTS.INC) {
      if (this.configValue < this.maxValue) this.configValue++;
    } else if (ev === EVENTS.START) {
      this.totalSeconds = this.configValue;
      this.remainingSeconds = this.totalSeconds;
      this.combo = [];
      this.transitionTo(this.estado_armed);
    }
  };
  estado_armed = (ev) => {
    if (ev === ENTRY) {
      this.myTimer.start();
      this.combo = [];
    } else if (ev === EVENTS.TICK) {
      if (this.remainingSeconds > 0) {
        this.remainingSeconds--;
        if (this.remainingSeconds === 0) {
          this.transitionTo(this.estado_timeout);
        } else {
          this.myTimer.start();
        }
      }
    }
    // --- NUEVO: A pausa / combo A-B-A vuelve a config ---
    else if (ev === EVENTS.DEC || ev === EVENTS.INC) {
      if (this.pushCombo(ev)) {
        this.transitionTo(this.estado_config);
        return;
      }
      if (ev === EVENTS.DEC) {
        // A = pause
        this.transitionTo(this.estado_paused);
      }
    } else if (ev === EXIT) {
      this.myTimer.stop();
    }
  };

  // --- NUEVO: estado de pausa ---
  estado_paused = (ev) => {
    if (ev === ENTRY) {
      this.myTimer.stop();
    } else if (ev === EVENTS.DEC || ev === EVENTS.INC) {
      // permite que el combo funcione también en pausa
      if (this.pushCombo(ev)) {
        this.transitionTo(this.estado_config);
        return;
      }
      if (ev === EVENTS.DEC) {
        // A = resume
        this.transitionTo(this.estado_armed);
      }
    } else if (ev === EXIT) {
      // al salir de pausa, reinicia el "tick" de 1s
      this.myTimer.start();
    }
  };

  estado_timeout = (ev) => {
    if (ev === ENTRY) {
      console.log("¡TIEMPO!");
      this.combo = [];
    } else if (ev === EVENTS.DEC) {
      this.transitionTo(this.estado_config);
    }
  };
}

let temporizador;
const renderer = new Map();
// =====================
//  micro:bit -> p5.webserial
// =====================
let serial;
let connectBtn;

function setupSerial() {
  serial = new p5.WebSerial();

  // Botón obligatorio (por permisos del navegador)
  connectBtn = createButton("Conectar micro:bit");
  connectBtn.position(12, 12);
  connectBtn.style("padding", "10px 14px");
  connectBtn.style("font-size", "14px");
  connectBtn.style("z-index", "9999");

  connectBtn.mousePressed(() => {
    serial.requestPort(); // abre selector de puertos
    serial.open({ baudRate: 115200 }); // debe coincidir con MicroPython
  });

  serial.on("connected", () => console.log("✅ Serial conectado"));
  serial.on("disconnected", () => console.log("⚠️ Serial desconectado"));
  serial.on("data", serialEvent);
}

function serialEvent() {
  let msg = serial.readLine();
  if (!msg) return;

  msg = msg.trim();
  console.log("📩 micro:bit:", msg);

  // Enviar eventos a tu FSM (igual que el teclado)
  if (msg === "A" || msg === "B" || msg === "S") {
    temporizador.postEvent(msg);
  }
}

function setup() {
  createCanvas(windowWidth, windowHeight);
  temporizador = new Temporizador(
    TIMER_LIMITS.min,
    TIMER_LIMITS.max,
    TIMER_LIMITS.defaultValue
  );

  textAlign(CENTER, CENTER);

  renderer.set(temporizador.estado_config, () => drawConfig(temporizador.configValue));
  renderer.set(temporizador.estado_armed, () => drawArmed(temporizador.remainingSeconds, temporizador.totalSeconds));
  renderer.set(temporizador.estado_paused, () => drawPaused(temporizador.remainingSeconds, temporizador.totalSeconds)); // NUEVO
  renderer.set(temporizador.estado_timeout, () => drawTimeout());

  setupSerial();
}

function draw() {
  temporizador.update();
  renderer.get(temporizador.currentState)?.();
}

function drawConfig(val) {
  background(20, 40, 80);
  fill(255);
  textSize(120);
  text(val, width / 2, height / 2);
  textSize(18);
  fill(200);
  text("A(-) B(+) S(start)", width / 2, height / 2 + 100);
}

function drawArmed(val, total) {
  background(20, 20, 20);
  let pulse = sin(frameCount * 0.1) * 10;
  noFill();
  strokeWeight(20);
  stroke(255, 100, 0, 50);
  ellipse(width / 2, height / 2, 250);
  stroke(255, 150, 0);
  let angle = map(val, 0, total, 0, TWO_PI);
  arc(width / 2, height / 2, 250, 250, -HALF_PI, angle - HALF_PI);
  fill(255);
  noStroke();
  textSize(100 + pulse);
  text(val, width / 2, height / 2);

  textSize(18);
  fill(200);
  text("A = pause | Combo: A-B-A → config", width / 2, height / 2 + 140);
}

// NUEVO
function drawPaused(val, total) {
  background(10, 10, 10);
  noFill();
  strokeWeight(20);
  stroke(255, 255, 255, 60);
  ellipse(width / 2, height / 2, 250);

  fill(255);
  noStroke();
  textSize(80);
  text(val, width / 2, height / 2 - 30);

  textSize(32);
  fill(255, 200, 0);
  text("PAUSADO", width / 2, height / 2 + 50);

  textSize(18);
  fill(200);
  text("A = resume | Combo: A-B-A → config", width / 2, height / 2 + 140);
}

function drawTimeout() {
  let bg = frameCount % 20 < 10 ? color(150, 0, 0) : color(255, 0, 0);
  background(bg);
  fill(255);
  textSize(100);
  text("¡TIEMPO!", width / 2, height / 2);
}

function keyPressed() {
  if (key === "a" || key === "A") temporizador.postEvent("A");
  if (key === "b" || key === "B") temporizador.postEvent("B");
  if (key === "s" || key === "S") temporizador.postEvent("S");
}

function windowResized() {
  resizeCanvas(windowWidth, windowHeight);
}

``` 

## Bitácora de reflexión
