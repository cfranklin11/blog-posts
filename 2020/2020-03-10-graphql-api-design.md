* https://graphql.org/learn/thinking-in-graphs/
  * Naming
    * Per usual, name things carefully, especially for APIs that are public facing (your clients/users are unlikely to have the same naming conventions that your team does).
    * Renaming classes and methods is very easy, so no reason to copy the names of your data models or their methods.
    * Examples:
      * `Email` becomes `Seller`
      * Mutable `Order` becomes `Cart` (while we transition from everything being an `order`, mutable or complete, to `cart` and `order` being separate entities)
      * Namespace types, because their names are global (putting them in modules makes no difference)
  * Separate API & logic layers
    * API should be about handling requests & responses, independent of underlying business logic
    * Different APIs (REST, GraphQL, mobile, etc.) should depend on the same underlying business logic: no duplication
    * Examples:
      * Bad: much of our order logic is duplicated across three APIs: mobile, GraphQL, internal Rails controller
      * Good: shared authentication logic across APIs
      * Good: shared search (i.e. ElasticSearch) logic across internal controllers & GQL API
  * Model how users interact with the data, not the DB schema
    * Difficult to accomplish for open APIs that can be used in any number of ways
    * Think hard about API consumer UX (user journeys, common use cases) rather than data model
      * The point is to give the client the data they need in the form they need rather than forcing them to work with your data model and transform it with their own middleware (i.e. GQL API _is_ the middleware shifted from the frontend to the backend)
  * GraphQL is easy to expand, damn hard to contract
    * Adding new types/fields is not a breaking change; removing or changing existing ones is
  * Use Relay for the backend part of the API
    * Our initial assessment was that Relay was too restrictive, and we wanted more flexibility to work with different clients
    * Turns out that non-Relay-compatible API can only work with a non-Relay frontend, but a Relay-compatible API can work with Relay _and_ non-Relay frontends: Relay compatibility offers more, not less flexibility regarding client-side implementations
    * A bit steeper learning curve (edges, connections, and nodes, oh my!), but worth it for consistent implementation of pagination, UUIDs, and modeling of relationships between types via connections and edges objects
* https://blog.apollographql.com/designing-graphql-mutations-e09de826ed97
  * Consistent payload type that can be expanded as needed
    * Make both the returned object and error field nullable for greater flexibility
* https://blog.apollographql.com/graphql-schema-design-building-evolvable-schemas-1501f3c59ed5
  * Prefer object types over simple data types (e.g. a TimeRange type rather than a list of datetimes). They are easier to extend with new fields
  * Model your business domain, not your data
* https://medium.com/@__xuorig__/graphql-mutation-design-anemic-mutations-dd107ba70496
  * Focus query/mutation design on behaviours, not data (related to specificity above)
    * Generic queries/mutations force generic types/rules, which are less expressive (i.e. CRUD actions rather than actions that represent specific behaviours)
* https://blog.apollographql.com/using-nullability-in-graphql-2254f84c4ed7
  * Be careful with nullable vs non-nullable fields: non-nullable simplifies client code, but restricts future changes (i.e. introduces lots of potential breaking changes)
    * Example: We based non-nullability on ActiveRecord validations, but some old data (created before the validation was implemented) was in fact null in some data bases (we have multiple data bases, one for each client)
* https://github.com/Shopify/graphql-design-tutorial/blob/master/TUTORIAL.md
  * Err on the side of more-specific queries/mutations
    * Makes writing and typing inputs/responses easier
    * Gives clients more-meaningful query names rather than `doTheThing` and basing functionality on which inputs are sent
* Marketplacer lessons learned
  * Example of Order/Cart migration + hiding complexity from client
    * Hacking `object_from_id` and `id` field in individual types
    * Joooohn Fucking Maddeeeen
