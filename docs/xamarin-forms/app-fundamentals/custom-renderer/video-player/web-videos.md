---
title: Reproducir un vídeo de Web
description: En este artículo se explica cómo reproducir vídeos web en una aplicación de Reproductor de vídeo mediante Xamarin.Forms.
ms.prod: xamarin
ms.assetid: 75781A10-865D-4BA8-8D6B-E3DA012922BC
ms.technology: xamarin-forms
author: davidbritch
ms.author: dabritch
ms.date: 02/12/2018
ms.openlocfilehash: 7f40d0d11fc932121b4ff7789969bbb1e354024c
ms.sourcegitcommit: 5fc171a45697f7c610d65f74d1f3cebbac445de6
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 11/20/2018
ms.locfileid: "52172215"
---
# <a name="playing-a-web-video"></a>Reproducir un vídeo de Web

El `VideoPlayer` clase define un `Source` propiedad que se utiliza para especificar el origen del archivo de vídeo, así como un `AutoPlay` propiedad. `AutoPlay` tiene un valor predeterminado de `true`, lo que significa que el vídeo debería comenzar a reproducirse automáticamente una vez `Source` se ha establecido:

```csharp
using System;
using Xamarin.Forms;

namespace FormsVideoLibrary
{
    public class VideoPlayer : View, IVideoPlayerController
    {
        ···
        // Source property
        public static readonly BindableProperty SourceProperty =
            BindableProperty.Create(nameof(Source), typeof(VideoSource), typeof(VideoPlayer), null);

        [TypeConverter(typeof(VideoSourceConverter))]
        public VideoSource Source
        {
            set { SetValue(SourceProperty, value); }
            get { return (VideoSource)GetValue(SourceProperty); }
        }

        // AutoPlay property
        public static readonly BindableProperty AutoPlayProperty =
            BindableProperty.Create(nameof(AutoPlay), typeof(bool), typeof(VideoPlayer), true);

        public bool AutoPlay
        {
            set { SetValue(AutoPlayProperty, value); }
            get { return (bool)GetValue(AutoPlayProperty); }
        }
        ···
    }
}
```

El `Source` propiedad es de tipo `VideoSource`, que está basado en Xamarin.Forms [ `ImageSource` ](xref:Xamarin.Forms.ImageSource) abstracta de la clase y sus tres derivados [ `UriImageSource` ](xref:Xamarin.Forms.UriImageSource), [ `FileImageSource` ](xref:Xamarin.Forms.FileImageSource), y [ `StreamImageSource` ](xref:Xamarin.Forms.StreamImageSource). No hay ninguna opción de la secuencia está disponible para el `VideoPlayer` sin embargo, porque iOS y Android no admiten reproducir un vídeo desde una secuencia.

## <a name="video-sources"></a>Orígenes de vídeo

El resumen `VideoSource` clase consta únicamente de los tres métodos estáticos que crean instancias de las tres clases que derivan de `VideoSource`:

```csharp
namespace FormsVideoLibrary
{
    [TypeConverter(typeof(VideoSourceConverter))]
    public abstract class VideoSource : Element
    {
        public static VideoSource FromUri(string uri)
        {
            return new UriVideoSource() { Uri = uri };
        }

        public static VideoSource FromFile(string file)
        {
            return new FileVideoSource() { File = file };
        }

        public static VideoSource FromResource(string path)
        {
            return new ResourceVideoSource() { Path = path };
        }
    }
}
```

La `UriVideoSource` clase se utiliza para especificar un archivo de vídeo descargable con un URI. Define una propiedad única de tipo `string`:

```csharp
namespace FormsVideoLibrary
{
    public class UriVideoSource : VideoSource
    {
        public static readonly BindableProperty UriProperty =
            BindableProperty.Create(nameof(Uri), typeof(string), typeof(UriVideoSource));

        public string Uri
        {
            set { SetValue(UriProperty, value); }
            get { return (string)GetValue(UriProperty); }
        }
    }
}
```

Los objetos de tipo de control `UriVideoSource` se describe a continuación.

El `ResourceVideoSource` clase se utiliza para tener acceso a archivos de vídeo que se almacenan como recursos incrustados en la aplicación de plataforma, también se especifica con un `string` propiedad:

```csharp
namespace FormsVideoLibrary
{
    public class ResourceVideoSource : VideoSource
    {
        public static readonly BindableProperty PathProperty =
            BindableProperty.Create(nameof(Path), typeof(string), typeof(ResourceVideoSource));

        public string Path
        {
            set { SetValue(PathProperty, value); }
            get { return (string)GetValue(PathProperty); }
        }
    }
}
```

Los objetos de tipo de control `ResourceVideoSource` se describe en el artículo [cargar vídeos de recursos de aplicación](loading-resources.md). La `VideoPlayer` clase no tiene ninguna facilidad para cargar un archivo de vídeo almacenado como un recurso en la biblioteca estándar. NET.

La `FileVideoSource` clase se utiliza para tener acceso a archivos de vídeo desde la biblioteca de vídeos del dispositivo. También es la única propiedad de tipo `string`:

```csharp
namespace FormsVideoLibrary
{
    public class FileVideoSource : VideoSource
    {
        public static readonly BindableProperty FileProperty =
                  BindableProperty.Create(nameof(File), typeof(string), typeof(FileVideoSource));

        public string File
        {
            set { SetValue(FileProperty, value); }
            get { return (string)GetValue(FileProperty); }
        }
    }
}
```

Los objetos de tipo de control `FileVideoSource` se describe en el artículo [acceso a la biblioteca de vídeos del dispositivo](accessing-library.md).

El `VideoSource` clase incluye un `TypeConverter` atributo que hace referencia a `VideoSourceConverter`:

```csharp
namespace FormsVideoLibrary
{
    [TypeConverter(typeof(VideoSourceConverter))]
    public abstract class VideoSource : Element
    {
        ···
    }
}
```

Se invoca este convertidor de tipos cuando el `Source` propiedad se establece en una cadena en XAML. Este es el `VideoSourceConverter` clase:

```csharp
namespace FormsVideoLibrary
{
    public class VideoSourceConverter : TypeConverter
    {
        public override object ConvertFromInvariantString(string value)
        {
            if (!String.IsNullOrWhiteSpace(value))
            {
                Uri uri;
                return Uri.TryCreate(value, UriKind.Absolute, out uri) && uri.Scheme != "file" ?
                                VideoSource.FromUri(value) : VideoSource.FromResource(value);
            }

            throw new InvalidOperationException("Cannot convert null or whitespace to ImageSource");
        }
    }
}
```

El `ConvertFromInvariantString` método intenta convertir la cadena en un `Uri` objeto. Si lo consigue, y el esquema no es `file:`, el método devuelve un `UriVideoSource`. De lo contrario, devuelve un `ResourceVideoSource`.

## <a name="setting-the-video-source"></a>Establecer el origen de vídeo

Toda la otra lógica que implican los orígenes de vídeo se implementa en los representadores de plataforma individuales. Las secciones siguientes muestran cómo los representadores de plataforma reproducirán vídeos cuando la `Source` propiedad está establecida en un `UriVideoSource` objeto.

### <a name="the-ios-video-source"></a>El origen de vídeo de iOS

Dos secciones de la `VideoPlayerRenderer` están implicados en la configuración del origen de vídeo del Reproductor de vídeo. Cuando Xamarin.Forms crea primero un objeto de tipo `VideoPlayer`, `OnElementChanged` se llama al método con el `NewElement` propiedad del objeto de argumentos que se establece en el que `VideoPlayer`. El `OnElementChanged` llamadas al método `SetSource`:

```csharp
namespace FormsVideoLibrary.iOS
{
    public class VideoPlayerRenderer : ViewRenderer<VideoPlayer, UIView>
    {
        ···
        protected override void OnElementChanged(ElementChangedEventArgs<VideoPlayer> args)
        {
            ···
            if (args.NewElement != null)
            {
                ···
                SetSource();
                ···
            }
        }

        protected override void OnElementPropertyChanged(object sender, PropertyChangedEventArgs args)
        {
            ···
            else if (args.PropertyName == VideoPlayer.SourceProperty.PropertyName)
            {
                SetSource();
            }
            ···
        }
        ···
    }
}
```

Más adelante en, cuando el `Source` se cambia la propiedad, el `OnElementPropertyChanged` se llama al método con un `PropertyName` propiedad de "Origen", y `SetSource` se llama de nuevo.

Para reproducir un archivo de vídeo en iOS, un objeto de tipo [ `AVAsset` ](https://developer.xamarin.com/api/type/AVFoundation.AVAsset/) se crea por primera vez para encapsular el archivo de vídeo, y que se usa para crear un [ `AVPlayerItem` ](https://developer.xamarin.com/api/type/AVFoundation.AVPlayerItem/), que, a continuación, se pasa a la `AVPlayer`objeto. Este es el modo `SetSource` método controla el `Source` cuando es de tipo de propiedad `UriVideoSource`:

```csharp
namespace FormsVideoLibrary.iOS
{
    public class VideoPlayerRenderer : ViewRenderer<VideoPlayer, UIView>
    {
        AVPlayer player;
        AVPlayerItem playerItem;
        ···
        void SetSource()
        {
            AVAsset asset = null;

            if (Element.Source is UriVideoSource)
            {
                string uri = (Element.Source as UriVideoSource).Uri;

                if (!String.IsNullOrWhiteSpace(uri))
                {
                    asset = AVAsset.FromUrl(new NSUrl(uri));
                }
            }
            ···
            if (asset != null)
            {
                playerItem = new AVPlayerItem(asset);
            }
            else
            {
                playerItem = null;
            }

            player.ReplaceCurrentItemWithPlayerItem(playerItem);

            if (playerItem != null && Element.AutoPlay)
            {
                player.Play();
            }
        }
        ···
    }
}
```

El `AutoPlay` propiedad no tiene ningún analógico en las clases de vídeo de iOS, por lo que se examina la propiedad al final de la `SetSource` método para llamar a la `Play` método en el `AVPlayer` objeto.

En algunos casos, vídeos continuación reproduciendo después de la página con el `VideoPlayer` vuelve a navegar a la página principal. Para detener el vídeo, el `ReplaceCurrentItemWithPlayerItem` también se establece el `Dispose` invalidar:

```csharp
namespace FormsVideoLibrary.iOS
{
    public class VideoPlayerRenderer : ViewRenderer<VideoPlayer, UIView>
    {
        ···
        protected override void Dispose(bool disposing)
        {
            base.Dispose(disposing);

            if (player != null)
            {
                player.ReplaceCurrentItemWithPlayerItem(null);
            }
        }
        ···
    }
}
```

### <a name="the-android-video-source"></a>El origen de vídeo de Android

Android `VideoPlayerRenderer` debe establecer el origen del Reproductor de vídeo cuando la `VideoPlayer` se primero crea y posterior cuando la `Source` los cambios de propiedad:

```csharp
namespace FormsVideoLibrary.Droid
{
    public class VideoPlayerRenderer : ViewRenderer<VideoPlayer, ARelativeLayout>
    {
        ···
        protected override void OnElementChanged(ElementChangedEventArgs<VideoPlayer> args)
        {
            ···
            if (args.NewElement != null)
            {
                ···
                SetSource();
                ···
            }
        }
        ···
        protected override void OnElementPropertyChanged(object sender, PropertyChangedEventArgs args)
        {
            ···
            else if (args.PropertyName == VideoPlayer.SourceProperty.PropertyName)
            {
                SetSource();
            }
            ···
        }
        ···
    }
}
```

El `SetSource` método controla los objetos de tipo `UriVideoSource` mediante una llamada a `SetVideoUri` en el `VideoView` con Android `Uri` objeto creado a partir de la cadena de URI. El `Uri` clase debe calificarse por completo aquí para distinguirlo de .NET `Uri` clase:

```csharp
namespace FormsVideoLibrary.Droid
{
    public class VideoPlayerRenderer : ViewRenderer<VideoPlayer, ARelativeLayout>
    {
        ···
        void SetSource()
        {
            isPrepared = false;
            bool hasSetSource = false;

            if (Element.Source is UriVideoSource)
            {
                string uri = (Element.Source as UriVideoSource).Uri;

                if (!String.IsNullOrWhiteSpace(uri))
                {
                    videoView.SetVideoURI(Android.Net.Uri.Parse(uri));
                    hasSetSource = true;
                }
            }
            ···

            if (hasSetSource && Element.AutoPlay)
            {
                videoView.Start();
            }
        }
        ···
    }
}
```

Android `VideoView` no tiene la correspondiente `AutoPlay` propiedad, por lo que el `Start` se llama al método si se ha establecido un nuevo vídeo.

Hay una diferencia entre el comportamiento de iOS y Android representadores si el `Source` propiedad de `VideoPlayer` está establecido en `null`, o si el `Uri` propiedad de `UriVideoSource` está establecido en `null` o una cadena vacía. Si el Reproductor de vídeo de iOS se está reproduciendo un vídeo, y `Source` está establecido en `null` (o la cadena es `null` o en blanco), `ReplaceCurrentItemWithPlayerItem` se llama con `null` valor. El vídeo actual se reemplaza y detiene la reproducción.

Android no admite una utilidad similar. Si el `Source` propiedad está establecida en `null`, el `SetSource` método simplemente lo ignora y el vídeo actual sigue reproduciéndose.

### <a name="the-uwp-video-source"></a>El origen de vídeo de UWP

UWP `MediaElement` define un `AutoPlay` propiedad, que se controla en el representador como cualquier otra propiedad:

```csharp
namespace FormsVideoLibrary.UWP
{
    public class VideoPlayerRenderer : ViewRenderer<VideoPlayer, MediaElement>
    {
        protected override void OnElementChanged(ElementChangedEventArgs<VideoPlayer> args)
        {
            ···
            if (args.NewElement != null)
            {
                ···
                SetSource();
                SetAutoPlay();
                ···    
            }
        }

        protected override void OnElementPropertyChanged(object sender, PropertyChangedEventArgs args)
        {
            ···
            else if (args.PropertyName == VideoPlayer.SourceProperty.PropertyName)
            {
                SetSource();
            }
            else if (args.PropertyName == VideoPlayer.AutoPlayProperty.PropertyName)
            {
                SetAutoPlay();
            }
            ···
        }
        ···
    }
}
```

El `SetSource` identificadores de propiedad un `UriVideoSource` objeto estableciendo la `Source` propiedad de `MediaElement` para .NET `Uri` valor, o a `null` si el `Source` propiedad de `VideoPlayer` está establecido en `null`:

```csharp
namespace FormsVideoLibrary.UWP
{
    public class VideoPlayerRenderer : ViewRenderer<VideoPlayer, MediaElement>
    {
        ···
        async void SetSource()
        {
            bool hasSetSource = false;

            if (Element.Source is UriVideoSource)
            {
                string uri = (Element.Source as UriVideoSource).Uri;

                if (!String.IsNullOrWhiteSpace(uri))
                {
                    Control.Source = new Uri(uri);
                    hasSetSource = true;
                }
            }
            ···
            if (!hasSetSource)
            {
                Control.Source = null;
            }
        }

        void SetAutoPlay()
        {
            Control.AutoPlay = Element.AutoPlay;
        }
        ···
    }
}
```

## <a name="setting-a-url-source"></a>Establecer un origen de la dirección URL

Con la implementación de estas propiedades en los representadores de tres, es posible reproducir un vídeo desde un origen de la dirección URL. El **reproducir vídeo Web** página en el [ **VideoPlayDemos** ]( https://developer.xamarin.com/samples/xamarin-forms/customrenderers/videoplayerdemos/index.md) programa está definido por el siguiente archivo XAML:

```xaml
<ContentPage xmlns="http://xamarin.com/schemas/2014/forms"
             xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml"
             xmlns:video="clr-namespace:FormsVideoLibrary"
             x:Class="VideoPlayerDemos.PlayWebVideoPage"
             Title="Play Web Video">

    <video:VideoPlayer Source="https://archive.org/download/BigBuckBunny_328/BigBuckBunny_512kb.mp4" />

</ContentPage>
```

El `VideoSourceConverter` clase convierte la cadena en un `UriVideoSource`. Cuando se desplaza a la **reproducir el vídeo de Web** página, el vídeo empieza a cargar y empieza a reproducir cuando se ha descargado y almacenado en búfer una cantidad suficiente de datos. El vídeo es aproximadamente 10 minutos de longitud:

[![Reproducir vídeo Web](web-videos-images/playwebvideo-small.png "reproducir vídeo Web")](web-videos-images/playwebvideo-large.png#lightbox "reproducir vídeo Web")

En cada una de las plataformas, los controles de transporte fundido de salida si no se usan pero que pueden restaurarse para ver, puntee en el vídeo.

Puede impedir que el vídeo se inicie automáticamente al establecer el `AutoPlay` propiedad `false`:

```xaml
<video:VideoPlayer Source="https://archive.org/download/BigBuckBunny_328/BigBuckBunny_512kb.mp4"
                   AutoPlay="false" />
```

Deberá presionar el **reproducir** botón para iniciar el vídeo.

De forma similar, puede suprimir la presentación de los controles de transporte estableciendo el `AreTransportControlsEnabled` propiedad `false`:

```xaml
<video:VideoPlayer Source="https://archive.org/download/BigBuckBunny_328/BigBuckBunny_512kb.mp4"
                   AreTransportControlsEnabled="False" />
```

Si establece ambas propiedades en `false`, no empiece a reproducir el vídeo y no habrá ninguna manera de iniciarlo. Debe llamar a `Play` desde el archivo de código subyacente, o crear sus propios controles de transporte como se describe en el artículo [implementar controles de transporte de vídeo personalizado](custom-transport.md).

El **App.xaml** archivo incluye recursos para dos vídeos adicionales:

```xaml
<Application xmlns="http://xamarin.com/schemas/2014/forms"
             xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml"
             xmlns:video="clr-namespace:FormsVideoLibrary"
             x:Class="VideoPlayerDemos.App">
    <Application.Resources>
        <ResourceDictionary>

            <video:UriVideoSource x:Key="ElephantsDream"
                                  Uri="https://archive.org/download/ElephantsDream/ed_hd_512kb.mp4" />

            <video:UriVideoSource x:Key="BigBuckBunny"
                                  Uri="https://archive.org/download/BigBuckBunny_328/BigBuckBunny_512kb.mp4" />

            <video:UriVideoSource x:Key="Sintel"
                                  Uri="https://archive.org/download/Sintel/sintel-2048-stereo_512kb.mp4" />

        </ResourceDictionary>
    </Application.Resources>
</Application>
```

Hacer referencia a una de estas otras películas, puede reemplazar la dirección URL explícita en el **PlayWebVideo.xaml** de archivos con un `StaticResource` extensión de marcado, en cuyo caso `VideoSourceConverter` no es necesario para crear el `UriVideoSource` objeto:

```xaml
<video:VideoPlayer Source="{StaticResource ElephantsDream}" />
```

Como alternativa, puede establecer el `Source` propiedad desde un archivo de vídeo en un `ListView`, tal y como se describe en el siguiente artículo, [enlace fuentes de vídeo al Reproductor](source-bindings.md).





## <a name="related-links"></a>Vínculos relacionados

- [Demostraciones en el Reproductor de vídeo (ejemplo)](https://developer.xamarin.com/samples/xamarin-forms/customrenderers/VideoPlayerDemos/)
