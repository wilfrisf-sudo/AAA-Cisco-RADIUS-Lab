# Guía Completa del Laboratorio AAA/RADIUS

## 📋 Tabla de Contenidos

1. [Requisitos Previos](#requisitos-previos)
2. [Configuración de Windows Server 2022](#configuración-de-windows-server-2022)
3. [Configuración del Router Cisco](#configuración-del-router-cisco)
4. [Validación de Funcionamiento](#validación-de-funcionamiento)
5. [Solución de Problemas](#solución-de-problemas)

---

## Requisitos Previos

### Hardware Recomendado
- Procesador: Intel i7/Ryzen 7 o superior (quad-core mínimo)
- RAM: 16GB mínimo, 32GB recomendado
- Almacenamiento: 100GB libre en SSD
- Conexión a Internet

### Software Necesario
- GNS3 v2.2.4 o superior
- VirtualBox o Hyper-V
- Windows Server 2022 imagen ISO
- Windows 10 Pro imagen
- Cisco IOSv 15.6 imagen

---

## Configuración de Windows Server 2022

### Paso 1: Active Directory Domain Services

```powershell
# Instalar AD DS
Install-WindowsFeature AD-Domain-Services -IncludeManagementTools

# Crear dominio
Install-ADDSForest -DomainName empresa.local -DomainNetbiosName EMPRESA -SafeModeAdministratorPassword (ConvertTo-SecureString "P@ssw0rd123!" -AsPlainText -Force) -Force
```

### Paso 2: Crear Usuarios y Grupos

```powershell
# Crear OU
New-ADOrganizationalUnit -Name "EMPRESAX" -Path "DC=empresa,DC=local"

# Crear grupos
New-ADGroup -Name "EMPRESARIOSX-L15" -SamAccountName "EMPRESARIOSX-L15" -GroupScope Global -Path "OU=EMPRESAX,DC=empresa,DC=local"
New-ADGroup -Name "EMPRESARIOSY-L1" -SamAccountName "EMPRESARIOSY-L1" -GroupScope Global -Path "OU=EMPRESAX,DC=empresa,DC=local"

# Crear usuarios
New-ADUser -Name "Empresario01" -SamAccountName "empresario01" -UserPrincipalName "empresario01@empresa.local" -AccountPassword (ConvertTo-SecureString "Empresario01!" -AsPlainText -Force) -Enabled $true -Path "OU=EMPRESAX,DC=empresa,DC=local"

New-ADUser -Name "Empleado02" -SamAccountName "empleado02" -UserPrincipalName "empleado02@empresa.local" -AccountPassword (ConvertTo-SecureString "Empleado02!" -AsPlainText -Force) -Enabled $true -Path "OU=EMPRESAX,DC=empresa,DC=local"

# Asignar usuarios a grupos
Add-ADGroupMember -Identity "EMPRESARIOSX-L15" -Members "empresario01"
Add-ADGroupMember -Identity "EMPRESARIOSY-L1" -Members "empleado02"
```

### Paso 3: Configurar NPS (RADIUS)

```powershell
# Instalar NPS
Install-WindowsFeature NPAS -IncludeManagementTools

# Configuración se realiza vía GUI (ver docs/NPS-RADIUS-Configuration.md)
```

### Paso 4: Configurar IIS

```powershell
# Instalar IIS
Install-WindowsFeature Web-Server -IncludeManagementTools

# Crear sitio HTML
$htmlContent = @"
<!DOCTYPE html>
<html>
<head>
    <title>Sitio IIS - Laboratorio AAA</title>
    <style>
        body { font-family: Arial; text-align: center; margin-top: 50px; }
        .container { max-width: 600px; margin: 0 auto; background: #f0f0f0; padding: 20px; border-radius: 8px; }
    </style>
</head>
<body>
    <div class="container">
        <h1>✅ Autenticación RADIUS Exitosa</h1>
        <p>Accediste a través de RDS/RemoteApp con credenciales de Active Directory.</p>
        <hr>
        <p>Servidor: <strong>empresa.local</strong></p>
        <p>Dominio: <strong>EMPRESA</strong></p>
    </div>
</body>
</html>
"@

Set-Content -Path "C:\inetpub\wwwroot\index.html" -Value $htmlContent
```

### Paso 5: Configurar RDS/RemoteApp

```powershell
# Instalar RDS
Install-WindowsFeature RDS-RD-Server -IncludeManagementTools

# Ver docs/IIS-RDS-Configuration.md para configuración detallada
```

---

## Configuración del Router Cisco

### Configuración Base

```cisco
Router# configure terminal
Router(config)# hostname R1
R1(config)# ip domain-name empresa.local

R1(config)# interface GigabitEthernet0/0
R1(config-if)# ip address 192.168.10.1 255.255.255.0
R1(config-if)# no shutdown
R1(config-if)# exit

R1(config)# crypto key generate rsa modulus 2048
R1(config)# ip ssh version 2
```

### Configuración AAA y RADIUS

```cisco
R1(config)# username cisco privilege 15 secret class

R1(config)# aaa new-model

R1(config)# radius server NPS-SERVER
R1(config-radius-server)# address ipv4 192.168.10.10 auth-port 1812 acct-port 1813
R1(config-radius-server)# key R4d1usSecr3t2024!
R1(config-radius-server)# exit

R1(config)# aaa authentication login default group radius local
R1(config)# aaa authorization exec default group radius local
R1(config)# aaa accounting exec default start-stop group radius
R1(config)# aaa authentication login CONSOLE-NOAUTH none
```

### Accesos y Logging

```cisco
R1(config)# line vty 0 4
R1(config-line)# transport input ssh
R1(config-line)# login authentication default
R1(config-line)# authorization exec default
R1(config-line)# exit

R1(config)# line con 0
R1(config-line)# login authentication CONSOLE-NOAUTH
R1(config-line)# exit

R1(config)# logging buffered 51200 debugging
R1(config)# end
R1# write memory
```

---

## Validación de Funcionamiento

### Prueba 1: Autenticación SSH - Usuario Administrador

```bash
ssh empresario01@192.168.10.1
Password: Empresario01!

R1>
# Usuario es administrador (Nivel 15), debería ver el prompt con #
R1# show running-config
```

### Prueba 2: Autenticación SSH - Usuario Empleado

```bash
ssh empleado02@192.168.10.1
Password: Empleado02!

R1>
# Usuario es de lectura (Nivel 1), solo modo >>
R1> show running-config
% Permission denied
```

### Prueba 3: Fallback a Cuenta Local

```bash
# Detener NPS en Windows Server

ssh cisco@192.168.10.1
Password: class

R1# show running-config
# Acceso exitoso con cuenta local
```

### Prueba 4: Verificar Logs de Autenticación

```cisco
R1# show logging
R1# show aaa statistics
```

---

## Solución de Problemas

Ver `docs/Troubleshooting.md` para guía completa de resolución de problemas.
