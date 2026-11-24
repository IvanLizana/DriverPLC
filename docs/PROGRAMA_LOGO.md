# Cómo adaptar tu programa del LOGO!

El driver no trae un programa para el LOGO!: usa el tuyo. Esta guía dice qué tiene que tener para que el driver pueda pedir órdenes y confirmarlas. Los cambios los haces tú, en LOGO!Soft Comfort; el driver nunca modifica el programa ni la configuración del LOGO!.

Antes de cargar un programa cambiado en el PLC, lee [SEGURIDAD.md](SEGURIDAD.md).

## Lo que el driver escribe y lo que lee

- **Escribe solo marcas M, dentro de M1 a M64, y solo las que configures** en `/setup` como órdenes. Nunca escribe salidas Q, entradas I, bits V ni registros. El rango M1–M64 es fijo: ninguna configuración lo amplía. Una escritura fuera de esas reglas se rechaza antes de enviarse.
- **Lee** entradas I (I1 a I24) y marcas M (M1 a M64): los fines de carrera, los puntos de estado de cada salida y el aviso de corte de red.
- Por Modbus usa solo tres funciones: leer bobinas (1), leer entradas discretas (2) y escribir una bobina (5).

Por eso **el programa del LOGO! decide**: el driver pone una M en 1 o en 0, y es tu programa el que convierte esa M en una salida, con los enclavamientos que hagan falta.

## Techo

### Órdenes de abrir y de cerrar, con relé de barrido

Para cada sentido, una M de orden entra en un **relé de barrido** (en LOGO!Soft Comfort, "relé de barrido" o *wiping relay*, salida de impulso) de **0,6 s**, y la salida del relé va a la salida Q del motor:

```
M de abrir   ──► relé de barrido (T = 0,6 s) ──► Q de abrir del motor
M de cerrar  ──► relé de barrido (T = 0,6 s) ──► Q de cerrar del motor
```

Por qué el relé de barrido:

- El relé da **un solo pulso** de 0,6 s con el flanco de subida de la M. Si la M se queda en 1 (por ejemplo, porque el PC se cayó en medio de la orden), la salida igual vuelve a 0 a los 0,6 s: el "botón" del motor nunca queda apretado.
- Un nivel sostenido **no vuelve a disparar**: para una orden nueva, la M tiene que pasar por 0. El driver lo hace siempre: escribe M = 0, espera 100 ms, escribe M = 1, la **sostiene 700 ms** (más que los 0,6 s del relé, para que el pulso salga entero) y vuelve a escribir M = 0.
- Si la M baja antes de los 0,6 s, el relé corta antes. Por eso el pulso del driver no puede durar menos de 700 ms.

Esto supone un motor que, con un pulso corto, **sostiene el movimiento por su cuenta** hasta su fin de carrera, y que con la orden contraria invierte el sentido. Si tu motor necesita la orden sostenida durante todo el recorrido, el driver no te sirve tal como está: compruébalo en tu instalación.

### Fines de carrera

Dos puntos propios del PLC, **independientes** de los fines de carrera internos del motor si los tiene:

- **abierto**: 1 con el techo completamente abierto;
- **cerrado**: 1 con el techo completamente cerrado.

Cada uno puede ser una entrada I (lo más directo) o una M que ponga el programa. El driver da por cumplida una orden solo cuando lee el fin de carrera de destino, y marca Error si los dos están en 1 a la vez o si ninguno está en 1 sin una orden suya ([USO.md](USO.md), "Estados del techo").

Compruébalo en tu instalación: con el techo abierto, abierto = 1 y cerrado = 0; con el techo cerrado, al revés. Si un fin de carrera tuyo da 0 en su posición (contacto NC sin invertir en el programa), invierte la señal en el programa: el techo no tiene opción "invertido" en el driver.

## Energía

Para cada salida de energía que quieras manejar desde N.I.N.A.:

- una **M de orden por nivel**, copiada a su salida: **M n = Q n** en el programa (M = 1 encendida, M = 0 apagada);
- un **punto de estado** (I o M) que confirme que la salida cambió: por ejemplo, una M que copie la Q, o una entrada cableada a un contacto auxiliar.

El driver escribe la M y espera leer el punto de estado (3 s por defecto). Si no llega, informa el error y no reintenta. El punto de estado **no puede ser la misma M de la orden**: el driver lo rechaza al validar la configuración, porque entonces no confirmaría nada.

Si el punto de estado está invertido (por ejemplo, un contacto NC), márcalo como "Invertido (contacto NC)" en `/setup`.

Hay salidas que **no** deberías hacer escribibles: las que alimentan el propio PC anfitrión, la red o los equipos de cómputo ([SEGURIDAD.md](SEGURIDAD.md)).

## Aviso de corte de red (opcional)

Si tu instalación avisa al LOGO! cuando se corta la energía de red, el driver puede leerlo:

- en una **entrada I** (por ejemplo, un relé NC que cae con el corte), o
- en una **marca M** que ponga tu programa.

En `/setup` → Techo, "Punto de corte de red", y "Hay corte cuando el punto vale" 0 o 1, según tu cableado. Con el punto configurado, el driver **no abre** el techo mientras haya corte y lo dice en N.I.N.A. Si además quieres verlo en N.I.N.A. como sensor, agrégalo como interruptor de solo lectura (sin orden).

## Cómo se traducen I y M a direcciones Modbus

No hace falta para usar el driver: en `/setup` escribes `I2` o `M9`, no números. Sirve si quieres comprobar los puntos con otra herramienta Modbus de solo lectura.

Según la tabla del servidor Modbus de la ayuda de LOGO!Soft Comfort V8.4 (el elemento *n* va en la dirección *n − 1* de la trama):

| Recurso | Tipo Modbus | Número | Dirección en la trama |
|---|---|---|---|
| I n (I1 a I24) | Entrada discreta (función 2) | n | n − 1 |
| Q n (Q1 a Q20) | Bobina | 8192 + n | 8191 + n |
| M n (M1 a M64) | Bobina (funciones 1 y 5) | 8256 + n | 8255 + n |

Ejemplo: M9 es la bobina 8265, dirección 8264 en la trama. El driver nunca escribe en las bobinas de las Q.

## Qué hace el PLC y qué hace el driver

| Función | PLC y motor | Driver |
|---|---|---|
| Sostener el movimiento hasta el final del recorrido | **Motor** (sus propios fines de carrera) | — |
| Cerrar gana a abrir durante un movimiento | **Motor** (invierte con la orden contraria) | Rechaza abrir mientras cierra |
| Ancho del pulso, sin "botón pegado" | **PLC** (relé de barrido de 0,6 s) | Sostiene la M 700 ms y la devuelve a 0 |
| Confirmar que el techo llegó | — | **Fines de carrera del PLC** y un plazo de 1,5 × el tiempo medido |
| Enclavamientos de seguridad (lluvia, tiempos máximos, montura) | **PLC y motor** | No los reemplaza |
| No abrir con corte de red | — | **Driver**, si el punto está configurado |
| Encender y apagar equipos | **PLC** (M n = Q n) | Orden por nivel y confirmación |

## Programa de ejemplo

Un programa de ejemplo con el que funciona el driver. **Tus marcas y entradas serán otras**: usa los números de tu programa.

| Qué | En el programa de ejemplo | En `/setup` |
|---|---|---|
| Abrir | M9 → relé de barrido (0,6 s) → Q del motor de abrir | Orden de abrir: `M9` |
| Cerrar | M6 → relé de barrido (0,6 s) → Q del motor de cerrar | Orden de cerrar: `M6` |
| Techo abierto | Fin de carrera en I2 | Fin de carrera abierto: `I2` |
| Techo cerrado | Fin de carrera en I3 | Fin de carrera cerrado: `I3` |

```
M9 ──► relé de barrido, T = 0,6 s, remanencia off ──► Q (abrir)
M6 ──► relé de barrido, T = 0,6 s, remanencia off ──► Q (cerrar)
I2 = techo abierto      I3 = techo cerrado
```

El mismo ejemplo, en `config.json` (sección `techo`; lo normal es cargarlo en `/setup`):

```json
"techo": {
  "ordenAbrir": "M9",
  "ordenCerrar": "M6",
  "estadoAbierto": "I2",
  "estadoCerrado": "I3",
  "tiempoAbrirMedidoS": null,
  "tiempoCerrarMedidoS": null,
  "pulsoMs": 700,
  "vencimientoReservaS": 120,
  "corteDeRed": null
}
```

Los tiempos medidos quedan vacíos hasta que los cronometres con el techo a la vista ([INICIO.md](INICIO.md), paso 5).

## Seguridad del puerto 502 (opcional)

Cualquier equipo de la red que llegue al puerto Modbus del LOGO! puede escribir sus marcas. Si tu LOGO! lo permite, limita la **conexión de servidor Modbus** a la IP del PC anfitrión del driver: cada conexión de servidor del LOGO! acepta a todos o a una sola IP.

Si otro programa del mismo PC también lee el LOGO! por Modbus, no está verificado si una conexión limitada a una IP admite dos clientes desde ella o si hace falta otra conexión de servidor (por ejemplo, en el puerto 503): compruébalo en tu instalación.

## Otros programas que hablan con el PLC

Si otro programa (por ejemplo, un panel o un flujo de automatización) también abre o cierra el techo o maneja la energía, haz que escriba en **las mismas M** que el driver, no directamente en las Q: el programa recalcula las Q a partir de las M en cada ciclo, y una escritura directa en una Q puede quedar pisada.

El driver tolera los movimientos que no ordenó: los muestra como Error ("Posición intermedia sin orden del driver") hasta que el techo llega a un fin de carrera. Un cierre automático de tu programa (por ejemplo, a una hora fija) funciona igual.

## Lo que no está verificado de forma general

- Qué hace el LOGO! con una M que estaba en 1 cuando volvió la energía (remanencia de las marcas) y si eso puede disparar el relé de barrido. Lo seguro es que las M de orden no sean remanentes: compruébalo en tu instalación.
- Cómo se habilita el servidor Modbus y se limita a una IP en tu versión de LOGO!Soft Comfort: compruébalo en tu instalación.
