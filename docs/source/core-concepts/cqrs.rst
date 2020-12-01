CQRS with MediatR
=================

In this article let’s talk about **CQRS** and it’s implementation along with **MediatR** and **Entity Framework Core** using a code first approach. 
I will implement this pattern on a WebApi Project. The source code of this sample is linked at the end of the post. Of the several design patterns available, 
CQRS is one of the most commonly used patterns that helps architect the solution to accommodate the **Onion Architecture**. 

What is CQRS?
-------------

**CQRS**, Command Query Responsibility Segregation is a design pattern that separates the read and write operations of a data source. 
Here *Command* refers to a database command, which can be either an Insert, Update or Delete operation, whereas *Query* refers to querying data from a source, put another way, 
commands affect state, queries do not.

It essentially separates the concerns in terms of reading and writing, which makes quite a lot of sense. This pattern was originated from the Command Query Separation 
principle devised by Bertrand Meyer. 

.. admonition:: Wikipedia 

   Every method should either be a command that performs an action, or a query that returns data to the caller, but not both. 
   In other words, asking a question should not change the answer. More formally, methods should return a value only if they are referentially 
   transparent and hence possess no side effects.

The problem with traditional architectural patterns is that the same data model or DTO is used to query as well as update a data source. 
This can be the go-to approach when your application is related to just CRUD operations and nothing more. 
But when your requirements suddenly start getting complex, this basic approach can prove to be a disaster.

In practical applications, there is always a mismatch between the read and write forms of data, like the extra properties you may require to preform an update. 
Parallel operations may even lead to data loss in the worst cases. That means, you will be stuck with just one Data Transfer Object for the entire lifetime of 
the application unless you choose to introduce yet another DTO, which in-turn may break your application architecture.

The idea with **CQRS** is to allow an application to work with different models, you may have one model that has data needed to update a record, 
another model to insert a record, yet another to query a record. This gives you flexibility with varying and complex scenarios. 
You don’t have to rely on just one DTO for the entire CRUD operation by implementing CQRS.  In fact, in many cases the command and query requests and responses are the DTO.

.. image:: /_static/images/cqrs-asp.net-core-3.1-explained.png
   :width: 50%

Pros of CQRS
------------

There are quite of lot of advantages on using the CQRS pattern for your application;

Optimised Data Transfer Objects
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Thanks to the segregated approach of this pattern, we will no longer need those complex model classes within our application. Rather we have one model per data operation.
These should not be shared.

Highly Scalable
^^^^^^^^^^^^^^^

Having control over the models in accordance with the type of data operations makes your application highly scalable.

Improved Performance
^^^^^^^^^^^^^^^^^^^^

Generally speaking there are usually many more **Read** operations as compared to **Write** operations. With this pattern you could speed up the 
performance on your read operations by introducing a read optimised database. CQRS pattern will support this usage out of the box.

Secure Parallel Operations
^^^^^^^^^^^^^^^^^^^^^^^^^^

Since we have dedicated models per operation, there is no possibility of data loss while doing parallel operations.

Cons of CQRS
------------

Added Complexity and More Code
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The one thing that may concern a few programmers is that this is a code demanding pattern. In other words, you will 
end up with more code lines than you usually would. But everything comes for a price. This is a small price to pay while 
getting the awesome features and possibilities with the pattern.

