---
title: Http Client
category: clients
permalink: /clients/index.html
children: /clients/http-client/
caption: Http Client
ktor_version_review: 1.2.3
redirect_from:
  - /clients/http-client.html
---

{::options toc_levels="1..2" /}

In addition to HTTP serving, Ktor also includes a flexible asynchronous HTTP client.
This client supports several [configurable engines](/clients/http-client/engines.html), and has its own set of [features](/clients/http-client/features.html).

The main functionality is available through the `io.ktor:ktor-client-core:$ktor_version` artifact.
And each engine, is provided in [separate artifacts](/clients/http-client/engines.html).
{: .note.artifact }

**Table of contents:**

* TOC
{:toc}

## Calls: Requests and Responses

{: #requests-responses }

You can check [how to make requests](/clients/http-client/call/requests.html),
and [how to receive responses](/clients/http-client/call/responses.html) in their respective sections.

## Concurrency

Remember that requests are asynchronous, but when performing requests, the API suspends further requests
and your function will be suspended until done. If you want to perform several requests at once
in the same block, you can use `launch` or `async` functions and later get the results.
For example:

### Sequential requests

```kotlin
suspend fun sequentialRequests() {
    val client = HttpClient()

    // Get the content of an URL.
    val firstBytes = client.get<ByteArray>("https://127.0.0.1:8080/a")

    // Once the previous request is done, get the content of an URL.
    val secondBytes = client.get<ByteArray>("https://127.0.0.1:8080/b")

    client.close()
}
```

### Parallel requests

```kotlin
suspend fun parallelRequests() = coroutineScope<Unit> {
    val client = HttpClient()

    // Start two requests asynchronously.
    val firstRequest = async { client.get<ByteArray>("https://127.0.0.1:8080/a") }
    val secondRequest = async { client.get<ByteArray>("https://127.0.0.1:8080/b") }

    // Get the request contents without blocking threads, but suspending the function until both
    // requests are done.
    val bytes1 = firstRequest.await() // Suspension point.
    val bytes2 = secondRequest.await() // Suspension point.

    client.close()
}
```

To create a parallel list of jobs with error handling use the supervisorScope (this allow individual jobs to fail). The following code will run all of the jobs simultaneously and return when they are all completed.
```kotlin
suspend fun parallelRequests(requests: List<String>) = supervisorScope<Unit> {
    // Create our HTTP client
    val client = HttpClient(Apache) {
        install(JsonFeature) {
            serializer = GsonSerializer {
                // .GsonBuilder
                serializeNulls()
                disableHtmlEscaping()
            }
        }
    }
    // Create a list of our jobs
    val jobs = mutableListOf<Deferred<Boolean>>()
    for (req in requests) {
        // Launch job asynchronosly 
        val job = async() {
            val results = client.get<ChannelSummary> {
                url(req)
                contentType(ContentType.Application.Json)
            }
            // Use data in another function
            insertChannel(results.channel)
        }
        // Add job to job list.
        jobs.add(job)
    }
    // Now collect all our jobs. Deal with errors.
    for (job in jobs) {
        try {
            job.await()
        } catch (e: Throwable) {
            println(e)
        }
    }
    // Close HTTP client.
    client.close()
}
```

## Examples
{: #examples }

For more information, check the [examples page](/clients/http-client/examples.html) with some examples.

## Features
{: #features}

For more information, check the [features page](/clients/http-client/features.html) with all the available features.
