# Driver PLC: documentación

**Driver PLC** es un driver ASCOM Alpaca para Windows que deja a N.I.N.A. abrir y cerrar un **techo corredizo** y prender y apagar **salidas de energía** a través de un PLC **Siemens LOGO! 8.FS4**, por Modbus TCP. Publica un dispositivo **Dome** (el techo) y uno o más dispositivos **Switch** (la energía), así N.I.N.A. usa sus propios ítems de domo e interruptores. Cada orden se confirma leyendo el PLC: el driver escribe una marca M y espera ver el fin de carrera o el punto de estado que corresponde.

> **El driver mueve un techo y corta la energía de equipos reales.** Lee [SEGURIDAD.md](SEGURIDAD.md) antes de pasar del modo simulación al PLC.

## Qué no es

- **No es la lógica de seguridad del techo.** Los enclavamientos (fines de carrera, tiempos máximos, lluvia, montura aparcada) tienen que estar en el programa del LOGO! y en el motor. El driver solo **pide** y después **comprueba**.
- **No es un monitor meteorológico ni de seguridad.** Para eso N.I.N.A. usa su propio monitor de seguridad.
- **No cambia el programa del LOGO!** ni su configuración. Eso lo haces tú con LOGO!Soft Comfort.
- No sabe dónde está la montura: no tiene sensor de montura aparcada.

## Orden de lectura

| # | Guía | Para qué |
|---|---|---|
| 1 | [INICIO.md](INICIO.md) | Los cinco pasos para empezar, en orden. **Empieza aquí** |
| 2 | [REQUISITOS.md](REQUISITOS.md) | Qué necesitas: Windows, N.I.N.A., el LOGO! y su programa |
| 3 | [SEGURIDAD.md](SEGURIDAD.md) | Advertencias de seguridad. Léela entera antes de la primera maniobra real |
| 4 | [INSTALACION.md](INSTALACION.md) | Instalar, actualizar y desinstalar con el instalador; comprobar que funciona |
| 5 | [SIMULACION.md](SIMULACION.md) | Probar todo en modo simulación, sin tocar el PLC |
| 6 | [PROGRAMA_LOGO.md](PROGRAMA_LOGO.md) | Cómo adaptar tu programa del LOGO! para el driver |
| 7 | [USO.md](USO.md) | La página `/setup`, los estados del techo, abortar y los interruptores |
| 8 | [NINA.md](NINA.md) | Opciones de N.I.N.A., la secuencia recomendada y "¿puede chocar tu techo con el telescopio?" |
| 9 | [VARIOS_PC.md](VARIOS_PC.md) | Usar el driver desde varios PC que comparten el techo |
| 10 | [PROBLEMAS.md](PROBLEMAS.md) | Síntoma → causa → qué hacer |

## Dónde quedan las cosas

- Programa: `%ProgramFiles%\AscomLogo\`. Corre como servicio de Windows **"Driver PLC"** (nombre interno `AscomLogo`).
- Configuración, respaldos y registros: `%ProgramData%\AscomLogo\`. La configuración se edita en `http://localhost:11111/setup` y se revierte desde `/setup` → Respaldos.

Lo marcado **"compruébalo en tu instalación"** depende de tu PLC, de tu programa o de tu versión de N.I.N.A., y no se pudo verificar de forma general.

El driver se distribuye **sin garantía**, con licencia MIT (archivo `LICENSE`).
