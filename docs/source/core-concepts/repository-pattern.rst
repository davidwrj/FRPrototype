Repository Pattern
==================

What’s a Repository Pattern?
----------------------------

A Repository pattern is a design pattern that mediates data from and to the Domain and Data Access Layers (like Entity Framework Core/Dapper). 
Repositories are classes that hide the logic required to store or retrieve data. Thus, our application will not care about what kind of ORM 
we are using, as everything related to the ORM is handled within a repository layer. This allows you to have a cleaner separation of concerns. 
Repository pattern is one of the heavily used Design Patterns to build cleaner solutions.

Benefits of Repository Pattern
------------------------------

Reduces Duplicate Queries
^^^^^^^^^^^^^^^^^^^^^^^^^

Imagine having to write lines of code to just fetch some data from your datastore. Now what if this set of queries are going to be used in 
multiple places in the application. Not very ideal to write this code over and over again, right? Here is the added advantage of 
Repository classes. You could write your data access code within the Repository and call it from multiple Controllers/Libraries.

De-couples the application from the Data Access Layer
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

There are quite a lot of ORMs available for .NET Core. Currently the most popular one is Entity Framework Core. But that could change in 
the upcoming years. To keep pace with the evolving technologies and to keep our solutions up to date, it is crucial to build applications 
that can switch over to a new data access technology with minimal impact on our application’s code base.

There can be also cases where you need to use multiple ORMs in a single solution. Probably Dapper to fetch the data and EF Core to write the data. 
This is solely for performance optimizations.

Repository pattern helps us to achieve this by creating an abstraction over the data access layer. Now, you no longer have to depend on EF Core
or any other ORM for your application. EF Core becomes one of your options rather than your only option to access data.

.. admonition:: Tip

   The Architecture should be independent of the Frameworks.
   – Uncle Bob (Robert Cecil Martin)

Building an enterprise level .NET Core application would really need repository patterns to keep the codebase future proof for at least 
the next 20-25 years (After which, probably the robots would take over 😛).

Is Repository Pattern Dead?
^^^^^^^^^^^^^^^^^^^^^^^^^^^

This is one of the most debated topics within the .NET Core community.  Microsoft has built Entity Framework Core using the repository pattern
and unit of work patterns. So, why do we need to add another layer of abstraction over the Entity Framework Core, which is yet another abstraction
of data access. The answer to this is also given by Microsoft.

`Read more here`_

.. _`Read more here`: https://docs.microsoft.com/en-us/dotnet/architecture/microservices/microservice-ddd-cqrs-patterns/infrastructure-persistence-layer-implemenation-entity-framework-core#using-a-custom-repository-versus-using-ef-dbcontext-directly

Microsoft themselves recommend using repository patterns in complex scenarios to reduce the coupling and provide better testability of your solutions. 
In cases where you want the simplest possible code, you would want to avoid the repository pattern.

.. image:: /_static/images/custom-repo-versus-db-context-1024x579.png
   :width: 50%

Adding the repository has it’s own benefits too. But i strongly advice to not use design patterns everywhere. Try to use it only whenever 
the scenario demands the usage of a design pattern. That being said, repository pattern is something that can benefit you in the long run.

