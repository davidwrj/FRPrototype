Sample Onion Implementation
===========================

We will build a WebApi that follows a variant of **Onion Architecture** so that we get to see why it is important to implement 
such an architecture.   You can find the source code of this implementation `somewhere yet to be defined`_.

.. _`somewhere yet to be defined`: https://github.com/Allann/FreightRate

Another more fully implemented demonstration can be found at `Onion-DevOps-Architecture`_ which is written and maintained by Jeff.

.. _`Onion-DevOps-Architecture`: https://dev.azure.com/clearmeasurelabs/Onion-DevOps-Architecture

Implementing Onion Architecture in ASP.NET Core WebApi Project
--------------------------------------------------------------

To keep things simple but demonstrate the architecture to the fullest, we will build an ASP.NET Core Web API that is quite scalable. For this article, Let’s have a WebApi that has just one entity, Product. We will perform CRUD Operations on it while using the Onion architecture. This will give you quite a clear picture.

Here is a list of features and tech we will be using for this setup.

 * Onion Architecture
 * Entity Framework Core
 * .NET Core 3.1 Library / .NET Standard 2.1 Library / ASP.NET Core 3.1 WebApi
 * Swagger
 * CQRS / Mediator Pattern using MediatR Library
 * Wrapper Class for Responses
 * CRUD Operations
 * Inverted Dependencies
 * API Versioning

Setting up the Solution Structure
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

We will start off by creating a Blank Solution on Visual Studio.

.. image:: /_static/images/blank.png
   :width: 50%

Let’s give it a proper Name.

.. image:: /_static/images/newproject.png
   :width: 50%

Under the Blank Solution, add 3 new folders.

 * Core – will contain the Domain and Application layer Projects
 * Infrastructure – will include any projects related to the Infrastructure of the ASP.NET Core 3.1 Web Api (Authentication, Persistence etc)
 * Presentation – The Projects that are linked to the UI or API . In our case, this folder will hold the API Project.

.. image:: /_static/images/blanksolution-1.png
   :width: 50%

Let’s start adding the required projects. Firstly, under Core Folder Add a new .NET Standard Library and name it Domain.

Why .NET Standard? We know that Domain and Application Project does not depend on any other layers. Also the fact that these projects can be shared with other solutions if needed (Maybe another solution that is not .NET Core, but .NET Framework 4.7) . Get the point?

.. image:: /_static/images/domain-1024x485.png
   :width: 50%

.. note:: 

   A wise person once said – "Delete the Default Class1 Created by Visual Studio. Always Delete them."

After creating the Domain project, right click on properties and change the target framework to .NET Standard 2.1 (which is the latest .NET Standard version at the time of writing this article.)

.. image:: /_static/images/domainproperties-1024x476.png
   :width: 50%

Similary, create another .NET Standard Library Project in the Core Folder. Name it Application. Do not forget to change the target version here as well.

Next, let’s go to the Infrastructure Folder and add a layer for Database, (EFCore). This is going to be a .NET Core Library Project. We will name it Persistence.

.. image:: /_static/images/persistence-1024x531.png
   :width: 50%

Finally, in the Presentation layer, add a new ASP.NET Core 3.1 WebApi Project and name it WebApi.

.. image:: /_static/images/api.png
   :width: 50%

This is what we will be having right now. You can see the clear seperation of concerns as we have read earlier. Let’s start build up the architecture now.

.. image:: /_static/images/finalstrcuture.png
   :width: 50%

Adding Swagger To WebApi Project
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. admonition:: Tip #1 

   Always use Swagger while working with WebApis. It is so much helpful to have it.

Install the Following packages ot the WebApi Project via Package Manager Console

.. code-block:: rst

   Install-Package Swashbuckle.AspNetCore
   Install-Package Swashbuckle.AspNetCore.Swagger

We will have to register Swager within the application service container. Navigate to ../Startup.cs and add these lines to the ConfigureServices method.

.. code-block:: csharp

   #region Swagger
   services.AddSwaggerGen(c =>
   {
       c.IncludeXmlComments(string.Format(@"{0}\OnionArchitecture.xml", System.AppDomain.CurrentDomain.BaseDirectory));
       c.SwaggerDoc("v1", new OpenApiInfo
       {
           Version = "v1",
           Title = "OnionArchitecture",
       });
   });
   #endregion

Then, add these lines to the Configure method.

.. code-block:: csharp

   #region Swagger
   // Enable middleware to serve generated Swagger as a JSON endpoint.
   app.UseSwagger();

   // Enable middleware to serve swagger-ui (HTML, JS, CSS, etc.),
   // specifying the Swagger JSON endpoint.
   app.UseSwaggerUI(c =>
   {
       c.SwaggerEndpoint("/swagger/v1/swagger.json", "OnionArchitecture");
   });
   #endregion

Next, we will need to add the XML File (For Swagger Documentaion). To do this, right click the WebApi Project and go to propeties. In the Build Tab enable the XML Documentation file and give an appropriate file name and location. I have added the xml file to the root of the API Project.

.. image:: /_static/images/xml-1024x476.png
   :width: 50%

Make sure that the WebApi Project is selected as the Startup Project. Now Build / Run the Application and navigate to ../swagger. We have got swagger up and running.

.. image:: /_static/images/swagger-980x479.png
   :width: 50%

.. admonition:: Tip #2 

   While running the application, you would see that it navigated to ../weatherforecast by default. This is because of launchSettings.json settings. In the WebApi Project, Properties drill down, you can find a launchsettings.json file. This file holds all the configuration required for the app launch. Change the launch URL to swagger. Thus, swagger will open up by default every time you run the application. This helps you save some time.

.. image:: /_static/images/launch.png
   :width: 50%

Adding The Entities to the Domain Project
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Now, let’s work on the Core Layers starting from the Domain Project. So what is the function of the Domain Layer? It basically has the models/entities, Exception, validation rules, Settings, and anything that is quite common throughout the solution.

Let’s start by adding a BaseEntity class at Common/BaseEntity.cs in the Domain Project. This abstract class will be used as a base class for our entities.

.. code-block:: csharp

   public abstract class BaseEntity
   {
       public int Id { get; set; }
   }

Now add a Product Class that inherits the Id from the BaseEntity. Create a new class Entities/Product.cs in the Domain Project.

.. code-block:: csharp

   public class Product : BaseEntity
   {
       public string Name { get; set; }
       public string Barcode { get; set; }
       public string Description { get; set; }
       public decimal Rate { get; set; }
   }

Adding the Required Interfaces And Packages in Application Layer
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

As mentioned earlier, the Application Layer will contain the Interfaces and Types that are specific for this Application.

Firstly, Add Reference to the Domain Project.

Then, install the required packages via Console.

.. code-block:: rst

   Install-Package MediatR.Extensions.Microsoft.DependencyInjection
   Install-Package Microsoft.EntityFrameworkCore

We have a Entity named Product. Now we need to establish this class as a Table using Entity Framework Core. So we will need a ApplicationDBContext. But the catch is that, we won’t create the actual concrete implementation of the ApplicationDbContext here in the Application Layer. Rather, we will just add a IApplicatoinDbContext Interface so that the EF Logics does not fall under the Application Layer, but goes to the Persistence layer which is outside the core,

This is how you can invert the dependencies to build scalable applications. Now , the advantage is that, tommorow, you need a different implementation of the ApplicationDbContext, you don’t need to touch the existing code base, but just add another Infrastructure layer for this purpose and implement the IApplicationDbContext. As simple as that.

Create a new folder Interfaces in the Application Project. Add a new interface in it, IApplicationDbContext

.. code-block:: csharp

   public interface IApplicationDbContext
   {
       DbSet<Product> Products { get; set; }
       Task<int> SaveChanges();
   }

This is another variant that i have noticed in many huge solutions. Let’s say you have around 100 interfaces and 100 implementations. Do you add all this 100 lines of code to the Startup.cs to register them in the container? That would be insane in the maintainability point of view. To keep things clean, what we can do is, Create a DependencyInjection static Class for every layer of the solution and only add the corresponding . required services to the corresponding Class.

In this way, we are decentralizing the code lines and keeping our Startup class neat and tidy. Here is an extension method over the IServiceCollection.

.. code-block:: csharp

   public static class DependencyInjection
   {
       public static void AddApplication(this IServiceCollection services)
       {
           services.AddMediatR(Assembly.GetExecutingAssembly());
       }
   }

Here we will just Add Mediator to the service collection. We will implement Mediator pattern later in this tutorial.

And all you have to do in the WebApi’s Startup class in just add one line. This essentially registers all the services associated with the Application Layer into the container. Quite handy, yeah?

.. code-block:: csharp

   services.AddApplication();

Implementing MediatR for CRUD Operations
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

In Application Layer, Create a New Folder called Features. This will have all the logics related to each Feature / Entity. Under this folder, add a new one and name it ProductFeatures. Then add a Commands and Queries folder to it.

I have already written a detailed article on MediatR and CQRS pattern in ASP.NET Core 3.1 WebApi Project. You can follow that article and add the Required Commands and Handlers to the Application Layer.

.. image:: /_static/images/cqrs.png
   :width: 50%

I will add the links to the source code of each file. Basically these 5 Classes would cover our CRUD Operations implementation. Make sure that you have gone through my article about CQRS for ASP.NET Core before proceeding.

 * CreateCommand
 * DeleteCommand
 * UpdateCommand
 * GetAllQuery
 * GetByIdQuery

Setting Up EF Core on the Persistence Project
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Firstly, add a connection string to the appsettings.json found in the WebApi Project.

.. code-block:: json

   {
      "ConnectionStrings": 
      {
         "DefaultConnection": "Server=(localdb)\\mssqllocaldb;Database=onionDb;Trusted_Connection=True;MultipleActiveResultSets=true"
      }
   }

With the CRUD logics out of the ways, let’s setup EFCore in the Persistence Layer and try to generate a database. Install the following packages to the Persistence Project.

.. code-block:: rst

   Install-Package Microsoft.EntityFrameworkCore
   Install-Package Microsoft.EntityFrameworkCore.SqlServer

Remember we created an IApplicationDBContext Interface in the Application Layer? This is where we will be implementing it. Create a new folder named Context and add a new class ApplicationDbContext. This class will implement IApplicationDBContext.

.. code-block:: csharp

   public class ApplicationDbContext : DbContext, IApplicationDbContext
   {
       public ApplicationDbContext(DbContextOptions<ApplicationDbContext> options)
           : base(options)
       {
       }
       public DbSet<Product> Products { get; set; }
       public async Task<int> SaveChanges()
       {
           return await base.SaveChangesAsync();
       }
   }

We will have to register IApplicationDBContext and bind it to ApplicationDbContext, right? Similar to the Application layer, we will have to create a new class just to register the dependencies and services of this layer to the service container.

Add a new static class, DependencyInjection

.. code-block:: csharp

   public static class DependencyInjection
   {
       public static void AddPersistence(this IServiceCollection services, IConfiguration configuration)
       {
           services.AddDbContext<ApplicationDbContext>(options =>
               options.UseSqlServer(
                   configuration.GetConnectionString("DefaultConnection"),
                   b => b.MigrationsAssembly(typeof(ApplicationDbContext).Assembly.FullName)));
           services.AddScoped<IApplicationDbContext>(provider => provider.GetService<ApplicationDbContext>());
       }
   }

And in the Startup class/ ConfigureServices method of the WebApi Just Add the following line. You can now see the advantage of this kind of approach.

.. code-block:: csharp

   services.AddPersistence(Configuration);

Generate the Migrations and the Database
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

As our ApplicationDbContext is configured, let’s generate the migrations and ultimately create a Database using Ef Core Tools – Code First Approach.

Install the following packages in the WebApi Project.

.. code-block:: rst

   Install-Package Microsoft.EntityFrameworkCore.Tools
   Install-Package Microsoft.EntityFrameworkCore.Design

Now, open up the package manager console and select the Persistence project as the default prject (as mentioned in the sceenshot below.). This is because the actual ApplicationDBContext is implemented in the Persistence layer, remember?

Then, run the following commands to add migrations and to generate / update the database.

.. code-block:: rst

   add-migration Initial
   update-database

.. image:: /_static/images/pmc.png
   :width: 50%

You will get a ‘Done’ message.

Adding API Versioning
^^^^^^^^^^^^^^^^^^^^^

Just to make our solution a bit more clean, let’s also add API Versioning to the WebAPI.

I have written a detailed article on API Versioning in ASP.NET Core 3.1 WebApi. Feel feel to read it to get a complete idea of this concept.

Install the required package.

.. code-block:: rst

    Install-Package Microsoft.AspNetCore.Mvc.Versioning

In the Startup/ConfigureServices of the API project, add these lines to register the Versioning.

.. code-block:: csharp

    #region API Versioning
    // Add API Versioning to the Project
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

Setting up the Controllers
^^^^^^^^^^^^^^^^^^^^^^^^^^

This is the final step of setting up Onion Architecture In ASP.NET Core. We will have to wire up a controller to the Application Layer.

Create a Base Api Controller. This will be an Empty API Controller which will have Api Versioning enabled in the Attribute and also a MediatR object. What is aim of this Base Controller? It is just to reduce the lines of code. Say, we add a new controller. We will not have to re-define the API Versioning route nor the Mediatr object. But we will just add the BaseAPI Controller as the base class. Get it? I will show it in implementation.

Add new Empty API Controller in the Controllers folder and name it BaseApiController.

.. code-block:: csharp

    using MediatR;
    using Microsoft.AspNetCore.Http;
    using Microsoft.AspNetCore.Mvc;
    using Microsoft.Extensions.DependencyInjection;
    namespace WebApi.Controllers
    {
        [ApiController]
        [Route("api/v{version:apiVersion}/[controller]")]
        public abstract class BaseApiController : ControllerBase
        {
            private IMediator _mediator;
            protected IMediator Mediator => _mediator ??= HttpContext.RequestServices.GetService<IMediator>();
        }
    }

You can see that we are adding the API Versioning data to the route attribute and also creating a IMediator object.

Next, let’s create our actual ENtity endpoint. Create a new folder inside the Controllers folder and name it ‘v1’. This means that this folder will contain all the Version 1 API Controllers. Read more about API Versioning to understand the need for this here.

Inside the v1 Folder, add a new empty API Controller named ProductController. Since this is a very basic controller that calls the mediator object, I will not go in deep. However, I have previously written a detailed article on CQRS implementation in ASP.NET Core 3.1 API. You could go through that article which covers the same scenario. Read it here.

.. code-block:: csharp

    [ApiVersion("1.0")]
    public class ProductController : BaseApiController
    {
        /// <summary>
        /// Creates a New Product.
        /// </summary>
        /// <param name="command"></param>
        /// <returns></returns>
        [HttpPost]
        public async Task<IActionResult> Create(CreateProductCommand command)
        {
            return Ok(await Mediator.Send(command));
        }
        /// <summary>
        /// Gets all Products.
        /// </summary>
        /// <returns></returns>
        [HttpGet]
        public async Task<IActionResult> GetAll()
        {
            return Ok(await Mediator.Send(new GetAllProductsQuery()));
        }
        /// <summary>
        /// Gets Product Entity by Id.
        /// </summary>
        /// <param name="id"></param>
        /// <returns></returns>
        [HttpGet("{id}")]
        public async Task<IActionResult> GetById(int id)
        {
            return Ok(await Mediator.Send(new GetProductByIdQuery { Id = id }));
        }
        /// <summary>
        /// Deletes Product Entity based on Id.
        /// </summary>
        /// <param name="id"></param>
        /// <returns></returns>
        [HttpDelete("{id}")]
        public async Task<IActionResult> Delete(int id)
        {
            return Ok(await Mediator.Send(new DeleteProductByIdCommand { Id = id }));
        }
        /// <summary>
        /// Updates the Product Entity based on Id.   
        /// </summary>
        /// <param name="id"></param>
        /// <param name="command"></param>
        /// <returns></returns>
        [HttpPut("[action]")]
        public async Task<IActionResult> Update(int id, UpdateProductCommand command)
        {
            if (id != command.Id)
            {
                return BadRequest();
            }
            return Ok(await Mediator.Send(command));
        }
    }

That’s quite everything in this simple yet powerful implementation of Onion Architecture in ASP.NET Core. Build the application and let’s test it.

Since we are already talking about a form of Clean Architecture in ASP.NET Core Applications, it would help if you read about certain tips to write clean and scalable C# Code. This knowledge will drastically improve the way you start building applications in .NET – Read the article here (20 Tips to write Clean C# Code)

Testing
-------

Run the application and open up Swagger. We will do a simple test to ensure that our solution works. I will just create a new product and make a request to query all the existing products as well.

.. image:: /_static/images/create-new-product-980x489.png
   :width: 50%

.. image:: /_static/images/get-all-980x489.png
   :width: 50%

You can see that we receive the expected data.

