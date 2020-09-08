---
layout: post
title: Test EF Core using SQL Server Docker images
tags: [ entity-framework, sql-server ]
---

Some tests in the Entity Framework Core codebase require features only available on Microsoft SQL Server Developer
Edition or higher. Since most developers only have Microsoft SQL Server 2016 LocalDB (and even then only because it was
installed by Visual Studio), it can be troublesome to run these tests.

Using the [SQL Server Docker images][1] can dramatically simplify this task.

Start by installing [Docker for Windows][2].

Download, create, and start a new container using the [mcr.microsoft.com/mssql/server][3] image. Publish port 1433 to the host.

    docker run -d -p 1433:1433 -e SA_PASSWORD=Password12! -e ACCEPT_EULA=Y mcr.microsoft.com/mssql/server

At this point, you can start ad-hoc testing against the server using the following connection string.

    Data Source=(local)\MSSQLSERVER;Initial Catalog=MyTestDatabase;User ID=sa;Password=Password12!

To run the full test suite against the server, set a few environment variables first.

```bat
SET "Test__SqlServer__DefaultConnection=Data Source=(local)\MSSQLSERVER;User ID=sa;Password=Password12!"
SET Test__SqlServer__SupportsMemoryOptimized=true
SET Test__SqlServer__SupportsHiddenColumns=true
build
```

Voila! You can now test against SQL Server Developer Edition without installing it locally.


  [1]: https://github.com/Microsoft/mssql-docker
  [2]: https://www.docker.com/docker-windows
  [3]: https://hub.docker.com/_/microsoft-mssql-server
