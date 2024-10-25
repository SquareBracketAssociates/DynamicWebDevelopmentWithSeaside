## Deployment
@cha:deploy

At some point you certainly want to go public with your web application. This means you need to find a server that is publicly reachable and that can host your Seaside application. If your application is successful, you might need to scale it to handle thousands of concurrent users. All this requires some technical knowledge. 

In Section *@ref:deployment-preparing@* we are going to have a look at some best practices before deploying an application.  Next, in Section *@ref:deployment-apache@* we present how to setup your own server using Apache. Last but not least, in Section *@ref:maintaining@*, we demonstrate ways to maintain a deployed image.

### Preparing for Deployment
@ref:deployment-preparing

Because Pharo offers you an image-based development environment, deploying your application can be as simple as copying your development image to your server of choice. However, this approach has a number of drawbacks. We will review a number of these drawbacks and how to overcome them.

#### Stripping down your image.


 The image you have been working in may have accumulated lots of code and tools that aren't needed for your final application; removing these will give you a smaller, cleaner image. How much you remove will depend on how many support tools you wish to include in your deployed image.
 
 @todo: we should mention the mini image

Alternatively, you may find it easier to copy your application code into a pre-prepared, \`stripped-down' image. For Pharo we have had good experiences using the [Pharo Core](http://www.pharo-project.org/) or Pharo minimal images.

#### Preparing Seaside.

 The first task in preparing Seaside for a server image is to remove all unused applications. To do this go to the _configuration_ application at [http://localhost:8080/config](http://localhost:8080/config) and click on _remove_ for all the entry points you don't need to be deployed. Especially make sure that you remove (or password protect) the _configuration_ application and the _code browser_ (at [http://localhost:8080/tools/classbrowser](http://localhost:8080/tools/classbrowser)), as these tools allow other people to access and potentially execute arbitrary code on your server.

@note The best way to deploy is to build a fresh image without the development, the tests and admin tool packages loaded in the first place. That will ensure nothing dangerous is there at all.

#### Disable Development Tools.

 If you still want the development tools loaded, then the best way is to remove `WADevelopmentConfiguration` from the shared configuration called "Application Defaults". You can do this by evaluating the code:

```
WAAdmin applicationDefaults
    removeParent: WADevelopmentConfiguration instance
```


You can always add it back by evaluating:

```
WAAdmin applicationDefaults
    addParent: WADevelopmentConfiguration instance
```


Alternatively you can use the configuration interface: In the configuration of any application select _Application Defaults_ from the list of the _Assigned parents_ in the _Inherited Configuration_ section and click on _Configure_. This opens an editor on the settings that are common to all registered applications. Remove `WAToolDecoration` from the list of _Root Decoration Classes_.

#### Password Protection.

 If you want to limit access to deployed applications make sure that you password protect them. To password protect an application do the following:

1. Click on _Configure_ of the particular entry point.
1. In the section _Inherited Configuration_ click select `WAAuthConfiguration` from the drop down box and click on _Add_. This will will add the authentication settings below.
1. Set _login_ and _password_ in the _Configuration_ section below.
1. Click on _Save_.


![Configure an application for deployment. % width=80&anchor=fig:configuration](figures/config.png)


If you want to programmatically change the password of the Seaside configure application, adapt and execute the following code:

```
| application |
application := WADispatcher default handlerAt: 'config'.
application configuration 
    addParent: WAAuthConfiguration instance.
application
    preferenceAt: #login put: 'admin';
    preferenceAt: #passwordHash put: (GRPlatform current secureHashFor: 'seaside').
application
    addFilter: WAAuthenticationFilter new.
```



Alternatively you can use the method `WAAdmin>>register:asApplicationAt:user:password:` to do all that for you when registering the application:

```
WAConfigurationTool class >> initialize
     WAAdmin register: self asApplicationAt: 'config' user: 'admin' password: 'seaside'
```


Next we have a look at the configuration settings relevant for deployment. Click on _Configure_ of the application you are going to deploy. If you don't understand all the settings described here, don't worry, everything will become clearer in the course of the following sections.

#### Resource Base URL.

 This defines the URL prefix for URLs created with  `WAAnchorTag>>resourceUrl:`. This setting prevents you having to duplicate the base-path for URLs to resource files all over your application. You will find this setting useful if you host your static files on a different machine than the application itself or if you want to quickly change the resources depending on your deployment scenario.

As an example, let's have a look at the following rendering code: `html image resourceUrl: 'logo-plain.png'`. If the resource base URL setting is set to [http://www.seaside.st/styles/](http://www.seaside.st/styles/), this will point to the image at [http://www.seaside.st/styles/logo-plain.png](http://www.seaside.st/styles/logo-plain.png). Note that this setting only affects URLs created with `WAImageTag>>resourceUrl:`, it does not affect the generated pages and URLs otherwise.

Set it programmatically with:

```
application
    preferenceAt: #resourceBaseUrl
    put: 'http://www.seaside.st/resources/'
```


#### Server Protocol, Hostname, Port, and Base Path.

 Seaside creates absolute URLs by default. This is necessary to properly implement HTTP redirects. To be able to know what absolute path to generate, Seaside needs some additional information and this is what these settings are about. These settings will be useful if you are deploying Seaside behind an external front-end web server like Apache.


![Configuration options for absolute URLs.](figures/deploy-url.png width=80&anchor=fig:deploy-url)


Have a look at *@fig:deploy-url@* to see visually how these settings affect the URL. _Server Protocol_ lets you change between `http` and `https` (Secure HTTP). Note that this changes only the way the URL is generated, it does not implement HTTPS -- if you want secure HTTP you need to pass the requests through a HTTPS proxy. _Server Hostname_ and _Server Port_ define the hostname and port respectively. In most setups, you can leave these settings undefined, as Seaside is able to figure out the correct preferences itself. If you are using an older web server such as Apache 1 you have to give the appropriate values here. _Server Path_ defines the URL prefix that is used in the URLs. Again, this only affects how the URL is generated, it does not change the lookup of the application in Seaside. This setting is useful if you want to closely integrate your application into an existing web site or if you want to get rid or change the prefix `/appname` of your applications.

Again, if you want to script the deployment, adapt and execute the following code:

```
application
    preferenceAt: #serverProtocol put: 'http';
    preferenceAt: #serverHostname put: 'localhost';
    preferenceAt: #serverPort put: 8080;
    preferenceAt: #serverPath put: '/'
```



### Deployment with Apache
@ref:deployment-apache

In this section we discuss a typical server setup for Seaside using Debian Linux as an operating system. Even if you are not on a Unix system, you might want to continue reading, as the basic principles are the same everywhere. Due to the deviation of different Linux and Apache distributions, the instructions given here cannot replace the documentation for your particular target system.

#### Preparing the server


Before getting started you need a server machine. This is a computer with a static IP address that is always connected to the Internet. It is not required that you have physical access to your machine. You might well decide to host your application on a virtual private server (VPS). It is important to note that you require superuser-level access to be able to run Smalltalk images. The ability to execute PHP scripts is not enough.

We assume that your server already has a working Linux installation. 
In the following sections, we use Debian Linux 5.0 (Lenny), however with 
minor adjustments you will also be able to deploy applications on other 
distributions. 

Before starting with the setup of your application, it is important to make sure that your server software is up-to-date. It is crucial that you always keep your server up to the latest version to prevent malicious attacks and well-known bugs in the software. 

To update your server execute the following commands from a terminal. Most commands we use in this chapter require administrative privileges, therefore we prepend them with sudo (super user do):

```
$ sudo apt-get update
$ sudo apt-get upgrade
Reading package lists... Done
Building dependency tree... Done
0 upgraded, 0 newly installed, 0 to remove and 0 not upgraded.
```


#### Installing Apache


Next we install Apache 2.2, an industry-leading open-source web server. Depending on your requirements you might decide to install a different web server. Lighttpd, for example, might be better suited in a high-performance environment.

Some people prefer to use one of the web servers written in Smalltalk. This is a good choice during development and prototyping, as such a server is easy to set up and maintain. We strongly discourage to use such a setup for production applications due to the following reasons:
 
- The web server is something accessible from outside the world and therefore exposed to malicious attacks. Apache is a proven industry standard used by more than 50% (depending on the survey) of all of today's web sites.
- To listen on port 80, the standard port used by the HTTP protocol, the web server needs to run as root. Running a public service as root is a huge security issue. Dedicated web servers such as Apache drop their root privileges after startup. This allows them to listen to port 80 while not being root. Unfortunately, this is not something that can be easily done from within the Smalltalk VM.
- Smalltalk is relatively slow when reading files and processing large amounts of data (the fact that everything is an object is rather a disadvantage in this case). A web server running natively on the host platform is always faster by an  order of magnitude. A standalone web server can take advantage of the underlying operating system and advise it to directly stream data from the file-system to the socket as efficiently as possible. Furthermore, web servers usually provide highly efficient caching strategies.
- Most of today's Smalltalk systems (with the exception of GemStone) are single-threaded. This means that when your image is serving files, Seaside is blocked and cannot produce dynamic content at the same time. On most of today's multi-core systems, you get much better performance when serving static files through Apache running in parallel to your Seaside application server.
- External web servers integrate well with the rest of the world. Your web application might need to integrate into an existing site. Often a web site consists of static as well as dynamic content provided by different technologies. The seamless integration of all these technologies is simple with Apache.


Let's go and install Apache then. If you are running an older version you might want to consider upgrading, as it makes the integration with Seaside considerably simpler, although it is not strictly necessary.

```
$ sudo apt-get install apache2
```


Ensure the server is running and make it come up automatically when the machine boots:

```
$ sudo apache2 -k restart
$ sudo update-rc.d apache2 defaults
```


#### Installing the VM


!!todo update 

Depending on the Smalltalk dialect you are using, the installation of 
the VM is different. Installing Pharo on a Debian system is simple. 
Install Pharo by entering the following command on the terminal: 

```
$ sudo apt-get install squeak-vm
```


Note that installing and running Squeak does not require you to have the _X Window System_ installed. Just tell the installer not to pull these dependencies in, when you are asked for it. Squeak remains runnable headless without a user-interface, this is what you want to do on most servers anyway. 

Now you should be able to start the VM. Typing the `squeak` command executes a helper script that allows one to install new images and sources in the current directory, and run the VM. The VM itself can be started using the `squeakvm` command.

```
$ squeakvm -help
$ squeakvm -vm-display-null imagename.image
```



You can find additional help on starting the VM and the possible command line parameters in the man pages:

```
$ man squeak
```


In the next section we are going to look at how we can run the VM as a daemon.

#### Running the VM


Before we hook up the Pharo side with the web server, we need a reliable way to start and keep the Smalltalk images running as daemon (background process). We have had positive experience using the _daemontools_, a free collection of tools for managing UNIX service written by Daniel J. Bernstein. Contrary to other tools like `inittab`, `ttys`, `init.d`, or `rc.local`, the `daemontools` are reliable and easy to use. Adding a new service means linking a directory with a script that runs your VM into a centralized place. Removing the service means removing the linked directory.

Type the following command to install daemontools. Please refer to official website ([http://cr.yp.to/daemontools.html](http://cr.yp.to/daemontools.html)) for additional information.

```
$ apt-get install daemontools-run
```


On the server create a new folder to carry all the files of your Seaside application. We usually put this folder into a subdirectory of `/srv` and name it according to our application `/srv/appname`, but this is up to you. Copy the deployment image you prepared in  into that directory. Next we create a `run` script in the same directory:

```
#!/bin/bash

# settings
USER="www-data"
VM="/usr/bin/squeakvm"
VM_PARAMS="-mmap 256m -vm-sound-null -vm-display-null"
IMAGE="seaside.image"

# start the vm
exec \
    setuidgid "$USER" \
    "$VM" $VM_PARAMS "$IMAGE"
```



On lines 3 to 7 we define some generic settings. `$USER` is the user of the system that should run your Smalltalk image. If you don't set a user here, the web service will run as root, something you must avoid. Make sure you have the user specified here on your system. `www-data` is the default user for web services on Debian systems. Make sure that this is not a user with root privileges. `$VM` defines the full path to the Squeak VM. If you have a different installation or Smalltalk dialect, you again need to adapt this setting. `$VM_PARAMS` defines the parameters passed to the Squeak VM. If you use a different environment, you need to consult the documentation and adapt these parameters accordingly. The first parameter `-mmap 256m` limits the dynamic heap size of the Squeak VM to 256 MB and makes Squeak VMs run more stably. `-vm-sound-null` disables the sound plugin, something we certainly don't need on the server. `-vm-display-null` makes the VM run headless. This is crucial on our server, as we presumably don't have any windowing server installed.

The last four lines of the script actually start the VM with the given parameters. Line 11 changes the user id of the Squeak VM, and line 12 actually runs the Squeak VM.

To test the script mark it as executable and run it from the command line. You need to do this as superuser, otherwise you will get an error when trying to change the user id of the VM.

```
$ chmod +x ./run
$ sudo ./run
```


The VM should be running now. You can verify that by using on of the UNIX console tools. In the following example we assume that the image has a web server installed and is listening on port 8080:

```
$ curl http://localhost:8080
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Strict//EN"
"http://www.w3.org/TR/xhtml1/DTD/xhtml1-strict.dtd"><html
xmlns="http://www.w3.org/1999/xhtml" xml:lang="en" lang="en">
<head><title>Dispatcher at /</title>
...
```



You might want to change the URL to point to your application. As long as you don't get an error message like `curl: (7) couldn't connect to host` everything is fine. You can install curl using 

```
apt-get install curl
```


#### Troubleshooting the VM.

 You may encounter some common problems at this point. One of these problems is that the VM is unable to find or read the image, change or source files. Make sure that all these files are in the same directory and that their permissions are correctly set so that the user `www-data` can actually read them. Also, ensure that the image has been saved with the web server running on the correct port.

!!important Pharo displays an error message when the `.changes` or `.sources` file cannot be found or accessed. Unfortunately, this message is not visible from the console, but only pops up virtually in the headless window of the VM. The modal dialog prevents the VM from starting up the server and thus the image remains unreachable. To solve the problem make sure that `.changes` and `.sources` files are in the same directory as the image-file and that all files can be read by the user `www-data`. Another possibility (if you really do not want to distribute the files) is to disable the missing files warning using:

```
Preferences disable: #warnIfNoSourcesFile
```


A valuable tool to further troubleshoot a running but otherwise not responding VM is `lsof`, an utility that lists opened files and sockets by process. You can install it using `apt-get install lsof`. Use a different terminal to type the following commands:

```
$ ps -A | grep squeakvm
	22315 ?        00:17:49 squeakvm
$ lsof -p 22315 | grep LISTEN
squeakvm 22315 www-data  7u  IPv4 411140468  TCP *:webcache (LISTEN)
squeakvm 22315 www-data  8u  IPv4 409571873  TCP *:5900 (LISTEN)
```



With the first line,  we find out the process id of the running Squeak VM. `lsof -p 22315` lists all the open files and sockets of process `22315`. Since we are only interested in the sockets Squeak is listening on we grep for the string `LISTEN`. In this case we see that a web server is correctly listening on port 8080 (webcache) and that another service is listening on port 5900. In fact the latter is the RFB server, which we will discuss in .

In the original terminal, press Ctrl+C to stop the VM. 

#### Starting the service.

Go to `/service` and link the directory with your run-script in there, to let _daemontools_ automatically start your image.

Go to `/etc/service` (on other distributions this might be `/service`) and link 
the directory with your run-script in there, to let _daemontools_ automatically start your image.

```
$ cd /service
$ ln -s /srv/appname .
```


You can do an `svstat` to see if the new service is running correctly:

```
$ svstat *
appname: up (pid 4165) 8 seconds
```


The output of the command tells you that the service `appname` is up and running for 8 seconds with process id 4165. From now on _daemontools_ makes sure that the image is running all the time. The image gets started automatically when the machine boots and -- should it crash -- it is immediately restarted.

#### Stopping the service.

 To stop the image unlink the directory from `/service` and terminate the service.

 ```
$ rm /etc/service/appname
$ cd /srv/appname 
$ svc -t .
```

Note that only unlinking the service doesn't stop it from running, it is just taking it away from the pool of services. Also note that terminating the service without first unlinking it from the service directory will cause it to restart immediately. _daemontools_ is very strict on that and will try to keep all services running all the time.

As you have seen starting and stopping an image with _daemontools_ is simple and can be easily scripted with a few shell scripts.

#### Configuring Apache


Now we have all the pieces to run our application. The last remaining thing to do is to get Apache configured correctly. In most UNIX distributions this is done by modifying or adding configuration files to `/etc/apache2`. On older systems the configuration might also be in a directory named `/etc/httpd`. 

The main configuration file is situated in `apache2.conf`, or `httpd.conf` on older systems. Before we change the configuration of the server and add the Seaside web application, have a look at this file. It is usually instructive to see how the default configuration looks like and there is plenty of documentation in the configuration file itself. On most systems this main configuration file includes other configuration files that specify what modules (plug-ins) are loaded and what sites are served through the web server.

#### Loading Modules.

On Debian the directory `/etc/apache2/mods-available` contains all the available modules that could be loaded. To make them actually available from your configuration you have to link (`ln`) the `.load` files to `/etc/apache2/mods-enabled`. We need to do that for the proxy and rewrite modules:

```
$ a2enmod proxy 
$ a2enmod proxy_http 
$ a2enmod rewrite 
```


If you are running an older version you might need to uncomment some lines in the main configuration file to add these modules.

#### Adding a new site.

 Next we add a new site. The procedure here is very similar to the one of the modules, except that we have to write the configuration file ourselves. `/etc/apache2/sites-available/` contains configuration directives files of different virtual hosts that might be used with Apache 2. `/etc/apache2/sites-enabled/` contains links to the sites in `sites-enabled/` that the administrator wishes to enable. 

**Step 1.** In `/etc/apache2/sites-available/` create a new configuration file called `appname.conf` as follow. 

```
<VirtualHost *>
    # set server name
    ProxyPreserveHost On
    ServerName www.appname.com

    # rewrite incoming requests
    RewriteEngine On
    RewriteRule ^/(.*)$ http://localhost:8080/appname/$1 [proxy,last]

</VirtualHost>
```


The file defines a default virtual host. If you want to have different virtual hosts (or domain names) to be served from the same computer replace the `*` in the first line with your domain name. The domain name that should be used to generate absolute URLs is specified with `ServerName`. The setting `ProxyPreserveHost` enables Seaside to figure out the server name automatically, but this setting is not available for versions prior to Apache 2. In this case you have to change the Seaside preference \`hostname' for all your applications manually, see 

 `RewriteEngine` enables the rewrite engine that is used in the line below. The last line actually does all the magic and passes on the request to Seaside. A rewrite rule always consists of 3 parts. The first part matches the URL, in this case all URLs are matched. The second part defines how the URL is transformed. In this case this is the URL that you would use locally when accessing the application. And finally, the last part in square brackets defines the actions to be taken with the transformed URL. In this case we want to proxy the request, this means it should be passed to Squeak using the new URL. We also tell Apache that this is the last rule to be used and no further processing should be done.

**Step 2.** Now link the file from `/etc/apache2/sites-enabled/` and restart Apache:

```
$ cd /etc/apache2/sites-enabled
$ ln -s /etc/apache2/sites-available/appname.conf .
$ sudo apache2ctl restart
```


If the domain name `appname.com` is correctly setup and pointing to your machine, everything should be up and running now. Make sure that you set the \`server base path' in the application configuration to `/`, so that Seaside creates correct URLs.

**Troubleshooting the Proxy.** On some systems the above configuration may not suffice to get Seaside working. Check the error\_log and if you get an error messages in the Apache log saying `client denied by server configuration` just remove the file `/etc/mods-enabled/proxy.conf` from the configuration.



### Serving File with Apache


Most web applications consist of serving static files at one point or the other. These are all files that don't change over time, as opposed to the XHTML of the web applications. Common static files are style sheets, Javascript code, images, videos, sound or simply document files. As we have seen in Chapter  such files can be easily served through the image, however the same drawbacks apply here as those listed in .

The simplest way to use an external file server is to overlay a directory tree on the hard disk of the web server over the Seaside application. For example, when someone requests the file [http://www.appname.com/seaside.png](http://www.appname.com/seaside.png) the server serves the file `/srv/appname/web/seaside.png`. With Apache this can be done with a few additional statements in the configuration file:

```
<VirtualHost *>

    # set server name
    ProxyPreserveHost On
    ServerName www.appname.com
    
    # configure static file serving
    DocumentRoot /srv/appname/web
    <Directory /srv/appname/web>
        Order deny,allow
        Allow from all
    </Directory>
    
    # rewrite incoming requests
    RewriteEngine On
    RewriteCond /srv/appname/web%{REQUEST_FILENAME} !-f
    RewriteRule ^/(.*)$ http://localhost:8080/appname/$1 [proxy,last]

</VirtualHost>
```


The added part starts line 9 with the  comment `#configure static file serving`. Line 9 and following mark a location on the local harddisc to be used as the source of files. So when someone requests the file [http://www.appname.com/seaside.png](http://www.appname.com/seaside.png) Apache will try to serve the file found at `/srv/appname/web/seaside.png`. For security reasons, the default Apache setup forbids serving any files from the local hard disk, even if the filesystem permissions allow the process to access these files. With the lines between `Directory` and `Directory` we specify that Apache can serve all files within `/srv/appname/web`. There are many more configuration options available there, so check out the Apache documentation.

The next thing we have to do is add a condition in front of our rewrite rule. As you certainly remember, this rewrite rule passes (proxies) all incoming requests to Seaside. Now, we would only like to do this if the requested file does not exist on the file-system. To take the previous example again, if somebody requests [http://www.appname.com/seaside.png](http://www.appname.com/seaside.png) Apache should check if a file named `/srv/appname/web/seaside.png` exists. 
This is the meaning of the line 16 where `%{REQUEST_FILENAME}` is a variable representing the file looked up, here the variable  `REQUEST_FILENAME` is bound to `seaside.png`. Furthermore, the cryptic expression `!-f` means that the following rewrite rule should conditionally be executed _if the file specified does not exist_. In our case, assuming the file `/srv/appname/web/seaside.png` exists, this means that the rewrite rule is skipped and Apache does the default request handling. Which is to serve the static files as specified with the `DocumentRoot` directive. Most other requests, assuming that there are only a few files in `/srv/appname/web`, are passed on to Seaside.

A typical layout of the directory `/srv/appname/web` might look like this:


| `favicon.ico` | A shortcut icon, which most graphical web browsers automatically  make use of. The icon is typically displayed next to the URL and within the list of bookmarks. |
| `robots.txt` | A robots exclusion standard, which most search engines request to get information on what parts of the site should be indexed. |
| `resources/` | A subdirectory of resources used by the graphical designer of your application. |
| `resources/css/` | All the CSS resources of your application. |
| `resources/script/` | All the external Javascript files of your application. |



### Load Balancing Multiple Images


There are several ways to load balance Seaside images, one common way is to use [mod\_proxy\_balancer](http://httpd.apache.org/docs/2.2/mod/mod_proxy_balancer.html) that comes with Apache. Compared to other solutions it is relatively easy to set up, does not modify the response and thus has no performance impact. It requires to only load one small additional package to load into Seaside. Additionally `mod_proxy_balancer` provides some advanced features like a manager application, pluggable scheduler algorithms and configurable load factors.

When load balancing multiple Seaside images care must be taken that all requests that require a particular session are processed by the same image, because unless GemStone is used session do not travel between images. This has to work with and without session cookies. This is referred to as sticky sessions because a session sticks to its unique image. `mod_proxy_balancer` does this by associating each image with an route name. Seaside has to append this route to the session id. `mod_proxy_balancer` reads the route from the request and proxies to the appropriate image. This has the advantage that `mod_proxy_balancer` does not have to keep track of all the sessions and does not have to modify the response. Additionally the mapping is defined statically in the the Apache configuration and is not affected by server restarts.

First we need to define our cluster of images. In this example we use two images, the first one on port 8881 with the route name "first" the second on port 8882 with route name "second". We can of course choose other ports and route names as long as they are unique:

```
<Proxy balancer://mycluster>
   BalancerMember http://127.0.0.1:8881 route=first
   BalancerMember http://127.0.0.1:8882 route=second
</Proxy>
```



Next we need to define the actual proxy configuration, which is similar to what we do with a single Seaside image behind an Apache:

```
ProxyPass / balancer://mycluster/ stickysession=_s|_s nofailover=On
=ProxyPassReverse / http://127.0.0.1:8881/
=ProxyPassReverse / http://127.0.0.1:8882/
```

Note that we configure `_s` to be the session id for the URL and the cookie.

Finally, we can optionally add the balancer manager application:

```
ProxyPass /balancer-manager !
<Location /balancer-manager>
   SetHandler balancer-manager
</Location>
```

As well as an optional application displaying the server status:

```
ProxyPass /server-status !
<Location /server-status>
   SetHandler server-status
</Location>
```


Putting all the parts together this gives an apache configuration like the following:

```
<VirtualHost *>

   ProxyRequests Off
   ProxyStatus On
   ProxyPreserveHost On
   ProxyPass /balancer-manager !
   ProxyPass /server-status !
   ProxyPass / balancer://mycluster/ STICKYSESSION=_s|_s
   ProxyPassReverse / http://127.0.0.1:8881/
   ProxyPassReverse / http://127.0.0.1:8882/
   
   <Proxy balancer://mycluster>
      BalancerMember http://127.0.0.1:8881 route=first
      BalancerMember http://127.0.0.1:8882 route=second
   </Proxy>
   
   <Location /balancer-manager>
      SetHandler balancer-manager
   </Location>
   
   <Location /server-status>
      SetHandler server-status
   </Location>

</VirtualHost>
```


The rest can be configured from within Seaside. First we need to load the package `Seaside-Cluster` from [http://www.squeaksource.com/ajp/](http://www.squeaksource.com/ajp/). Then we need to configure each image individually with the correct route name. It is important that this matches the Apache configuration from above. The easiest way to do this is evaluate the expression:

```
WAAdmin makeAllClusteredWith: 'first'
```

in the image on port 8881 and

```
WAAdmin makeAllClusteredWith: 'second'
```

in the image on port 8882.


![Apache Load Balancer Manager. %width=80&anchor=fig:balancer-manager](figures/balancer-manager.png )

This is it, the cluster is ready to go. If enabled the server manager can be found at [http://localhost/balancer-manager](http://localhost/balancer-manager), see Figure *@fig:balancer-manager@*, and the server status can be queried on [http://localhost/server-status](http://localhost/server-status). Note that before going to production these two admin applications need to be properly protected from unauthorized access.

### Using AJP


AJPv13 is a binary protocol between Apache and Seaside. It was originally developed for Java Tomcat but there is nothing Java specific about it. Compared to conventional HTTP it has less overhead and better support for SSL.

Starting with version Apache 2.2 the required module  [mod_proxy_ajp](http://httpd.apache.org/docs/2.2/mod/mod_proxy_ajp.html)
is included with the default setup making it much simpler to use. The configuration looks almost the same as the one we saw in Section . The only difference is that you need to replace `proxy_http` with `proxy_ajp`, and that the protocol in the URL of the rewrite rule is `ajp` instead of `http`.

The adapted configuration looks like this:

```
<VirtualHost *>

    # set server name
    ProxyPreserveHost On
    ServerName www.appname.com

    # rewrite incoming requests
    RewriteEngine On
    RewriteRule ^/(.*)$ ajp://localhost:8003/appname/$1 [proxy,last]

</VirtualHost>
```


On the Smalltalk side you need to load the packages `AJP-Core` and `AJP-Pharo-Core` directly with Monticello from [http://www.squeaksource.com/ajp](http://www.squeaksource.com/ajp). More conveniently you can also use the following Metacello script:

```
Gofer new
    squeaksource: 'MetacelloRepository';
    package: 'ConfigurationOfAjp';
    load.
(Smalltalk globals at: #ConfigurationOfAjp)
    project latestVersion load: 'AJP-Core'
```


At that point you need to add and start the `AJPPharoAdaptor` on the correct port from within your image (8003 in this example) and your web application is up and running with AJP.


### Maintaining Deployed Images
@ref:maintaining

If you followed all the instructions up to now, you should have a working Seaside server based on Seaside, Apache and some other tools. Apache is handling the file requests and passing on other requests to your Seaside application server. This setup is straightforward and enough for smaller productive applications.

As your web application becomes widely used, you want to regularly provide fixes and new features to your users. Also, you might want to investigate and debug the deployed server. To do that, you need a way to get our hands on the running VM. There are several possibilities to do that, we are going to look at those in this section.

#### Headful Pharo


Instead of running the VM headless as we did previously, it is also possible to run it headful as you do during development. This is common practice on Windows servers, but it is rarely done on Unix. Normally servers doesn’t come with a windowing system for performance and security reasons. Managing a headful image is straightforward, so we will not be discussing this case further.

#### VNC


A common technique is to run a VNC server within your deployed image. VNC (Virtual Network Computing) is a graphical desktop sharing system, which allows one to visually control another computer. The server constantly sends graphical screen updates through the network using a remote frame buffer (RFB) protocol, and the client sends back keyboard and mouse events. VNC is platform-independent and there are several open-source server and client implementations available.

Pharo comes with a VNC client and server implementation, which can optionally be loaded. It is called _Remote Frame Buffer (RFB)_. Unfortunately, the project is not officially maintained anymore and the latest code is broken in Pharo, however you can get a working version from [http://source.lukas-renggli.ch/unsorted/](http://source.lukas-renggli.ch/unsorted/).

!!todo Update 

Install the RFB package, define a password and start the server. Now you are able to connect to the Pharo screen using any VNC client. Either using the built-in client from a different Pharo image, or more likely using any other native client. Now you are able to connect to the server image from anywhere in the world, and this even works if the image is started headless. This is very useful to be able to directly interact with server images, for example to update code or investigate and fix a problem in the running image.


### Deployment tools


Seaside comes with several tools that help you with the management of deployed applications. The tools included with the Pharo distribution of Seaside include:


| Configuration | [http://localhost:8080/config](http://localhost:8080/config) |
| System Status | [http://localhost:8080/status](http://localhost:8080/status) |
| Class Browser | [http://localhost:8080/tools/classbrowser](http://localhost:8080/tools/classbrowser) |
| Screenshot | [http://localhost:8080/tools/screenshot](http://localhost:8080/tools/screenshot) |
| Version Uploader | [http://localhost:8080/tools/versionuploader](http://localhost:8080/tools/versionuploader) |

**Configuration.** The Seaside configuration interface is described throughout this book, especially in the previous sections so we are not going to discuss this further.

**System Status.** The System status is a tool that provides useful information on the system. It includes information on the image (see *@fig:system-status@*), the virtual machine, Seaside, the garbage collector and the running processes.


![System Status Tool.](figures/system-status.png width=80&anchor=fig:system-status)



**Class Browser.** The class browser provides access the source code of the system and allows you to edit any method or class while your application is running.

**Screenshot.** The screenshot application provides a view into your image, even if it runs headless. Clicking on the screenshot even allows you to open menus and to interact with the tools within your deployed image.

**Version Uploader.** The version uploader is a simple interface to Monticello. It allows you to check what code is loaded into the image and gives you the possibility to update the code on the fly.

!!important If you plan to use any of these tools in a deployed image make sure that they are properly secured from unauthorized access. You don't want that any of your users accidentally stumble upon one of them and you don't want to give hackers the possibility to compromise your system.



### Request Handler


Another possibility to manage your headless images from the outside is to add a request handler that allows an administrator to access and manipulate the deployed application.

!!advanced The technique described here is not limited to administering images, but can also be used for public services and data access using a RESTful API. Many web applications today provide such a functionality to interact with other web and desktop application.

To get started we subclass `WARequestHandler` and register it as a new entry point:

```
WARequestHandler << #ManagementHandler
     package: 'ManagementHandler'
```


```
ManagementHandler class >> initialize
     WAAdmin register: self at: 'manager'
```


The key method to override is `#handleFiltered:`. If the URL `manager` is accessed, then the registered handler receives `aRequestContext` passed into this method and has the possibility to produce a response and pass it back to the web server.

The most generic way of handling this is to provide a piece of code that can be called to evaluate Pharo code within the image:

```
ManagementHandler >> handleFiltered: aRequestContext
    | source result |
    source := aRequestContext request
        at: 'code'
        ifAbsent: [ self error: 'Missing code' ].
    result := Compiler evaluate: source.
    aRequestContext respond: [ :response |
        response
            contentType: WAMimeType textPlain;
            nextPutAll: result asString ]
```


The first few lines of the code fetch the request parameter with the name code and store it into the temp `source`. Then we call the compiler to evaluate the code and store it into `result`. The last few lines generate a textual response and send it back to the web server.

Now you can go to a web server and send commands to your image by navigating to an URL like: [http://localhost:8080/manager?code=SystemVersion current](http://localhost:8080/manager?code=SystemVersion current). This will send the Smalltalk code `SystemVersion current` to the image, evaluate it, and send you back the result.

Alternatively, you might want to write some scripts that allow you to directly contact one or more images from the command line:

```
$ curl 'http://localhost:8080/manager?code=SystemVersion current'
Pharo1.0rc1 of 19 October 2009 update 10492
```


If you install a request handler like the one presented here in your application make sure to properly protect it from unauthorized access, see Section XXXX .

### Summary


We show how you can deploy your Seaside applications as well as maintained them over time. 