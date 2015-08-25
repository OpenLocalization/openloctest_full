<properties 
	pageTitle="Ruby on Rails Web App on Azure using Linux VM" 
	description="Host a Ruby on Rails-based website on Azure using a Linux virtual machine." 
	services="virtual-machines" 
	documentationCenter="ruby" 
	authors="MikeWasson" 
	manager="wpickett" 
	editor=""/>

<tags 
	ms.service="virtual-machines" 
	ms.workload="web" 
	ms.tgt_pltfrm="vm-linux" 
	ms.devlang="ruby" 
	ms.topic="article" 
	ms.date="06/09/2015" 
	ms.author="mwasson"/>





#Ruby on Rails Web application on an Azure VM

This tutorial shows how to host a Ruby on Rails website on Azure using a Linux virtual machine.  

This tutorial was validated using Ubuntu Server 14.04 LTS. If you use a different Linux distribution, you might need to modify the steps to install Rails.

## Create an Azure VM

Start by creating an Azure VM with a Linux image. 

To create the VM, you can use the Azure Management Portal or the Azure Command-Line Interface (CLI).

### Azure Management Portal

1. Sign into the [Azure Management Portal](http://manage.windowsazure.com)
2. Click **New** > **Compute** > **Virtual Machine** > **Quick Create**. Select a Linux image.
3. Enter a password.

After the VM is provisioned, click on the VM name, and click **Dashboard**. Find the SSH endpoint, listed under **SSH Details**.

### Azure CLI

Follow the steps in [Create a Virtual Machine Running Linux][vm-instructions].

After the VM is provisioned, you can get the SSH endpoint by running the following command:

	azure vm endpoint list <vm-name>  

## Install Ruby on Rails

1. Use SSH to connect to the VM. 
	
2. From the SSH session, use the following commands to install Ruby on the VM:

		sudo apt-get update -y
		sudo apt-get upgrade -y
		sudo apt-get install ruby ruby-dev build-essential libsqlite3-dev zlib1g-dev nodejs -y

	The installation may take a few minutes. When it completes, use the following command to verify that Ruby is installed:

		ruby -v

	This returns the version of Ruby that was installed.

3. Use the following command to install Rails:

		sudo gem install rails --no-rdoc --no-ri

	Use the --no-rdoc and --no-ri flags to skip installing the documentation, which is faster.

## Create and run an app

While still logged in via SSH, run the following commands:

	rails new myapp
	cd myapp
	rails server -b 0.0.0.0 -p 3000

The [new](http://guides.rubyonrails.org/command_line.html#rails-new) command creates a new Rails app. The [server](http://guides.rubyonrails.org/command_line.html#rails-server) command starts the WEBrick web server that comes with Rails. (For production use, you would probably want to use a different server, such as Unicorn or Passenger.)

You should see output similar to the following. 

	=> Booting WEBrick
	=> Rails 4.2.1 application starting in development on http://0.0.0.0:3000
	=> Run `rails server -h` for more startup options
	=> Ctrl-C to shutdown server
	[2015-06-09 23:34:23] INFO  WEBrick 1.3.1
	[2015-06-09 23:34:23] INFO  ruby 1.9.3 (2013-11-22) [x86_64-linux]
	[2015-06-09 23:34:23] INFO  WEBrick::HTTPServer#start: pid=27766 port=3000


## Add an endpoint

1. Go to the [Azure Management Portal][management-portal] and select your VM.

	![virtual machine list][vmlist]

2. Select **ENDPOINTS** at the top of the page, and then click **+ ADD ENDPOINT** at the bottom of the page.

	![endpoints page][endpoints]

3. In the **ADD ENDPOINT** dialog, select "Add a standalone endpoint" and click the **Next** arrow.

	![new endpoint dialog][new-endpoint1]

3. In the next dialog page, enter the following information:

	* **NAME**: HTTP

	* **PROTOCOL**: TCP

	* **PUBLIC PORT**: 80

	* **PRIVATE PORT**: 3000

	This will create a public port of 80 that will route traffic to the private port of 3000, where the Rails server is listening.

	![new endpoint dialog][new-endpoint]

4. Click the check mark to save the endpoint.

5. A message should appear that states **UPDATE IN PROGRESS**. Once this message disappears, the endpoint is active. You may now test your application by navigating to the DNS name of your virtual machine. The website should appear similar to the following:

	![default rails page][default-rails-cloud]


##<a id="next"></a>Next steps

In this tutorial, you did most of the steps manually. In a production environment, you would write your app on a development machine and deploy it to the Azure VM. Also, most production environments host the Rails application in conjunction with another server process such as Apache or NginX, which handles request routing to multiple instances of the Rails application and serving static resources. For more information, see http://rubyonrails.org/deploy/.

To learn more about Ruby on Rails, visit the [Ruby on Rails Guides][rails-guides].

To use Azure services from your Ruby application, see:

* [Store unstructured data using blobs][blobs]

* [Store key/value pairs using tables][tables]

* [Serve high bandwidth content with the Content Delivery Network][cdn-howto]



<!-- WA.com links -->
[blobs]: ../storage-ruby-how-to-use-blob-storage.md

[cdn-howto]: /develop/ruby/app-services/

[management-portal]: https://manage.windowsazure.com/

[tables]: /develop/ruby/how-to-guides/table-service/

[vm-instructions]: virtual-machines-linux-tutorial.md


<!-- External Links -->
[rails-guides]: http://guides.rubyonrails.org/

[sqlite3]: http://www.sqlite.org/

<!-- Images -->

[default-rails-cloud]: ./media/virtual-machines-ruby-rails-web-app-linux/basicrailscloud.png

[vmlist]: ./media/virtual-machines-ruby-rails-web-app-linux/vmlist.png

[endpoints]: ./media/virtual-machines-ruby-rails-web-app-linux/endpoints.png

[new-endpoint]: ./media/virtual-machines-ruby-rails-web-app-linux/newendpoint.png

[new-endpoint1]: ./media/virtual-machines-ruby-rails-web-app-linux/newendpoint1.png
 