# Avisos de terceros

El instalador de **Driver PLC** (`Setup-DriverPLC-<versión>.exe`) redistribuye los componentes de terceros que siguen. Driver PLC se distribuye con licencia MIT ([LICENSE](LICENSE)); cada componente conserva su propia licencia y sus avisos de copyright.

Lista tomada del archivo de dependencias publicado (`AscomLogo.Driver.deps.json`) de la versión 1.0.0. La licencia de cada paquete se comprobó en su archivo `.nuspec` (paquete NuGet oficial de esa versión exacta): todos declaran la expresión de licencia **MIT**.

## 1. Runtime de .NET y de ASP.NET Core

El driver se publica **autocontenido**: el instalador lleva el runtime, así no hace falta instalar .NET.

| Componente | Versión | Licencia | Copyright | Fuente |
|---|---|---|---|---|
| Microsoft.NETCore.App.Runtime.win-x64 (runtime de .NET) | 8.0.34 | MIT | © Microsoft Corporation. All rights reserved. (el `LICENSE.TXT` del paquete dice: Copyright (c) .NET Foundation and Contributors) | https://github.com/dotnet/runtime |
| Microsoft.AspNetCore.App.Runtime.win-x64 (runtime de ASP.NET Core) | 8.0.34 | MIT | © Microsoft Corporation. All rights reserved. | https://github.com/dotnet/aspnetcore |

El runtime de .NET y el de ASP.NET Core incluyen, a su vez, código de terceros con otras licencias. Sus avisos completos están en el archivo `THIRD-PARTY-NOTICES.TXT` que viene dentro de cada uno de esos dos paquetes NuGet (versión 8.0.34) y en la raíz de cada repositorio de la tabla.

## Programa de instalación: Inno Setup

El instalador está hecho con **Inno Setup 7**, y su programa de instalación forma parte de Inno Setup.

- Licencia: licencia de Inno Setup (permisiva; permite redistribuir, también con fines comerciales, si se conservan los avisos de copyright). Comprobada en el `license.txt` de la instalación de Inno Setup 7.1.1 con la que se compiló.
- Copyright (C) 1997-2026 Jordan Russell. Portions Copyright (C) 2000-2026 Martijn Laan.
- Fuente: https://jrsoftware.org/

## 2. Bibliotecas de ASCOM

| Paquete | Versión | Licencia | Copyright | Fuente |
|---|---|---|---|---|
| ASCOM.Alpaca.Device | 4.1.0 | MIT | Copyright (c) ASCOM Initiative 2019-2026 | https://ascom-standards.org/ |
| ASCOM.Common.Components | 4.1.0 | MIT | Copyright (c) ASCOM Initiative 2019-2026 | https://ascom-standards.org/ |
| ASCOM.Exception.Library | 7.1.2 | MIT | (c) ASCOM Initiative 2025 | https://github.com/ASCOMInitiative/ASCOMPlatform |

## 3. Bibliotecas de Microsoft (NuGet)

Todas: licencia **MIT**, © Microsoft Corporation. All rights reserved. Fuente: https://github.com/dotnet/runtime (salvo donde se indica https://github.com/dotnet/dotnet).

| Paquete | Versión |
|---|---|
| Microsoft.Extensions.Configuration | 8.0.0 |
| Microsoft.Extensions.Configuration.Abstractions | 8.0.0 |
| Microsoft.Extensions.Configuration.Binder | 8.0.2 |
| Microsoft.Extensions.Configuration.CommandLine | 8.0.0 |
| Microsoft.Extensions.Configuration.EnvironmentVariables | 8.0.0 |
| Microsoft.Extensions.Configuration.FileExtensions | 8.0.1 |
| Microsoft.Extensions.Configuration.Json | 8.0.1 |
| Microsoft.Extensions.Configuration.UserSecrets | 8.0.1 |
| Microsoft.Extensions.DependencyInjection | 8.0.1 |
| Microsoft.Extensions.DependencyInjection.Abstractions | 8.0.2 |
| Microsoft.Extensions.Diagnostics | 8.0.1 |
| Microsoft.Extensions.Diagnostics.Abstractions | 8.0.1 |
| Microsoft.Extensions.FileProviders.Abstractions | 8.0.0 |
| Microsoft.Extensions.FileProviders.Physical | 8.0.0 |
| Microsoft.Extensions.FileSystemGlobbing | 8.0.0 |
| Microsoft.Extensions.Hosting | 8.0.1 |
| Microsoft.Extensions.Hosting.Abstractions | 8.0.1 |
| Microsoft.Extensions.Hosting.WindowsServices | 8.0.1 |
| Microsoft.Extensions.Logging | 8.0.1 |
| Microsoft.Extensions.Logging.Abstractions | 8.0.2 |
| Microsoft.Extensions.Logging.Configuration | 8.0.1 |
| Microsoft.Extensions.Logging.Console | 8.0.1 |
| Microsoft.Extensions.Logging.Debug | 8.0.1 |
| Microsoft.Extensions.Logging.EventLog | 8.0.1 |
| Microsoft.Extensions.Logging.EventSource | 8.0.1 |
| Microsoft.Extensions.Options | 8.0.2 |
| Microsoft.Extensions.Options.ConfigurationExtensions | 8.0.0 |
| Microsoft.Extensions.Primitives | 8.0.0 |
| System.Diagnostics.EventLog | 8.0.1 |
| System.IO.Pipelines | 10.0.8 (fuente: https://github.com/dotnet/dotnet) |
| System.ServiceProcess.ServiceController | 8.0.1 |
| System.Text.Encodings.Web | 10.0.8 (fuente: https://github.com/dotnet/dotnet) |
| System.Text.Json | 10.0.8 (fuente: https://github.com/dotnet/dotnet) |

En una publicación autocontenida, varios de estos paquetes no aportan un archivo propio porque el runtime ya trae la misma biblioteca; se listan igual porque figuran en el archivo de dependencias.

## Texto de la licencia MIT

Texto de la licencia MIT, que se aplica a cada componente de esta lista con el aviso de copyright que se indica en su fila:

```
MIT License

Copyright (c) <año> <titular>

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
```
