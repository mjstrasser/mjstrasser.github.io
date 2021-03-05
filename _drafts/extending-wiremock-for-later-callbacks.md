---
layout: post
title: Extending WireMock for later callbacks
categories: tech
tags: testing test-double kotlin
---

[WireMock](http://wiremock.org/) is a flexible Java test double for HTTP APIs that can be run in-process and 
as a standalone application. It has many built-in features and can also be extended.

# Asynchronous APIs

A little while back I had set up WireMock in standalone mode for performance testing of a service that makes a large
number of HTTP requests. In some cases the service makes HTTP calls to services that initiate some work and
call back later. The delay can be as much as 10 minutes: much longer than an HTTP server timeout.

I was able to model these asynchronous APIs in a WireMock extension as described here. The
extension is in Kotlin and is the GitHub
project [WireMock extension for asynchronous APIs with later callbacks](https://github.com/mjstrasser/wiremock-async).

# WireMock configuration

[WireMock in standalone mode](https://wiremock.org/docs/running-standalone/) can be configured with mappings 
specified in JSON. Here is one for the asynchronous API:

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
- Calls the transformer with name `DelayedCallback` to transform the response.
- Specifies parameters `median` and `sigma` for the transformer.

# Contract

The asynchronous APIs follow a contract that includes a payload and a callback URL. In Kotlin:

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

    override fun getName() = "DelayedCallback"

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
        val context = CallbackContext(contractRequest)

        // 3
        val delayMillis = callbackDelayMillis(parameters)
        // 4
        executor.schedule({ context.callback() }, delayMillis, TimeUnit.MILLISECONDS)

        val result = ContractResponse(
            context.callbackId,
            "Acknowledged the request. Will call back after $delayMillis ms"
        )
        // 5
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

- The string returned by `getName()` is used to identify the transformer in mapping specifications.

- `applyGlobally()` must be `false` so the transformer is only used for mappings where it is explicitly specified.

## The `transform` function

1. It reads the `ContractRequest` value from the incoming request.

2. Constructs a `CallbackContext` object from the contract request. It is defined in Kotlin:
   ```kotlin
   data class CallbackContext(
       val contractRequest: ContractRequest,
       val callbackId: String = UUID.randomUUID().toString()
   )
   ```

3. Calculates the callback delay in milliseconds from a lognormal distribution using the supplied parameters and the
   simple default of fixed, 1-second delay. See below for information about reading double parameter values.

4. Schedules the callback function using the executor service.

5. Returns a contract response.

## The callback function

This function is defined as an extension on the `CallbackContext` class. It uses the
[OkHttp](https://square.github.io/okhttp/) client from the library already present in WireMock
and sends another `ContractResponse` object.

```kotlin
fun CallbackContext.callback() {

    val okClient = OkHttpClient()

    val body = objectMapper.writeValueAsString(ContractResponse(callbackId, "All processing complete"))
    val request = Request.Builder()
        .url(contractRequest.callbackUrl)
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

The command-line to start it becomes:

```shell
java -cp wiremock-async-uber.jar \
  com.github.tomakehurst.wiremock.standalone.WireMockServerRunner \
  --extensions mjs.wiremock.DelayedCallback
```
