# ADR-0004: Externalización de BLOBs y Mitigación de Over-fetching de Datos

**Status:** PROPOSED  
**Date:** 2026-05-11  
**Author:** ALBERTO ARROYO  
**Severity:** HIGH PERFORMANCE / SCALABILITY  

---

## 1. RESUMEN EJECUTIVO

**Problema Detectado:**
Las bases de datos actuales almacenan archivos pesados (PDFs, imágenes, adjuntos) directamente dentro de las tablas transaccionales utilizando tipos de datos `VARBINARY(MAX)` o `IMAGE` (ej: entidades como `TR_UNI_Archivo` o `TR_UNI_Reclamo_Doc`). Esto produce un fenómeno de **Over-fetching**, donde consultar un simple listado de registros "hidrata" innecesariamente megabytes de arreglos de bytes en memoria RAM, colapsando el ancho de banda entre la BD y el IIS, incrementando radicalmente el costo de almacenamiento y ralentizando los Backups de base de datos.

**Decisión:**
Adoptar una arquitectura de **Desacoplamiento de Datos no Estructurados**. Los contenidos binarios (Archivos) serán extraídos del motor relacional y movidos a un proveedor de almacenamiento externo, cuyas opciones tecnológicas deberán evaluarse en conjunto con Infraestructura (Azure Blob Storage, Network File Share / Azure Front Door). La base de datos únicamente almacenará metadatos y la **URL / URI de referencia** al archivo.

---

## 2. CONTEXTO TÉCNICO

### 2.1 Síntomas del Diseño Actual
Actualmente las consultas traen todo el objeto:
```sql
-- Antipatrón observado (Carga Eager de BLOBs)
SELECT ID, Fecha, NombreArchivo, ContenidoBinario FROM TR_UNI_Archivo;
```
Si la tabla tiene 1000 registros con PDFs de 5MB cada uno, una sola consulta arrastra **5 Gigabytes** de datos a través de la red, causando un consumo instantáneo de RAM de la aplicación que satura el Application Pool de IIS.

### 2.2 Impacto Operativo
- **Inflamiento de Backups (Database Bloat):** El 80% del peso de la base de datos son archivos binarios, no transacciones.
- **Pérdida de Caché de SQL Server:** Las páginas de datos binarios desplazan las páginas de índices críticos de la memoria RAM del motor SQL.
- **Timeouts Críticos:** Descarga extremadamente lenta en dispositivos de baja velocidad.

---

## 3. DISEÑO PROPUESTO (CÓMO HACERLO)

### 3.1 Rediseño de Base de Datos
Transición gradual añadiendo la columna de enlace y manteniendo la binaria como opcional durante la migración.

| Antes | Después (Transicional) | Después (Final Target) |
| :--- | :--- | :--- |
| `IdArchivo INT` | `IdArchivo INT` | `IdArchivo INT` |
| `Nombre VARCHAR(200)` | `Nombre VARCHAR(200)` | `Nombre VARCHAR(200)` |
| `Contenido VARBINARY(MAX)` | `Contenido VARBINARY(MAX) NULL` | -- ELIMINADO -- |
| | `UrlArchivo VARCHAR(500) NULL` | `UrlArchivo VARCHAR(500) NOT NULL` |
| | `StorageProvider VARCHAR(50) NULL`| `StorageProvider VARCHAR(50)` |

### 3.2 Flujo Lógico de Aplicación
1. **Lectura:** La aplicación web genera un enlace directo al Storage (o un Token SAS / CDN link firmado) y el navegador del cliente descarga el archivo directamente, sin pasar por el servidor de base de datos ni por el IIS de la app.
2. **Escritura:** El controlador web sube el `Stream` al Storage utilizando el adaptador inyectado, recibe la URL, y guarda únicamente la URL en el SQL Server.

### 3.3 Opciones de Capa de Almacenamiento (Stacks Evaluados)
- **Stack A: Cloud Nativo:** Azure Blob Storage. Máxima durabilidad y redundancia nativa.
- **Stack B: On-Premise Legacy:** Windows Shared Folder (UNC Path) mapeado en una SAN local + Virtual Directory en IIS expuesto como servidor de medios estáticos.
- **Stack C: Modern Cloud Azure:** Azure Blob Storage + Azure Front Door / Azure CDN. Maximiza el rendimiento de entrega mediante caché en el borde (Edge) y protege el storage primario mediante reglas de firewall WAF empresarial de Microsoft.

---

## 4. EJEMPLO DE IMPLEMENTACIÓN (MIGRADOR / C#)

A continuación, se presentan los patrones de diseño desacoplados para los distintos proveedores de almacenamiento evaluados. La aplicación inyectará la implementación concreta según el resultado de la sesión con Infraestructura.

#### A. Interfaz del Servicio (Business Layer)
```csharp
public interface IStorageService
{
    /// <summary>
    /// Sube archivo y retorna la URL pública o con SAS token
    /// </summary>
    Task<string> UploadFileAsync(Stream content, string fileName, string contentType);
}
```

#### B. Implementación Opcional 1: Azure Blob Storage
```csharp
using Azure.Storage.Blobs;
using Azure.Storage.Blobs.Models;

public class AzureBlobStorageService : IStorageService
{
    private readonly string _connectionString;
    private readonly string _containerName = "unimar-adjuntos";

    public AzureBlobStorageService(string connectionString)
    {
        _connectionString = connectionString;
    }

    public async Task<string> UploadFileAsync(Stream content, string fileName, string contentType)
    {
        var blobServiceClient = new BlobServiceClient(_connectionString);
        var containerClient = blobServiceClient.GetBlobContainerClient(_containerName);
        string uniqueName = $"{Guid.NewGuid()}_{fileName}";
        var blobClient = containerClient.GetBlobClient(uniqueName);

        var blobHttpHeader = new BlobHttpHeaders { ContentType = contentType };
        await blobClient.UploadAsync(content, new BlobUploadOptions { HttpHeaders = blobHttpHeader });

        return blobClient.Uri.ToString();
    }
}
```

#### C. Implementación Opcional 2: File Server On-Premise (Red Compartida)
```csharp
using System.IO;

public class LocalNetworkStorageService : IStorageService
{
    private readonly string _baseNetworkPath = @"\\fileserver01\Shared\UnimarDocs";
    private readonly string _publicBaseUrl = "https://static.unimar.com.pe/docs/";

    public async Task<string> UploadFileAsync(Stream content, string fileName, string contentType)
    {
        string uniqueName = $"{Guid.NewGuid()}_{fileName}";
        string fullDestPath = Path.Combine(_baseNetworkPath, uniqueName);

        // Copiar Stream a la ruta UNC de red (Asíncrono)
        using (var fileStream = new FileStream(fullDestPath, FileMode.Create, FileAccess.Write, FileShare.None, 4096, true))
        {
            await content.CopyToAsync(fileStream);
        }

        // Retorna la URL pública mapeada en el IIS virtual directory
        return _publicBaseUrl + uniqueName;
    }
}
```

#### D. Implementación Opcional 3: Azure Blob + Azure Front Door / CDN (Alto Rendimiento)
**Dependencia NuGet:** `Azure.Storage.Blobs`, `Azure.Storage.Sas`

Esta opción combina el almacenamiento base de Azure con la red de distribución (Edge) de Microsoft para optimizar descargas y securizar el acceso sin exponer el Storage primario.

```csharp
using Azure.Storage.Blobs;
using Azure.Storage.Sas;

public class AzureFrontDoorStorageService : IStorageService
{
    private readonly string _connectionString;
    private readonly string _frontDoorHost = "https://unimar-cdn.azurefd.net"; // Endpoint de Front Door

    public AzureFrontDoorStorageService(string connectionString)
    {
        _connectionString = connectionString;
    }

    public async Task<string> UploadFileAsync(Stream content, string fileName, string contentType)
    {
        var blobServiceClient = new BlobServiceClient(_connectionString);
        var containerClient = blobServiceClient.GetBlobContainerClient("adjuntos-privados");
        string uniqueName = $"{Guid.NewGuid()}_{fileName}";
        var blobClient = containerClient.GetBlobClient(uniqueName);

        // Subida al Origen
        await blobClient.UploadAsync(content);

        // Generación de Token SAS para descarga segura a través del CDN (Válido 15 min)
        BlobSasBuilder sasBuilder = new BlobSasBuilder()
        {
            BlobContainerName = "adjuntos-privados",
            BlobName = uniqueName,
            Resource = "b",
            ExpiresOn = DateTimeOffset.UtcNow.AddMinutes(15)
        };
        sasBuilder.SetPermissions(BlobSasPermissions.Read);

        string sasToken = blobClient.GenerateSasUri(sasBuilder).Query;

        // Retorna la URL mapeada al FRONT DOOR o CDN en lugar del host primario de Azure
        return $"{_frontDoorHost}/adjuntos-privados/{uniqueName}{sasToken}";
    }
}
```

#### E. Uso en Controller / BLL (Cualquier Implementación)
```csharp
// En lugar de guardar byte[] archivoEnMemoria en el SP
string fileUrl = await _storageService.UploadFileAsync(archivoHttp.InputStream, archivoHttp.FileName, archivoHttp.ContentType);

// Guardar en BD sólo la metadata y el enlace
await _repo.InsertarRegistroMetadata(idReclamo, fileUrl);
```

---

## 5. ESTRATEGIA DE MIGRACIÓN DE DATOS (ETL)

No se requiere un "Big Bang" de desconexión. Se puede migrar progresivamente:

1. **Implementar Escritura Dual (Día 1):** La lógica de la aplicación debe habilitar retrocompatibilidad; nuevos archivos van al Storage elegido, pero se siguen sirviendo los viejos directamente de BD si `UrlArchivo` es nula.
2. **Requisito de Producción - Desarrollo de Batch Migrador:** Dado que el aplicativo ya opera en producción, es **MANDATORIO** desarrollar una utilidad ejecutable (Consola .NET o Script PowerShell) que recorra secuencialmente los registros históricos, extraiga el BLOB binario, lo transmita al Storage Service definido y actualice la base de datos con la nueva URL. 
3. **Limpieza y Liberación de Espacio (Post-Migración):** Una vez validado el 100% de la migración de producción, ejecutar `TRUNCATE` por lotes sobre la columna histórica y realizar un Shrink de Base de Datos controlado para recuperar el almacenamiento físico consumido.

---

## 6. COORDINACIÓN CON INFRAESTRUCTURA (BLOQUEANTE DE DISEÑO)

⚠️ **TAREA OBLIGATORIA ANTES DE CODE-FREEZE:**

El equipo de Desarrollo debe agendar una sesión de trabajo con el Coordinador de Infraestructura para determinar la opción más simple y disponible actualmente en UNIMAR, facilitando la decisión final sobre la tecnología de File Server a implementar y validando las siguientes definiciones técnicas:

1. **Disponibilidad Cloud:** Validar si Unimar cuenta con suscripción de Azure activa o si la política de datos exige On-Premise.
2. **Infraestructura de Red:** En caso de File Server local, definir la ruta UNC y asegurar que el **IIS AppPool Identity** tenga permisos de lectura/escritura (`NTFS Modify`) en dicha carpeta.
3. **Exposición Web:** Determinar si se configurará un subdominio estático (ej: `archivos.unimar.com.pe`) mapeando el Storage o si se utilizará una capa CDN frente a él (ej: Azure Front Door / CDN estándar) para cachear imágenes y reducir la carga al servidor principal.
4. **Aislamiento de Tráfico:** Evaluar si la transferencia del storage viaja por red interna (VPN/ExpressRoute) para no saturar el peering de internet durante las cargas masivas.

---

## 7. REFERENCIAS TÉCNICAS Y BUENAS PRÁCTICAS

### 6.1 Referencias
- **Patrón Valet Key:** [Microsoft Architecture Center - Valet Key Pattern](https://learn.microsoft.com/en-us/azure/architecture/patterns/valet-key) para permitir a los clientes descargar directo del storage de forma segura.
- **Azure Blob Storage docs:** [.NET SDK Introduction for Blob service](https://learn.microsoft.com/en-us/azure/storage/blobs/storage-quickstart-blobs-dotnet).
- **SQL Server Blob Storage vs FileStream:** Análisis comparativo de Microsoft sobre cuándo mover datos fuera de tablas SQL.

### 6.2 Mejores Prácticas (Security First)
1. **Uso de Contenedores Privados:** Los archivos confidenciales (ej. facturas) NO deben estar en contenedores de acceso público. Usar **Shared Access Signatures (SAS)** con una vigencia de 5 minutos para la visualización.
2. **Nombres Deterministas:** Guardar archivos con estructura `YYYY/MM/DD/guid_nombre.ext` para optimizar la indexación lógica en el almacenamiento.
3. **Backups Habilitados:** Habilitar el "Soft Delete" en Azure Storage para proteger los archivos de borrados accidentales sin depender de los backups de la BD.

---

## 8. APROBACIÓN TÉCNICA

| Rol | Nombre | Firma | Fecha |
| :--- | :--- | :---: | :--- |
| **Lead Architect** | - | _ _ _ | 2026-05-11 |
| **DBA / Lead Data** | - | _ _ _ | 2026-05-11 |
| **Developer** | Edwin Aztuvilca | _ _ _ | 2026-05-11 |

---

## 9. CHANGELOG

| Versión | Fecha | Autor | Descripción de cambios |
| :--- | :--- | :--- | :--- |
| 1.0 | 2026-05-11 | ALBERTO ARROYO | Creación del ADR para la externalización de archivos y optimización de BD. |
| 1.1 | 2026-05-11 | ALBERTO ARROYO | Se agregan opciones On-Premise, CDN y tarea de coordinación con Infra. |
