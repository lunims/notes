* GraphQL is a query language used by Web APIs as an alternative to REST
* single endpoint that handles all queries

## Basic Overview
* Most commonly: `/graphql` or `/api/graphql`
* GraphQL queries select `fields` of objects
* each object is of a specific `type`
* We can query the `id`, `username` and `role` fields of all `User` objects by running the `users` query:
```graphql
{ 
	users { 
		id 
		username 
		role 
	} 
}
```