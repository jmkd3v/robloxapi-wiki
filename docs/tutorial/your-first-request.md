---
title: Your first request
description: Learn the basics of sending requests to API endpoints
---

At this point, you’re ready to use most of the endpoints that take in GET requests on these domains, as they won’t require
authentication, like users.roblox.com/v1/users/{userId}. This endpoint returns response data that looks like this:

```json
{
    "description": "Welcome to the Roblox profile! This is where you can check out the newest items in the catalog, and get a jumpstart on exploring and building on our Imagination Platform. If you want news on updates to the Roblox platform, or great new experiences to play with friends, check out blog.roblox.com. Please note, this is an automated account. If you need to reach Roblox for any customer service needs find help at www.roblox.com/help",
    "created": "2006-02-27T21:06:40.3Z",
    "isBanned": false,
    "externalAppDisplayName": null,
    "hasVerifiedBadge": true,
    "id": 1,
    "name": "Roblox",
    "displayName": "Roblox"
}
```

You’ll probably use this endpoint along with other endpoints on users.roblox.com quite a lot, as they are extremely useful
for grabbing information about a user.

=== "Python"
    ```py
    import requests

    user_id = 1  # User ID
    user_req = requests.get(f"https://users.roblox.com/v1/users/{user_id}")
    user_data = user_req.json()

    print("Name:", user_data["name"])
    print("Display Name:", user_data["displayName"])
    print("User ID:", user_data["id"])
    print("Description:", user_data["description"])
    ```
=== "Ruby"
    Requires the [http.rb](https://github.com/httprb/http) and [json](https://github.com/flori/json) gems.
    ```ruby
    require "http"
    require "json"

    USER_ID = 1
    response = HTTP.get("https://users.roblox.com/v1/users/#{USER_ID}")
    body = JSON.parse(response.body)

    puts "Username: #{body["name"]}"
    puts "Display name: #{body["displayName"]}"
    puts "User ID: #{body["id"]}"
    puts "Description: #{body["description"]}"
    ```
=== "Rust"
    Requires the [reqwest](https://docs.rs/reqwest/latest/reqwest/), [tokio](https://docs.rs/tokio/latest/tokio/) and
    [serde_json](https://docs.rs/serde_json/latest/serde_json/) crates.
    ```rust
    use reqwest::get;
    use serde_json::{from_str, Value};

    const USER_ID: i64 = 1;

    #[tokio::main]
    async fn main() {
        let response = get(format!("https://users.roblox.com/v1/users/{}", USER_ID))
            .await
            .unwrap()
            .text()
            .await
            .unwrap();
        let body: Value = from_str(&response).unwrap();

        println!("Username: {}", body["name"].as_str().unwrap());
        println!("Display name: {}", body["displayName"].as_str().unwrap());
        println!("User ID: {}", body["id"].as_i64().unwrap());
        println!("Description: {}", body["description"].as_str().unwrap());
    }
    ```
=== "F#"
    ```f#
    open System.Net.Http
    open System.Text.Json

    type Response =
        { name: string
          displayName: string
          id: int
          description: string }

    [<Literal>]
    let USER_ID = 1

    (task {
        use client = new HttpClient()
        let! response = client.GetAsync $"https://users.roblox.com/v1/users/{USER_ID}"
        let! content = response.Content.ReadAsStreamAsync()
        let! body = JsonSerializer.DeserializeAsync<Response> content

        printfn "Username: %s" body.name
        printfn "Display name: %s" body.displayName
        printfn "User ID: %d" body.id
        printfn "Description: %s" body.description
    })
        .Wait()
    ```
=== "C#"
    ```cs
    using System.Net.Http;
    using System.Text.Json;

    const int USER_ID = 1;
    var client = new HttpClient();
    var response = await client.GetAsync($"https://users.roblox.com/v1/users/{USER_ID}");
    // Can also deserialize to a named type instead of dynamic to access the fields directly
    dynamic body = JsonSerializer.Deserialize<dynamic>(
        await response.Content.ReadAsStringAsync()
    );

    Console.WriteLine($"Username: {body.GetProperty("name")}");
    Console.WriteLine($"Display name: {body.GetProperty("displayName")}");
    Console.WriteLine($"User ID: {body.GetProperty("id")}");
    Console.WriteLine($"Description: {body.GetProperty("description")}");
    ```

=== "JavaScript"
    If your runtime doesn't support native fetch, like for example pre-v21 Node.js, you may need to install a package
    like [node-fetch](https://www.npmjs.com/package/node-fetch).
    ```js
    const USER_ID = 1;
    const response = await fetch(`https://users.roblox.com/v1/users/${USER_ID}`);
    const body = await response.json();

    console.log(`Username: ${body["name"]}`);
    console.log(`Display name: ${body["displayName"]}`);
    console.log(`User ID: ${body["id"]}`);
    console.log(`Description: ${body["description"]}`);
    ```