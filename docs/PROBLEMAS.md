# Problemas frecuentes

Primero mira el **registro del día** (`%ProgramData%\AscomLogo\logs\ascomlogo-AAAAMMDD.log`) y el estado en `http://localhost:11111/setup` (en el PC anfitrión). Casi todo error deja ahí su motivo.

## Instalación y servicio

| Síntoma | Causa probable | Qué hacer |
|---|---|---|
| **El instalador no arranca**: Windows muestra un aviso de SmartScreen | El instalador no está firmado | Comprueba la suma SHA-256 ([INSTALACION.md](INSTALACION.md), "Antes de instalar"). Si coincide, "Más información" → "Ejecutar de todas formas" |
| El instalador dice **"No se pudo detener el servicio. No se cambió ningún archivo"** | El servicio no se detuvo a tiempo al actualizar | Detén el servicio a mano (`Stop-Service AscomLogo`, como administrador) y vuelve a ejecutar el instalador. El mensaje trae la ruta del registro del instalador (`%ProgramData%\AscomLogo\logs\instalador-*.log`) |
| **El servicio no arranca tras instalar** ("El servicio quedó registrado, pero no arrancó") | El puerto 11111 está ocupado, la cuenta del servicio no puede escribir en `%ProgramData%\AscomLogo\`, u otro error de arranque | Mira el registro del instalador y el del driver, y el Visor de eventos → Aplicación (orígenes `AscomLogo` y `.NET Runtime`). Para ver el error en pantalla, prueba el programa en una consola (abajo) |
| En el registro o en el Visor de eventos, **"acceso denegado"** sobre `%ProgramData%\AscomLogo\`, o `/setup` no puede guardar | El servicio corre con la cuenta **LocalService** (la de menos privilegios) y le falta el permiso de modificación sobre esa carpeta | Vuelve a ejecutar el instalador (actualización): repone el permiso. Si sigue, compruébalo en tu instalación: las propiedades de seguridad de `%ProgramData%\AscomLogo\` tienen que dar "Modificar" a "SERVICIO LOCAL" (LOCAL SERVICE) |
| **El acceso directo no abre** `/setup` | Cambiaste `alpaca:puerto` en `appsettings.json`: el acceso directo sigue apuntando al 11111 | Abre `http://localhost:<tu puerto>/setup` a mano |
| `/setup` avisa **"alpaca.puerto vale …, pero el driver escucha en el …"** | El `alpaca.puerto` de `config.json` no es el de `appsettings.json` | El que vale es el de `appsettings.json`. Iguala los dos en `/setup` (y reinicia el servicio si cambias `appsettings.json`) |
| `/setup` muestra **"MODO SEGURO: config.json no es válido"** | `config.json` está dañado o es inválido. El driver arrancó en simulación y no habla con el PLC | El registro de arranque lista los errores ("Configuración inválida en …"). Corrígela en `/setup` y guarda, o revierte a un respaldo en "Respaldos". El archivo dañado no se toca hasta que guardes |
| Después de actualizar, **cambió algo de `appsettings.json`** | La actualización lo reemplaza | Tu versión anterior está en `%ProgramFiles%\AscomLogo\appsettings.json.anterior` |

## N.I.N.A.

| Síntoma | Causa probable | Qué hacer |
|---|---|---|
| **N.I.N.A. no encuentra el driver** (en el anfitrión) | El servicio no está en marcha, el descubrimiento está apagado, u otro programa usa el puerto 11111 o el 32227 | `Get-Service AscomLogo` debe decir `Running`. Abre `http://localhost:11111/setup`. Revisa `alpaca:descubrimiento` en `appsettings.json`. En el registro de arranque debe estar "Descubrimiento Alpaca activo en UDP 32227" |
| **N.I.N.A. no encuentra el driver** (en un PC remoto) | Faltan las reglas de firewall en el anfitrión, la red del anfitrión es "Pública" y las reglas son para "Privada", el PC no está en `/setup` → Clientes, o el anfitrión está apagado | Revisa [VARIOS_PC.md](VARIOS_PC.md), pasos 1 y 2. Desde el PC remoto, `Test-NetConnection <IP del anfitrión> -Port 11111` debe dar `TcpTestSucceeded : True`. Con N.I.N.A. 3.3 o posterior, prueba escribir la dirección a mano |
| **403** en el navegador o en N.I.N.A. | La petición viene de un equipo que no es el anfitrión ni un cliente configurado, o un PC remoto intenta cambiar algo en `/setup` (solo el anfitrión puede) | Para un PC remoto: agrégalo en `/setup` del anfitrión con su IP **fija** ([VARIOS_PC.md](VARIOS_PC.md)). Si su IP cambió, corrígela. El registro dice la IP rechazada ("Petición rechazada (403) desde …") |
| No se puede **guardar** en `/setup`: "Hay un dispositivo conectado" | El domo o un interruptor siguen conectados en algún N.I.N.A. | Desconéctalos en todos los PC y vuelve a guardar |

## Techo

| Síntoma | Causa probable | Qué hacer |
|---|---|---|
| Techo en Error: **"Sin lectura del PLC"** | El driver no recibe respuesta del LOGO!: cable, red, PLC apagado, IP, puerto o Unit ID mal configurados, servidor Modbus no habilitado, o el LOGO! sin conexiones de servidor libres | Comprueba que el LOGO! esté encendido y en la red (`Test-NetConnection <IP del LOGO!> -Port 502` desde el anfitrión). Revisa IP, puerto y Unit ID en `/setup` y el servidor Modbus en el programa ([PROGRAMA_LOGO.md](PROGRAMA_LOGO.md)). En el registro, busca "Modbus". Mientras dure, el driver no puede abrir ni cerrar |
| Techo en Error: **"Plazo de apertura vencido"** o **"Plazo de cierre vencido"** | La maniobra tardó más de 1,5 veces el tiempo medido: motor lento, obstáculo, fin de carrera que no llegó, o el motor no arrancó | **Mira el techo.** Si se ve bien, cierra desde N.I.N.A. o con "Cerrar ya". Si se repite, vuelve a medir los tiempos y revisa el motor y los fines de carrera. El registro advierte antes, cuando una maniobra pasa de 1,2 veces lo medido |
| Techo en Error: **"Posición intermedia sin orden del driver (movimiento externo o falla)"** | Ningún fin de carrera activo sin una orden del driver: lo movió otro (un cierre automático del programa, otro programa, el mando) o se detuvo a medio camino (por ejemplo, un corte de red) | Si se está moviendo, espera: al llegar a un fin de carrera vuelve a Abierto o Cerrado. Si está quieto, **mira el techo** y ordena cerrar. Si hubo corte de red y el motor no tiene SAI, no anda hasta que vuelva la energía |
| Techo en Error: **"Fines de carrera incoherentes"** | Abierto y cerrado activos a la vez: falla de cableado, de un fin de carrera, o un punto mal configurado | Revisa en `/setup` los puntos del techo, y después los fines de carrera y su cableado. El driver no abre en Error |
| Techo en Error: **"No se pudo enviar la orden al PLC"** | Falló la escritura de la M de orden | Revisa la comunicación como en "Sin lectura del PLC". El registro dice la marca y el error. La orden no se reintenta |
| La orden se cumple pero el techo **no se mueve** y termina en "Plazo … vencido" | La M de orden no está en el programa del LOGO!, no pasa por el relé de barrido, o el motor no recibe la Q | Revisa el programa en LOGO!Soft Comfort ([PROGRAMA_LOGO.md](PROGRAMA_LOGO.md)). En el registro, "Escritura M… = 1 (…): correcta." confirma que el driver escribió |
| Abrir rechazado: **"Faltan los tiempos medidos de apertura y de cierre del techo…"** | No están configurados los tiempos | Cronométralos con el techo a la vista y cárgalos en `/setup` → Techo |
| Abrir rechazado: **"Corte de red (…): no se abre el techo, porque el motor no tiene SAI."** | El punto de corte de red indica corte | Espera a que vuelva la energía. Si no hubo corte, revisa el punto y "Hay corte cuando el punto vale" en `/setup` |
| Abrir rechazado: **"El techo se está cerrando: cerrar tiene prioridad sobre abrir."** | Hay un cierre en curso | Espera a que cierre y vuelve a abrir |
| Cerrar falla con **"Tu reserva quedó liberada. El techo sigue abierto porque lo usa…"** | Otro PC tiene el techo reservado ("cierra el último") | Es lo esperado: se cerrará cuando el otro PC cierre o cuando venza su reserva ([VARIOS_PC.md](VARIOS_PC.md)). Para cerrar ya, "Cerrar ya" en `/setup` del anfitrión |

## Energía

| Síntoma | Causa probable | Qué hacer |
|---|---|---|
| Interruptor: **"La salida <nombre> no confirmó en <plazo> ms: revisa el programa del LOGO! (M = Q) y el punto de estado."** | Se escribió la orden, pero el punto de estado no cambió en el plazo: la M no está en el programa, el punto de estado está mal configurado o invertido, o la salida o el equipo fallan | Revisa en `/setup` la orden, el punto de estado y "Invertido" de ese interruptor, y en LOGO!Soft Comfort que esa M mueva su salida ([PROGRAMA_LOGO.md](PROGRAMA_LOGO.md)). El driver no reintenta |
| Un interruptor **no se puede escribir** desde un PC | Ese Switch es de otro PC (solo escribe su dueño), o el interruptor no tiene orden (solo lectura) | Conecta en N.I.N.A. el Switch de este PC, o revisa el dueño en `/setup` → Switches |
| `/setup` advierte que una salida "es escribible y su nombre o descripción menciona…" y que **"no deberían ser escribibles"** | Su nombre o descripción sugiere el PC, la red o un equipo de cómputo | Si de verdad alimenta uno de esos equipos, quítale la orden ([SEGURIDAD.md](SEGURIDAD.md)). Si no, cámbiale el nombre o la descripción |

## Probar el programa en una consola

Sirve para ver los errores de arranque en pantalla. Como administrador:

```powershell
Stop-Service AscomLogo
```

```powershell
& "$env:ProgramFiles\AscomLogo\AscomLogo.Driver.exe"
```

Usa la misma configuración que el servicio (`%ProgramData%\AscomLogo\config.json`): si el backend es Real, **habla con el PLC real**, igual que el servicio. Ciérralo con Ctrl+C y vuelve a arrancar el servicio:

```powershell
Start-Service AscomLogo
```
