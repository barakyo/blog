---
title: Play! Framework -- Returning JSON Responses
date: 2014-04-06
publishdate: 2014-04-06
categories:
  - Development
tags:
  - Java
  - Play!
---

I've recently started a project at working using the [Play Framework](http://www.playframework.com/) and while its a great framework, I was having a lot of trouble with some of the simplest tasks. I wouldn't blame Play for my problems, returning to Java after a long hiatus, being spoiled by dynamicly typed languages, and lack of documentation really made such tasks like returning a JSON response difficult.

I figured that I may not be the only in this position, judging by the questions in the IRC channel and lack of responses, I figured it may be a good idea to jot down some of my notes, not only for myself, my coworkers, but for all my fellow Play framework noobs. Apologies for the introduction, I'll try to keep the rest of this posts and the futures, short, succinct, and to the point.

## tl;dr ##
Use ObjectNode class found in the `com.fasterxml.jackson.databind.node.ObjectNode` package.
{{< highlight java >}}
    import com.fasterxml.jackson.databind.node.ObjectNode;

    ObjectNode response = Json.newObject();
    response.put("status", 200);
    response.put("message", "Request was successful");
    response.put("data", Json.toJson(users));
    return ok(response);
{{< / highlight >}}

## Using ObjectNode ##
To easily return mixed response types, I suggest using the `ObjectNode` class found in the `com.fasterxml.jackson.databind.node.ObjectNode` package. Using the class is simple, first instantiate an object:
{{< highlight java >}}
    ObjectNode response = Json.newObject();
{{< / highlight >}}    
From there, you can treat the object similar to a map but using `put()` function.
{{< highlight java >}}
    response.put("status", 200);
    response.put("message", "Request was successful");
{{< / highlight >}}
Often times we want to return an object or array of objects. Using the `Json.toJson()` function found in `play.libs.Json` we can easily convert objects or lists of objects to be included in our JSON response.
{{< highlight java >}}
    response.put("data", Json.toJson(users));
{{< / highlight >}}
Finally, you'll probably want to use the static return response methods to easily return our JSON response.
{{< highlight java >}}
    return ok(response);
{{< / highlight >}}
