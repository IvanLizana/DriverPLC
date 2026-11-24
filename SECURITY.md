# Política de seguridad

## Versiones con soporte
Solo la última versión publicada en [Releases](../../releases) recibe correcciones. Antes de reportar, comprueba que el problema sigue en esa versión.

## Cómo reportar una vulnerabilidad
**No abras un issue público.** Usa el reporte privado de GitHub:

1. Entra en la pestaña **Security** de este repositorio.
2. Pulsa **Report a vulnerability**.
3. Describe el problema, la versión afectada y, si puedes, cómo reproducirlo.

Se intentará responder dentro de una semana. Si se confirma, se publicará una versión corregida y se te dará crédito en las notas, salvo que prefieras quedar en el anonimato.

También cuentan como problemas de seguridad los que puedan **mover el techo o cambiar una salida sin que nadie lo pida**, o dejar el techo a medio camino.

## Qué hace Driver PLC en tu equipo
Para que puedas valorar los riesgos:
- El instalador **pide permisos de administrador**: instala el programa en `%ProgramFiles%\AscomLogo\` y crea el servicio de Windows **"Driver PLC"**.
- El servicio atiende Alpaca en el puerto **TCP 11111** y responde al descubrimiento Alpaca en el **UDP 32227**. Solo atiende al propio PC y a los clientes que configures en `/setup`; a cualquier otro le responde 403.
- **Reglas de firewall:** solo si marcas esa tarea en el instalador, y solo para las IP que escribas. Por defecto no abre nada a la red.
- **No envía datos a internet ni busca actualizaciones.**
- Habla por **Modbus TCP** solo con la IP del LOGO! que configures, y **solo escribe marcas M** entre M1 y M64, las que elijas. Nunca escribe salidas Q, entradas ni registros.
- Guarda su configuración, sus respaldos y sus registros en `%ProgramData%\AscomLogo\`.

Verifica siempre el instalador con el SHA-256 publicado en las notas de cada versión (`SHA256SUMS.txt`).
