# Requisitos

## PC anfitrión

El driver corre en **un** PC, el anfitrión: el que está en la misma red local que el LOGO!. Los demás PC que compartan el techo se conectan a él por la red ([VARIOS_PC.md](VARIOS_PC.md)).

- **Windows 10 u 11 de 64 bits.**
- Permisos de administrador para instalar.
- El puerto TCP **11111** libre (Alpaca) y el UDP **32227** (descubrimiento Alpaca).
- **No hace falta instalar .NET:** el instalador lo trae.
- Recomendado: SAI para el PC anfitrión y para la red ([SEGURIDAD.md](SEGURIDAD.md)).

## N.I.N.A.

- **N.I.N.A. 3.2 o posterior.** N.I.N.A. encuentra el driver por **descubrimiento Alpaca**. La entrada manual de la dirección del driver existe desde la 3.3; con la 3.2, solo por descubrimiento.
- El driver no necesita ningún complemento de N.I.N.A.: usa los ítems estándar de domo (`OpenDomeShutter`, `CloseDomeShutter`, `ParkDome`) e interruptores (`SetSwitchValue`).

## PLC

- **Siemens LOGO! 8.FS4.** Es el único modelo con el que se probó el driver. Con otros modelos, compruébalo en tu instalación.
- **Servidor Modbus TCP habilitado** en LOGO!Soft Comfort, en el puerto **502** (el driver admite de 502 a 510), con una **conexión de servidor libre** para el driver. El driver abre una sola conexión. El LOGO! comparte sus conexiones de servidor entre S7 y Modbus: si otros programas ya las ocupan todas, el driver no entra. Cómo se habilita exactamente depende de tu versión de LOGO!Soft Comfort: compruébalo en tu instalación ([PROGRAMA_LOGO.md](PROGRAMA_LOGO.md)).
- El **Unit ID** del LOGO! (en el driver, 1 por defecto; se cambia en `/setup`).
- Una **dirección IP fija** del LOGO! en la red local.

## Programa del LOGO!

El driver no trae un programa para el LOGO!: usa el tuyo, con lo siguiente. Cómo prepararlo: [PROGRAMA_LOGO.md](PROGRAMA_LOGO.md).

| Para | Necesitas en el programa |
|---|---|
| Abrir el techo | Una **marca M de orden** (entre M1 y M64) que, a través de un **relé de barrido**, accione la salida Q de abrir del motor |
| Cerrar el techo | Otra **M de orden** que haga lo mismo con la salida de cerrar |
| Saber dónde está el techo | **Dos fines de carrera** propios del PLC: abierto y cerrado, cada uno en una entrada I (I1 a I24) o en una M |
| Cada salida de energía | Una **M de orden** (M n = Q n por nivel) y un **punto de estado** (I o M) que confirme que la salida cambió |
| Aviso de corte de red (opcional) | Una entrada I o una M que indique el corte de la energía de red |

Además:

- Los **enclavamientos de seguridad** del techo (fines de carrera del motor, tiempos máximos, lluvia, lo que haga falta en tu observatorio) tienen que estar en el programa y en el motor. El driver no los reemplaza ([SEGURIDAD.md](SEGURIDAD.md)).
- Un motor de techo que **sostenga el movimiento por su cuenta** hasta su fin de carrera con un pulso corto de abrir o de cerrar, y que con la orden contraria invierta el sentido. Si tu motor no funciona así, compruébalo en tu instalación antes de usar el driver.

## Red

- El PC anfitrión y el LOGO! en la **misma red local**.
- Si otros PC van a usar el driver: cada uno con una **IP fija o reservada** ([VARIOS_PC.md](VARIOS_PC.md)).

## Lo que no hace falta

- .NET ni ningún runtime aparte.
- ASCOM Platform: el driver es Alpaca, N.I.N.A. lo encuentra por la red.
- Cambiar nada en N.I.N.A. más allá de sus opciones ([NINA.md](NINA.md)).
