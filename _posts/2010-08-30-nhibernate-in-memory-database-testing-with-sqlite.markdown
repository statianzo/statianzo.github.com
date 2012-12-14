---
layout: post
title: NHibernate in Memory Database Testing with SQLite
---

One of the pain points that can come along with most tests that hit a database
is possibility for inconsistent results. "Just run it again" is not the best
mentality to have about it. For example, when someone is running tests locally
at the same time your continuous integration is running. The CI fails and you're
faced with the task of figuring out what's wrong, when it's really nothing.

We've got an ORM to abstract the database from our app. Why not put it to use?

~

SQLite
---

Enter SQLite. [SQLite][sqlite] is a basic, lightweight SQL database wrapped
into a single library. A benefit of its use is that SQLite can create an
in-memory database that cleans up after all connections are closed. Its
excellent for testing. Starting with a clean slate each test run can provide
for consistent, reproducible results.

Prerequisites
---

[NHibernate][nhforge] comes with support for SQLite from the
*NHibernate.Driver.SQLite20Driver* class.  However, you must provide two other
files for it to work. First, [sqlite3.dll][sqlitedll], the SQLite unmanaged
library; include it in your test project and ensure "copy to output directory"
is marked.  Second, [System.Data.SQLite][sqlitemanaged] a managed library that
allows ADO.NET to interact with SQLite; include this as a reference in your test
project.

A Base Class
---

The following is a basic example of how to set up a base class for in memory
database testing. Any tests that require database interaction should inherit
from this class and will have the Session object available. Sessions last as
long as each test fixture. Also, `typeof(Plan)` refers to the type of any of
your domain classes.

    public class InMemoryDatabaseTest
    {
      private static Configuration _configuration;
      protected readonly ISession Session;
      private readonly object _baton = new object();

      private readonly ISessionFactory
        _sessionFactory;

      public InMemoryDatabaseTest()
      {
        if (_configuration == null)
          lock (_baton)
            if (_configuration == null)
            {
              _configuration = new Configuration()
                .SetProperty(Environment.ReleaseConnections, "on_close")
                .SetProperty(Environment.ProxyFactoryFactoryClass,
                             "NHibernate.ByteCode.Castle.ProxyFactoryFactory, NHibernate.ByteCode.Castle")
                .SetProperty(Environment.CacheProvider,
                             "NHibernate.Caches.SysCache.SysCacheProvider, NHibernate.Caches.SysCache")
                .SetProperty(Environment.ConnectionDriver, typeof (SQLite20Driver).AssemblyQualifiedName)
                .SetProperty(Environment.ConnectionProvider,
                             "NHibernate.Connection.DriverConnectionProvider")
                .SetProperty(Environment.Isolation, "ReadCommitted")
                .SetProperty(Environment.ConnectionString, "Data Source=:memory:;Version=3;New=True;")
                .SetProperty(Environment.ShowSql, "True")
                .SetProperty(Environment.Dialect, typeof (SQLiteDialect).AssemblyQualifiedName)
                .AddAssembly(Assembly.GetAssembly(typeof(YourDomainObject)));

        _sessionFactory = _configuration.BuildSessionFactory();
        Session = _sessionFactory.OpenSession();
        var schemaExport = new SchemaExport(_configuration);
        schemaExport.Execute(false, true, false, Session.Connection, null);
      }

      public void Dispose()
      {
        Session.Dispose();
      }
    }


Support for Schemas
---

SQLite doesn't provide native support for schemas. We use SQL Server, and this
led to a problem when any of our mappings pointed to tables in anything other
than the *dbo* schema. A simple fix to this is to replace all periods with
underscores prior to creating your session factory. This gives the desired
behavior without requiring any modification to your mapping files. 

    if (_configuration == null) {
      _configuration = new Configuration()
      //Configuation...
      foreach (PersistentClass classMapping in _configuration.ClassMappings)
      {
        if (classMapping.Table.Name.Contains("."))
          classMapping.Table.Name = classMapping.Table.Name.Replace(".", "_");
      }
    }

Working with AutoMockContainer
---

If you're a user of [Moq-Contrib's AutoMockContainer][moqcontrib], you can
register the Session object into the container to have it provided instead of a
mocked session. It's as simple as adding the following after creating your
session object.

    MockContainer.Register(Session); 

Testing with NHProf
---

If you use NHibernate Profiler (if you don't, you should be), your SQLite
database interaction can be monitored using that as well. Add an
AssemblyInitialize (or your testing framework's variant) to your
InMemoryDatabaseTest class and all interactions will be profiled.

    [AssemblyInitialize]
    public static void AssemblyInit(TestContext testContext)
    {
      NHibernateProfiler.Initialize(); 
    } 

Having trustworthy results is essential in any test driven development.  Using
SQLite for testing can help testing scenarios that have consistent results
each time.

[sqlite]:http://www.sqlite.org/  
[moqcontrib]:http://code.google.com/p/moq-contrib/ 
[sqlitemanaged]:http://sqlite.phxsoftware.com/
[nhforge]:http://nhforge.org/
[sqlitedll]:http://www.sqlite.org/download.html


