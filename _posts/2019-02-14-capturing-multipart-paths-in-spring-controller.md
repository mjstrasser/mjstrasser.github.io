---
layout: post
title: Capturing multi-part paths in Spring controllers
categories: tech
tags: spring java
---

A while back I worked on a Spring Boot application that stores and works with Swagger files.
It has a controller that needs to capture file paths at the end of request URIs. 

We need to extract three variables: `project`, `repo` and `path`, where `path` may traverse multiple
layers. Some examples:

| URI | project | repo | path |
|-----|---------|------|------|
| `/swagger/AAA/domain-api/swagger.yaml` | `AAA` | `domain-api` | `swagger.yaml` |
| `/swagger/BBB/domain-api/swaggers/domain.json` | `BBB` | `domain-api` | `swaggers/domain.json` |

This naive attempt at a solution does not work:

```java
@GetMapping(path = "/swagger/{project}/{repo}/{path}")
public ResponseEntity<String> swaggerFile(@PathVariable("project") String project,
                                          @PathVariable("repo") String repo,
                                          @PathVariable("path") String path) {
```

With this, when using the first URI, the `path` variable gets the value `swagger` without the file extension.
The second URI does not match at all.

The file extension can be captured by using a regular expression, for example:

```java
@GetMapping(path = "/swagger/{project}/{repo}/{path:.+}")
```

Here, given the first URI, the `path` variable gets the value `swagger.yaml` as desired.
But the second URI still does not match at all.

There is no simple way to extract a deep path into a single variable. I think
this is because the Spring classes split the path into segments using slash characters
before matching each segment with a regular expression.

A working solution is to get the entire path from the servlet request object and parse
it locally.

```java
@GetMapping(path = "/swagger/{project}/{repo}/**")
public ResponseEntity<String> swaggerFile(@PathVariable("project") String project,
                                          @PathVariable("repo") String repo,
                                          HttpServletRequest request) {
    String prefix = String.format("/swagger/%s/%s", project, repo);
    String path = request.getRequestURI().substring(prefix.length() + 2);
    // Use path
}
``` 
