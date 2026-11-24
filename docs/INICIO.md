# Guía de inicio

Cinco pasos, en este orden. No te saltes el 3: todo lo que puedas probar en simulación, pruébalo ahí.

> **Seguridad:** el driver mueve un techo y corta la energía de equipos reales. Mientras esté en **modo simulación** no habla con el PLC; a partir del paso 4 sí. Lee [SEGURIDAD.md](SEGURIDAD.md) antes del paso 4.

## 1. Comprueba los requisitos

Necesitas Windows 10 u 11 de 64 bits, N.I.N.A. 3.2 o posterior, un LOGO! 8.FS4 con el servidor Modbus TCP habilitado y un programa del LOGO! que el driver pueda usar: **marcas M de orden** y **puntos de estado** (fines de carrera, confirmación de cada salida).

→ [REQUISITOS.md](REQUISITOS.md). Si tu programa todavía no tiene M de orden, no pasa nada: el paso 3 no lo necesita, y el paso 4 explica cómo agregarlas.

## 2. Instala el driver

Descarga `Setup-DriverPLC-<versión>.exe` y su `SHA256SUMS.txt` de la página de versiones del proyecto, comprueba la suma y ejecuta el instalador como administrador en el **PC anfitrión** (el que está en la misma red que el LOGO!). El instalador deja el servicio "Driver PLC" en marcha y en **modo simulación**: no toca el PLC.

→ [INSTALACION.md](INSTALACION.md), incluida la comprobación final ("Comprobar que funciona").

## 3. Prueba en simulación con N.I.N.A.

Con el driver recién instalado, conecta N.I.N.A. al domo **Driver PLC Techo**, abre, cierra, aborta y ensaya fallas desde `/setup`. Arma y prueba tu secuencia de N.I.N.A. completa.

→ [SIMULACION.md](SIMULACION.md) y [NINA.md](NINA.md).

## 4. Adapta el programa del LOGO! y configura `/setup`

1. Lee [SEGURIDAD.md](SEGURIDAD.md) entera.
2. Adapta tu programa en LOGO!Soft Comfort: M de abrir y de cerrar a través de un relé de barrido, fines de carrera, una M por cada salida de energía (M n = Q n) con su punto de estado, y el aviso de corte de red si lo tienes. → [PROGRAMA_LOGO.md](PROGRAMA_LOGO.md).
3. Carga **tus** puntos en `/setup` con el backend todavía en **Simulado** y vuelve a probar: el simulador sigue los puntos que configures. → [SIMULACION.md](SIMULACION.md), "Probar tu configuración".
4. Cuando todo funcione en simulación, cambia el backend a **Real** en `/setup`. → [USO.md](USO.md).
5. Con el backend Real, mira primero solo las lecturas en `/setup` (fines de carrera, puntos de estado) sin ordenar nada: tienen que coincidir con lo que ves en el techo y en las salidas.

## 5. Primera maniobra real, con el techo a la vista

- De día, **con el techo a la vista** y alguien junto al motor que pueda cortarlo.
- Cronometra la apertura y el cierre (por ejemplo, con el mando o el panel del motor) y carga los **tiempos medidos** en `/setup` → Techo. Sin ellos, el driver no abre.
  - Si no tienes otra forma de mover el techo, carga una estimación **generosa** (por ejemplo, el doble de lo que crees que tarda), haz la primera apertura y el primer cierre mirando el techo, y después carga los tiempos reales: el registro dice cuánto tardó cada maniobra ("Techo Abierto: maniobra Abrir de las … completada en … s."). Mientras uses la estimación, el driver tarda más en detectar un techo trabado.
- Desde N.I.N.A., abre y cierra. Comprueba que el estado llega a Abierto y a Cerrado, y que abortar durante una apertura cierra el techo.
- Después, prende y apaga **una salida que no sea crítica** y comprueba que confirma.

→ [SEGURIDAD.md](SEGURIDAD.md) ("Primera maniobra real") y [USO.md](USO.md).

Si algo falla en cualquier paso: [PROBLEMAS.md](PROBLEMAS.md). Si varios PC comparten el techo: [VARIOS_PC.md](VARIOS_PC.md).
