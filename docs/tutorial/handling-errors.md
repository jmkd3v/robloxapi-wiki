---
title: Handling errors
description: Learn how to handle API errors
---

Here’s an example of an error returned from the users.roblox.com/v1/users/{userId} endpoint, you can test this yourself by
going to https://users.roblox.com/v1/users/4:

```json
{
    "errors": [
        {
            "code": 3,
            "message": "The user id is invalid.",
            "userFacingMessage": "Something went wrong"
        }
    ]
}
```

As you can see, it’s an errors list containing objects with a code, message, and userFacingMessage.

-   The `code` is an identifier used for an error on an endpoint.
-   The `message` is information about the error intended for the developer.
-   The `userFacingMessage` is information about the error indented for the user. It isn’t always present and often isn’t
    as informative as the message.

We can use this code along with the HTTP response code (in this case, 404) and the endpoint’s "Responses" section from the
documentation to find more information.

![](../assets/screenshots/v1_users_responses.png){: .center }

In this case, it’s just telling us what we already know from the message. There are many cases where the API just doesn't
give us useful information, like this:

```json
{
    "errors": [
        {
            "code": 0,
            "message": "Something went wrong with the request, see response status code."
        }
    ]
}
```

In this case, we don’t get much information - the best we can do is use the documentation to try to figure out what’s
wrong here.
