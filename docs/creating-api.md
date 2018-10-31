# Creating a new API

## YOUR Project Structure
When creating the API you must define two names:
1. The short name used for repo names and other files (e.g. *R4R*). Hereforth known as **\<SHORT_NAME\>**
2. A longer, more descriptive name for projects and namespaces (e.g. ) Hereforth known as **\<LONG_NAME\>**

The structure should adhear to the structure below. *NOTE:* Not every single file or folder is listed here. As this is a normal .NET application you will have other supporting files, such as CS Project files.
* **\<SHORT_NAME\>-api** - The repo root
  * **src** - Where the deployable source is placed
    * **NCI.OCPL.Api.\<LONG_NAME\>** - The API's code
      * **Controllers** - The controllers for the project
      * **Models** - The *Plain old CLR Objects (POCO)* classes that represents the API's input and output data, and any other *Dependency Injection (DI)*  Service data. 
      * **Services** - The interfaces and concrete implementations of DI services required by your API. For example, you **WILL** have some sort of service that maps the API request to the ElasticSearch servers. This should contain both the service interfaces and the concrete implementations for the API. There should *NOT* be any test implementations.
      * **Program.cs** - The main class and entrypoint for the API. Its job is to create an instance of the API.
      * **Startup.cs** - The class that initializes the services and configures the API application. This is what Program.cs tells the hosting engine to use to get the setup for the API.
  * **test** - Where the unit test projects go. 
    * **NCI.OCPL.Api.\<LONG_NAME\>.Tests** - The main API test project
      * **TestData** - This is where static data for the tests should be placed. Examples of this include JSON or other types of responses to be mocked
      * **TestDataObjects** - This is where reusable complex objects should be placed. Also test helper objects can be placed here as well. An example of a test helper object is a class that has some sort of path to a JSON response AND an instance of the expected result. Then in a [Theory](https://xunit.github.io/docs/getting-started-dotnet-core#write-first-theory) you can keep the code readable by referencing the classes in your [MemberData](https://andrewlock.net/creating-strongly-typed-xunit-theory-test-data-with-theorydata/).
      * **Tests** - This is where the unit tests should go 
        * **Controllers** - Unit tests for the controllers
        * **Models** - Unit tests for the models; probably more so for anything that serializes or deserializes
        * **Services** - Unit tests for your services
      * **Util** - Any helpers     
  * **integration-tests** - Folder holding all the API integration/behavioral tests
    * **\<SHORT_NAME\>-integration-tests** - Our API testing framework is [Karate](https://en.wikipedia.org/wiki/Karate_(software)), which is an web-API automation framework created by Intuit, and is based on [Cucumber](https://en.wikipedia.org/wiki/Cucumber_(software)) and Java. As such this will be a Maven project. More details can be found at [Karate Integration Tests](INTEGRATION_TESTS.md).
  * **tools** - CI/CD Tooling
    * **integration-test-docker** - Docker compose stack for running your API, Elasticsearch, other services, somesort of data loading for ES, and the integration tests. Basically it should be able to stand up a complete cluster for your App. 
    NOTE: 
  * **\<SHORT_NAME\>.sln** - The solution file



## Steps to create an API 

### 1. Create the repository
1. Create a Git repo for the solution (with a README.md file please)
   * Name it \<SHORT_NAME\>-api
1. Clone the solution locally

### 2. Create the project structure
1. Create an empty solution in Visual Studio such that the .sln file is:
    * named the same thing as the repo (e.g. \<SHORT_NAME\>-api.sln)
    * is in the root of the repo (e.g. \<SHORT_NAME\>-api/\<SHORT_NAME\>-api.sln)
1. Create the Source project
    1. Add a new *ASP.NET Core Web API* project:
        * The name should be NCI.OCPL.Api.\<LONG_NAME\> 
        * The parent folder should be <SHORT_NAME>-api/src
    1. Ensure the default namespace and assembly name are NCI.OCPL.Api.\<LONG_NAME> (They should be because you named them that way)
1. Create the Unit Tests project
    1. Add a new *xUnit Test Project* project:
        * The name should be NCI.OCPL.Api.\<LONG_NAME\>.Tests 
        * The parent folder should be <SHORT_NAME>-api/test
    1. Ensure the default namespace and assembly name are NCI.OCPL.Api.\<LONG_NAME>.Tests (They should be because you named them that way)
1. Make a copy of the *NCI.OCPL.Api.Common* and *NCI.OCPL.Api.Common.Testing* projects from this repository into the \<SHORT_NAME\>/src folder.
1. Include the two Common projects into the solution

### 3. Setup the API source project
All tasks within this section should be done to the src project, i.e. NCI.OCPL.Api.\<LONG_NAME\>
1. Add a project reference to *NCI.OCPL.Api.Common*
1. Remove the NuGet depenency to Microsoft.AspNetCore.All
   * I experienced an issue with restoring the packages when there was a dependency in both. The conflic was that the main project was trying to restore 2.0.0 and the Common was using 2.0.9.  2.0.9 was the dependency in all the projects, so 2.0.0 should not have been used at all. 
   * Common comes with the following depencencies:
     * *Microsoft.AspNetCore.All*
     * *NEST*
     * *NSwag.AspNetCore*
1. Add **good** XML comments to Program.cs if they do not exist. (Visual Studio for Mac did not add any comments!) This is the class which hosts the service.
1. Add API Configuration
   1. Create a folder named *Models*
   1. Under *Models* create a folder named *Options*
      * This is where configuration models go
   1. Add a file named *\<SHORT_NAME\>APIOptions* with the code below, replacing *\<SHORT_NAME\>* and *\<LONG_NAME\>* as needed.
      * This will map to a *\<SHORT_NAME\>API* property in the *appSettings.json* file
```C#
namespace NCI.OCPL.Api.<LONG_NAME>.Models
{
    /// <summary>
    /// Configuration options for the <LONG_NAME> API
    /// </summary>
    public class <SHORT_NAME>APIOptions
    {
        //INSERT CONFIGURATION OPTIONS HERE
    }
}
```
1. Replace the contents of the Startup class in Startup.cs with the following
```C#
using NCI.OCPL.Api.Common;

namespace NCI.OCPL.Api.<LONG_NAME>
{
    /// <summary>
    /// Defines the configuration for the Best Bets API
    /// </summary>
    public class Startup : NciStartupBase
    {
        /// <summary>
        /// Initializes a new instance of the <see cref="T:NCI.OCPL.Api.<LONG_NAME>.Startup"/> class.
        /// </summary>
        /// <param name="env">Env.</param>
        public Startup(IHostingEnvironment env)
            : base(env) { }


        /*****************************
         * ConfigureServices methods *
         *****************************/

        /// <summary>
        /// Adds the configuration mappings.
        /// </summary>
        /// <param name="services">Services.</param>
        protected override void AddAdditionalConfigurationMappings(IServiceCollection services)
        {            
            services.Configure<<SHORT_NAME>APIOption>(Configuration.GetSection("<SHORT_NAME>API"));
        }

        /// <summary>
        /// Adds in the application specific services
        /// </summary>
        /// <param name="services">Services.</param>
        protected override void AddAppServices(IServiceCollection services)
        {
            //Add your services here
            services.AddTransient<YOUR_SERVICE_INTERFACE, YOUR_SERVICE CLASS>();
        }

        /*****************************
         *     Configure methods     *
         *****************************/

        /// <summary>
        /// Configure the specified app and env.
        /// </summary>
        /// <returns>The configure.</returns>
        /// <param name="app">App.</param>
        /// <param name="env">Env.</param>
        /// <param name="loggerFactory">Logger.</param>
        // This method gets called by the runtime. Use this method to configure the HTTP request pipeline.
        protected override void ConfigureAppSpecific(IApplicationBuilder app, IHostingEnvironment env, ILoggerFactory loggerFactory)
        {
            return;
        }
    }
}
```

### 3. Setup the unit tests project
1. Add a project reference to *NCI.OCPL.Api.Common.Testing*
   * This should bring in the references for:
     * xUnit.net
     * Coverlet
     * FluentAssertions
1. Add a project reference to *NCI.OCPL.Api.\<SHORT_NAME\>*
1. Make some tests

### 4. Setup integration tests
See [Karate Integration Tests : Creating a test project](integration-tests.md#Creating-a-test-project).

### 5. Setup integration test cluster
TBD
