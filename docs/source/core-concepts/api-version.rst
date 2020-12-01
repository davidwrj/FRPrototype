API Versioning
==============

What is API Versioning? (and why do you need it?)
-------------------------------------------------

Before deploying an API, there is a checklist of a few features that are considered vital. API versioning tops that list. 
It is highly crucial to anticipate changes that may be required once the API is published and a few clients are already using it. 
After publishing the API to a production server, we have to be careful with any future change. These changes should not break 
the existing client applications that are already using our API. It is also not a good idea to go on and change the API calls 
in each and every client application. This is how the concept of API versioning came about.

API versioning is a technique by which different clients can get different implementations of the same controller based on the request or the URL.
So essentially, you build an API that has multiple versions that may behave differently. This is a great approach to accommodate the fact that 
requirements can change at any given time, and we do not have to compromise the integrity and availability of data for our 
already existing client applications.

Different Ways to Implement API versioning
------------------------------------------

There are multiple ways to achieve API versioning in applications. The commonly used approach to version a WebApi are as follows:

 * Query String based.
 * URL based.
 * HTTP Header based.

.. note:: 

   There are other ways like media type, accept-header, that can be quite complex, from a practical point of view, 
   I believe these 3 are the go-to approaches while versioning an API.

Getting Started
---------------

Installing the Package
^^^^^^^^^^^^^^^^^^^^^^

Microsoft has it’s own package to facilitate the process of versioning .NET Core APIs. This Package is called **Microsoft.AspNetCore.Mvc.Versioning**. 

.. code-block:: rst

   Install-Package Microsoft.AspNetCore.Mvc.Versioning

Configuring the application
^^^^^^^^^^^^^^^^^^^^^^^^^^^

Navigate to the Startup class of the API Project and modify the Configure Services method to support versioning.

.. code-block:: csharp

    public void ConfigureServices(IServiceCollection services)
    {
        #region Api Versioning

        services.AddApiVersioning(config =>
        {
            // Specify the default API Version as 1.0
            config.DefaultApiVersion = new ApiVersion(1, 0);
            // If the client hasn't specified the API version in the request, use the default API version number 
            config.AssumeDefaultVersionWhenUnspecified = true;
            // Advertise the API versions supported for the particular endpoint
            config.ReportApiVersions = true;
        });
        #endregion

        services.AddControllers();
    }

URL Based API Versioning
------------------------

Personally, this is my favorite approach and I implement this to nearly all of the APIs that I work on. It gives a clean separation between different versions. 
Versions are explicitly mentioned in the URL of the API endpoints. Here is how it would look like.

https://secureddata.com/api/v1/users
https://secureddata.com/api/v2/users

Implementation
^^^^^^^^^^^^^^

Here is the code for **v1/DataController**

.. code-block:: csharp

    namespace Versioning.WebApi.Controllers.v1
    {
        [ApiVersion("1.0")]
        [Route("api/v{version:apiVersion}/[controller]")]
        [ApiController]
        public class DataController : ControllerBase
        {
            [HttpGet]
            public string Get()
            {
                return "data from api v1";
            }
        }
    }

And here goes the **v2/DataController**

.. code-block:: csharp

    namespace Versioning.WebApi.Controllers.v2
    {
        [ApiVersion("2.0")]
        [Route("api/v{version:apiVersion}/[controller]")]
        [ApiController]
        public class DataController : ControllerBase
        {
            [HttpGet]
            public string Get()
            {
                return "data from api v2";
            }
        }
    }

.. image:: /_static/images/apidatafromv1.png
   :width: 50%

.. image:: /_static/images/apidatafromv2.png
   :width: 50%

If you notice the difference in the addresses, we have already implemented the API versioning. URL based versioning is more conventional as the 
clients and fellow developers are able to make out the version by seeing the URL itself. It is easier to develop APIs using this approach. 
We will move forward to Query Based Versioning on the same application.

Query Based API Versioning
--------------------------

While the previous approach has the version within the URL, this technique allows you to pass the version of the API as a parameter in the URL, 
i.e. a query string. Here is how the URL may look like.

https://secureddata.com/api/users?api-version=1
https://secureddata.com/api/users?api-version=2

Implementation
^^^^^^^^^^^^^^

There is not much of a difference from the previous implementation in terms of the code. Just change each of the controllers to [Route(“api/[controller]”)] 
instead of [Route(“api/v{version:apiVersion}/[controller]”)]. 

.. image:: /_static/images/apidatafromv1query.png
   :width: 50%

.. image:: /_static/images/apidatafromv2query.png
   :width: 50%

When you pass a api version that does not exist you will receive an error

.. image:: /_static/images/apiversionnotsupported-980x146.png
   :width: 50%

It throws an **UnsupportedAPIVersion** exception saying that there is not a resource that matches API Version 999. 

HTTP Header Based API Versioning
--------------------------------

This approach is slightly different compared to the previous implementations and to test this, we need Postman or Swagger. 
I will use Postman in this demonstration to test the implementation. In the previous 2 techniques, we had the Version number 
either within the URL or passed it to the application via a query string. Here we will pass the API version as HTTP Header in 
our request. This is exactly why it cannot be demonstrated using just a browser.

Implementation
^^^^^^^^^^^^^^

For this approach, we will have to modify our Startup class to support the reading of API version from HTTP Header. 

.. code-block:: csharp

    public void ConfigureServices(IServiceCollection services)
    {
        #region Api Versioning
        // Add API Versioning to the Project
        services.AddApiVersioning(config =>
        {
            // Specify the default API Version as 1.0
            config.DefaultApiVersion = new ApiVersion(1, 0);
            // If the client hasn't specified the API version in the request, use the default API version number 
            config.AssumeDefaultVersionWhenUnspecified = true;
            // Advertise the API versions supported for the particular endpoint
            config.ReportApiVersions = true;
            //HTTP Header based versioning
            config.ApiVersionReader = new HeaderApiVersionReader("x-api-version");
        });
        #endregion
        services.AddControllers();
    }

Run the API and open up Postman. Paste the URL as below and add a Header Key , “x-api-version” with the required API Version. Here are the results.

.. image:: /_static/images/apidatafromv1http.png
   :width: 50%

.. image:: /_static/images/apidatafromv2http.png
   :width: 50%

This usage of versioning can be a bit challenging for Clients and may increase the lines of codes in the Client App.

Supporting Multiple API Versioning Schemes
------------------------------------------

It is possible to apply all these approaches to your API out of the box by making these simple changes and let the client choose what he/she will use.

Firstly, add the route that supports URL based routing. Make sure each of your API Controllers has the following attribute.

.. code-block:: csharp

   [Route(“api/[controller]”)]
   [Route(“api/v{version:apiVersion}/[controller]”)]

Next, We will have to combine the HTTP and Query based versioning. For this go back to the ConfigService of the Startup class and modify our last line of code in the ApIVersioning extension.

.. code-block:: csharp

   config.ApiVersionReader = ApiVersionReader.Combine(new HeaderApiVersionReader("x-api-version"), new QueryStringApiVersionReader("api-version"));

Now you will be able to use all the 3 API versioning Schemes seamlessly.

Deprecating an API Version
--------------------------

Advertising the supported API versions of an endpoint, the versions that are going to be removed in the near future can also be advertised on the HTTP response Header. 
This can be done by modifying the Route attribute of the specific version of the API Controller.

.. code-block:: csharp

   // DEPRECATING an API Version
   [ApiVersion("1.0", Deprecated = true)]
