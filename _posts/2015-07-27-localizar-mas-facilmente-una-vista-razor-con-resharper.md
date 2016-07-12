---
layout: post
title:  "Localizar más facilmente una vista Razor con Resharper"
date:   2015-07-27
categories: [Visual Studio]
tags: [MVC, Razor, Resharper, Visual Studio 2010]
---

Si alguna vez habéis tenido que localizar y maquetar una web que inicialmente estaba en un idioma en varios sabréis que es algo que **quita mucho tiempo y desespera profundamente.**

Es un proceso lento, repetitivo y que requiere compilar para ver los cambios… vamos, una fiesta.

Como me ha tocado hacer lo propio con las nuevas vistas de **[Terminis](https://www.terminis.com/es)** me he planteado la pregunta, ¿**Resharper** no me podrá ayudar a hacerlo más rápido?

Respuesta rápida: **NO por defecto, pero podemos mejorarlo un poco**.

Resharper tiene una opción para refactorizar un string y moverlo a un fichero de recursos pero esta opción **solo funciona en código interpretado no en texto suelto de vistas Razor** :_(

Investigando un poco he encontrado un pequeño truco que permite convertir un trozo del HTML en “código interpretado” por el compilador y así poder recfactorizarlo a recurso: crear un **surround template:**

[![Template Explorer](http://www.compilando.es/wp-content/uploads/2015/07/TemplateExplorer.png)](http://www.compilando.es/wp-content/uploads/2015/07/TemplateExplorer.png)

## Creando el Surround Template

Resharper usa los templates para ayudarte a codificar más rápido y son francamente útiles. En concreto, los surround templates, sirven para seleccionar un texto y envolverlo con, por ejemplo, un tag HTML.

Para nuestra pequeña “trampa” necesitamos crear uno que envuelva el texto seleccionado entre @(“TEXTO”) para que resharper pueda interpretarlo y moverlo a un fichero de recursos.

Para ello seleccionamos en el Template Explorer la opción de “Surround Templates” en los tabs de arriba, elegimos el scope “Razor” y pulsamos en “New Template”:

[![New Surround Template](http://www.compilando.es/wp-content/uploads/2015/07/NewSurroundTemplate.png)](http://www.compilando.es/wp-content/uploads/2015/07/NewSurroundTemplate.png)

Esto nos abrirá una ventana para que peguemos el código del template y le demos un nombre. Yo lo he llamado “_@” (si, lo sé, super original..)_

Este es el código:

`@(“$SELECTION$”)`

Luego, en la lista, tienes que arrastrarlo a la quick list para tenerlo más a mano (como en la imagen de arriba).

Bien, con esto ya tenemos el surround template listo y tan solo tenemos que usarlo.

## Usandolo en nuestra vista Razor

Tenemos un texto en un h2 que queremos incluir en el fichero de recursos lo más rápido posible.

[![h2](http://www.compilando.es/wp-content/uploads/2015/07/h2.png)](http://www.compilando.es/wp-content/uploads/2015/07/h2.png)

Hacerlo de la forma “tradicional” implicaría ir al fichero de recursos, añadir uno nuevo, darle un nombre, copiar&pegar este texto, clonarlo para el resto de idiomas, localizar en cada idioma y volver a la vista de razor para sustituir el texto por la llamada al fichero de recursos: **un coñazo**.

Con surround template (y con los benditos shortcuts!):

1) _CTRL + W_ hasta seleccionar toda la frase a localizar:

[![h2_1](http://www.compilando.es/wp-content/uploads/2015/07/h2_1.png)](http://www.compilando.es/wp-content/uploads/2015/07/h2_1.png)

2) _CTRL + E , J_: Surround with: @:

[![h2_2](http://www.compilando.es/wp-content/uploads/2015/07/h2_2.png)](http://www.compilando.es/wp-content/uploads/2015/07/h2_2.png)

3) _CTRL + R, O_: Refactor, Move To:

[![RefactorMoveTo](http://www.compilando.es/wp-content/uploads/2015/07/RefactorMoveTo.png)](http://www.compilando.es/wp-content/uploads/2015/07/RefactorMoveTo.png)

¡voilá! Resharper se encarga el solo de crear el recurso (en el fichero que le digas), sustituirlo en el Razor y dejarlo listo para clonarlo en el resto de idiomas.

### ACTUALIZADO

El gran **[Subgurim](http://www.subgurim.net/)** me comenta una forma aún más rápida de refactorzar texto:

En el explorador de soluciones seleccionamos el proyecto y, en sus **propiedades -> Resharper** modificamos:

*   _Localizable: True_
*   _Localizable inspector: Pessimistic_

[![ResharperLocalizationProperties](http://www.compilando.es/wp-content/uploads/2015/07/ResharperLocalizationProperties1.png)](http://www.compilando.es/wp-content/uploads/2015/07/ResharperLocalizationProperties1.png)

Con esto habilitamos la opción de Refactor > Move To Resources en el menú de acceso rápido con **ALT+ENTER** lo cual hace el paso 3 mucho más rápido:

¡Gracias Maestro!

* * *

No es la solución que esperaba encontrar pero me ha ahorrado horas de tedioso copy&paste…

¡Nos vemos compilando!