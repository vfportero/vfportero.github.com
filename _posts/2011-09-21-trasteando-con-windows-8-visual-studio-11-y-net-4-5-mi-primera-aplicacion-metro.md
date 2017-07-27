---
layout: post
title:  "Trasteando con Windows 8, Visual Studio 11 y .NET 4.5: Mi primera aplicación Metro"
date:   2011-09-21
categories: [.NET 4.5,Windows 8, WPF]
tags: [.NET 4.5, C#5, Metro UI, Windows 8, WPF]
---
La pasada semana sufrimos un aluvión de noticias relacionadas con la presentación de la nueva criatura de Microsoft: **Windows 8\.**

# **Windows 8**

Con una renovada Interfaz llamada “**Metro**“, Windows 8 promete revolucionar el mercado de los operativos ofrenciendo un sistema multiplataforma capaz de ser ejecutado desde el sobremesa más potente hasta la tablet más económica.

![](https://escapistas.blob.core.windows.net/compilando-post-images/Windows8.PNG)

Pero este post no va sobre que es Metro, Windows 8 o que trae de nuevo el Internet Explorer 10; vamos a hablar de lo que realmente nos interesa:

*   ¿Qué trae el nuevo Visual Studio 11 y su .NET 4.5?
*   ¿Cómo se desarrolla con Metro?
*   ¿Qué son y cómo funcionan los nuevo métodos asíncronos?

# Visual Studio 11: Creando un proyecto Metro

En la beta del VS 11 nos vienen por defecto varios tipos de plantillas de aplicaciones: **Metro, Grid y Split**. En este caso vamos a usar la plantilla por defecto “**Metro**” para hacer una aplicación Metro de ejemplo con el listado en tiempo real de post de este blog y su contenido como detalle.

Lo primero que nos llama la atención es que la interfaz está completamente en **WPF (Windows Presentation Foundation)** así que solo tenemos una página .xaml y su correspondiente .cs. 

Ok, no nos asustemos, es algo distinto a lo que estamos acostumbrados pero no pasa nada. El objetivo de este ejemplo es conseguir una aplicación que **liste el contenido del [feed](http://www.compilando.es/syndication.axd) de esta web** en un ListView y que **cuando pulsemos en un post cargue el html de dicho post en el cuerpo de la aplicación.**

Vamos a empezar primero por crear la clase **FeedData** que almacenará los datos leídos para luego **obtener de forma asíncrona** el valor del feed directamente de la url: http://www.compilando.es/syndication.axd

# C# 5 y Windows.Web.Syndication: Obtener datos del Feed Rss

Bien, vamos a bindear una lista de “items” rss que tendrán las propiedades de Título, Fecha, Autor y Contenido. Lo primero es **crearnos la clase** que usaremos para bindear con el futuro control y nos creamos también una coleccion de tipo “**ObservableCollection**” para que el xaml pueda bindearlo correctamente.

El código como veis es muy simple:

```csharp
public class FeedData
    {
        public string Titulo { get; set; }

        private ObservableCollection<FeedItem> _Items = new ObservableCollection<FeedItem>();
        public ObservableCollection<FeedItem> Items
        {
            get
            {
                return this._Items;
            }
        }
    }

    public class FeedItem
    {
        public string Titulo { get; set; }
        public string Autor { get; set; }
        public string Contenido { get; set; }
        public DateTime Fecha { get; set; }

    }
```

Una vez creada la clase vamos al meollo del asunto que es como leed del feed, bajarse los datos y crear los correspondientes “FeedItems”. Actualmente tenemos que usar XmlReaders y métodos más o menos manuales para leer un feed rss.

Con **.NET 4.5** tenemos una multitud de clases nuevas que nos ofrecen funcionalidades no existentes hasta ahora como **Windows.Web.Syndication** que nos da una serie de clase y métodos para tratar con feed’s rss así como la **posibilidad de ejecutarlo de forma asíncrona**.

**¡Al grano!** Necesitamos un método que se **ejecute al iniciar la aplicación**, que **lea del feed** y **bindee** el resultado en **el “DataContext”** de la página. Como queremos que la aplicación no se quede “frita” mientras baja el feed **vamos a hacer uso de los métodos asíncronos para invocar el método**.

Como **vale más código que mil palabras** os lo pego y lo voy comentado inline!

```csharp
// Hay que firmar el método como "async" y "Task" para que el compilador cree un hilo para el solo y podamos hacer uso de las herramientas asíncronas
        private async Task ObtenerFeedAsincronamente(string feedUrl)
        {
            // using Windows.Web.Syndication;
            // SyndicationClient -> Clase para trabajar con Feed's RSS
            SyndicationClient client = new SyndicationClient();
            Uri feedUri = new Uri(feedUrl);

                //Obtenemos el feed de la URL dada y esperamos a que termine añadiendo la palabra clave "await" 
                SyndicationFeed feed = await client.RetrieveFeedAsync(feedUri);

                // Ya tenemos en "feed" todos los datos necesarios. Sólo tenemos que recorrerlo y crear las clases para el bindeo
                FeedData feedData = new FeedData();
                feedData.Titulo = feed.Title.Text;

                foreach (SyndicationItem item in feed.Items)
                {
                    FeedItem feedItem = new FeedItem();
                    feedItem.Titulo = item.Title.Text;
                    feedItem.Fecha = item.PublishedDate.DateTime;
                    feedItem.Autor = item.Authors[0].Name;
                    if (feed.SourceFormat == SyndicationFormat.Atom10)
                    {
                        feedItem.Contenido = item.Content.Text;
                    }
                    else if (feed.SourceFormat == SyndicationFormat.Rss20)
                    {
                        feedItem.Contenido = item.Summary.Text;
                    }
                    feedData.Items.Add(feedItem);
                }
                // Bindeamos el ObservableCollection<FeedItem> y seleccionamos el primero
                this.DataContext = feedData;
                ItemListView.SelectedIndex = 0;

        }
```

¡Así de fácil! Basta con poner **“async”** como firma del método y **“await”** en el momento que queremos que se pare la ejecución de este método para que el compilador salte fuera del método, siga su ejecución y escuchando eventos y, cuando termine la llamada “await” vuelva a la siguiente linea y siga ejecutandose. Con esto conseguimos que el usuario pueda seguir trabajando con la aplicación mientras la aplicación trabaja por debajo.

Bien, ya hemos obtenido los datos pero, ¿cómo los muestro en mi interfaz?

**¡Vamos a por el WPF!**

# Controles WPF: Bindeo de Datos en Metro UI

En nuestro MainPage.xaml por defecto veremos un “**Grid**” ya creado que es la rejilla en la que vamos a crear los controles. Como queremos hacer una aplicación **Metro** bien chula como las que se ven en los videos de presentación de Windows 8 vamos a montar una interfaz con una cabecera, una lista de artículos en un lateral usando 2/5 de la pantalla y su contenido en el centro ocupando los 3/5 restantes.

Bien, con estas premisas necesitamos hacer uso de dos controles: **ListView** y **WebView**.

Un **ListView** es muy muy parecido a lo que hemos visto toda la vida: **un control que repite ciertos elementos una vez por cada item**. 

**WebView** es un control que **interpreta HTML como si de un navegador se tratará** lo cual nos viene de perlas para mostrar el contenido del post que previamente hemos leído.

Como antes os comento entre líneas. **Ojo a cómo se bindea el Grid que contiene el cuerpo del post que parece magia!**

```xml
<Grid x:Name="LayoutRoot" Background="#FF0C0C0C">
        <!-- Definimos 2 lineas. Una de 140px para el título y la siguiente que llegue hasta el final-->
        <Grid.RowDefinitions>
            <RowDefinition Height="140" />
            <RowDefinition Height="*" />
        </Grid.RowDefinitions>

        <!-- Titulo. Corresponde con la primera "row" de 140px -->
        <TextBlock x:Name="TitleText" Text="Compilando.ES RSS FEED"
                   VerticalAlignment="Center" FontSize="48" Margin="56,0,0,0"/>

        <!-- Cuerpo. Como veis indicamos que es la row "1" por lo que debe usar el resto de la pantalla -->
        <Grid Grid.Row="1">
            <!--Definimos dentro del Grid principal otro Grid secundario dividido en 5 partes con dos columnas.-->
            <!--La primera tendrá un ancho mínimo de 320px y usará 2/5 partes del espacio total-->
            <!--La segunda usará 3/5 parte del ancho total del Grid-->
            <Grid.ColumnDefinitions>
                <ColumnDefinition Width="2*" MinWidth="320" />
                <ColumnDefinition Width="3*" />
            </Grid.ColumnDefinitions>

            <!-- Listado de Post -->
            <!-- Indicamos que utilice el ObservableCollection<FeedItem> llamado Items del DataContext de la página -->
            <ListView x:Name="ItemListView"  
                      ItemsSource="{Binding Path=Items}"
                      SelectionChanged="ItemListView_SelectionChanged"
                      Margin="60,0,0,10">
                <ListView.ItemTemplate>
                    <DataTemplate>
                        <!-- Controles que se repetirán una vez por cada Item en "Items" -->
                        <!-- Usamos un StackPanel con 3 TextBlock bindeados -->
                        <StackPanel>
                            <TextBlock Text="{Binding Path=Titulo}"  
                                       FontSize="24" Margin="5,0,0,0" TextWrapping="Wrap" />
                            <TextBlock Text="{Binding Path=Autor}" 
                                       FontSize="16" Margin="15,0,0,0"/>
                            <TextBlock Text="{Binding Path=Fecha}" 
                                       FontSize="16" Margin="15,0,0,0"/>
                        </StackPanel>
                    </DataTemplate>
                </ListView.ItemTemplate>
            </ListView>

            <!-- Cuerpo del Post -->
            <!-- Se bindea en función del elemento seleccionado del ItemListView -->
            <Grid DataContext="{Binding ElementName=ItemListView, Path=SelectedItem}"
                  Grid.Column="1" Margin="25,0,0,0">
                <!-- Como antes le decimos que tiene 2 filas. La primera será un TextBlock con el título del Post y la segunda el control WebView que mostrará el contenido del post -->
                <Grid.RowDefinitions>
                    <RowDefinition Height="Auto" />
                    <RowDefinition Height="*" />
                </Grid.RowDefinitions>
                <TextBlock Text="{Binding Path=Titulo}" FontSize="24"/>
                <WebView x:Name="ContentView" Grid.Row="1" Margin="0,5,20,20"/>
            </Grid>
        </Grid>
    </Grid>
```

¡Casi casi lo tenemos! Como veis en los comentarios el Grid “Cuerpo” se **bindea en función del elemento seleccionado del ItemListView** (lo que nos ahorra bastante código) pero ¿cómo cargamos el contenido HTML dentro del WebView?

Bueno, nos toca usar el evento “**ItemListView_SelectionChanged**” del **ListView** para ello:

```csharp
private void ItemListView_SelectionChanged(object sender, SelectionChangedEventArgs e)
        {
            FeedItem feedItem = (sender as ListView).SelectedItem as FeedItem;
            if (feedItem != null)
            {
                // Si tiene contenido simplemente se lo "encasquetamos" al WebView y el solito interpreta el HTML como si fuera un navegador.
                ContentView.NavigateToString(feedItem.Contenido);
            }
        }
```

**¡Ya está!** Tenemos nuestra primera aplicación Metro que encima es útil y hasta bonita!!

![](https://escapistas.blob.core.windows.net/compilando-post-images/metrovs2.png)

# Conclusiones

Las nuevas mejoras de C# 5 sobre sincronía son fantásticas ya que nos permite mover el flujo de la aplicación como queramos de forma muy sencilla aunque quizá haya que aprender bien a como y cuando usarlas y sobre la interfaz Metro que decir… yo estoy  enamorado. Decían que Microsoft tenía mal gusto… Quien rie el último… jeje !

No voy a entrar en polémicas sobre si se llevará un batacazo o no, sobre si la gente sabrá usar la nueva interfaz o si se perderá en el intento, si Windows se ha reinventado a si mismo… sólo sé que este movimiento de Microsoft facilitando las herramientas de desarrollo con tanto tiempo de antelación de manera gratuita para que todos las probemos no es una casualidad. Cuando tengamos Windows 8 entre nuestras manos ya habrán cientos de apps listas para ser usadas e incluidas en al Windows Store que funcionarán tanto en Windows 8 como en Windows Phone y yo, por mi parte, no pienso perder la oportunidad de seguir trabajando con esta tecnología !


¡Nos vemos Compilando!