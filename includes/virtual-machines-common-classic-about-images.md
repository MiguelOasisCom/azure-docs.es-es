

Para proporcionar una nueva máquina virtual con un sistema operativo en Azure se usan imágenes. Una imagen también pueden tener uno o más discos de datos. Las imágenes están disponibles desde diferentes fuentes:

* Azure ofrece imágenes en [Marketplace](https://azure.microsoft.com/gallery/virtual-machines/). Hay versiones recientes de Windows Server y distribuciones del sistema operativo Linux. Algunas imágenes también contienen aplicaciones, como SQL Server. Los suscriptores de Pago por uso de MSDN y Ventajas de MSDN tienen acceso a imágenes adicionales.
* La comunidad de código abierto ofrece imágenes a través de [VM Depot](http://vmdepot.msopentech.com/List/Index).
* También puede almacenar y usar sus propias imágenes en Azure, bien a través de una captura de una máquina virtual de Azure ya existente para usarla como imagen o a través de la carga de una imagen.

## <a name="about-vm-images-and-os-images"></a>Acerca de las imágenes de máquina virtual y las de sistema operativo
En Azure se pueden usar dos tipos de imágenes: *imagen de máquina virtual* e *imagen de SO*. Una imagen de máquina virtual incluye un sistema operativo y todos los discos conectados a una máquina virtual cuando se crea la imagen. Una imagen de máquina virtual es el tipo de imagen más reciente. Antes de la introducción de las imágenes de máquina virtual, una imagen en Azure podía tener sólo un sistema operativo generalizado sin discos adicionales. Una imagen de máquina virtual que contenga sólo un sistema operativo generalizado es básicamente igual que el tipo de imagen original, la imagen SO.

Puede crear sus propias imágenes a partir de una máquina virtual en Azure o una máquina virtual que se ejecute en cualquier sitio desde donde pueda copiar y cargar. Si desea usar una imagen para crear más de una máquina virtual, debe prepararla para usarla como una imagen generalizándola. Para crear una imagen de Windows Server, ejecute el comando Sysprep en el servidor para generalizar antes de cargar el archivo .vhd. Para más información acerca de Sysprep, consulte [How to Use Sysprep: An Introduction](http://go.microsoft.com/fwlink/p/?LinkId=392030) (Introducción al uso de Sysprep) y [Sysprep Support for Server Roles](https://msdn.microsoft.com/windows/hardware/commercialize/manufacture/desktop/sysprep-support-for-server-roles) (Compatibilidad de Sysprep con roles de servidor). Cree una copia de seguridad de la VM antes de ejecutar Sysprep. La creación de una imagen de Linux varía según la distribución. Normalmente, debe ejecutar un conjunto de comandos específico para la distribución y ejecutar el Agente de Linux de Azure.



<!--HONumber=Nov16_HO3-->


