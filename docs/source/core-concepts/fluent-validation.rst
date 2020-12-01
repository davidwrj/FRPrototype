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