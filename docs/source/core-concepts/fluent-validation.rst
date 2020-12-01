Fluent Validation
=================

When it comes to validating models, we normally lean towards data annotations. There are quite a lot of serious issues with this approach for a scalable system. 
There is a library, `Fluent Validation`_ that can turn up the validation game to a whole new level, giving you total control without mixing concerns.

In this article, we will talk about `Fluent Validation`_ and it’s implementation in the prototype. We will discuss the preferred alternative to data annotations
and implement it in a sample.

The Problem
-----------

Data validation is extremely vital for any application. The go to approach for model validation in most demos is data annotations, where you have to declare attributes 
over the property of models.

.. code-block:: csharp

    public class Developer
    {
        [Required]
        public string FirstName { get; set; }
        [Required]
        public string LastName { get; set; }
        [EmailAddress]
        public string Email { get; set; }
        [Range(minimum:5,maximum:20)]
        public decimal Experience { get; set; }
    }

It is fine for beginners and demos, but once you start learning *clean code*, or begin to understand the *SOLID* principles of application design, 
you would just never be happy with data annotations as you were before. It is clearly not a good approach to combine your model and validation logic.

With the implementation of data annotations in .NET classes, the problem is that there will be a lot of duplicated lines of code throughout your application. 
What if the developer's model class is to be used in another application/method where this attribute validation changes? 
What if you need to validate a model that you don’t have access to? 
Unit testing can get messy as well. 
You will definitely end up building multiple model classes which will no longer be maintainable.

Introducing Fluent Validation – The Solution
--------------------------------------------

`Fluent Validation`_ is a free to use .NET validation library that helps you make your validation logic clean, easy to create, and maintain. 
It even works on external models that you don’t have access to, with ease. With this library, you can separate the model classes from the validation 
logic like it is supposed to be. Also, better control of validation is something that makes a developer prefer Fluent Validation.

`Fluent Validation`_ uses lamba expressions to build validation rules.

.. _`Fluent Validation`: https://docs.fluentvalidation.net/en/latest/

Fluent Validations with MediatR
-------------------------------

For validating our MediatR requests, we will use the Fluent Validation library.
Let use the following use case as an example: 

.. admonition:: Use Case

   We have an API endpoint that is responsible for creating a product in the database from 
   the request object that includes product name, prices, barcode and so on. 
   But we would want to validate this request in the pipeline itself.

Here are the MediatR Request and Handler

.. code-block:: csharp

    public class CreateProductCommand : IRequest<int>
    {
        public string Name { get; set; }
        public string Barcode { get; set; }
        public string Description { get; set; }
        public decimal BuyingPrice { get; set; }
        public decimal Rate { get; set; }

        public class CreateProductCommandHandler : IRequestHandler<CreateProductCommand, int>
        {
            private readonly IApplicationContext _context;
            public CreateProductCommandHandler(IApplicationContext context)
            {
                _context = context;
            }
            public async Task<int> Handle(CreateProductCommand command, CancellationToken cancellationToken)
            {
                var product = new Product();
                product.Barcode = command.Barcode;
                product.Name = command.Name;
                product.BuyingPrice = command.BuyingPrice;
                product.Rate = command.Rate;
                product.Description = command.Description;
                _context.Products.Add(product);
                await _context.SaveChangesAsync();
                return product.Id;
            }
        }
    }

Add a new folder to the root of our application and name it *Validators*. Here is where we are going to add all the validators related to the domain. 
Since we are going to validate the **CreateProductCommand** object, let’s follow convention and name our validator **CreateProductCommandValidator**. 
So add a new file to the validators folder named CreateProductCommandValidator.

.. code-block:: csharp
    public class CreateProductCommndValidator : AbstractValidator<CreateProductCommand>
    {
        public CreateProductCommndValidator()
        {
            RuleFor(c => c.Barcode).NotEmpty();
            RuleFor(c => c.Name).NotEmpty();
        }
    }

We will keep things simple for this article. We create 2 rules,  checking if the Name and Barcode numbers are not empty. 
You could take this a step further by injecting a DbContext to this constructor and chack if the barcode already exists. 

We have n number of similar validators for each command and query. This helps keep the code well organized and easy to test.

Before continuing, let’s register this validator with the DI container. Navigate to Startup.cs ConfigureServices method and add in the following line.

.. code-block:: csharp
   services.AddValidatorsFromAssembly(typeof(Startup).Assembly);

This essentially registers all the validators that are available within the current assembly.

Now that we have our validator set up, let’s add it to the pipeline behaviour. Create another new folder in the root of the application and name it 
PipelineBehaviours. Here, add a new class, ValidationBehaviour.cs.

.. code-block:: csharp
    public class ValidationBehaviour<TRequest, TResponse> : IPipelineBehavior<TRequest, TResponse> where TRequest : IRequest<TResponse>
    {
        private readonly IEnumerable<IValidator<TRequest>> _validators;

        public ValidationBehaviour(IEnumerable<IValidator<TRequest>> validators)
        {
            _validators = validators;
        }

        public async Task<TResponse> Handle(TRequest request, CancellationToken cancellationToken, RequestHandlerDelegate<TResponse> next)
        {
            if (!_validators.Any())
                return await next();

            var context = new ValidationContext<TRequest>(request);
            var validationResults = await Task.WhenAll(_validators.Select(v => v.ValidateAsync(context, cancellationToken)));
            var failures = validationResults.SelectMany(r => r.Errors).Where(f => f != null).ToList();
            if (failures.Count != 0)            
                throw new FluentValidation.ValidationException(failures);
        }
    }

We have one last thing to do. Register this pipeline behaviour in the DI container. Again, go back to Startup.cs ConfigureServices and add this.

.. code-block:: csharp
   services.AddTransient(typeof(IPipelineBehavior<,>), typeof(ValidationBehaviour<,>));

Since we need to validate each and every request, we add it with a **Transient** scope.

That’s it, quite simple to setup.


