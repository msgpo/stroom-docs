# Stroom Components

Stroom is broken down into separate components. Each component encapsulates a specific area of Stroom functionality and aids development by providing a single area of focus for new features and ensures separate components remain as loosely coupled as possible. Components can be tested in isolation and their interaction with other components easily understood by only allowing dependencies via minimal APIs.

Some examples of components in Stroom include
* stroom-activity - Component for recording a users actions against a current activity
* stroom-dictionary - Component for storing lists of words.
* stroom-statistics - Component for recording statistical data, e.g. amount of data received in X minutes.

In the project structure a component appears as a first level subdirectory of the root project folder. Components have further subdirectories (modules) that make up the various parts of the component, e.g.

* `stroom` - root project
  * `stroom-activity` - component
    * `stroom-activity-api` - API module for `stroom-activity`
    * `stroom-activity-impl` - implementation of the API and other module implementation code
    * `stroom-activity-impl-db` - database persistence implementation used by impl
    * `stroom-activity-impl-db-jooq` - JOOQ generated classes used by `stroom-activity-impl-db`
    * `stroom-activity-impl-mock` - mock persistence for the `stroom-activity` component

## Component API, e.g. modules ending in `-api`

### API layer

All communication between components in stroom are made via the components API. The API provides the minimum surface area for communication between components and decouples dependencies between components to just the API code. For component testing purposes mock implementations of these APIs can be used for dependant components to limit testing to just a single component.

## Component API and service implementation, e.g. modules ending in `-impl`

### Client interaction - REST services and GWT Action Handlers

The uppermost layer of the server side code services requests from the client. The client may make restful calls as is the case for the new UI or will use Actions that are handles with ActionHandlers as is the case for the legacy GWT UI.

Since this layer deals with all client interaction it should be responsible for creating audit logs for all user activity, e.g. accessing documents, searching etc. No audit logging should need to be performed at a lower level within the application as deeper levels have less knowledge of user intent since they may just be playing a part in the wider request.

The client interaction layer adds no logic and asks the underlying service layer to service the encapsulated request away from the REST endpoint wrapping code or GWT action handler code. This allows multiple types of endpoint to use the same underlying service layer.

### Service layer

The service layer applies permission constraints to any requests being made so that only calls from identified and permitted users are allowed to proceed. The service layer performs all business logic and is responsible for all mutations of objects that will be persisted by the underlying persistence layer such as stamping objects to be updated with the current user and update time.

The service layer provides implementations for any API that the component may have.

The service layer provides the DAO (Data Access Object) API for the persistence layer to implement but maintains no knowledge of underlying persistence implementation, e.g. database queries.

## Persistence implementation, e.g. modules ending in `-impl-db`

### Persistence layer - DAOs

The persistence layer is an implementation of one or more DAOs specified in the service layer. The persistence layer provides no logic, it just stores and retrieves objects in a database or other persistence technology. If serialisation/de-serialisation is required in order to persist the object then that should also be performed by this layer so that no code above this layer has to care about this implementation detail.
