# N.I.N.A. con el driver

Qué configurar en N.I.N.A. para usar el techo y la energía. El driver no cambia N.I.N.A.: todo esto lo pones tú. Antes de decidir, responde la pregunta de la sección "¿Puede chocar tu techo con el telescopio?".

## Opciones del domo

En N.I.N.A., opciones del domo (Opciones → Equipo → Domo). Los nombres de abajo son los internos de N.I.N.A.; en pantalla aparecen con un texto descriptivo parecido, que puede cambiar con la versión: compruébalo en tu instalación.

Configuración recomendada (para un techo que **puede** chocar con el telescopio, la más segura):

| Opción | Valor | Por qué |
|---|---|---|
| `ParkMountBeforeShutterMove` (aparcar la montura antes de mover la persiana) | **Sí** | El techo no se mueve con la montura fuera de su posición de aparcado |
| `RefuseUnsafeShutterMove` (rechazar mover la persiana si es inseguro) | **Sí**, para abrir | Con el monitor de seguridad de N.I.N.A. en "inseguro", N.I.N.A. no abre |
| `ParkDomeBeforeShutterMove` (aparcar el domo antes de mover la persiana) | **No** | En este driver, aparcar el domo es cerrar: cerraría antes de abrir |
| `CloseOnUnsafe` (cerrar al volverse inseguro) | **No** | Ver "Cierre automático en segundo plano" |

## ¿Puede chocar tu techo con el telescopio?

Depende de tu observatorio: de la altura de los telescopios, de cómo corre el techo y de hacia dónde pueden apuntar. **El driver no lo sabe ni lo comprueba**: no conoce la posición de la montura, no hay sensor de montura aparcada y atiende cualquier orden de cerrar venga de donde venga. La respuesta solo cambia lo que configuras en N.I.N.A.

Compruébalo en tu instalación: con el techo abierto, mueve cada telescopio (a mano o a baja velocidad) por las posiciones que use de noche y mira si alguna parte queda en el recorrido del techo.

### Si puede chocar

- `ParkMountBeforeShutterMove` = **Sí**.
- En la secuencia, **aparca la montura antes de cerrar** (`ParkScope` antes de `CloseDomeShutter`), también en el bloque que reacciona a un "inseguro".
- **No actives `CloseOnUnsafe`**: no está verificado que N.I.N.A. aparque la montura antes de ese cierre.
- Si varios PC comparten el techo, **todos** tienen que aparcar antes de cerrar ([VARIOS_PC.md](VARIOS_PC.md)).
- Lo más seguro es que tu programa del LOGO! o el motor tengan su propio enclavamiento (por ejemplo, un sensor de montura aparcada que bloquee el cierre). Eso no lo hace el driver.

### Si no puede chocar en ninguna posición

- Puedes dejar `ParkMountBeforeShutterMove` en **No**: el techo se mueve sin esperar a la montura.
- Puedes activar `CloseOnUnsafe` para que N.I.N.A. cierre por su cuenta ante un "inseguro" (sección "Cierre automático en segundo plano", más abajo).
- Aun así, aparcar antes de cerrar sigue siendo una buena costumbre: protege si cambias de telescopio o de montaje y olvidas revisar esto.

Si dudas, trata tu techo como uno que **puede** chocar.

## Secuencia recomendada

1. **`SetSwitchValue`**: encender los equipos. Espera a que arranquen (por ejemplo, unos 3 minutos).
2. Conectar los equipos.
3. **`OpenDomeShutter`**: abrir el techo.
4. … la noche …
5. **`ParkScope`**: aparcar la montura.
6. **`CloseDomeShutter`** (o **`ParkDome`**): cerrar el techo.
7. **`SetSwitchValue`**: apagar los equipos.

En los ítems del techo (pasos 3 y 6), pon **`ErrorBehavior` = `AbortOnError`**: si el cierre falla, la secuencia se detiene **sin cortar la energía**, y la montura sigue alimentada y aparcada.

Si el techo lo comparte otro PC, el final de la noche cambia: [VARIOS_PC.md](VARIOS_PC.md), "Final de la noche con dos PC".

Ante un "inseguro" durante la noche, la secuencia es quien cierra: detener el guiado, aparcar, comprobar que la montura está aparcada y cerrar. Arma ese bloque con los disparadores y las condiciones de seguridad de N.I.N.A., y ensáyalo en simulación ([SIMULACION.md](SIMULACION.md)).

## Cierre automático en segundo plano (`CloseOnUnsafe`)

N.I.N.A. puede cerrar el techo por su cuenta cuando su monitor de seguridad pasa a "inseguro", aunque no haya una secuencia corriendo. Es la opción `CloseOnUnsafe`.

**Desactivada por defecto**, y la configuración recomendada no la usa: cierra desde la secuencia, que primero aparca.

**Para activarla:** en las opciones del domo de N.I.N.A., activa la opción de cerrar la persiana cuando el monitor de seguridad informe inseguro, con el monitor de seguridad conectado.

**Riesgos, antes de activarla:**

- **No está verificado que N.I.N.A. aparque la montura antes de ese cierre.** El cierre puede llegar con la montura fuera de su posición de aparcado.
- **No hay sensor de montura aparcada.** Ni el driver ni el LOGO! saben dónde está la montura: nada impide cerrar con el telescopio en el recorrido del techo.
- Actívala solo si tu techo no puede chocar con el telescopio (sección anterior).

El driver no distingue de dónde viene el cierre: atiende el `CloseShutter` venga de la secuencia, de `CloseOnUnsafe` o de otro programa.

## Lo que ve N.I.N.A.

- **Abortar:** cancelar una secuencia mientras el techo abre lo **cierra** ([USO.md](USO.md), "Abortar").
- **Error con motivo:** si el domo pasa a Error, el motivo está en `/setup` y en el registro. Desde Error, N.I.N.A. cierra antes de abrir.
- **Rechazos:** si el driver no puede abrir (faltan los tiempos medidos, corte de red, cierre en curso), N.I.N.A. recibe el motivo en el error.
- **Simulación:** con backend Simulado, la descripción de los dispositivos empieza con "[SIMULACIÓN]".
- **Interruptores de otro PC:** se ven como sensores (no se pueden escribir).
