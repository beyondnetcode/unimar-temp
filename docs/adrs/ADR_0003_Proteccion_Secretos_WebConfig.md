# ADR-0003: Protección de Secretos y Cifrado de Cadenas de Conexión (Anti-Exposición)

**Status:** PROPOSED  
**Date:** 2026-05-11  
**Author:** ALBERTO ARROYO  
**Severity:** CRITICAL SECURITY (OWASP A02 / A05)  

---

## 1. RESUMEN EJECUTIVO

**Problema Crítico:**
Los archivos `Web.config` de las soluciones Extranet e Intranet contienen cadenas de conexión a la base de datos (`connectionStrings`) y claves de API (`appSettings`) en **texto plano**, incluyendo usuarios y contraseñas del sistema administrador. Esto viola directamente el principio de defensa en profundidad y expone la base de datos a compromisos masivos en caso de fuga del código fuente o acceso no autorizado al servidor.

**Decisión Recomendada:**
Implementar el cifrado a nivel de sistema operativo mediante el proveedor **RSA Protected Configuration** (`aspnet_regiis.exe`) para entornos On-Premise (Inmediato) y planificar la migración a un gestor de secretos centralizado (Vault) para la fase de modernización.

---

## 2. CONTEXTO TÉCNICO

### 2.1 Riesgo Detectado
Actualmente, cualquier usuario con permisos de lectura en la carpeta del IIS o acceso al repositorio de código puede ver credenciales productivas:
```xml
<!-- ESTADO ACTUAL (INSEGURO) -->
<connectionStrings>
  <add name="ConexionMain" connectionString="Data Source=SERVER_IP;Initial Catalog=BD;User ID=sa;Password=SuperSecretPass123!;" />
</connectionStrings>
```

### 2.2 Impacto
- **Compromiso total de BD:** Exfiltración de datos sensibles de clientes y operaciones.
- **Movimiento lateral:** Reutilización de credenciales en otros sistemas de la red interna.
- **Incumplimiento normativo:** Falla directa en auditorías de seguridad (PCI-DSS, ISO 27001).

---

## 3. ESTRATEGIA DE IMPLEMENTACIÓN (FASES)

### Fase 0: Contención Inmediata (Táctica)
Utilizar la funcionalidad nativa de .NET Framework para cifrar las secciones del `web.config` en el servidor IIS local. Esto no requiere cambios en el código fuente de C#.

### Fase 3+: Modernización Estratégica
Migrar a Azure Key Vault, HashiCorp Vault o el uso de variables de entorno gestionadas por pipelines CI/CD en la fase de arquitectura target.

---

## 4. GUÍA DE EJECUCIÓN (CÓMO HACERLO)

### 4.1 Paso 1: Rotación de Credenciales
Antes de cifrar, **DEBEN ROTARSE LAS CONTRASEÑAS**. Cifrar una contraseña comprometida no elimina el riesgo.
1. Generar una nueva contraseña robusta (20+ caracteres, alfanumérico + símbolos).
2. Actualizar la clave en el SQL Server.
3. Reemplazar temporalmente en el `Web.config` del servidor.

### 4.2 Paso 2: Cifrado con `aspnet_regiis`
Ejecutar este comando en la consola de comandos de Administrador en el servidor web de destino.

**Sintaxis:**
```cmd
REM Navegar a la ruta de .NET Framework 4.0 (64-bit)
cd C:\Windows\Microsoft.NET\Framework64\v4.0.30319

REM Cifrar sección connectionStrings (-pe = Protect Section)
REM -app es la ruta virtual en IIS, o -pef es la ruta física
aspnet_regiis.exe -pef "connectionStrings" "D:\Ruta\Fisica\De\Tu\SitioWeb"
```

### 4.3 Resultado Esperado en el Web.config
El archivo ya no expondrá la contraseña y se verá de la siguiente manera:
```xml
<connectionStrings configProtectionProvider="RsaProtectedConfigurationProvider">
  <EncryptedData Type="http://www.w3.org/2001/04/xmlenc#Element" xmlns="http://www.w3.org/2001/04/xmlenc#">
    <EncryptionMethod Algorithm="http://www.w3.org/2001/04/xmlenc#tripledes-cbc" />
    <KeyInfo xmlns="http://www.w3.org/2000/09/xmldsig#">
      <EncryptedKey xmlns="http://www.w3.org/2001/04/xmlenc#">
        <CipherData>
          <CipherValue>dF4GgH... [VALOR_CIFRADO_GIGANTE] ...</CipherValue>
        </CipherData>
      </EncryptedKey>
    </KeyInfo>
    <!-- ... data cifrada ... -->
  </EncryptedData>
</connectionStrings>
```

*💡 Nota: .NET descifra esto automáticamente en memoria durante la ejecución sin necesidad de cambiar ni una sola línea de código C#.*

---

## 5. CÓMO MANEJAR GRANJAS DE SERVIDORES (WEB FARMS)

Si hay balanceo de carga (múltiples servidores IIS), el cifrado por defecto no funcionará si se copia el archivo de un servidor a otro (porque usan llaves RSA locales distintas). Se debe exportar e importar el contenedor de claves.

**1. Crear contenedor de llaves a medida en el Servidor A:**
```cmd
aspnet_regiis -pc "ClaveUnimarCluster" -exp
```

**2. Asignar permisos al Application Pool en Servidor A:**
```cmd
aspnet_regiis -pa "ClaveUnimarCluster" "IIS AppPool\NombreTuAppPool"
```

**3. Exportar contenedor a archivo XML:**
```cmd
aspnet_regiis -px "ClaveUnimarCluster" "D:\temp\ClavesUnimar.xml" -pri
```

**4. Importar en Servidor B y C:**
```cmd
aspnet_regiis -pi "ClaveUnimarCluster" "D:\temp\ClavesUnimar.xml"
```

---

## 6. MEJORES PRÁCTICAS (BEST PRACTICES)

1. **Principio de Privilegio Mínimo:** El usuario de base de datos no debe ser `sa` (System Administrator). Debe tener un rol específico (ej: `db_datareader`, `db_datawriter`) limitado a las tablas requeridas.
2. **Uso de .gitignore:** Nunca subir archivos `Web.config` con claves reales al repositorio Git. Mantener un `Web.config.template` o usar transformaciones XML (`Web.Release.config`).
3. **Cifrado de appSettings:** Repetir el proceso de `aspnet_regiis` con la sección `appSettings` si contiene llaves de APIs, tokens o URLs sensibles.
4. **Respaldos Seguros:** Antes de cifrar, guarda una copia de seguridad del config encriptado y del descifrado en un entorno seguro (ej. 1Password corporativo o Key Vault), no en el servidor.

---

## 7. REFERENCIAS DE INVESTIGACIÓN Y TÉCNICAS

- **Microsoft Docs - Encrypting Configuration Information:** [How To: Encrypt Configuration Sections in ASP.NET 2.0 Using RSA](https://learn.microsoft.com/en-us/previous-versions/aspnet/dtkwfdky(v=vs.100))
- **OWASP Top 10 (A02: Cryptographic Failures):** [OWASP Reference on unprotected storage of secrets](https://owasp.org/Top10/A02_2021-Cryptographic_Failures/)
- **Machine Keys Configuration:** [MachineKey guidance for ASP.NET Web Forms / MVC](https://learn.microsoft.com/en-us/aspnet/web-forms/overview/deployment/visual-studio-web-deployment/deploying-a-database-update-without-redeploying-the-database)
- **RSA Key Containers Management:** Documentación oficial sobre administración de contenedores de claves para granjas web.

---

## 8. APROBACIÓN TÉCNICA

| Rol | Nombre | Firma | Fecha |
| :--- | :--- | :---: | :--- |
| **Lead Architect** | - | _ _ _ | 2026-05-11 |
| **Infrastructure Lead** | - | _ _ _ | 2026-05-11 |
| **Developer** | Edwin Aztuvilca | _ _ _ | 2026-05-11 |

---

## 9. CHANGELOG

| Versión | Fecha | Autor | Descripción de cambios |
| :--- | :--- | :--- | :--- |
| 1.0 | 2026-05-11 | ALBERTO ARROYO | Creación del ADR para la protección de secretos táctica en IIS. |
