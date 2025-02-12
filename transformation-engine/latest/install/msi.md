---
title: Installation
---

The standalone Alfresco Document Transformation Engine is installed by using an `.msi` file where you can either:

* Select to install a T-Engine wrapper from the `.msi`.
* Install a hybrid version by using a Docker Compose file to install the T-Engine.

In previous versions the installation files were contained within a `.zip` file. This file also contained `.amp` files that enabled you to install the Document Transformation client into Alfresco Content Services. In the current version this is not possible.

* [Install with MSI](#install-with-msi)
* [Install T-Engine using Docker Compose](#install-t-engine-using-docker-compose)

## Install with MSI

> **Note:** When upgrading the Document Transformation Engine, the previous installation must be uninstalled first.
>
> * If your old version is earlier than 1.3.1, use the Control Panel **Uninstall a program** option to remove the old version, and then manually remove the Document Transformation Engine directory. By default, the Document Transformation Engine directory is `C:\Program Files (x86)\Transformation Engine\`.
> * If your old version is 1.3.1 or later, the new Document Transformation Engine MSI prompts you to uninstall the previous version. When the uninstall is complete, you can run the MSI package again to install the new version. There is no need to manually remove anything.

1. Download `alfresco-document-transformation-engine-server-2.3.0.msi` from the [Alfresco Support Portal](http://support.alfresco.com){:target="_blank"}.

2. Log into the Microsoft Windows Server as an administrator.

3. Double click the `.msi` installer package, and then click **Next**.

4. Review the supported software requirements, and then click **Next**.

5. (Optional) Select DTE T-Engine.

    > **Important:** If you do not intend to use the DTE T-Engine Docker image, you must select this option for DTE to work correctly.

6. Click **Next** and the license information screen displays.

7. Click **Next** and select an installation folder or accept the default folder, and then click **Next**.

8. Click **Next** to start the installation.

    You will see a progress bar and a command line window during the installation. The installer will show a confirmation when the installation is finished.

9. Click **Close** to finish the installation.

10. Verify that the installation has completed successfully.

    1. Check the Windows Services in the management console.

    2. Locate the new service called **Document Transformation Engine**, and check that it is **Started**.

> **Note:** Each time a file is transformed in Alfresco Content Services, the `.NET` program starts and Microsoft Office tries to check for a Certificate Revocation List (CRL). Depending on the access that the Document Transformation Engine has to the Internet when transforming a file, this check can delay the operation for up to two minutes, and will therefore, delay transformation of the file. To prevent this, use the Windows server firewall to block internet access for all office binaries.

11. Add the following property to `alfresco-global.properties`

```bash
localTransform.transform-dte.url=http:<dte hostname>:8080/transform-dte
```

<!-- (Will be commented back in once 2.4 is released)

## Install the Alfresco Transformation client into Alfresco Content Services

The Alfresco Transformation client is installed as two Alfresco Module Packages (AMP) files into Alfresco Content Services and requires the license to be updated.

Before starting verify that:

* Your Alfresco Content Services server is correctly configured and tested.
* You have the correct Document Transformation Engine ZIP file for the version of Alfresco Content Services that you are running.
* You have an updated license file (a `*.lic` file). You can request a license from the [Alfresco Support Portal](http://support.alfresco.com){:target="_blank"}.

1. Stop the Alfresco Content Services server.

2. Open a terminal (Linux) or command line window (Windows).

3. Unzip the `alfresco-document-transformation-engine-2.3.x.zip` file.

4. Copy `alfresco-document-transformation-engine-repo-2.3.x.amp` to the `<ALFRESCO_HOME>/amps` folder, and copy `alfresco-document-transformation-engine-share-2.3.x.amp` to the `<ALFRESCO_HOME>/amps_share` folder.

5. Install the AMP files using the Module Management Tool (MMT).

6. Copy your updated license file into the Alfresco Content Services installation folder.

    Delete all files with the extension `*.installed` in this directory.

7. Start the Alfresco Content Services server.

8. Monitor the Alfresco Content Services log.

    You will see successful log entries about the license installation and the installation of the Alfresco Module Package (depending on the configuration of your log level).
-->

## Install T-Engine using Docker Compose

To deploy the Document Transformation Engine T-Engine with the Transform Service, you'll need to update your Docker Compose file to include the Document Transformation Engine T-Engine.

> **Important:** You still need to install the Document Transformation Engine using the `.msi`.

> **Note:** While Docker Compose is often used for production deployments, the Docker Compose file provided is recommended for development and test environments only. Customers are expected to adapt this file to their own requirements, if they intend to use Docker Compose to deploy a production environment.

1. Add the Document Transformation Engine T-Engine container to your `docker-compose.yaml` file:

    ```yaml
    transform-dte-engine:
        image: quay.io/alfresco/transform-dte-engine:1.0.0
        mem_limit: 2g
        environment:
            JAVA_OPTS: " -Xms256m -Xmx512m -DdteServerUrl=http://<dte hostname>:8080/transformation-server"
            ACTIVEMQ_URL: "nio://activemq:61616"
            ACTIVEMQ_USER: "admin"
            ACTIVEMQ_PASSWORD: "admin"
            FILE_STORE_URL: "http://shared-file-store:8099/alfresco/api/-default-/private/sfs/versions/1/file"
        ports:
            - 8091:8090
        links:
            - activemq
    ```

2. Add the following `JAVA_OPTS` property to the `alfresco` container:

    ```yaml
    -DlocalTransform.transform-dte.url=http://transform-dte-engine:8090/
    ```

See the Content Services documentation - [T-Engine configuration](https://github.com/Alfresco/acs-packaging/blob/master/docs/creating-a-t-engine.md#t-engine-configuration){:target="_blank"} for more details. For further development, see [Content Transformers and Renditions Extension Points]({% link content-services/latest/develop/repo-ext-points/content-transformers-renditions.md %}).
