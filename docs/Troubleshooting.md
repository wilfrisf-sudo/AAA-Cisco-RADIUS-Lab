# Guía de Solución de Problemas

## 🔴 Problemas Comunes y Soluciones

### 1. Router No Reconoce Comando `radius-server`

**Síntoma:**
```
R1(config)# radius server NPS-SERVER
% Invalid input detected at '^' marker.
```

**Causa Raíz:**
La imagen Cisco IOSv 15.6(2)T requiere sintaxis moderna para RADIUS.

**Solución:**
Usar el bloque de configuración moderno:
```cisco
R1(config)# radius server NPS-SERVER
R1(config-radius-server)# address ipv4 192.168.10.10 auth-port 1812 acct-port 1813
R1(config-radius-server)# key R4d1usSecr3t2024!
```

---

### 2. Comandos AAA Rechazados

**Síntoma:**
```
R1(config)# aaa authentication login default group radius local
% AAA is not enabled
```

**Causa Raíz:**
El servicio AAA no está habilitado por defecto.

**Solución:**
Habilitar AAA antes de configurar políticas:
```cisco
R1(config)# aaa new-model
R1(config)# aaa authentication login default group radius local
```

---

### 3. "This line may not run PPP" Error

**Síntoma:**
```
ssh empresario01@192.168.10.1
% This line may not run PPP
Connection closed by foreign host.
```

**Causa Raíz:**
El servidor NPS está enviando el atributo `Framed-Protocol = PPP`, que es incompatible con conexiones SSH/terminal.

**Solución:**
1. Abrir NPS Console
2. Navegar a: `Network Policies` → Seleccionar directiva
3. Click `Properties` → `Settings` → `Advanced`
4. **Eliminar** el atributo `Framed-Protocol`
5. Asegurar que `Service-Type = Login` esté presente
6. Click OK

---

### 4. Bloqueo Total de Acceso a la Consola

**Síntoma:**
```
R1 con 0
% Authentication failed
Connection closed.
```

**Causa Raíz:**
La consola heredó las políticas RADIUS, que requieren servidor externo.

**Solución:**
Aplicar una política `none` exclusiva para la consola:
```cisco
R1(config)# aaa authentication login CONSOLE-NOAUTH none
R1(config)# line con 0
R1(config-line)# login authentication CONSOLE-NOAUTH
R1(config-line)# exit
```

Ahora presionar Enter en la consola sin requerir contraseña.

---

### 5. Fallo al Crear Certificado RDS vía GUI

**Síntoma:**
```
Error: Certificate creation failed
Error Code: 0x80070057 (E_INVALIDARG)
```

**Causa Raíz:**
Limitación en la GUI de Windows Server 2022 para crear certificados auto-firmados.

**Solución:**
Generar certificado vía PowerShell:
```powershell
$cert = New-SelfSignedCertificate -DnsName "192.168.10.10" `
    -CertStoreLocation "cert:\LocalMachine\My" `
    -NotAfter (Get-Date).AddYears(5) `
    -FriendlyName "RDS-Cert"

Write-Host "Certificado creado: $($cert.Thumbprint)"
```

Luego importar manualmente en RDS Manager.

---

### 6. Usuario No Recibe Privilegios de Nivel 15

**Síntoma:**
```
ssh empresario01@192.168.10.1
Password: Empresario01!
R1>  # Debería ser R1#
```

**Causa Raíz:**
Atributo Cisco `shell:priv-lvl` no está configurado o el usuario no está en el grupo correcto.

**Solución:**

1. **Verificar pertenencia a grupo AD:**
   ```powershell
   Get-ADGroupMember -Identity EMPRESARIOSX-L15
   ```

2. **Verificar configuración en NPS:**
   - Abrir directiva → `Settings` → `Advanced`
   - Confirmar: `Cisco Vendor Specific Attribute` → `shell:priv-lvl` = `15`

3. **Verificar recepción de atributo en router:**
   ```cisco
   R1# debug aaa authorization
   R1# debug radius
   # Intentar SSH y observar la salida
   ```

---

### 7. Conectividad entre Router y Servidor RADIUS

**Síntoma:**
```
ssh empresario01@192.168.10.1
% Authentication failed
# Y no cae al fallback local
```

**Causa Raíz:**
No hay conectividad entre router y NPS.

**Solución:**

1. **Desde el router:**
   ```cisco
   R1# ping 192.168.10.10
   Type escape sequence to abort.
   Sending 5, 100-byte ICMP Echos to 192.168.10.10, timeout is 2 seconds:
   !!!!!
   Success rate is 100 percent (5/5), round-trip min/avg/max = 1/2/4 ms
   ```

2. **Verificar puertos NPS:**
   ```powershell
   # En Windows Server
   netstat -ano | findstr /i "1812\|1813"
   # Debería mostrar estado LISTENING
   ```

3. **Verificar firewall:**
   ```powershell
   # En Windows Server
   Get-NetFirewallRule -DisplayName "*RADIUS*" | Select-Object DisplayName, Enabled
   ```

---

### 8. RDS/RemoteApp No Carga

**Síntoma:**
```
https://192.168.10.10/RDWeb/webclient
ERROR: Page cannot be displayed
```

**Causa Raíz:**
- IIS no está configurado correctamente
- Certificado SSL no es válido
- RD Web Client no está instalado

**Solución:**

1. **Verificar IIS en ejecución:**
   ```powershell
   Get-Service -Name W3SVC
   # Debe mostrar Status: Running
   ```

2. **Verificar RD Web Client:**
   ```powershell
   Get-Service -Name RDS*
   ```

3. **Reinstalar RD Web Client:**
   ```powershell
   Uninstall-RDWebClientManagement
   Install-RDWebClientManagement
   ```

4. **Verificar certificado:**
   ```powershell
   Get-ChildItem -Path Cert:\LocalMachine\My |
   Where-Object {$_.Subject -like "*192.168.10.10*"}
   ```

---

### 9. Cuenta Local No Funciona como Fallback

**Síntoma:**
```
ssh cisco@192.168.10.1
Password: class
% Authentication failed
# No debería fallar, debería usar cuenta local
```

**Causa Raíz:**
Configuración AAA no especifica `local` como fallback.

**Solución:**

```cisco
R1(config)# aaa authentication login default group radius local
# Nota: Incluir 'local' al final de la línea
```

Verificar:
```cisco
R1# show running-config | include "aaa authentication login default"
aaa authentication login default group radius local
```

---

### 10. Logs No se Registran en el Router

**Síntoma:**
```
R1# show logging
R1# show aaa statistics
# Salida vacía
```

**Causa Raíz:**
Logging no está habilitado o el buffer es insuficiente.

**Solución:**

```cisco
R1(config)# logging buffered 51200 debugging
R1(config)# aaa accounting exec default start-stop group radius
R1(config)# end
R1# write memory

# Verificar después de 5-10 minutos
R1# show logging
R1# show aaa statistics
```

---

## 🏠 Problemas Avanzados

### Debugging RADIUS

```cisco
# En el router:
R1# debug radius
R1# debug aaa authentication
R1# debug aaa authorization

# Realizar intento SSH
ssh empresario01@192.168.10.1

# Detener debug
R1# undebug all
R1# show logging
```

### Inspeccionar Tráfico RADIUS

```powershell
# Usando Wireshark en Windows Server
# Capturar en adaptador de red
# Filtro: radius
# Buscar atributos inesperados
```

---

## ✅ Checklist de Verificación

Antes de reportar un problema, verificar:

- [ ] Conectividad de red entre router y NPS: `ping`
- [ ] Shared Secret coincide en ambos extremos
- [ ] Usuario existe en Active Directory: `Get-ADUser`
- [ ] Usuario es miembro del grupo correcto: `Get-ADGroupMember`
- [ ] Servidor RADIUS está en línea: `Get-Service -Name IAS`
- [ ] NPS ha recibido la solicitud: Revisar logs en Event Viewer
- [ ] Router tiene `aaa new-model` habilitado
- [ ] Router tiene cuenta local de respaldo: `show running-config | include username`
- [ ] SSH está habilitado: `show ip ssh`
- [ ] Interfaz de red es accesible: `show ip interface brief`

---

## 📞 Recursos Adicionales

- **Cisco AAA Documentation**: https://www.cisco.com/c/en/us/support/security/aaa-identity-and-access/models.html
- **Microsoft NPS Documentation**: https://docs.microsoft.com/en-us/windows-server/networking/nps/nps-top
- **GNS3 Community**: https://gns3.com/community
