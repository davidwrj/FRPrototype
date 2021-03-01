Onion Architecture
==================

This describes Onion Architecture as used by the Freight Rate Prototype and it’s perceived advantages.  
If you want to build a project that utilises what is described here, see the :doc:`onion application <onion-application>` documentation.

The four tenets of Onion Architecture:

 * The application is built around an independent object model
 * Inner layers define interfaces.  Outer layers implement interfaces
 * Direction of coupling is toward the center
 * All application core code **can be compiled and run** separate from infrastructure

**Onion Architecture** works well with and without DDD patterns.  It works well with CQRS, forms over data, and DDD.  
It is merely an architectural pattern where the core object model is represented in a way that does not accept dependencies on less stable code.

The need to follow an architecture
----------------------------------

To maintain structural sanity in mid to larger solutions, it is recommended to follow some kind of architecture. 
You may have seen other open source projects having multiple layers of projects within a complex folder structure.  

.. admonition:: Example

   `dotnetcore CAP`_ is a library based on .NET standard, which is a solution to deal with distributed transactions, 
   has the function of EventBus, it is lightweight, easy to use, and efficient.

.. _`dotnetcore CAP`: https://github.com/dotnetcore/CAP/tree/master/src

Layers vs Tiers
^^^^^^^^^^^^^^^

When there is just a logical separation in your application, we can term it as layers or N-Layers. 
In cases where there is both a physical and logical separation of concerns, it is often referred to as 
an n-tiered application where n is the number of separations. 3 is the most common value of N. 

This layering can help in the separation of concerns, subdividing the solution into smaller units 
so that each unit is responsible for a specific task and also to take advantage of abstraction. 
For mid to larger scaled projects, layering has very obvious advantages. It lets a specific team 
or individual work on a particular layer without disturbing the integrity of the others. 

Also, **it just makes your entire solution look clean**, easy to maintain, and for new members, easier to learn. 

Before getting into **Onion Architecture** in the prototype, let’s first refresh our knowledge on N-Layer architecture.

Brief Overview of N-Layer Architecture
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Let’s look at one of the most popular architectures in an application. Here is a simple diagrammatic 
representation of a variant of the N-Layer Architecture. The *Presentation Layer* usually holds the code that the user 
can interact with, i.e. WebApi, MVC, WebForms and so on. *Business Logic Layer* is probably the most important part of 
the application. It holds all the logic related to the business features. Ideally, every application has it’s own dedicated database.
In order to access the database, we introduce a *Data Access Layer*. This layer usually holds ORMs for ASP.NET to fetch/write to the database.

.. image:: /_static/images/N-Tier-Architecture.png
   :width: 50%

Disadvantages of N-Layer Architecture
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

To clearly understand the advantages of an **Onion Architecture**, we will need to study the issues with N-Layer architecture. 
It is one of the commonly used solution architectures amongst .NET developers.

Instead of building a highly decoupled structure, we often end up with several layers that are depending on each other. 
This is something really bad in building scalable applications and may pose issues with the growth of the codebase. 
To keep it clear, in the above diagram we can see that the presentation layer depends on the logic layer, 
which in turn depends on the data access and so on.

Thus, we would be creating a bunch of unnecessary couplings. Is it really needed? 
In most of the cases the UI (presentation) layer would be coupled to the Data Access Layers as well. 
This would defeat the purpose of having a clean architecture.

In N-Layer architecture, the database is usually the core of the entire application, i.e. it is the only layer that 
doesn’t have to depend on anything else. Any small change in the business logic or data access layer may prove dangerous 
to the integrity of the entire application.

Getting Started with Onion Architecture
---------------------------------------

The **Onion Architecture**, introduced `by Jeffrey Palermo`_, overcomes the issues of the layered architecture with great ease. 
With **Onion Architecture**, the major difference is that the **Domain Layer** (*Entities*, *Validation Rules* and *behaviour* common to a business use case) 
is at the core of the entire application. This means higher flexibility and lesser coupling. 
In this approach, we can see that the layers are dependent only on inner layers, the arrows NEVER point outwards.

.. image:: /_static/images/Onion-Architecture-In-ASP.NET-Core.png
   :width: 50%

Here is how I would breakdown the structure of the proposed solution.

.. _`by Jeffrey Palermo`: https://jeffreypalermo.com/2008/07/the-onion-architecture-part-1/

Domain and Application Layers
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Will be at the center of the design. We can refer to these layers as the *Core Layers*. These layers will not depend on any other layers.

The **Domain Layer** usually contains the domain knowledge and behaviours. **Application Layer** would have interfaces and types. 

As mentioned earlier, the core layers will never depend on any other layer. Therefore what we do is that we create interfaces in 
the application layer and these interfaces get implemented in the external layers. This is also known as Dependency Inversion principle.

.. admonition:: Example

   If your application needs to send a mail, we define an IMailService in the application layer and implement it outside the core layers. 
   Using DI principles, it is easily possible to switch the implementation. This helps build testable applications.

Presentation Layer
^^^^^^^^^^^^^^^^^^

Where you would put the project that the user or other external applications can access. This could be a WebApi, Mvc Project, etc.

Infrastructure Layer
^^^^^^^^^^^^^^^^^^^^

A bit more tricky, but is where you add your *infrastructure*. Infrastructure can be anything but is typically the concrete implementation of 
external services that provide specific functionality for a specific technology. 

.. admonition:: Example

   An Entity Framework implementation for accessing the database, or a service specifically made to generate JWT Tokens for authentication 
   or a Hangfire service. 

This should become clearer when we start implementing **Onion Architecture** in the prototype application.

Advantages of Onion Architecture
--------------------------------

The advantages of this designs is as follows.

 * **Highly Testable** – Since the Core has no dependencies on anything else, writing automated tests are flexible,
 * **Database Independent** – Since we have a clean separation of data access, it is quite easy to switch between different database providers, e.g.  SQL Server and/or MongoDB.
 * **Switchable UI Layer** (Presentation) – Since we are keeping all the crucial logic away from the presentation layer, it is quite easy to switch to another tech – including *Blazor*.
 * Much cleaner code base with well structured projects for better understanding within teams.

