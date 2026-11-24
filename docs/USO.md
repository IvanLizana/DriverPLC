# Uso del driver

El driver publica, por ASCOM Alpaca:

- **Dome #0, "Driver PLC Techo"**: el techo corredizo. Abrir, cerrar y estado leído de los fines de carrera.
- **Switch #0, #1…**: uno por cada PC que usa el driver, con sus salidas de energía y sus puntos de solo lectura. Cada Switch se llama como lo configures en `/setup` (en los ejemplos, "Energía PC1"). Solo el PC dueño puede escribir en su Switch; los demás lo ven como sensores.

El driver **pide** y **confirma**: escribe una marca M del LOGO! y espera leer el estado (fin de carrera o punto de estado). La seguridad del techo vive en el programa del LOGO! y en el motor ([SEGURIDAD.md](SEGURIDAD.md)).

En esta guía, el PC anfitrión (donde corre el driver) se llama **PC1**, como en los ejemplos; tú le pones el nombre que quieras en `/setup` → Clientes.

## La página `/setup`

En el PC anfitrión, abre el acceso directo "Configuración de Driver PLC" del menú Inicio, o en el navegador:

```
http://localhost:11111/setup
```

Solo desde el propio PC anfitrión se puede cambiar algo. Desde otro PC configurado como cliente, la página solo muestra el estado. Desde cualquier otro equipo, el driver responde 403.

| Parte | Qué hace |
|---|---|
| **Estado del driver** | Se refresca cada 2 s: el driver y sus advertencias, el PLC (conexión y última lectura), el techo con su estado y su motivo, las reservas del techo, los clientes, los Switches y los dispositivos conectados |
| **Cerrar ya** | Borra todas las reservas y cierra el techo. Pide confirmación. Para emergencias o cuando un PC quedó con el techo reservado ([VARIOS_PC.md](VARIOS_PC.md)) |
| **Editar configuración** | Backend (Simulado o Real), PLC (LOGO!), Alpaca, Clientes, Techo, Switches y Simulación. "Validar y revisar los cambios" muestra los errores por campo y las diferencias; "Guardar y aplicar" guarda con respaldo y aplica sin reiniciar el servicio |
| **Respaldos** | La lista de respaldos y "Revertir a la anterior", con las diferencias |
| **Fallas para ensayar** | Solo en simulación ([SIMULACION.md](SIMULACION.md)) |

- Para guardar hay que **desconectar antes el domo y los interruptores** en N.I.N.A., en todos los PC. Si hay un dispositivo conectado, puedes revisar los cambios pero no guardarlos.
- Pasar a backend **Real** exige marcar "Entiendo que las órdenes moverán el techo y la energía reales" además de confirmar.
- Cada guardado deja un respaldo de la configuración anterior en `%ProgramData%\AscomLogo\`.
- El puerto de escucha no se cambia en `/setup`: está en `appsettings.json` ([INSTALACION.md](INSTALACION.md)).

### Qué se configura

| Sección | Campos principales |
|---|---|
| PLC (LOGO!) | IP del LOGO!, puerto Modbus (502 a 510), Unit ID (1 por defecto), plazo de respuesta y período de sondeo |
| Clientes | Nombre del PC local (anfitrión) y clientes remotos (hasta 3, con IP fija) |
| Techo | Orden de abrir y de cerrar (M1 a M64), fines de carrera abierto y cerrado (I1 a I24 o M1 a M64), tiempos medidos de apertura y de cierre, pulso (700 ms como mínimo), vencimiento de reserva y punto de corte de red |
| Switches | Uno por PC, con su dueño y sus interruptores: nombre, descripción, orden (M, o vacía para solo lectura), punto de estado, "Invertido (contacto NC)" y plazo de confirmación |
| Simulación | Recorridos del motor simulado y retardo de las salidas simuladas |

Un ejemplo completo y comentado está en `%ProgramData%\AscomLogo\config.ejemplo.json`. Qué poner según tu programa del LOGO!: [PROGRAMA_LOGO.md](PROGRAMA_LOGO.md).

## Modo simulación

Con backend **Simulado** el driver no habla con el PLC real ([SIMULACION.md](SIMULACION.md)). Se nota en la franja de **SIMULACIÓN** de `/setup`, en "[SIMULACIÓN]" en la descripción de los dispositivos y en "backend SIMULADO" en el registro de arranque.

El driver arranca en simulación cuando no hay `config.json` y cuando la configuración es inválida (**modo seguro**: franja "MODO SEGURO" en `/setup` y el error en el registro).

## Conectar desde N.I.N.A.

N.I.N.A. encuentra el driver por **descubrimiento Alpaca** (UDP 32227):

1. **Equipo → Domo**: en la lista aparece **Driver PLC Techo**. Elígelo y conecta.
2. **Equipo → Interruptor**: aparecen los Switches. Elige el de este PC y conecta.

Si no aparecen, revisa [PROBLEMAS.md](PROBLEMAS.md). Las opciones del domo que conviene poner están en [NINA.md](NINA.md). Para otros PC: [VARIOS_PC.md](VARIOS_PC.md).

Cada PC se conecta por separado: desconectar en un PC no desconecta a los demás.

## Estados del techo

| Estado en N.I.N.A. | Significado |
|---|---|
| **Abierto** | Fin de carrera de abierto activo y el de cerrado no |
| **Cerrado** | Fin de carrera de cerrado activo y el de abierto no |
| **Abriendo** | Se ordenó abrir y todavía no llegó al fin de carrera, dentro del plazo |
| **Cerrando** | Se ordenó cerrar y todavía no llegó al fin de carrera, dentro del plazo |
| **Error** | Uno de los motivos de abajo. El motivo aparece en `/setup` y en el registro |

| Motivo del Error | Qué pasa |
|---|---|
| **Sin lectura del PLC** | No hay comunicación con el LOGO!, o la última lectura correcta tiene más de 3 s |
| **Fines de carrera incoherentes** | Abierto y cerrado activos a la vez |
| **Plazo de apertura vencido** / **Plazo de cierre vencido** | La maniobra tardó más de 1,5 veces el tiempo medido (sin tiempo medido de cierre, el plazo de cierre es de 240 s) |
| **Posición intermedia sin orden del driver (movimiento externo o falla)** | Ningún fin de carrera activo sin una orden del driver: el techo lo movió otro (un cierre automático del programa, otro programa, el mando del motor) o se detuvo a medio camino. Al llegar a un fin de carrera vuelve a Abierto o Cerrado |
| **No se pudo enviar la orden al PLC** | Falló la escritura de la M de orden. La orden termina en Error y no se reintenta |

Reglas de las órdenes:

- **Cerrar** se puede siempre que haya lectura del PLC, incluso desde Error o durante una apertura (el motor invierte). Si ya está cerrado, no hace nada.
- **Abrir** solo desde Cerrado. Se rechaza si hay un cierre en curso, si el estado es Error, si faltan los tiempos medidos de apertura y de cierre, o si el punto de corte de red indica corte (o no tiene lectura). Desde Error, N.I.N.A. cierra primero y después abre.
- **Aparcar el domo** (`ParkDome`) es lo mismo que cerrar. "Aparcado" es lo mismo que Cerrado.
- Con varios PC, cerrar puede quedar **pendiente** ([VARIOS_PC.md](VARIOS_PC.md)).
- Cada maniobra registra su duración real. Si pasa de 1,2 veces lo medido, el registro advierte que el motor o la mecánica pueden estar degradándose.

### Abortar (`AbortSlew`)

El motor no tiene parada y el techo nunca debe quedar a medio camino. Por eso **abortar lleva al estado seguro** (se aparta de la norma ASCOM, que dice "detener"):

- si el techo está **abriendo**, se ordena **cerrar**;
- si está **cerrando**, **sigue cerrando**;
- si está quieto, no hace nada.

N.I.N.A. aborta al cancelar una secuencia durante un movimiento del techo: **cancelar una apertura cierra el techo**.

## Interruptores (Switch)

- Cada interruptor escribible es **encendido / apagado** (valor 1 o 0; cualquier otro valor se rechaza).
- Al cambiarlo, el driver escribe la M y **espera a leer el estado** en su punto de estado. Si no llega en el plazo (3 s por defecto), N.I.N.A. recibe el error "La salida <nombre> no confirmó en <plazo> ms: revisa el programa del LOGO! (M = Q) y el punto de estado." No reintenta.
- El valor que se lee es siempre el del **punto de estado**, nunca el de la orden.
- Los puntos de solo lectura (por ejemplo, el aviso de corte de red) se ven como sensores.
- Sin lectura fresca del PLC, leer un interruptor da error: el driver no inventa valores.
