---
layout: post
title: Creating a Java Web Service Client from a WSDL
---

In my previous post, I discussed the creation
of a web service. That's all good and dandy, but can be rather boring because we
can't do anything with it other than touch it with a web browser and see a bunch
of XML. Although this example is very specific to the web service created
prior, the concepts can be applied to most web services.

Generating Classes from a WSDL
---

This project is different than some because it starts
with a build.xml. Within it is the necessary task to generate `*.java` files we
will use later for communicating with the web service.
    <?xml version="1.0"?>
    <!--build.xml WebService Client Example by Jason Staten-->
    <project basedir="." default="client" name="MathServiceExample">
      <property environment="env"/>
      <property name="lib.home" value="${env.JAXWS_HOME}/lib"/>
      <property name="build.home" value="${basedir}/build"/>
      <property name="build.classes.home" value="${build.home}/classes"/>
      <path id="jaxws.classpath">
        <pathelement location="${java.home}/../lib/tools.jar"/>
        <fileset dir="${lib.home}">
          <include name="*.jar"/>
          <exclude name="j2ee.jar"/>
        </fileset>
      </path>
      <taskdef name="wsimport" classname="com.sun.tools.ws.ant.WsImport">
        <classpath refid="jaxws.classpath"/>
      </taskdef>
      <target name="setup">
        <mkdir dir="${build.home}"/>
        <mkdir dir="${build.classes.home}"/>
      </target>
      <target name="generate-client">
        <wsimport debug="true"
                  verbose="${verbose}"
                  keep="true"
                  sourcedestdir="${basedir}/src"
                  package="com.jstaten.webService.client.generated"
                  wsdl="http://localhost:7070/MathExample/MathService?wsdl"
                  xnocompile="true">
    </wsimport>
      </target>
      <target name="client" depends="generate-client,setup">
        <javac  fork="true"
                srcdir="${basedir}/src"
                destdir="${build.classes.home}"
                includes="**/client/**,**/common/**">
          <classpath refid="jaxws.classpath"/>
        </javac>
      </target>
      <target name="clean">
        <delete dir="${build.home}" includeEmptyDirs="true"/>
      </target>
    </project>

There are two key parts to this build file, the taskdef that declares what the *wsimport*
tag actually is and the task *generate-client* that uses wsimport to
generate the java that provides an interface to the webservice. If you look
within your *src* directory, you will now notice that there are now
`*.java` files for use with your own code. Let's get to using it.

Writing the Client
---

    package com.jstaten.webService.client;
    import com.jstaten.webService.client.generated.*;
    public class MathClient {
        public static void main(String[] args) {
            MathExampleImpl port = new
            MathExampleImplService().getMathExampleImplPort();
            int addA = 10;
            int addB = 20;
            float multiplyA = 30;
            float multiplyB = 20.59f;
            
            System.out.println(addA + " + " + addB + " =");
            
            System.out.println(port.addInts(addA, addB));
            System.out.println(multiplyA +
              " * " + multiplyB + " =");
        }
    }
        
        
 
The code is relatively straightforward. We make an import call to
the generated classes. Next we create a new *MathExampleImpl* object
that is our interface to the webservice. Finally, we run a few test methods
on it and return the result to the console so we can see the result.

Build the Client
---

    <?xml version="1.0"?>
    <target name="client" depends="generate-client,setup">
      <javac fork="true"
             srcdir="${basedir}/src"
             destdir="${build.classes.home}"
             includes="**/client/**,**/common/**">
        <classpath refid="jaxws.classpath"/>
      </javac>
    </target>

Adding the above into your build file within the *project* element will allow
you to build the client. Essentially it's just a javac call with some
arguments. Having it in the build file allows you to not have to type all
that stuff into the command line every time you compile - much more
convenient. Run the compiled client and you should now see the results you
printed to the console. Next time, doing the same thing via .NET in C

*Originally Written 12/2/2008*
