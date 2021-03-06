---
title: "Configuración del acceso condicional local mediante el registro de dispositivos de Azure Active Directory | Microsoft Docs"
description: "Guía paso a paso para habilitar el acceso condicional a aplicaciones locales mediante Servicios de federación de Active Directory (AD FS) en Windows Server 2012 R2."
services: active-directory
documentationcenter: 
author: femila
manager: swadhwa
editor: 
ms.assetid: 6ae9df8b-31fe-4d72-9181-cf50cfebbf05
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 09/27/2016
ms.author: femila
translationtype: Human Translation
ms.sourcegitcommit: dcda8b30adde930ab373a087d6955b900365c4cc
ms.openlocfilehash: 02d8de2e37af9ccbf79bb77180b0eda0d187eb5c


---
# <a name="setting-up-on-premises-conditional-access-using-azure-active-directory-device-registration"></a>Configuración del acceso condicional local mediante el registro de dispositivos de Azure Active Directory
Los dispositivos de propiedad personal de los usuarios pueden marcarse como conocidos para la organización exigiendo a los usuarios la unión de sus dispositivos al área de trabajo para el servicio Registro de dispositivos de Azure Active Directory. A continuación, se ofrece una guía paso a paso para habilitar el acceso condicional a aplicaciones locales mediante Servicios de federación de Active Directory (AD FS) en Windows Server 2012 R2.

> [!NOTE]
> Se requiere licencia de Azure AD Premium o de Office 365 al usar dispositivos registrados en directivas de acceso condicional del servicio Registro de dispositivos de Azure Active Directory. Esto incluye las directivas que exigen los Servicios de federación de Active Directory (AD FS) con recursos locales.
> 
> 

Para obtener más información sobre los escenarios de acceso condicional en local, vea [Unión al área de trabajo desde cualquier dispositivo para SSO y segundo factor de autenticación sin interrupciones a través de aplicaciones de la compañía](https://technet.microsoft.com/library/dn280945.aspx).

Estas capacidades están disponibles para los clientes que compren una licencia de Azure Active Directory Premium.

## <a name="supported-devices"></a>Dispositivos compatibles
* Dispositivos Windows 7 unidos a un dominio.
* Dispositivos Windows 8.1 personales y unidos a un dominio.
* iOS 6 y versiones posteriores, para el explorador Safari
* Teléfonos Android 4.0 o versiones posteriores, Samsung GS3 o modelos posteriores, tabletas Samsung Note2 o versiones posteriores.

## <a name="scenario-prerequisites"></a>Requisitos previos de escenario
* Suscripción a Office 365 o a Azure Active Directory Premium
* Un inquilino de Azure Active Directory
* Windows Server Active Directory (Windows Server 2008 o versiones posteriores)
* Esquema actualizado en Windows Server 2012 R2
* Licencia de Azure Active Directory Premium
* Servicios de federación con Windows Server 2012 R2, configurados para SSO en Azure AD
* Windows Server 2012 R2, Web Application Proxy, Microsoft Azure Active Directory Connect (Azure AD Connect). [Descargue Azure AD Connect aquí](http://www.microsoft.com/en-us/download/details.aspx?id=47594).
* Dominio comprobado.

## <a name="known-issues-in-this-release"></a>Problemas conocidos en esta versión
* Las directivas de acceso condicional basado en dispositivos requieren reescritura de objetos de dispositivo en Active Directory de Azure Active Directory. Los objetos de dispositivo pueden tardar hasta tres horas en reescribirse en Active Directory.
* Los dispositivos iOS7 siempre solicitarán al usuario que seleccione un certificado durante la autenticación de certificados de cliente.
* Algunas versiones de iOS8 no funcionan antes de iOS 8.3.

## <a name="scenario-assumptions"></a>Escenarios hipotéticos
En este escenario se asume que tiene un entorno híbrido que consta de un inquilino de Azure AD y una versión local de Active Directory. Estos inquilinos deben conectarse mediante Azure AD Connect y con un dominio comprobado y AD FS para SSO. La lista de comprobación siguiente le ayudará a configurar su entorno para la fase descrita anteriormente.

## <a name="checklist-prerequisites-for-conditional-access-scenario"></a>Lista de comprobación: requisitos previos para un escenario de acceso condicional
Conecte su inquilino de Azure AD a versión local de Active Directory.

## <a name="configure-azure-active-directory-device-registration-service"></a>Configuración del servicio de registro de dispositivos de Azure Active Directory
Use esta guía para implementar y configurar el servicio Registro de dispositivos de Azure Active Directory para su organización.

En esta guía se asume que ya configuró Windows Server Active Directory y se suscribió a Microsoft Azure Active Directory. Vea los requisitos previos anteriores.

Para implementar el servicio Registro de dispositivos de Azure Active Directory con su inquilino de Azure Active Directory, complete por orden las tareas de la lista de comprobación siguiente. Cuando un vínculo de referencia lleve a un tema conceptual, vuelva a esta lista de comprobación después de revisar dicho tema para continuar con las tareas restantes. Algunas tareas incluirán un paso de validación del escenario que puede ayudarle a confirmar que el paso se completó correctamente.

## <a name="part-1-enable-azure-active-directory-device-registration"></a>Paso 1: habilitación del registro de dispositivos de Azure Active Directory
Siga la lista de comprobación mostrada a continuación para habilitar y configurar el servicio Registro de dispositivos de Azure Active Directory.

| Tarea | Referencia |
| --- | --- |
| Habilite el Registro de dispositivos en el inquilino de Azure Active Directory para permitir que los dispositivos se unan al área de trabajo. De forma predeterminada, Multi-Factor Authentication no está habilitada para el servicio. Aunque se recomienda usar Multi-Factor Authentication al registrar un dispositivo. Antes de habilitar Multi-Factor Authentication en ADRS, asegúrese de que AD FS está configurado para un proveedor de Multi-Factor Authentication. |[Habilitación del Registro de dispositivos de Azure Active Directory](active-directory-conditional-access-device-registration-overview.md) |
| Los dispositivos detectarán el servicio Registro de dispositivos de Azure Active Directory buscando registros DNS conocidos. Debe configurar el DNS de su compañía para que los dispositivos puedan detectar el servicio Registro de dispositivos de Azure Active Directory. |[Configuración de la detección del Registro de dispositivos de Azure Active Directory](active-directory-conditional-access-device-registration-overview.md) |

## <a name="part-2-deploy-and-configure-windows-server-2012-r2-active-directory-federation-services-and-set-up-a-federation-relationship-with-azure-ad"></a>Parte 2: implementación de Servicios de federación de Active Directory Windows Server 2012 R2 y configuración de una relación de federación con Azure AD.
| Tarea | Referencia |
| --- | --- |
| Implemente el dominio de Servicios de dominio de Active Directory con las extensiones de esquema de Windows Server 2012 R2. No es preciso actualizar los controladores de dominio a Windows Server 2012 R2. El único requisito es la actualización del esquema. |[Actualización del esquema de Servicios de dominio de Active Directory](#upgrade-your-active-directory-domain-services-schema) |
| Los dispositivos detectarán el servicio Registro de dispositivos de Azure Active Directory buscando registros DNS conocidos. Debe configurar el DNS de su compañía para que los dispositivos puedan detectar el servicio Registro de dispositivos de Azure Active Directory. |[Preparación de los dispositivos para la compatibilidad con Active Directory](#prepare-your-active-directory-to-support-devices) |

## <a name="part-3-enable-device-writeback-in-azure-ad"></a>Parte 3: habilitación de reescritura de dispositivos en Azure AD
| Tarea | Referencia |
| --- | --- |
| Complete la parte 2 de Habilitación de reescritura de dispositivos en Azure AD Connect. Tras la finalización, vuelva a esta guía. |[Habilitación de escritura diferida de dispositivos en Azure AD Connect](#upgrade-your-active-directory-domain-services-schema) |

## <a name="optional-part-4-enable-multi-factor-authentication"></a>[Opcional] Parte 4: Habilitar Multi-factor Authentication
Se recomienda encarecidamente configurar una de las distintas opciones de Multi-Factor Authentication. Si desea requerir MFA, consulte [Selección de la solución de seguridad multifactor más adecuada](../multi-factor-authentication/multi-factor-authentication-get-started.md). Incluye una descripción de cada solución y vínculos para ayudarle a configurar la solución que haya elegido.

## <a name="part-5-verification"></a>Parte 5: verificación
La implementación ha finalizado. Ya puede probar algunos escenarios. Siga los vínculos siguientes para probar el servicio y familiarizarse con las características

| Tarea | Referencia |
| --- | --- |
| Conecte varios dispositivos al área de trabajo con el Registro de dispositivos de Azure Active Directory Puede conectar dispositivos iOS, Windows y Android |[Unión de dispositivos al área de trabajo mediante el Registro de dispositivos de Azure Active Directory](#join-devices-to-your-workplace-using-azure-active-directory-device-registration) |
| Los dispositivos registrados se pueden ver y habilitar o deshabilitar desde el Portal de administrador. En esta tarea verá algunos dispositivos registrados con el Portal de administrador. |[Descripción general sobre el registro de dispositivos de Azure Active Directory](active-directory-conditional-access-device-registration-overview.md) |
| Compruebe que los objetos de dispositivo se reescriben desde Azure Active Directory a Windows Server Active Directory. |[Comprobación de que los dispositivos registrados se reescriben en Active Directory](#verify-registered-devices-are-written-back-to-active-directory) |
| Ahora que los usuarios pueden registrar sus dispositivos, puede crear en AD FS directivas de acceso a la aplicación que solo permitan dispositivos registrados. En esta tarea creará una regla de acceso a la aplicación y un mensaje de denegación de acceso personalizado |[Creación de una directiva de acceso a aplicaciones y un mensaje personalizado de acceso denegado](#create-an-application-access-policy-and-custom-access-denied-message) |

## <a name="integrate-azure-active-directory-with-on-premises-active-directory"></a>Integración de Azure Active Directory con una versión local de Active Directory
Esto le ayudará a integrar un inquilino de Azure AD con la versión local de Active Directory mediante Azure AD Connect. Aunque los pasos están disponibles en el Portal de Azure clásico, anote las instrucciones especiales que se enumeran en esta sección.

1. Inicie sesión en el Portal de Azure clásico con una cuenta que sea un administrador global en Azure AD.
2. En el panel izquierdo, seleccione **Active Directory**.
3. En la pestaña **Directorio** , seleccione su directorio.
4. Seleccione la pestaña **Integración de directorios** .
5. En la sección acerca de la **implementación y administración** , siga los pasos 1 a 3 para integrar Azure Active Directory en su directorio local.
   
   1. Adición de dominios.
   2. Instalar y ejecutar Azure AD Connect: para instalar Azure AD Connect, siga las instrucciones que se indican a continuación, [Instalación personalizada de Azure AD Connect](connect/active-directory-aadconnect-get-started-custom.md).
   3. Comprobación y administración de la sincronización de directorios. Las instrucciones de inicio de sesión único están disponibles en este paso.
   
   > [!NOTE]
   > Configurar la federación con AD FS como se describe en el documento vinculado arriba. No es preciso configurar las características de vista previa.
   > 
   > 

## <a name="upgrade-your-active-directory-domain-services-schema"></a>Actualización del esquema de Servicios de dominio de Active Directory
> [!NOTE]
> La actualización del esquema de Active Directory no se puede revertir. Se recomienda realizar la operación primero en un entorno de prueba.
> 
> 

1. Inicie sesión en el controlador de dominio con una cuenta que tenga derechos de administrador de organización y de administrador de esquema.
2. Copie el directorio **[media]\support\adprep** y los subdirectorios en uno de los controladores de dominio de Active Directory.
3. Donde [media] es la ruta de acceso a los medios de instalación de Windows Server 2012 R2.
4. En un símbolo del sistema, navegue hasta el directorio adprep y ejecute: **adprep.exe /forestprep**. Siga las instrucciones en pantalla para completar la actualización del esquema.

## <a name="prepare-your-active-directory-to-support-devices"></a>Preparación de Active Directory para que admita los dispositivos
> [!NOTE]
> Se trata de una operación que se realiza una sola vez que debe ejecutar para preparar el bosque de Active Directory para admitir los dispositivos. Para completar este procedimiento, debió iniciar sesión con permisos de administrador de organización y el bosque de Active Directory debe tener el esquema de Windows Server 2012 R2.
> 
> 

## <a name="prepare-your-active-directory-forest-to-support-devices"></a>Preparación del bosque de Active Directory para que admita los dispositivos
> [!NOTE]
> Se trata de una operación que se realiza una sola vez que debe ejecutar para preparar el bosque de Active Directory para admitir los dispositivos. Para completar este procedimiento, debió iniciar sesión con permisos de administrador de organización y el bosque de Active Directory debe tener el esquema de Windows Server 2012 R2.
> 
> 

### <a name="prepare-your-active-directory-forest"></a>Preparación del bosque de Active Directory
1. En el servidor de federación, abra una ventana Comandos de Windows PowerShell y escriba: Initialize-ADDeviceRegistration
2. Cuando le pidan el valor de **ServiceAccountName**, escriba el nombre de la cuenta de servicio que seleccionó como cuenta de servicio de AD FS. Si es una cuenta de gMSA, escríbala con el formato **dominio\nombreDeCuenta$**. En el caso de una cuenta de dominio, use el formato **dominio\nombreDeCuenta**.

### <a name="enable-device-authentication-in-ad-fs"></a>Habilitación de la autenticación del dispositivo en AD FS
1. En el servidor de federación, abra la consola de administración de AD FS y navegue hasta **AD FS**  >  **Directivas de autenticación**.
2. Seleccione **Editar autenticación principal global…** from the **Acciones** .
3. Active **Habilitar autenticación de dispositivo** y seleccione **Aceptar**.
4. De forma predeterminada, AD FS quitará periódicamente los dispositivos no usados de Active Directory. Debe deshabilitar esta tarea cuando use el Registro de dispositivos de Azure Active Directory para que los dispositivos se pueden administrar en Azure.

### <a name="disable-unused-device-cleanup"></a>Deshabilitación de la limpieza de dispositivos no usados
1. En el servidor de federación, abra una ventana Comandos de Windows PowerShell y escriba: Set-AdfsDeviceRegistration -MaximumInactiveDays 0.

### <a name="prepare-azure-ad-connect-for-device-writeback"></a>Preparación de Azure AD Connect para la reescritura de dispositivos
1. Complete la Parte 1: Preparar Azure AD Connect

## <a name="join-devices-to-your-workplace-using-azure-active-directory-device-registration"></a>Unión de dispositivos al área de trabajo mediante el Registro de dispositivos de Azure Active Directory
### <a name="join-an-ios-device-using-azure-active-directory-device-registration"></a>Unión de un dispositivo iOS al área de trabajo mediante el Registro de dispositivos de Azure Active Directory
El Registro de dispositivos de Azure Active Directory usa el proceso de inscripción de perfil móvil para dispositivos iOS. El proceso comienza con la conexión del usuario a la dirección URL de inscripción del perfil mediante el explorador web Safari. El formato de la dirección URL es el siguiente:

    https://enterpriseregistration.windows.net/enrollmentserver/otaprofile/"yourdomainname"

Donde `yourdomainname` es el nombre de dominio que se configuró con Azure Active Directory. Por ejemplo, si el nombre de dominio es contoso.com, la dirección URL sería:

    https://enterpriseregistration.windows.net/enrollmentserver/otaprofile/contoso.com

Existen varias formas de comunicar la URL a los usuarios. Una forma que se recomienda consiste en publicar esta dirección URL en un mensaje personalizado de acceso denegado a la aplicación en AD FS. Este tema se trata en la siguiente sección: [Creación de una directiva de acceso a aplicaciones y mensaje personalizado de acceso denegado](#create-an-application-access-policy-and-custom-access-denied-message).

### <a name="join-a-windows-81-device-using-azure-active-directory-device-registration"></a>Unión de un dispositivo Windows 8.1 al área de trabajo mediante el Registro de dispositivos de Azure Active Directory
1. En el dispositivo Windows 8.1, navegue hasta **Configuración de PC** > **Red** > **Área de trabajo**.
2. Escriba su nombre de usuario en formato UPN. Por ejemplo: dan@contoso.com..
3. Seleccione **Combinar**.
4. Cuando se lo pidan, inicie sesión con sus credenciales. El dispositivo ahora está unido.

### <a name="join-a-windows-7-device-using-azure-active-directory-device-registration"></a>Unión de un dispositivo Windows 7 al área de trabajo mediante el Registro de dispositivos de Azure Active Directory
Para registrar los dispositivos Windows 7 unidos a un dominio debe implementar el paquete de software de registro del dispositivo. El paquete de software se llama Unión al lugar de trabajo para Windows 7 y está disponible para su descarga en el [sitio web de Microsoft Connect](https://connect.microsoft.com/site1164). Las instrucciones para utilizar el paquete están disponibles en [Configure el registro automático de dispositivos para dispositivos Windows 7 unidos a un dominio](active-directory-conditional-access-automatic-device-registration-windows7.md).

### <a name="join-an-android-device-using-azure-active-directory-device-registration"></a>Unión de un dispositivo Android al área de trabajo mediante el Registro de dispositivos de Azure Active Directory
El [tema sobre Microsoft Authenticator para Android](active-directory-conditional-access-azure-authenticator-app.md) incluye instrucciones sobre cómo instalar la aplicación Microsoft Authenticator en el dispositivo Android y agregar una cuenta profesional. Cuando una cuenta profesional se crea correctamente en un dispositivo Android, el dispositivo queda unido al área de trabajo en la organización.

## <a name="verify-registered-devices-are-written-back-to-active-directory"></a>Comprobación de que los dispositivos registrados se reescriben en Active Directory
Puede ver y comprobar que los objetos de dispositivo se reescribieron en Active Directory con LDP.exe o con el Editor ADSI. Ambos están disponibles con las herramientas del administrador de Active Directory.

De forma predeterminada, los objetos de dispositivo que se reescriben desde Azure Active Directory se colocan en el mismo dominio que la granja de AD FS.

    CN=RegisteredDevices,defaultNamingContext

## <a name="create-an-application-access-policy-and-custom-access-denied-message"></a>Creación de una directiva de acceso a aplicaciones y un mensaje personalizado de acceso denegado
Considere el siguiente escenario: se crea una aplicación de confianza para usuario autenticado en AD FS y se configura una regla de autorización de emisión que solo permite dispositivos registrados. Ahora solo los dispositivos que están registrados pueden tener acceso a la aplicación. Para que a los usuarios les resulte más fácil obtener acceso a la aplicación, configure un mensaje personalizado de acceso denegado que incluya instrucciones sobre cómo deben unir el dispositivo. Ahora los usuarios disponen de una manera directa de registrar sus dispositivos para tener acceso a una aplicación.

Los pasos siguientes le indicarán cómo implementar este escenario:

> [!NOTE]
> En esta sección se supone que ya configuró una relación de confianza para usuario autenticado para la aplicación en AD FS.
> 
> 

1. Abra la herramienta MMC de AD FS y navegue hasta AD FS > Relaciones de confianza > Relaciones de confianza para usuario autenticado.
2. Busque la aplicación a la que se aplicará la nueva regla de acceso. Haga clic con el botón derecho en la aplicación y seleccione Editar reglas de notificación...
3. Seleccione la pestaña **Reglas de autorización de emisión** y luego **Agregar regla…**
4. En la lista desplegable de plantillas **Regla de notificaciones**, seleccione **Permitir o denegar usuarios según notificación entrante**. Seleccione **Siguiente**.
5. En el campo Nombre de regla de notificaciones:, especifique: **Permitir acceso desde dispositivos registrados**
6. En la lista desplegable Tipo de notificación entrante:, seleccione **Es usuario registrado**.
7. En el campo Valor de notificación entrante:, especifique: **true**
8. Seleccione el botón de radio **Permitir acceso a usuarios con esta notificación entrante** .
9. Seleccione **Finalizar** y luego **Aplicar**.
10. Quite todas las reglas que son más permisivas que la regla que acaba de crear. Por ejemplo, quite la regla **Permitir acceso a todos los usuarios** predeterminada.

Ahora, la aplicación está configurada para permitir el acceso solo cuando el usuario procede de un dispositivo registrado y unido al área de trabajo. Para ver directivas de acceso más avanzadas, vea [Administración de riesgos con el control de acceso multifactor](https://technet.microsoft.com/library/dn280949.aspx).

A continuación, configurará un mensaje de error personalizado para la aplicación. El mensaje de error notificará a los usuarios que han de unir el dispositivo al área de trabajo para tener acceso a la aplicación. Puede usar HTML personalizado y Windows PowerShell para crear un mensaje personalizado de acceso denegado a la aplicación.

En el servidor de federación, abra una ventana de comandos de Windows PowerShell y escriba el comando siguiente. Reemplazar partes del comando con elementos específicos de su sistema:

    Set-AdfsRelyingPartyWebContent -Name "relying party trust name" -ErrorPageAuthorizationErrorMessage
Para poder acceder a esta aplicación, debe registrar el dispositivo.

**Si usa un dispositivo iOS, seleccione este vínculo para conectarlo**:

    a href='https://enterpriseregistration.windows.net/enrollmentserver/otaprofile/yourdomain.com

Una el dispositivo iOS al área de trabajo.

**Si usa un dispositivo Windows 8.1**, puede conectarlo desde **Configuración de PC**> **Red** > **Área de trabajo**.

Donde "**nombre de confianza de usuario de confianza**" es el nombre del objeto Confianza de usuario de confianza en AD FS.
Donde **yourdomain.com** es el nombre de dominio que configuró en Azure Active Directory. Por ejemplo, contoso.com.
Asegúrese de quitar los saltos de línea (si existen) del contenido HTML que se pasa al cmdlet **Set-AdfsRelyingPartyWebContent** .

Ahora, cuando los usuarios accedan a la aplicación desde un dispositivo que no esté registrado con en el servicio Registro de dispositivos de Azure Active Directory, recibirán una página similar a la captura de pantalla siguiente.

![Captura de pantalla de un error cuando los usuarios no han registrado su dispositivo en Azure AD](./media/active-directory-conditional-access/error-azureDRS-device-not-registered.gif)

## <a name="related-articles"></a>Artículos relacionados
* [Índice de artículos sobre la administración de aplicaciones en Azure Active Directory](active-directory-apps-index.md)




<!--HONumber=Jan17_HO1-->


