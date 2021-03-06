---
title: "Preguntas más frecuentes sobre Event Hubs | Microsoft Docs"
description: "Preguntas más frecuentes sobre los Centros de eventos"
services: event-hubs
documentationcenter: na
author: sethmanheim
manager: timlt
editor: 
ms.assetid: bfa10984-eb22-4671-861a-f377a90d9372
ms.service: event-hubs
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 12/07/2016
ms.author: sethm
translationtype: Human Translation
ms.sourcegitcommit: a584086e459c5446a814bbca3e50ac343fa9201e
ms.openlocfilehash: f7b3974bf789df8c87254cc4186d8c7c85282aaa


---
# <a name="event-hubs-faq"></a>Preguntas más frecuentes sobre los Centros de eventos
Centros de eventos ofrece consumo, persistencia y procesamiento a gran escala de eventos de datos de orígenes de datos de alto rendimiento o millones de dispositivos. Cuando se empareja con temas y colas de Bus de servicio, Centros de eventos permite implementaciones de comando y control persistentes para escenarios de [Internet de las cosas](https://azure.microsoft.com/services/iot-hub/) .

En este artículo se proporciona información sobre los precios, así como respuestas a algunas preguntas frecuentes acerca de Event Hubs:

## <a name="pricing-information"></a>Información de precios
Para obtener una completa información sobre los precios de los Centros de eventos, consulte los [detalles de precios de los Centros de eventos](https://azure.microsoft.com/pricing/details/event-hubs/).

## <a name="how-are-event-hubs-ingress-events-calculated"></a>¿Cómo se calculan los eventos de entrada de los Centros de eventos?
Cada evento enviado a un Centro de eventos cuenta como un mensaje facturable. Un *evento de entrada* se define como una unidad de datos que es menor o igual que 64 KB. Cualquier evento que tenga un tamaño menor o igual que 64 KB se considera un evento facturable. Si el evento es mayor de 64 KB, el número de eventos facturables se calcula según el tamaño del evento, en múltiplos de 64 KB. Por ejemplo, un evento de 8 KB enviado al Centro de eventos se factura como un evento, pero un mensaje de 96 KB enviado al Centro de eventos se factura como dos eventos.

Los eventos consumidos de un Centro de eventos, así como las operaciones de administración y las llamadas de control como los puntos de comprobación, no se cuentan como eventos de entrada facturables, pero se acumulan hasta la asignación de unidad de procesamiento.

## <a name="what-are-event-hubs-throughput-units"></a>¿Qué son las unidades de procesamiento de los Centros de eventos?
Las unidades de procesamiento de Centros de eventos se seleccionan explícitamente, mediante el Portal de Azure clásico o las plantillas de Resource Manager para Centros de eventos. Las unidades de procesamiento se aplican a todos los Centros de eventos de un espacio de nombres de Centros de eventos, y cada unidad de procesamiento da derecho al espacio de nombres a las siguientes funcionalidades:

* Hasta 1 MB por segundo de eventos de entrada (eventos enviados a un Centro de eventos), pero no más de 1000 eventos de entrada, operaciones de administración o llamadas a la API de control por segundo.
* Hasta 2 MB por segundo de eventos de salida (eventos consumidos de un Centro de eventos).
* Hasta 84 GB de almacenamiento de eventos (suficiente para el período de retención predeterminado de 24 horas).

Las unidades de procesamiento de los Centros de eventos se facturan por horas, según el número máximo de unidades seleccionadas durante la hora en cuestión.

## <a name="how-are-event-hubs-throughput-unit-limits-enforced"></a>¿Cómo se aplican los límites de unidades de rendimiento de los Centros de eventos?
Si el procesamiento de entrada total o la tasa de eventos de entrada total en todos los Centros de eventos en un espacio de nombres superan las unidades de procesamiento totales permitidas, los remitentes se limitarán y recibirán errores que indican que se superó la cuota de entrada.

Si el procesamiento de salida total o la tasa de eventos de salida total en todos los Centros de eventos en un espacio de nombres superan las unidades de procesamiento totales permitidas, los receptores se limitarán y recibirán errores que indican que se superó la cuota de salida. Las cuotas de entrada y de salida se aplican por separado, por lo que ningún remitente puede provocar que se ralentice el consumo de eventos, ni tampoco puede un receptor impedir que los eventos se envíen a un Centro de eventos.

Tenga en cuenta que la selección de la unidad de procesamiento es independiente del número de particiones de los Centros de eventos. Aunque cada partición ofrece un procesamiento máximo de 1 MB por segundo de entrada (con un máximo de 1000 eventos por segundo) y 2 MB por segundo de salida, no hay ningún cargo fijo por las propias particiones. El cargo es para las unidades de procesamiento totales en todos los Centros de eventos en un espacio de nombres de Centros de eventos. Con este patrón, puede crear particiones suficientes para admitir la carga máxima anticipada de sus sistemas, sin incurrir en cargos por unidades de procesamiento hasta que la carga de eventos en el sistema requiera realmente mayores cifras de procesamiento, sin tener que cambiar la estructura y la arquitectura de los sistemas a medida que la carga del sistema aumente.

## <a name="is-there-a-limit-on-the-number-of-throughput-units-that-can-be-selected"></a>¿Hay un límite en el número de unidades de procesamiento que se pueden seleccionar?
Hay una cuota predeterminada de 20 unidades de procesamiento por espacio de nombres. Puede solicitar una cuota de unidades de procesamiento mayor, si rellena una incidencia de soporte técnico. Más allá del límite de 20 unidades de procesamiento, hay disponibles paquetes de 20 y 100 unidades de procesamiento. Tenga en cuenta que al usar más de 20 unidades de procesamiento, se quita la capacidad de cambiar el número de unidades de procesamiento sin rellenar una incidencia de soporte técnico.

## <a name="is-there-a-charge-for-retaining-event-hubs-events-for-more-than-24-hours"></a>¿Hay un cargo por retener eventos de los Centros de eventos durante más de 24 horas?
El nivel Standard de los Centros de eventos permiten períodos de retención de mensajes superiores a 24 horas, hasta un máximo de 30 días. Si el tamaño del número total de eventos almacenados supera la asignación de almacenamiento para el número de unidades de procesamiento seleccionadas (84 GB por unidad de procesamiento), el tamaño que supere la asignación se cargará con la tarifa publicada de almacenamiento de blobs de Azure. La asignación de almacenamiento en cada unidad de procesamiento cubre todos los costos de almacenamiento de los períodos de retención de 24 horas (valor predeterminado), incluso aunque la unidad de procesamiento se consuma hasta la asignación de entrada máxima.

## <a name="what-is-the-maximum-retention-period"></a>¿Cuál es el período de retención máximo?
El nivel Standard de los Centros de eventos admite actualmente un período de retención máximo de 7 días. Tenga en cuenta que los Centros de eventos no están concebidos como un almacén de datos permanentes. Los períodos de retención superiores a 24 horas están pensados para escenarios en los que es útil volver a reproducir un flujo de eventos en los mismos sistemas; por ejemplo, para entrenar o comprobar un nuevo modelo de aprendizaje automático con datos existentes.

## <a name="how-is-the-event-hubs-storage-size-calculated-and-charged"></a>¿Cómo se calcula y se cobra el tamaño de almacenamiento de los Centros de eventos?
El tamaño total de todos los eventos almacenados, incluida la sobrecarga interna de encabezados de eventos o las estructuras de almacenamiento en disco de todos los Centros de eventos, se mide a lo largo del día. Al final del día, se calcula el tamaño máximo de almacenamiento. La asignación de almacenamiento diario se calcula basándose en el número mínimo de unidades de procesamiento seleccionadas durante el día (cada unidad de procesamiento ofrece una asignación de 84 GB). Si el tamaño total supera la asignación de almacenamiento diaria calculada, el exceso de almacenamiento se factura con las tarifas de almacenamiento de blobs de Azure (con la tarifa de **almacenamiento con redundancia local** ).

## <a name="can-i-use-a-single-amqp-connection-to-send-and-receive-from-event-hubs-and-service-bus-queuestopics"></a>¿Puedo usar una sola conexión AMQP para enviar y recibir desde Centros de eventos y los temas y colas de Bus de servicio?
Sí, siempre y cuando todos los Centros de eventos, colas y temas se encuentren en el mismo espacio de nombres. Como tal, puede implementar una conectividad desacoplada e indirecta a muchos dispositivos, con latencias de fracciones de segundos, de manera rentable y altamente escalable.

## <a name="do-brokered-connection-charges-apply-to-event-hubs"></a>¿Los cargos por conexión desacoplada se aplican a los Centros de eventos?
Para los remitentes, los cargos de conexión se aplican solo cuando se usa el protocolo AMQP. No hay ningún cargo de conexión por el envío de eventos mediante HTTP, independientemente del número de sistemas o dispositivos emisores. Si tiene previsto usar AMQP (por ejemplo, para conseguir un flujo de eventos más eficiente o para habilitar la comunicación bidireccional en escenarios de comando y control de IoT), consulte la página de [información de precios del Bus de servicio](https://azure.microsoft.com/pricing/details/service-bus/) para obtener información sobre lo que constituye una conexión desacoplada y cómo se mide.

## <a name="what-is-the-difference-between-event-hubs-basic-and-standard-tiers"></a>¿Cuál es la diferencia entre los niveles Basic y Standard de los Centros de eventos?
El nivel Estándar de Centros de eventos ofrece más características que el nivel Básico, y que algunos sistemas de la competencia. Estas características incluyen períodos de retención superiores a 24 horas y la capacidad para usar una conexión AMQP única para enviar comandos a un gran número de dispositivos con latencias de fracciones de segundos, así como para enviar telemetría desde estos dispositivos a Centros de eventos. Para obtener la lista de características, consulte los [detalles de precios de los Centros de eventos](https://azure.microsoft.com/pricing/details/event-hubs/).

## <a name="geographic-availability"></a>Disponibilidad geográfica

Azure Event Hubs está disponible en todas las regiones de Azure admitidas. Para obtener una lista, visite la página [Regiones de Azure][].  

## <a name="support-and-sla"></a>SLA y soporte técnico
El soporte técnico para los Centros de eventos está disponible a través de los [foros de la comunidad](https://social.msdn.microsoft.com/forums/azure/home). Se ofrece de forma gratuita soporte técnico para la administración de suscripciones y la facturación.

Para más información sobre nuestro SLA, consulte la página [Acuerdos de nivel de servicio](https://azure.microsoft.com/support/legal/sla/) .

## <a name="next-steps"></a>Pasos siguientes
Para obtener más información sobre los Centros de eventos, consulte los siguientes artículos:

* [Información general sobre Event Hubs][Event Hubs overview].
* Una [aplicación de ejemplo completa que usa Event Hubs][sample application that uses Event Hubs].

[Event Hubs overview]: event-hubs-overview.md
[sample application that uses Event Hubs]: https://code.msdn.microsoft.com/Service-Bus-Event-Hub-286fd097
[Regiones de Azure]: https://azure.microsoft.com/regions/



<!--HONumber=Dec16_HO3-->


