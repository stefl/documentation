# Creating Conversations

You can create Conversations using the following endpoint.

```request
POST /apps/:app_uuid/conversations
```

### Parameters

| Name    |  Type | Description |
|---------|-------|-------------|
| **participants** | array  | User IDs (strings) identifying who will participate in the Conversation |
| **distinct** | boolean | Create or find a unique Conversation between these participants |
| **metadata** | object | Arbitrary set of name value pairs representing initial state of Conversation metadata |

### Example Request: Create a Conversation

```json
{
    "participants": [
        "1234",
        "5678"
    ],
    "distinct": false,
    "metadata": {
        "background_color": "#3c3c3c"
    }
}
```

### Response `201 (Created)`

```json
{
    "id": "layer:///conversations/f3cc7b32-3c92-11e4-baad-164230d1df67",
    "url": "https://api.layer.com/apps/24f43c32-4d95-11e4-b3a2-0fd00000020d/conversations/f3cc7b32-3c92-11e4-baad-164230d1df67",
    "messages_url": "https://api.layer.com/apps/24f43c32-4d95-11e4-b3a2-0fd00000020d/conversations/f3cc7b32-3c92-11e4-baad-164230d1df67/messages",
    "created_at": "2014-09-15T04:44:47+00:00",
    "participants": [
        "1234",
        "5678"
    ],
    "distinct": false,
    "metadata": {
        "info": {
            "background_color": "#3c3c3c",
            "title": "A conversation about Coffee"
        }
    }
}
```

```console
curl  -X POST \
      -H 'Accept: application/vnd.layer+json; version=1.1' \
      -H 'Authorization: Bearer TOKEN' \
      -H 'Content-Type: application/json' \
      -d '{"participants": ["a", "b"], "distinct": false, "metadata": {"info": {"background_color": "#3c3c3c", "title": "A conversation about Coffee"}}}' \
      https://api.layer.com/apps/APP_UUID/conversations
```

## Distinct Conversations

If `User A` wants to talk to `User B`, they should not need to create a new Conversation every time they talk. By reusing an existing Conversation, they can access the message history and context around their previous communications. To help ensure that users do not inadvertently create multiple Conversations when they intend to maintain a single thread of communication, Layer supports the notion of Distinct Conversations.

In a Distinct Conversation, it is guaranteed that among the given set of participants there will exist one (and only one) Conversation. Each Conversation has a `distinct` property that determines how it is created. Possible values are:

* `true` If there is a matching Conversation, return it. Otherwise a new Conversation is created and returned.
* `false` Always create and return a new Conversation.

An existing Conversation matches if it is itself Distinct, and it has the same participants.

> A Distinct Conversation becomes a non-distinct Conversation if you change its participant list.

## Handling Metadata with Distinct Conversations

When creating a Distinct Conversation, there are three possible results.

### Response `201 (Created)`

If there is no existing Distinct Conversation that matches the request, then create a new Conversation and return it.  Result is the same as creating a non-distinct Conversation.

### Response `200 (OK)`

If there is a matching Distinct Conversation, and one of these  holds true, then an existing Conversation is returned.

1. The `metadata` property was not included in the request
2. The `metadata` property was included but with a value of `null`
3. The `metadata` property value is identical to the metadata of the matching Distinct Conversation

In addition to returning the existing Conversation, a `Location` header will be returned with the URL to that Conversation.

### Response Header

```text
Location: /apps/24f43c32-4d95-11e4-b3a2-0fd00000020d/conversations/f3cc7b32-3c92-11e4-baad-164230d1df67
```

### Example Request: Create a Distinct Conversation

```json
{
    "id": "layer:///conversations/f3cc7b32-3c92-11e4-baad-164230d1df67",
    "url": "https://api.layer.com/apps/24f43c32-4d95-11e4-b3a2-0fd00000020d/conversations/f3cc7b32-3c92-11e4-baad-164230d1df67",
    "messages_url": "https://api.layer.com/apps/24f43c32-4d95-11e4-b3a2-0fd00000020d/conversations/f3cc7b32-3c92-11e4-baad-164230d1df67/messages",
    "created_at": "2014-09-15T04:44:47+00:00",
    "participants": [
        "1234",
        "5678"
    ],
    "distinct": true,
    "metadata": {
        "background_color": "#3c3c3c"
    }
}
```

### Response `409 (Conflict)`

If the matching Distinct Conversation has metadata different from what was requested, return an error that contains the matching Conversation so that the application can determine what steps to take next (e.g. use the Conversation or modify it).

```json
{
    "id": "resource_conflict",
    "code": 108,
    "message": "The requested Distinct Conversation was found but had metadata that did not match your request.",
    "url": "https://developer.layer.com/api.md#creating-a-conversation",
    "data": {
        "id": "layer:///conversations/f3cc7b32-3c92-11e4-baad-164230d1df67",
        "url": "https://api.layer.com/apps/24f43c32-4d95-11e4-b3a2-0fd00000020d/conversations/f3cc7b32-3c92-11e4-baad-164230d1df67",
        "messages_url": "https://api.layer.com/apps/24f43c32-4d95-11e4-b3a2-0fd00000020d/conversations/f3cc7b32-3c92-11e4-baad-164230d1df67/messages",
        "created_at": "2014-09-15T04:44:47+00:00",
        "participants": [
            "1234",
            "5678"
        ],
        "distinct": true,
        "metadata": {
            "background_color": "#3c3c3c"
        }
    }
}
```
