# Implementing IHybridRealm
## Notes
* There are two versions of the `ValidatePassword` method that have to be implemented as part of IHybridClass:
  * `ValidatePassword(INTEGER, CHARACTER)` are used when the server is configured to receive passwords in clear-text.
  * `ValidatePassword(INTEGER, CHARACTER, CHARACTER, CHARACTER)` is used when the server is configured to receive passwords in HTTP Digest format.
  
  Which one of these is used is decided by the `realmPwdAlg` setting in the Spring configuration file (as described in ['Configuring OERealm Authentication'](https://github.com/17cupsofcoffee/openedge-rest-tutorials/blob/master/authentication/configuring-oerealm-authentication.md)). As only one can be active at once, it is safe (and probably a good security practice) to leave the unused function as a stub.
