# AAA Cisco RADIUS Lab - Autenticación, Autorización y Auditoría Centralizado

## Objetivo
Implementar y validar un entorno de autenticación, autorización y auditoría (AAA) centralizado para un router Cisco IOS, utilizando un servidor Windows (NPS/RADIUS) y Active Directory. Además, publicar un sitio web IIS interno mediante Servicios de Escritorio Remoto (RDS) vía RemoteApp y cliente HTML5.

### Metas Principales
- ✅ Centralizar credenciales en Active Directory
- ✅ Diferenciar privilegios dinámicamente: Administrador (Nivel 15) y Solo lectura (Nivel 1)
- ✅ Auditar eventos de autenticación en el router
- ✅ Asegurar acceso con cuentas locales de respaldo frente a caídas del servidor RADIUS

---

## 📊 Topología de Red

Implementado en **GNS3** usando máquinas virtuales nativas (Windows Server 2022 y Windows) y un router Cisco IOSv 15.6. Se utilizó un diseño de segmento plano sin VLANs para aislar el entorno de validación AAA y RemoteApp.

### Direccionamiento IP

| Dispositivo | Interfaz / IP | Rol |
|---|---|---|
| R1 (Cisco IOSv) | Gi0/0: 192.168.10.1 /24 | Gateway LAN. NAS RADIUS |
| R1 (Cisco IOSv) | Gi0/1: DHCP | Interfaz WAN (Salida a Internet NAT/PAT) |
| Windows Server 2022 | Eth0: 192.168.10.10 /24 | AD DS, IIS, RDS/RemoteApp, NPS (RADIUS) |
| Windows 10 Pro | Eth0: DHCP (192.168.10.x) | Cliente del dominio para pruebas |

---

## 🔧 Configuración

Cada componente está documentado en detalle en sus respectivos archivos de configuración:

### Windows Server 2022
- **Active Directory (AD DS)**: `docs/AD-Configuration.md`
- **IIS y RDS**: `docs/IIS-RDS-Configuration.md`
- **NPS/RADIUS Server**: `docs/NPS-RADIUS-Configuration.md`

### Cisco Router
- **Configuración Base**: `cisco/router-base-config.ios`
- **AAA y RADIUS**: `cisco/router-aaa-radius-config.ios`
- **Accesos y Logging**: `cisco/router-access-logging-config.ios`
- **Configuración Completa**: `cisco/complete-config.ios`

### Scripts y Automatización
- **PowerShell Scripts**: `scripts/powershell/`
- **Bash Scripts**: `scripts/bash/`

---

## 📁 Estructura del Repositorio

```
AAA-Cisco-RADIUS-Lab/
├── README.md                           # Este archivo
├── CHANGELOG.md                        # Historial de cambios
├── docs/
│   ├── Lab-Guide.md                    # Guía de laboratorio paso a paso
│   ├── AD-Configuration.md             # Configuración de Active Directory
│   ├── IIS-RDS-Configuration.md        # Configuración de IIS y RDS/RemoteApp
│   ├── NPS-RADIUS-Configuration.md     # Configuración del servidor RADIUS
│   ├── Cisco-Router-Setup.md           # Guía completa del router
│   └── Troubleshooting.md              # Resolución de problemas
├── cisco/
│   ├── router-base-config.ios          # Configuración base del router
│   ├── router-aaa-radius-config.ios    # Configuración AAA y RADIUS
│   ├── router-access-logging-config.ios # Accesos y Logging
│   └── complete-config.ios             # Configuración completa
├── scripts/
│   ├── powershell/
│   │   ├── Setup-AD-Domain.ps1         # Crear dominio AD
│   │   ├── Create-AD-Users.ps1         # Crear usuarios y grupos
│   │   ├── Setup-NPS-Policies.ps1      # Configurar políticas NPS
│   │   ├── Setup-IIS-Site.ps1          # Configurar sitio IIS
│   │   └── Setup-RDS-RemoteApp.ps1     # Configurar RDS/RemoteApp
│   ├── bash/
│   │   └── cisco-config.sh              # Script de configuración para router
│   └── automation/
│       └── complete-lab-setup.ps1       # Script de configuración completa
├── gns3/
│   ├── lab-topology.gns3               # Archivo de topología GNS3
│   └── README.md                       # Instrucciones para importar en GNS3
├── captures/
│   ├── README.md                       # Descripciones de capturas
│   └── images/                         # Pantallazos de pruebas exitosas
└── troubleshooting/
    ├── known-issues.md                 # Problemas conocidos y soluciones
    ├── debug-commands.md               # Comandos de depuración
    └── logs-examples.md                # Ejemplos de logs
```

---

## 🚀 Inicio Rápido

### Requisitos Previos
- GNS3 instalado (v2.2.4 o superior)
- Windows Server 2022 VM
- Windows 10 Pro VM
- Cisco IOSv 15.6 imagen
- Acceso a Internet (para descargar VMs)

### Pasos de Implementación

1. **Importar topología GNS3**
   ```bash
   # Abrir GNS3 e importar gns3/lab-topology.gns3
   ```

2. **Configurar Windows Server 2022**
   ```powershell
   # Ejecutar script de setup
   .\scripts\powershell\Setup-AD-Domain.ps1
   ```

3. **Configurar Router Cisco**
   ```
   # Copiar configuración desde cisco/complete-config.ios
   # o ejecutar línea por línea
   ```

4. **Validar Autenticación**
   ```
   # Conectar desde Windows 10 Pro al router
   ssh empresario01@192.168.10.1
   ssh empleado02@192.168.10.1
   ```

---

## ✅ Validación y Pruebas

### Pruebas de Autenticación
- ✅ Usuario Administrador ingresa en modo privilegiado (Nivel 15)
- ✅ Usuario Empleado en modo básico (Nivel 1)
- ✅ Fallback a cuentas locales si RADIUS está caído

### Pruebas de RDS/RemoteApp
- ✅ Acceso vía cliente HTML5 (https://FQDN/RDWeb/webclient)
- ✅ Internet Explorer abre sitio IIS automáticamente
- ✅ Restricción de grupos AD validada

### Pruebas de Auditoría
- ✅ Logs de autenticación en el router
- ✅ Eventos de autenticación en servidor NPS
- ✅ Auditoria en Windows Event Viewer

---

## 🔍 Resolucion de Incidencias Clave

| Síntoma | Causa Raíz | Solución Aplicada |
|---|---|---|
| Router no reconoce comando `radius-server` | La imagen IOSv 15.6(2)T requiere sintaxis nueva | Uso del bloque moderno `radius server <nombre>` |
| Comandos AAA rechazados | Servicio AAA apagado por defecto | Ejecución previa de `aaa new-model` |
| Bloqueo total de consola | Consola heredó políticas RADIUS | Se aplicó una política `none` exclusiva para `line con 0` |
| Error "This line may not run PPP" | NPS enviaba protocolo PPP incompatible con terminal | Se eliminó el atributo `Framed-Protocol` en servidor NPS |
| Fallo en creación de certificado RDS vía GUI | Limitación de GUI en Windows Server 2022 | Generación vía PowerShell con `New-SelfSignedCertificate` |

Ver `docs/Troubleshooting.md` para más detalles.

---

## 📚 Documentación Completa

Para documentación detallada, consulta los siguientes archivos:

- **Guía de Laboratorio**: `docs/Lab-Guide.md`
- **Solución de Problemas**: `docs/Troubleshooting.md`
- **Referencia de Configuración**: `docs/Cisco-Router-Setup.md`
- **Configuración NPS**: `docs/NPS-RADIUS-Configuration.md`
- **Comandos de Depuración**: `troubleshooting/debug-commands.md`

---

## 🤝 Contribuciones

Las contribuciones son bienvenidas. Por favor:

1. Fork el proyecto
2. Crea una rama para tu feature (`git checkout -b feature/nueva-feature`)
3. Commit tus cambios (`git commit -m 'Agrega nueva feature'`)
4. Push a la rama (`git push origin feature/nueva-feature`)
5. Abre un Pull Request

---

## 📝 Licencia

Este proyecto está bajo licencia MIT. Ver archivo `LICENSE` para más detalles.

---

## 📞 Soporte

Para reportar problemas o sugerencias:
- Abre un Issue en GitHub
- Consulta `docs/Troubleshooting.md`
- Revisa los ejemplos de logs en `troubleshooting/logs-examples.md`

---

**Última actualización**: 2026-07-18
**Versión del Lab**: 1.0