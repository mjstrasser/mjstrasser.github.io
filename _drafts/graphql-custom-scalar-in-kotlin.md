---
layout: post
title: GraphQL custom scalars in Kotlin
categories: tech
tags: graphql kotlin spring
---

Graham Cox’s [Writing a GraphQL service using Kotlin and Spring
Boot](https://blog.pusher.com/writing-graphql-service-using-kotlin-spring-boot/) is a very good introduction
to using [GraphQL Java](https://github.com/graphql-java/graphql-java) in a [Kotlin](https://kotlinlang.org)
[Spring Boot](https://spring.io/projects/spring-boot) application.

I have been tinkering with a and wanted to define a custom [GraphQL scalar
type](https://graphql.org/learn/schema/#scalar-types) in Kotlin for an instant in time. In Java it uses
`java.time.Instant`.

{% highlight kotlin %}
@Component
class InstantScalar : GraphQLScalarType(
    "Instant",
    "GraphQL scalar for instant",
    object : Coercing<Instant, String> {

        override fun serialize(dataFetcherResult: Any): String {
            if (dataFetcherResult is Instant) {
                return dataFetcherResult.toString()
            }
            throw CoercingSerializeException("Invalid value '$dataFetcherResult' instant")
        }

        override fun parseValue(input: Any): Instant {
            try {
                return Instant.parse(input.toString())
            } catch (ex: Exception) {
                throw CoercingParseValueException("Invalid value '$input' for instant")
            }
        }

        override fun parseLiteral(input: Any): Instant {
            if (input is StringValue) {
                return Instant.parse(input.value)
            }
            throw CoercingParseLiteralException("Invalid value '$input' for instant")
        }
    }
)
{% endhighlight %}
