# Seguridad

**Léela entera antes de pasar el driver del modo simulación al PLC real.**

## Lo esencial

- **El driver mueve un techo y corta la energía de equipos reales.** Un techo que se cierra sobre un telescopio, o una montura que se apaga a mitad de un movimiento, pueden romper equipo caro o lastimar a alguien.
- **El driver pide; el PLC y el motor deciden.** El driver escribe una marca M y comprueba el resultado leyendo el PLC. Los **enclavamientos** (fines de carrera del motor, tiempo máximo de maniobra, lluvia, lo que haga falta en tu observatorio) tienen que estar en el **programa del LOGO!** y en el **motor**. El driver no los reemplaza ni los duplica.
- **Sin garantía.** El driver se distribuye con licencia MIT, "tal cual", sin garantía de ningún tipo (archivo `LICENSE`). La responsabilidad de la instalación es tuya.

## Antes de conectarlo al PLC

- **Prueba todo en modo simulación** primero, con tu propia configuración ([SIMULACION.md](SIMULACION.md)).
- Revisa que tu programa del LOGO! haga lo que esperas con cada M de orden ([PROGRAMA_LOGO.md](PROGRAMA_LOGO.md)).
- Configura como escribibles **solo** las salidas que de verdad quieres manejar desde N.I.N.A.

## Salidas que no deben ser escribibles

No configures como escribibles (con una M de orden) las salidas que alimentan:

- el propio **PC anfitrión** del driver;
- la **red** (router, switch, punto de acceso);
- los **equipos de cómputo** de los que depende la sesión.

Apagarlas por error desde una secuencia deja al observatorio sin control, con el techo como esté. Si quieres verlas en N.I.N.A., agrégalas **sin orden**: quedan como sensores de solo lectura. `/setup` advierte cuando el nombre o la descripción de una salida escribible sugiere uno de estos equipos.

## Cortes de energía: SAI

- El **PLC** debería tener SAI (alimentación ininterrumpida).
- Si el **motor del techo no tiene SAI**, un corte de red con el techo abierto **no se puede resolver desde el software**: ni el driver ni N.I.N.A. pueden cerrarlo hasta que vuelva la energía o entre un generador. Es un riesgo del hardware.
- Si tu programa avisa el corte de red, configúralo en `/setup` → Techo: el driver no abre con corte y lo dice ([PROGRAMA_LOGO.md](PROGRAMA_LOGO.md)).
- Conviene que el PC anfitrión y la red también tengan SAI.

## ¿Puede chocar el techo con el telescopio?

Si al cerrarse el techo puede golpear un telescopio que no esté aparcado, **aparca siempre la montura antes de cerrar**. Ni el driver ni el LOGO! saben dónde está la montura: no hay sensor de montura aparcada. Cómo configurarlo en N.I.N.A.: [NINA.md](NINA.md), "¿Puede chocar tu techo con el telescopio?".

## Abortar cierra el techo

El driver no tiene una orden de "detener": el motor no la tiene y el techo nunca debe quedar a medio camino. Por eso **abortar lleva al estado seguro**:

- si el techo está abriendo, **se ordena cerrar**;
- si está cerrando, sigue cerrando;
- si está quieto, no hace nada.

N.I.N.A. aborta al cancelar una secuencia durante un movimiento del techo: **cancelar una apertura cierra el techo**. Tenlo en cuenta antes de cancelar con el techo moviéndose.

## Si el PC anfitrión se cae

- El motor **termina la maniobra en curso** por su cuenta (sostiene el movimiento con el pulso que ya recibió).
- Mientras el PC anfitrión no vuelva, **ningún PC** puede abrir, cerrar ni conmutar energía con el driver, tampoco los que se conectan por la red ([VARIOS_PC.md](VARIOS_PC.md)).
- Siguen funcionando lo que haga tu programa del LOGO! por su cuenta (por ejemplo, un cierre automático) y el mando del motor.
- Al volver, el driver lee los fines de carrera y queda en Abierto, en Cerrado o en Error si el techo quedó a medio camino.

## Otros programas que escriben en el PLC

Si otro programa también abre o cierra el techo, el driver no lo controla: ve esos movimientos como Error hasta que el techo llega a un fin de carrera. Haz que escriba en las mismas M, no en las Q ([PROGRAMA_LOGO.md](PROGRAMA_LOGO.md)).

Si tu LOGO! lo permite, limita su servidor Modbus a la IP del PC anfitrión ([PROGRAMA_LOGO.md](PROGRAMA_LOGO.md), "Seguridad del puerto 502").

## Primera maniobra real

- **De día**, con buena luz.
- **Con el techo a la vista** y con **alguien junto al motor que pueda cortarlo** (interruptor, mando o corte de energía del motor).
- Sin nada en el recorrido del techo; la montura aparcada si el techo puede chocar con ella.
- Primero, solo lecturas: en `/setup`, los fines de carrera y los puntos de estado tienen que coincidir con lo que ves.
- Cronometra la apertura y el cierre con tus medios y carga los **tiempos medidos** en `/setup`: sin ellos el driver no abre, y con ellos detecta una maniobra que tarda demasiado (Error al pasar 1,5 veces lo medido). Si no puedes cronometrar antes, sigue [INICIO.md](INICIO.md), paso 5.
- Abre y cierra desde N.I.N.A. Prueba también abortar durante una apertura: tiene que cerrar.
- Prueba la energía con **una salida que no sea crítica** antes que con la montura o la cámara.

Los registros del driver (`%ProgramData%\AscomLogo\logs\`) guardan cada escritura al PLC y cada orden con quién la pidió: revísalos después de la primera maniobra.
