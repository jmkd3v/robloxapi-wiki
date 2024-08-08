---
title: X-CSRF-TOKEN
description: Learn how to handle the anti-cross-site-request-forgery token
---

At this point, we’re now authenticated - but there’s one thing missing. If we try to send a POST request, you’ll notice
that the request still fails.

Here's an example of some code that won't work due to the `X-CSRF-TOKEN`: 

=== "Python"
    ```py
    import requests

    cookie = "_|WARNING:-DO-NOT-SHARE-THIS.--Sharing-this-will-allow-someone-to-log-in-as-you-and-to-steal-your-ROBUX-and-items.|_TOKEN"

    session = requests.Session()
    session.cookies[".ROBLOSECURITY"] = cookie
    req = session.post(
        url="https://auth.roblox.com/v2/login"
    )
    error = req.json()

    print(req.status_code)
    print("Error code:", error["code"])
    print("Error message:", error["message"])
    ```
=== "Ruby"
    Requires the [http.rb](https://github.com/httprb/http) and [json](https://github.com/flori/json) gems.
    ```rb
    require "http"
    require "json"

    COOKIE = "_|WARNING:-DO-NOT-SHARE-THIS.--Sharing-this-will-allow-someone-to-log-in-as-you-and-to-steal-your-ROBUX-and-items.|_TOKEN"

    response = HTTP.cookies({
      ".ROBLOSECURITY": COOKIE
    }).post("https://auth.roblox.com")
    error = JSON.parse(response)

    puts response.status
    puts "Error code: #{error['code']}"
    puts "Error message: #{error['message']}"
    ```
=== "JavaScript"
    If your runtime doesn't support native fetch, like for example pre-v21 Node.js, you may need to install a package
    like [node-fetch](https://www.npmjs.com/package/node-fetch).
    ```js
    const COOKIE = "_|WARNING:-DO-NOT-SHARE-THIS.--Sharing-this-will-allow-someone-to-log-in-as-you-and-to-steal-your-ROBUX-and-items.|_TOKEN";

    const response = await fetch("https://auth.roblox.com", {
        headers: {
            Cookie: `.ROBLOSECURITY=${COOKIE};`,
            "Content-Length": "0",
        },
        method: "POST",
    });
    const body = await response.json();

    console.log(response.status);
    console.log(`Error code: ${body.code}`)
    console.log(`Error message: ${body.message}`)
    ```
=== "Rust"
    Requires the [reqwest](https://docs.rs/reqwest/latest/reqwest/), [tokio](https://docs.rs/tokio/latest/tokio/) and
    [serde_json](https://docs.rs/serde_json/latest/serde_json/) crates.
    ```rs
    use reqwest::header::HeaderMap;
    use reqwest::Client;
    use serde_json::{from_str, Value};

    const COOKIE: &str = "_|WARNING:-DO-NOT-SHARE-THIS.--Sharing-this-will-allow-someone-to-log-in-as-you-and-to-steal-your-ROBUX-and-items.|_TOKEN";

    #[tokio::main]
    async fn main() {
        let client = Client::new();
        let mut headers = HeaderMap::new();
        headers.insert(
            "Cookie",
            format!(".ROBLOSECURITY={};", COOKIE).parse().unwrap(),
        );

        let response = client
            .post("https://auth.roblox.com")
            .headers(headers)
            .send()
            .await
            .unwrap();

        let status = response.status(); // get status here because .text() consumes the response
        let body: Value = from_str(&response.text().await.unwrap()).unwrap();

        println!("{}", status);
        println!("Error code: {}", body["code"]);
        println!("Error message: {}", body["message"]);
    }
    ```

=== "F#"
    ```f#
    open System.Net
    open System.Net.Http
    open System.Text.Json

    type Error = { code: int; message: string }

    [<Literal>]
    let COOKIE =
        "_|WARNING:-DO-NOT-SHARE-THIS.--Sharing-this-will-allow-someone-to-log-in-as-you-and-to-steal-your-ROBUX-and-items.|_TOKEN"

    (task {
        let cookieContainer = CookieContainer()
        cookieContainer.Add(Cookie(".ROBLOSECURITY", COOKIE, Domain = ".roblox.com"))

        use httpClient =
            new HttpClient(new HttpClientHandler(UseCookies = true, CookieContainer = cookieContainer))

        let! response = httpClient.SendAsync(new HttpRequestMessage(HttpMethod.Post, "https://auth.roblox.com/"))
        let! content = response.Content.ReadAsStreamAsync()
        let! error = JsonSerializer.DeserializeAsync<Error>(content)

        printfn "%d" (int response.StatusCode)
        printfn "Error code: %d" error.code
        printfn "Error message: %s" error.message
    })
        .Wait()
    ```
=== "C#"
    ```cs
    using System.Net;
    using System.Net.Http;
    using System.Text.Json;

    const string COOKIE = "_|WARNING:-DO-NOT-SHARE-THIS.--Sharing-this-will-allow-someone-to-log-in-as-you-and-to-steal-your-ROBUX-and-items.|_TOKEN";

    var cookieContainer = new CookieContainer();
    cookieContainer.Add(new Cookie(".ROBLOSECURITY", COOKIE) { Domain = ".roblox.com" });

    var client = new HttpClient(new HttpClientHandler() { UseCookies = true, CookieContainer = cookieContainer });

    var response = await client.PostAsync("https://auth.roblox.com/", null);
    dynamic body = JsonSerializer.Deserialize<dynamic>(
        await response.Content.ReadAsStringAsync()
    );

    Console.WriteLine(((int)response.StatusCode).ToString());
    Console.WriteLine($"Error code: {body.GetProperty("code")}");
    Console.WriteLine($"Error message: {body.GetProperty("message")}");
    ```
=== "Elixir"
    Compile with `iex -S mix`, then execute `RobloxAPI.main`.

    Dependencies: [httpoison](https://hex.pm/packages/httpoison) and [poison](https://hex.pm/packages/poison)
    ```ex
    defmodule RobloxAPI do
      @roblosecurity "_|WARNING:-DO-NOT-SHARE-THIS.--Sharing-this-will-allow-someone-to-log-in-as-you-and-to-steal-your-ROBUX-and-items.|_TOKEN"

      def main do
        headers = %{Cookie: ".ROBLOSECURITY=#{@roblosecurity};"}

        with {:ok, response} <- HTTPoison.post("https://auth.roblox.com", "", headers),
             {:ok, body} <- Poison.decode(response.body) do
          IO.puts(response.status_code)
          IO.puts("Error code: #{body["code"]}")
          IO.puts("Error message: #{body["message"]}")
        else
          {:error, error} ->
            IO.inspect(error)
        end
      end
    end
    ```

This code should output something like the following:

    403
    Error code: 0
    Error message: Token Validation Failed

The `403 Forbidden` status code is returned when the client "is not permitted access to the resource despite providing
authentication such as insufficient permissions of the authenticated account".

If you saw this while trying to write your own code to access the API, you might ask "why is this error coming up? My
.ROBLOSECURITY token is correct, and it worked when I used the "Try it out!" button on the documentation page."

The truth is that this error message isn’t referring to "token" as in your .ROBLOSECURITY token - it’s actually referring
to a header that you have to supply to all requests that change data called the `X-CSRF-TOKEN`. It is used to prevent a
[cross-site request forgery attack](https://en.wikipedia.org/wiki/Cross-site_request_forgery), which would enable a
malicious website to send an authenticated request to the Roblox API if you have a logged-in session in your browser.

To handle this token, each time we send a request, we'll save the `X-CSRF-TOKEN` - which is present in the response headers
- to a value. Then, if the request failed with a status code of 403, and one of the errors has the code 0, we'll send the
request again with the `X-CSRF-TOKEN` we just got the first request as a request header. 

=== "Python"
    ```py
    # With the Session object, we can just store the token in the headers dictionary, but you can pass them directly to each request as well.
    import requests

    cookie = "_|WARNING:-DO-NOT-SHARE-THIS.--Sharing-this-will-allow-someone-to-log-in-as-you-and-to-steal-your-ROBUX-and-items.|_TOKEN"

    session = requests.Session()
    session.cookies[".ROBLOSECURITY"] = cookie

    # send first request
    req = session.post(
        url="https://auth.roblox.com/"
    )

    if "X-CSRF-Token" in req.headers:  # check if token is in response headers
        session.headers["X-CSRF-Token"] = req.headers["X-CSRF-Token"]  # store the response header in the session

    # send second request
    req2 = session.post(
        url="https://auth.roblox.com/"
    )

    print("First:", req.status_code)
    print("Second:", req2.status_code)
    ```
=== "Ruby"
    Requires the [http.rb](https://github.com/httprb/http) and [json](https://github.com/flori/json) gems.
    ```rb
    require "http"
    require "json"

    COOKIE = "_|WARNING:-DO-NOT-SHARE-THIS.--Sharing-this-will-allow-someone-to-log-in-as-you-and-to-steal-your-ROBUX-and-items.|_TOKEN"

    client = HTTP.cookies({
        ".ROBLOSECURITY": COOKIE
    })

    first_response = client.post("https://auth.roblox.com")

    client = client.headers({
        "x-csrf-token": first_response.headers["x-csrf-token"]
    })

    second_response = client.post("https://auth.roblox.com")

    puts "First: #{first_response.status}"
    puts "Second: #{second_response.status}"
    ```
=== "JavaScript"
    If your runtime doesn't support native fetch, like for example pre-v21 Node.js, you may need to install a package
    like [node-fetch](https://www.npmjs.com/package/node-fetch).
    ```js
    const COOKIE = "_|WARNING:-DO-NOT-SHARE-THIS.--Sharing-this-will-allow-someone-to-log-in-as-you-and-to-steal-your-ROBUX-and-items.|_TOKEN";

    const firstResponse = await fetch("https://auth.roblox.com", {
        headers: {
            Cookie: `.ROBLOSECURITY=${COOKIE};`,
            "Content-Length": "0",
        },
        method: "POST",
    });

    const secondResponse = await fetch("https://auth.roblox.com", {
        headers: {
            Cookie: `.ROBLOSECURITY=${COOKIE};`,
            "x-csrf-token": firstResponse.headers.get("x-csrf-token"),
            "Content-Length": "0",
        },
        method: "POST",
    });

    console.log(`First: ${firstResponse.status}`);
    console.log(`Second: ${secondResponse.status}`);
    ```
=== "Rust"
    Requires the [reqwest](https://docs.rs/reqwest/latest/reqwest/) and [tokio](https://docs.rs/tokio/latest/tokio/) crates.
    ```rs
    use reqwest::header::HeaderMap;
    use reqwest::Client;

    const COOKIE: &str = "_|WARNING:-DO-NOT-SHARE-THIS.--Sharing-this-will-allow-someone-to-log-in-as-you-and-to-steal-your-ROBUX-and-items.|_TOKEN";

    #[tokio::main]
    async fn main() {
        let client = Client::new();
        let mut headers = HeaderMap::new();
        headers.insert(
            "Cookie",
            format!(".ROBLOSECURITY={};", COOKIE).parse().unwrap(),
        );

        let first_response = client
            .post("https://auth.roblox.com")
            .headers(headers)
            .send()
            .await
            .unwrap();

        let mut headers = HeaderMap::new();
        headers.insert(
            "Cookie",
            format!(".ROBLOSECURITY={};", COOKIE).parse().unwrap(),
        );
        headers.insert(
            "x-csrf-token",
            first_response
                .headers()
                .get("x-csrf-token")
                .unwrap()
                .to_str()
                .unwrap()
                .parse()
                .unwrap(),
        );

        let second_response = client
            .post("https://auth.roblox.com")
            .headers(headers)
            .send()
            .await
            .unwrap();

        println!("First: {}", first_response.status());
        println!("Second: {}", second_response.status());
    }
    ```
=== "C#"
    ```cs
    using System.Net;
    using System.Net.Http;
    using System.Net.Http.Headers;
    using System.Text.Json;

    const string COOKIE = "_|WARNING:-DO-NOT-SHARE-THIS.--Sharing-this-will-allow-someone-to-log-in-as-you-and-to-steal-your-ROBUX-and-items.|_TOKEN";

    var cookieContainer = new CookieContainer();
    cookieContainer.Add(new Cookie(".ROBLOSECURITY", COOKIE) { Domain = ".roblox.com" });

    var httpClient = new HttpClient(new HttpClientHandler() { UseCookies = true, CookieContainer = cookieContainer });

    var firstResponse = await httpClient.PostAsync("https://auth.roblox.com/", null);
    httpClient.DefaultRequestHeaders.Add("x-csrf-token", firstResponse.Headers.GetValues("x-csrf-token").First());

    var requestMessage = new HttpRequestMessage(HttpMethod.Post, "https://auth.roblox.com/");
    requestMessage.Headers.Accept.Add(new MediaTypeWithQualityHeaderValue("application/json"));
    var secondResponse = await httpClient.SendAsync(requestMessage);

    Console.WriteLine($"First: {firstResponse.StatusCode}");
    Console.WriteLine($"Second: {secondResponse.StatusCode}");
    ```
=== "Elixir"
    Compile with `iex -S mix`, then execute `RobloxAPI.main`.

    Dependencies: [httpoison](https://hex.pm/packages/httpoison) and [poison](https://hex.pm/packages/poison)
    ```ex
    defmodule RobloxAPI do
      @roblosecurity "_|WARNING:-DO-NOT-SHARE-THIS.--Sharing-this-will-allow-someone-to-log-in-as-you-and-to-steal-your-ROBUX-and-items.|_TOKEN"
    
      def main do
        headers = %{Cookie: ".ROBLOSECURITY=#{@roblosecurity};"}
    
        with {:ok, first_response} <- HTTPoison.post("https://auth.roblox.com", "", headers),
             {:ok, second_response} <-
               HTTPoison.post(
                 "https://auth.roblox.com",
                 "",
                 Map.put(
                   headers,
                   :"x-csrf-token",
                   # response.headers is a list of tuples, where the first element is the header name and the second the header's value
                   Enum.find_value(first_response.headers, fn {name, value} ->
                     if name == "x-csrf-token", do: value
                   end)
                 )
               ) do
          IO.puts("First: #{first_response.status_code}")
          IO.puts("Second: #{second_response.status_code}")
        else
          {:error, error} ->
            IO.inspect(error)
        end
      end
    end
    ```

This program will send one request, check if the X-CSRF-Token was present in the response, and if so will store it back
into the session's headers. We then repeat the first request again, and then outputs the status codes from both requests.

This code should output something like the following:

    First: 403
    Second: 200

This solution works - but it doesn't scale well. If we want to properly do this, we’ll put all of this logic in a function
that handles our requests for us and then call that when sending requests. This is (essentially) what the request wrappers
in Roblox API wrapper libraries do.

## Request function

Here's an example of a function that does what we need: 

=== "Python"
    ```py
    import requests

    cookie = "_|WARNING:-DO-NOT-SHARE-THIS.--Sharing-this-will-allow-someone-to-log-in-as-you-and-to-steal-your-ROBUX-and-items.|_TOKEN"

    session = requests.Session()
    session.cookies[".ROBLOSECURITY"] = cookie


    def rbx_request(method, url, **kwargs):
        request = session.request(method, url, **kwargs)
        method = method.lower()
        if method in {"post", "put", "patch", "delete"}:
            if "X-CSRF-TOKEN" in request.headers:
                session.headers["X-CSRF-TOKEN"] = request.headers["X-CSRF-TOKEN"]
                if request.status_code == 403:
                    body = request.body()
                    if body.get("code", -1) == 0: # Request failed, send it again
                        request = session.request(method, url, **kwargs)
        return request


    req = rbx_request("POST", "https://auth.roblox.com/")
    print(req.status_code)
    ```
=== "F#"
    ```f#
    open System.Net
    open System.Net.Http
    open System.Text.Json

    type Error = { code: int; message: string }

    [<Literal>]
    let COOKIE =
        "_|WARNING:-DO-NOT-SHARE-THIS.--Sharing-this-will-allow-someone-to-log-in-as-you-and-to-steal-your-ROBUX-and-items.|_TOKEN"

    let cookieContainer = CookieContainer()
    cookieContainer.Add(Cookie(".ROBLOSECURITY", COOKIE, Domain = ".roblox.com"))

    let httpClient =
        new HttpClient(new HttpClientHandler(UseCookies = true, CookieContainer = cookieContainer))

    let rec rbxRequest (method: HttpMethod) (url: string) (body: 'a option) =
        task {
            let! response =
                httpClient.SendAsync(
                    new HttpRequestMessage(
                        method,
                        url,
                        Content =
                            new StringContent(
                                JsonSerializer.Serialize(
                                    body
                                    |> Option.defaultValue Unchecked.defaultof<'a>
                                ),
                                System.Text.Encoding.UTF8,
                                "application/json"
                            )
                    )
                )

            if response.StatusCode = HttpStatusCode.Forbidden then
                let! content = response.Content.ReadAsStreamAsync()
                let! error = JsonSerializer.DeserializeAsync<Error> content

                if error.code = 0 then
                    httpClient.DefaultRequestHeaders.Add(
                        "x-csrf-token",
                        response.Headers.GetValues "x-csrf-token"
                        |> Seq.head
                    )

                    return! rbxRequest method url body
                else
                    return Error response
            else
                return Ok response
        }

    (task {
        let! result = rbxRequest HttpMethod.Get "https://auth.roblox.com/" None

        match result with
        | Ok response -> printfn "%d" (int response.StatusCode)
        | Error error -> failwithf "Expected status code 200, got %d" (int error.StatusCode)
    })
        .Wait()
    ```
=== "Ruby"
    Requires the [http.rb](https://github.com/httprb/http) and [json](https://github.com/flori/json) gems.
    ```rb
    require "http"
    require "json"
    
    COOKIE = "_|WARNING:-DO-NOT-SHARE-THIS.--Sharing-this-will-allow-someone-to-log-in-as-you-and-to-steal-your-ROBUX-and-items.|_TOKEN"
    METHODS = %i(post put patch delete)
    
    module APIHelper
      @client = HTTP.cookies({
        :".ROBLOSECURITY" => COOKIE
      })
    
      def self.rbx_request(verb, url, *args)
        response = @client.request(verb, url, *args)
    
        if METHODS.include?(verb) and response.headers.include?("x-csrf-token")
          @client = @client.headers({
            "x-csrf-token": response.headers["x-csrf-token"]
          })
    
          if response.status == 403
            body = JSON.parse(response.body)
            if body["code"] == 0
              response = rbx_request(verb, url, *args)
            end
          end
        end
    
        response
      end
    end
    
    response = APIHelper.rbx_request(:post, "https://auth.roblox.com")
    puts response.status
    ```
=== "JavaScript"
    If your runtime doesn't support native fetch, like for example pre-v21 Node.js, you may need to install a package
    like [node-fetch](https://www.npmjs.com/package/node-fetch).
    ```js
    const COOKIE = "_|WARNING:-DO-NOT-SHARE-THIS.--Sharing-this-will-allow-someone-to-log-in-as-you-and-to-steal-your-ROBUX-and-items.|_TOKEN";

    let xCsrfToken = "";

    const rbxRequest = async (verb, url, body) => {
        verb = verb.toUpperCase();
        let response = await fetch(url, {
            headers: {
                Cookie: `.ROBLOSECURITY=${COOKIE};`,
                "x-csrf-token": xCsrfToken,
                "Content-Length": (body?.length ?? 0).toString(),
            },
            method: verb,
            body: body || "",
        });
        if (
            ["POST", "PUT", "PATCH", "DELETE"].includes(verb) &&
            response.headers.has("x-csrf-token")
        ) {
            xCsrfToken = response.headers.get("x-csrf-token");
            if (response.status == 403) {
                const responseBody = await response.json();
                if (responseBody.code === 0)
                    response = await rbxRequest(verb, url, body);
            }
        }
        return response;
    };

    const response = await rbxRequest("POST", "https://auth.roblox.com");
    console.log(response.status);
    ```
=== "Rust"
    Requires the [reqwest](https://docs.rs/reqwest/latest/reqwest/), [tokio](https://docs.rs/tokio/latest/tokio/),
    [serde_json](https://docs.rs/serde_json/latest/serde_json/) and [lazy_static](https://docs.rs/lazy_static/latest/lazy_static/)
    crates.
    ```rs
    use lazy_static::lazy_static;
    use reqwest::{header::HeaderMap, Method};
    use reqwest::{Client, Response, StatusCode};
    use serde_json::{from_str, Value};
    use std::sync::{Arc, Mutex};

    const COOKIE: &str = "_|WARNING:-DO-NOT-SHARE-THIS.--Sharing-this-will-allow-someone-to-log-in-as-you-and-to-steal-your-ROBUX-and-items.|_TOKEN";

    lazy_static! {
        static ref HTTP_CLIENT: Client = Client::new();
        static ref HEADERS: Arc<Mutex<HeaderMap>> = Arc::new(Mutex::new({
            let mut headers = HeaderMap::new();
            headers.insert(
                "Cookie",
                format!(".ROBLOSECURITY={};", COOKIE).parse().unwrap(),
            );
            headers
        }));
    }

    async fn request(verb: Method, url: String, body: Option<Value>) -> Result<Response, ()> {
        let arc_ref = HEADERS.clone(); // get reference to the arc here so it lives as long as headers
        let mut headers = arc_ref.lock().unwrap();

        let response = HTTP_CLIENT
            .request(verb.clone(), url.clone())
            .headers(headers.clone())
            .json(&body)
            .send()
            .await
            .unwrap();

        // this is kinda botched because on the branch where .text() is called and the code is not 0, there's no response to
        // return because it's consumed, you can upgrade it to return the errors instead of unit, see for example
        // https://github.com/zmadie/oxid_roblox/blob/5fbe3553871c048158d54269e3e0d57dbc78ab97/src/util/api_helper.rs#L54-L67
        if let Some(x_csrf_token) = response.headers().get("x-csrf-token").cloned() {
            headers.insert("x-csrf-token", x_csrf_token);
            if response.status() == StatusCode::FORBIDDEN {
                let body: Value = from_str(&response.text().await.unwrap()).unwrap();
                if body["code"].as_i64().unwrap() == 0 {
                    return Ok(HTTP_CLIENT
                        .request(verb, url)
                        .headers(headers.clone())
                        .json(&body)
                        .send()
                        .await
                        .unwrap());
                } else {
                    return Err(());
                }
            }
        }
        Ok(response)
    }

    #[tokio::main]
    async fn main() {
        let response = request(
            Method::GET,
            "https://users.roblox.com/v1/users/1".to_string(),
            None,
        )
        .await
        .unwrap();

        println!("{}", response.status());
    }
    ```
=== "C#"
    ```cs
    using System.Net;
    using System.Net.Http;
    using System.Text.Json;

    const string COOKIE = "_|WARNING:-DO-NOT-SHARE-THIS.--Sharing-this-will-allow-someone-to-log-in-as-you-and-to-steal-your-ROBUX-and-items.|_TOKEN";

    var cookieContainer = new CookieContainer();
    cookieContainer.Add(new Cookie(".ROBLOSECURITY", COOKIE) { Domain = ".roblox.com" });

    var httpClient = new HttpClient(new HttpClientHandler() { UseCookies = true, CookieContainer = cookieContainer });
    async Task<HttpResponseMessage> Request(HttpMethod method, string url, dynamic body = null)
    {
        var response = await httpClient.SendAsync(
            new HttpRequestMessage(
                method,
                url
            )
            {
                Content =
                    new StringContent(
                        JsonSerializer.Serialize(body ?? new { }),
                        Encoding.UTF8,
                        "application/json"
                    )
            }

        );
        if (response.StatusCode == HttpStatusCode.Forbidden)
        {
            dynamic error = await JsonSerializer.DeserializeAsync<dynamic>(await response.Content.ReadAsStreamAsync());
            if (error.GetProperty("code").GetInt32() == 0)
            {
                httpClient.DefaultRequestHeaders.Add("x-csrf-token", response.Headers.GetValues("x-csrf-token").First());
                return await Request(method, url, body);
            }
        }

        return response;
    }

    var response = await Request(HttpMethod.Post, "https://auth.roblox.com");
    Console.WriteLine(response.StatusCode);
    ```
=== "Elixir"
    Compile with `iex -S mix`, then execute `RobloxAPI.main`.

    Dependencies: [httpoison](https://hex.pm/packages/httpoison) and [poison](https://hex.pm/packages/poison)
    ```elixir
    defmodule RobloxAPI do
      use Agent
    
      @roblosecurity "_|WARNING:-DO-NOT-SHARE-THIS.--Sharing-this-will-allow-someone-to-log-in-as-you-and-to-steal-your-ROBUX-and-items.|_TOKEN"
    
      def start_link do
        Agent.start_link(fn -> %{Cookie: ".ROBLOSECURITY=#{@roblosecurity}"} end, name: __MODULE__)
      end
    
      defp request(verb, url, body \\ %{}) do
        headers = Agent.get(__MODULE__, & &1)
    
        with {:ok, encoded_body} <- Poison.encode(body),
             {:ok, %HTTPoison.Response{status_code: 403} = response} <-
               HTTPoison.request(verb, url, encoded_body, headers),
             {:ok, body} <-
               Poison.decode(response.body) do
          xcsrf_token =
            Enum.find_value(response.headers, fn {name, value} ->
              if name == "x-csrf-token", do: value
            end)
    
          if xcsrf_token == nil do
            {:ok, response}
          else
            headers = Map.put(headers, :"x-csrf-token", xcsrf_token)
            Agent.update(__MODULE__, fn _ -> headers end)
    
            if body["code"] == 0 do
              request(verb, url, body)
            else
              {:ok, response}
            end
          end
        else
          {:ok, response} ->
            {:ok, response}
    
          {:error, error} ->
            {:error, error}
        end
      end
    
      def main do
        start_link()
    
        case request(:post, "https://auth.roblox.com") do
          {:ok, %HTTPoison.Response{status_code: status_code}} ->
            IO.puts(status_code)
    
          {:error, error} ->
            IO.inspect(error)
        end
      end
    end
    ```

This code should output something like the following:

    200

Now that we’ve done this, sending any kind of requests to the API is boilerplate-less.
