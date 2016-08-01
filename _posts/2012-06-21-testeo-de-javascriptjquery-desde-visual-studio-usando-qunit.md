---
layout: post
title:  "Testo de Javascript/Jquery desde Visua Studio usando Qunit"
date:   2012-06-21
categories: [Testing]
tags: [Javascript,JQuery, Nuget, QUnit,TDD, Test, Tutorial, Visual Studio]
---

Conforme pasa el tiempo la importancia del **javascript** en nuestros sitios web se hace más evidente y, en demasiadas ocasiones, esto se olvida y se relega dejando **todo nuestro código .js fuera de la cobertura de tests**.

En el nuevo proyecto que estoy desarrollando en [Avanzis](http://www.avanzis.com) me he propuesto tener el mayor **Code Coverage** posible y, para ello, voy a testear también el javascript que escriba.

Para ello necesito un modo de hacerlo de forma **cómoda** y **rápida** y que, al ser un proyecto NET MVC, cumpla las siguientes condiciones:

*   Los tests de javascript se puedan lanzar desde **Visual Studio.**
*   El **TeamCity** sea capaz de lanzarlos automáticamente en cada compilación (o en un Nightly Build en caso de que tarde mucho)
*   Se pueda calcular el **Code Coverage**

<div>Con estas tres premisas me he lanzado a investigar y he encontrado la combinación perfecta: **QUnit + NQUnit + JSCoverage**</div>

<div>¿Y como monto este puzzle? ¡Vamos a ello!</div>

<div>Este primer post explica como configurar Visual Studio para poder lanzar test QUnit de forma sencilla. En otros post hablaremos sobre NQUnit, como integrarlo con TeamCity y como usar JSCoverage.</div>

# Proyecto de QUnit en Visual Studio

QUnit es un framework de Javascript para realizar pruebas unitarios desarrollado por la gente de jQuery para testear sus propios plugins.

El modo más cómodo de integrar QUnit en Visual Studio es instalar el paquete de **NuGet** “**QUnit for ASP.NET MVC**” ([http://nuget.org/packages/QUnit-MVC](http://nuget.org/packages/QUnit-MVC)) en un nuevo proyecto web (en mi ejemplo: MiProyecto.Javascript.Tests) para que nos cree una estructura similar a esta:

 ![](http://res.cloudinary.com/escapistasclub/image/upload/v1470037487/Compilando/qunit.png)

En **Content** tendremos el css necesario para mostrar la web de resultados de QUnit.

“qunit.js” es la propia librería de QUnit.

**Scripts** es la carpeta en la que vamos a crear la estructura de carpetas de nuestros tests. En mi caso he optado por crear una carpeta “Tests” para los ficheros .js de test de QUnit y otra llamada como el proyecto (MiProyecto/Common) para los ficheros a testear.

Obviamente estos ficheros (miproyecto.common.math.js) están en su carpeta correspondiente dentro del proyecto de la web MVC por lo que debemos hacer es **agregarlos como enlace desde la carpeta del proyecto de test**: (en el ejemplo, botón derecho sobre “Common” > Add Existing Item > Buscamos el .js en su carpeta original y seleccionamos en el desplegable de “Add” la opción “**Add as Link**“:

![](http://res.cloudinary.com/escapistasclub/image/upload/v1470037485/Compilando/RNox0.png)

De este modo puedes acceder al .js original desde el proyecto de Test sin tener que copiar el fichero.

Por último solo necesitamos una **página html** en la que incluir estos ficheros javascript con una estructura prefijada por QUnit para mostrar los resultados.

El código de la página debe ser similar a este:

```html
<html>
<head>
    <title>QUnit Test Suite</title>
    <link rel="stylesheet" href="Content/qunit.css" type="text/css" media="screen" />
    <script type="text/javascript" src="http://ajax.aspnetcdn.com/ajax/jQuery/jquery-1.7.2.min.js"></script>
    <script type="text/javascript" src="Scripts/qunit.js"></script>
    <script type="text/javascript" src="Scripts/MiProyecto/Common/math.js"></script>
    <script type="text/javascript" src="Scripts/Tests/math.tests.js"></script>
</head>
<body>
    <h1 id="qunit-header">
        MiProyecto QUnit Test Suite</h1>
    <h2 id="qunit-banner">
    </h2>
    <div id="qunit-testrunner-toolbar">
    </div>
    <h2 id="qunit-userAgent">
    </h2>
    <ol id="qunit-tests">
    </ol>
    <div id="qunit-fixture">
    </div>
</body>
</html>
```

Como ves hay que añadir tanto los scripts propios de QUnit como los ficheros a testear y los propios ficheros de tests.

**Nota**: Desde **Resharper** se pueden lanzar los tests directamente sin tener que ejecutar esta página **PERO** tiene ciertas limitaciones (no se puede usar la creación dinámica de valores en el qunit-fixture y no se puede depurar facilmente) por lo que al final opté por hacer la página y ejecutarla manualmente para poder depurar con Firebug.

¡Bien! Ya tenemos la estructura creada pero, ¿cómo se escriben los tests? ¡Allá vamos!

# Tests en QUnit

Supongamos que tenemos el siguiente fichero javascript en nuestra web (miproyecto.common.math.js) que tiene un par de métodos para hacer sumas, uno que simplemente suma dos números y otro que obtiene el valor de dos inputs por id, los suma y devuelve un span con el resultado:

```javascript
//Suma dos valores SOLO si son números 
var sumNumbers = function (x, y) {
    if (typeof x === "number" && typeof y === "number")
    { return x + y; } else {
        return "error";
    }
}

//Obtiene el valor de dos elementos por su id, los suma (si son números) y crea un <span> con el resultado
var createSpanSumResultFromInputs = function (idX, idY) {

    var spanResultado = $(document.createElement('span'));

    var x = $("#" + idX).val();
    var y = $("#" + idY).val();
    var result = sumNumbers(x, y);
    if (typeof result === "number") {
        var stringResult = x + '+' + y + '=' + result;
        spanResultado.InnerHtml = stringResult;
    } else {
        spanResultado.InnerHtml = "error";
    }

    return spanResultado;
}

```

Estos dos ejemplos sirven para demostrar como hacer tests simples para validar entrada de datos, validar resultado y validar modificación de la DOM (lo más común en nuestros scripts).

Para nuestro **fichero de test** creamos un nuevo fichero js (miproyecto.common.math.tests.js) que está estructurado de la siguiente forma:

```javascript
// Referencia: Ruta relativa del fichero javascript a testear (para que Resharper la incluya antes de lanzar el test)
/// <reference path="/Scripts/Terminis/Common/math.js"/>

// Clase que engloba todos los tests. Es aquí donde se especifica el SetUp y el TearDown
module('Math Tests', {
    setup: function () {
        // setup code

    },
    teardown: function () {
        // teardown code
    }
});

// Método de test
test('nombre del test', function () {
    //... Asserts
});
```
Como ves la estructura es similar a la de una clase de test de **NUnit**. Tenemos **SetUp** y **TearDown** y varios métodos de tests con varios **asserts** (o afirmaciones). ¡Importante! no olvidar el “**<reference>**” o nuestros tests fallarán al invocar los métodos a probar.

Para empezar vamos a probar nuestro método “**sum**” validando que **solo sume dos números** y **devuelva el valor correcto o el string “error” en caso contrario**:

 ```javascript
 test('Sum Validation.', function () {
    equal(sumNumbers("", ""), "error", "sumar dos cadenas vacias devuelve error");
    equal(sumNumbers("zadasdad", "asdadsa"), "error", "sumar dos textos devuelve error");
    equal(sumNumbers("asdasd", 1), "error", "sumar texto y número devuelve error");
    equal(sumNumbers(1, "asdasdasd"), "error", "sumar número y texto devuelve error");
});

test('Sum Correct Results.', function () {
    equal(sumNumbers(1, 2), 3, "suma de dos enteros");
    equal(sumNumbers(-1, -2), -3, "suma de dos negativos");
    equal(sumNumbers(-2, 1), -1, "suma mixta");
    equal(sumNumbers(1, 2.5), 3.5, "suma de entero y decimal");
    equal(sumNumbers(1.5, 2.5), 4, "suma de dos decimales");
});
```

Es muy sencillo hacer los asserts. Tan solo hay que hacer uso de la **API de QUnit** (usando “ok”, “equal”, “notEqual”, “raises”, etc…), poner el resultado de la llamada, el valor esperado y un mensaje para identificar el tipo de afirmación que estamos probando.

En el segundo método las pruebas se complican ya que **los valores de entrada dependen de elementos de la DOM** y el resultado es un <span> por lo que **necesitamos crear lo que necesitamos en el SetUp antes de lanzar los tests**:

```javascript
module('createSpanSumResultFromInputs Tests', {
    setup: function () {
        // setup code
        $('#qunit-fixture').append('<input type="text" id="xTest" value="1"/>');
        $('#qunit-fixture').append('<input type="text" id="yTest" value="2"/>');
        $('#qunit-fixture').append('<input type="text" id="wrongValue" value="this is not a number"/>');
    }
});
```

En el **SetUp** de este nuevo módulo (solo se ejecutará este SetUp antes de las siguientes llamadas “test()”) hemos creado 3 inputs, dos con valores correctos y uno incorrecto para tests de validación, como si de una página “mockeada” se tratara. 

Para crearlos los hemos añadido al contenedor con id “**qunit-fixture**“, este elemento es un div cuyo contenido se crea y de destruye en cada ejecución por lo que podemos guarrear todo lo que queramos con él.

Los tests en este caso no diferirían demasiado a los ya visto con anterioridad:

```javascript
test('Validation Tests', function () {
    equal(createSpanSumResultFromInputs("not exists", "yTest").text(), "error", "Si no existe el idX devuelve error");
    equal(createSpanSumResultFromInputs("xTest", "not exists").text(), "error", "Si no existe el idY devuelve error");
    equal(createSpanSumResultFromInputs("xTest", "wrongValue").text(), "error", "Si uno de los dos no es un número devuelve error");
});

test('Correct Test', function () {
    equal(createSpanSumResultFromInputs("xTest", "yTest").text(), "1+2=3" , "Obtiene los dos valores de sendos input's y devuelve <span> con la operacion");
});
```

Con esto ya sabemos todo lo básico de los test con QUnit. Existen muchas más cosas como el asyncTest() para peticiones que incluyan callbacks que son harina de otro costal.

El resultado es una web similar a esta con todos los test organizados por módulos > tests > asserts pudiendo relanzar test, ocultar test correctos y depurar con FireBug:

[![](http://res.cloudinary.com/escapistasclub/image/upload/v1470037485/Compilando/qunit_web.png)](http://res.cloudinary.com/escapistasclub/image/upload/v1470037485/Compilando/qunit_web.png)

Con esto ya sabemos la base para empezar a chapurrear tests de javascript en nuestro Visual Studio ![:)](http://www.compilando.es/wp-includes/images/smilies/simple-smile.png)

En la próxima entrega hablaremos sobre como lanzar estos tests desde un servidor de integración continua (como TeamCity) y calcular el Code Coverage.

Un saludo.

**¡Nos vemos Compilando!**