# Changelog

Todos los cambios notables en este proyecto serán documentados en este archivo.

## [1.0.0] - 2026-07-18

### Agregado
- Documentación completa del laboratorio AAA/RADIUS
- Configuración base del router Cisco IOSv 15.6
- Configuración de Active Directory en Windows Server 2022
- Configuración del servidor NPS (RADIUS)
- Guía de troubleshooting con soluciones a 10 problemas comunes
- Configuración de IIS y RDS/RemoteApp
- Scripts PowerShell para automatización
- Topología de red GNS3

### Documentos Incluidos
- README.md - Overview del proyecto
- docs/Lab-Guide.md - Guía paso a paso
- docs/NPS-RADIUS-Configuration.md - Configuración del servidor RADIUS
- docs/Troubleshooting.md - Resolución de problemas
- cisco/complete-config.ios - Configuración completa del router
- CHANGELOG.md - Este archivo

### Validado
- ✅ Autenticación SSH con credenciales de Active Directory
- ✅ Diferenciación de privilegios (Nivel 15 vs Nivel 1)
- ✅ Fallback a cuentas locales
- ✅ Auditoría de eventos de autenticación
- ✅ RDS/RemoteApp con HTML5 Web Client

### Conocido
- El atributo `Framed-Protocol` causaba errores de terminal (resuelto)
- La GUI de Windows Server 2022 requería PowerShell para certificados SSL (documentado)
- La consola heredaba políticas RADIUS sin exclusión adecuada (resuelto)

---

## Notas

- Versión del Router: Cisco IOSv 15.6(2)T
- Versión de Windows Server: 2022
- Versión de NPS: Incluida en Windows Server 2022
- Versión de GNS3: 2.2.4 o superior recomendado
