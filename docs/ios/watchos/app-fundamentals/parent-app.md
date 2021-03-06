---
title: Trabajar con la aplicación principal en Xamarin de watchOS
description: Este documento describe cómo trabajar con una aplicación de watchOS primaria en Xamarin. Describe las extensiones de aplicación de WatchKit, aplicaciones de iOS, almacenamiento compartido y mucho más.
ms.prod: xamarin
ms.assetid: 9AD29833-E9CC-41A3-95D2-8A655FF0B511
ms.technology: xamarin-ios
author: lobrien
ms.author: laobri
ms.date: 03/17/2017
ms.openlocfilehash: 745c10b381ef2bd578278cb8d141a944ef1087e0
ms.sourcegitcommit: e268fd44422d0bbc7c944a678e2cc633a0493122
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 10/25/2018
ms.locfileid: "50121989"
---
# <a name="working-with-the-watchos-parent-application-in-xamarin"></a>Trabajar con la aplicación principal en Xamarin de watchOS

> [!IMPORTANT]
> Acceso a la aplicación principal con los ejemplos siguientes solo funciona en aplicaciones de inspección para watchOS 1.


Hay diferentes formas de comunicación entre la aplicación del reloj y la aplicación de iOS se incluye con:

- Ver extensiones puede [llamar a un método](#code) frente a la aplicación principal que se ejecuta en segundo plano en el iPhone.

- Ver extensiones puede [comparten una ubicación de almacenamiento](#storage) con la aplicación de iPhone primario.

- Uso de entrega para pasar datos de una vista o una notificación a la aplicación del reloj, enviar al usuario a un controlador de interfaz específica en la aplicación.

La aplicación primaria también en ocasiones se conoce como la aplicación de contenedor.


<a name="code" />

## <a name="run-code"></a>Ejecutar código

Comunicación entre una extensión de inspección y la aplicación de iPhone primario se muestra en el [GpsWatch ejemplo](https://developer.xamarin.com/samples/GpsWatch).
La extensión de inspección puede solicitar la aplicación para iOS principal para realizar algún procesamiento en su nombre utilizando el `OpenParentApplication` método.

Esto es especialmente útil para tareas en ejecución largas (incluidas las solicitudes de red): solo el elemento principal aplicación de iOS pueden aprovechar las ventajas del procesamiento en segundo plano para completar estas tareas y guardar los datos recuperados en una ubicación accesible para la extensión de inspección.



### <a name="watch-kit-app-extension"></a>Extensión de la aplicación de inspección Kit

Llame a la `WKInterfaceController.OpenParentApplication` en su extensión de inspección de la aplicación. Devuelve un `bool` que indica si la solicitud del método se ha enviado correctamente o no. Compruebe el `error` parámetro en la respuesta `Action` determinar si existe, se ha producido durante el método que se ejecuta en la aplicación de iPhone.

```csharp
WKInterfaceController.OpenParentApplication (new NSDictionary (), (replyInfo, error) => {
    if(error != null) {
        Console.WriteLine (error);
        return;
    }
    Console.WriteLine ("parent app responded");
    // do something with replyInfo[] dictionary
});
```


### <a name="ios-app"></a>Aplicación de iOS

Todas las llamadas desde una extensión de aplicación de inspección se enrutan a través de la aplicación de iPhone `HandleWatchKitExtensionRequest` método.
Si está realizando solicitudes diferentes en la aplicación del reloj, a continuación, este método tendrá que consultar el `userInfo` diccionario para determinar cómo procesar la solicitud.


```csharp
[Register ("AppDelegate")]
public partial class AppDelegate : UIApplicationDelegate
{
    // ... other AppDelegate methods
    public override void HandleWatchKitExtensionRequest
        (UIApplication application, NSDictionary userInfo, Action<NSDictionary> reply)
    {
        var status = 2;
        // do something in the background, and respond
        reply (new NSDictionary (
            "count", NSNumber.FromInt32 ((int)status),
            "value2", new NSString("some-info")
            ));
    }
}
```


<a name="storage" />

## <a name="shared-storage"></a>Almacenamiento compartido

Si configura un [grupo de aplicaciones](~/ios/watchos/app-fundamentals/app-groups.md) , a continuación, las extensiones de iOS 8 (incluidas las extensiones de inspección) pueden compartir datos con la aplicación primaria.

<a name="nsuserdefaults" />

### <a name="nsuserdefaults"></a>Valores NSUserDefaults

Se puede escribir el código siguiente en la extensión de la aplicación de inspección y la aplicación de iPhone primario para que puede hacer referencia a un conjunto común de `NSUserDefaults`:

```csharp
NSUserDefaults shared = new NSUserDefaults(
        "group.com.your-company.watchstuff",
        NSUserDefaultsType.SuiteName);

// set values
shared.SetInt (2, "count");
shared.Synchronize ();

// get values
shared.Synchronize ();
var count = shared.IntForKey ("count");
```

<a name="files" />

### <a name="files"></a>Archivos

La extensión de inspección y aplicación de iOS también puede compartir archivos con una ruta de acceso de archivo comunes.

```csharp
var FileManager = new NSFileManager ();
var appGroupContainer =
            FileManager.GetContainerUrl ("group.com.your-company.watchstuff");
var appGroupContainerPath = appGroupContainer.Path;
Console.WriteLine ("agcpath: " + appGroupContainerPath);
// use the path to create and update files
```

Nota: si la ruta de acceso es `null` , a continuación, compruebe el [configuración del grupo de aplicación](~/ios/watchos/app-fundamentals/app-groups.md) para asegurarse de los perfiles de aprovisionamiento se han configurado correctamente y han sido descargado o instalado en el equipo de desarrollo.

Para obtener más información, consulte el [las capacidades de grupos de aplicación](~/ios/deploy-test/provisioning/capabilities/app-groups-capabilities.md) documentación.

## <a name="wormholesharp"></a>WormHoleSharp

Un mecanismo de código abierto popular para watchOS 1 (según [MMWormHole](https://github.com/mutualmobile/MMWormhole)) para pasar datos o los comandos entre la aplicación primaria y la aplicación del reloj.

Puede configurar el túnel mediante un grupo de aplicaciones similar al siguiente en su aplicación iOS y ver la extensión:

```csharp
// AppDelegate (iOS) or InterfaceController (watch extension)
Wormhole wormHole;
// ...
wormHole = new Wormhole ("group.com.your-company.watchstuff", "messageDir");
```

Descargue el C# versión [WormHoleSharp](https://github.com/Clancey/WormHoleSharp).



## <a name="related-links"></a>Vínculos relacionados

- [GpsWatch (ejemplo)](https://developer.xamarin.com/samples/monotouch/WatchKit/WatchKitCatalog/)
- [WormHoleSharp (ejemplo)](https://github.com/Clancey/WormHoleSharp)
- [Referencia de WKInterfaceController de Apple](https://developer.apple.com/library/prerelease/ios/documentation/WatchKit/Reference/WKInterfaceController_class/index.html#//apple_ref/occ/clm/WKInterfaceController/openParentApplication:reply:)
- [Apple compartir datos con la aplicación contenedora](https://developer.apple.com/library/ios/documentation/General/Conceptual/ExtensibilityPG/ExtensionScenarios.html)
