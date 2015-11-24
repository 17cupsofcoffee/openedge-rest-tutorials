# Basic REST Business Entities
## Creating a New Entity
* In Progress Developer Studio, right-click your project's AppServer folder, then choose `New → Business Entity`.
* Enter a name for the business entity, and optionally a package to namespace it within, then click `Next >`.
* Enter a name for the resource that will be exposed via the REST service, if you want this to differ from the
business entity's name.
* Set which operations should be initially available on the entity, and whether or not a before-image is required
for the associated DataSet. All of the selected methods will be (by default) exposed to the REST service, so if
creating create/update/delete methods, ensure any neccerary permission checks are in place.
* Choose whether to create a schema based upon a database table or to use a pre-made schema file. If using the
former option, ensure that you remove any fields containing private data from the generated temp-table definition,
as anything listed will be exposed through the REST service.
* Ensure `Expose as Data Object service` is ticked, then click `Finish`.
  * *Note: Data Object projects/services were referred to as Mobile projects/services in OpenEdge versions prior
  to 11.6. More information on the changes of terminology can be found in the OpenEdge documentation
  [here](https://documentation.progress.com/output/ua/OpenEdge_latest/index.html#page/gspub/data-object-service-terminology-and-uri-differen.html).*
  * A new .cls file will be generated for your business entity, set up with basic REST capabilities.
  
