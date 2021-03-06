---
layout: post
title: Extending WireMock for delayed callbacks
categories: tech
tags: testing test-double kotlin
---

[WireMock](http://wiremock.org/) is a flexible Java test double for HTTP APIs that can be run in-process and 
as a standalone application. It has many built-in features and can also be extended.

# Delayed callbacks

A little while back I had set up WireMock in standalone mode as part of testing a client’s service.
In some cases the service makes HTTP calls to other services that perform some work and
call back later. The delay can be as much as 10 minutes: much longer than an HTTP server timeout.

I wrote a WireMock extension that models these asynchronous APIs so we could test them.
The extension is in Kotlin and a simplified version is the GitHub project [WireMock extension for asynchronous APIs
with later callbacks](https://github.com/mjstrasser/wiremock-async).

# WireMock configuration

[WireMock in standalone mode](https://wiremock.org/docs/running-standalone/) is usually configured with mappings 
specified in JSON. Here is an example mapping for the asynchronous API:

```json
{
  "request": {
    "method": "POST",
    "url": "/contract/action"
  },
  "response": {
    "transformers": [
      "DelayedCallback"
    ],
    "transformerParameters": {
      "median": 4000,
      "sigma": 0.4
    }
  }
}
```

This mapping:

- Accepts POST requests to the URI `/contract/action`.
- Transforms the response using a transformer with name `DelayedCallback`.
- Specifies parameters `median` and `sigma` for the transformer.

# Contract

The asynchronous APIs follow a contract that includes a payload and a callback URL. Here is a simplified version:

```kotlin
data class ContractRequest(
    val correlationId: String,
    val payload: String,
    val callbackUrl: String,
)

data class ContractResponse(
    val correlationId: String,
    val message: String,
    val timestamp: String = Instant.now().toString(),
)
```

# Transformer

The `DelayedCallback` transformer extends the WireMock `ResponseTransformer` abstract class.

```kotlin
class DelayedCallback : ResponseTransformer() {

    companion object {
        @JvmStatic
        val executor: ScheduledExecutorService = Executors.newScheduledThreadPool(10)
        val objectMapper = jacksonObjectMapper()
    }

    // Identify this transformer in mapping specifications.
    override fun getName() = "DelayedCallback"
    // Only apply the transformer when specified.
    override fun applyGlobally() = false

    override fun transform(request: Request, response: Response, files: FileSource, parameters: Parameters): Response {

        // 1
        val contractRequest = try {
           objectMapper.readValue<ContractRequest>(request.bodyAsString)
        } catch (ex: Exception) {
           logger.error("Exception reading contract request", ex)
           return Response.Builder.like(response)
               .status(400)
               .body(ex.message)
               .build()
        }

        // 2
        val delayMillis = callbackDelayMillis(parameters)

        // 3
        executor.schedule({ contractRequest.callback() }, delayMillis, TimeUnit.MILLISECONDS)

        // 4
        val result = ContractResponse(
            contractRequest.correlationId,
            "Acknowledged the request. Will call back after $delayMillis ms",
        )
        return Response.Builder.like(response)
            .status(200)
            .body(objectMapper.writeValueAsString(result))
            .headers(HttpHeaders(HttpHeader.httpHeader("Content-Type", "application/json")))
            .build()
    }

    private fun callbackDelayMillis(parameters: Parameters?) =
        parameters?.let {
            val median = it.getDoubleValue("median", 1000.0)
            val sigma = it.getDoubleValue("sigma", 0.1)
            randomLogNormalMillis(median, sigma)
        } ?: 1000L

    private fun randomLogNormalMillis(median: Double, sigma: Double) =
        (exp(ThreadLocalRandom.current().nextGaussian() * sigma) * median).roundToLong()
}
```

## The `transform` function

1. Reads the `ContractRequest` value from the incoming request.

2. Calculates the callback delay in milliseconds from a lognormal distribution using the supplied parameters and the
   simple default of fixed, 1-second delay. See below for information about reading double parameter values.

3. Schedules the callback function using the executor service.

4. Returns a contract response with the specified `correlationId`.

## The callback function

This function is defined as an extension on `ContractRequest`. It uses the
[OkHttp](https://square.github.io/okhttp/) client from the library already present in WireMock
and sends another `ContractResponse` object with the same `correlationId`.

```kotlin
fun ContractRequest.callback() {

    val okClient = OkHttpClient()

    val body = objectMapper.writeValueAsString(ContractResponse(correlationId, "All processing complete"))
    val request = Request.Builder()
        .url(callbackUrl)
        .post(body.toRequestBody("application/json".toMediaType()))
        .build()

    okClient.newCall(request).execute().use { response ->
        if (!response.isSuccessful)
            logger.error("Error calling back: ${response.message}")
        else
            logger.info("Callback successful: ${response.message}")
    }
}
```

## WireMock Parameters extension function

An extension to the WireMock `Parameters` class makes it easy to read double values safely:

```kotlin
fun Parameters.getDoubleValue(key: String, default: Double) = if (key in this)
    when (val value = get(key)) {
        is Double -> value
        is Int -> value.toDouble()
        is String -> value.toDoubleOrNull() ?: default
        else -> default
    }
else default
```

The [tests for this function](https://github.com/mjstrasser/wiremock-async/blob/main/src/test/kotlin/mjs/wiremock/ParametersTest.kt)
show how it works.

# Putting it together

A Gradle ‘uber-JAR’ task bundles WireMock standalone and extension code into one large JAR:

```kotlin
tasks.register<Jar>("uberJar") {
    archiveClassifier.set("uber")

    from(sourceSets.main.get().output)

    dependsOn(configurations.runtimeClasspath)
    from({
        configurations.runtimeClasspath.get().filter { it.name.endsWith("jar") }.map { zipTree(it) }
    })
}
```

The command line to start it becomes:

```shell
java -cp wiremock-async-uber.jar \
  com.github.tomakehurst.wiremock.standalone.WireMockServerRunner \
  --extensions mjs.wiremock.DelayedCallback
```

- The main class must be specified explicitly.
- WireMock is passed a comma-separated list of extension class names.
