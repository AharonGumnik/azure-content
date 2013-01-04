<properties linkid="dev-php-how-to-cloud-services" urlDisplayName="PHP Cloud Services" pageTitle="How to create PHP web and worker roles" metaKeywords="Azure PHP cloud services, web role, worker role" metaDescription="Learn how to create PHP web and worker roles in Windows Azure." metaCanonical="" umbracoNaviHide="0" disqusComments="1" writer="bswan" editor="mollybos" manager="paulettm" />

<div chunk="../chunks/article-left-menu.md" />

#How to create PHP web and worker roles

This guide will show you how to create PHP web or worker roles in a Windows development environment, choose a specific version of PHP from the "built-in" versions available, change the PHP configuration, enable extensions, and finally, how to deploy to Windows Azure. It also describes how to configure a web or worker role to use a PHP runtime (with custom configuration and extensions) that you provide.

<h2 id="TableOfContents">Table of Contents</h2>
* [What are PHP web and worker roles?](#WhatIs)
* [Download the Windows Azure SDK for PHP](#DownloadSdk)
* [How to: Create a Cloud Services project](#CreateProject)
* [How to: Add PHP Web and worker roles](#AddRole)
* [How to: Specify the built-in PHP version](#SpecifyPHPVersion)
* [How to: Customize the built-in PHP runtime](#CustomizePHP)
* [How to: Use your own PHP runtime](#OwnPHP)
* [How to: Run your application in the Compute and Storage Emulators](#Emulators)
* [How to: Publish your application](#Publish) 

<h2 id="WhatIs">What are PHP web and worker roles?</h2>
Windows Azure provides three compute models for running applications: [Windows Azure Web Sites][execution model-web sites], [Windows Azure Virtual Machines][execution model-vms], and [Windows Azure Cloud Services][execution model-cloud services]. All three models support PHP. Cloud Services, which include web and worker roles, provide *Platform as a Service (PaaS)*. Within a cloud service, a web role provides a dedicated Internet Information Services (IIS) web server to host front-end web applications, while a worker role can run asynchronous, long-running or perpetual tasks independent of user interaction or input.

For more information, see [What is a Cloud Service?].

<h2 id="DownloadSdk">Download the Windows Azure SDK for PHP</h2>

The [Windows Azure SDK for PHP] consists of several components. This article will use to of them: Windows Azure PowerShell and the Windows Azure Emulators. These two components can be installed via the Microsoft Web Platform Installer here: [Install Windows Azure PowerShell and the Windows Azure Emulators][install ps and emulators].

<h2 id="CreateProject">How to: Create a Cloud Services project</h2>

The first step in creating a PHP web or worker role is to create a Windows Azure Service project. A Windows Azure Service project serves as a logical container for web and worker roles, and contains the project's [service definition (.csdef)] and [service configuration (.cscfg)] files. 

To create a new Windows Azure Servcie project, execute the following command:

	PS C:\>New-AzureServiceProject myProject

This command will create a new directory (`myProject`) to which you can add web and worker roles.

<h2 id="AddRole">How to: Add PHP web or worker roles</h2>

To add a PHP web role to a project, run the following command from within the project's root directory:

	PS C:\myProject> Add-AzurePHPWebRole roleName

For a worker role, use this command:

	PS C:\myProject> Add-AzurePHPWorkerRole roleName

<div class="dev-callout"> 
<b>Note</b> 
<p>The <code>roleName</code> parameter is optional. If it is omitted, the role name will be automatically generated. The first web role created will be <code>WebRole1</code>, the second <code>WebRole2</code>, and so on. The first worker role created will be <code>WorkerRole1</code>, the second <code>WorkerRole2</code>, and so on.</p> 
</div>

<h2 id="SpecifyPHPVerison">How to: Specify the built-in PHP Version</h2>

When you add a PHP web or worker role to a project, the project's configuration files are modified so that PHP will be installed on each web or worker instance of your application when it is deployed. To see the version of PHP that will be installed by default, run the following command:

	PS C:\myProject> Get-AzureServiceProjectRoleRuntime

The output from the command above will look similar to what is shown below. In this example, the `IsDefault` flag is set to `true` for PHP 5.3.17, indicating that it will be the default PHP version installed. 

	Runtime Version		PackageUri						IsDefault
	------- ------- 	----------  					---------
   	Node 0.6.17      	http://nodertncu.blob.core...   False
   	Node 0.6.20         http://nodertncu.blob.core...   True
   	Node 0.8.4          http://nodertncu.blob.core...   False
	IISNode 0.1.21      http://nodertncu.blob.core...   True
  	Cache 1.8.0         http://nodertncu.blob.core...   True
    PHP 5.3.17          http://nodertncu.blob.core...   True
    PHP 5.4.0           http://nodertncu.blob.core...   False

You can set the PHP runtime version to any of the PHP versions that are listed. For example, to set the PHP version (for a role with name `roleName`) to 5.4.0, use the following command:

	PS C:\myProject> Set-AzureServiceProjectRole roleName php 5.4.0

<div class="dev-callout"> 
<b>Note</b> 
<p>More PHP versions may be available in the future, and the available versions may change.</p> 
</div>

<h2 id="CustomizePHP">How to: Customize the built-in PHP runtime</h2>

You have complete control over the configuration of the PHP runtime that is installed when you follow the steps above, including modification of `php.ini` settings and enabling of extensions.

To customize the built-in PHP runtime, follow these steps:

1. Add a new folder, named `php`, to the `bin` directory of your web role. For a worker role, add it to the role's root directory.
2. In the `php` folder, create another folder called `ext`. Put any `.dll` extension files (e.g. `php_mongo.dll`) you want to enable in this folder.
3. Add a `php.ini` file to the `php` folder. Enable any custom extensions and set any PHP directives in this file. For example, if you wanted to turn `display_errors` on and enable the `php_mongo.dll` extension, the contents of your `php.ini` file would be as follows:

		display_errors=On
		extension=php_mongo.dll

<div class="dev-callout"> 
<b>Note</b> 
<p>Any settings that you don't explicity set in the <code>php.ini</code> file that you provide will automatically be set to their default values. However, keep in mind that you can add a complete <code>php.ini</code> file. </p> 
</div>

<h2 id="OwnPHP">How to: Use your own PHP runtime</h2>
In some cases, instead of selecting a built-in PHP runtime and configuring it as described above, you may want to provide your own PHP runtime. For example, you can use the same PHP runtime in a web or worker role that you use in your development environment, making it easier to ensure that application will not change behavior in your production environment.

<h3 id="OwnPHPWebRole">Configuring a web role to use your own PHP runtime</h3>

To configure a web role to use a PHP runtime that you provide, follow the steps below.

1. Create a Windows Azure Service project and add a PHP web role as described in the [How to: Create a cloud services project](#CreateProject) and [How to: Add PHP web or worker roles](#AddRole) sections above.
2. Create a `php` folder in the `bin` folder that is in your web role's root directory, then add your PHP runtime (all binaries, configuration files, subfolders, etc.) to the `php` folder.
3. (OPTIONAL) If your PHP runtime uses the [Microsoft Drivers for PHP for SQL Server][sqlsrv drivers], you will need to configure your web role to install [SQL Server Native Client 2012][sql native client] when it is provisioned. To do this, add the `sqlncli.msi` installer to the `bin` folder in your web role's root directory. You can download the installer here: [sqlncli.msi x64 installer]. The startup script described in the next step will silently run the installer when the role is provisioned. If your PHP runtime does not use the Microsoft Drivers for PHP for SQL Server, you can remove the following line from the script shown in the next step:

		msiexec /i sqlncli.msi /qn IACCEPTSQLNCLILICENSETERMS=YES

4. The next step is to define a startup task that configures [Internet Information Services (IIS)][iis.net] to use your PHP runtime to handle requests for `.php` pages. To do this, open the `setup_web.cmd` file (in the `bin` file of your web role's root directory) in a text editor and replace its contents with the following script:

		@ECHO ON
		cd "%~dp0"
		
		if "%EMULATED%"=="true" exit /b 0
		
		msiexec /i sqlncli.msi /qn IACCEPTSQLNCLILICENSETERMS=YES
		
		SET PHP_FULL_PATH=%~dp0php\php-cgi.exe
		SET NEW_PATH=%PATH%;%RoleRoot%\base\x86
		
		%WINDIR%\system32\inetsrv\appcmd.exe set config -section:system.webServer/fastCgi /+"[fullPath='%PHP_FULL_PATH%',maxInstances='12',idleTimeout='60000',activityTimeout='3600',requestTimeout='60000',instanceMaxRequests='10000',protocol='NamedPipe',flushNamedPipe='False']" /commit:apphost
		%WINDIR%\system32\inetsrv\appcmd.exe set config -section:system.webServer/fastCgi /+"[fullPath='%PHP_FULL_PATH%'].environmentVariables.[name='PATH',value='%NEW_PATH%']" /commit:apphost
		%WINDIR%\system32\inetsrv\appcmd.exe set config -section:system.webServer/fastCgi /+"[fullPath='%PHP_FULL_PATH%'].environmentVariables.[name='PHP_FCGI_MAX_REQUESTS',value='10000']" /commit:apphost
		%WINDIR%\system32\inetsrv\appcmd.exe set config -section:system.webServer/handlers /+"[name='PHP',path='*.php',verb='GET,HEAD,POST',modules='FastCgiModule',scriptProcessor='%PHP_FULL_PATH%',resourceType='Either',requireAccess='Script']" /commit:apphost
		%WINDIR%\system32\inetsrv\appcmd.exe set config -section:system.webServer/fastCgi /"[fullPath='%PHP_FULL_PATH%'].queueLength:50000"

5. Add your application files to your web role's root directory. This will be the web server's root directory.

6. Publish your application as described in the [How to: Publish your applicaiton](#Publish) section below.

<div class="dev-callout"> 
<b>Note</b> 
<p>The <code>download.ps1</code> script (in the <code>bin</code> folder of the web role's root directory) can be deleted after following the steps described above for using your own PHP runtime.</p> 
</div>

<h3 id="OwnPHPWorkerRole">Configuring a worker role to use your own PHP runtime</h3>

To configure a worker role to use a PHP runtime that you provide, follow the steps below.

1. Create a Windows Azure Service project and add a PHP worker role as described in the [How to: Create a cloud services project](#CreateProject) and [How to: Add PHP web or worker roles](#AddRole) sections above.
2. Create a `php` folder in the worker role's root directory, then add your PHP runtime (all binaries, configuration files, subfolders, etc.) to the `php` folder.
3. (OPTIONAL) If your PHP runtime uses [Microsoft Drivers for PHP for SQL Server][sqlsrv drivers], you will need to configure your worker role to install [SQL Server Native Client 2012][sql native client] when it is provisioned. To do this, add the `sqlncli.msi` installer to the worker role's root directory. You can download the installer here: [sqlncli.msi x64 installer]. The startup script described in the next step will silently run the installer when the role is provisioned. If your PHP runtime does not use the Microsoft Drivers for PHP for SQL Server, you can remove the following line from the script shown in the next step:

		msiexec /i sqlncli.msi /qn IACCEPTSQLNCLILICENSETERMS=YES

4. The next step is to define a startup task that adds your `php.exe` executable to the worker role's PATH environment variable when the role is provisioned. To do this, open the `setup_worker.cmd` file (in the worker role's root directory) in a text editor and replace its contents with the following script:

		@echo on

		cd "%~dp0"

		echo Granting permissions for Network Service to the web root directory...
		icacls ..\ /grant "Network Service":(OI)(CI)W
		if %ERRORLEVEL% neq 0 goto error
		echo OK

		if "%EMULATED%"=="true" exit /b 0

		msiexec /i sqlncli.msi /qn IACCEPTSQLNCLILICENSETERMS=YES

		setx Path "%PATH%;%~dp0php" /M

		if %ERRORLEVEL% neq 0 goto error

		echo SUCCESS
		exit /b 0

		:error

		echo FAILED
		exit /b -1	

5. Add your application files to your worker role's root directory.

6. Publish your application as described in the [How to: Publish your applicaiton](#Publish) section below.

<h2 id="Emulator">How to: Run your application in the Compute and Storage Emulators</h2>

The Windows Azure Compute and Storage Emulators provide a local environment in which you can test your Windows Azure applicaiton before deploying it to the cloud. There are some differences between the emulators and the Windows Azure environment. To understand this better, see [Differences Between the Compute Emulator and Windows Azure](http://msdn.microsoft.com/en-us/library/windowsazure/gg432960.aspx) and [Differences Between the Storage Emulator and Windows Azure Storage Services](http://msdn.microsoft.com/en-us/library/windowsazure/gg433135.aspx).

Note that you must have PHP installed locally to use the Compute Emulator. The Compute Emulator will use your local PHP installation to run your application.

To run your project in the emulators, execute the following command from your project's root directory:

	PS C:\MyProject> Start-AzureEmulator

You will see out put similar to this:

	Creating local package...
	Starting Emulator...
	Role is running at http://127.0.0.1:81
	Started

You can see your application running in the emulator by opening a web browser and browsing to local address shown in the output (`http://127.0.0.1:81` in the example output above).

To stop the emulators, execute this command:

	PS C:\MyProject> Stop-AzureEmulator

<h2 id="Publish">How to: Publish your application</h2>

To publish your application, you need to first import your publish settings  with the **Import-PublishSettingsFile** cmdlet, then you can publish your application with the **Publish-AzureServiceProject** cmdlet. Details on using each of these cmdlets can be found in [How to: Import publish settings] and [How to: Deploy a cloud service to Windows Azure] respectively.

[execution model-web sites]: /en-us/develop/net/fundamentals/compute/#WebSites
[execution model-vms]: /en-us/develop/net/fundamentals/compute/#VMachine
[execution model-cloud services]: /en-us/develop/net/fundamentals/compute/#CloudServices
[Windows Azure SDK for PHP]: /en-us/develop/php/common-tasks/download-php-sdk/
[install ps and emulators]: http://go.microsoft.com/fwlink/?LinkId=253447&clcid=0x409
[What is a Cloud Service?]: /en-us/manage/services/cloud-services/what-is-a-cloud-service/
[service definition (.csdef)]: http://msdn.microsoft.com/en-us/library/windowsazure/ee758711.aspx
[service configuration (.cscfg)]: http://msdn.microsoft.com/en-us/library/windowsazure/ee758710.aspx
[iis.net]: http://www.iis.net/
[sql native client]: http://msdn.microsoft.com/en-us/sqlserver/aa937733.aspx
[sqlsrv drivers]: http://php.net/sqlsrv
[sqlncli.msi x64 installer]: http://go.microsoft.com/fwlink/?LinkID=239648
[How to: Import publish settings]: /en-us/develop/php/how-to-guides/powershell-cmdlets/#ImportPubSettings
[How to: Deploy a cloud service to Windows Azure]: /en-us/develop/php/how-to-guides/powershell-cmdlets/#Deploy