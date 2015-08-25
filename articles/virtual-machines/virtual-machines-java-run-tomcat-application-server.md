<properties
	pageTitle="Tomcat on a virtual machine - Azure tutorial"
	description="Learn how to create a Windows Virtual machine and configure the machine to run a Apache Tomcat application server."
	services="virtual-machines"
	documentationCenter="java"
	authors="rmcmurray"
	manager="wpickett"
	editor="jimbe"/>

<tags
	ms.service="virtual-machines"
	ms.workload="web"
	ms.tgt_pltfrm="vm-windows"
	ms.devlang="Java"
	ms.topic="article"
	ms.date="06/03/2015"
	ms.author="robmcm"/>

# How to run a Java application server on a virtual machine

With Azure, you can use a virtual machine to provide server capabilities. As an example, a virtual machine running on Azure can be configured to host a Java application server, such as Apache Tomcat. After you complete this guide, you will have an understanding of how to create a virtual machine running on Azure and configure it to run a Java application server.

You will learn:

* How to create a virtual machine that has a Java Development Kit (JDK) already installed.
* How to remotely sign in to your virtual machine.
* How to install a Java application server on your virtual machine.
* How to create an endpoint for your virtual machine.
* How to open a port in the firewall for your application server.

For the purposes of this tutorial, an Apache Tomcat application server will be installed on a virtual machine. The completed installation will result in a Tomcat installation such as the following.

![Virtual machine running Apache Tomcat][virtual_machine_tomcat]

[AZURE.INCLUDE [create-account-and-vms-note](../../includes/create-account-and-vms-note.md)]

## To create a virtual machine

1. Sign in to the [Azure portal](https://manage.windowsazure.com).
2. Click **New**, click **Compute**, click **Virtual machine**, and then click **From Gallery**.
3. In the **Virtual machine image select** dialog box, select **JDK 7 Windows Server 2012**.
Note that **JDK 6 Windows Server 2012** is available if you have legacy applications that are not ready to run in JDK 7.
4. Click **Next**.
5. In the <strong>Virtual machine configuration</strong> dialog box:
    1. Specify a name for the virtual machine.
    2. Specify the size to use for the virtual machine.
    3. Enter a name for the administrator in the **User Name** field. Remember this name and the password you will enter next, you will use them when you remotely sign in to the virtual machine.
    4. Enter a password in the **New password** field, and re-enter it in the **Confirm** field. This is the Administrator account password.
    5. Click **Next**.
6. In the next <strong>Virtual machine configuration</strong> dialog box:
    1. For **Cloud service**, use the default **Create a new cloud service**.
    2. The value for **Cloud service DNS name** must be unique across cloudapp.net. If needed, modify this value so that Azure indicates it is unique.
    2. Specify a region, affinity group, or virtual network. For the purposes of this tutorial, specify a region such as **West US**.
    2. For **Storage Account**, select **Use an automatically generated storage account**.
    3. For **Availability Set**, select **(None)**.
    4. Click **Next**.
7. In the final <strong>Virtual machine configuration</strong> dialog box:
    1. Accept the default endpoint entries.
    2. Click **Complete**.

## To remotely sign in to your virtual machine

1. Log on to the [Management portal](https://manage.windowsazure.com).
2. Click **Virtual machines**.
3. Click the name of the virtual machine that you want to sign in to.
4. After the virtual machine has started, a pop-up menu at the bottom of the page allows connections.
5. Click **Connect**.
6. Respond to the prompts as needed to connect to the virtual machine. This should entail saving or opening the .rdp file that contains the connection details. You might have to copy the url:port as the last part of the first line of the .rdp file and paste it in a remote sign-in application.

## To install a Java application server on your virtual machine

You can copy a Java application server to your virtual machine, or you can install a Java application server through an installer.

For the purposes of this tutorial, Tomcat will be installed.

1. When you are signed in to your virtual machine, open a browser session to [Apache Tomcat](http://tomcat.apache.org/download-70.cgi).
2. Double-click the link for **32-bit/64-bit Windows Service Installer**. By using this technique, Tomcat will be installed as a Windows service.
3. When prompted, choose to run the installer.
4. Within the **Apache Tomcat Setup** wizard, follow the prompts to install Tomcat. For the purposes of this tutorial, accepting the defaults is fine. When you reach the **Completing the Apache Tomcat Setup Wizard** dialog box, you can optionally check **Run Apache Tomcat** to have Tomcat start now. Click **Finish** to complete the Tomcat setup process.

## To start Tomcat
If you did not choose to run Tomcat in the **Completing the Apache Tomcat Setup Wizard** dialog box, start it by opening a command prompt on your virtual machine and running **net start Tomcat7**.

You should now see Tomcat running if you run the virtual machine's browser and open <http://localhost:8080>.

To see Tomcat running from external machines, you need to create an endpoint and open a port.

## To create an endpoint for your virtual machine
1. Sign in to the [Management portal](https://manage.windowsazure.com).
2. Click **Virtual machines**.
3. Click the name of the virtual machine that is running your Java application server.
4. Click **Endpoints**.
5. Click **Add**.
6. In the **Add endpoint** dialog box, ensure **Add standalone endpoint** is selected, and then click **Next**.
7. In the <strong>New endpoint details</strong> dialog box:
    1. Specify a name for the endpoint; for example, **HttpIn**.
    2. Specify **TCP** for the protocol.
    3. Specify **80** for the public port.
    4. Specify **8080** for the private port.
    5. Click the **Complete** button to close the dialog box. Your endpoint will now be created.

## To open a port in the firewall for your virtual machine
1. Sign in to your virtual machine.
2. Click **Windows Start**.
3. Click **Control Panel**.
4. Click **System and Security**, click **Windows Firewall**, and then click **Advanced Settings**.
5. Click **Inbound Rules**, and then click **New Rule**.

 ![New inbound rule][NewIBRule]

6. For the **Rule Type**, select **Port**, and then click **Next**.

 ![New inbound rule port][NewRulePort]

7. On the **Protocol and Ports** screen, select **TCP**, specify **8080** as the **Specific local port**, and then click **Next**.

 ![New inbound rule ][NewRuleProtocol]

8. On the **Action** screen, select **Allow the connection**, and then click **Next**.

 ![New inbound rule action][NewRuleAction]

9. On the **Profile** screen, ensure that **Domain**, **Private**, and **Public** are selected, and then click **Next**.

 ![New inbound rule profile][NewRuleProfile]

10. On the **Name** screen, specify a name for the rule, such as **HttpIn** (the rule name is not required to match the endpoint name, however), and then click **Finish**.  

 ![New inbound rule name][NewRuleName]

At this point, your Tomcat website should be viewable from an external browser by using a URL of the form **http://*your\_DNS\_name*.cloudapp.net**, where ***your\_DNS\_name*** is the DNS name you specified when you created the virtual machine.

## Application lifecycle considerations
* You could create your own web application archive (WAR) and add it to the **webapps** folder. For example, create a basic Java Service Page (JSP) dynamic web project and export it as a WAR file, copy the WAR to the Apache Tomcat **webapps** folder on the virtual machine, then run it in a browser.
* By default when the Tomcat service is installed, it is set to start manually. You can switch it to start automatically by using the Services snap-in. Start the Services snap-in by clicking **Windows Start**, **Administrative Tools**, and then **Services**. Double-click the **Apache Tomcat** service  and set **Startup type** to **Automatic**.

    ![Setting a service to start automatically][service_automatic_startup]

    The benefit of having Tomcat start automatically is that it will start if the virtual machine is rebooted (for example, after software updates that require a reboot are installed).

## Next steps
Learn about other services (such as Azure Storage, service bus, and SQL Database) that you may want to include with your Java applications by viewing the information available at the [Java Developer Center](http://azure.microsoft.com/develop/java/).

[virtual_machine_tomcat]: ./media/virtual-machines-java-run-tomcat-application-server/WA_VirtualMachineRunningApacheTomcat.png

[service_automatic_startup]: ./media/virtual-machines-java-run-tomcat-application-server/WA_TomcatServiceAutomaticStart.png









[NewIBRule]: ./media/virtual-machines-java-run-tomcat-application-server/NewInboundRule.png
[NewRulePort]: ./media/virtual-machines-java-run-tomcat-application-server/NewRulePort.png
[NewRuleProtocol]: ./media/virtual-machines-java-run-tomcat-application-server/NewRuleProtocol.png
[NewRuleAction]: ./media/virtual-machines-java-run-tomcat-application-server/NewRuleAction.png
[NewRuleName]: ./media/virtual-machines-java-run-tomcat-application-server/NewRuleName.png
[NewRuleProfile]: ./media/virtual-machines-java-run-tomcat-application-server/NewRuleProfile.png
