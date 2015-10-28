# Implementing IHybridRealm
## Notes
* The properties and methods relating to retrieving and modifying 'attributes' are only utilized by Progress BPM's admin panel (in order to allow it to interact with the user database table). If you are not using that product, it is safe to stub out these parts of the class rather than fully implementing them.
* There are two versions of the `ValidatePassword` method that have to be implemented as part of IHybridClass:
  * `ValidatePassword(INTEGER, CHARACTER)` are used when the server is configured to receive passwords in clear-text.
  * `ValidatePassword(INTEGER, CHARACTER, CHARACTER, CHARACTER)` is used when the server is configured to receive passwords in HTTP Digest format.
  
  Which one of these is used is decided by the `realmPwdAlg` setting in the Spring configuration file (as described in ['Configuring OERealm Authentication'](https://github.com/17cupsofcoffee/openedge-rest-tutorials/blob/master/authentication/configuring-oerealm-authentication.md)). As only one can be active at once, it is safe (and probably a good security practice) to leave the unused function as a stub.
