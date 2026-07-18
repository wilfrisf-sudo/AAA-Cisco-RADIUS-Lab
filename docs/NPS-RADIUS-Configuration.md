# Configuración del Servidor NPS (RADIUS)

## 1. Instalación de NPS

### Instalación del Rol NPS

```powershell
# Como Administrador
Install-WindowsFeature NPAS -IncludeManagementTools
```

Después de instalar, abrire la consola de NPS:
- `Start > Search > Network Policy Server`
- O vía PowerShell:
```powershell
npas.msc
```

---

## 2. Registrar Cliente RADIUS (Router Cisco)

### Pasos en la Consola NPS

1. **Navegar a RADIUS Clients**
   - Expandir: `NPS (Local)` → `RADIUS Clients and Servers` → `RADIUS Clients`

2. **Agregar Nuevo Cliente RADIUS**
   - Click derecho → `New RADIUS Client`
   - **Friendly Name**: `Cisco-R1`
   - **Address (IPv4)**: `192.168.10.1`
   - **Shared Secret**: `R4d1usSecr3t2024!`
   - Confirmar con OK

---

## 3. Crear Directivas de Red (Network Policies)

### Directiva 1: Administradores (Nivel 15)

#### Paso 1: Crear la Directiva

1. **Navegar a Network Policies**
   - Expandir: `NPS (Local)` → `Policies` → `Network Policies`

2. **Agregar Nueva Directiva**
   - Click derecho → `New Policy`
   - **Policy Name**: `Cisco-Admin-L15`
   - **Type of network access server**: `Unspecified`
   - Click `Next`

#### Paso 2: Especificar Condiciones

1. **Conditions**
   - Click `Add` → `User Groups`
   - **Add Group**: `EMPRESARIOSX-L15`
   - Click `Add` → `Service-Type`
   - **Service-Type**: `Login` (SSH/Telnet)
   - Click `Next`

#### Paso 3: Especificar Acceso

1. **Access Permission**
   - Seleccionar: `Access granted`
   - Click `Next`

#### Paso 4: Configurar Atributos

1. **Constraints**
   - Dejar configuración por defecto
   - Click `Next`

2. **Settings**
   - En la sección **Advanced**, click `Configure`
   - Hacer clic en `Add`
   - **Vendor**: `Cisco`
   - **Attribute**: `shell:priv-lvl`
   - **Value**: `15`

   ⚠️ **IMPORTANTE**: Eliminar el atributo `Framed-Protocol` si está presente

   - Hacer clic en `Add` nuevamente
   - **Vendor**: `RADIUS Standard`
   - **Attribute**: `Service-Type`
   - **Value**: `Login`

   - Click `OK` para cerrar
   - Click `Next` → `Finish`

---

### Directiva 2: Empleados (Nivel 1)

**Repetir los pasos anteriores con los siguientes cambios:**

- **Policy Name**: `Cisco-User-L1`
- **Conditions**:
  - User Groups: `EMPRESARIOSY-L1`
  - Service-Type: `Login`
- **Cisco Attributes**:
  - `shell:priv-lvl` = `1`

---

## 4. Pruebas de Conectividad

### Verificar Registro del Cliente RADIUS

```powershell
# En PowerShell como Administrador
Get-NpsRadiusClient
```

Debería mostrar:
```
Address            : 192.168.10.1
FriendlyName       : Cisco-R1
SharedSecret       : R4d1usSecr3t2024!
```

### Prueba de Conectividad RADIUS

1. **Desde Windows 10 Pro** (cliente del dominio)
2. **SSH al router**:
   ```bash
   ssh empresario01@192.168.10.1
   Password: Empresario01!
   ```
3. **Verificar en NPS**:
   - `NPS (Local)` → `Accounting`
   - Se debe registrar el intento de autenticación

---

## 5. Solución de Problemas Comunes

### Problema 1: "This line may not run PPP"

**Causa**: Atributo `Framed-Protocol` establecido en `PPP`

**Solución**:
1. Abrir la Directiva en NPS
2. En **Settings** → **Advanced**
3. Buscar y eliminar `Framed-Protocol = PPP`
4. Dejar solo `Service-Type = Login`

### Problema 2: "Authentication Failed"

**Verificar**:
1. Shared Secret coincide en router y NPS
2. Usuario está en el grupo AD correcto
3. Servidor RADIUS está en línea: `ping 192.168.10.10`
4. Puertos RADIUS abiertos (1812, 1813)

```powershell
# Verificar puertos
Get-NetNetworkConfiguration -all | Where-Object {$_.port -eq 1812}
```

### Problema 3: Cuentas Locales no Funcionan como Fallback

**Verificar en Router**:
```cisco
R1# show aaa methods authentication
R1# show running-config | include aaa authentication
```

Debe mostrar:
```
aaa authentication login default group radius local
```

Si solo muestra `group radius`, agregar `local` al final.

---

## 6. Monitoreo y Auditoría

### Ver Eventos de Autenticación

```powershell
# Event Viewer
Get-EventLog -LogName "Security" -Source "IAS" -Newest 20

# O abrir Event Viewer manualmente
# Event Viewer > Windows Logs > Security
```

### Habilitar Logging Detallado en NPS

```powershell
# Registrar todas las solicitudes RADIUS
# En NPS Console: NPS > Accounting > Configure Logging

# Habilitar:
# - Authentication requests
# - Accounting requests
# - Periodic status

# Ubicación de logs:
# C:\Windows\System32\Logs\IAS\*.log
```

---

## 7. Configuración de Certificados (Opcional - para RADIUS/TLS)

```powershell
# Crear certificado auto-firmado
$cert = New-SelfSignedCertificate -DnsName "192.168.10.10" -CertStoreLocation "cert:\LocalMachine\My" -NotAfter (Get-Date).AddYears(5)

# En NPS Console:
# NPS > RADIUS Clients > Click derecho > Properties
# Marcar: "Use a certificate for the specified RADIUS client"
# Seleccionar el certificado creado
```

---

## 8. Checklist de Validación

- [ ] NPS instalado y ejecutándose
- [ ] Cliente RADIUS (Router) registrado
- [ ] Grupo EMPRESARIOSX-L15 creado en AD
- [ ] Grupo EMPRESARIOSY-L1 creado en AD
- [ ] Usuario empresario01 es miembro de EMPRESARIOSX-L15
- [ ] Usuario empleado02 es miembro de EMPRESARIOSY-L1
- [ ] Directiva "Cisco-Admin-L15" con shell:priv-lvl=15
- [ ] Directiva "Cisco-User-L1" con shell:priv-lvl=1
- [ ] Atributo Framed-Protocol eliminado de ambas directivas
- [ ] Shared Secret coincide en NPS y router
- [ ] Conectividad de red entre router (192.168.10.1) y NPS (192.168.10.10)
- [ ] Prueba SSH exitosa con ambos usuarios
