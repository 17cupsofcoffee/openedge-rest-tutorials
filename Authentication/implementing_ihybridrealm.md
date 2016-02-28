# Implementing IHybridRealm
## Notes
* The following methods are, by default, only used by Progress BPM's admin panel, and can be safely left as unimplemented stubs if you do not plan to use that product:
  * `GetAttributeNames`
  * `GetUserNames`
  * `GetUserNamesByQuery`
  * `RemoveAttribute`
  * `SetAttribute`
* If not using the built-in Progress role system, `GetAttribute` needs to return static values for certain attribute names - it can't entirely be stubbed like the other functions, as it is called on every HTTP request. This can be implemented like so:

  ```ABL
  METHOD PUBLIC CHARACTER GetAttribute(INPUT theUserId AS INTEGER, INPUT attrName AS CHARACTER):
    DEFINE VARIABLE retVal AS CHARACTER NO-UNDO.
  
    IF THIS-OBJECT:ValidateClient() = FALSE THEN
      UNDO, THROW NEW Progress.Lang.AppError("Unauthorized client", 1).
  
    CASE attrname:
      WHEN "ATTR_ROLES"   THEN retval = "PSCUser".
      WHEN "ATTR_ENABLED" THEN retVal = "".
      WHEN "ATTR_LOCKED"  THEN retval = "".
      WHEN "ATTR_EXPIRED" THEN retval = "".
    END CASE.
         
    RETURN retVal.
  END METHOD.
  ```
  The return value for `ATTR_ROLES` should be whatever is set in the `grantedAuthorities` property of `oeablSecurity-basic-oerealm.xml`/`oeablSecurity-file-oerealm.xml`, minus the `rolePrefix`:
  ```xml
  <b:bean id="OERealmUserDetails"
    class="com.progress.appserv.services.security.OERealmUserDetailsImpl">
    ...
    <b:property name="grantedAuthorities" value="ROLE_PSCUser" />
    <b:property name="rolePrefix" value="ROLE_" />
    ...
  </b:bean>
  ```
* There are two versions of the `ValidatePassword` method that have to be implemented as part of IHybridClass:
  * `ValidatePassword(INTEGER, CHARACTER)` are used when the server is configured to receive passwords in clear-text.
  * `ValidatePassword(INTEGER, CHARACTER, CHARACTER, CHARACTER)` is used when the server is configured to receive passwords in HTTP Digest format.
  
  Which one of these is used is decided by the `realmPwdAlg` setting in the Spring configuration file (as described in [Configuring OERealm Authentication](configuring_oerealm_authentication.md)). As only one can be active at once, it is safe (and probably a good security practice) to leave the unused function as a stub.
