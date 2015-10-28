# Configuring OERealm Authentication
## Setting Up The Pacific Application Server
The following steps only have to be carried out once per PASOE instance - however, the settings used will be required whenever you are configuring a new REST project, so keep note of them.

* In `<OpenEdge Work Folder>/<Server Name>/conf/jvm.properties`, add or modify the following setting:
```ini
-Dorg.apache.catalina.STRICT_SERVLET_COMPLIANCE=false
```
* Open a Proenv command prompt as administrator and run `cd <OpenEdge Work Folder>/<Server Name>/common/lib`.
* Generate a Client-Principal file using the `genspacp.bat` utility. This will be used to secure the HybridRealm class from being called by other PASOE clients.
```
genspacp -password <Password> -role <Role Name>

genspacp 1.0
Generated sealed Client Principal...
    User: BPSServer@OESPA
    Id: SmjnCQ1kTm2fY5r8pxQg5A
    Role: <Role Name>
    Encoded Password: <Encoded Password>
    File: oespaclient.cp
    State: SSO from external authentication system
    Seal is valid
```
* In `<OpenEdge Work Folder>/<Server Name>/openedge`, create a new plain-text file called `spaservice.properties`, and enter the following options. Some important things to note:
	* The `Password` refers to the encoded password returned by `genspacp`, **not** the plain-text one entered on the command-line.
	* **Make sure a blank line is left at the end of the file!** Not doing this can lead to strange errors when the file is being parsed.
```ini
Password=<Encoded Password>
Role=<Role Name>
DebugMsg=true
```
## Setting Up The REST Service
These steps need to be carried out every time you create a new project, regardless of whether there are other projects running on the same PASOE instance.

### Creating a REST Project
* Add the server to Progress Developer Studio and set up your project using either the REST or Mobile template - the former if you want fine-grained control over data mappings, or the latter if you wish to use the JSDO data transfer protocol.
* Create an implementation of IHybridRealm and a Properties class in `<Project Root>/AppServer/auth`. Examples can be found on [Progress KnowledgeBase](http://knowledgebase.progress.com/servlet/fileField?id=0BEa0000000LNZj).

### Configuring Spring Security
* In `<Project Root>/PASOEContent/WEB-INF/web.xml`, set `contextConfigLocation` to either `/WEB-INF/oeablSecurity-form-oerealm.xml` or `/WEB-INF/oeablSecurity-basic-oerealm.xml`, depending on which authentication method you want to utilize.
```xml
<context-param>
    <param-name>contextConfigLocation</param-name>
    <param-value>
<!-- USER EDIT: Select which application security model to employ
        /WEB-INF/oeablSecurity-basic-local.xml
        /WEB-INF/oeablSecurity-anonymous.xml
        /WEB-INF/oeablSecurity-form-local.xml
        /WEB-INF/oeablSecurity-container.xml
        /WEB-INF/oeablSecurity-basic-ldap.xml
        /WEB-INF/oeablSecurity-form-ldap.xml
        /WEB-INF/oeablSecurity-basic-oerealm.xml
        /WEB-INF/oeablSecurity-form-oerealm.xml
        /WEB-INF/oeablSecurity-form-saml.xml
        /WEB-INF/oeablSecurity-basic-saml.xml
-->
        /WEB-INF/oeablSecurity-basic-oerealm.xml
    </param-value>
</context-param>
```
* Open the configuration file that you specified in the previous step.
* Configure `OERealmAuthProvider` with the following settings.
	* `key` should be set to the encoded password returned by `genspacp`.
	* If you want to use the built-in OpenEdge user domains/database table, set `userDomain` to the name of the domain, and set `authz` to `true`. However, if you have a custom user table/authentication system, `userDomain` must be blank, and `authz` must be `false`. If you get this wrong, things will not work!
```xml
<b:bean id="OERealmAuthProvider"
        class="com.progress.appserv.services.security.OERealmAuthProvider" >
        <b:property name="userDetailsService">
                    <b:ref bean="OERealmUserDetails"/>
        </b:property>
        
        <b:property name="createCPAuthn" value="true" />
        <b:property name="multiTenant" value="false" />
        <b:property name="userDomain" value="" />
        <b:property name="key" value="<Encoded Password>" />
        <b:property name="authz" value="false" />
</b:bean>
```
* Configure `OERealmUserSettings` with the following settings.
	* `realmURL` should be set to the URL of your ABL application's APSV transport, which will take the format shown in the example below. Ensure the APSV transport is enabled by following the steps outlined in the [documentation](http://documentation.progress.com/output/ua/OpenEdge_latest/index.html#page/ompas/managing-apsv-transports.html), as the server will not connect without it.
	* `realmClass` should be set to the namespaced path of your HybridRealm class. If you are using the example implementation, this will probably be `auth.HybridRealm`.
	* `realmPwdAlg` sets the algorithm that the server should use to validate the password. If the value is `0`, the password will be sent to the server in clear-text - unless you are using SSL to connect, this is almost certainly insecure, as it makes it possible for the password to be intercepted by packet-sniffer software. If the value is `3`, the password will be sent using HTTP Digest, which encodes the password on the client side before sending it to the server. Unless your backend absolutely requires a different approach, using Digest authentication is recommended.
	* `realmTokenFile` should be set to the filename of the client-principal file generated by `genspacp`. Unless you went out of your way to rename it, this should be `oespaclient.cp`.

```xml
<b:bean id="OERealmUserDetails"
        class="com.progress.appserv.services.security.OERealmUserDetailsImpl" >
        <b:property name="realmURL" value="http://<Host Name>:<Port>/<Application Name>/apsv" />
        <b:property name="realmClass" value="<HybridRealm Class>" />
        <b:property name="grantedAuthorities" value="ROLE_PSCUser" />
        <b:property name="rolePrefix" value="ROLE_" />
        <b:property name="roleAttrName" value="ATTR_ROLES" />
        <b:property name="enabledAttrName" value="ATTR_ENABLED" />
        <b:property name="lockedAttrName" value="ATTR_LOCKED" />
        <b:property name="expiredAttrName" value="ATTR_EXPIRED" />
        <b:property name="realmPwdAlg" value="3" />
        <b:property name="realmTokenFile" value="oespaclient.cp" />
        <!-- For SSL connection to the oeRealm appserver provide the complete
             path of psccerts.jar as the value of 'certLocation' property
        -->
        <b:property name="certLocation" value="" />
        <!-- set appendRealmError = true in order to append the Realm 
        class thrown error in the error details send to the REST Client -->
        <b:property name="appendRealmError" value="false" /> 
    </b:bean>
```
