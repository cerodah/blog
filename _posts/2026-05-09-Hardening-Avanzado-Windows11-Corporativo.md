---
title: "Hardening Avanzado en Windows 11 Corporativo: Las Técnicas que Nadie Implementa"
published: true
---

Llevo un tiempo trabajando en entornos corporativos y siempre me sorprende lo mismo: las empresas invierten miles de euros en soluciones EDR, SIEM y otras siglas, pero tienen la configuración base de Windows prácticamente intacta. El antivirus actualizado, el firewall encendido, y a correr.

El problema es que Windows 11 tiene una cantidad brutal de mecanismos de seguridad avanzados que vienen **desactivados por defecto** o que requieren configuración explícita para funcionar como deberían. La mayoría de estos ni siquiera aparecen en las guías de hardening populares. En este artículo voy a repasar los que considero más críticos y menos conocidos, con ejemplos reales de cómo implementarlos.

---

## 1. LSA Protection (RunAsPPL): Blindar el Proceso que Guarda tus Credenciales

El proceso `lsass.exe` es el objetivo número uno de cualquier atacante que quiera volcar credenciales en memoria. Herramientas como Mimikatz dependen completamente de poder leer la memoria de ese proceso.

Windows tiene una protección para esto: **LSA Protected Process Light (PPL)**. Cuando está activo, lsass corre como proceso protegido y ningún proceso sin firma de Microsoft puede inyectarse o leerlo.

Lo curioso es que esto está desactivado por defecto en la mayoría de instalaciones corporativas.

**Cómo activarlo:**

```powershell
# Verificar el estado actual
Get-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Control\Lsa" -Name "RunAsPPL"

# Activar LSA Protection
Set-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Control\Lsa" -Name "RunAsPPL" -Value 1 -Type DWord

# Con UEFI Secure Boot activo, el valor debe ser 2 para protección completa
Set-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Control\Lsa" -Name "RunAsPPL" -Value 2 -Type DWord
```

O mediante Group Policy: `Computer Configuration > Windows Settings > Security Settings > Local Policies > Security Options > Local Security Authority (LSA) protection`.

**Verificación:**

Tras reiniciar, en el Visor de Eventos debería aparecer el evento **ID 12** en `Microsoft-Windows-Kernel-General`:

```
LSASS.exe was started as a protected process with level: 4
```

**Limitación real:** herramientas como PPLdump o PPLBlade pueden bypassear esto si el atacante tiene acceso físico o drivers firmados vulnerables. La protección adicional viene de combinarlo con HVCI (punto 2).

---

## 2. HVCI + Memory Integrity: El Hipervisor como Guardián

**Hypervisor-Protected Code Integrity (HVCI)** es probablemente la protección más potente que puedes activar y también una de las más ignoradas. Básicamente mueve la verificación de integridad del código fuera del kernel y la ejecuta dentro del hipervisor de Hyper-V, haciendo que sea prácticamente imposible para un rootkit modificar el kernel en tiempo de ejecución.

Combinado con LSA Protection, hace que volcar credenciales desde modo kernel sea una tarea extremadamente compleja.

**Cómo verificar si está activo:**

```powershell
Get-CimInstance -ClassName Win32_DeviceGuard -Namespace root\Microsoft\Windows\DeviceGuard | 
    Select-Object -Property VirtualizationBasedSecurityStatus, 
                             CodeIntegrityPolicyEnforcementStatus,
                             UsermodeCodeIntegrityPolicyEnforcementStatus
```

Un valor de `2` en `VirtualizationBasedSecurityStatus` significa que VBS está corriendo. Si ves `0` o `1`, no está activo.

**Activación por GPO:**

`Computer Configuration > Administrative Templates > System > Device Guard > Turn On Virtualization Based Security`

- Virtualization Based Security: **Enabled**
- Select Platform Security Level: **Secure Boot and DMA Protection**
- Virtualization Based Protection of Code Integrity: **Enabled with UEFI lock**
- Credential Guard Configuration: **Enabled with UEFI lock**

El UEFI lock es importante: significa que no se puede desactivar sin reiniciar y borrar la variable UEFI manualmente. Un atacante con acceso a OS no puede revertirlo.

**Impacto en rendimiento:** en hardware moderno (12ª gen Intel o Ryzen 5000+) el impacto es mínimo, alrededor de un 2-5%. En hardware antiguo puede ser significativo.

---

## 3. Windows Defender Application Control (WDAC): AppLocker lo hace mal

La mayoría de equipos de seguridad que implementan control de aplicaciones usan AppLocker. Es comprensible, lleva años en Windows y tiene una interfaz gráfica. El problema es que AppLocker tiene limitaciones serias: se puede bypassear desde procesos con privilegios de administrador, no controla drivers del kernel, y en Windows 11 enterprise está básicamente en modo mantenimiento.

**WDAC es el reemplazo real** y opera a nivel de kernel. Sus ventajas:

- Se aplica aunque el atacante tenga privilegios de administrador
- Controla qué drivers pueden cargarse (protección contra BYOVD - Bring Your Own Vulnerable Driver)
- Se puede implementar a través de Intune sin infraestructura on-prem
- Compatible con Managed Installer (marca aplicaciones instaladas por SCCM/Intune como confiables automáticamente)

**Crear una política WDAC básica:**

```powershell
# Generar política base en modo Audit (no bloquea, solo registra)
New-CIPolicy -Level Publisher -FilePath "C:\Policies\BasePolicy.xml" -UserPEs -MultiplePolicyFormat

# Convertir a formato binario
ConvertFrom-CIPolicy -XmlFilePath "C:\Policies\BasePolicy.xml" -BinaryFilePath "C:\Windows\System32\CodeIntegrity\SiPolicy.p7b"
```

**Política de bloqueo de drivers vulnerables conocidos:**

Microsoft mantiene una lista de drivers con vulnerabilidades conocidas. Para activar el bloqueo automático:

```powershell
# Descargar la lista oficial de Microsoft (HVCI Recommended Driver Block List)
$url = "https://aka.ms/VulnerableDriverBlockList"
# Esta lista bloquea drivers como gdrv.sys, RTCore64.sys, etc. usados en ataques BYOVD
```

La integración con **Smart App Control** (SAC) en W11 22H2+ añade una capa adicional de reputación basada en la nube para ejecutables desconocidos.

---

## 4. Bloquear NTLM y Forzar Kerberos: La Migración que Nadie Hace

NTLM lleva años siendo el objetivo preferido para ataques de Pass-the-Hash, NTLM relay, y captura de hashes con Responder. Kerberos es más seguro, pero la mayoría de empresas mantienen NTLM activo "por compatibilidad".

Windows 11 24H2 introduce **NTLM Auditing nativa** sin necesidad de soluciones de terceros, lo que permite saber exactamente qué aplicaciones y servicios aún dependen de NTLM antes de bloquearlo.

**Fase 1 - Auditoría (antes de bloquear nada):**

```
Computer Configuration > Windows Settings > Security Settings > 
Local Policies > Security Options

Network security: Restrict NTLM: Audit NTLM authentication in this domain
→ Enable all

Network security: Restrict NTLM: Audit Incoming NTLM Traffic  
→ Enable auditing for all accounts
```

Los eventos **4776** (autenticación NTLM) en el DC te dirán qué aplicaciones necesitan migración.

**Fase 2 - Bloqueo progresivo:**

```powershell
# Bloquear NTLM saliente desde estaciones de trabajo (menos impacto que bloquearlo en DCs)
Set-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Control\Lsa\MSV1_0" `
    -Name "RestrictSendingNTLMTraffic" -Value 2 -Type DWord
# Valor 2 = Deny all

# Bloquear NTLM entrante en servidores miembro
Set-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Control\Lsa\MSV1_0" `
    -Name "RestrictReceivingNTLMTraffic" -Value 2 -Type DWord
```

**Protección adicional: Protected Users Security Group**

Este grupo, que muy poca gente usa, aplica automáticamente estas restricciones a sus miembros:

- No puede autenticarse con NTLM (solo Kerberos)
- No puede usar DES o RC4 en Kerberos (solo AES)
- Las credenciales no se cachean localmente
- Los tickets Kerberos expiran a las 4 horas independientemente de la política

```powershell
# Añadir una cuenta al grupo Protected Users
Add-ADGroupMember -Identity "Protected Users" -Members "AdminDomain"

# Verificar quién está en el grupo
Get-ADGroupMember -Identity "Protected Users" | Select-Object Name, SamAccountName
```

**Importante:** antes de meter cuentas de servicio aquí, verificad que no dependen de delegación Kerberos no restringida o NTLM. Puede romper cosas.

---

## 5. Attack Surface Reduction (ASR): Las Reglas que Nadie Activa

Las reglas ASR de Microsoft Defender son específicas, granulares, y la mayoría de equipos de seguridad activan cuatro o cinco genéricas y se quedan ahí. Hay reglas muy específicas para técnicas de ataque concretas que apenas se mencionan en documentación estándar.

**Reglas críticas que suelen ignorarse:**

```powershell
# Bloquear el abuso de drivers de impresión vulnerables (PrintNightmare)
# GUID: 56a863a9-875e-4185-98a7-b882c64b5ce5
Set-MpPreference -AttackSurfaceReductionRules_Ids 56a863a9-875e-4185-98a7-b882c64b5ce5 `
    -AttackSurfaceReductionRules_Actions Enabled

# Bloquear creación de procesos desde WMI (técnica común de lateral movement)
# GUID: e6db77e5-3df2-4cf1-b95a-636979351e5b
Set-MpPreference -AttackSurfaceReductionRules_Ids e6db77e5-3df2-4cf1-b95a-636979351e5b `
    -AttackSurfaceReductionRules_Actions Enabled

# Bloquear procesos hijos desde Office (macro abuse)
# GUID: d4f940ab-401b-4efc-aadc-ad5f3c50688a
Set-MpPreference -AttackSurfaceReductionRules_Ids d4f940ab-401b-4efc-aadc-ad5f3c50688a `
    -AttackSurfaceReductionRules_Actions Enabled

# Bloquear inyección de código en procesos del sistema desde scripts
# GUID: 75668c1f-73b5-4cf0-bb93-3ecf5cb7cc84
Set-MpPreference -AttackSurfaceReductionRules_Ids 75668c1f-73b5-4cf0-bb93-3ecf5cb7cc84 `
    -AttackSurfaceReductionRules_Actions Enabled
```

**Verificar qué reglas están activas y su estado:**

```powershell
Get-MpPreference | Select-Object -ExpandProperty AttackSurfaceReductionRules_Ids
Get-MpPreference | Select-Object -ExpandProperty AttackSurfaceReductionRules_Actions
```

Los estados son: `0` = Disabled, `1` = Block, `2` = Audit, `6` = Warn.

**Consejo:** activar primero en modo Audit (valor `2`) durante 2 semanas y revisar los eventos **1121** y **1122** en el Visor de Eventos para identificar falsos positivos antes de pasar a Block.

---

## 6. PowerShell: Logging Completo y Constrained Language Mode

PowerShell sigue siendo el entorno de ejecución preferido de cualquier atacante en un entorno Windows. Living-off-the-land con PowerShell es prácticamente el estándar. Sin embargo, pocas empresas tienen el logging configurado correctamente.

**Script Block Logging + Module Logging + Transcription:**

```powershell
# Activar Script Block Logging (registra todo el código PS que se ejecuta, incluso ofuscado)
$path = "HKLM:\SOFTWARE\Policies\Microsoft\Windows\PowerShell\ScriptBlockLogging"
New-Item -Path $path -Force
Set-ItemProperty -Path $path -Name "EnableScriptBlockLogging" -Value 1
Set-ItemProperty -Path $path -Name "EnableScriptBlockInvocationLogging" -Value 1

# Module Logging (registra llamadas a módulos y sus parámetros)
$path2 = "HKLM:\SOFTWARE\Policies\Microsoft\Windows\PowerShell\ModuleLogging"
New-Item -Path $path2 -Force
Set-ItemProperty -Path $path2 -Name "EnableModuleLogging" -Value 1

# Transcription (genera archivos de log con toda la sesión)
$path3 = "HKLM:\SOFTWARE\Policies\Microsoft\Windows\PowerShell\Transcription"
New-Item -Path $path3 -Force
Set-ItemProperty -Path $path3 -Name "EnableTranscripting" -Value 1
Set-ItemProperty -Path $path3 -Name "EnableInvocationHeader" -Value 1
Set-ItemProperty -Path $path3 -Name "OutputDirectory" -Value "\\servidor\logs\pstranscripts"
```

Los Script Block Logs aparecen en: `Microsoft-Windows-PowerShell/Operational`, **Evento 4104**.

**Constrained Language Mode (CLM):**

CLM restringe PowerShell a un subconjunto seguro del lenguaje. Bloquea clases .NET arbitrarias, COM objects, y muchas técnicas de ejecución de código. Se puede forzar mediante WDAC o AppLocker:

```powershell
# Verificar el modo actual
$ExecutionContext.SessionState.LanguageMode
# FullLanguage = sin restricciones
# ConstrainedLanguage = modo restringido activo
```

Con WDAC configurado correctamente, cualquier script no firmado que intente correr en FullLanguage automáticamente cae a CLM. Esto rompe la mayoría de payloads PowerShell sin modificación.

---

## 7. LAPS v2 Nativo: Adiós al Legacy

Si todavía estáis usando el LAPS clásico (el que se instalaba aparte con la extensión de AD), Windows 11 22H2 + KB5025229 trae **Windows LAPS nativo** integrado en el sistema operativo. Las diferencias son importantes:

- Almacenamiento en **Azure AD / Entra ID** además de AD on-prem
- Soporte para **cifrado de contraseña** en AD (el legacy guardaba en texto claro en el atributo `ms-Mcs-AdmPwd`)
- **Historial de contraseñas** (guarda las últimas N contraseñas)
- **Account management**: puede gestionar cualquier cuenta local, no solo el Administrador

**Configuración básica mediante PowerShell:**

```powershell
# Comprobar si Windows LAPS está disponible
Get-Command Get-LapsAADPassword -ErrorAction SilentlyContinue

# Configurar política LAPS (requiere módulo LAPS)
Set-LapsADComputerSelfPermission -Identity "OU=Workstations,DC=empresa,DC=local"

# Ver la contraseña actual de un equipo
Get-LapsADPassword -Identity "PC-FINANZAS-01" -AsPlainText

# Forzar rotación inmediata de contraseña
Reset-LapsPassword -Identity "PC-FINANZAS-01"
```

**Migración desde LAPS legacy:** ambas versiones pueden coexistir temporalmente. El atributo nuevo es `msLAPS-Password` (cifrado) vs el antiguo `ms-Mcs-AdmPwd` (plaintext).

---

## Conclusión

Estas no son técnicas teóricas sacadas de un paper. Son configuraciones que se pueden implementar hoy, en producción, con las herramientas que ya tiene Windows 11. El problema habitual es que requieren planificación: activar HVCI sin verificar compatibilidad de drivers puede dejar equipos sin arrancar, y bloquear NTLM sin auditoría previa puede romper aplicaciones de negocio.

El orden que recomiendo para no montar un caos:

1. **Auditoría primero** — ASR en Audit, NTLM logging, PS transcription
2. **Quick wins** — LSA Protection, LAPS v2, Protected Users para cuentas admin
3. **Cambios con impacto** — HVCI, WDAC, bloqueo NTLM

Si queréis profundizar, las guías STIG de DISA para Windows 11 y el Microsoft Security Baseline son los documentos de referencia más completos que existen para esto. Nada de lo que he mencionado aquí está inventado — todo tiene fuente en documentación oficial de Microsoft.
