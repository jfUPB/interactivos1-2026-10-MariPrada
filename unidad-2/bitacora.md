# Unidad 2

## Bitácora de proceso de aprendizaje


## Bitácora de aplicación 

### Actividad 04
<img width="1528" height="668" alt="image" src="https://github.com/user-attachments/assets/7f55687e-43e0-4eff-b412-1b9552c967fe" />


```python
from microbit import *
import utime
import music

# ---------- Helpers del enunciado ----------
def make_fill_images(on='9', off='0'):
    imgs = []
    for n in range(26):
        rows = []
        k = 0
        for y in range(5):
            row = []
            for x in range(5):
                row.append(on if k < n else off)
                k += 1
            rows.append(''.join(row))
        imgs.append(Image(':'.join(rows)))
    return imgs

FILL = make_fill_images()

ENTRY = "ENTRY"
EXIT  = "EXIT"


# ---------- Timer del curso ----------
class Timer:
    def __init__(self, owner, event_to_post, duration):
        self.owner = owner
        self.event = event_to_post
        self.duration = duration
        self.start_time = 0
        self.active = False

    def start(self, new_duration=None):
        if new_duration is not None:
            self.duration = new_duration
        self.start_time = utime.ticks_ms()
        self.active = True

    def stop(self):
        self.active = False

    def update(self):
        if self.active:
            if utime.ticks_diff(utime.ticks_ms(), self.start_time) >= self.duration:
                self.active = False
                self.owner.post_event(self.event)


# ---------- Task ----------
class Task:
    def __init__(self):
        self.event_queue = []
        self.timers = []
        self.estado_actual = None

        # Configuración del temporizador
        self.n = 20                 # valor inicial (15..25)
        self.MIN_N = 15
        self.MAX_N = 25

        # Un timer que postea "Timeout" cada 1000ms cuando esté activo
        self.tickTimer = self.createTimer("Timeout", 1000)

        self.transicion_a(self.estado_config)

    def createTimer(self, event, duration):
        t = Timer(self, event, duration)
        self.timers.append(t)
        return t

    def post_event(self, ev):
        self.event_queue.append(ev)

    def update(self):
        # 1) Actualizar timers
        for t in self.timers:
            t.update()

        # 2) Procesar cola
        while len(self.event_queue) > 0:
            ev = self.event_queue.pop(0)
            if self.estado_actual:
                self.estado_actual(ev)

    def transicion_a(self, nuevo_estado):
        if self.estado_actual:
            self.estado_actual(EXIT)
        self.estado_actual = nuevo_estado
        self.estado_actual(ENTRY)

    # ---------- Estados ----------
    def estado_config(self, ev):
        if ev == ENTRY:
            self.tickTimer.stop()
            self.n = 20
            display.show(FILL[self.n])

        elif ev == "A":
            # subir tiempo (hasta 25)
            if self.n < self.MAX_N:
                self.n += 1
                display.show(FILL[self.n])

        elif ev == "B":
            # bajar tiempo (hasta 15)
            if self.n > self.MIN_N:
                self.n -= 1
                display.show(FILL[self.n])

        elif ev == "S":
            # armar: inicia cuenta regresiva
            self.transicion_a(self.estado_running)

    def estado_running(self, ev):
        if ev == ENTRY:
            display.show(FILL[self.n])
            self.tickTimer.start(1000)

        elif ev == "Timeout":
            # apagar un pixel cada segundo
            if self.n > 0:
                self.n -= 1
                display.show(FILL[self.n])

            if self.n <= 0:
                self.transicion_a(self.estado_alarm)
            else:
                self.tickTimer.start(1000)

        # (opcional) ignorar A/B/S mientras corre

        elif ev == EXIT:
            self.tickTimer.stop()

    def estado_alarm(self, ev):
        if ev == ENTRY:
            display.show(Image.SKULL)
            # Sonido no bloqueante (mejor que esperar)
            try:
                music.play(music.POWER_DOWN, wait=False, loop=False)
            except:
                pass

        elif ev == "A":
            # volver a configuración (reinicia a 20)
            self.transicion_a(self.estado_config)


# ---------- Loop principal ----------
task = Task()

while True:
    if button_a.was_pressed():
        task.post_event("A")
    if button_b.was_pressed():
        task.post_event("B")
    if accelerometer.was_gesture("shake"):
        task.post_event("S")

    task.update()
    utime.sleep_ms(20)  # esto va en el loop principal, no en los estados
``` 

## Bitácora de reflexión

