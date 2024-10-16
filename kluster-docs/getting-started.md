---
title: API documentation
description:
hide:
- footer
---

# API Reference

Welcome to the Kluster.ai getting started guide! This document will show you how to get started submitting Batch jobs quickly. If you are using the OpenAI API endpoints, then the Kluster.ai are compatible. If you are not using them, then you follow [these](https://platform.openai.com/docs/libraries/python-library) instructions to install the Python library.

In the next sections, we’ll show you Curl and Python examples of how to locate your API key, define a collection of Batch jobs as a JSON lines file, upload the file to the Kluster.ai endpoint, invoke the chat completion end point, monitor progress of the Batch job, retrieve the result of the Batch job, list all Batch objects, and cancel a Batch request.

## Get Your API Key

Navigate to the [platform.kluster.ai](http://platform.kluster.ai){target=\_blank} web app and select **API Keys** from the left hand menu. Create a new API key by specifying the API key name. You’ll need this to set the auth header in all of the API requests.

## List Supported Models

First, let’s use the models endpoint to list out the models that we support. Currently, only Meta-Llama-3.1-8B-Instruct, Meta-Llama-3.1-70B-Instruct, and Meta-Llama-3.1-405B-Instruct are supported. The response is a list of model objects. 

<div class="grid" markdown>
<div markdown>

**Request**

---

`get https://api.kluster.ai/v1/models` - Lists the currently available models.

**Returns**

---

`id` ++"string"++ - The model identifier, which can be referenced in the API endpoints.

---

`created` ++"integer"++ - The Unix timestamp (in seconds) when the model was created.

---

`object` ++"string"++ - The object type, which is always `model`.

---

`owned_by` ++"string"++ - The organization that owns the model.

</div>
<div markdown>

```bash title="Example request"
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

**Request body**

--8<-- ".snippets/api/chat-completion-object.md"

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
                    {"role": "user", "content": "You are an experienced maths tutor."}, {"role": "user", "content": "Explain the Pythagorean theorem."},
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
                    {"role": "system", "content": "You are an astronomer."},
                    {"role": "user", "content": "You are an experienced maths tutor."}, {"role": "user", "content": "What is the distance between the Earth and the Moon?"},
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

## Upload your Batch input file

`https://api.kluster.ai/v1/files`

Upload a [JSON Lines](https://jsonlines.org/) document to the Kluster.ai endpoint and take a note of the `id` in the response object.

<div class="grid" markdown>
<div markdown>

**Request**

`file` ++"file"++ <span class="Required" markdown>++"Required"++</span> - The File object (not file name) to be uploaded.

---

`purpose` ++"string"++ <span class="Required" markdown>++"Required"++</span> - The intended purpose of the uploaded file. Use `batch` for the Batch API.

---

**Returns**

The uploaded [File - todo](link to Kluster file object description!!) object.

`id` ++"string"++ - The file identifier, which can be referenced in the API endpoints.

---

`object` ++"string"++ - The object type, which is always `file`.

---

`bytes` ++"integer"++ - The size of the file, in bytes.

---

`created_at` ++"integer"++ - The Unix timestamp (in seconds) for when the file was created.

---

`filename` ++"string"++ - The name of the file.

---

`purpose` ++"string"++ - The intended purpose of the file. Currently, only `batch` is supported.

---

</div>
<div markdown>

=== "Curl"

    ```bash title="Example request"
    curl -s https://api.kluster.ai/v1/files \
    -H "Authorization: Bearer $API_KEY" \
    -H "Content-Type: multipart/form-data" \
    -F "file=@mybatchtest.jsonl" \
    -F "purpose=batch"
    ```

=== "Python"

    ```python title="Example request"
    # Upload the file to the platform
    batch_file = client.files.create(file=open(file_name, "rb"), purpose="batch")
    print(f"Batch file uploaded. File ID: {batch_file.id}")
    ```
</div>
</div>


## Invoke the chat completion end point

`post https://api.kluster.ai/v1/batches`

Next, to create a Batch job, you invoke the chat completion API using the `input_file_id` from the previous step.

<div class="grid" markdown>
<div markdown>

**Request**

---

`input_file_id` ++"string"++ <span class="Required" markdown>++"Required"++</span> - The ID of an uploaded file that contains requests for the new Batch.

---

Your input file must be formatted as a [JSONL file](https://jsonlines.org/), and must be uploaded with the purpose `batch`. The file can contain up to 50,000 requests, and can be up to 100 MB in size.

---

`endpoint` ++"string"++ <span class="Required" markdown>++"Required"++</span> - The endpoint to be used for all requests in the Batch. Currently, only `/v1/chat/completions` is supported.

---

`completion_window` ++"string"++ <span class="Required" markdown>++"Required"++</span> - The time frame within which the Batch should be processed. Currently, only **24h** is supported.

---

`metadata` ++"Object or null"++ - Custom metadata for the Batch.

---

**Returns**

---
The created [Batch](#batch-object) object.

`id` ++"string"++ - The ID of the batch.

---

`object` ++"string"++ - The object type, which is always `batch`.

---

`endpoint` ++"string"++ - The Kluster.ai API endpoint used by the batch.

---

`errors` ++"object"++ - Show properties.

---

`input_file_id` ++"string"++ - The ID of the input file for the batch.

---

`completion_window` ++"string"++ - The time frame within which the batch should be processed.

---

`status` ++"string"++ - The current status of the batch.

---

`output_file_id` ++"string"++ - The ID of the file containing the outputs of successfully executed requests.

---

`error_file_id` ++"string"++ - The ID of the file containing the outputs of requests with errors.

---

`created_at` ++"integer"++ - The Unix timestamp (in seconds) for when the Batch was created.

---

`in_progress_at` ++"integer"++ - The Unix timestamp (in seconds) for when the Batch started processing.

---

`expires_at` ++"integer"++ - The Unix timestamp (in seconds) for when the Batch will expire.

---

`finalizing_at` ++"integer"++ - The Unix timestamp (in seconds) for when the Batch started finalizing.

---

`completed_at` ++"integer"++ - The Unix timestamp (in seconds) for when the Batch was completed.

---

`failed_at` ++"integer"++ - The Unix timestamp (in seconds) for when the Batch failed.

---

`expired_at` ++"integer"++ - The Unix timestamp (in seconds) for when the Batch expired.

---

`cancelling_at` ++"integer"++ - The Unix timestamp (in seconds) for when the Batch started cancelling.

---

`cancelled_at` ++"integer"++ - The Unix timestamp (in seconds) for when the Batch was cancelled.

---

`request_counts` ++"object"++ - The request counts for different statuses within the Batch.

---

`metadata` ++"map"++ - Set of 16 key-value pairs that can be attached to an object. This is useful for storing additional information about the object in a structured format. Keys can be a maximum of 64 characters long, and values can be a maximum of 512 characters long.

</div>
<div markdown>

=== "Curl"

    ```bash title="Example request"
    curl -s https://api.kluster.ai/v1/batches \
    -H "Authorization: Bearer $API_KEY" \
    -H "Content-Type: application/json" \
    -d '{
    "input_file_id": "kluster-file-123",
    "endpoint": "/v1/chat/completions",
    "completion_window": "24h"
    }'
    ```

=== "Python"

    ```python title="Example request"
    # Submit the batch request
    batch_request = client.batches.create(
        input_file_id=batch_file.id,
        endpoint="/v1/chat/completions",
        completion_window="24h",  # Maximum time allowed for the batch to complete
        metadata={"description": "Batch job for chat completions"},
    )

    print(f"Batch request submitted. Batch ID: {batch_request.id}")
    ```


```Json title="Response"
{
    "id": "2b96e3e4-cd7b-43fd-9dd3-d82153bdd752",
    "object": "batch",
    "endpoint": "/v1/chat/completions",
    "errors": null,
    "input_file_id": "kluster-input-file-123",
    "completion_window": "24h",
    "status": "Validating",
    "output_file_id": null,
    "error_file_id": null,
    "created_at": "2024-10-07T12:45:38.109049154Z",
    "in_progress_at": null,
    "expires_at": "2024-10-08T12:45:38.109049154Z",
    "finalizing_at": null,
    "completed_at": null,
    "failed_at": null,
    "expired_at": null,
    "cancelling_at": null,
    "cancelled_at": null,
    "request_counts": {
        "total": 0,
        "completed": 0,
        "failed": 0
    },
    "metadata": {}
}
```
</div>
</div>

## Monitor the progress of the Batch job

---

`get https://api.kluster.ai/v1/batches/{batch_id}`

To verify that the Batch job has finished, check the `status` property for `completed` using the Batch id from the previous step.

<div class="grid" markdown>
<div markdown>

**Path parameters**

---

`batch_id` ++"string"++ <span class="Required" markdown>++"Required"++</span> - The ID of the Batch to retrieve.

**Returns**

---

The [Batch - todo](EH update) object matching the specified `id`.

</div>
<div markdown>

=== "Curl"

    ```bash title="Example request"
        curl -s https://api.kluster.ai/v1/batches/2b96e3e4-cd7b-43fd-9dd3-d82153bdd752 \
        -H "Authorization: Bearer $API_KEY" \
        -H "Content-Type: application/json"
    ```

=== "Python"
    
    ```python title="Example request"
    import time

    # Poll the batch status until it's complete
    while True:
        batch_status = client.batches.retrieve(batch_request.id)
        print(f"Batch status: {batch_status.status}")
        print(
            f"Completed tasks: {batch_status.request_counts.completed} / {batch_status.request_counts.total}"
        )

        if batch_status.status.lower() in ["completed", "failed", "canceled"]:
            break

        time.sleep(10)  # Wait for 10 seconds before checking again
    ```

```Json title="Response"
{
  "id": "2b96e3e4-cd7b-43fd-9dd3-d82153bdd752",
  "object": "batch",
  "endpoint": "/v1/chat/completions",
  "errors": null,
  "input_file_id": "kluster-input-file-123",
  "completion_window": "24h",
  "status": "Completed",
  "output_file_id": "kluster-output-file-123",
  "error_file_id": null,
  "created_at": "2024-10-07T13:08:52.821427Z",
  "in_progress_at": null,
  "expires_at": "2024-10-08T13:08:52.821427Z",
  "finalizing_at": null,
  "completed_at": null,
  "failed_at": null,
  "expired_at": null,
  "cancelling_at": null,
  "cancelled_at": null,
  "request_counts": {
    "total": 3,
    "completed": 3,
    "failed": 0
  },
  "metadata": {}
}
```
</div>
</div>

---
<div class="grid" markdown>
<div markdown>

## Retrieve the file content of the Batch job

---

`get https://api.kluster.ai/v1/files/{file_id}/content`

To retrieve the file content of the Batch job, send a request to the `files` end point specifying the `output_file_id` and redirecting standard output to a file.

**Path parameters**

---

`file_id` ++"string"++ <span class="Required" markdown>++"Required"++</span> - The ID of the file to use for this request

**Returns**

---

The output file content matching the specified file ID.

</div>
<div markdown>

=== "Curl"

    ```bash title="Example request"
        curl -s https://api.kluster.ai/v1/files/kluster-output-file-123"/content \
        -H "Authorization: Bearer $API_KEY" > batch_output.jsonl
    ```

=== "Python"
    
    ```python title="Example request"
        # Check if the batch completed successfully
        if batch_status.status.lower() == "completed":
            # Retrieve the results
            result_file_id = batch_status.output_file_id
            results = client.files.content(result_file_id).content

            # Save results to a file
            result_file_name = "data/batch_results.jsonl"
            with open(result_file_name, "wb") as file:
                file.write(results)
            print(f"Results saved to {result_file_name}")
        else:
            print(f"Batch failed with status: {batch_status.status}")
    ```
</div>
</div>

<div class="grid" markdown>
<div markdown>

## List all Batch jobs

---

`get https://api.kluster.ai/v1/batches`

To list all of your Batch objects, send a request to the batches endpoint without specifying a batch_id. To constrain the query response, you can also use a limit parameter. 

**Query parameters**

---

`after` ++"string"++ - A cursor for use in pagination. `after` is an object ID that defines your place in the list. For instance, if you make a list request and receive 100 objects, ending with `obj_foo`, your subsequent call can include `after=obj_foo` in order to fetch the next page of the list.

---

`limit` ++"integer"++ - A limit on the number of objects to be returned. Limit can range between 1 and 100, and the

---

**Returns**

---

A list of paginated [Batch - to do]() objects.

The status of a Batch object can be one of the following:

| Status          | Description                                                            |
|-----------------|------------------------------------------------------------------------|
| `validating`    | The input file is being validated.                                     |
| `failed`        | The input file failed the validation process.                          |
| `in_progress`   | The input file was successfully validated and the Batch is in progress.|
| `finalizing`    | The Batch job has completed and the results are being finalized.       |
| `completed`     | The Batch has completed and the results are ready.                     |
| `expired`       | The Batch was not completed within the 24-hour time window.            |
| `cancelling`    | The Batch is being cancelled (may take up to 10 minutes).              |
| `cancelled`     | The Batch was cancelled.                                               |

</div>
<div markdown>

=== "Curl"

    ```bash title="Example" 
        curl https://api.kluster.ai/v1/batches \
        -H "Authorization: Bearer $API_KEY"
    ```

=== "Python"

    ```Json title="Example"
    # Configure OpenAI client
    client = OpenAI(
        base_url="https://api.kluster.ai/v1",  # Change to "http://localhost:9000/api/v1" for local development
        api_key=my_api_key,  # Replace with your actual API key
    )

    print(client.batches.list(limit=2))
    ```

```Json title="Response"
    {
    "object": "list",
    "data": [
        {
        "id": "92aa978c-9dd1-49af-baa8-485ae6fb8019",
        "object": "batch",
        "endpoint": "/v1/chat/completions",
        "errors": null,
        "input_file_id": "kluster-input-file-123",
        "completion_window": "24h",
        "status": "Completed",
        "output_file_id": "kluster-output-file-123",
        "error_file_id": null,
        "created_at": "2024-10-07T16:53:53.046181Z",
        "in_progress_at": null,
        "expires_at": "2024-10-08T16:53:53.046181Z",
        "finalizing_at": null,
        "completed_at": null,
        "failed_at": null,
        "expired_at": null,
        "cancelling_at": null,
        "cancelled_at": null,
        "request_counts": {
            "total": 2,
            "completed": 2,
            "failed": 0
        },
        "metadata": {}
        },
    { ... },
    ],
    "first_id": "92aa978c-9dd1-49af-baa8-485ae6fb8019",
    "last_id": "0d29e406-51b2-4e5b-a9f4-6cb460eaeb59",
    "has_more": false,
    "count": 1,
    "page": 1,
    "page_count": -1,
    "items_per_page": 9223372036854775807
    }
```


</div>
</div>
---
<div class="grid" markdown>
<div markdown>

## Cancelling a Batch job

`post https://api.kluster.ai/v1/batches/{batch_id}/cancel`

To cancel an in-progress Batch job, send a cancel request to the batches endpoint specifying the `batch_id`. 

**Path parameters**

---

`batch_id` ++"string"++ <span class="Required" markdown>++"Required"++</span> - The ID of the Batch to cancel.

---

**Returns**

---

The [Batch - to do](EH update) object matching the specified ID.

</div>
<div markdown>

=== "Curl"

    ```bash title="Example"
    curl -s https://api.kluster.ai/v1/batches/$BATCH_ID/cancel \
    -H "Authorization: Bearer $API_KEY" \
    -H "Content-Type: application/json" \
    -X POST
    ```

=== "Python"

    ```python title="Example"
    client = OpenAI(
        base_url="https://api.kluster.ai/v1",  
        api_key=my_api_key,  # Replace with your actual API key
    )
    client.batches.cancel("kluster_123")
    ```

**Response**
```Json title="Response"
{
  "id": "642853d4-e816-4be2-8453-6ce6f0f00b9c",
  "object": "batch",
  "endpoint": "/v1/chat/completions",
  "errors": null,
  "input_file_id": "kluster-input-file-123",
  "completion_window": "24h",
  "status": "Cancelling",
  "output_file_id": "kluster-output-file-123",
  "error_file_id": null,
  "created_at": "2024-10-07T17:15:09.885223Z",
  "in_progress_at": null,
  "expires_at": "2024-10-08T17:15:09.885223Z",
  "finalizing_at": null,
  "completed_at": null,
  "failed_at": null,
  "expired_at": null,
  "cancelling_at": "2024-10-07T17:15:17.934748Z",
  "cancelled_at": null,
  "request_counts": {
    "total": 3,
    "completed": 3,
    "failed": 0
  },
  "metadata": {}
}
```
</div>
</div>

## Summary

You’ve now run a simple Batch use case by sending a collection of Batch request input objects to the chat completion end point, monitored the Batch interface for the progress of the job, and downloaded the result of the Batch job. To learn more about the end points we support, refer to the API documentation (link).