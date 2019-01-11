---
title: Play! Framework -- Parsing JSON
date: 2014-04-06
publishdate: 2014-04-06
---

## tl;dr ##
I suggest using `ObjectMapper` found in the `com.fasterxml.jackson.databind.ObjectMapper`

{{< highlight java >}}
    ObjectMapper mapper = new ObjectMapper();
    try {
        JsonNode root = request().body().asJson();
        JsonNode jsonPhoneNumbers = root.get("phoneNumbers");
        for(JsonNode phoneNumberString : jsonPhoneNumbers) {
            String phoneNumber = phoneNumberString.asText();
        }
    catch (NullPointerException npe) {
        // No valid phone numbers were provided
    }
{{< / highlight >}}

## Using ObjectMapper ##
For parsing arrays in JSON requests, I suggest using the `ObjectMapper`. Assuming you have a request that looks like:

{{< highlight json >}}
    {
        "phoneNumbers": [
            "555-555-5555",
            "555-555-5556",
            "555-555-5557"
        ]
    }
{{< / highlight >}}

We can easily access the phone number values by using `ObjectMapper` class. The first thing that you'd want to do is instantiate an instance of this class, which is pretty straight forward.
{{< highlight java >}}    
    ObjectMapper mapper = new ObjectMapper();
{{< / highlight >}}

From there, we want a JSON version of our request, the Play framework allows you to easily do so by chaining the `asJson()` method to the `request().body()` functions.
{{< highlight java >}}    
    JsonNode root = request().body().asJson();
{{< / highlight >}}

Using the `get()` function on our `root` object and providing the correct key, we can easily get our phoneNumbers as a JsonNode, which is also iterable. By iterating through each of the JsonNode objects, we can just return the value for the corresponding phone number as a `String` by using the `asText()` method.
{{< highlight java >}}    
    JsonNode jsonPhoneNumbers = root.get("phoneNumbers");
    for(JsonNode phoneNumberString : jsonPhoneNumbers) {
        String phoneNumber = phoneNumberString.asText();
    }
{{< / highlight >}}

Since there is a possibility that the request could be empty (null), I suggest wrapping the request processing in a try-catch block. If a request is null or the specified key was not included in the request (in this example it would be `phoneNumbers`) then we can use the `catch` portion of the block to easily return a bad request.

Put this all together looks like:

{{< highlight java >}}
    import com.fasterxml.jackson.databind.JsonNode;
    import com.fasterxml.jackson.databind.ObjectMapper;
    import com.fasterxml.jackson.databind.node.ObjectNode;
    import play.libs.Json;

    ObjectMapper mapper = new ObjectMapper();
    ObjectNode response = Json.newObject();
    try {
        JsonNode root = request().body().asJson();
        JsonNode jsonPhoneNumbers = root.get("phoneNumbers");
        for(JsonNode phoneNumberString : jsonPhoneNumbers) {
            String phoneNumber = phoneNumberString.asText();
        }
    catch (NullPointerException npe) {
        // No valid phone numbers were provided
        response.put("status", 400);
        response.put("message", "No phone numbers were provided");
        return badRequest(response);
    }
{{< / highlight >}}
