---
title: Datos de ruta de acceso SVG de SkiaSharp
description: En este artículo se explica cómo definir rutas de acceso de SkiaSharp mediante cadenas de texto en el formato de Scalable Vector Graphics y esto se muestra con código de ejemplo.
ms.prod: xamarin
ms.technology: xamarin-skiasharp
ms.assetid: 1D53067B-3502-4D74-B89D-7EC496901AE2
author: davidbritch
ms.author: dabritch
ms.date: 05/24/2017
ms.openlocfilehash: 3571da005f48bda630b4202b804f3efe9c892f60
ms.sourcegitcommit: 5fc171a45697f7c610d65f74d1f3cebbac445de6
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 11/20/2018
ms.locfileid: "52172332"
---
# <a name="svg-path-data-in-skiasharp"></a>Datos de ruta de acceso SVG de SkiaSharp

_Definir rutas de acceso mediante cadenas de texto en el formato de Scalable Vector Graphics_

El [ `SKPath` ](xref:SkiaSharp.SKPath) clase admite la definición de objetos de ruta de acceso completa de las cadenas de texto en un formato que se establece mediante la especificación de Scalable Vector Graphics (SVG). Más adelante en este artículo verá cómo puede representar una ruta de acceso completa en una cadena de texto como esta:

![](path-data-images/pathdatasample.png "Una ruta de acceso de ejemplo definida con los datos de ruta de acceso SVG")

SVG es un lenguaje de programación de páginas web de gráficos basados en XML. Dado que SVG debe permitir que las rutas de acceso que se definen en el marcado en lugar de una serie de llamadas de función, SVG estándar incluye una manera muy concisa de especificar una ruta de acceso completa de gráficos como una cadena de texto.

Dentro de SkiaSharp, este formato se conoce como "SVG ruta de acceso de datos". El formato también se admite en entornos de programación basado en XAML de Windows, incluidos la Windows Presentation Foundation y la plataforma Universal de Windows, donde se conoce como el [sintaxis de marcado de trazados](/dotnet/framework/wpf/graphics-multimedia/path-markup-syntax) o [mover y dibuje la sintaxis de comandos](/windows/uwp/xaml-platform/move-draw-commands-syntax/). También puede servir como un formato de intercambio de gráficos vectoriales, especialmente en los archivos basado en texto, como XML.

El [ `SKPath` ](xref:SkiaSharp.SKPath) clase define dos métodos con las palabras `SvgPathData` en sus nombres:

```csharp
public static SKPath ParseSvgPathData(string svgPath)

public string ToSvgPathData()
```

Estático [ `ParseSvgPathData` ](xref:SkiaSharp.SKPath.ParseSvgPathData(System.String)) método convierte una cadena en un `SKPath` objeto, mientras que [ `ToSvgPathData` ](xref:SkiaSharp.SKPath.ToSvgPathData) convierte un `SKPath` objeto en una cadena.

Esta es una cadena SVG para una estrella de cinco puntas centrada en el punto (0, 0) con un radio de 100:

```
"M 0 -100 L 58.8 90.9, -95.1 -30.9, 95.1 -30.9, -58.8 80.9 Z"
```

Las letras son comandos que crean un `SKPath` objeto: `M` indica un `MoveTo` llamar, `L` es `LineTo`, y `Z` es `Close` para cerrar un contorno. Cada par de números proporciona una coordenada X e Y de un punto. Tenga en cuenta que el `L` comando va seguido de varios puntos separados por comas. En una serie de coordenadas y puntos, comas y espacios en blanco se tratan de forma idéntica. Algunos programadores prefieren poner comas entre las coordenadas X e Y, en lugar de entre los puntos, pero comas o espacios sólo son necesarias para evitar la ambigüedad. Esto es perfectamente válido:

```
"M0-100L58.8 90.9-95.1-30.9 95.1-30.9-58.8 80.9Z"
```

La sintaxis de los datos de ruta de acceso SVG está documentada formalmente en [8.3 de la sección de la especificación de SVG](http://www.w3.org/TR/SVG11/paths.html#PathData). Este es un resumen:

## <a name="moveto"></a>**MoveTo**

```
M x y
```

Esto inicia un nuevo contorno en la ruta de acceso estableciendo la posición actual. Datos de ruta de acceso siempre deben comenzar por una `M` comando.

## <a name="lineto"></a>**LineTo**

```
L x y ...
```

Este comando agrega una línea recta (o líneas) a la ruta de acceso y establece la nueva posición actual hasta el final de la última línea. Puede seguir el `L` con varios pares de *x* y *y* coordenadas.

## <a name="horizontal-lineto"></a>**LineTo horizontal**

```
H x ...
```

Este comando agrega una línea horizontal a la ruta de acceso y establece la nueva posición actual hasta el final de la línea. Puede seguir el `H` comando con varios *x* coordenadas, pero no tiene mucho sentido.

## <a name="vertical-line"></a>**Línea vertical**

```
V y ...
```

Este comando agrega una línea vertical a la ruta de acceso y establece la nueva posición actual hasta el final de la línea.

## <a name="close"></a>**Cerrar**

```
Z
```

El `C` comando cierra el contorno agregando una línea recta desde la posición actual hasta el principio del contorno.

## <a name="arcto"></a>**ArcTo**

El comando para agregar un arco elíptico al contorno, sin duda es el comando más complejo en toda la especificación de datos de ruta de acceso SVG. Es el único comando en el que los números pueden representar un valor distinto de los valores de coordenadas:

```
A rx ry rotation-angle large-arc-flag sweep-flag x y ...
```

El *rx* y *ntentar* parámetros son los radios horizontales y verticales de la elipse. El *ángulo de giro* es hacia la derecha en grados.

Establecer el *gran marca de arco* en 1 del arco grandes o en 0 para el arco pequeño.

Establecer el *barrido marca* en 1 para a la derecha y en 0 para hacia la izquierda.

El arco se dibuja en el punto (*x*, *y*), que se convierte en la nueva posición actual.

## <a name="cubicto"></a>**CubicTo**

```
C x1 y1 x2 y2 x3 y3 ...
```

Este comando agrega una curva Bézier cúbica desde la posición actual hasta (*x3*, *y3*), que se convierte en la nueva posición actual. Los puntos (*x1*, *y1*) y (*x2*, *y2*) son puntos de control.

Se pueden especificar varias curvas Bézier mediante un único `C` comando. El número de puntos debe ser un múltiplo de 3.

También es un comando de curva Bézier "suave":

```
S x2 y2 x3 y3 ...
```

Este comando debe seguir un comando de Bézier normal (aunque no es estrictamente necesario). El comando de Bézier smooth calcula el primer punto de control para que sea un reflejo del segundo punto de control de la anterior Bézier en torno a su punto mutua. Estos tres puntos, por tanto, son colineales, y la conexión entre las dos curvas de Bézier es suave.

## <a name="quadto"></a>**QuadTo**

```
Q x1 y1 x2 y2 ...
```

Para curvas de Bézier cuadráticas, el número de puntos debe ser un múltiplo de 2. Es el punto de control (*x1*, *y1*) y el punto final (y la nueva posición actual) son (*x2*, *y2*)

También es un comando de curva cuadrática suavizada:

```
T x2 y2 ...
```

El punto de control se calcula basándose en el punto de control de la curva cuadrática anterior.

Todos estos comandos también están disponibles en las versiones "relativas", donde los puntos de coordenadas son relativas a la posición actual. Estos comandos relativos empiezan con letras en minúsculas, por ejemplo, `c` lugar `C` para la versión del comando Bézier cúbica relativa.

Se trata de la extensión de la definición de datos de ruta de acceso SVG. Hay una funcionalidad para grupos de comandos de repetición o para realizar cualquier tipo de cálculo. Comandos para `ConicTo` o no están disponibles los otros tipos de especificaciones del arco.

Estático [ `SKPath.ParseSvgPathData` ](xref:SkiaSharp.SKPath.ParseSvgPathData(System.String)) método espera una cadena válida de comandos SVG. Si se detecta algún error de sintaxis, el método devuelve `null`. Es la indicación de error único.

El [ `ToSvgPathData` ](xref:SkiaSharp.SKPath.ToSvgPathData) método es muy útil para obtener datos de ruta de acceso SVG a partir de una máquina `SKPath` objetos para transferir a otro programa, o para almacenar en un formato de archivo basado en texto, como XML. (El `ToSvgPathData` método no se muestra en el código de ejemplo en este artículo.) Hacer *no* espera `ToSvgPathData` para devolver una cadena que corresponde exactamente a las llamadas de método que crea la ruta de acceso. En concreto, descubrirá que los arcos se convierten a varios `QuadTo` comandos, y eso es cómo aparecen estas en los datos de ruta de acceso devuelto desde `ToSvgPathData`.

El **Hello de ruta de acceso de datos** página hechizos fuera la palabra "HELLO" con datos de ruta de acceso SVG. Tanto el `SKPath` y `SKPaint` objetos se definen como campos en el [ `PathDataHelloPage` ](https://github.com/xamarin/xamarin-forms-samples/blob/master/SkiaSharpForms/Demos/Demos/SkiaSharpFormsDemos/Curves/PathDataHelloPage.cs) clase:

```csharp
public class PathDataHelloPage : ContentPage
{
    SKPath helloPath = SKPath.ParseSvgPathData(
        "M 0 0 L 0 100 M 0 50 L 50 50 M 50 0 L 50 100" +                // H
        "M 125 0 C 60 -10, 60 60, 125 50, 60 40, 60 110, 125 100" +     // E
        "M 150 0 L 150 100, 200 100" +                                  // L
        "M 225 0 L 225 100, 275 100" +                                  // L
        "M 300 50 A 25 50 0 1 0 300 49.9 Z");                           // O

    SKPaint paint = new SKPaint
    {
        Style = SKPaintStyle.Stroke,
        Color = SKColors.Blue,
        StrokeWidth = 10,
        StrokeCap = SKStrokeCap.Round,
        StrokeJoin = SKStrokeJoin.Round
    };
    ...
}
```

La ruta de acceso a la definición de la cadena de texto comienza en la esquina superior izquierda en el punto (0, 0). Cada letra es 50 unidades de anchas y alto de 100 unidades y las letras se separan mediante otro 25 unidades, lo que significa que la ruta de acceso completa es 350 unidades amplias.

'H' de "Hello" se compone de tres perfiles de una línea, mientras que 'E' es dos curvas de Bézier cúbicas conectadas. Tenga en cuenta que el `C` comando va seguido de seis puntos y dos de los puntos de control tienen las coordenadas Y de -10 y 110, lo que los coloca fuera del intervalo de las coordenadas Y de las demás letras. La 'L' es de dos líneas conectadas, mientras que la ' o ' es una elipse que se representa con un `A` comando.

Tenga en cuenta que el `M` comando que comienza el contorno de la última establece la posición en el punto (350, 50), que es el centro vertical de la izquierda en paralelo de la ' o '. Tal como se indica por los siguientes números primeros el `A` comando, la elipse tiene un radio horizontal del 25 y un radio vertical de 50. El punto final se indica mediante el último par de números en el `A` comando, que representa el punto (300, 49,9). Que es deliberadamente es ligeramente diferente desde el punto inicial. Si el punto de conexión se establece igual que el punto inicial, no se representará el arco. Para dibujar una elipse completa, debe establecer el punto de conexión cercano a (pero no es igual a) el punto de inicio, o bien debe usar dos o más `A` comandos, cada uno para la parte de la elipse completa.

Es posible que desee agregar la siguiente instrucción al constructor de la página y, a continuación, establezca un punto de interrupción para examinar la cadena resultante:

```csharp
string str = helloPath.ToSvgPathData();
```

Descubrirá que el arco se ha reemplazado por una larga serie de `Q` comandos para una aproximación por etapas del arco con curvas Bézier cuadráticas.

El `PaintSurface` controlador obtiene los límites de una estrecha de la ruta de acceso, que no incluye los puntos de control para 'E' y ' o ' curvas. Las tres transformaciones mover el centro de la ruta de acceso al punto (0, 0), escale la ruta de acceso para el tamaño de lienzo (pero teniendo en cuenta también el ancho del trazo) y, a continuación, mover el centro de la ruta de acceso al centro del lienzo:

```csharp
public class PathDataHelloPage : ContentPage
{
    ...
    void OnCanvasViewPaintSurface(object sender, SKPaintSurfaceEventArgs args)
    {
        SKImageInfo info = args.Info;
        SKSurface surface = args.Surface;
        SKCanvas canvas = surface.Canvas;

        canvas.Clear();

        SKRect bounds;
        helloPath.GetTightBounds(out bounds);

        canvas.Translate(info.Width / 2, info.Height / 2);

        canvas.Scale(info.Width / (bounds.Width + paint.StrokeWidth),
                     info.Height / (bounds.Height + paint.StrokeWidth));

        canvas.Translate(-bounds.MidX, -bounds.MidY);

        canvas.DrawPath(helloPath, paint);
    }
}
```

La ruta de acceso rellena el lienzo, que es más razonable cuando se ve en modo horizontal:

[![](path-data-images/pathdatahello-small.png "Captura de pantalla de la página Hello de ruta de acceso datos triple")](path-data-images/pathdatahello-large.png#lightbox "Triple captura de pantalla de la página Hello de ruta de acceso datos")

El **Cat de datos de ruta de acceso** página es similar. Los objetos de ruta de acceso y dibujo se definen como campos en el [ `PathDataCatPage` ](https://github.com/xamarin/xamarin-forms-samples/blob/master/SkiaSharpForms/Demos/Demos/SkiaSharpFormsDemos/Curves/PathDataCatPage.cs) clase:

```csharp
public class PathDataCatPage : ContentPage
{
    SKPath catPath = SKPath.ParseSvgPathData(
        "M 160 140 L 150 50 220 103" +              // Left ear
        "M 320 140 L 330 50 260 103" +              // Right ear
        "M 215 230 L 40 200" +                      // Left whiskers
        "M 215 240 L 40 240" +
        "M 215 250 L 40 280" +
        "M 265 230 L 440 200" +                     // Right whiskers
        "M 265 240 L 440 240" +
        "M 265 250 L 440 280" +
        "M 240 100" +                               // Head
        "A 100 100 0 0 1 240 300" +
        "A 100 100 0 0 1 240 100 Z" +
        "M 180 170" +                               // Left eye
        "A 40 40 0 0 1 220 170" +
        "A 40 40 0 0 1 180 170 Z" +
        "M 300 170" +                               // Right eye
        "A 40 40 0 0 1 260 170" +
        "A 40 40 0 0 1 300 170 Z");

    SKPaint paint = new SKPaint
    {
        Style = SKPaintStyle.Stroke,
        Color = SKColors.Orange,
        StrokeWidth = 5
    };
    ...
}
```

El encabezado de un gato es un círculo, y aquí se representa con dos `A` comandos, cada uno de los cuales dibuja un semicírculo. Ambos `A` los comandos para el encabezado define radios horizontal y vertical de 100. El primer arco comienza en (240, 100) y termina en (240, 300), que se convierte en el punto inicial del segundo arco que termina en (240, 100).

Los dos ojos también se representan con dos `A` comandos y al igual que con la cabeza del cat, el segundo `A` comando finaliza en el mismo punto como el comienzo de la primera `A` comando. Sin embargo, estos pares de `A` comandos no definen una elipse. El con cada arco es 40 unidades y el radio también es 40 unidades, lo que significa que estos arcos no semicircles completas.

El `PaintSurface` controlador realiza transformaciones similar que en el ejemplo anterior, pero se establece un valor single `Scale` factor que hay que mantener la relación de aspecto y proporcionar un poco margen para bigotes de cat no toque los lados de la pantalla:

```csharp
public class PathDataCatPage : ContentPage
{
    ...
    void OnCanvasViewPaintSurface(object sender, SKPaintSurfaceEventArgs args)
    {
        SKImageInfo info = args.Info;
        SKSurface surface = args.Surface;
        SKCanvas canvas = surface.Canvas;

        canvas.Clear(SKColors.Black);

        SKRect bounds;
        catPath.GetBounds(out bounds);

        canvas.Translate(info.Width / 2, info.Height / 2);

        canvas.Scale(0.9f * Math.Min(info.Width / bounds.Width,
                                     info.Height / bounds.Height));

        canvas.Translate(-bounds.MidX, -bounds.MidY);

        canvas.DrawPath(catPath, paint);
    }
}
```

Este es el programa que se ejecuta:

[![](path-data-images/pathdatacat-small.png "Captura de pantalla triple de la página de la ruta de acceso datos Cat")](path-data-images/pathdatacat-large.png#lightbox "Triple captura de pantalla de la página de Cat de datos de ruta de acceso")

Normalmente, cuando un `SKPath` objeto se define como un campo, los contornos de la ruta de acceso deben definirse en el constructor u otro método. Sin embargo, cuando se usan datos de ruta de acceso SVG, hemos visto que se puede especificar la ruta de acceso completamente en la definición del campo.

La anterior **feo reloj analógico** ejemplo en el [ **girar la transformación** ](~/xamarin-forms/user-interface/graphics/skiasharp/transforms/rotate.md) artículo muestra las manecillas del reloj como líneas simples. El **bastante analógico reloj** programa siguiente reemplaza dichas líneas con `SKPath` objetos definidos como campos en el [ `PrettyAnalogClockPage` ](https://github.com/xamarin/xamarin-forms-samples/blob/master/SkiaSharpForms/Demos/Demos/SkiaSharpFormsDemos/Curves/PrettyAnalogClockPage.cs) junto con la clase `SKPaint` objetos:

```csharp
public class PrettyAnalogClockPage : ContentPage
{
    ...
    // Clock hands pointing straight up
    SKPath hourHandPath = SKPath.ParseSvgPathData(
        "M 0 -60 C   0 -30 20 -30  5 -20 L  5   0" +
                "C   5 7.5 -5 7.5 -5   0 L -5 -20" +
                "C -20 -30  0 -30  0 -60 Z");

    SKPath minuteHandPath = SKPath.ParseSvgPathData(
        "M 0 -80 C   0 -75  0 -70  2.5 -60 L  2.5   0" +
                "C   2.5 5 -2.5 5 -2.5   0 L -2.5 -60" +
                "C 0 -70  0 -75  0 -80 Z");

    SKPath secondHandPath = SKPath.ParseSvgPathData(
        "M 0 10 L 0 -80");

    // SKPaint objects
    SKPaint handStrokePaint = new SKPaint
    {
        Style = SKPaintStyle.Stroke,
        Color = SKColors.Black,
        StrokeWidth = 2,
        StrokeCap = SKStrokeCap.Round
    };

    SKPaint handFillPaint = new SKPaint
    {
        Style = SKPaintStyle.Fill,
        Color = SKColors.Gray
    };
    ...
}
```

Ahora las manos hour y minute han entre áreas. Para realizar estas manos distinguirse entre sí, se dibujan con un contorno negro y un relleno de color gris utilizando el `handStrokePaint` y `handFillPaint` objetos.

En el anterior **feo reloj analógico** ejemplo, el pequeño círculos que marcan las horas y minutos que dibujó en un bucle. En este **bastante analógico reloj** ejemplo, se usa un enfoque totalmente diferente: las marcas de hora y minuto son líneas de puntos que se dibuja con el `minuteMarkPaint` y `hourMarkPaint` objetos:

```csharp
public class PrettyAnalogClockPage : ContentPage
{
    ...
    SKPaint minuteMarkPaint = new SKPaint
    {
        Style = SKPaintStyle.Stroke,
        Color = SKColors.Black,
        StrokeWidth = 3,
        StrokeCap = SKStrokeCap.Round,
        PathEffect = SKPathEffect.CreateDash(new float[] { 0, 3 * 3.14159f }, 0)
    };

    SKPaint hourMarkPaint = new SKPaint
    {
        Style = SKPaintStyle.Stroke,
        Color = SKColors.Black,
        StrokeWidth = 6,
        StrokeCap = SKStrokeCap.Round,
        PathEffect = SKPathEffect.CreateDash(new float[] { 0, 15 * 3.14159f }, 0)
    };
    ...
}
```

El [ **puntos y guiones** ](~/xamarin-forms/user-interface/graphics/skiasharp/paths/dots.md) artículo describe cómo puede usar el [ `SKPathEffect.CreateDash` ](xref:SkiaSharp.SKPathEffect.CreateDash*) método para crear una línea discontinua. El primer argumento es un `float` matriz que generalmente tiene dos elementos: el primer elemento es la longitud de los guiones, y el segundo elemento es la separación entre los guiones. Cuando el `StrokeCap` propiedad está establecida en `SKStrokeCap.Round`, a continuación, los extremos del guión redondeados eficazmente alargar la longitud del guión por el ancho del trazo en ambos lados del guión. Por lo tanto, si el primer elemento de matriz se establece en 0, crea una línea de puntos.

La distancia entre estos puntos se rige por el segundo elemento de matriz. Como verá en breve, estos dos `SKPaint` objetos se utilizan para dibujar círculos con un radio de 90 unidades. Por lo tanto, la circunferencia de este círculo es 180π, lo que significa que las marcas de 60 minutos deben aparecer cada unidades 3π, que es el segundo valor en el `float` de matriz en `minuteMarkPaint`. Las marcas de hora 12 deben aparecer cada unidades 15π, que es el valor en el segundo `float` matriz.

El `PrettyAnalogClockPage` clase establece un temporizador para invalidar la superficie cada 16 milisegundos y el `PaintSurface` controlador se llama a este ritmo. Las definiciones anteriores de la `SKPath` y `SKPaint` objetos permiten muy limpios al código de dibujo:

```csharp
public class PrettyAnalogClockPage : ContentPage
{
    ...
    void OnCanvasViewPaintSurface(object sender, SKPaintSurfaceEventArgs args)
    {
        SKImageInfo info = args.Info;
        SKSurface surface = args.Surface;
        SKCanvas canvas = surface.Canvas;

        canvas.Clear();

        // Transform for 100-radius circle in center
        canvas.Translate(info.Width / 2, info.Height / 2);
        canvas.Scale(Math.Min(info.Width / 200, info.Height / 200));

        // Draw circles for hour and minute marks
        SKRect rect = new SKRect(-90, -90, 90, 90);
        canvas.DrawOval(rect, minuteMarkPaint);
        canvas.DrawOval(rect, hourMarkPaint);

        // Get time
        DateTime dateTime = DateTime.Now;

        // Draw hour hand
        canvas.Save();
        canvas.RotateDegrees(30 * dateTime.Hour + dateTime.Minute / 2f);
        canvas.DrawPath(hourHandPath, handStrokePaint);
        canvas.DrawPath(hourHandPath, handFillPaint);
        canvas.Restore();

        // Draw minute hand
        canvas.Save();
        canvas.RotateDegrees(6 * dateTime.Minute + dateTime.Second / 10f);
        canvas.DrawPath(minuteHandPath, handStrokePaint);
        canvas.DrawPath(minuteHandPath, handFillPaint);
        canvas.Restore();

        // Draw second hand
        double t = dateTime.Millisecond / 1000.0;

        if (t < 0.5)
        {
            t = 0.5 * Easing.SpringIn.Ease(t / 0.5);
        }
        else
        {
            t = 0.5 * (1 + Easing.SpringOut.Ease((t - 0.5) / 0.5));
        }

        canvas.Save();
        canvas.RotateDegrees(6 * (dateTime.Second + (float)t));
        canvas.DrawPath(secondHandPath, handStrokePaint);
        canvas.Restore();
    }
}
```

Sin embargo algo especial se realiza con la mano de segundo. Dado que el reloj se actualiza cada 16 milisegundos, el `Millisecond` propiedad de la `DateTime` valor puede emplearse para animar un barrido de segunda mano en lugar de uno que se mueve en saltos discretos de segundo a segundo. Pero este código no permite el movimiento a ser suaves. En su lugar, usa Xamarin.Forms [ `SpringIn` ](xref:Xamarin.Forms.Easing.SpringIn) y [ `SpringOut` ](xref:Xamarin.Forms.Easing.SpringOut) animación funciones para otro tipo de movimiento de aceleración. Estas funciones de aceleración provocar el segundero mover de una manera es &mdash; extrayendo volver un poco antes de que lo mueve, y, a continuación, ligeramente disparen su destino, un efecto que lamentablemente no se puede reproducir en estas capturas de pantalla estáticas:

[![](path-data-images/prettyanalogclock-small.png "Captura de pantalla de la página bastante analógico reloj triple")](path-data-images/prettyanalogclock-large.png#lightbox "Triple captura de pantalla de la página bastante analógico reloj")


## <a name="related-links"></a>Vínculos relacionados

- [API de SkiaSharp](https://docs.microsoft.com/dotnet/api/skiasharp)
- [SkiaSharpFormsDemos (ejemplo)](https://developer.xamarin.com/samples/xamarin-forms/SkiaSharpForms/Demos/)
