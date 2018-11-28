---
title: Work with SQLite in an ASP.NET Core MVC app
author: rick-anderson
description: Using SQLite with a simple MVC app
ms.author: riande
ms.date: 04/07/2017
uid: tutorials/first-mvc-app-xplat/working-with-sql
---

[!INCLUDE [adding-model](../../includes/mvc-intro/sql.md)]

> [!div class="step-by-step"]
> [Previous - Add a model](adding-model.md)


The MvcMovieContext object handles the task of connecting to the database and mapping Movie objects to database records. The database context is registered with the Dependency Injection container in the ConfigureServices method in the Startup.cs file:
C#

      public void ConfigureServices(IServiceCollection services)
      {
          // Add framework services.
          services.AddMvc();

          services.AddDbContext<MvcMovieContext>(options =>
                  options.UseSqlite("Data Source=MvcMovie.db"));

      }

SQLite

The SQLite website states:

    SQLite is a self-contained, high-reliability, embedded, full-featured, public-domain, SQL database engine. SQLite is the most used database engine in the world.

There are many third party tools you can download to manage and view a SQLite database. The image below is from DB Browser for SQLite. If you have a favorite SQLite tool, leave a comment on what you like about it.

DB Browser for SQLite showing movie db
Seed the database

Create a new class named SeedData in the Models folder. Replace the generated code with the following:
C#

using Microsoft.EntityFrameworkCore;
using Microsoft.Extensions.DependencyInjection;
using System;
using System.Linq;

namespace MvcMovie.Models
{
    public static class SeedData
    {
        public static void Initialize(IServiceProvider serviceProvider)
        {
            using (var context = new MvcMovieContext(
                serviceProvider.GetRequiredService<DbContextOptions<MvcMovieContext>>()))
            {
                // Look for any movies.
                if (context.Movie.Any())
                {
                    return;   // DB has been seeded
                }

                context.Movie.AddRange(
                     new Movie
                     {
                         Title = "When Harry Met Sally",
                         ReleaseDate = DateTime.Parse("1989-1-11"),
                         Genre = "Romantic Comedy",
                         Price = 7.99M
                     },

                     new Movie
                     {
                         Title = "Ghostbusters ",
                         ReleaseDate = DateTime.Parse("1984-3-13"),
                         Genre = "Comedy",
                         Price = 8.99M
                     },

                     new Movie
                     {
                         Title = "Ghostbusters 2",
                         ReleaseDate = DateTime.Parse("1986-2-23"),
                         Genre = "Comedy",
                         Price = 9.99M
                     },

                   new Movie
                   {
                       Title = "Rio Bravo",
                       ReleaseDate = DateTime.Parse("1959-4-15"),
                       Genre = "Western",
                       Price = 3.99M
                   }
                );
                context.SaveChanges();
            }
        }
    }
}

If there are any movies in the DB, the seed initializer returns.
C#

if (context.Movie.Any())
{
    return;   // DB has been seeded.
}

Add the seed initializer

Add the seed initializer to the Main method in the Program.cs file:
C#

using Microsoft.AspNetCore;
using Microsoft.AspNetCore.Hosting;
using Microsoft.Extensions.DependencyInjection;
using Microsoft.Extensions.Logging;
using System;
using Microsoft.EntityFrameworkCore;
using MvcMovie.Models;
using MvcMovie;

namespace MvcMovie
{
    public class Program
    {
        public static void Main(string[] args)
        {
            var host = CreateWebHostBuilder(args).Build();

            using (var scope = host.Services.CreateScope())
            {
                var services = scope.ServiceProvider;

                try
                {
                    var context = services.GetRequiredService<MvcMovieContext>();
                    context.Database.Migrate();
                    SeedData.Initialize(services);
                }
                catch (Exception ex)
                {
                    var logger = services.GetRequiredService<ILogger<Program>>();
                    logger.LogError(ex, "An error occurred seeding the DB.");
                }
            }

            host.Run();
        }

        public static IWebHostBuilder CreateWebHostBuilder(string[] args) =>
            WebHost.CreateDefaultBuilder(args)
                .UseStartup<Startup>();
    }
}

Test the app

Delete all the records in the DB (So the seed method will run). Stop and start the app to seed the database.

The app shows the seeded data.
































> [Next - Controller methods and views](controller-methods-views.md)
