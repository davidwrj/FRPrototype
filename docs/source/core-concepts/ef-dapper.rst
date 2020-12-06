Entity Framework Core and Dapper
================================

The describes using Entity Framework Core and Dapper together in the same application. 
Another major point of discussion will be transactions. By the end of this, we will have an application that works with both 
Entity Framework Core and Dapper alongside each other, but also intelligent enough to rollback data whenever there is an exception with the process.

We will also take a look at `ThrowR`_, a simple library for .NET Core that can make your code clean and readable by eliminating the use of *if* statements unnecessarily.

.. _`ThrowR`: https://www.nuget.org/packages/CWM.DotNetCore.ValidatR/

Dapper
------

Dapper is a simple Object Mapping Framework or Micro-ORM that helps us to map the data from the result of an SQL Query to a .NET Class efficiently. 
It would be as simple as executing a SQL Select statement using the SQL Client object and returning the result as a mapped C# class. 
It’s more like AutoMapper for the SQL world. This powerful ORM was build by the folks at StackOverflow and is definitely faster at querying 
data when compared to the performance of Entity Framework. This is possible because Dapper works directly with the RAW SQL and hence the 
time-delay is quite less. This boosts the performance of Dapper.

Dapper vs Entity Framework Core
-------------------------------

Dapper is literally much faster than Entity Framework Core considering the fact that there are no bells and whistles in Dapper. 
It is a straight forward Micro ORM that has minimal features as well. It is always up to the developer to choose between these 2 data access technologies. 
This does not mean that Entity Framework Core is any slower. With every update, the performance seems to be improving as well. 
Dapper is heaven for those who still like to work with raw queries rather than LINQ with EF Core.

Now, Entity Framework Core has tons of features included along with performance improvements as well. So the question is, why choose between 
Dapper and Entity Framework Core when you can use both and take maximum advantage of both?

Dapper is able to handle complex queries that have multiple joins and some real long business logic. 
Entity Framework Core is great for class generation, object tracking, mapping to multiple nested classes, 
and quite a lot more. So it’s usually performance and features when talking about these 2 ORMs.

Requirement
-----------

We will design a simple ASP.NET Core WebAPI for an imaginary company. This company has a policy that says every other Employee has to be 
linked to a unique Department. To be more clear, every time you add a new employee via the API endpoint, you have to create a new department 
record as well. A very imaginary requirement, yeah? Along with this, we will have 2 other endpoints that return all Employees and Employee by Id.

Expanding on the details, we will have to ensure the newly added Department does not already exist. You will get a grasp of this once you get to see the Domain entities.

To demonstrate the usage of Dapper, Entity Framework Core, and both combined, we will implement them each in the 3 Endpoints. 
For the GetAll Endpoints, we will use Dapper. The GetById Endpoint would use Entity Framework Core with eager loading to display 
the Department details as well. And finally, the POST Endpoint would take advantage of both these data access technologies
and cleanly demonstrate transactions in .NET Core.

Along the way, we will get introduced to few libraries for .NET Core that could probably save you some development time as well.

Important Aspect to Handle – Transactions
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Now, according to our requirement, we need both Entity Framework Core and Dapper to work alongside each other. 
This is quite easy to achieve actually. But the important detail to take care of is that we need to ensure that 
both Entity Framework Core and Dapper should participate in the same DB Transaction so that the overall process can be robust.

For example, a particular Write Operation can involve multiple entities and tables. This in turn can have operations 
that are easy to be handled by Entity Framework Core, and let’s say a bunch of complex Queries that are meant to be 
executed by Dapper. In such cases, we must make sure that it should be possible to rollback the SQL Execute operation 
when any operation/query fails. This is the aspect that can introduce a small complexity to our system design.

If we are not considering this, the overall process would be so straight forward. Let me put the idea into steps.

1. Configure Entity Framework Core.
2. Configure Dapper. You can achieve this by using the same connection string that is being used by EFCore as well. (Obviously, from appsettings.json)
3. Register the services into the Container and start using the Context / Dapper as required.

But we will go for a more complex and future proof mechanism that will handle really everything including Rollbacks and Transactions.

