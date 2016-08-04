---
layout: post
title:  "Primeros pasos desarrollando en Windows Phone: Consejos Útiles"
date:   2012-01-26
categories: [Windows Phone 7]
tags: [Marketplace, Metro UI, Reflexiones, WP7]
---
![](http://res.cloudinary.com/escapistasclub/image/upload/v1470346924/Compilando/windows_phone_sdk_7_1.png)

**Windows Phone 7** es el nuevo **sistema operativo móvil de Microsoft** con el que pretende competir con dos grandes ya asentando en el mercado de los operativos para smartphone como iOS y Android.

Windows Phone 7 pretende reinventar el concepto de operativo movil apostando fuerte por un resideño completo de la interfaz (la famosa **[Metro UI](http://www.compilando.es/?tag=/metro-ui)**) así como la integración con otros sistemas como **XBox Live** y **Windows 8**.

Yo, como buen fan de los productos de Microsoft, no podía perderme esta nueva entrada de los de Redmond al mercado móvil y es por eso que me aventuré a desarrollar una aplicación para el **[Marketplace](http://www.windowsphone.com/es-ES/marketplace?wa=wsignin1.0)** en un principio por aprender pero, conforme le he ido dando forma, pensando en, ¿por qué no? ganar algún eurillo !

Más adelante le dedicaré un post a mi primera app de Windows Phone **“Echemos cuentas”** (que ya está en proceso de aceptación por parte de Microsoft) así que el motivo de este artículo es intentar ayudar a los que, como yo, quieran subirse al barco de Windows Phone Marketplace y no sepan por donde empezar.

¡Aquí van unos cuantos consejitos!

# ¡Empecemos! ¿Qué me tengo que descargar?

Bien, partimos de la base de que eres un desarrollar .NET y que tienes el Visual Studio instalado (y si no ya tardas, jeje) por lo que tenemos que descargar e instalar es el **SDK 7.1 de Windows Phone**:

*   **Windows Phone SDK 7.1**: [http://www.microsoft.com/download/en/details.aspx?displaylang=en&id=27570](http://www.microsoft.com/download/en/details.aspx?displaylang=en&id=27570)

Estas herramientas incluyen todo lo necesario para crear un proyecto para Windows Phone así como el **emulador** para probarlo y depurarlo, **herramientas de testeo** previo a la publicación al Marketplace y el **Expression Blend** para poder diseñar los formularios WPF de forma sencilla y eficaz.

# Bien, ¿por dónde empiezo?

Esto puede parecer un mundo si no damos los pasos correctos y, como no quiero abrumar con mucha información os comentaré los pasos que yo hice hasta poder terminar con éxito mi primera aplicación:

## Revisa los “Quick Starts” del App Hub

**App Hub** es el punto de encuentro de todos los desarrolladores de Windows Phone y XBox Live en el que encontraremos **tutoriales**, **herramientas**, **recursos** y los **foros** en los que preguntar nuestras dudas.

Un ejemplo de estos tutoriales es el clásico “Hello World” que ilustra lo fácil que es crear un nuevo proyecto en Windows Phone y lanzarlo en nuestro emulador o teléfono: 

*   Hello World en Windows Phone:  [http://create.msdn.com/en-US/education/quickstarts/Get_Started_Creating_a_Windows_Phone_Application](http://create.msdn.com/en-US/education/quickstarts/Get_Started_Creating_a_Windows_Phone_Application)
*   Otros “Quick Starts”:  [http://create.msdn.com/en-us/education/quickstarts](http://create.msdn.com/en-us/education/quickstarts)
*   Hacer los retos de Windows Phone de la MSDN: 

*   [http://blogs.msdn.com/b/esmsdn/archive/2011/07/11/windows-phone-mi-primera-interfaz-metro.aspx](http://blogs.msdn.com/b/esmsdn/archive/2011/07/11/windows-phone-mi-primera-interfaz-metro.aspx)
*   [http://blogs.msdn.com/b/esmsdn/archive/2011/07/22/reto-windows-phone-navegar-por-tu-aplicaci-243-n.aspx](http://blogs.msdn.com/b/esmsdn/archive/2011/07/22/reto-windows-phone-navegar-por-tu-aplicaci-243-n.aspx)
*   [http://blogs.msdn.com/b/esmsdn/archive/2011/08/17/reto-windows-phone-control-panorama.aspx](http://blogs.msdn.com/b/esmsdn/archive/2011/08/17/reto-windows-phone-control-panorama.aspx)

Con esto tienes material para probar, aprender y coger soltura con el framework.

## No inventes la rueda: usa los controles Silverlight de Codeplex

Los controles por defecto del SDK están muy bien ya que te permiten montar una interfaz Metro sin apenas esfuerzo pero si de verdad queremos sacarle jugo a nuestra aplicación y que deslumbre a todo aquel que la use no te compliques y usas los controles Silverlight para Windows Phone de **Codeplex**:

*   **Controles Silverlight para Windows Phone**:  [http://silverlight.codeplex.com/releases/view/75888](http://silverlight.codeplex.com/releases/view/75888)

Controles como el **LongListSelector** o el **ExpanderView** harán nuestro trabajo de diseño y funcionalidad muchísimo más fácil.

Además estos controles son OpenSource y se van actualizando periodicamente así que no pierdas de vista Codeplex!

## No tengas miedo y pregunta en los foros de desarrolladores

Si algo tiene la comunidad de desarrolladores de Windows Phone de habla hispana es una **gran predisposición para ayudar a los demás**, resolver sus dudas, dar consejos y crear comunidad.

Todo esto se ve plasmado de forma clara en los más que útiles **foros de MSDN para Desarrollo de Windows Phone** que modera el gran **[Josue Yeray](http://geeks.ms/blogs/jyeray/)** (un referente dentro del mundo del desarrollo Windows Phone):

*   **Foros MSDN para Desarrolladores de Windows Phone:**  [http://social.msdn.microsoft.com/Forums/es-ES/category/windowsphone](http://social.msdn.microsoft.com/Forums/es-ES/category/windowsphone)

En el foro podrás preguntar cualquier duda que tengas desde aspectos técnicos hasta dudas sobre el modo de publicar una app en el Marketplace.

Echa un vistazo y plantea tus dudas. Verás como en breve un desarrollador como tu te echa una mano !


# Y, ¿qué tipo de aplicación hago?

A esta pregunta hay dos posibles respuestas.

Puedes hacer algo que te **sea útil / interesante a ti** y que no haya nada en el Marketplace que lo haga como tu quieres (esa es la ventaja de poder programar lo que quieras desde el mismo Visual Studio y poder instalarlo directamente en tu movil Windows Phone). En mi caso fue solventar un problema de gestión que tenemos en mi grupo de amigos a la hora de hacer las cuentas en los viajes, cenas, cumpleaños, etc… así que acabé programando una aplicación dedicada en la que yo era mi propio cliente.

Por otro lado, si quieres **ganar dinero con la aplicación**, lo que te recomiendo es buscar un nicho de mercado en el [Marketplace](http://www.windowsphone.com/es-ES/marketplace?wa=wsignin1.0) y hacer una aplicación que rellene ese hueco. También puedes fijarte en aplicaciones que han tenido éxito en otras plataformas y, no copiar, si no hacer una versión con características similares aprovechando toda la potencia de la interfaz Metro y los Live Tiles de Mango.

**Windows Phone 7 es una plataforma joven** y aún quedan muchos tipos de aplicaciones por hacer así que busca algo que pueda interesar al público y ¡ponte con ello!

Ah! Y no desesperes si tu aplicación no funciona del todo bien en el mercado. Puedes publicar las aplicaciones que pago que quieras y hasta 100 gratuitas (a partir de 100 se pagan 19.90$ por cada publicación) así que no desesperes y sigue probando.

Quien sabe… a lo mejor estás detrás del nuevo WhatsApp ![😉](http://s.w.org/images/core/emoji/72x72/1f609.png)

# ¿Dónde consigo material más avanzado?

Si ya has hecho los tutoriales, tienes una buena idea y las estás llevando a cabo es posible que te surjan dudas sobre cómo hacer ciertas cosas o sobre como utilizar ciertos controles.

Como he comentado antes los foros son muy útiles pero, para estas situaciones, lo que realmente me ha sacado las castañas del fuego es el libro de **Josuey Yeray**: “**Windows Phone 7.5: Desarrollo de aplicaciones en Silverlight**” que podeis comprar aquí: [http://www.campusmvp.com/Catalogo/Product-Windows-Phone-7.5-Mango—Desarrollo-Silverlight_142.aspx](http://www.campusmvp.com/Catalogo/Product-Windows-Phone-7.5-Mango---Desarrollo-Silverlight_142.aspx)

El libro explica de forma precisa todos los aspectos que envuelven al desarrollo de aplicaciones para Windows Phone Mango con claros **ejemplos** (todos adjuntados en forma de proyectos para Visual Studio que podemos compilar y probar directamente en nuestro emulador!).

![](http://res.cloudinary.com/escapistasclub/image/upload/v1470346935/Compilando/Libro-Windows-Phone-7.5-Mango.jpg)

¡Sin duda una gran compra **100% recomendable**!

Por otro lado, otro proyecto al que no podemos quitarle el ojo de encima es el recién estrenado PodCast sobre desarrollo en Windows Phone: **Windows Phone Controla** del omnipresente **Josue Yeray** y **Rafael Serna**.

*   **Windows Phone Controla: ** [http://www.wpcontrola.com/](http://www.wpcontrola.com/)

**Mensualmente** nos traerán las **noticias** más destacadas dentro del mundo del desarrollo en Windows Phone así como **explicar una API** dentro del framework (por ejemplo, en este primer podcast explican el API del Live Connect!!)

![](http://res.cloudinary.com/escapistasclub/image/upload/v1470346935/Compilando/wpcontrola_tile_thumb_49099CBE.jpg)

Lo dicho, un proyecto más que interesante que promete ser una fuente de información y conocimiento cómoda, accesible y frecuente y es que, ¿Quien no querría sentarse en un bar a tomar una cerveza con dos grandes de Windows Phone como son Yeray y Serna?

# ¡A programar!

Con esto termino mi pequeña ruta de viaje dentro del mundo de Windows Phone. Espero que estos consejos os sean útiles a la hora de empezar con vuestra primera aplicación para el Marketplace y recordad que, si tenéis cualquier duda estamos en los foros de la MSDN para resolverla.

Un saludo y **nos vemos compilando!!**