# nci-ocpl-api-common

See [Creating an API](docs/creating-api.md) for more details on creating your own OCPL API.

## Background
In building out a number of ASP Net Core microservices over the past year or so, there have been a number of patterns discovered and code repeated over and over. This project will serve as a basis for all ElasticSearch backed microservices we create. The goal someday will be to have a NuGet repository to store these classes.

These APIs are built on [Microsoft's AspNetCore MVC package](https://docs.microsoft.com/en-us/aspnet/core/tutorials/first-web-api?view=aspnetcore-2.0). Some key aspects of the MVC package are:
* Auto-magical Routing and Parameter Mappings for API endpoints - By decorating controller classes with attributes, you can easily:
  * identity a base Route for all methods in the class
  * the method specific paths
  * the HTTP verbs allowed for the methods
  * what parts of the route and/or query parameters that map to the parameters of the method. 
* [Dependency Injection (DI)](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/dependency-injection?view=aspnetcore-2.0) Container - This allows  programmers to create testable "services" which are then injected into the constructors of controllers.

  For example, you have a controller that searches and retrieves definitions from an ElasticSearch instance. Traditionally in order to test that controller's business logic, you would have needed a working ElasticSearch instance and you would have most likely created an instance of the ElasticSearch client in your controller's constructor. With DI you can:
   * Create an interface to represent the service. (e.g. IDefinitionService)
   * Create a concrete instance of the IDefinitionService that connects to ElasticSearch. (e.g. ElasticDefinitionService)
   * Add an IDefinitionService parameter to the controller's Constructor method
   * Register the ElasticDefinitionService with the DI Container so that when an IDefinitionService is needed, an ElasticDefinitionService should be used.
   * In the unit tests for the controller, you can mock a IDefinitionService instance that will return what is needed to ensure the business logic of the controller is correct. No ES server needed.
  One of the neat things with MVC's DI Container is that any service registered with the container can have its required "services" injected injected into the service's constructor as well. So in our above ElasticDefinitionService class the constructor would require an IElasticClient, which is used to communicate with the ES server. This allows the ElasticDefinitionService to have unit tests to ensure that the service behaves as expected. (e.g. it creates the correct queries and returns the right objects when ES responds)
* Self hosting - The service is self-hosted and can run without IIS or any web server. This makes it easier to test.



## Projects
* **NCI.OCPL.Api.Common** - Common objects for making APIs
  * *NciStartupBase.cs* - A common base class to use for your Startup.cs class. Hooks up configuration & Elasticsearch services for you. Also hooks up Swagger & error handling functionality.
  * *Configurations* - The base also provides configuration options for Elasticsearch connectivity and NSwag
* **NCI.OCPL.Api.Common.Testing** - A number of helper classes used in setting up Elasticsearch and network responses in unit tests.

## Latest Versions of NuGet Dependencies
**(as of 10/30/2018)**
Code
* Microsoft.AspNetCore.All (2.0.9) -- Rollup package of all the required NetCore packages to run a AspNet MVC-based API. (There are a lot of dependencies)
* [NEST](https://www.elastic.co/guide/en/elasticsearch/client/net-api/5.x/introduction.html) - 5.6.1 -- This is the Elasticsearch client library. This is pretty much pegged to the version of the service that is being used. NOTE: This is not the easiest of APIs to use as it make copious use of the .NET `dynamic` keyword. If you have never used it before, check some existing code.
* NSwag.AspNetCore - 11.20.1 -- .NET Swagger service that creates our API documentation on the fly 
Testing
* [xUnit.net](https://xunit.github.io/) - 2.3.1 -- Test runner and unit testing framework.  Note, this should NOT be confused with any other libraries named xUnit. (Of which there are a number...) Also note the documentation is thin - so leverage Intellisense, Google and StackOverflow!
* [FluentAssertions](https://fluentassertions.com/) - 5.3.0 -- Testing helper that allows us to maintain sanity. Reduces need for custom comparers and set comparisons.
* [coverlet.msbuild](https://github.com/tonerdo/coverlet) - 1.1.1 -- Code Coverage tool for DotNetCore. 
* OPTIONAL: [RichardSzalay.MockHttp](https://github.com/richardszalay/mockhttp) - 3.3.0 -- Sometimes you need to mock responses from HttpClient requests. You can't exactly, but what you can do is create an HttpMessageHandler and just have it return what you want. MockHttp helps you do just that!

