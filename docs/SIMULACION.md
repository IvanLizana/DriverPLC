# Probar en modo simulación

En **modo simulación** (backend **Simulado**) el driver arranca un LOGO! simulado dentro del propio PC, con un motor de techo que tarda lo que configures. **No habla con el PLC real** ni mueve nada. Sirve para aprender el driver, ensayar tu secuencia de N.I.N.A. y probar tu configuración antes de pasar a Real.

## Cómo saber que estás en simulación

- Una franja de **SIMULACIÓN** en todas las páginas de `/setup`.
- "[SIMULACIÓN]" al principio de la descripción de los dispositivos que ve N.I.N.A.
- "backend SIMULADO" en la línea de arranque del registro (`%ProgramData%\AscomLogo\logs\`).

## 1. El driver recién instalado ya está en simulación

Sin `config.json`, el driver arranca en simulación con un techo de ejemplo (abrir M1, cerrar M2, abierto I1, cerrado I2), **sin tiempos medidos** y **sin ningún Switch**. Abre `http://localhost:11111/setup` en el PC anfitrión: debes ver la franja de SIMULACIÓN y el techo en Cerrado.

La página avisa "Faltan los tiempos medidos: el techo no abrirá." Es lo esperado: el siguiente punto lo resuelve.

## 2. Carga tiempos para el techo simulado

En `/setup` → **Editar configuración**:

1. En **Simulación**, deja o cambia "Recorrido de apertura (s)" y "Recorrido de cierre (s)" (15 s por defecto): es lo que tarda el motor simulado.
2. En **Techo**, pon "Tiempo medido de apertura (s)" y "Tiempo medido de cierre (s)" con los mismos valores (como mínimo 5 s). El driver da Error si una maniobra tarda más de 1,5 veces lo medido.
3. "Validar y revisar los cambios" y, si no hay errores, "Guardar y aplicar". Se aplica sin reiniciar el servicio.

Para guardar, el domo y los interruptores tienen que estar desconectados en N.I.N.A.

## 3. Conecta N.I.N.A.

En el PC anfitrión, en N.I.N.A.:

1. **Equipo → Domo**: elige **Driver PLC Techo** y conecta. La descripción empieza con "[SIMULACIÓN]".
2. Si agregaste un Switch (punto 5), **Equipo → Interruptor**: elige el Switch de este PC y conecta.

Si no aparecen, revisa [PROBLEMAS.md](PROBLEMAS.md).

## 4. Abre, cierra y aborta

Desde el panel del domo de N.I.N.A.:

- **Abrir:** el estado pasa a Abriendo y, al cabo del recorrido simulado, a Abierto.
- **Cerrar:** pasa a Cerrando y después a Cerrado.
- **Cerrar durante una apertura:** el motor simulado invierte y el techo cierra.
- **Abortar durante una apertura** (o cancelar una secuencia mientras abre): el techo **cierra**. Así funciona también con el PLC real ([USO.md](USO.md), "Abortar").

En `/setup` ves el estado, el motivo de un Error y las reservas del techo; en el registro, cada orden y cada escritura (al LOGO! simulado).

## 5. Ensaya la energía

En **Editar configuración** → **Switches**, "Agregar Switch" con este PC como dueño y uno o dos interruptores (orden y punto de estado, por ejemplo `M10` y `M40`). El simulador conecta cada M de orden escribible a una salida simulada que mueve su punto de estado. Si alguno no se puede simular, `/setup` lo advierte ("Simulación: … no se simula; no confirmará.").

En N.I.N.A., prende y apaga: cada orden se confirma enseguida (la salida simulada tarda lo que diga "Retardo de las salidas (ms)", 300 ms por defecto). Si subes ese retardo por encima del plazo de confirmación del interruptor (3000 ms por defecto), verás el error de una salida que no confirmó.

## 6. Ensaya fallas

En la página principal de `/setup`, **Fallas para ensayar (solo en simulación)**. Afectan solo al LOGO! simulado:

| Falla | Qué deberías ver |
|---|---|
| Cortar la red / Restablecer la red | El motor simulado se detiene. Si configuraste un punto de corte de red, el driver no abre mientras dure |
| Pegar el fin de carrera abierto (o cerrado) en 1 / Liberar los fines de carrera | Con los dos en 1, Error "Fines de carrera incoherentes". Solo funciona con fines de carrera en entradas I |
| Dejar de responder / Volver a responder | Error "Sin lectura del PLC" a los pocos segundos; al volver, el estado real |
| Orden externa de cierre | Un pulso en la M de cerrar, como lo haría otro programa que escribe en el PLC: el driver lo ve como Error ("Posición intermedia…") hasta que el techo llega a Cerrado |

Prueba también qué hace tu secuencia de N.I.N.A. con cada una.

## 7. Prueba tu configuración antes de pasar a Real

El simulador **sigue los puntos que configures**: las M de orden y los fines de carrera del techo (I o M), el corte de red (I o M) y los interruptores.

1. Con el backend todavía en **Simulado**, carga en `/setup` los puntos de **tu** programa del LOGO! ([PROGRAMA_LOGO.md](PROGRAMA_LOGO.md)), tus clientes y tus Switches.
2. "Validar y revisar los cambios": corrige cada error que marque.
3. Guarda y repite los puntos 4 a 6.

Si el techo no se puede simular con tus puntos (por ejemplo, un fin de carrera que es también una M de orden), el driver arranca igual y `/setup` lo advierte.

Cuando todo funcione, cambia el backend a **Real**. Desde ese momento las órdenes mueven el techo y la energía de verdad: antes, lee [SEGURIDAD.md](SEGURIDAD.md) y sigue [INICIO.md](INICIO.md), paso 5.

## 8. Arma y ensaya tu secuencia de N.I.N.A.

Con el driver en simulación, arma la secuencia completa de la noche ([NINA.md](NINA.md), "Secuencia recomendada") y córrela de principio a fin. Prueba:

- que se detiene sin cortar la energía si el cierre falla (por ejemplo, con "Dejar de responder" antes de cerrar);
- qué pasa si cancelas durante la apertura;
- con varios PC, el final de la noche ([VARIOS_PC.md](VARIOS_PC.md)).

Volver a simulación más adelante es seguro: cambia el backend a Simulado en `/setup` y guarda.
