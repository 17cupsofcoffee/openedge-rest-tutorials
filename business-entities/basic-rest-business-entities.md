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

## REST Annotations
While at this point, the business entity is functional (if somewhat minimal), it is still a good idea to have an
understanding of exactly how OpenEdge creates the link between your class and the REST service so that you
can convert/update existing logic in your application - the only difference between a standard business entity
and a REST business entity is that there are extra ABL annotations applied, so exposing existing functionality
is not too difficult a task.

### File Annotations
```ABL
@program FILE(name="<Class Filename>", module="AppServer").
@openapi.openedge.export FILE(type="REST", executionMode="singleton", useReturnValue="false", writeDataSetBeforeImage="false").
@progress.service.resource FILE(name="<Resource Name>", URI="/<Resource Name>", schemaName="<DataSet Variable Name>", schemaFile="<Path to Schema File>").
```
* `@program` registers Meta Catalog data - this is unimportant to understand fully, but it is used to link things
up behind the scenes.
* `@openapi.openedge.export` denotes that you wish to allow this class to be invoked from an external service -
in this case REST. You may wish to customize the `executionMode` parameter, which determines how the code will be
executed; however, the others should be left as their defaults. The options available for `executionMode` are:
  * `single-run` creates a new instance of the class when a method is called, then disposes of it once the method
  returns.
  * `singleton` keeps an instance of the class in memory, then uses it to service all requests to the resource. This
  may perform better than a `single-run` class (due to it only needing to be instantiated once), but note
  that despite there being a persistant instance, you should not store state between calls unless completely
  necessary; REST is intended to be a stateless communication method, with each HTTP request happening in isolation.
  * `external` specifies that the file only contains a single procedure or function to be called. This could be
  useful if exposing a piece of legacy code, but it is usually a better idea to use the object-oriented style.
* `@progress.service.resource` sets up the REST resource and the URL that should route to it. The properties to be
set on it are:
  * `name` should be the public facing name of the resource - this can be used as an identifier on the client-side to
  access server data, when using the JSDO library.
  * `URI` is the base URI that the resource will be available at, relative to the root of the service. The URIs of the
  class' methods will be appended to this value.
  * `schemaName` is the name of the variable within the file located at `schemaName` that contains the entity's DataSet.
  This will be used to expose the schema to the client, if you choose to use the JSDO library.
