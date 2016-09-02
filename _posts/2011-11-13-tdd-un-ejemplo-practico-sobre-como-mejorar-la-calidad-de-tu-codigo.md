---
layout: post
title:  "TDD, un ejemplo práctico sobre mejorar la calidad de tu código"
date:   2011-11-13
categories: [Ingeniería del Software, Testing]
tags: [12 Meses 12 Katas, C#, Ingeniería del Software, NUnit, TDD, Test]
---
Ultimamente se habla mucho de **diseño ágil** pero la realidad de muchas empresas de desarrollo dista mucho de implementarlo de forma eficiente o directamente de usarlo en cualquiera de sus aspectos

Y es que el agilismo es una metodología muy fácil de entender pero costosa de poner en práctica si no se hace de la manera adecuada.

Por ese mismo motivo he visto que sería interesante dedicar una entrada a **explicar de forma práctica y sencilla qué es** **TDD** y cómo se utiliza en un proyecto.

Lo primero de todo confesar que **no soy un experto en la materia** y por ello es de obligada necesidad indicar que, si quereis profundizar más en estos temas, vayais a la web **“[www.dirigidoportests.com](http://www.dirigidoportests.com/)“** de **[Carlos Blé](https://twitter.com/#!/carlosble)**, autor de “**[Diseño Ágil con TDD](http://www.dirigidoportests.com/el-libro)**” el libro de cabecera de todo aquel desarrollador que quiera empezar a trabajar con esta metodología.

Además dar las gracias encarecidamente a **[Javier Casal](https://twitter.com/jcasalruiz)** por darme las clases magistrales de TDD y la gente de **[12Meses12Katas](http://12meses12katas.com/)** por el ejemplo práctico para compartir con todos vosotros !:-)

## ¿Qué es TDD (Test Driven Development)?

El **diseño dirigido por test**, o TDD, es una técnica de diseño e implementación de software consistente en **generar las pruebas** unitarias **justas** y necesarias **para obtener la implementación de las funciones que el cliente** **necesita** minimizando los errores que llegan a producción.

El diseño por test no solo **nos ahorra implementar más de lo debido** si no que nos informa de si un **cambio posterior “rompe” algo de lo que ya hemos implementado** y nos facilita la tarea de entregar un **código más limpio y libre de errores**.

TDD no se trata de escribir cientos y cientos de pruebas si no se escribir las que necesites (y cuanto más simples mejor) para cumplir las necesidades del cliente y poder, posteriormente, empezar a desarrollar para cumplir dichos requisitos.

Como indico en el título **esto es un ejemplo práctico sobre como utilizar TDD** para resolver un problema concreto por lo que he hecho uso del problema de **[Noviembre del 2011 de 12Meses12Katas](https://github.com/12meses12katas/Noviembre-KataPotter)** para ilustrar este post: **KataPotter**

Para ello vamos a crear una biblioteca de clases **C#** en **Visual Studio 2010** y usaremos el framework de pruebas **NUnit**.

## Explicando el problema

Bien, el problema que se nos plantea es la creación de un programa que **calcule el importe** **de una coleción de libros aplicando un determinado descuento** en función de los distintos tipos de libros que compremos. Partiendo de que **cada libro tiene un coste de 8€** el cliente nos indica que, **si se adquieren dos libros distintos, se aplique un 5% de descuento a ambos libros**, un 10% si se compran 3, un 20% para 4 distintos y un 25% para la serie completa de 5 libros.

Otra condición del cliente es que **el programa obtenga el precio más económico** si hay varias combinaciones posibles de packs.

Como entrada recibiremos un array con los códigos de los libros a comprar (de 0 a 4) y como salida obtendremos un valor double.

¡Muy bien! Ya sabemos que tenemos que hacer y que quiere el cliente por lo que vamos a escribir las primeras pruebas.

## Pruebas Sencillas

Lo primero que debemos hacer es **escribir unas pruebas sencillas que implementen los casos más básicos**. La primera y más sencilla es que cuando nuestra cesta de compra esté vacia el precio total es 0\. En NUnit sería así:

```c#
[Test]
public void EmptyBasket()
{
     Assert.AreEqual(0, _bookPriceCalculator.CalculateBasket(new int[] { }));
}
```

Sin entrar en explicar la sintaxis de NUnit simplemente indicar que esta prueba comprueba que si se le pasa un array vacio el sistema devuelve 0.

**¿Cómo resolvemos esta prueba con la menor cantidad de código posible?** Muy fácil:

```c#
public double CalculateBasket(int[] books)
{
     return 0;
}
```

**¡Exacto!** Esa es la solución más sencilla que hace que la prueba sea correcta. Obviamente en cuanto añadamos una segunda prueba tendremos que rehacer nuestro código pero **de esto se trata TDD**.

¡Ok! Vamos ha añadir una nueva prueba para indicar una nueva condición impuesta por el cliente: **Cada libro vale 8€**

```c#
[Test]
public void TwoSameBook()
{
     Assert.AreEqual(8 * 2, _bookPriceCalculator.CalculateBasket(new int[] { 0, 0 }));
}
```

En esta prueba indicamos que si se compran dos veces el libro “0” el precio total es de 16€. **¿Cómo modificamos nuestro método para que pase las dos pruebas?**

```c#
public double CalculateBasket(int[] books)
{
     return books.Length * 8;
}
```

¡Vale! No es muy elegante pero solucionaría cualquier prueba que no implique un descuento por lo que podemos cerrar el capítulo de pruebas sencillas.

## Añadiendo Funcionalidad: Calculando Descuentos

Nuestro cliente nos había puesto como premisa que, si se compraban dos o más libros distintos había que aplicar un descuento determinado por lo que vamos a crear una serie de pruebas de ejemplo que implementen esta funcionalidad:

```c#
[Test]
public void TwoDifferentBooks()
{
            Assert.AreEqual(8 * 2 * 0.95, _bookPriceCalculator.CalculateBasket(new int[] { 0, 1 }));
}

[Test]
public void SeveralDiscountsTwo_One()
{
            Assert.AreEqual(8 + (8 * 2 * 0.95), _bookPriceCalculator.CalculateBasket(new int[] { 0, 0, 1 }));
}
```

Como veis en las pruebas tan solo declaramos una entrada de datos y la solución que nuestro cliente espera. Es trabajo del desarrollador el **encontrar el modo en que el código del programa satisfaga todas las nuevas pruebas sin “romper” ninguna de las anteriores ya implementadas**.

La solución que se me ha ocurrido a mi es hacer una clase “BookPack” que simbolice un “paquete de libros” donde todos los libros deben de ser distintos por lo que cada compra se traduce realmente en varios “BookPacks” cada uno con su precio y descuento.

Sin tener en cuenta la última condición (que indicaba que se ha de escoger la combinación más económica), la solución más sencilla es esta:

```c#
List<BookPack> BookPacks = new List<BookPack>();

foreach (int bookCode in books)
{
     BookPack currentBookPack = null;

     foreach (var bookPack in BookPacks)
     {
          if (!bookPack.HasThisBook(bookCode))
          {
              currentBookPack = bookPack;          
          }
     }

     if (currentBookPack == null)
            {
                BookPacks.Add(new BookPack(bookCode));
            }
            else
            {
                currentBookPack .Books.Add(bookCode);
            }
}
return BookPacks.Sum(bookPack => BookPack.GetBookPackPrice(bookPack.Books.Count));
```

Donde “GetBookPackPrice” devuelve el precio del paquete en función de cuantos libros tiene:

```c#
public static double GetBookPackPrice(int quantity)
        {
            return 8 * quantity * GetDiscountPerDifferentBooks(quantity);
        }
//GetDiscountPerDifferentBooks devuelve 1, 0.95, 0.9, 0.8 o 0.75 en función de si se compran 1, 2, 3, 4 o 5 libros distintos respectivamente.
```

Con esto ya tenemos implementada tanto la funcionalidad básica (ningún libro o varios libros iguales) como el descuento básico por “paquetes de libros”. Sólo nos faltaría añadir la última condición del cliente: que se elija la combinación más económica a la hora de hacer los paquetes.

## Pruebas Finales: Obteniendo el paquete más económico

No es lo mismo hacer dos paquetes de 4 libros (51.2€ en total) que 1 de 5 y otro de 3 (51.6€). Con el código anterior obtendríamos la opción más desfavorable ya que solo se crea un nuevo paquete cuando el libro actual ya se encuentra en el paquete actual y no se tiene en cuenta el importe final.

Para añadir esta nueva funcionalidad vamos a escribir la última prueba:

```c#
[Test]
public void SeveralDiscountsFourFour()
{
    Assert.AreEqual(2 * (8 * 4 * 0.8), _bookPriceCalculator.CalculateBasket(new int[] { 0, 0, 1, 1, 2, 2, 3, 4 }));
}
```

La implementación para este caso consiste en cambiar el concepto “CurrentBookPack” que tan solo va rellenando paquetes por “CheapestBookPack” el cual compara entre los distintos paquetes y se queda con el más económico.

**Como el código resultante ya es más largo y enrevesado como para ponerlo en un post** mejor lo veis directamente en mi repositorio de [**GitHub**](https://github.com/vfportero) para este ejemplo ;-):

[h**ttps://github.com/vfportero/Noviembre-KataPotter/tree/master/vfportero**](https://github.com/vfportero/Noviembre-KataPotter/tree/master/vfportero)

## Conclusiones Finales

Creo que este es un gran ejemplo para explicar las virtudes de TDD ya que en un mismo proyecto **hemos tenido que hacer 3 modificaciones que han alterado por completo el funcionamiento de la aplicación**. 

Al principio solo teníamos que calcular importes sencillos (8 * unidades), después se añadió unos descuentos por volumen para terminar con un sistema más inteligente que se queda con la combinación más económica.

Ahora **extrapola este ejemplo a un proyecto real con cientos de clases y miles de líneas** y piensa que pasaría si tuvieras que hacer este tipo de cambios… **asusta, ¿verdad?**

Y es que ¿cuantas veces hemos **“arreglado algo y roto otro cosa”**? ¿cuantas veces los **requerimientos del software cambian** y debemos **adaptarnos rapidamente** a las nuevas necesidades? ¿cuantas veces programamos más de lo debido?

**TDD** ayuda a solucionar estos problemas **facilitandote el modo en el que organizas tus tareas**. Cuando programas primero una prueba estás abstrayendote de algoritmos y estrategias para centrarte en entender el problema y la solución exacta que quiere el cliente.

Si logras crear pruebas simples que explican una pequeña parte del código podrás tener un sistema casi perfecto cuando el producto se entregue además de ganar en flexibilidad en caso de los requerimientos cambien en medio del desarrollo.

Está claro que **escribir pruebas antes de programar es un trabajo extra** que en muchos casos no se puede asumir pero, **a la larga, ahorra mucho tiempo en mantenimiento** **y evita problemas de código heredado.**

**TDD**, como otras herramientas dentro del **agilismo**, solo te da una serie de “**buenas practicas**” que te ayudan a mejorar tu calidad como desarrollador. **Está en tu mano decidir hasta que punto utilizar TDD** !:-)

Para terminar solo invitar a todos aquellos interesados en este tema a que se pasen por la web de **[12Meses12Katas](http://12meses12katas.com/) **donde encontraran ejemplos y soluciones a problemas como este “KataPotter” que hemos explicado aquí. Os recomiendo que os junteis con ese amigo programador con el que siempre haceis el friki, compreis una caja de cervezas, y empeceis a trabajar con una de estas Katas en parejas.

Mientras uno programa una prueba el otro programa el código que la implementa y viceversa. Vereis como TDD se ve de otra manera !

**¡Nos vemos Compilando!**