---
layout: post
title: Logging to CouchDB from C#
---

Setting up a centralized form of logging can be a pain. CouchDB can help make it easy.

Prerequisites
---

First, grab the [log4net][l4n] library. It's common, proven, and makes logging
straightforward.

Second, pull down [PostLog][pl] from github and build

    $ git clone https://statianzo@github.com/statianzo/PostLog.git
    $ cd PostLog
    $ msbuild PostLog.sln


PostLog is an HttpAppender for log4net. It will make an HTTP request for logging events.

Finally, with an instance of [CouchDB][couch] running, create a database for logging against.

    $ curl -X PUT http://localhost:5984/testlog

Use log4net like normal
---

If your project currently uses, you're in business.
If not, take the following example:

    using System;
    using log4net.Config;
    using log4net;

    namespace CouchDBAppenderExample
    {
      class MainClass
      {
        public static void Main (string[] args)
        {
          XmlConfigurator.Configure();
          ILog logger = LogManager.GetLogger(typeof(MainClass));
          Console.WriteLine ("Press enter to do it");
          Console.ReadLine();
          try {
            throw new InvalidOperationException("Don't do that!");
          }
          catch(InvalidOperationException e) {
            logger.Error("It blew up", e);
          }
          Console.WriteLine("Done");
          Console.ReadLine();
        }
      }
    }

A typical log4net usage. An exception occurs and it gets logged.

Configure log4net
---

PostLog's `HttpAppender` is configured in the `log4net` app.config section.
Several options are available.

- `uri` The location to send the HTTP Request
- `method` The HTTP method to be used (defaults to POST)
- `useragent` The value of the useragent header
- `format` The format to serialize the log data. Options are *form*, *json*, and *xml*
- `xmlrootname` root element tag for xml

A sample configuration is as follows:

    <?xml version="1.0"?>
    <configuration>
      <configSections>
        <section name="log4net"
                 type="log4net.Config.Log4NetConfigurationSectionHandler, log4net" />
      </configSections>

      <log4net>
        <appender name="HttpAppender" type="PostLog.HttpAppender, PostLog">
          <uri value="http://localhost:5984/testlog" />
          <formatterType value="PostLog.JsonBodyFormatter, PostLog" />
          <parameter>
            <name value="date" />
            <layout type="log4net.Layout.RawTimeStampLayout" />
          </parameter>
          <parameter>
            <name value="level" />
            <layout type="log4net.Layout.PatternLayout">
              <conversionPattern value="%level" />
            </layout>
          </parameter>
          <parameter>
            <name value="message" />
            <layout type="log4net.Layout.PatternLayout">
              <conversionPattern value="%message" />
            </layout>
          </parameter>
          <parameter>
            <name value="exception" />
            <layout type="log4net.Layout.ExceptionLayout" />
          </parameter>
        </appender>
        <root>
          <level value="DEBUG" />
          <appender-ref ref="HttpAppender" />
        </root>
      </log4net>
    </configuration>

With the new appender in place, your log statements should start hitting CouchDB.

Views
---

Now that your log statements are in Couch, you probably want to look at them.

Let's create a file named logging.json. The file has a view to see all error log statements.

    {
      "_id":"_design/logging",
      "language": "javascript",
      "views":
      {
        "error": {
          "map": "function(doc) { if (doc.level === 'ERROR')  emit(doc.date, doc) }"
        },
      }
    }

Send it to Couch with curl:

    $ curl -T - http://localhost:5984/testlog/_design/logging < logging.json

Now we can view all error statements by calling

    $ curl http://localhost:5984/testlog/_design/logging/_view/error

Example
---

An example project is available on github at: 
https://github.com/statianzo/CouchDBAppenderExample

[l4n]: http://logging.apache.org/log4net/index.html
[pl]: https://github.com/statianzo/PostLog
[couch]: http://couchdb.apache.org/
