---
layout: post
title:  "Lanzar automáticamente tests de QUNIT desde Teamcity"
date:   2012-06-25
categories: [Testing]
tags: [Nuget, Qunit, TDD, Teamcity, PhantomJS, Test, Tutorial]
---

Como comentaba en el [anterior post](http://www.compilando.es/post/2012/06/21/Testeo-de-javascriptjquery-desde-Visual-Studio-usando-QUnit-NQUnit.aspx) estoy investigando sobre **testeo de Javascript/Jquery desde Visual Studio** y para ello he optado por usar el framework **QUnit**.

Después de las primeras pruebas y solventar los primeros problemas (sobre todo como “mockear” las llamadas ajax o ciertas peticiones a nivel de servidor) ha llegado el momento de añadir a nuestro **servidor de integración continua** (en nuestro caso el **TeamCity**) un nuevo paso para que ejecute automáticamente estos tests cada vez que alguien haga un commit.+

El problema de esto radica en que TeamCity no dispone de un driver nativo para ejecutar QUnit ni para lanzar un navegador y leer los resultados así que para que los tests de QUnit se ejecuten como un test más necesitamos:

1.  Un navegador que lance nuestra web de pruebas
2.  Un dirver que le diga al TeamCity el resultado de cada test (usando los [TeamCity Service Messages](http://confluence.jetbrains.net/display/TCD65/Build+Script+Interaction+with+TeamCity))

## PhantomJS: Ejecutando JavaScript desde la línea de comandos

Lanzar un navegador por cada test de QUnit y leer su resultado es como matar moscas a cañonazos. Lo que queremos es simple: ejecutar javascript y enviarle al TeamCity un mensaje con la respuesta.

Para ello necesitamos un **intérprete de Javascript** lo más ligero posible y que se pueda lanzar desde el cmd de Windows. Para ello hemos optado por usar **[PhantomJS](http://phantomjs.org/).**

**PhantomJS** nos permite ejecutar un fichero .js como si de un navegador se tratara con la ventaja de que, al no tener interfaz gráfica, tan solo ejecuta el javascript y escribe en el log lo que le digamos.

Ahora necesitamos un script que ejecute nuestros tests y escriba por consola mensajes para el TeamCity.

## QunitTeamCityDriver

Para hacernos la vida más fácil disponemos en NuGet de una librería llamada “[QUnitTeamCityDriver](https://github.com/redbadger/QUnitTeamCityDriver)” que nos añadirá a nuestro proyecto dos ficheros javascript:

*   **QUnitTeamCityDriver.phantom.js**

*   Script ejecutado por el PhantomJs al que le pasaremos como parámetro el fichero .html que contiene los tests QUnit

*   **QUnitTeamCityDriver.js**

*   Script encargado de leer la respuesta del PhatomJS y escribir en consola los mensajes necesarios para integrarlo con el TeamCity

<div>Con estos dos ingredientes tenemos todo lo necesario para crear un nuevo Build Step en TeamCity para que se lancen solitos los tests QUnit.</div>

## Configurando TeamCity para lanzar los tests QUnit mediante PhantomJS

Como decía al principio TeamCity no soporta de manera nativa la ejecución de tests QUnit por lo que necesitamos un ejecutable externo que se encargue de interpretar javascript y enviar la respuesta al TeamCity.

Primero, descargamos en el servidor de TeamCity el **[PhantomJS](http://code.google.com/p/phantomjs/downloads/list)** y lo dejamos en una **ruta reconocible** (por ejemplo, _C:\tools\phatomjs_). Si tienes más de un agente recuerda hacer este paso en todos ellos.

En segunto lugar. desde la interfaz de administrador del TeamCity, creamos un nuevo **Build Step** de tipo “**Command Line**“, le ponemos el nombre que queramos y establecemos la ruta del PhatonmJS y los parámetros.

**¡Importante!** Hay que pasarle dos parámetros al PhatomJs, el primero es la ruta al fichero “**QUnitTeamCityDriver.phantom.js**” descargado previamente del paquete del NuGet, el segundo (bueno, realmente es un parámetro que le pasamos a QUnitTeamCityDriver.phantom.js) es la ruta del **fichero HTML que contiene las pruebas** tal y como explicamos en el [anterior post](http://www.compilando.es/post/2012/06/21/Testeo-de-javascriptjquery-desde-Visual-Studio-usando-QUnit-NQUnit.aspx).

**¡OJO CON LAS RUTAS!** Recuerda que, por defecto, TeamCity ejecuta los steps desde el directorio de checkout (aunque puedes modificarlo añadiendo otra ruta en el **Working Directory** del step).

El resultado debe de ser algo similar a esto:

![](http://res.cloudinary.com/escapistasclub/image/upload/v1470036642/Compilando/teamcity-qunit.png)

Una vez hecho esto ya podremos **ejecutar el Build** y veremos como los tests se ejecutan de forma habitual como si fueran de NUnit indicandonos en verde o rojo el resultado de los mismos ![:)](http://www.compilando.es/wp-includes/images/smilies/simple-smile.png)

Espero que os sea útil.

**¡Nos vemos Compilando!**