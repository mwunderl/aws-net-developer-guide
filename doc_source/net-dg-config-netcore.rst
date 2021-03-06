.. Copyright 2010-2019 Amazon.com, Inc. or its affiliates. All Rights Reserved.

   This work is licensed under a Creative Commons Attribution-NonCommercial-ShareAlike 4.0
   International License (the "License"). You may not use this file except in compliance with the
   License. A copy of the License is located at http://creativecommons.org/licenses/by-nc-sa/4.0/.

   This file is distributed on an "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND,
   either express or implied. See the License for the specific language governing permissions and
   limitations under the License.

.. _net-dg-config-netcore:

#########################################
Configuring the |sdk-net| with |net-core|
#########################################

One of the biggest changes in |net-core| is the removal of :code:`ConfigurationManager` and the standard 
:file:`app.config` and :file:`web.config` files that were used with .NET Framework and 
ASP.NET applications.

Configuration in |net-core| is based on key-value pairs established by configuration providers.
Configuration providers read configuration data into key-value pairs from a variety of configuration sources,
including command-line arguments, directory files, environment variables, and settings files.

.. note:: For further information, see
   `Configuration in ASP.NET Core <https://docs.microsoft.com/en-us/aspnet/core/fundamentals/configuration/?view=aspnetcore-2.2>`_.

To make it easy to use the |sdk-net| with |net-core|, you can use the 
:code:`AWSSDK.Extensions.NETCore.Setup` NuGet package. Like many |net-core| libraries, it adds 
extension methods to the :code:`IConfiguration` interface to make getting the AWS configuration 
seamless.

.. _net-core-configuration-builder:

Using AWSSDK.Extensions.NETCore.Setup
======================================

When you create an ASP.NET Core MVC application in Visual Studio, the constructor for :file:`Startup.cs` 
handles configuration by reading in various input sources from configuration providers,
such as reading *appsettings.json*.

.. literalinclude:: how-to/net-core/configurationBuilder.cs
   :language: csharp

To use the :code:`Configuration` object to get the AWS options, first add the 
:code:`AWSSDK.Extensions.NETCore.Setup` NuGet package. Then, add your options to the configuration 
file. Notice one of the files added to the :code:`ConfigurationBuilder` is called 
:code:`$"appsettings.{env.EnvironmentName}.json"`. If you look at the Debug tab in your project's 
properties, you can see this file is set to **Development**. This works great for local testing 
because you can put your configuration in the :file:`appsettings.Development.json` file, which is 
read-only during local testing. When you deploy an |EC2| instance that has :code:`EnvironmentName` 
set to **Production**, this file is ignored and the |sdk-net| falls back to the IAM credentials 
and region configured for the |EC2| instance.

The following configuration settings show examples of the values you can add in the 
:file:`appsettings.Development.json` file in your project to supply AWS settings. 

.. literalinclude:: how-to/net-core/appsettings-development.json
   :language: json

To access a setting in an *CSHTML* file,
use the :code:`Configuration` directive:

.. literalinclude:: how-to/net-core/contact.cshtml
   :language: html

To access the AWS options set in the file from code, call the :code:`GetAWSOptions` extension method 
added on :code:`IConfiguration`.

To construct a service client from these options, call 
:code:`CreateServiceClient`. The following example code shows how to create an |S3| service client. 

.. literalinclude:: how-to/net-core/create-s3-client.cs
   :language: csharp

You can also create multiple service clients with incompatible settings using multiple entries in the 
:file:`appsettings.Development.json` file, as shown in the following examples where the configuration for 
:code:`service1` includes the :code:`us-west-2` Region
and the configuration for :code:`service2` includes the special endpoint *URL*.

.. literalinclude:: how-to/net-core/appsettings-multiple.json
   :language: json
      
You can then get the options for a specific service by using the entry in the JSON file.
For example, to get the settings for :code:`service1`:

.. code-block:: csharp

   var options = Configuration.GetAWSOptions("service1");

.. _net-core-appsettings-values:

Allowed Values in appsettings File
----------------------------------

The following app configuration values can be set in the :file:`appsettings.Development.json` file. 
The field names must use the casing shown in the list below. For details on these settings, refer to 
the :sdk-net-api:`AWS.Runtime.ClientConfg <Runtime/TClientConfig>` class.

* Region
* Profile
* ProfilesLocation
* SignatureVersion
* RegionEndpoint
* UseHttp
* ServiceURL
* AuthenticationRegion
* AuthenticationServiceName
* MaxErrorRetry
* LogResponse
* BufferSize
* ProgressUpdateInterval
* ResignRetries
* AllowAutoRedirect
* LogMetrics
* DisableLogging
* UseDualstackEndpoint

   
.. _net-core-dependency-injection:

ASP.NET Core Dependency Injection 
=================================

The *AWSSDK.Extensions.NETCore.Setup* NuGet package also integrates with a new dependency injection 
system in ASP.NET Core. The :code:`ConfigureServices` method in :code:`Startup` is where the MVC 
services are added. If the application is using Entity Framework, this is also where that is 
initialized. 

.. literalinclude:: how-to/net-core/configureServices.cs
   :language: csharp
   
.. note:: Background on dependency injection in |net-core| is available on the |net-core| 
   `documentation site <https://docs.asp.net/en/latest/fundamentals/dependency-injection.html>`_. 

The :code:`AWSSDK.Extensions.NETCore.Setup` NuGet package adds new extension methods to 
:code:`IServiceCollection` that you can use to add AWS services to the dependency injection. The 
following code shows how to add the AWS options that are read from :code:`IConfiguration` to add 
|S3| and |DDB| to our list of services. 

.. literalinclude:: how-to/net-core/configureAwsServices.cs
   :language: csharp
   
Now, if your MVC controllers use either :code:`IAmazonS3` or :code:`IAmazonDynamoDB` as parameters 
in their constructors, the dependency injection system passes in those services. 
   
.. literalinclude:: how-to/net-core/homeController.cs
   :language: csharp
   
   
