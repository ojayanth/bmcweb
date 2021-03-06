# OpenBMC webserver #

This component attempts to be a "do everything" embedded webserver for openbmc.


## Capabilities ##
At this time, the webserver implements a few interfaces:
+ Authentication middleware that supports cookie and token based authentication, as well as CSRF prevention backed by linux PAM authentication credentials.
+ An (incomplete) attempt at replicating phosphor-dbus-rest interfaces in C++.  Right now, a few of the endpoint definitions work as expected, but there is still a lot of work to be done.  The portions of the interface that are functional are designed to work correctly for phosphor-webui, but may not yet be complete.
+ Replication of the rest-dbus backend interfaces to allow bmc debug to logged in users.
+ An initial attempt at a read-only redfish interface.  Currently the redfish interface targets ServiceRoot, SessionService, AccountService, Roles, and ManagersService.  Some functionality here has been shimmed to make development possible.  For example, there exists only a single user role.
+ SSL key generation at runtime.  See the configuration section for details.
+ Static file hosting.  Currently, static files are hosted from the fixed location at /usr/share/www.  This is intended to allow loose coupling with yocto projects, and allow overriding static files at build time.
+ Dbus-monitor over websocket.  A generic endpoint that allows UIs to open a websocket and register for notification of events to avoid polling in single page applications.  (this interface may be modified in the future due to security concerns.

## Configuration

BMCWeb is configured by setting `-D` flags that correspond to options
in `bmcweb/CMakeLists.txt` and then compiling.  For example, `cmake
-DBMCWEB_ENABLE_KVM=NO ...` followed by `make`.  The option names
become C++ preprocessor symbols that control which code is compiled
into the program.

When BMCWeb starts running, it reads persistent configuration data
(such as UUID and session data) from a local file.  If this is not
usable, it generates a new configuration.

When BMCWeb SSL support is enabled and a usable certificate is not
found, it will generate a self-sign a certificate before launching the
server.  The keys are generated by the `prime256v1` algorithm.  The
certificate
 - is issued by `C=US, O=OpenBMC, CN=testhost`,
 - is valid for 10 years,
 - has a random serial number, and
 - is signed using the `SHA-256` algorithm.

