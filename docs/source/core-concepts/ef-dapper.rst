Entity Framework Core and Dapper
================================

The describes using Entity Framework Core and Dapper together in the same application. 
Another major point of discussion will be transactions. By the end of this, we will have an application that works with both 
Entity Framework Core and Dapper alongside each other, but also intelligent enough to rollback data whenever there is an exception with the process.

We will also take a look at `ThrowR`_, a simple library for .NET Core that can make your code clean and readable by eliminating the use of *if* statements unnecessarily.

.. _`ThrowR`: https://www.nuget.org/packages/CWM.DotNetCore.ValidatR/

