---
title: API Reference
description: A reference for the types and interfaces in the Moonbeam XCM SDK that can be used to send XCM transfers between chains within the Polkadot/Kusama ecosystems.
hide:
- footer
---

# API Reference

Welcome to the Kluster.ai getting started guide! This document will show you how to get started submitting Batch jobs quickly. If you are using the OpenAI API endpoints, then the Kluster.ai endpoints will be familiar. In the next sections, we’ll show you Curl and Python examples of how to locate your API key, define a collection of Batch jobs as a JSON lines file, upload the file to the Kluster.ai endpoint, invoke the chat completion end point, monitor progress of the Batch job, retrieve the result of the Batch job, list all Batch objects, and cancel a Batch request.

## Get Your API Key

Navigate to the [platform.kluster.ai](http://platform.kluster.ai){target=\_blank} web app and select **API Keys** from the left hand menu. Create a new API key by specifying the API key name. You’ll need this to set the auth header in all of the API requests.

## List Supported Models

First, let’s use the models endpoint to list out the models that we support. Currently, only Meta-Llama-3.1-8B-Instruct, Meta-Llama-3.1-70B-Instruct, and Meta-Llama-3.1-405B-Instruct are supported. The response is a list of model objects. 

<div class="grid" markdown>
<div markdown>

**Request**

`get https://api.kluster.ai/v1/models` - Lists the currently available models.

**Returns**

- `id` ++"string"++ - The model identifier, which can be referenced in the API endpoints.
- `created` ++"integer"++ - The Unix timestamp (in seconds) when the model was created.
- `object` ++"string"++- The object type, which is always `model`.
- `owned_by` ++"string"++- The organization that owns the model.

</div>
<div markdown>

```bash title="Query LLMs"
curl https://api.kluster.ai/v1/models \
  -H "Authorization: Bearer $API_KEY" 
```

```json title="Response"
[
   {
      "id":"meta-llama/Meta-Llama-3.1-70B-Instruct",
      "object":"model",
      "created":"1970-01-01T00:00:00Z",
      "owned_by":"owner1"
   },
   {
      "id":"meta-llama/Meta-Llama-3.1-8B-Instruct",
      "object":"model",
      "created":"1970-01-01T00:00:00Z",
      "owned_by":"owner2"
   },
   {
      "id":"meta-llama/Meta-Llama-3.1-405B-Instruct",
      "object":"model",
      "created":"1970-01-01T00:00:00Z",
      "owned_by":"owner3"
   }
]
```

</div>
</div>

---

## Create a Batch File with a Collection of Jobs

Create a [JSON Lines](https://jsonlines.org/) file containing a collection of `batch request input` objects. The body of each is a `chat completion` object with the endpoint `/v1/chat/completions`. Each request must include a unique `custom_id` which is used to reference results after the Batch job has completed.

<div class="grid" markdown>
<div markdown>

**Batch request input object**

`custom_id` ++"string"++

A developer-provided per-request id that will be used to match outputs to inputs. Must be unique for each request in a Batch.

---

`method` ++"string"++

The HTTP method to be used for the request. Currently only `POST` is supported.

---

`url` ++"string"++

The `/v1/chat/completions` API relative URL.

---

`body` ++"TODO: update type"++

The request body object (chat completion object).

??? child "Show properties"

    --8<-- 'text/api/chat-completion-object.md'

**Returns**

A chat completion object.

</div>
<div markdown>

=== "Curl"

    ```json title="Example: Collection of Batch Jobs"
    {"custom_id": "request-1", "method": "POST", "url": "/v1/chat/completions", "body": {"model": "meta-llama/Meta-Llama-3.1-8B-Instruct", "messages": [{"role": "system", "content": "You are a helpful assistant."}, {"role": "user", "content": "What is the capital of Argentina?"}],"max_tokens":1000}}
    {"custom_id": "request-1", "method": "POST", "url": "/v1/chat/completions", "body": {"model": "meta-llama/Meta-Llama-3.1-70B-Instruct", "messages": [{"role": "system", "content": "You are an experienced maths tutor."}, {"role": "user", "content": "Explain the Pythagorean theorem to a 10 year old child"}],"max_tokens":1000}}
    {"custom_id": "request-1", "method": "POST", "url": "/v1/chat/completions", "body": {"model": "meta-llama/Meta-Llama-3.1-405B-Instruct", "messages": [{"role": "system", "content": "You are a helpful assistant."}, {"role": "user", "content": "What is the distance between the Earth and the Moon"}],"max_tokens":1000}}
    ```

=== "Python"

    ```python title="Example: Collection of Batch Jobs"
    tasks = [{
            "custom_id": "request-1",
            "method": "POST",
            "url": "/v1/chat/completions",
            "body": {
                "model": "meta-llama/Meta-Llama-3.1-8B-Instruct",
                "messages": [
                    {"role": "system", "content": "You are a helpful assistant."},
                    {"role": "user", "content": "What is the capital of Argentina?"},
                ],
                "max_tokens": 1000,
            },
        },
        {
            "custom_id": "request-2",
            "method": "POST",
            "url": "/v1/chat/completions",
            "body": {
                "model": "meta-llama/Meta-Llama-3.1-70B-Instruct",
                "messages": [
                    {"role": "system", "content": "You are a maths tutor."},
                    {"role": "user", "content": "You are an experienced maths tutor."}, {"role": "user", "content": "Explain the Pythagorean theorem to a 10 year old child."},
                ],
                "max_tokens": 1000,
            },
        },
        {
            "custom_id": "request-3",
            "method": "POST",
            "url": "/v1/chat/completions",
            "body": {
                "model": "meta-llama/Meta-Llama-3.1-405B-Instruct",
                "messages": [
                    {"role": "system", "content": "You are a maths tutor."},
                    {"role": "user", "content": "You are an experienced maths tutor."}, {"role": "user", "content": "What is the distance between the Earth and the Moon."},
                ],
                "max_tokens": 1000,
            },
        }
        # Additional tasks can be added here
    ]

    # Save tasks to a JSONL file (newline-delimited JSON)
    file_name = "batch_tasks.jsonl"
    with open(file_name, "w") as file:
        for task in tasks:
            file.write(json.dumps(task) + "\n")
    ```

</div>
</div>

---

## Update Profile

<div class="grid" markdown>
<div markdown>

`updateProfile(userId, newData)` - Updates the user's profile with new information.

**Parameters**

- `userId` ++"string"++ - The unique identifier of the user whose profile is to be updated.
- `newData` ++"object"++ - An object containing the new profile information to be updated.

**Returns**

- `boolean` - `true` if the profile was successfully updated, `false` otherwise.

</div>
<div markdown>

```javascript title="Example"
const updated = updateProfile(
  '123456', 
  { name: 'Jane Doe', email: 'jane@example.com' }
);
console.log(updated);
```

```js title="Response"
true
```

</div>
</div>
