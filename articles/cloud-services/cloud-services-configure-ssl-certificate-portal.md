<properties 
	pageTitle="Configure SSL for a cloud service - Azure" 
	description="Learn how to specify an HTTPS endpoint for a web role and how to upload an SSL certificate to secure your application." 
	services="cloud-services" 
	documentationCenter=".net" 
	authors="Thraka" 
	manager="timlt" 
	editor=""/>

<tags 
	ms.service="cloud-services" 
	ms.workload="tbd" 
	ms.tgt_pltfrm="na" 
	ms.devlang="na" 
	ms.topic="article" 
	ms.date="06/28/2015"
	ms.author="adegeo"/>




# Configuring SSL for an application in Azure

> [AZURE.SELECTOR]
- [Azure Portal](cloud-services-configure-ssl-certificate.md)
- [Azure Preview Portal](cloud-services-configure-ssl-certificate-portal.md)

Secure Socket Layer (SSL) encryption is the most commonly used method of securing data sent across the internet. This common task discusses how to specify an HTTPS endpoint for a web role and how to upload an SSL certificate to secure your application.

> [AZURE.NOTE] The procedures in this task apply to Azure Cloud Services; for Websites, see [Configuring an SSL certificate for an Azure website](../web-sites-configure-ssl-certificate.md).

This task will use a production deployment; information on using a staging deployment is provided at the end of this topic.

Read [this](cloud-services-how-to-create-deploy-portal.md) first if you have not yet created a cloud service.

[AZURE.INCLUDE [websites-cloud-services-css-guided-walkthrough](../../includes/websites-cloud-services-css-guided-walkthrough.md)]

## Step 1: Get an SSL certificate

To configure SSL for an application, you first need to get an SSL certificate that has been signed by a Certificate Authority (CA), a trusted third-party who issues certificates for this purpose. If you do not already have one, you will need to obtain one from a company that sells SSL certificates.

The certificate must meet the following requirements for SSL certificates in Azure:

-   The certificate must contain a private key.
-   The certificate must be created for key exchange, exportable to a Personal Information Exchange (.pfx) file.
-   The certificate's subject name must match the domain used to access the cloud service. You cannot obtain an SSL certificate from a certificate authority (CA) for the cloudapp.net domain. You must acquire a custom domain name to use when access your service. When you request a certificate from a CA the certificate's subject name must match the custom domain name used to access your application. For example, if your custom domain name is **contoso.com** you would request a certificate from your CA for ***.contoso.com** or **www.contoso.com**.
-   The certificate must use a minimum of 2048-bit encryption.

For test purposes, you can create and use a self-signed certificate. A self-signed certificate is not authenticated through a CA and can use the cloudapp.net domain as the website URL. For example, the task below uses a self-signed certificate in which  the common name (CN) used in the certificate is **sslexample.cloudapp.net**. For details about how to create a self-signed certificate using IIS Manager, See [How to create a certificate for a role][].

Next, you must include information about the certificate in your service definition and service configuration files.

<a name="modify"> </a>
## Step 2: Modify the service definition and configuration files

Your application must be configured to use the certificate, and an HTTPS endpoint must be added. As a result, the service definition and service configuration files need to be updated.

1.  In your development environment, open the service definition file
    (CSDEF), add a **Certificates** section within the **WebRole**
    section, and include the following information about the
    certificate:

        <WebRole name="CertificateTesting" vmsize="Small">
        ...
            <Certificates>
                <Certificate name="SampleCertificate" 
							 storeLocation="LocalMachine" 
                    		 storeName="CA" />
            </Certificates>
        ...
        </WebRole>

    The **Certificates** section defines the name of our certificate, its location, and the name of the store where it is located. We have chosen to store the certificate in the CA (Certificate Authority)tore, but you can choose other options as well. See [How to associate a certificate with a service][] for more information.

2.  In your service definition file, add an **InputEndpoint** element
    within the **Endpoints** section to enable HTTPS:

        <WebRole name="CertificateTesting" vmsize="Small">
        ...
            <Endpoints>
                <InputEndpoint name="HttpsIn" protocol="https" port="443" 
                    certificate="SampleCertificate" />
            </Endpoints>
        ...
        </WebRole>

3.  In your service definition file, add a **Binding** element within
    the **Sites** section. This adds an HTTPS binding to map the
    endpoint to your site:

        <WebRole name="CertificateTesting" vmsize="Small">
        ...
            <Sites>
                <Site name="Web">
                    <Bindings>
                        <Binding name="HttpsIn" endpointName="HttpsIn" />
                    </Bindings>
                </Site>
            </Sites>
        ...
        </WebRole>

    All of the required changes to the service definition file have been
    completed, but you still need to add the certificate information to
    the service configuration file.

4.  In your service configuration file (CSCFG), ServiceConfiguration.Cloud.cscfg, add a **Certificates**
    section within the **Role** section, replacing the sample thumbprint
    value shown below with that of your certificate:

        <Role name="Deployment">
        ...
            <Certificates>
                <Certificate name="SampleCertificate" 
                    thumbprint="9427befa18ec6865a9ebdc79d4c38de50e6316ff" 
                    thumbprintAlgorithm="sha1" />
            </Certificates>
        ...
        </Role>

(The example above uses **sha1** for the thumbprint algorithm. Specify the appropriate value for your certificate's thumbprint algorithm.)

Now that the service definition and service configuration files have
been updated, package your deployment for uploading to Azure. If
you are using **cspack**, ensure that you don't use the
**/generateConfigurationFile** flag, as that will overwrite the
certificate information you just inserted.

## Step 3: Upload a certificate

Connect to the portal and...

1. Select your cloud service by either:
    - In the Portal, select your **Cloud Service**. (Which will be in the **Browse All/Recent area**.)
    
        ![Publish your cloud service](media/cloud-services-configure-ssl-certificate-portal/browse.png)
    
        **OR**
        
    - Under **Browse All** select **Cloud Services** under **Filter By** and select the cloud service instance you want. 

3. Open the **settings** for the cloud service.

    ![Open settings](media/cloud-services-configure-ssl-certificate-portal/all-settings.png)

4. Click on **Certificates**.

    ![Click the certificates icon](media/cloud-services-configure-ssl-certificate-portal/certificate-item.png)

4. Provide the **File**, **Password**, then click **Upload**.

## Step 4: Connect to the role instance by using HTTPS

Now that your deployment is up and running in Azure, you can
connect to it using HTTPS.
    
1.  Click on the **Site URL** to open up the web browser.

    ![Click on Site URL](media/cloud-services-configure-ssl-certificate-portal/navigate.png)

2.  In your web browser, modify the link to use **https** instead of **http**, and then visit the page.

    >[AZURE.NOTE] If you are using a self-signed certificate, when you browse to an HTTPS endpoint that's associated with the self-signed certificate you will see a certificate error in the browser. Using a certificate signed by a trusted certification authority will eliminate this problem; in the meantime, you can ignore the error. (Another option is to add the self-signed certificate to the user's trusted certificate authority certificate store.)

    ![Site preview](media/cloud-services-configure-ssl-certificate-portal/show-site.png)

    >[AZURE.TIP] If you want to use SSL for a staging deployment instead of a production deployment, you'll first need to determine the URL used for the staging deployment. Once your cloud service has been deployed, the URL to the staging envorionment is determined by the **Deployment ID** GUID in this format: `https://deployment-id.cloudapp.net/`  
      
    >Create a certificate with the common name (CN) equal to the GUID-based URL (for example, **328187776e774ceda8fc57609d404462.cloudapp.net**), use the portal to add the certificate to your staged cloud service, add the certificate information to your CSDEF and CSCFG files, repackage your application, and update your staged deployment to use the new package and CSCFG file.

## Next steps

* [How to associate a certificate with a service][]
* [How to configure an SSL certificate on an HTTPS endpoint][]

[How to create a certificate for a role]: http://msdn.microsoft.com/library/azure/gg432987.aspx
[How to associate a certificate with a service]: http://msdn.microsoft.com/library/azure/gg465718.aspx
[How to configure an SSL certificate on an HTTPS endpoint]: http://msdn.microsoft.com/library/azure/ff795779.aspx
[Azure Portal]: http://portal.azure.com/
