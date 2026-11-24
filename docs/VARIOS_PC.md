# Usar el driver desde varios PC

Si varios telescopios comparten el techo, cada uno con su PC, **un solo driver**, en el PC **anfitrión** (el que está en la red del LOGO!), sirve a todos. Los demás PC son **clientes remotos**: se conectan por la red local. Se admiten hasta 3 clientes remotos además del anfitrión. El driver mantiene una sola conexión Modbus con el LOGO! y es el único árbitro del techo.

En los ejemplos, el anfitrión se llama **PC1** y el remoto **PC2**; tú les pones los nombres que quieras.

```
 N.I.N.A. de PC1 ──(mismo equipo)──┐
                                   ├──► Driver PLC (en PC1) ──Modbus──► LOGO!
 N.I.N.A. de PC2 ──(red local)─────┘     Dome #0 (común), Switch #0 (PC1), Switch #1 (PC2)
```

No instales el driver en los PC remotos: solo en el anfitrión.

## Qué tiene que estar encendido

- **El PC anfitrión, con el servicio "Driver PLC" en marcha.** Si se apaga, se cuelga o se reinicia, los PC remotos **pierden el techo y la energía**: no pueden abrir, cerrar ni conmutar nada.
- Si el anfitrión cae durante una maniobra, el motor la termina solo.
- Mientras el anfitrión no vuelva, sigue funcionando lo que haga el programa del LOGO! por su cuenta (por ejemplo, un cierre automático).
- El driver identifica a cada PC por su **IP**: cada PC remoto necesita una **IP fija o reservada** en el router.

## 1. Reglas de firewall en el PC anfitrión

Por defecto, el firewall de Windows del anfitrión no deja entrar a nadie de la red. Hacen falta dos reglas de entrada **por cada PC remoto**, limitadas a su IP:

- **TCP 11111**: Alpaca (HTTP);
- **UDP 32227**: descubrimiento Alpaca.

### Con el instalador (recomendado)

Al instalar o actualizar, marca la tarea "Permitir que otros PC de la red usen el driver (reglas de firewall)" y escribe la IP de cada PC (hasta 3). El instalador crea las reglas "Driver PLC - Alpaca HTTP (<IP>)" y "Driver PLC - Descubrimiento (<IP>)", en el perfil **Privado** y en el grupo "Driver PLC (instalador)". La desinstalación las quita.

Para verlas:

```powershell
Get-NetFirewallRule -Group "Driver PLC (instalador)" | Format-Table DisplayName, Enabled, Profile
```

### A mano (alternativa)

Como administrador, en el PC **anfitrión**. Reemplaza `<IP del PC remoto>` por su IP fija y `<nombre>` por un nombre que lo identifique:

```powershell
New-NetFirewallRule -DisplayName "Driver PLC manual - Alpaca HTTP (<nombre>)" -Direction Inbound -Protocol TCP -LocalPort 11111 -RemoteAddress <IP del PC remoto> -Action Allow -Profile Private
```

```powershell
New-NetFirewallRule -DisplayName "Driver PLC manual - Descubrimiento (<nombre>)" -Direction Inbound -Protocol UDP -LocalPort 32227 -RemoteAddress <IP del PC remoto> -Action Allow -Profile Private
```

Para verlas o quitarlas:

```powershell
Get-NetFirewallRule -DisplayName "Driver PLC manual*" | Format-Table DisplayName, Enabled, Profile
```

```powershell
Remove-NetFirewallRule -DisplayName "Driver PLC manual - Alpaca HTTP (<nombre>)"
```

Las reglas creadas a mano no las toca el instalador: si desinstalas el driver, quítalas tú.

### En los dos casos

- La regla se limita a la IP de ese PC: **no abras los puertos a toda la red**.
- Las reglas son para el perfil **Privado**. Si la red del anfitrión figura como "Pública", no se aplican. Compruébalo con `Get-NetConnectionProfile` (columna `NetworkCategory`): si dice `Public`, cambia la red a privada en la configuración de Windows (o, en las reglas a mano, usa `-Profile Public`).
- En el PC remoto normalmente no hace falta ninguna regla: la respuesta del descubrimiento es la respuesta a su propia consulta.
- Aunque el firewall deje pasar a un equipo, el driver **solo atiende** al propio anfitrión y a los clientes configurados en `/setup`; a cualquier otro le responde 403.

## 2. Agregar el cliente en `/setup`

En el PC **anfitrión**, `http://localhost:11111/setup` → **Editar configuración**:

1. En **Clientes**, "Nombre del PC local (anfitrión)" (por ejemplo, PC1) y "Agregar cliente remoto": nombre (por ejemplo, PC2) e IP fija. El nombre "local" está reservado.
2. Si ese PC maneja energía, en **Switches**, "Agregar Switch" con ese cliente como **dueño** y sus interruptores. Solo el dueño puede escribir en su Switch. Las M de orden no se pueden repetir entre Switches ni con las del techo.
3. "Validar y revisar los cambios" y "Guardar y aplicar". Se aplica sin reiniciar el servicio (antes, desconecta el domo y los interruptores en N.I.N.A., en todos los PC).

## 3. Conectar N.I.N.A. en el PC remoto

- **N.I.N.A. 3.2:** solo por **descubrimiento**. Equipo → Domo: aparece **Driver PLC Techo**. Equipo → Interruptor: aparece el Switch de ese PC y los de los demás. Conecta el **suyo** (los otros los ve como sensores).
- **N.I.N.A. 3.3 o posterior:** además del descubrimiento, puedes escribir a mano la dirección del anfitrión y el puerto 11111.

Desde el PC remoto, `http://<IP del anfitrión>:11111/setup` muestra el estado, pero no deja cambiar nada.

Si no aparece, revisa [PROBLEMAS.md](PROBLEMAS.md) ("N.I.N.A. no encuentra el driver").

## 4. La reserva del techo: "cierra el último"

Con varios PC, el techo no se cierra mientras otro lo esté usando:

- **Abrir** desde un PC **reserva** el techo para ese PC. Si ya estaba abierto, solo agrega la reserva.
- **Cerrar** (o aparcar el domo, o abortar) desde un PC **libera su reserva**.
  - Si no queda ninguna reserva, el techo se cierra.
  - Si queda la de otro PC, el cierre queda **pendiente** y el techo **no se mueve**. N.I.N.A. recibe el error: "Tu reserva quedó liberada. El techo sigue abierto porque lo usa <PC>; se cerrará cuando lo liberen."
- **Vencimiento:** si un PC deja de hablar con el driver durante 120 s (configurable de 60 a 900 s en `/setup` → Techo), su reserva vence y queda una advertencia en el registro. Si al vencer no queda ninguna reserva y había un cierre pendiente, **el techo se cierra**.
- Cuando el techo llega a Cerrado por cualquier causa (también un cierre que no ordenó el driver), se borran todas las reservas.
- Las reservas y el cierre pendiente se ven en `/setup`.

Con un solo PC, nada de esto se nota: abrir y cerrar funcionan directamente.

**Si tu techo puede chocar con un telescopio**, cada PC tiene que aparcar su montura antes de pedir el cierre: el cierre lo ejecuta el último que lo pide (o el vencimiento), y el driver no sabe dónde están las monturas ([NINA.md](NINA.md)).

### Final de la noche con dos PC

El primer PC que termina **no puede cerrar**: su `CloseDomeShutter` falla con "El techo sigue abierto porque lo usa…". Con `ErrorBehavior` = `AbortOnError`, su secuencia se detiene ahí y **no apaga la energía**. Para evitarlo, elige una de estas dos formas:

**A. Apagar antes de cerrar** (recomendada):

1. `ParkScope`;
2. `SetSwitchValue` (apagar los equipos de este PC);
3. `CloseDomeShutter` con `AbortOnError`.

Si el cierre queda pendiente, la secuencia se detiene, pero la montura ya está aparcada y apagada. El último PC en terminar cierra el techo.

**B. Seguir aunque el cierre quede pendiente:**

1. `ParkScope`;
2. `CloseDomeShutter` con **`ContinueOnError`**;
3. `SetSwitchValue` (apagar).

Con `ContinueOnError`, un cierre que falla de verdad (por ejemplo, "Plazo de cierre vencido") **tampoco** detiene la secuencia: revisa el registro por la mañana.

Ejemplo: PC1 termina a las 4:30 y PC2 a las 5:10. PC1 aparca, apaga y pide cerrar: el cierre queda pendiente porque PC2 tiene la reserva. A las 5:10, PC2 aparca, apaga y cierra: no queda ninguna reserva y el techo se cierra. Si PC2 se colgara a las 4:45, su reserva vencería a los 120 s sin peticiones y el techo se cerraría solo, porque estaba el cierre pendiente de PC1.

### Cerrar ya

En `/setup` del **anfitrión**, **Cerrar ya** borra todas las reservas y cierra el techo, sin esperar a nadie. Pide confirmación. Úsalo si un PC quedó con el techo reservado o ante una emergencia. Solo se puede usar desde el anfitrión.
