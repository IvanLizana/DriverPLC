# Instalación

El driver se instala con `Setup-DriverPLC-<versión>.exe` en el **PC anfitrión** (el que está en la misma red que el LOGO!). Queda como **servicio de Windows "Driver PLC"** (nombre interno `AscomLogo`): arranca con Windows, sin que nadie inicie sesión, y se reinicia solo si falla.

## Requisitos

- Windows 10 u 11 de 64 bits y permisos de administrador.
- Los puertos TCP 11111 y UDP 32227 libres en el PC.
- No hace falta .NET: el instalador lo trae.

Lista completa: [REQUISITOS.md](REQUISITOS.md).

## Antes de instalar: comprueba la descarga

El instalador **no está firmado**. Cada versión se publica con un archivo `SHA256SUMS.txt` con la suma SHA-256 del instalador. Compárala con la de tu descarga:

```powershell
Get-FileHash .\Setup-DriverPLC-<versión>.exe -Algorithm SHA256
```

El valor de `Hash` tiene que ser igual al de `SHA256SUMS.txt` (sin importar mayúsculas). Si no coincide, no lo ejecutes: descárgalo de nuevo.

### Aviso de SmartScreen

Como el instalador no está firmado, Windows puede mostrar un aviso de SmartScreen ("Windows protegió tu PC") al abrirlo. Si comprobaste la suma, elige "Más información" y después "Ejecutar de todas formas". Los textos exactos cambian con la versión de Windows: compruébalo en tu instalación.

## Instalar

1. Ejecuta `Setup-DriverPLC-<versión>.exe` (pide permisos de administrador).
2. Acepta la licencia (MIT).
3. Elige las tareas:

   | Tarea | Por defecto | Qué hace |
   |---|---|---|
   | Iniciar el servicio al terminar | Marcada | Arranca el servicio al final de la instalación |
   | Crear un acceso directo a la página de configuración | Marcada | "Configuración de Driver PLC" en el menú Inicio, que abre `http://localhost:11111/setup` |
   | Permitir que otros PC de la red usen el driver (reglas de firewall) | **Desmarcada** | Solo si otros PC van a usar el driver ([VARIOS_PC.md](VARIOS_PC.md)) |

4. Si marcaste el firewall, la página **Otros PC de la red** pide la IP de cada PC que va a usar el driver (de 1 a 3). Se crean reglas de entrada **solo para esas IP** (TCP 11111 y UDP 32227), en el perfil de red **Privado**. Solo se acepta la IPv4 de un equipo concreto: no se admiten redes enteras, máscaras, 0.0.0.0 ni direcciones de difusión. **Además** tienes que agregar cada PC en `/setup` → Clientes ([VARIOS_PC.md](VARIOS_PC.md)).
5. Al terminar, el instalador puede abrir la página de configuración.

Los textos estándar del asistente tratan de "usted"; los propios del instalador, de "tú".

## Qué deja y dónde

| Carpeta | Contenido | ¿Lo cambia una actualización? |
|---|---|---|
| `%ProgramFiles%\AscomLogo\` | El programa (`AscomLogo.Driver.exe` y sus archivos), `appsettings.json` y `servicio.ps1` (lo usa el instalador) | Sí, se reemplaza |
| `%ProgramData%\AscomLogo\config.json` | **Tu configuración.** No existe hasta que guardes en `/setup` | **Nunca** |
| `%ProgramData%\AscomLogo\config.ejemplo.json` | Un ejemplo comentado de configuración, con valores neutros | Se reemplaza en cada versión |
| `%ProgramData%\AscomLogo\identidad.json` | Los identificadores únicos de los dispositivos (los usa N.I.N.A. para reconocerlos) | Nunca |
| `%ProgramData%\AscomLogo\config.<fecha>.json` y `respaldos_antiguos\` | Respaldos de cada cambio de configuración | Nunca |
| `%ProgramData%\AscomLogo\logs\` | Registros del driver, uno por día (`ascomlogo-AAAAMMDD.log`), y del instalador (`instalador-*.log`) | Nunca |

El instalador **nunca crea ni sobrescribe `config.json`**.

## Comprobar que funciona

1. El servicio está en marcha (PowerShell):

   ```powershell
   Get-Service AscomLogo
   ```

   Debe decir `Running`. En la consola de servicios de Windows aparece como **Driver PLC**.
2. Abre `http://localhost:11111/setup` (o el acceso directo del menú Inicio). Sin configuración, muestra "Estado del driver" y la franja de **SIMULACIÓN**: el driver arrancó en **modo simulación** y no habla con el PLC hasta que lo configures.
3. La API Alpaca responde:

   ```powershell
   Invoke-RestMethod http://localhost:11111/management/apiversions
   ```

   Debe devolver `Value` = `1`.
4. En `%ProgramData%\AscomLogo\logs\` hay un registro del día. La primera línea de cada arranque dice la versión, el puerto, el backend (Real o SIMULADO) y si corre como servicio o como consola.
5. En N.I.N.A. del mismo PC, **Equipo → Domo**: aparece **Driver PLC Techo**.
6. Reinicia Windows y, sin abrir nada, vuelve al punto 1: el servicio tiene que arrancar solo.

Las advertencias y los errores también van al **Visor de eventos** → Registros de Windows → Aplicación, con el origen `AscomLogo`.

Siguiente: [SIMULACION.md](SIMULACION.md).

## Actualizar

1. En N.I.N.A., **desconecta el domo y los interruptores en todos los PC**, y comprueba que **el techo esté quieto**.
2. Ejecuta el instalador de la versión nueva. Detecta la instalación anterior y muestra la página **Actualización**: marca la casilla "Desconecté el domo y los interruptores en N.I.N.A. (en todos los PC) y el techo está quieto". Sin ella no sigue.
3. El instalador detiene el servicio, reemplaza el programa, lo vuelve a registrar y lo arranca.

Lo que se conserva y lo que no:

- **Se conserva** todo `%ProgramData%\AscomLogo\`: configuración, identificadores, respaldos y registros. El driver arranca con tu configuración; revisa `/setup` antes de volver a conectar N.I.N.A.
- **`appsettings.json` se reemplaza** por el de la versión nueva. El anterior queda como `%ProgramFiles%\AscomLogo\appsettings.json.anterior`. Si lo habías cambiado (puerto, nivel de registro), copia tus cambios al nuevo y reinicia el servicio.
- Si el servicio no se puede detener, el instalador se detiene **sin cambiar ningún archivo** y muestra la ruta de su registro.
- Si el techo se moviera al detener el servicio, el motor termina la maniobra solo; el registro lo dice ("servicio detenido con el techo en movimiento").

### Si instalaste la 0.1.0 con scripts

El instalador actualiza encima de una 0.1.0 instalada con los scripts de PowerShell: detecta el servicio, guarda lo instalado en `%ProgramFiles%\AscomLogo\anteriores\<versión>\` y conserva tu configuración y los identificadores de N.I.N.A. El servicio pasa a llamarse "Driver PLC" (antes, "ASCOM LOGO") y los dispositivos, "Driver PLC Techo" y "Driver PLC Energía …". Si el servicio usaba la cuenta LocalSystem, la conserva. Las reglas de firewall creadas a mano ("ASCOM LOGO - …") no se tocan.

## Desinstalar

En Windows, **Configuración → Aplicaciones → Aplicaciones instaladas → Driver PLC → Desinstalar** (o "Agregar o quitar programas").

La desinstalación:

1. detiene y elimina el servicio;
2. quita **solo** las reglas de firewall que creó el instalador (grupo "Driver PLC (instalador)");
3. borra `%ProgramFiles%\AscomLogo\` entera (también `anteriores\`, si venías de la 0.1.0);
4. pregunta: **"¿Borrar también la configuración, los respaldos y los registros de %ProgramData%\AscomLogo?"**, con **No** por defecto.

Responde **No** si vas a reinstalar: la configuración queda lista para la próxima instalación. Responde **Sí** solo si quieres borrarlo todo; antes, guarda una copia de `config.json`. Una desinstalación silenciosa nunca borra `%ProgramData%\AscomLogo\`.

## Instalación silenciosa

Para instalar sin preguntas (por ejemplo, desde un script), como administrador:

```powershell
.\Setup-DriverPLC-<versión>.exe /VERYSILENT
```

- Para **actualizar** en silencio hace falta además `/TECHOQUIETO=si` (equivale a la casilla de la página Actualización). Sin él, no cambia nada.
- Las tareas se eligen con `/TASKS=` (`iniciar`, `accesodirecto`, `firewall`), y las IP del firewall con `/IPS=` separadas por comas.

## Ajustes en `appsettings.json`

Están en `%ProgramFiles%\AscomLogo\appsettings.json`. Después de cambiarlos, reinicia el servicio como administrador (`Restart-Service AscomLogo`).

| Ajuste | Por defecto | Para qué |
|---|---|---|
| `alpaca:puerto` | `11111` | Puerto HTTP de Alpaca. Si lo cambias, el acceso directo del menú Inicio sigue apuntando al 11111: abre `/setup` a mano con el puerto nuevo. El `alpaca.puerto` de `config.json` es solo informativo: `/setup` avisa si no coinciden |
| `alpaca:direccion` | `*` | `*` escucha en todas las interfaces: hace falta para el descubrimiento y para otros PC |
| `alpaca:descubrimiento` | `true` | Responde al descubrimiento Alpaca (UDP 32227) |
| `Logging:LogLevel:Default` | `Information` | Nivel de registro: `Debug`, `Information`, `Warning` o `Error` |
| `registro:archivo` | `true` | Registro en archivo |
| `registro:carpeta` | vacío | Vacío: `%ProgramData%\AscomLogo\logs` |
| `clientes` | `[]` | **Obsoleta.** Los PC remotos se configuran en `/setup`. Déjala vacía |

## Registros

- Un archivo por día, `ascomlogo-AAAAMMDD.log`, con la fecha local. Se puede abrir mientras el servicio corre (Bloc de notas o `Get-Content -Wait`).
- Quedan 30 días en `logs\`; los más viejos se mueven a `logs\antiguos\`. Nadie los borra: si ocupan demasiado, bórralos tú de `logs\antiguos\`.
- Se registra cada escritura al PLC (marca, valor y motivo), cada cambio de estado del techo, cada orden y quién la pidió, y cada petición rechazada con su motivo.
- El instalador deja su propio registro, `instalador-AAAAMMDD-HHMMSS.log`, en la misma carpeta.
- Los registros contienen la IP del PLC y de los PC: no los publiques.
