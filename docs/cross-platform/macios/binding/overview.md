---
title: Información general de los enlaces de Objective-c.
description: Este documento proporciona información general de maneras diferentes de crear enlaces de C# para código de Objective-C, incluidos los enlaces de la línea de comandos, proyectos de enlace y Sharpie objetivo. También se explica cómo funciona el enlace.
ms.prod: xamarin
ms.assetid: 9EE288C5-8952-C5A9-E542-0BD847300EC6
author: asb3993
ms.author: amburns
ms.openlocfilehash: 97d0c5b9f61d4dafe144d2b2f22df6d465cbbccb
ms.sourcegitcommit: ec50c626613f2f9af51a9f4a52781129bcbf3fcb
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 07/05/2018
ms.locfileid: "37855278"
---
# <a name="overview-of-objective-c-bindings"></a>Información general de los enlaces de Objective-c.

_Detalles de cómo funciona el proceso de enlace_

Enlace de una biblioteca de Objective-C para su uso con Xamarin toma de tres pasos:

1. Escribir un C# "Definición de la API" para describir cómo se expone la API nativa de. NET, y cómo se asigna a la subyacente Objective-C. Esto se hace utilizando C# estándar construcciones como `interface` y enlace diversas **atributos** (verlo [ejemplo sencillo](~/cross-platform/macios/binding/objective-c-libraries.md#Binding_an_API)).

2. Una vez se ha escrito la "definición de API" en C#, compilarlo para generar un ensamblado de "enlace". Esto puede hacerse en el [ **línea de comandos** ](#commandline) o mediante un [ **proyecto enlace** ](#bindingproject) en Visual Studio para Mac o Visual Studio.

3. Dicho ensamblado "enlace", a continuación, se agrega al proyecto de aplicación de Xamarin, por lo que puede tener acceso a la funcionalidad nativa mediante la API que se ha definido.
  El proyecto de enlace es completamente independiente de los proyectos de aplicación.

**Nota:** paso 1 se puede automatizar con la Ayuda de [ **objetivo Sharpie**](#objectivesharpie). Examina la API de Objective-C y genera una propuesta de C# "Definición de API". Puede personalizar los archivos creados por objetivo Sharpie y usarlos en un proyecto de enlace (o en la línea de comandos) para crear el ensamblado de enlace. Sharpie objetivo no crear enlaces por sí mismo, es simplemente una parte opcional del proceso mayor.

También puede leer detalles más técnicos de [cómo funciona](#howitworks), que le ayudarán a escribir sus enlaces.

<a name="Command_Line_Bindings" /><a name="commandline" />

## <a name="command-line-bindings"></a>Enlaces de la línea de comandos

Puede usar el `btouch-native` para Xamarin.iOS (o `bmac-native` si usa Xamarin.Mac) para crear enlaces directamente. Funciona pasando las definiciones de API de C# que ha creado manualmente (o mediante Sharpie objetivo) a la herramienta de línea de comandos (`btouch-native` para iOS o `bmac-native` para Mac).


La sintaxis general para llamar a estas herramientas es:

```csharp
# Use this for Xamarin.iOS:
bash$ /Developer/MonoTouch/usr/bin/btouch-native -e cocos2d.cs -s:enums.cs -x:extensions.cs
```

```csharp
# Use this for Xamarin.Mac:
bash$ bmac-native -e cocos2d.cs -s:enums.cs -x:extensions.cs
```

El comando anterior generará el archivo `cocos2d.dll` en el directorio actual, y contendrá la biblioteca totalmente dependiente que puede usar en el proyecto. Esta es la herramienta que usa Visual Studio para Mac para crear los enlaces si usa un proyecto de enlace (se describe [debajo](#bindingproject)).


<a name="bindingproject" />

## <a name="binding-project"></a>Proyecto de enlace

Un proyecto de enlace se puede crear en Visual Studio para Mac o Visual Studio (Visual Studio solo admite los enlaces de iOS) y resulta más fácil editar y compilar definiciones de API para el enlace (frente al uso de la línea de comandos).

Siga este [Guía de introducción](~/cross-platform/macios/binding/objective-c-libraries.md#Getting_Started) para ver cómo crear y usar un proyecto de enlace para generar un enlace.

<a name="objectivesharpie" />

## <a name="objective-sharpie"></a>Sharpie objetivo

Sharpie objetivo es la herramienta independiente, otra línea de comandos que ayuda a las fases iniciales de la creación de un enlace. No crea un enlace por sí mismo, en su lugar automatiza el paso inicial de generar una definición de API de la biblioteca nativa de destino.

Leer el [docs objetivo Sharpie](~/cross-platform/macios/binding/objective-sharpie/index.md) para obtener información sobre cómo analizar las bibliotecas nativas, marcos nativos y CocoaPods en definiciones de API que se pueden integrar en los enlaces.

<a name="howitworks" />

## <a name="how-binding-works"></a>Cómo funciona el enlace

Es posible utilizar el [[registrar]](https://developer.xamarin.com/api/type/Foundation.RegisterAttribute/) atributo, [[Exportar]](https://developer.xamarin.com/api/type/Foundation.ExportAttribute/) atributo, y [manual invocación del selector de Objective-C](~/ios/internals/objective-c-selectors.md) conjuntamente para enlazar manualmente nuevos (anteriormente tipos de Objective-C sin enlazar).

En primer lugar, se encuentra un tipo que desea enlazar. Para fines de análisis (y simplicidad), se deberá enlazar la [NSEnumerator](http://developer.apple.com/iphone/library/documentation/Cocoa/Reference/Foundation/Classes/NSEnumerator_Class/Reference/Reference.html) tipo (que ya se ha enlazado en [Foundation.NSEnumerator](https://developer.xamarin.com/api/type/Foundation.NSEnumerator/); simplemente por ejemplo, la siguiente implementación es con fines).

En segundo lugar, se debe crear el tipo de C#. Probablemente desearemos colocar esto en un espacio de nombres; Puesto que Objective-C no admite espacios de nombres, se deberá usar el `[Register]` atributo para cambiar el nombre del tipo que se registrarán Xamarin.iOS en el tiempo de ejecución de C de objetivo. El tipo de C# también debe heredar de [Foundation.NSObject](https://developer.xamarin.com/api/type/Foundation.NSObject/):

```csharp
namespace Example.Binding {
    [Register("NSEnumerator")]
    class NSEnumerator : NSObject
    {
        // see steps 3-5
    }
}
```

En tercer lugar, revise la documentación de Objective-C y crear [ObjCRuntime.Selector](https://developer.xamarin.com/api/type/ObjCRuntime.Selector/) instancias para cada selector que desee usar. Colocar esas opciones en el cuerpo de la clase:

```csharp
static Selector selInit       = new Selector("init");
static Selector selAllObjects = new Selector("allObjects");
static Selector selNextObject = new Selector("nextObject");
```

En cuarto lugar, el tipo se deberá proporcionar constructores. Le *debe* encadenar la invocación del constructor al constructor de clase base. El `[Export]` atributos permiten que el código de Objective-C para llamar a los constructores con el nombre de selector especificada:

```csharp
[Export("init")]
public NSEnumerator()
    : base(NSObjectFlag.Empty)
{
    Handle = Messaging.IntPtr_objc_msgSend(this.Handle, selInit.Handle);
}
```

```csharp
// This constructor must be present so that Xamarin.iOS
// can create instances of your type from Objective-C code.
public NSEnumerator(IntPtr handle)
    : base(handle)
{
}
```

En quinto lugar, proporcionan métodos para cada uno de los selectores se declaran en el paso 3. Estos usarán `objc_msgSend()` para invocar el selector en el objeto nativo. Tenga en cuenta el uso de [Runtime.GetNSObject()](https://developer.xamarin.com/api/member/ObjCRuntime.Runtime.GetNSObject/(System.IntPtr)) para convertir un `IntPtr` en tipado adecuado `NSObject` (sub) tipo. Si desea que el método se pueda llamar desde código de Objective-C, el miembro *debe* ser **virtual**.

```csharp
[Export("nextObject")]
public virtual NSObject NextObject()
{
    return Runtime.GetNSObject(
        Messaging.IntPtr_objc_msgSend(this.Handle, selNextObject.Handle));
}
```

```csharp
// Note that for properties, [Export] goes on the get/set method:
public virtual NSArray AllObjects {
    [Export("allObjects")]
    get {
        return (NSArray) Runtime.GetNSObject(
            Messaging.IntPtr_objc_msgSend(this.Handle, selAllObjects.Handle));
    }
}
```

En resumen:

```csharp
using System;
using Foundation;
using ObjCRuntime;

namespace Example.Binding {
    [Register("NSEnumerator")]
    class NSEnumerator : NSObject
    {
        static Selector selInit       = new Selector("init");
        static Selector selAllObjects = new Selector("allObjects");
        static Selector selNextObject = new Selector("nextObject");

        [Export("init")]
        public NSEnumerator()
            : base(NSObjectFlag.Empty)
        {
            Handle = Messaging.IntPtr_objc_msgSend(this.Handle,
                selInit.Handle);
        }

        public NSEnumerator(IntPtr handle)
            : base(handle)
        {
        }

        [Export("nextObject")]
        public virtual NSObject NextObject()
        {
            return Runtime.GetNSObject(
                Messaging.IntPtr_objc_msgSend(this.Handle,
                    selNextObject.Handle));
        }

        // Note that for properties, [Export] goes on the get/set method:
        public virtual NSArray AllObjects {
            [Export("allObjects")]
            get {
                return (NSArray) Runtime.GetNSObject(
                    Messaging.IntPtr_objc_msgSend(this.Handle,
                        selAllObjects.Handle));
            }
        }
    }
}
```

## <a name="related-links"></a>Vínculos relacionados

- [Curso de Xamarin University: Creación de una biblioteca de enlaces de Objective-c.](https://university.xamarin.com/classes/track/all#building-an-objective-c-bindings-library)
- [Curso de Xamarin University: Compilar una biblioteca de enlaces de Objective-C con Sharpie objetivo](https://university.xamarin.com/classes/track/all#build-an-objective-c-bindings-library-with-objective-sharpie)