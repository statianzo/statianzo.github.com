---
layout: post
title: Creating a simple web service using JAX-WS and no container
---

Web services are something new for me. I've heard all about them and the
popularity of them, but taking the task upon myself to create one was something
that I'd never done. The WSDLs looked somewhat complicated and I just had no
motivation until yesterday. However, to my surprise, I found that the process is
not really too complicated at all. In this article I'll show you how to make a
simple web service that adds and multiplies.

The Webservice
---

{% highlight java %}
//MathExampleImpl.java -- WebService Example by Jason Staten
package com.jstaten.webService.server;

import javax.jws.WebMethod;
import javax.jws.WebService;

@WebService
public class MathExampleImpl {
  @WebMethod(operationName = "addInts")
  public int addInts(int a, int b) {
    return a + b;
  }

  @WebMethod(operationName = "multiplyFloats")
  public float multiplyFloats(float a, float b) {
    return a * b;
  }
}
{% endhighlight %}

The service is very straightforward. Essentially you create a class with the
operations that will show up in the WSDL. This class has two, *addInts* and
*multiplyFloats* which will add integers and multiply floats, respectively. The
annotations *@WebService* and *@WebMethod* are later parsed when building this
application.

The Server
---

{% highlight java %}
//MathService.java -- WebService Example by Jason Staten
package com.jstaten.webService.server;

import javax.xml.ws.Endpoint;

public class MathService {
  private Endpoint endpoint = null;

  public MathService() {
    endpoint = Endpoint.create(new MathExampleImpl());
  }

  private void publish() {
    endpoint.publish("http://localhost:7070/MathExample/MathService");
  }

  public static void main(String[] args) {
    MathService hws = new MathService();
    hws.publish();
    System.out.println("Waiting");
  }
}
{% endhighlight %}


The server relies on one key component, the *Endpoint* class.
Endpoints are used to publish a webservice to a specific address. In
this very simple example, we take a new instance of our MathExampleImpl
class that we created earlier and publish it to
*http://localhost:7070/MathExample/MathService *which will be
where we direct any webservice calls to from a client. Finally, we give
the "Waiting" message to know that the server has started and is waiting
for a connection.

The Build File
---

{% highlight xml %}
<?xml version="1.0"
encoding="UTF-8"?> <!--build.xml WebService Example by Jason Staten-->
<project basedir="." default="build-server" name="MathServiceExample">
  <property environment="env" />
  <property name="lib.home" value="${env.JAXWS_HOME}/lib" />
  <property name="build.home" value="${basedir}/build" />
  <property name="build.classes.home" value="${build.home}/classes" />
  <path id="jaxws.classpath">
    <pathelement location="${java.home}/../lib/tools.jar" />
    <fileset dir="${lib.home}">
      <include name="*.jar" />
      <exclude name="j2ee.jar" />
    </fileset>
  </path>
  <taskdef name="apt" classname="com.sun.tools.ws.ant.Apt">
    <classpath refid="jaxws.classpath" />
  </taskdef>
  <target name="setup">
    <mkdir dir="${build.home}" />
    <mkdir dir="${build.classes.home}" />
  </target>
  <target name="build-server" depends="setup">
    <apt fork="true"
         debug="true"
         verbose="${verbose}"
         destdir="${build.classes.home}"
         sourcedestdir="${build.classes.home}"
         sourcepath="${basedir}/src">
      <classpath>
        <path refid="jaxws.classpath" />
        <pathelement location="${basedir}/src" />
      </classpath>
      <option key="r" value="${build.home}" />
      <source dir="${basedir}/src">
        <include name="**/server/*.java" />
        <include name="**/common/*.java" />
      </source>
    </apt>
  </target>
  <target name="clean">
    <delete dir="${build.home}" includeEmptyDirs="true" />
  </target>
</project>
{% endhighlight %}

The build file is where a lot of the magic is done. At first glace it may seem
a little confusing, but broken down, it is actually quite simple. Lets step
through it.

{% highlight xml %}
<property environment="env" />
<property name="lib.home" value="${env.JAXWS_HOME}/lib" />
<property name="build.home" value="${basedir}/build" />
<property name="build.classes.home" value="${build.home}/classes" />
{% endhighlight %}

The first section is properties
that will be used throughout the rest of the build file describing the
location of JAX-WS and the location to place compiled files.

{% highlight xml %}
<path id="jaxws.classpath">
  <pathelement location="${java.home}/../lib/tools.jar" />
  <fileset dir="${lib.home}">
    <include name="*.jar" />
    <exclude name="j2ee.jar" />
  </fileset>
</path>
{% endhighlight %}

This section defines the classpath of JAX-WS for usage within the
build file.

{% highlight xml %}
<taskdef name="apt" classname="com.sun.tools.ws.ant.Apt">
  <classpath refid="jaxws.classpath"/>
</taskdef>
{% endhighlight %}

Within this element, we declare *apt* which is what does the actual parsing
of the annotations within your source files.
We will use it as an element later on in the build file.

{% highlight xml %}
<target name="build-server" depends="setup">
<apt  fork="true"
      debug="true"
      verbose="${verbose}"
      destdir="${build.classes.home}"
      sourcedestdir="${build.classes.home}"
      sourcepath="${basedir}/src">
    <classpath>
      <path refid="jaxws.classpath" />
      <pathelement location="${basedir}/src" />
    </classpath>
    <option key="r" value="${build.home}" />
    <source dir="${basedir}/src">
      <include name="**/server/*.java" />
      <include name="**/common/*.java" />
    </source>
  </apt>
</target>
{% endhighlight %}

The build-server target is the main operation
within the build file. It utilizes *apt* like we declared earlier
and tells it where to place the compiled code as well as the generated
source. Using the properties we declared earlier, we can essentially
just fill in the blanks. destdir is the location of the compiled code,
sourcedestdir is the placement of generated source code(from parsing the
attributes), and sourcepath is the path to the source files that you
created earlier.

{% highlight xml %}
<target name="clean">
  <delete dir="${build.home}" includeEmptyDirs="true"/>
</target>
{% endhighlight %}

Finally, the clean target removes old compiled code.
This can be run by using *ant clean*
After compiling a simple *java com.jstaten.webService.server.MathService* will
start the server and a quick pointing of your browser to
*http://localhost:7070/MathExample/MathService?wsdl* will result
in receiving a WSDL. In a later article I will discuss the creation of a
client to use this service a within both Java and C#.

*Originally Written 11/20/2008*
