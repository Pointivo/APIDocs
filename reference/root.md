# Root

{% api-method method="get" host="https://api.pointivo.com" path="/" %}
{% api-method-summary %}
Get Root
{% endapi-method-summary %}

{% api-method-description %}
Gets the root resource of the API, supplying links to all the other resources available through the API.
{% endapi-method-description %}

{% api-method-spec %}
{% api-method-request %}
{% api-method-path-parameters %}
{% api-method-parameter name="authentication" type="string" required=false %}
See Authentication for details.
{% endapi-method-parameter %}
{% endapi-method-path-parameters %}
{% endapi-method-request %}

{% api-method-response %}
{% api-method-response-example httpCode=200 %}
{% api-method-response-example-description %}

{% endapi-method-response-example-description %}

```
{
    "projects": "/projects",
    "domains": "/domains",
    "jobs": "/jobs"
}
```
{% endapi-method-response-example %}

{% api-method-response-example httpCode=401 %}
{% api-method-response-example-description %}
The user is not authorized to retrieve the root resource.
{% endapi-method-response-example-description %}

```
{
    "code": "unauthorized",
    "message": "Unauthorized"
}
```
{% endapi-method-response-example %}
{% endapi-method-response %}
{% endapi-method-spec %}
{% endapi-method %}



