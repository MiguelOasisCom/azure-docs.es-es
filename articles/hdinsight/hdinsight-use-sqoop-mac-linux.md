---
title: Uso de Sqoop de Hadoop en HDInsight basado en Linux| Microsoft Docs
description: "Aprenda a ejecutar Sqoop para importar y exportar entre un clúster de Hadoop basado en Linux en HDInsight y una base de datos SQL de Azure."
editor: cgronlun
manager: jhubbard
services: hdinsight
documentationcenter: 
author: Blackmist
tags: azure-portal
ms.assetid: 303649a5-4be5-4933-bf1d-4b232083c354
ms.service: hdinsight
ms.workload: big-data
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 10/03/2016
ms.author: larryfr
translationtype: Human Translation
ms.sourcegitcommit: 0587dfcd6079fc8df91bad5a5f902391d3657a6b
ms.openlocfilehash: f39e0aec85856adba8dea99159f94fc55c822224


---
# <a name="use-sqoop-with-hadoop-in-hdinsight-ssh"></a>Uso de Sqoop con Hadoop en HDInsight (SSH)
[!INCLUDE [sqoop-selector](../../includes/hdinsight-selector-use-sqoop.md)]

Aprenda a utilizar Sqoop para importar y exportar entre un clúster de HDInsight basado en Linux y la base de datos de SQL Server o la base de datos SQL de Azure.

> [!NOTE]
> En los pasos de este artículo se usa SSH para conectarse a un clúster de HDInsight basado en Linux. Los clientes de Windows también pueden usar Azure PowerShell y el SDK de .NET de HDInsight para trabajar con Sqoop en clústeres basados en Linux. Utilice el selector de pestañas para abrir esos artículos.
>
>

## <a name="prerequisites"></a>Requisitos previos
Antes de empezar este tutorial, debe contar con lo siguiente:

* **Un clúster de Hadoop en HDInsight** y una instancia de **Azure SQL Database**: los pasos descritos en este documento se basan en el clúster y la base de datos creados mediante el documento [Creación del clúster y la base de datos SQL](hdinsight-use-sqoop.md#create-cluster-and-sql-database). Si ya tiene un clúster de HDInsight y una base de datos SQL, puede sustituir los valores utilizados en este documento por aquellos.
* **Estación de trabajo**: un equipo con un cliente SSH.

## <a name="install-freetds"></a>Instalación de FreeTDS
1. Use SSH para conectarse al clúster de HDInsight basado en Linux. La dirección que se usará al conectarse es `CLUSTERNAME-ssh.azurehdinsight.net`; y el puerto, `22`.

    Para obtener más información sobre el uso de SSH para conectarse a HDInsight, vea los siguientes documentos:

   * **Clientes de Linux, Unix y OS X**: vea [Conectarse a un clúster de HDInsight basado en Linux desde Linux, OS X o Unix](hdinsight-hadoop-linux-use-ssh-unix.md)
   * **Clientes de Windows**: vea [Conectarse a un clúster de HDInsight basado en Linux desde Windows](hdinsight-hadoop-linux-use-ssh-windows.md)
2. Use el siguiente comando para instalar FreeTDS:

        sudo apt-get --assume-yes install freetds-dev freetds-bin

    FreeTDS se utilizará en varios pasos para la conexión con la base de datos SQL.

## <a name="create-the-table-in-sql-database"></a>Creación de la tabla en la base de datos SQL
> [!IMPORTANT]
> Si está usando un clúster de HDInsight y una base de datos SQL creados mediante los pasos descritos en [Creación del clúster y la base de datos SQL](hdinsight-use-sqoop.md), omita los indicados en esta sección, ya que la base de datos y la tabla se generaron al seguir el procedimiento de dicho documento.
>
>

1. Desde la conexión SSH con HDInsight, use el comando siguiente para conectarse al servidor de base de datos SQL y crear la tabla que se usará en el resto de los presentes pasos:

        TDSVER=8.0 tsql -H <serverName>.database.windows.net -U <adminLogin> -P <adminPassword> -p 1433 -D sqooptest

    Recibirá un resultado similar al siguiente.

        locale is "en_US.UTF-8"
        locale charset is "UTF-8"
        using default charset "UTF-8"
        Default database being set to sqooptest
        1>
2. En el símbolo del sistema `1>` , introduzca las líneas siguientes:

        CREATE TABLE [dbo].[mobiledata](
        [clientid] [nvarchar](50),
        [querytime] [nvarchar](50),
        [market] [nvarchar](50),
        [deviceplatform] [nvarchar](50),
        [devicemake] [nvarchar](50),
        [devicemodel] [nvarchar](50),
        [state] [nvarchar](50),
        [country] [nvarchar](50),
        [querydwelltime] [float],
        [sessionid] [bigint],
        [sessionpagevieworder] [bigint])
        GO
        CREATE CLUSTERED INDEX mobiledata_clustered_index on mobiledata(clientid)
        GO

    Cuando se haya especificado la instrucción `GO` , se evaluarán las instrucciones anteriores. En primer lugar, se crea la tabla **mobiledata** y, a continuación, se agrega un índice agrupado a ella (es necesario para la base de datos SQL).

    Para comprobar que se ha creado la tabla, utilice lo siguiente:

        SELECT * FROM information_schema.tables
        GO

    Debería ver una salida similar a la siguiente:

        TABLE_CATALOG   TABLE_SCHEMA    TABLE_NAME      TABLE_TYPE
        sqooptest       dbo     mobiledata      BASE TABLE
3. Para salir de la utilidad tsql, escriba `exit` at the `1>` .

## <a name="sqoop-export"></a>Exportación de Sqoop
1. Desde la conexión SSH con HDInsight, utilice el siguiente comando para comprobar que Sqoop puede ver la base de datos SQL:

        sqoop list-databases --connect jdbc:sqlserver://<serverName>.database.windows.net:1433 --username <adminLogin> --password <adminPassword>

    Esto debe devolver una lista de bases de datos, incluida la base de datos **sqooptest** que creó anteriormente.
2. Use el comando siguiente para exportar los datos de **hivesampletable** a la tabla **mobiledata**:

        sqoop export --connect 'jdbc:sqlserver://<serverName>.database.windows.net:1433;database=sqooptest' --username <adminLogin> --password <adminPassword> --table 'mobiledata' --export-dir 'wasbs:///hive/warehouse/hivesampletable' --fields-terminated-by '\t' -m 1

    Esto indica a Sqoop que debe conectarse a la base de datos SQL, a la base de datos **sqooptest**, y exportar los datos de **wasbs:///hive/warehouse/hivesampletable** (archivos físicos de *hivesampletable*,) a la tabla **mobiledata**.
3. Una vez completado el comando, utilice lo siguiente para conectarse a la base de datos mediante TSQL:

        TDSVER=8.0 tsql -H <serverName>.database.windows.net -U <adminLogin> -P <adminPassword> -p 1433 -D sqooptest

    Una vez conectado, utilice las instrucciones siguientes para comprobar que los datos se exportaron a la tabla **mobiledata** :

        SELECT * FROM mobiledata
        GO

    Debería ver una lista de los datos de la tabla. Escriba `exit` para salir de la utilidad de tsql.

## <a name="sqoop-import"></a>Importación de Sqoop
1. Use lo siguiente para importar datos de la tabla **mobiledata** de SQL Database al directorio **wasbs:///tutorials/usesqoop/importeddata** de HDInsight:

        sqoop import --connect 'jdbc:sqlserver://<serverName>.database.windows.net:1433;database=sqooptest' --username <adminLogin> --password <adminPassword> --table 'mobiledata' --target-dir 'wasbs:///tutorials/usesqoop/importeddata' --fields-terminated-by '\t' --lines-terminated-by '\n' -m 1

    Los datos importados tendrán campos separados por un carácter de tabulación y las líneas terminarán con un carácter de nueva línea.
2. Una vez completada la importación, use el siguiente comando para enumerar los datos en el nuevo directorio:

        hadoop fs -text wasbs:///tutorials/usesqoop/importeddata/part-m-00000

## <a name="using-sql-server"></a>Uso de SQL Server
También puede utilizar Sqoop para importar y exportar datos de SQL Server, tanto en el centro de datos como en una máquina Virtual hospedada en Azure. Las diferencias entre el uso de la base de datos SQL y SQL Server son:

* HDInsight y SQL Server deben estar en la misma red virtual de Azure

  > [!NOTE]
  > HDInsight solo admite redes virtuales basadas en la ubicación y actualmente no funciona con redes virtuales basadas en grupos de afinidad.
  >
  >

    Cuando use SQL Server en el centro de datos, debe configurar la red virtual como de *sitio a sitio* o de *punto a sitio*.

  > [!NOTE]
  > En el caso de las redes virtuales de **punto a sitio**, SQL Server debe ejecutarse en la aplicación de configuración de clientes VPN, que se encuentra disponible en el **Panel** de la configuración de red virtual de Azure.
  >
  >

    Para más información sobre Azure Virtual Network, consulte [Información general sobre redes virtuales](../virtual-network/virtual-networks-overview.md).
* SQL Server estar configurado para permitir la autenticación SQL. Para obtener más información, consulte [Elegir un modo de autenticación](https://msdn.microsoft.com/ms144284.aspx)
* Es posible que tenga que configurar SQL Server para aceptar conexiones remotas. Para obtener más información, consulte [Solución de problemas de conexión al motor de base de datos de SQL Server](http://social.technet.microsoft.com/wiki/contents/articles/2102.how-to-troubleshoot-connecting-to-the-sql-server-database-engine.aspx)
* Debe crear la base de datos **sqooptest** en SQL Server con una utilidad como **SQL Server Management Studio** o **tsql**. Los pasos para usar la CLI de Azure solo funcionan para Azure SQL Database.

    Las instrucciones de TSQL para crear la tabla **mobiledata** son similares a las que se utilizan para la base de datos SQL, a excepción de la creación de un índice agrupado. Esto no es necesario para SQL Server:

        CREATE TABLE [dbo].[mobiledata](
        [clientid] [nvarchar](50),
        [querytime] [nvarchar](50),
        [market] [nvarchar](50),
        [deviceplatform] [nvarchar](50),
        [devicemake] [nvarchar](50),
        [devicemodel] [nvarchar](50),
        [state] [nvarchar](50),
        [country] [nvarchar](50),
        [querydwelltime] [float],
        [sessionid] [bigint],
        [sessionpagevieworder] [bigint])
* Cuando se conecta a SQL Server desde HDInsight, es posible que deba utilizar la dirección IP de SQL Server a menos que haya configurado un Sistema de nombres de dominio (DNS) para resolver los nombres de la Red virtual de Azure. Por ejemplo:

        sqoop import --connect 'jdbc:sqlserver://10.0.1.1:1433;database=sqooptest' --username <adminLogin> --password <adminPassword> --table 'mobiledata' --target-dir 'wasbs:///tutorials/usesqoop/importeddata' --fields-terminated-by '\t' --lines-terminated-by '\n' -m 1

## <a name="limitations"></a>Limitaciones
* Exportación masiva: con HDInsight basado en Linux, el conector Sqoop que se utiliza para exportar datos a Microsoft SQL Server o Base de datos SQL Azure no es compatible actualmente con las inserciones masivas.
* Procesamiento por lotes: con HDInsight basado en Linux, cuando se usa `-batch` al realizar inserciones, Sqoop realizará varias inserciones en lugar de procesar por lotes las operaciones de inserción.

## <a name="next-steps"></a>Pasos siguientes
Ahora ya ha aprendido a usar Sqoop. Para obtener más información, consulte:

* [Uso de Oozie con HDInsight][hdinsight-use-oozie]: use la acción Sqoop en un flujo de trabajo de Oozie.
* [Análisis de la información de retraso de vuelos con HDInsight][hdinsight-analyze-flight-data]: use Hive para analizar la información de retraso de los vuelos y luego use Sqoop para exportar los datos a una base de datos SQL de Azure.
* [Carga de datos en HDInsight][hdinsight-upload-data]: busque otros métodos para cargar datos en HDInsight o Azure Blob Storage.

[hdinsight-versions]:  hdinsight-component-versioning.md
[hdinsight-provision]: hdinsight-provision-clusters.md
[hdinsight-get-started]: hdinsight-hadoop-linux-tutorial-get-started.md
[hdinsight-storage]: ../hdinsight-hadoop-use-blob-storage.md
[hdinsight-analyze-flight-data]: hdinsight-analyze-flight-delay-data.md
[hdinsight-use-oozie]: hdinsight-use-oozie.md
[hdinsight-upload-data]: hdinsight-upload-data.md
[hdinsight-submit-jobs]: hdinsight-submit-hadoop-jobs-programmatically.md

[sqldatabase-get-started]: ../sql-database-get-started.md
[sqldatabase-create-configue]: ../sql-database-create-configure.md

[powershell-start]: http://technet.microsoft.com/library/hh847889.aspx
[powershell-install]: /powershell/azureps-cmdlets-docs
[powershell-script]: http://technet.microsoft.com/library/ee176949.aspx

[sqoop-user-guide-1.4.4]: https://sqoop.apache.org/docs/1.4.4/SqoopUserGuide.html



<!--HONumber=Dec16_HO2-->


