# Cómo colaborar con Driver PLC

¡Gracias por querer ayudar! Este repositorio publica **solo el instalador** de Driver PLC y su documentación; el código fuente no es público, así que **no se aceptan pull requests con código**. Hay otras formas muy valiosas de colaborar:

## Reportar un fallo
1. Revisa [docs/PROBLEMAS.md](docs/PROBLEMAS.md) y, en [Issues](../../issues), que no esté reportado ya.
2. Abre un issue con la plantilla **«Reportar un fallo»** y cuenta:
   - versión del driver (la del instalador, o en Windows: Configuración → Aplicaciones → Aplicaciones instaladas → Driver PLC), de Windows y de N.I.N.A.;
   - si estaba en modo simulación o con el PLC real;
   - los pasos para que el fallo se repita;
   - qué esperabas y qué pasó (con capturas de `/setup` o de N.I.N.A. si puedes);
   - las líneas del registro del momento del problema (`%ProgramData%\AscomLogo\logs\`).
3. **Antes de publicarlo, quita lo que no quieras hacer público:** direcciones IP, nombres de equipos, rutas con tu nombre de usuario y la ubicación de tu observatorio.

## Proponer una mejora
Abre un issue con la plantilla **«Proponer una mejora»**: qué te gustaría poder hacer y para qué. Un esquema de tu instalación (techo, motor, PLC) ayuda mucho.

## Compartir experiencias
- Cómo armaste el programa de tu LOGO! para el techo y la energía (sin datos privados).
- Otros motores o techos con los que lo usas, y qué tiempos te funcionaron.

## Documentación
Si algo de [docs/](docs/LEEME.md) no se entiende o está mal, indícalo en un issue o envía un pull request que **solo** corrija la documentación.

## Seguridad
Los problemas de seguridad **no** se reportan en issues públicos: ver [SECURITY.md](SECURITY.md).

Al participar aceptas el [Código de conducta](CODE_OF_CONDUCT.md).
