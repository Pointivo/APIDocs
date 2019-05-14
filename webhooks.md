# Webhooks

{% hint style="info" %}
"Callbacks" \(Pointivo API v2\) are now called "Webhooks"
{% endhint %}

Webhooks allow 3rd party applications and services to receive near real-time data payloads when events are triggered in Pointivo. In most cases, webhooks are a faster, more efficient alternative to polling the API for changes in the data.

#### Example Webhook Delivery

```javascript
POST http://api.example.com/server/endpoint

{
    id: "eaab9e79-1452-4bd7-a91d-48aaabe44ac6",
    subscription_id: "479b9918-d48e-405b-8c35-3e7f525a66c2",
    type: "PROJECT_CREATED",
    attempt_number: 3,
    data: {
        "foo": "bar"
    }
}
```

This is an example webhook delivery for a `PROJECT_CREATED` event. In this example, the subscription was configured for `http://api.example.com/server/endpoint`and this is delivery attempt number 3.

In addition to the payload data, each webhook call includes a unique identifier, the subscription which originated the request, a type, and the current number of attempts. We recommend that you use this data for logging and surfacing helpful messaging in your application or service.

#### **Request & Responses**

Webhook requests are `POST` only. If your server doesn't respond or responds with anything other than `200 OK`, Pointivo will automatically retry according to the [schedule](webhooks.md#failures-and-retries) below. Responding with a `410 Gone` will deactivate the webhook subscription.

#### **Failures & Retry Schedule**

The webhook will automatically retry delivery up to 16 times with an exponential backoff. You can also manually redeliver a webhook at https://admin.pointivo.com/webhooks.

Requests will be retried using the following schedule after the first failure: 30 seconds, 1 minute, 2 minutes, 4 minutes, 8 minutes, 16 minutes, 32 minutes, 64 minutes, 128 minutes \(2.1 hours\), 256 minutes \(4.2 hours\), 512 minutes \(8.5 hours\), 1,024 minutes \(17 hours\), and finally once per day, up to 4 days.

