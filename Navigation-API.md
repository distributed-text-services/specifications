# Distributed Text Services - Navigation API

Navigation API replicates functionality of getValidRef in CTS. Its role is to provide a list of passages that are available for a given resource.

## URI 

## Query Parameters

| name | description                              | methods |
|------|------------------------------------------|---------|
| id   | identifier for a document |  GET    |
| passage | passage identifier (used together with `id`) | GET    |
| level | Depth for passages we want to retrieve identifiers of  | GET    |

## Response Headers

The response contains the following response headers:

| name | description |
|------|-------------|
| Link | Gives relation to next and previous pages |