MediatR
=======

**MediatR** is a library which essentially can make your controllers thin and decouple the functionality to a more message-driven approach. 
This is generally coupled with an implementation of the **CQRS** pattern which is Command Query Responsibility Segregation.

The idea of pipelines, *MediatR Pipeline Behaviour*, how to intersect the pipeline and add various services like Logging and Validations are going to be covered here.

Pipelines – Overview
--------------------

What happens internally when you send a request to any application? Ideally it returns the response, but there is one thing you might not be aware of yet, pipelines. 
These requests and response travel back and forth through pipelines. So, when you send a request, the request message passes from the user through a pipeline towards
the application, where you perform the requested operation with the request message. Once done, the application sends back the message as a response through the 
pipeline towards the user-end. Thus these pipelines are completely aware of what the request or response is. This is also a very important concept while learning 
about Middleware in ASP.NET Core web sites and apis.

Here is an image depicting the above mentioned concept.

.. image:: /_static/images/Pipeline-Concept.png
   :width: 50%

Let’s say I want to validate the request object. How would you do it? You would basically write the validation logic which executes after the request has reached 
the end of the pipeline towards the application. That means, you are validating the request only after it has reached inside the application. 
Why would you attach the validation logic to the application when you can already validate the incoming requests even before it hits any of the application logic?

A better approach would be to somehow wire up your validation logic within the pipeline, so that the flow becomes;

1. user sends request through pipeline,
2. validation logic intercepts request, 
3. if request is valid, continue, 
   else throws a validation exception.
   
This makes quite a lot of sense in terms of efficiency. Why hit the application with invalid data, when you could filter it out much before?

This is not only applicable for validation, but for various other operations like logging, performance tracking and much more. 

MediatR Pipeline Behaviour
--------------------------

**MediatR** takes a more pipeline kind of approach where your queries, commands and responses flow through a pipeline.  
The **Pipeline Behaviour** was made available from Version 3 of this library.

MediatR requests or commands are like the first contact within our application, so why not attach some behaviours?  By doing this, we will be 
able to execute service logic like validation even before the *Command* or *Query* handlers know about it. This way, we will be sending only 
necessary valid requests to the *CQRS Implementation*. Logging using this **Pipeline Behavior** is also a common implementation.

Getting Started
---------------

We will use the CQRS where I already have setup MediatR. We will be adding Validation (via **Fluent Validation**) and general logging to the commands and requests that go through the pipeline.

Here is what we are going to build, in our CQRS solution, we are going to add validation and logging via the **MediatR** pipeline.

Thus, any <Feature>Command or <Feature>Query request would be validated even before it hits the application logic. 
Also, we will log every request and response that goes through the **MediatR** pipeline.

