## Deployment
    removeParent: WADevelopmentConfiguration instance
    addParent: WADevelopmentConfiguration instance
application := WADispatcher default handlerAt: 'config'.
application configuration 
    addParent: WAAuthConfiguration instance.
application
    preferenceAt: #login put: 'admin';
    preferenceAt: #passwordHash put: (GRPlatform current secureHashFor: 'seaside').
application
    addFilter: WAAuthenticationFilter new.
     WAAdmin register: self asApplicationAt: 'config' user: 'admin' password: 'seaside'
    preferenceAt: #resourceBaseUrl
    put: 'http://www.seaside.st/resources/'
    preferenceAt: #serverProtocol put: 'http';
    preferenceAt: #serverHostname put: 'localhost';
    preferenceAt: #serverPort put: 8080;
    preferenceAt: #serverPath put: '/'
$ sudo apt-get upgrade
Reading package lists... Done
Building dependency tree... Done
0 upgraded, 0 newly installed, 0 to remove and 0 not upgraded.
$ sudo update-rc.d apache2 defaults
$ squeakvm -vm-display-null imagename.image

# settings
USER="www-data"
VM="/usr/bin/squeakvm"
VM_PARAMS="-mmap 256m -vm-sound-null -vm-display-null"
IMAGE="seaside.image"

# start the vm
exec \
    setuidgid "$USER" \
    "$VM" $VM_PARAMS "$IMAGE"
$ sudo ./run
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Strict//EN"
"http://www.w3.org/TR/xhtml1/DTD/xhtml1-strict.dtd"><html
xmlns="http://www.w3.org/1999/xhtml" xml:lang="en" lang="en">
<head><title>Dispatcher at /</title>
...
	22315 ?        00:17:49 squeakvm
$ lsof -p 22315 | grep LISTEN
squeakvm 22315 www-data  7u  IPv4 411140468  TCP *:webcache (LISTEN)
squeakvm 22315 www-data  8u  IPv4 409571873  TCP *:5900 (LISTEN)
$ ln -s /srv/appname .
appname: up (pid 4165) 8 seconds
$ a2enmod proxy_http 
$ a2enmod rewrite 
    # set server name
    ProxyPreserveHost On
    ServerName www.appname.com

    # rewrite incoming requests
    RewriteEngine On
    RewriteRule ^/(.*)$ http://localhost:8080/appname/$1 [proxy,last]

</VirtualHost>
$ ln -s /etc/apache2/sites-available/appname.conf .
$ sudo apache2ctl restart

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
   BalancerMember http://127.0.0.1:8881 route=first
   BalancerMember http://127.0.0.1:8882 route=second
</Proxy>
=ProxyPassReverse / http://127.0.0.1:8881/
=ProxyPassReverse / http://127.0.0.1:8882/
<Location /balancer-manager>
   SetHandler balancer-manager
</Location>
<Location /server-status>
   SetHandler server-status
</Location>

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

    # set server name
    ProxyPreserveHost On
    ServerName www.appname.com

    # rewrite incoming requests
    RewriteEngine On
    RewriteRule ^/(.*)$ ajp://localhost:8003/appname/$1 [proxy,last]

</VirtualHost>
    squeaksource: 'MetacelloRepository';
    package: 'ConfigurationOfAjp';
    load.
(Smalltalk globals at: #ConfigurationOfAjp)
    project latestVersion load: 'AJP-Core'
     instanceVariableNames: ''
     classVariableNames: ''
     package: 'ManagementHandler'
     WAAdmin register: self at: 'manager'
    | source result |
    source := aRequestContext request
        at: 'code'
        ifAbsent: [ self error: 'Missing code' ].
    result := Compiler evaluate: source.
    aRequestContext respond: [ :response |
        response
            contentType: WAMimeType textPlain;
            nextPutAll: result asString ]
Pharo1.0rc1 of 19 October 2009 update 10492