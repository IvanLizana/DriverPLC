<div align="center">
  <img src="https://img.shields.io/badge/ASTROREMOTO-Precision_Software-blue?style=for-the-badge" alt="Astroremoto" />
  <br>
  <p>
    <strong>
      <a href="#-español-documentación-oficial">🇪🇸 ESPAÑOL (LEER PRIMERO)</a>
      &nbsp;&nbsp;&nbsp;|&nbsp;&nbsp;&nbsp;
      <a href="#-english-official-documentation">🇬🇧 ENGLISH (READ BELOW)</a>
    </strong>
  </p>
</div>

---

<a name="-español-documentación-oficial"></a>
# 🔭 Driver PLC - Techo y Energía para N.I.N.A. con LOGO! (v1.0.0)

![Versión](https://img.shields.io/badge/Versión-v1.0.0-gold)
![Plataforma](https://img.shields.io/badge/Plataforma-Windows_10%2F11_x64-blue)
![Estándar](https://img.shields.io/badge/ASCOM-Alpaca-cyan)
![PLC](https://img.shields.io/badge/PLC-Siemens_LOGO!_8.FS4-teal)
![Idioma](https://img.shields.io/badge/Idioma-Español-orange)
![Licencia](https://img.shields.io/badge/Licencia-MIT-green)

<div align="center">
  <h3>Abre y cierra el techo corredizo y maneja la energía del observatorio desde N.I.N.A.</h3>
  <p><em>Un driver ASCOM Alpaca que habla por Modbus TCP con tu PLC Siemens LOGO! y confirma cada orden leyendo el PLC.</em></p>

  <a href="https://github.com/IvanLizana/DriverPLC/releases">
    <img src="https://img.shields.io/badge/DESCARGAR_INSTALADOR_v1.0.0-Clic_Aquí-success?style=for-the-badge&logo=windows&logoColor=white" alt="Descargar" />
  </a>
  &nbsp;
  <a href="docs/INICIO.md">
    <img src="https://img.shields.io/badge/GUÍA_DE_INICIO-Docs-informational?style=for-the-badge&logo=readthedocs&logoColor=white" alt="Guía de inicio" />
  </a>
</div>

> # ⚠️ ADVERTENCIA DE SEGURIDAD
> **ESTE DRIVER MUEVE UN TECHO Y CORTA LA ENERGÍA DE EQUIPOS REALES.** Un error de configuración puede mover el techo con el telescopio debajo o apagar un equipo en uso.
> 1.  **Enclavamientos:** los de seguridad (fines de carrera, tiempos máximos, lluvia, montura aparcada) tienen que estar en el **programa del LOGO!** y en el **motor**. El driver solo pide y comprueba: **no los reemplaza**.
> 2.  **Simulación primero:** después de instalarlo, el driver arranca en **modo simulación** y no habla con el PLC. Prueba todo así antes de pasar al PLC real, y haz la primera maniobra real **con el techo a la vista**.
> 3.  **Lee [docs/SEGURIDAD.md](docs/SEGURIDAD.md)** entera antes de la primera maniobra real.
>
> Se distribuye **sin ninguna garantía** (licencia MIT). Lo usas bajo tu responsabilidad.

---

## 📄 Descripción General

**Driver PLC** es un driver **ASCOM Alpaca** para Windows que deja a **N.I.N.A.** abrir y cerrar un **techo corredizo** y prender y apagar **salidas de energía** a través de un PLC **Siemens LOGO! 8.FS4**, por **Modbus TCP**.

Publica un dispositivo **Dome** ("Driver PLC Techo") y uno o más dispositivos **Switch** ("Driver PLC Energía …"). Así N.I.N.A. usa sus propios ítems de domo e interruptores, sin complementos. Cada orden se **confirma leyendo el PLC**: el driver escribe una marca M y espera ver el fin de carrera o el punto de estado que corresponde; si no llega a tiempo, informa un error.

---

## 📜 Historial de Versiones

### [v1.0.0] - Primera versión publicada
* Techo (Dome) con apertura y cierre por pulso en marcas M y confirmación con los fines de carrera.
* Energía (Switch) por nivel, con confirmación de cada salida.
* Página de configuración `/setup` con estado en vivo, validación, respaldos y vuelta atrás.
* Modo simulación, varios PC y servicio de Windows.

---

## 🔬 Funcionalidades Core

* **Techo:** abre y cierra con un pulso en una marca M de orden (una para abrir y otra para cerrar) y confirma con los fines de carrera del PLC (entradas I o marcas M).
* **Energía:** prende y apaga salidas por nivel (M = 1 encendido, M = 0 apagado) y confirma cada orden con su punto de estado.
* **Solo Escribe Marcas M:** entre M1 y M64, y solo las que elijas en la configuración. Nunca escribe salidas Q, entradas ni registros.
* **Abortar Cierra:** no hay orden de "detener"; abortar lleva siempre al estado seguro (techo cerrado).
* **Página de Configuración:** `http://localhost:11111/setup`, con estado en vivo, edición con validación, respaldos y vuelta atrás.
* **Varios PC** que comparten el techo (hasta 4): un solo driver en el PC anfitrión, con un Switch por PC y "cierra el último".
* **Modo Simulación** con un LOGO! simulado, para probar N.I.N.A. y tu secuencia sin tocar el PLC.
* **Servicio de Windows** ("Driver PLC"): arranca con Windows, sin que nadie inicie sesión.

### Qué no hace
* **No es la lógica de seguridad del techo** ni un monitor meteorológico o de seguridad.
* **No cambia el programa del LOGO!** ni su configuración: eso lo haces tú con LOGO!Soft Comfort. El driver no trae un programa para el LOGO!.
* **No sabe dónde está la montura:** no tiene sensor de montura aparcada.

---

## 📋 Requisitos

* **Windows 10 u 11 de 64 bits**, con permisos de administrador para instalar. No hace falta instalar .NET: el instalador lo trae.
* **N.I.N.A. 3.2 o posterior** (encuentra el driver por descubrimiento Alpaca).
* **Siemens LOGO! 8.FS4** con el servidor **Modbus TCP** habilitado, una conexión de servidor libre e IP fija en la red local.
* Un **programa del LOGO!** con marcas M de orden y puntos de estado (fines de carrera, confirmación de cada salida). Cómo prepararlo: [docs/PROGRAMA_LOGO.md](docs/PROGRAMA_LOGO.md).

Lista completa: [docs/REQUISITOS.md](docs/REQUISITOS.md).

---

## 📥 Instrucciones de Instalación y Uso

1.  **Descargar:** ve a **[Releases (Lanzamientos)](../../releases)** y baja `Setup-DriverPLC-1.0.0.exe` y `SHA256SUMS.txt`.
2.  **Comprobar:** en PowerShell, en la carpeta de la descarga:
    ```powershell
    Get-FileHash .\Setup-DriverPLC-1.0.0.exe -Algorithm SHA256
    ```
    El valor de `Hash` tiene que ser igual al de `SHA256SUMS.txt` (sin importar mayúsculas). Si no coincide, **no lo ejecutes**: descárgalo de nuevo.
3.  **Instalar:** ejecútalo como administrador y sigue el asistente ([docs/INSTALACION.md](docs/INSTALACION.md)).
4.  **Simular:** abre `http://localhost:11111/setup` y prueba con N.I.N.A. en modo simulación ([docs/SIMULACION.md](docs/SIMULACION.md)).
5.  **Pasar al PLC real:** prepara tu programa del LOGO!, configura el driver y haz la primera maniobra con el techo a la vista ([docs/INICIO.md](docs/INICIO.md)).

> **⚠️ Nota sobre Windows Defender / SmartScreen:**
> El instalador **no está firmado** con un certificado de firma de código, así que Windows puede mostrar "Windows protegió tu PC" o "Editor desconocido", y algunos antivirus pueden advertir.
> * Si comprobaste la suma SHA-256, haz clic en **"Más información"** y luego en **"Ejecutar de todas formas"**.
> * Los textos exactos cambian con la versión de Windows.

Toda la documentación está en [docs/](docs/LEEME.md), empezando por [docs/INICIO.md](docs/INICIO.md).

---

## 🤝 Comunidad

* **¿Encontraste un fallo o tienes una idea?** Mira primero [docs/PROBLEMAS.md](docs/PROBLEMAS.md) y, si no se resuelve, abre un **[Issue](../../issues/new/choose)** con la versión del driver, de Windows y de N.I.N.A., y las líneas del registro del momento del problema (`%ProgramData%\AscomLogo\logs\`).
* **Antes de publicarlo, quita lo que no quieras hacer público:** direcciones IP, nombres de equipos, rutas con tu nombre de usuario y la ubicación de tu observatorio.
* Antes de participar, lee el **[Código de Conducta](CODE_OF_CONDUCT.md)** y **[Cómo Colaborar](CONTRIBUTING.md)**.
* Problemas de seguridad: en privado, según **[SECURITY.md](SECURITY.md)**. Accesibilidad: **[ACCESSIBILITY.md](ACCESSIBILITY.md)**.

---
---

<a name="-english-official-documentation"></a>
# 🔭 Driver PLC - Roof and Power for N.I.N.A. with LOGO! (v1.0.0)

![Version](https://img.shields.io/badge/Version-v1.0.0-gold)
![Platform](https://img.shields.io/badge/Platform-Windows_10%2F11_x64-blue)
![Standard](https://img.shields.io/badge/ASCOM-Alpaca-cyan)
![PLC](https://img.shields.io/badge/PLC-Siemens_LOGO!_8.FS4-teal)
![Language](https://img.shields.io/badge/Language-Spanish_only-orange)
![License](https://img.shields.io/badge/License-MIT-green)

<div align="center">
  <h3>Open and close your roll-off roof and switch your observatory's power from N.I.N.A.</h3>
  <p><em>An ASCOM Alpaca driver that talks Modbus TCP to your Siemens LOGO! PLC and confirms every command by reading the PLC back.</em></p>

  <a href="https://github.com/IvanLizana/DriverPLC/releases">
    <img src="https://img.shields.io/badge/DOWNLOAD_INSTALLER_v1.0.0-Click_Here-success?style=for-the-badge&logo=windows&logoColor=white" alt="Download" />
  </a>
  &nbsp;
  <a href="docs/INICIO.md">
    <img src="https://img.shields.io/badge/GETTING_STARTED-Docs_(Spanish)-informational?style=for-the-badge&logo=readthedocs&logoColor=white" alt="Getting started" />
  </a>
</div>

> # ⚠️ SAFETY WARNING
> **THIS DRIVER MOVES A ROOF AND CUTS POWER TO REAL EQUIPMENT.** A configuration mistake can move the roof with the telescope underneath or switch off a device that is in use.
> 1.  **Interlocks:** the safety interlocks (limit switches, maximum times, rain, mount parked) must live in the **LOGO! program** and in the **motor**. The driver only requests and checks: **it does not replace them**.
> 2.  **Simulation first:** after installation the driver starts in **simulation mode** and does not talk to the PLC. Test everything that way before going to the real PLC, and do the first real move **with the roof in sight**.
> 3.  **Read [docs/SEGURIDAD.md](docs/SEGURIDAD.md)** (Spanish) in full before the first real move.
>
> Provided **without any warranty** (MIT license). Use it at your own risk.

---

## 📄 Overview

**Driver PLC** is an **ASCOM Alpaca** driver for Windows that lets **N.I.N.A.** open and close a **roll-off roof** and switch **power outlets** through a **Siemens LOGO! 8.FS4** PLC over **Modbus TCP**.

It exposes a **Dome** device ("Driver PLC Techo") and one or more **Switch** devices ("Driver PLC Energía …"), so N.I.N.A. uses its own dome and switch items, with no plugins. Every command is **confirmed by reading the PLC**: the driver writes an M flag and waits for the matching limit switch or status point; if it does not arrive in time, it reports an error.

**The program, the setup page and the documentation are in Spanish only.**

---

## 📜 Version History

### [v1.0.0] - First public release
* Roof (Dome) opened and closed with a pulse on M flags, confirmed by the limit switches.
* Power (Switch) by level, with confirmation of every output.
* `/setup` configuration page with live status, validation, backups and rollback.
* Simulation mode, multiple PCs and Windows service.

---

## 🔬 Core Features

* **Roof:** opens and closes with a pulse on a command M flag (one to open, one to close) and confirms with the PLC limit switches (I inputs or M flags).
* **Power:** switches outputs by level (M = 1 on, M = 0 off) and confirms each command with its status point.
* **Writes M Flags Only:** between M1 and M64, and only the ones you choose. It never writes Q outputs, inputs or registers.
* **Abort Closes:** there is no "stop" command; aborting always goes to the safe state (roof closed).
* **Setup Page:** `http://localhost:11111/setup`, with live status, validated editing, backups and rollback.
* **Multiple PCs** sharing the roof (up to 4): a single driver on the host PC, one Switch per PC and "last one closes".
* **Simulation Mode** with a simulated LOGO!, to test N.I.N.A. and your sequence without touching the PLC.
* **Windows Service** ("Driver PLC"): starts with Windows, with nobody logged in.

### What it does not do
* **It is not the roof's safety logic**, nor a weather or safety monitor.
* **It does not change the LOGO! program** or its settings: you do that with LOGO!Soft Comfort. No LOGO! program is included.
* **It does not know where the mount is:** there is no mount-parked sensor.

---

## 📋 Requirements

* **Windows 10 or 11, 64-bit**, with administrator rights to install. No need to install .NET: the installer includes it.
* **N.I.N.A. 3.2 or later** (finds the driver through Alpaca discovery).
* **Siemens LOGO! 8.FS4** with the **Modbus TCP** server enabled, a free server connection and a fixed IP on the local network.
* A **LOGO! program** with command M flags and status points (limit switches, confirmation of each output): [docs/PROGRAMA_LOGO.md](docs/PROGRAMA_LOGO.md).

---

## 📥 Installation & Usage Instructions

1.  **Download:** go to **[Releases](../../releases)** and download `Setup-DriverPLC-1.0.0.exe` and `SHA256SUMS.txt`.
2.  **Verify:** in PowerShell, in the download folder, run `Get-FileHash .\Setup-DriverPLC-1.0.0.exe -Algorithm SHA256` and compare the `Hash` with `SHA256SUMS.txt`. If it does not match, **do not run it**.
3.  **Install:** run it as administrator and follow the wizard ([docs/INSTALACION.md](docs/INSTALACION.md)).
4.  **Simulate:** open `http://localhost:11111/setup` and test with N.I.N.A. in simulation mode ([docs/SIMULACION.md](docs/SIMULACION.md)).
5.  **Go live:** prepare your LOGO! program, configure the driver and do the first move with the roof in sight ([docs/INICIO.md](docs/INICIO.md)).

> **⚠️ Note on Windows Defender / SmartScreen:**
> The installer is **not signed** with a code-signing certificate, so Windows may show "Windows protected your PC" or "Unknown Publisher", and some antivirus programs may warn.
> * If you verified the SHA-256, click **"More info"** and then **"Run anyway"**.

---

## 🤝 Community

* **Found a bug or have an idea?** Check [docs/PROBLEMAS.md](docs/PROBLEMAS.md) first, then open an **[Issue](../../issues/new/choose)** with the driver, Windows and N.I.N.A. versions and the log lines from `%ProgramData%\AscomLogo\logs\`. Issues in English are welcome.
* **Before posting, remove what you do not want to make public:** IP addresses, device names, paths with your user name and your observatory's location.
* Before taking part, read the **[Code of Conduct](CODE_OF_CONDUCT.md)** and **[Contributing](CONTRIBUTING.md)** (in Spanish).
* Security issues: privately, as described in **[SECURITY.md](SECURITY.md)**. Accessibility: **[ACCESSIBILITY.md](ACCESSIBILITY.md)**.

---

## 📜 License / Licencia
The installer is distributed under the **MIT** license (see `LICENSE`); the source code is not published. Third-party components and their licenses: `TERCEROS.md`.
*El instalador se distribuye bajo la licencia **MIT** (ver `LICENSE`); el código fuente no se publica. Componentes de terceros y sus licencias: `TERCEROS.md`.*

ASCOM, N.I.N.A., Siemens and LOGO! are trademarks of their respective owners. Driver PLC is not affiliated with or endorsed by them; the names are used only to indicate compatibility.
*ASCOM, N.I.N.A., Siemens y LOGO! son marcas de sus respectivos dueños. Driver PLC no está afiliado a ellos ni cuenta con su respaldo; los nombres se usan solo para indicar compatibilidad.*

## 🤝 Credits / Créditos
* **Developer:** ASTROREMOTO.
* **Tech Stack:** .NET 8 + ASP.NET Core + ASCOM Alpaca + Modbus TCP + Inno Setup.
* **Support:** [GitHub Issues](../../issues).
