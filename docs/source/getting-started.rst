.. _getting-started:

***************
Getting Started
***************


WireMock is distributed in two flavours - a standard JAR containing just WireMock, and a standalone fat JAR containing WireMock plus all its dependencies.

The fat JAR's dependencies are shaded i.e. they are hidden in alternative packages. This allows WireMock to be used in projects with conflicting versions of its dependencies. The standalone JAR is also runnable (discussed in more detail below).


Maven
=====
To add the standard WireMock JAR as a project dependency, put the following in the dependencies section of your POM:

.. code-block:: xml

    <dependency>
        <groupId>com.github.tomakehurst</groupId>
        <artifactId>wiremock</artifactId>
        <version>2.0.8-beta</version>
    </dependency>

Or to use the standalone JAR:

.. code-block:: xml

    <dependency>
        <groupId>com.github.tomakehurst</groupId>
        <artifactId>wiremock-standalone</artifactId>
        <version>2.0.8-beta</version>
    </dependency>


Gradle
======


.. code-block:: groovy

    testCompile "com.github.tomakehurst:wiremock:2.0.8-beta"

Or

.. code-block:: groovy

    testCompile "com.github.tomakehurst:wiremock-standalone:2.0.8-beta"


JUnit 4.x
=========
To use WireMock's fluent API add the following import:

.. code-block:: java

    import static com.github.tomakehurst.wiremock.client.WireMock.*;

WireMock ships with some JUnit rules to manage the server's lifecycle and setup/tear-down tasks. To start and stop WireMock per-test case, add the following to your test class (or a superclass of it):

.. code-block:: java

    @Rule
    public WireMockRule wireMockRule = new WireMockRule(8089); // No-args constructor defaults to port 8080


Now you're ready to write a test case like this:

.. code-block:: java

    @Test
    public void exampleTest() {
        stubFor(get(urlEqualTo("/my/resource"))
                .withHeader("Accept", equalTo("text/xml"))
                .willReturn(aResponse()
                    .withStatus(200)
                    .withHeader("Content-Type", "text/xml")
                    .withBody("<response>Some content</response>")));

        Result result = myHttpServiceCallingObject.doSomething();

        assertTrue(result.wasSuccessFul());

        verify(postRequestedFor(urlMatching("/my/resource/[a-z0-9]+"))
                .withRequestBody(matching(".*<message>1234</message>.*"))
                .withHeader("Content-Type", notMatching("application/json")));
    }

For many more examples of JUnit tests look no further than `WireMock's own acceptance tests <https://github.com/tomakehurst/wiremock/tree/master/src/test/java/com/github/tomakehurst/wiremock>`_

For more details on verifying requests and stubbing responses, see :ref:`stubbing` and :ref:`verifying`

For more information on the JUnit rule see :ref:`junit-rule`.


Detailed configuration
======================

For a bit more control over the settings of the WireMock server created by the rule you can pass a fluently built
Options object to either (non-deprecated) rule's constructor:

.. code-block:: java

    import static com.github.tomakehurst.wiremock.core.WireMockConfiguration.wireMockConfig;
    ...

    @Rule
    public WireMockRule wireMockRule = new WireMockRule(wireMockConfig().port(8089).httpsPort(8443));


Random port numbers
-------------------
You can have WireMock (or more accurately the JVM) pick random, free HTTP and HTTPS ports (which is a great idea if you
want to run your tests concurrently):

.. code-block:: java

    @Rule
    public WireMockRule wireMockRule = new WireMockRule(wireMockConfig().dynamicPort().dynamicHttpsPort());


Then find out which ports to use from your tests as follows:

.. code-block:: java

    int port = wireMockRule.port();
    int httpsPort = wireMockRule.httpsPort();



Non-JUnit and general Java usage
================================

If you're not using JUnit or neither of the WireMock rules manage its lifecycle in a suitable way you can construct and start the
server directly:

.. code-block:: java

    WireMockServer wireMockServer = new WireMockServer(wireMockConfig().port(8089)); //No-args constructor will start on port 8080, no HTTPS
    wireMockServer.start();

    // Do some stuff

    WireMock.reset();

    // Finish doing stuff

    wireMockServer.stop();

If you've changed the port number and/or you're running the server on another host, you'll need to tell the client:

.. code-block:: java

    WireMock.configureFor("wiremock.host", 8089);

And if you've deployed it into a servlet container under a path other than root you'll need to set that too:

.. code-block:: java

    WireMock.configureFor("tomcat.host", 8080, "/wiremock");


Running standalone
==================

The WireMock server can be run in its own process, and configured via the Java API, JSON over HTTP or JSON files.

This will start the server on port 8080:

.. parsed-literal::

    $ java -jar wiremock-|version|-standalone.jar

You can `download the standalone JAR from here <http://repo1.maven.org/maven2/com/github/tomakehurst/wiremock-standalone/2.0.8-beta/wiremock-standalone-2.0.8-beta.jar>`_.

See :ref:`running-standalone` for more details and commandline options.


Fetching all of your stub mappings (and checking WireMock is working)
---------------------------------------------------------------------
A GET request to the root admin URL e.g ``http://localhost:8080/__admin`` will return all currently registered stub mappings. This is a useful way to check
whether WireMock is running on the host and port you expect:



Deploying into a servlet container
==================================

WireMock can be packaged up as a WAR and deployed into a servlet container, with some caveats:
fault injection and browser proxying won't work, __files won't be treated as a docroot as with standalone,
the server cannot be remotely shutdown, and the container must be configured to explode the WAR on deployment.
This has only really been tested in Tomcat 6 and Jetty, so YMMV. Running standalone is definitely the preferred option.

The easiest way to create a WireMock WAR project is to clone the `sample app <https://github.com/tomakehurst/wiremock/tree/master/sample-war>`_

Deploying under a sub-path of the context root
----------------------------------------------
If you want WireMock's servlet to have a non-root path, the additional init param ``mappedUnder`` must be set with the sub-path
web.xml (in addition to configuring the servlet mapping appropriately).

See `the custom mapped WAR example <https://github.com/tomakehurst/wiremock/blob/master/sample-war/src/main/webappCustomMapping/WEB-INF/web.xml>`_ for details.

