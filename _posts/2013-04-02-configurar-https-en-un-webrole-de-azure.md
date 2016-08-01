---
layout: post
title:  "Configurar HTTPS en un WebRole de Azure"
date:   2013-04-02
categories: [Azure]
tags: [Azure, Certificate, PFX,ServiceDefinition, SSL,Tutorial,WebRole]
---

Parece mentira pero algo tan sencillo como permitir el acceso por **HTTPS** con un **dominio personalizado** en **Azure** es algo que no es trivial.

Para empezar, a día de hoy _2 de Abril del 2013_, **no puedes usar certificados de seguridad** en un **Web Site** de **Azure** si no estás usando su dominio “azurewebsites.net” por lo que nos hemos visto obligados a mover la web a un **WebRole** el cual tiene menos opciones de deployment y es más complejo que el Web Site solo para poder poner habilitar el acceso a https://www.midominio.com… lamentable.

Además **el proceso completo** para poder pedir, instalar y configurar el certificado de seguridad **no es nada sencillo** así que, para todos los que se vayan a pelear con este tema hasta que lo mejoren, les dedico este artículo.

# Crear la solicitud del Certificado

Para poder configurar nuestro certificado de seguridad en Azure **necesitamos conocer la clave privada con la que se generó** la petición del certificado. Por este motivo no podemos crear la solicitud del certificado desde el IIS ya que no controlamos con qué clave se firma.

El mejor método de hacer esta solicitud es desde el **OpenSSL** siguiendo estos pasos:

1.  **Descargar** e **instalar** **OpenSsl** para Windows: [http://slproweb.com/products/Win32OpenSSL.html](http://slproweb.com/products/Win32OpenSSL.html)
2.  Abrir una **consola** (cmd.exe) y **establecer la variable de entorno** para el fichero de configuración del OpenSSL

`set OPENSSL_CONF=c:\[Ruta a la carpeta donde has instalado el OpenSSL]\bin\openssl.cfg`

3. **Crear la solicitud de certificado** y creación de la clave privada:

`openssl req -nodes -newkey rsa:2048 -keyout private.key -out request.csr`

Asegúrate de que tienes permisos de escritura en la carpeta en la que estás depositando el .key y el .csr.  
    Al lanzar esta llamada al openssl se te pedirá que rellenes una serie de datos para la solicitud del certificado. Ves introduciendo los datos y pulsa ENTER.  
    También se pedirá una **contraseña**. **Anótala** ya que la usaremos más adelante.  
    **IMPORTANTE** en “_Common Name (e.g. server FQDN or YOUR name) []:_” pon el nombre de dominio que quieres proteger (ej: www.compilando.es)

Con el “.csr” creado ya podemos pedir a nuestro **proveedor de SSL** de confianza que nos genere un certificado de seguridad instalable en nuestro servidor.

# Exportar el fichero .PFX

Para poder configurar el Web Role en Azure necesitamos obtener el fichero que contiene el certificado de seguridad y la clave privada que generó la petición por eso debemos obtener este **fichero .pfx** mediante otra llamada al **OpenSSL** usando el **“.crt”** que nos ha enviado el proveedor de SSL junto con el **“.key”** que creamos en el paso anterior.

Supongamos que nuestro proveedor nos envía el fichero de certficado llamado “compilando.crt”

Para ello seguimos los siguientes pasos:

*   Abrimos el certificado que nos envía el proveedor, si no está en formato **pkcs12** tenemos que convertirlo antes. Para ello abrimos una consola (cmd.exe) y ejecutamos el siguiente comando de openssl (ejemplo de conversión desde **pkcs7**)**:**

`openssl pkcs7 -print_certs -in compilando.crt -out compilando_pkcs12.crt`

*   Con el certificado ya en formato **pkcs12** exportamos el pfx:

`openssl pkcs12 -export -in compilando.crt -inkey private.key -out azure.pfx`

*   En la consola nos solicitarán la **contraseña** que pusimos en el primer paso cuando creamos “private.key” (por eso dije de anotarla ;-))

Con esto obtenemos un fichero llamado **“azure.pfx”** que ya podemos usar para configurar nuestro **WebRole**.

# Configurar HTTP en Azure

Con el certificado correctamente generado y exportado ya podemos **configurar nuestro WebRole** para disponer de un endpoint funcionando con HTTPS.

Lo primero que debemos hacer es **subir el .PFX** que acabamos de generar a Azure. 

Para ello nos conectamos al **panel de administración de Azure** (https://manage.windowsazure.com/) y hacemos lo siguiente:

*   Cloud Services > [Web Role] > Certificates > Upload
*   Escogemos el fichero .PFX que acabamos de crear y escribimos la **contraseña** (la misma que pusimos al crear el .key y al exportar el .pfx)

![](http://res.cloudinary.com/escapistasclub/image/upload/v1469571050/Compilando/azure-ssl-1.png)

*   Una vez subido veremos el certificado en el listado y copiamos el GUID de la columna “Thumbprint”

![](http://res.cloudinary.com/escapistasclub/image/upload/v1469571054/Compilando/azure-ssl-2.png)

Ahora lo que nos queda es configurar el **EndPoint** en el **Visual Studio**. Para ello vamos a la solución en la que tenemos nuestro proyecto de Azure y editamos la definición del Servicio (**ServiceDefinition.csdef**) y los ficheros de configuracón del Servicio (**ServiceConfiguration.XXXX.cscfg**) tal y como se ve en los ejemplos que hay a continuación:

## ServiceDefinition.csdef

```xml
<?xml version="1.0" encoding="utf-8"?>
<ServiceDefinition name="Compilando.WebRole" xmlns="http://schemas.microsoft.com/ServiceHosting/2008/10/ServiceDefinition" schemaVersion="2012-10.1.8">
  <WebRole name="Compilando.WebRole" vmsize="Small">  
    <Sites>
      <Site name="Web">
        <Bindings>
          <Binding name="Endpoint1" endpointName="Endpoint1" />
          <Binding name="HttpsIn" endpointName="HttpsIn" />
        </Bindings>
      </Site>
    </Sites>
    <Endpoints>
      <InputEndpoint name="Endpoint1" protocol="http" port="80" />
      <InputEndpoint name="HttpsIn" protocol="https" port="443" certificate="www.compilando.es" />
    </Endpoints>
    <Certificates>
      <Certificate name="www.compilando.es" storeLocation="LocalMachine" storeName="CA" />
    </Certificates>
  </WebRole>
</ServiceDefinition>
```

Como veis, se crea un **nuevo EndPoint** con el protocolo **https** apuntando al puerto **443** y usando el nuevo **certificado** que hemos subido.

## **ServiceConfiguration.Cloud.cscfg**

```xml
<?xml version="1.0" encoding="utf-8"?>
<ServiceConfiguration serviceName="Predeploy.Terminis.Web.Azure" xmlns="http://schemas.microsoft.com/ServiceHosting/2008/10/ServiceConfiguration" osFamily="3" osVersion="*" schemaVersion="2012-10.1.8">
  <Role name="Compilando.WebRole">  
    <Instances count="1" />
    <Certificates>
      <Certificate name="www.compilando.es" thumbprint="F319BC716E1B17FC8824B9325059EC503B6609AE" thumbprintAlgorithm="sha1" />
    </Certificates>
  </Role>
</ServiceConfiguration>
```
En este caso, en la configuración Cloud de nuestra web _“Compilando.WebRole”_ indicamos que el certificado llamado _“www.compilando.es”_ tiene el **Thumbprint** que hemos copiado del panel de control de Azure. Con esto ya podríamos **publicar el WebRole** (recordando **subir los cscfg y csdef actualizados**) para que use el nuevo certificado y ya podríamos visitar nuestra web _https://www.compilando.es_ sin problema alguno ![:)](http://www.compilando.es/wp-includes/images/smilies/simple-smile.png)

**¡Nos vemos Compilando!**