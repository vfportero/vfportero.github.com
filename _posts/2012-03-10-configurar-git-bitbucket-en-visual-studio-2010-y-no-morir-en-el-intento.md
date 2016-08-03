---
layout: post
title:  "Configurar GIT (BitBucket) en Visual Studio 2010 y no morir en el intento"
date:   2012-03-10
categories: [Ingeniería del Software, Source Control]
tags: [BitBucket,GIT,Tutorial,Visual Studio 2010]
---
Que **Git** se ha convertido en el más famoso y mejor **controlador de versiones de código** es una cosa que ya sabéis pero, ¿cómo puedo **configurar las herramientas** necesarias para poder **trabajar** con un **repositorio Git en mi Visual Studio 2010**?

El objetivo de este pequeño tutorial es conseguirlo sin morir en el intento

Nota: Este tutorial no explica el funcionamiento de un control de versiones por lo que doy por supuesto que todo el mundo sabe lo que es un commit, push, pull, branch, etc… Otro día haré un artículo dedicado a explicar más a fondo como funciona un control de versiones ! :)

## ¿BitBucket o GitHub?

¡Para empezar! Necesitamos una cuenta en algún servicio de repositorio de Git (si no dispones de uno local o el de tu empresa).

Actualmente existen dos grandes servicios gratuitos de Git que son **[GitHub](https://github.com/)** y **[BitBucket](https://bitbucket.org/)**. No voy a entrar a discutir cual es mejor en este artículo, solo diré que **BitBucket** te permite tener, **de forma gratuita**, **repositorios privados** lo cual es perfecto para esas pequeñas aplicaciones que tienes en casa y quieres salvaguardar.

Es por este motivo principalmente por lo que yo uso BitBucket como Source Control !

¿Ya tienes cuenta en **BitBucket**? ¡Descarguemos Git!

## Descargando y Configurando Git para Windows

Bien, lo primero que necesitamos es descargar Git para Windows el cual podemos encontrar aquí: **[http://code.google.com/p/msysgit/downloads/list](http://code.google.com/p/msysgit/downloads/list)**

Descarga e instala la última versión disponible.

No hay que modificar casi nada durante el proceso de instalación. La única opción que recomiendo modificar es la de “Adjusing your PATH environment” la cual aconsejo cambiar de “Use Git Bash only” a “Run git from the Windows Command Prompt” tal y como se ve en la siguiente captura:

 ![](http://res.cloudinary.com/escapistasclub/image/upload/v1470207921/Compilando/git_1.png)

El resto lo puedes dejar igual.

¡Bien! ¡Ya lo tenemos instalado! Ahora tenemos que configurarlo para que funcione con nuestra cuenta de BitBucktet.

Para ello ejecutamos el “**Git Bash**” (si, es una consola, no te asustes, jeje) e introducimos la configuración global de nuestro usuario de BitBucket e Email escribiendo:

```bash
git config --global user.name "tu usuario de BitBucket aquí"
git config --globla user.email "tu email de BitBucket aquí"
```

![](http://res.cloudinary.com/escapistasclub/image/upload/v1470207927/Compilando/git2.png)

Con esto ya lo tenemos!

Ahora vamos a configurar las herramientas en el Visual Studio 2010.

## Configurando Git en Visual Studio 2010

Debido a que **Visual Studio** no incluye Git como control de código por defecto (solo incluye SourceSafe y Team Foundation Server) necesitamos instalar una pequeña extensión de Git para que funcione.  
Para ello vamos a “**Tools > Extension Manager**” o “**Herramientas > Administrador de Extensiones**” en castellano y, en la Galería Online buscamos “Git” para instalar el “**Git Source Control Provider**“:

![](http://res.cloudinary.com/escapistasclub/image/upload/v1470207929/Compilando/git3.png)

Una vez instalado lo escogemos como Source Control por defecto en **“Herramientas > Opciones > Source Control > Selección de Complemento > Git Source Control Provider”**

Lo próximo que debemos hacer es abrir (o crear) la solución que queremos subir a BitBucket y, pulsando botón derecho sobre el nombre de la solución clickamos en “**New Repository**“.

Esto creará un nuevo repositorio Git en el mismo directorio en el que está la solución.

Verás que en el “Solution Explorer” aparecen unos símbolos con un “+” amarillo. Esto indica que el fichero es nuevo y no se encuentra en tu repositorio local.

Haciendo click derecho de nuevo sobre la solución y pulsando en “**Pending Changes**” podremos ver el listado de ficheros pendientes, escribir un comentario y hacer el **primer commit** contra tu repositorio local.

¡Ya está! ¡Ya tenemos configurado Git para Visual Studio! Eso si, de momento solo lo deja en local y BitBucket ni lo hemos usado.

¿Qué toca ahora? **Configurar Git Bash para que suba nuestros commits a BitBucket.**

## Configurar Remote y hacer Push desde Visual Studio a BitBucket

De nuevo **botón derecho en la solución > Pending Changes > Git Bash** y se nos abrirá la consola de Git. Ahora tenemos que configurar el acceso remoto.

Para ello escribimos lo siguiente:

`git remote add origin https://usuario@bitbucket.org/nombre_del_repositorio.git`

Supongamos que mi usuario es “vfportero” y mi repositorio en BitBucket se llama “demo”. El remote sería así:

`git remote add origin https://vfportero@bitbucket.org/demo.git`

¡Perfecto! Con esto ya podemos escribir: 

`git push`

Nos pedirá el password de BitBucket y, una vez escrito, y el bash solito se encargará de subir los ficheros al repositorio remoto.

Espero que os haya servido de ayuda.

**¡Nos vemos Compilando!**