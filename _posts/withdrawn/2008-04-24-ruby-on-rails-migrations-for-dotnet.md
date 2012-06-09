---
layout: post
title: Ruby on Rails Migrations for .NET
published: false
categories: dotnetmigrations
---
I've been looking a lot at Ruby on Rails lately, and one of the things
that really stands out to me are the
[migrations](http://wiki.rubyonrails.org/rails/pages/UnderstandingMigrations).
Migrations is the Rails way of defining your database schema in such a way that it is easy to:

- put into source control
- shared between developers
- rollback or upgrade to any version of the schema from any version

Plenty of info is available about migrations on at the Ruby on Rails website
so I won't duplicate that here. I do recommend reading the basics on
[how RoR migrations work](http://wiki.rubyonrails.org/rails/pages/UnderstandingMigrations)
before continuing.

Suffice to say I wanted something like this to use with my .NET applications pretty bad.
I searched around and couldn't find anything like this for .NET (without
jumping into some full blown OR/M) so I made one myself.

I call it "DotNetMigrations" and it's as close to the RoR version of migrations
as I could make it. The biggest difference is that your migration script is real
T-SQL instead of Ruby. This is actually a big plus in the .NET/SQL Server world
since you can just open your script in SQL Management Studio to test it, and
copy/paste the scripts Studio generates for you to create your migration scripts.

Here's a screenshot from the app with basic usage examples:

<a class="lightbox img" href="http://1.bp.blogspot.com/_uTRG_A-YdZY/SBEDtvhLfFI/AAAAAAAABf4/LjHlzRMz8N8/s1600-h/DotNetMigrations_usage.png">
    <img src="http://1.bp.blogspot.com/_uTRG_A-YdZY/SBEDtvhLfFI/AAAAAAAABf4/LjHlzRMz8N8/s400/DotNetMigrations_usage.png" /></a>

... and here's the readme file:

    README
    -------------------

    Run db.exe to see usage information.

    All migration scripts are stored underneath the current working directory in: /migrate/

    Connection strings to your database can be stored in the db.exe.config file.
    Connection strings in this file can be referenced by name when executing commands on db.exe.

    A [dbo].[schema_info] table will be created inside the database.
    This table is used to store the current schema version and will be updated
    whenever you run the 'migrate' command.

Of course everything the application does is wrapped in transactions so if at any
point during a migration an error occurs (i.e. some goof-ball screwed up the migration script)
it will roll back to the original version of the database.

Since your migration script is just T-SQL you can do anything from create tables and
add columns to doing data transformations. And assuming your scripts are well written,
you'll be able to transform your database to/from any version of the schema at anytime.

I'm considering creating a Google Code project for this if there's enough interest. Thoughts?