# Message filtering

Messages contain a key, a value, and headers for metadata. This metadata is used to connect clients to topics and can be used to filter messages.

### Example gateways.yaml

In this gateway manifest, you're creating two gateways and filtering messages by the sessionID value. `sessionId` is a UUID generated by the CLI at runtime. Here, let's assume sessionId=123.

`value-from-parameters` defines what value is being used for filtering. Since it's sessionId, the sessionId value is 123.

{% hint style="info" %}
Once a parameter is defined for a gateway, that parameter is required for every subsequent connection.&#x20;
{% endhint %}

The producer will produce to the `input-topic` with the key `langstream-client-session-id` and the value 123.

The consumer will watch for messages with this key value in the headers on the `input-topic`.

```yaml
gateways:
  - id: produce-input-no-auth
    type: produce
    topic: input-topic
    parameters:
      - sessionId
    produce-options:
      headers:
        - key: langstream-client-session-id
          value-from-parameters: sessionId
  - id: consume-no-auth
    type: consume
    topic: input-topic
    parameters:
      - sessionId
    consume-options:
      filters:
        headers:
          - key: langstream-client-session-id
            value-from-parameters: sessionId
```

### Chat gateway

Chat gateways behave a little differently, because they open producers and consumers across one connection. Chat gateways assume the producer and consumer are sharing a topic and want to talk to each other, so the gateway's parameters are implicit. This still requires a sessionId, but it is implicitly the same value for producer and consumer.

```yaml
  - id: chat-no-auth
    type: chat
    chat-options:
      headers:
      - value-from-parameters: session-id
      questions-topic: input-topic
      answers-topic: output-topic
```

### headers

<table><thead><tr><th width="244">Label</th><th width="113.33333333333331">Type</th><th>Description</th></tr></thead><tbody><tr><td>key</td><td>string</td><td>An identifier of the value. This can be thought of as the name. Allowed characters are a-zA-Z0-9._-</td></tr><tr><td>valueFromParameters</td><td>string</td><td>The mapped name to connect a querystring value in the Gateway URL to message headers. This must match a value set in the parameters collection.</td></tr><tr><td>valueFromAuthentication</td><td>string</td><td>The mapped name to connect a provided querysting value with the identity provider’s handshake value. See <a href="gateway-authentication.md">Gateway authentication.</a></td></tr></tbody></table>

### Message offset

When creating a “consume” URL you can optionally provide instructions for reading the next message to be added or reading from the first message saved in the topic. Add a querystring value to the URL with the label: “option:position” and a value of  “earliest”, “latest”, or "absolute".

Example of consuming the next message to arrive:

`ws://localhost:8091/consume/my-super-cool-tenant/some-application/bot-output?option:position=latest`

Example of consuming all messages currently on the topic:

`ws://localhost:8091/consume/my-super-cool-tenant/some-application/bot-output?option:position=earliest`

If the value is neither "latest" nor "earliest", it's assumed to be a Base64 encoded string representing an absolute position, or offset. This string is then decoded, he absolute position is set, and the consumer begins consuming at the current offset value.