# Projects

A [Project](projects.md) is a collection of related resources and outputs. Projects are also the primary delivery vehicle for consumption-based applications.

```http
POST   /v3/projects        // Creates a new project associated with an organization
GET    /v3/projects/:id    // Get a project by id
POST   /v3/projects/:id    // Updates an existing project by id
GET    /v3/projects        // Gets an inventory of projects
DELETE /v3/projects/:id    // Delete a project (requires Admin)
```

{% api-method method="get" host="" path="/projects" %}
{% api-method-summary %}
Get Project Inventory
{% endapi-method-summary %}

{% api-method-description %}
Gets the inventory of projects.
{% endapi-method-description %}

{% api-method-spec %}
{% api-method-request %}
{% api-method-headers %}
{% api-method-parameter name="Authentication" type="string" required=true %}
TODO
{% endapi-method-parameter %}
{% endapi-method-headers %}

{% api-method-query-parameters %}
{% api-method-parameter name="desc" type="boolean" required=false %}
`true` to sort projects in descending order; `false` to sort in ascending order. Defaults to `false`.
{% endapi-method-parameter %}

{% api-method-parameter name="size" type="integer" required=false %}
The number of projects to return per page.
{% endapi-method-parameter %}

{% api-method-parameter name="page" type="integer" required=false %}
The page number to return.
{% endapi-method-parameter %}
{% endapi-method-query-parameters %}
{% endapi-method-request %}

{% api-method-response %}
{% api-method-response-example httpCode=200 %}
{% api-method-response-example-description %}

{% endapi-method-response-example-description %}

```

```
{% endapi-method-response-example %}
{% endapi-method-response %}
{% endapi-method-spec %}
{% endapi-method %}

