---
layout: post
title:  "Exportar base de datos SQL SERVER a un BLOB en Azure como backup"
date:   2013-06-13
categories: [Sql Server, Azure]
tags: [Api REST, Azure, Bases de datos, DAC,Seguridad, Sql Server, Worker Role]
---

Decir que **Azure**, la plataforma Cloud de **Microsoft**, es una de las más potentes del mercado pese a su corto tiempo de vida no es algo nuevo aunque, precisamente por su juventud, nos encontramos con **algunas carencias** que hacen algo más complicado las gestiones típicas de un sistema en producción.

Una de estas carencias es la **posibilidad de automatizar backups** (copias de seguridad) de nuestras bases de datos **SQL SERVER** algo que necesitamos tener cuando nuestro proyecto pasa el estado de desarrollo.

La única posibilidad que nos ofrece Azure en este momento (25/06/2013) es **exportar/importar** nuestras bases de datos a un blob desde el panel de control a petición pero no podemos programar esta tarea para que se ejecute periódicamente.

![](http://res.cloudinary.com/escapistasclub/image/upload/v1468395047/Compilando/azure-sql1.png)

Esta opción genera en uno de nuestros **containers** un **blob** con extensión **.bacpac** con la copia de seguridad de la base de datos escogida para poder importarla en cualquier momento.

La pregunta es, ¿cómo podemos hacer esto automáticamente? La respuesta, con una aplicación que llame periódicamente a la misma **API REST** que usa Azure para estas tareas.

## Azure Sql Server API REST

Azure Sql Server expone un API REST con algunas tareas básicas que podemos invocar (con las debidas credenciales, claro está) para exportar o importar nuestras bases de datos contra Azure Storage.

Cada una de las regiones de Azure Sql Server tiene su propia url del servicio por lo que deberás asegurarte de usar la url correcta en función de en que región se encuentre tu base de datos:

| DataCenter       | EndPoint                                            |
|------------------|-----------------------------------------------------|
| North Central US | https://ch1prod-dacsvc.azure.com/DACWebService.svc/ |
| South Central US | https://sn1prod-dacsvc.azure.com/DACWebService.svc/ |
| North Europe     | https://db3prod-dacsvc.azure.com/DACWebService.svc/ |
| West Europe      | https://am1prod-dacsvc.azure.com/DACWebService.svc/ |
| East Asia        | https://hkgprod-dacsvc.azure.com/DACWebService.svc/ |
| Southeast Asia   | https://sg1prod-dacsvc.azure.com/DACWebService.svc/ |
| East US          | https://bl2prod-dacsvc.azure.com/DACWebService.svc/ |
| West US          | https://by1prod-dacsvc.azure.com/DACWebService.svc/ |

Las dos tareas expuestas por este servicio son “**Export**” e “**Import**” (para exportar e importar respectivamente) por lo que, por ejemplo, para solicitar una exportación de una base de datos alojada en el Norte de Europa deberíamos hacer un **POST** contra la url: `https://db3prod-dacsvc.azure.com/DACWebService.svc/Export`

## Solicitando Exportación desde código

Para automatizar el proceso de exportación podemos programar un pequeño **Worker Role** de Azure, un **servicio de Windows** que se ejecute en uno de nuestros servidores locales o incluso que lo implemente nuestra propia **web de administración**. En mi caso dispongo de un Worker Role propio con varias tareas de mantenimiento que se ejecutan cada cierto tiempo por lo que me fue muy fácil incluir una nueva tarea para hacer este backup.

Para hacer esta exportación, como hemos comentados antes, basta con hacer un **POST** contra la **[url del servicio] + “/Export”** incluyendo en el cuerpo de la petición un **xml** con los datos necesarios para la tarea.

Este xml que tenemos que postear tiene el siguiente formato:

```xml
<ExportInput xmlns="http://schemas.datacontract.org/2004/07/Microsoft.SqlServer.Management.Dac.ServiceTypes" xmlns:i="http://www.w3.org/2001/XMLSchema-instance">  
    <BlobCredentials i:type="BlobStorageAccessKeyCredentials"> 
        <Uri>[URL del Blob a crear]</Uri>  
        <StorageAccessKey>[Clave de acceso del Azure Storage]</StorageAccessKey>  
    </BlobCredentials>  
    <ConnectionInfo>  
        <DatabaseName>[Nombre de la base de datos a exportar]</DatabaseName>  
        <Password>[Password del usuario de la base de datos]</Password>  
        <ServerName>[Nombre del servidor en el que se encuentra la base de datos]</ServerName>  
        <UserName>[Usuario de acceso a la base de datos]</UserName>  
    </ConnectionInfo>  
</ExportInput>
```

Los parámetros que debes rellenar son:

*   **[URL del Blob a crear]**
    *   Url completa del blob que vas a crear con “.bacpac” al final.
        *   Ej:_ https://mistorageenazure.blob.core.windows.net/bak/backup-2013-06-25_09-49.bacpac_
*   **[Clave de acceso del Azure Storage]**
    *   La “Primary Access Key” de nuestro Storage en Azure. La podemos encontrar en el panel de administración de Azure > Storage > [el storage al que queremos acceder] > Manage Access Keys
*   **[Nombre del servidor en el que se encuentra la base de datos]**
    *   Nombre completo del servidor de SQL SERVER en el que se encuentra la base de datos a exportar (debe terminar en “database.windows.net”)
        *   Ej: _server.database.windows.net_
*   **Datos de la BD**
    *   Nombre, usuario y contraseña de la base de datos a exportar

Con este xml generado ya solo necesitamos hacer un **POST** contra la url del servicio para solicitar la tarea de exportación.

Este servicio nos devuelve un **GUID** para la posterior **comprobación del estado de la tarea**. Podemos guardarlo si queremos para poder saber si ha terminado con éxito o no.

Aquí un ejemplo con el código completo:

```c#
private Guid RequestExport(string databaseName, string containerUrl, string storageAccessKey, string blobName) {
	// Método que devolvería la URL del Servicio en función de la Región. Yo le paso un tipo enum propio por comodidad  
	var managementUrl = GetAzureManagementUrlByRegion(AzureRegions.NorthEurope);
	var request = WebRequest.Create(managementUrl + "/Export");
	request.Method = "POST";
	var dataStream = request.GetRequestStream();
	// Método que genera el XML que se le pasa como Body con el formato comentado anteriormente  
	var body = GetExportToBlobBodyRequest(blobUrl, storageAccessKey, databaseName);
	var utf8 = new UTF8Encoding();
	var buffer = utf8.GetBytes(body);
	dataStream.Write(buffer, 0, buffer.Length);
	dataStream.Close();
	request.ContentType = "application/xml";
	// La respuesta HTTP contiene el GUID que representa la tarea serializado como XML  
	using (var response = request.GetResponse()) {
		var encoding = Encoding.GetEncoding(1252);
		using (var responseStream = new StreamReader(response.GetResponseStream(), encoding)) {
			using (var reader = XmlDictionaryReader.CreateTextReader(responseStream.BaseStream, new XmlDictionaryReaderQuotas())) {
				var serializer = new DataContractSerializer(typeof(Guid));
				return (Guid)serializer.ReadObject(reader, true);
			}
		}
	}
}
```


**¡Listo!** Nuestra tarea de exportación ya se está lanzando y, si todo funciona como debe, en breve se creará un nuevo blob con el contenido de nuestra base de datos para poder importarlo en caso de catástrofe o para replicarlo en otra base de datos distinta para hacer pruebas.

## Comprobando el estado de la tarea

Si queremos saber si todo ha ido bien o ha fallado algo podemos hacer GET a otro método del **API REST** de Azure llamado “**Status**” indicando el GUID obtenido en la petición de la tarea. Para ello basta con hacer **GET** de la siguiente url:

`[Url del servicio API REST de tu Región]/Status?servername=[servidor de base de datos completo]&username=[usuario de la base de datos]&password=[password de la base de datos]&reqId=[GUID de la tarea]`

Esta url nos debe devolver un **XML** con información del estado de la tarea.

¡Hecho! Con un **simple cliente HTTP** podemos hacer **exportaciones e importaciones periódicas y automáticas** de nuestra base de datos y así protegernos ante cualquier problema que podamos tener.

Como siempre para cualquier duda ya sabéis dónde podéis encontrarme.

**¡Un saludo y nos vemos Compilando!**