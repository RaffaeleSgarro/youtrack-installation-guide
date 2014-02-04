This is the base directory for our issue tracking system, based on YouTrack.
TeamCity is integrated to allow YouTrack to link to source code changes.

Running the embedded programs
=============================
You can manually start/stop both YouTrack and TeamCity with the scripts provided
in bin/:

    bin/youtrack {start|stop}
    bin/teamcity {start|stop}

There are init scripts for Debian systems in init.d/. You need to copy them to
the system /etc/init.d, creat symlinks and enabling the service. This is done
as root:

    su
    cp init.d/youtrack /etc/init.d
    # ... Configure /etc/init.d/youtrack ...
    update-rc.d youtrack defaults
    update-rc.d youtrack enable

The same must be done for TeamCity. Then you can start/stop service with

    service youtrack {start|stop|restart}


Installation log: YouTrack installation
=======================================

Assuming there's already an instance of Tomcat up and running, you need to
configure the boot parameters and deploy the YouTrack webapp. These is the env
YouTrack needs to run in:

    CATALINA_OPTS="-XX:MaxPermSize=200M"

Check that Tomcat is up and running by pointing a browser to localhost:8080
Then it's time to deploy youtrack. I edited tomcat-users.xml to allow
the user "admin" (password "admin") to log into the applications manager app.
So point a browser to:

    http://localhost:8080/manager/html

And login with admin:admin. YouTrack requires some libraries, so you need to
download the jars and put them in $CATALINA_HOME/lib (I copied the jars in
youtrack/libs):

 - log4j
 - jetty-io
 - jetty-continuations
 - jetty-servlets
 - jetty-util
 - jetty-http

Then deploy the YouTrack app with the provided manager. I had to disable (-1)
the max-file-size check, because the YouTrack.war archive is quite big:

    <multipart-config>
      <!-- 50MB max -->
      <max-file-size>-1</max-file-size>
      <max-request-size>-1</max-request-size>
      <file-size-threshold>0</file-size-threshold>
    </multipart-config>

and uploaded the WAR via its nice GUI. Then start the context and point the
browser to the YouTrack context path to configure it.


Installation log: TeamCity
==========================

TeamCity is easier to run. You can use the management script in bin/:

    bin/teamcity {start|stop}
    
TeamCity can be reached at http://localhost:8111


TeamCity and YouTrack integration
===================================
Both the tracker and the CI must be configured to see each other. You need to
add a youtrack user among the TeamCity users with View access, and then you need
to add a teamcity user to the YouTrack users (this makes TeamCity able to build
links pointing back to YouTrack, but you have to configure TeamCity to do this).
Then:

- Create and configure a project in YouTrack
- Create and configure a project and a build in TeamCity
- Configure YouTrack to find the TeamCity server, and define the mapping between
  tracked projects and TeamCity's ones

