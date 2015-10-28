# Implementing IHybridRealm
## Notes
* The following methods are, by default, only used by Progress BPM's admin panel, and can be safely left as unimplemented stubs if you do not plan to use that product:
  * `GetAttributeNames`
  * `GetUserNames`
  * `GetUserNamesByQuery`
  * `RemoveAttribute`
  * `SetAttribute`
* There are two versions of the `ValidatePassword` method that have to be implemented as part of IHybridClass:
  * `ValidatePassword(INTEGER, CHARACTER)` are used when the server is configured to receive passwords in clear-text.
  * `ValidatePassword(INTEGER, CHARACTER, CHARACTER, CHARACTER)` is used when the server is configured to receive passwords in HTTP Digest format.
  
  Which one of these is used is decided by the `realmPwdAlg` setting in the Spring configuration file (as described in ['Configuring OERealm Authentication'](https://github.com/17cupsofcoffee/openedge-rest-tutorials/blob/master/authentication/configuring-oerealm-authentication.md)). As only one can be active at once, it is safe (and probably a good security practice) to leave the unused function as a stub.
