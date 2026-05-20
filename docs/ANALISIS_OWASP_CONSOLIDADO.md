# Análisis de Seguridad Consolidado (OWASP Top 10)

Este documento mapea los hallazgos técnicos y arquitectónicos descubiertos en la auditoría de los sistemas web de Unimar (Intranet y Extranet) contra el estándar de seguridad **OWASP Top 10 (2021)**. Este análisis sirve como línea base para priorizar el hardening y medir el riesgo de ciberseguridad actual de la organización.

## Estado General de Cumplimiento

Actualmente, **ambos ecosistemas (Intranet y Extranet) presentan violaciones críticas en 8 de las 10 categorías principales de OWASP**, siendo la más grave el uso prolongado de componentes vulnerables y desactualizados, lo que actúa como un multiplicador de riesgo para el resto de las vulnerabilidades.

---

## Mapeo de Vulnerabilidades OWASP Top 10

### 🔴 A01:2021 – Broken Access Control (Control de Acceso Roto)
*   **Hallazgo:** Autorización débil y no centralizada. Falta de atributos `[Authorize]` consistentes en controladores y delegación excesiva de validación de permisos a lógica manual o procedimientos almacenados.
*   **Impacto:** Escalación de privilegios; usuarios de Intranet podrían acceder a flujos no permitidos, o usuarios de Extranet podrían manipular parámetros en la URL/API para ver datos de otros clientes (IDOR).
*   **Remediación:** Implementar un proveedor de identidad centralizado (SSO/JWT) y forzar autorización global en MVC/WebApi mediante filtros base.

### 🔴 A02:2021 – Cryptographic Failures (Fallas Criptográficas)
*   **Hallazgos:**
    1.  **Transporte Inseguro:** Integraciones WCF y navegación interna usando `http://` en texto plano.
    2.  **Crisis TLS:** Uso de IIS 2008 en Extranet, el cual no soporta TLS 1.2/1.3 robusto, exponiendo a ataques MITM (Man-in-the-Middle) y causando caídas de clientes móviles modernos.
    3.  **Secretos Expuestos:** Cadenas de conexión a base de datos con usuarios y contraseñas en texto plano en archivos `Web.config`.
*   **Impacto:** Exfiltración total de la base de datos (por credenciales en repo) o robo de credenciales en tránsito.
*   **Remediación:** Migrar a IIS 10+, forzar HTTPS estricto (HSTS) y usar un Vault o *MachineKeys* para encriptar secretos en configuración.

### 🔴 A03:2021 – Injection (Inyección)
*   **Hallazgos:** Uso de `validateRequest="false"` en vistas y controladores de Intranet sin filtros sanitizadores complementarios, abriendo la puerta a inyecciones XSS (Cross-Site Scripting). Aunque se usan Procedimientos Almacenados (mitigando SQLi), el uso de DTOs anchos expone inyecciones de datos no intencionadas (Mass Assignment).
*   **Impacto:** Ejecución de scripts maliciosos en el navegador de los usuarios o manipulación de estado.
*   **Remediación:** Implementar **Virtual Patching mediante WAF** en Fase 0 para bloquear vectores de inyección conocidos. Habilitar la validación estricta de ASP.NET, utilizar librerías como *AntiXSS* y sanear todo input de usuario. Aislar los DTOs de lectura vs escritura.

### 🔴 A04:2021 – Insecure Design (Diseño Inseguro)
*   **Hallazgos:** Ausencia sistémica de controles Anti-CSRF (Cross-Site Request Forgery). No se implementa el token `[ValidateAntiForgeryToken]` en transacciones críticas (formularios POST).
*   **Impacto:** Un atacante puede forzar a un usuario autenticado a ejecutar acciones no deseadas (ej. borrar registros, cambiar configuraciones) a través de enlaces maliciosos.
*   **Remediación:** Inyectar tokens CSRF en todas las vistas Razor transaccionales y validar el atributo en los controladores POST.

### 🔴 A05:2021 – Security Misconfiguration (Configuración de Seguridad Incorrecta)
*   **Hallazgos:**
    1.  Extranet con `debug="true"` activado en configuración base, lo que expone stack traces detallados al usuario final ante errores.
    2.  Cookies de autenticación y sesión generadas sin las banderas `Secure` y `HttpOnly` / `SameSite`.
    3.  Duplicidad de proyectos web en Extranet, indicando falta de despliegue automatizado.
*   **Impacto:** Secuestro de sesiones (Session Hijacking) e inteligencia para atacantes gracias a los rastros de errores detallados.
*   **Remediación:** Deshabilitar debug en producción (`<compilation debug="false">`), configurar custom errors, y endurecer la emisión de cookies a través del `web.config` y código.

### 🔴 A06:2021 – Vulnerable and Outdated Components (Componentes Vulnerables y Desactualizados)
*   **Hallazgos:** **[EL RIESGO MAYOR DEL SISTEMA]**
    *   Sistemas operativos y servidores EOL (Windows Server 2008, IIS 2008).
    *   Framework Core obsoleto (.NET 4.5, ASP.NET MVC 4 / WebApi 4 sin soporte oficial).
    *   Librerías con CVEs críticos sin parchar (SharpZipLib 0.86, Newtonsoft 4.5/5.0, jQuery 2.2, Bootstrap 3).
*   **Impacto:** Permite a atacantes explotar vulnerabilidades de dominio público automatizadas (ej. metasploit) logrando ejecución remota de código (RCE) o denegación de servicio (DoS).
*   **Remediación:** Desplegar protección de **WAF** como mitigación temporal, e integrar un análisis **SCA (Software Composition Analysis)** en el pipeline. Ejecutar el plan por oleadas de actualización de librerías, y reescribir/migrar hacia .NET 8 como objetivo de largo plazo.

### 🔴 A07:2021 – Identification and Authentication Failures (Fallas de Identidad y Autenticación)
*   **Hallazgos:** Sistemas reinventan la rueda en autenticación, creando tokens en base de datos de manera custom en vez de apoyarse en estándares robustos (ej. OAuth 2.0 u OpenID Connect).
*   **Impacto:** Posible predicción de tokens, suplantación de identidad y falta de controles de fuerza bruta centralizados.
*   **Remediación:** Centralizar la autenticación corporativa en un proveedor de identidad moderno (Azure AD, Keycloak, etc.).

### 🔴 A09:2021 – Security Logging and Monitoring Failures (Fallas en el Registro y Monitoreo de Seguridad)
*   **Hallazgos:** Práctica extendida de capturar excepciones de manera silenciosa (`catch {}` vacíos). Ausencia de correlación de logs de auditoría por usuario/transacción. Inexistencia de telemetría de fallos.
*   **Impacto:** Las brechas de seguridad o caídas operativas no son detectadas hasta que el usuario las reporta. Imposibilidad de realizar análisis forense ante una intrusión.
*   **Remediación:** Implementar registro estructurado global (ej. Log4net con appenders centrales) y remover bloques catch vacíos para permitir que el manejador de errores global capture el evento.

---

## Conclusión Ejecutiva de Seguridad

El nivel de riesgo acumulado **supera el umbral aceptable para un sistema expuesto a internet (Extranet)**. La adopción del plan de remediación en Fases (iniciando por la **Fase 0 de Contención** que resuelve el A02 y A06 a nivel servidor) es mandatoria para evitar la paralización operativa y blindar la información confidencial de la corporación contra exfiltraciones masivas o ataques de interrupción de negocio.
