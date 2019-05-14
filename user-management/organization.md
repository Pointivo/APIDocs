# Organization

An Organization is an object representing a business or tenant under which Users are members. All permissions and roles are scoped under an Organization.

{% api-method method="get" host="https://api.pointivo.com" path="/v3/organizations/:id" %}
{% api-method-summary %}
Get Organization By Id
{% endapi-method-summary %}

{% api-method-description %}
\*\*\*\*
{% endapi-method-description %}

{% api-method-spec %}
{% api-method-request %}
{% api-method-path-parameters %}
{% api-method-parameter name="id" type="string" required=true %}
Organization ID
{% endapi-method-parameter %}
{% endapi-method-path-parameters %}
{% endapi-method-request %}

{% api-method-response %}
{% api-method-response-example httpCode=200 %}
{% api-method-response-example-description %}

{% endapi-method-response-example-description %}

```javascript
{
    "name": "ACME Company",
    "primary_owner": "",
    "billing_contact": {
        "name": "George P. Burdell",
        "email_address": "george.p.burdell@example.com"
    },
    "technical_contact": {
        "name": "John Doe"
        "email_address": "john.doe@example.com"
    },
    "branding": {
        "name": "ACME",
        "logo": "organizations/acme/logo.svg",
        "theme": "red"
    },
    "urlRules": [
        "acme",
        "acmecompany"
    ],
    "_links": {
        "self": "org/acme",
        "subscriptions_uri": "org/acme/subscriptions",
        ""
    }
}
```
{% endapi-method-response-example %}
{% endapi-method-response %}
{% endapi-method-spec %}
{% endapi-method %}

{% api-method method="get" host="https://api.pointivo.com" path="/v3/organizations" %}
{% api-method-summary %}
Get Organizations
{% endapi-method-summary %}

{% api-method-description %}

{% endapi-method-description %}

{% api-method-spec %}
{% api-method-request %}

{% api-method-response %}
{% api-method-response-example httpCode=200 %}
{% api-method-response-example-description %}

{% endapi-method-response-example-description %}

```javascript
{
    "": ,
    "data": []
}
```
{% endapi-method-response-example %}
{% endapi-method-response %}
{% endapi-method-spec %}
{% endapi-method %}

