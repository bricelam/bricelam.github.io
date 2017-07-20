---
layout: post
title: Test EF Core using SQL Server Docker images
tags: entity-framework
---

Some tests in the Entity Framework Core codebase require features only available on Microsoft SQL Server 2016 Developer
Edition or higher. Since most developers only have Microsoft SQL Server 2016 LocalDB (and even then only because it was
installed by Visual Studio), it can be troublesome to run these tests.

Using the [SQL Server Docker images][1] can dramatically simplify this task.

Start by installing [Docker for Windows][2]. Ensure you're using Windows containers by right-clicking on the Docker
system tray icon and selecting **Switch to Windows containers...** if it's available.

Download the [microsoft/mssql-server-windows-developer][3] image.

    docker pull microsoft/mssql-server-windows-developer

Create and start a new container with it. Use 2GB of memory. Publish port 1433 to the host.

    docker run -d -m 2g -p 1433:1433 -e sa_password=Password12! -e ACCEPT_EULA=Y microsoft/mssql-server-windows-developer

At this point, you can start ad-hoc testing against the server using the following connection string.

    Data Source=(local);Initial Catalog=MyTestDatabase;User ID=sa;Password=Password12!

To run the full test suite against the server, set a few environment variables first.

```shell
SET "Test__SqlServer__DefaultConnection=Data Source=(local);User ID=sa;Password=Password12!"
SET Test__SqlServer__SupportsMemoryOptimized=true
SET Test__SqlServer__SupportsHiddenColumns=true
build
```

Voila! You can now test against SQL Server 2017 Developer Edition without installing it locally.


  [1]: https://github.com/Microsoft/mssql-docker
  [2]: https://www.docker.com/docker-windows
  [3]: https://hub.docker.com/r/microsoft/mssql-server-windows-developer/
