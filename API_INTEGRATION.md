![Archway](https://adaptableapps.net/images/Archway_Logo_v1_Banner_White_On_Black.svg)

# API integration

Archway has a REST API for connecting your own systems to it. This page documents the **subscription check**:
a single call that tells your software whether one of your customers' subscriptions is currently active.

Typical uses:

- your application checks, when a customer signs in or starts it up, that their subscription is still paid up;
- a licence server unlocks features only while a subscription is active;
- a no-code automation (Zapier, Make, n8n and similar) branches on whether a customer is subscribed.

Questions are welcome at [support@adaptableapps.net](mailto:support@adaptableapps.net).

---

## Contents

- [How it works](#how-it-works)
- [What you need](#what-you-need)
- [Base URL](#base-url)
- [The request](#the-request)
- [The response](#the-response)
- [Status codes](#status-codes)
- [Examples](#examples)
- [No-code tools - Zapier, Make, n8n](#no-code-tools---zapier-make-n8n)
- [Keeping the keys safe](#keeping-the-keys-safe)
- [Good to know](#good-to-know)

---

## How it works

Every **customer account** in Archway has a secret key, and so does every **subscription**. Together, the two
keys identify one subscription belonging to one customer - and they are the *only* credentials the check needs.
There is no separate API key, token or sign-in.

You send the two keys, plus your tenancy's subdomain, and Archway answers with whether that subscription is
active right now.

---

## What you need

| Value | What it is | Where to find it |
|---|---|---|
| **Tenancy subdomain** | Your tenancy's subdomain - the first part of your Archway address. For `https://yourcompany.prod.us.app.archwayportal.com` it is `yourcompany`. | Your Archway address, or the tenancy's details in Tenant Center. |
| **Customer account secret key** | The secret key of the customer account that owns the subscription. | Open the customer account and choose **Copy Key** from the menu at the top right. |
| **Subscription secret key** | The secret key of the subscription itself. | Open the subscription (for example from **My Subscriptions**) and choose **Copy Key** from the menu at the top right. |

Both pages also offer **Regenerate Key**, which replaces the key with a new one. The old key stops working
immediately - see [Good to know](#good-to-know).

In practice your **customer** usually holds the two keys - you hand them over when they subscribe, or they copy
them from their own account - and enters them into your software, which then makes the check.

---

## Base URL

| Environment | Base URL |
|---|---|
| Production | `https://prod.us.api.archwayportal.com` |

All calls are made over **HTTPS**. Plain HTTP is not supported.

---

## The request

```
POST {base URL}/Sdk/CheckSubscriptionAsync
Content-Type: application/json
Accept: application/json
```

The body is a JSON object with three fields, all **required**:

| Field | Type | Description |
|---|---|---|
| `TntSubdomain` | string | Your tenancy subdomain, e.g. `yourcompany`. |
| `CustomerAccountSecretKey` | string | The customer account's secret key. |
| `SubscriptionSecretKey` | string | The subscription's secret key. |

```json
{
  "TntSubdomain": "yourcompany",
  "CustomerAccountSecretKey": "ab572cbc-bcf0-46b0-9535-80a6adfb18c1",
  "SubscriptionSecretKey": "00709b50-e7dc-4f6f-a8d1-82f13b057c91"
}
```

Field names are **not case-sensitive**: `subscriptionSecretKey` works just as well as `SubscriptionSecretKey`.

No other headers are needed - in particular, **no API key or `Authorization` header**. The keys travel in the
body, which HTTPS encrypts.

---

## The response

Every answer is a JSON object with the same two fields:

| Field | Type | Description |
|---|---|---|
| `IsSubscriptionActive` | boolean | `true` only when the keys matched **and** the subscription is active. Otherwise `false`. |
| `ResponseMessage` | string | A short, human-readable explanation. Meant for logs and people - do not parse it. |

```json
{
  "IsSubscriptionActive": true,
  "ResponseMessage": "Subscription is active."
}
```

**Decide on `IsSubscriptionActive` together with the HTTP status code** - see the next section.

---

## Status codes

This endpoint uses the HTTP status code to tell you what happened, so tools that only look at the status can
still react correctly.

| Status | Meaning | `IsSubscriptionActive` | `ResponseMessage` | What to do |
|---|---|---|---|---|
| **200 OK** | The keys matched and the subscription was checked. | `true` or `false` | `Subscription is active.` / `Subscription is not active.` | Use `IsSubscriptionActive`. `false` means the subscription exists but is not active - for example cancelled, or a payment has failed. |
| **400 Bad Request** | The request was incomplete or was not valid JSON. | `false` | `Invalid request. ...` | Fix the request - a field is missing or empty. |
| **401 Unauthorized** | The keys did not identify a subscription. | `false` | `Invalid credentials.` | Check the subdomain and both keys. A key may have been regenerated. |
| **500 Internal Server Error** | The check could not be completed on our side. | `false` | `The subscription could not be checked. Please try again later.` | Retry later. **Do not treat this as "not active".** |
| **403 Forbidden** | Blocked before reaching Archway - usually too many requests from one address. | - | - | The body may not be JSON. Slow down and retry after a few minutes. |

`401` deliberately does not say *which* value was wrong - an unknown subdomain, a wrong customer account key
and a wrong subscription key all give the same answer. That is on purpose, so the check cannot be used to find
out which keys or tenancies exist.

### Deciding what to do

```
200 and IsSubscriptionActive = true    ->  active: allow access
200 and IsSubscriptionActive = false   ->  not active: refuse, and tell the customer their subscription has lapsed
401                                    ->  wrong keys: ask the customer to re-enter them
400                                    ->  a bug in your request
500, 403, timeout or no connection     ->  could not tell: retry later; do not lock the customer out on this alone
```

For an application that runs offline or must not lock a paying customer out during an outage, a common
approach is to remember the last successful answer and only act on a definite `200` or `401`.

---

## Examples

All examples use the production base URL - replace the values with
your own.

### curl

```bash
curl -sS -X POST "https://prod.us.api.archwayportal.com/Sdk/CheckSubscriptionAsync" \
  -H "Content-Type: application/json" \
  -H "Accept: application/json" \
  -d '{
        "TntSubdomain": "yourcompany",
        "CustomerAccountSecretKey": "<customer account secret key>",
        "SubscriptionSecretKey": "<subscription secret key>"
      }' \
  -w "\nHTTP %{http_code}\n"
```

### PowerShell 7

`-SkipHttpErrorCheck` keeps the body of a `400`/`401` instead of throwing it away.

```powershell
$body = @{
  TntSubdomain             = "yourcompany"
  CustomerAccountSecretKey = "<customer account secret key>"
  SubscriptionSecretKey    = "<subscription secret key>"
} | ConvertTo-Json

$response = Invoke-WebRequest -Method Post `
  -Uri "https://prod.us.api.archwayportal.com/Sdk/CheckSubscriptionAsync" `
  -ContentType "application/json" -Body $body -SkipHttpErrorCheck

$result = $response.Content | ConvertFrom-Json
"HTTP $($response.StatusCode) - active: $($result.IsSubscriptionActive) - $($result.ResponseMessage)"
```

### C# (.NET)

```csharp
using System.Net.Http.Json;

// Fine for a one-off check. In an application that checks repeatedly, reuse one HttpClient (or use
// IHttpClientFactory) instead of creating and disposing one per call.
using var http = new HttpClient { Timeout = TimeSpan.FromSeconds(30) };

var response = await http.PostAsJsonAsync(
  "https://prod.us.api.archwayportal.com/Sdk/CheckSubscriptionAsync",
  new
  {
    TntSubdomain = "yourcompany",
    CustomerAccountSecretKey = "<customer account secret key>",
    SubscriptionSecretKey = "<subscription secret key>"
  });

var result = await response.Content.ReadFromJsonAsync<SubscriptionCheckResult>();

var isActive = response.StatusCode == System.Net.HttpStatusCode.OK && result?.IsSubscriptionActive == true;

public record SubscriptionCheckResult(bool IsSubscriptionActive, string? ResponseMessage);
```

### JavaScript (Node.js 18+)

```javascript
const response = await fetch("https://prod.us.api.archwayportal.com/Sdk/CheckSubscriptionAsync", {
  method: "POST",
  headers: { "Content-Type": "application/json", "Accept": "application/json" },
  body: JSON.stringify({
    TntSubdomain: "yourcompany",
    CustomerAccountSecretKey: "<customer account secret key>",
    SubscriptionSecretKey: "<subscription secret key>"
  })
});

const result = await response.json().catch(() => null);
const isActive = response.status === 200 && result?.IsSubscriptionActive === true;
```

### Python

```python
import requests

response = requests.post(
    "https://prod.us.api.archwayportal.com/Sdk/CheckSubscriptionAsync",
    json={
        "TntSubdomain": "yourcompany",
        "CustomerAccountSecretKey": "<customer account secret key>",
        "SubscriptionSecretKey": "<subscription secret key>",
    },
    timeout=30,
)

result = response.json() if response.headers.get("content-type", "").startswith("application/json") else None
is_active = response.status_code == 200 and bool(result and result.get("IsSubscriptionActive"))
```

### TypeScript (Node.js 18+, Deno, Bun)

Returns one of four outcomes, matching [Deciding what to do](#deciding-what-to-do).

```typescript
interface SubscriptionCheckRequest {
  TntSubdomain: string;
  CustomerAccountSecretKey: string;
  SubscriptionSecretKey: string;
}

interface SubscriptionCheckResult {
  IsSubscriptionActive: boolean;
  ResponseMessage?: string;
}

type SubscriptionStatus = "active" | "not-active" | "invalid-credentials" | "unknown";

async function checkSubscription(request: SubscriptionCheckRequest): Promise<SubscriptionStatus> {
  try {
    const response = await fetch("https://prod.us.api.archwayportal.com/Sdk/CheckSubscriptionAsync", {
      method: "POST",
      headers: { "Content-Type": "application/json", "Accept": "application/json" },
      body: JSON.stringify(request),
      signal: AbortSignal.timeout(30_000),
    });

    if (response.status === 401) return "invalid-credentials";
    if (response.status !== 200) return "unknown"; // 400, 403, 500 ...

    const result = (await response.json()) as SubscriptionCheckResult;
    return result.IsSubscriptionActive === true ? "active" : "not-active";
  } catch {
    return "unknown"; // timeout, no connection, or a body that is not JSON
  }
}

checkSubscription({
  TntSubdomain: "yourcompany",
  CustomerAccountSecretKey: "<customer account secret key>",
  SubscriptionSecretKey: "<subscription secret key>",
}).then((subscriptionStatus) => console.log(subscriptionStatus));
```

### F# (.NET)

```fsharp
open System
open System.Net
open System.Net.Http
open System.Net.Http.Json

type SubscriptionCheckResult = { IsSubscriptionActive: bool; ResponseMessage: string }

let check () =
    task {
        // Fine for a one-off check. In an application that checks repeatedly, reuse one HttpClient (or use
        // IHttpClientFactory) instead of creating and disposing one per call.
        use http = new HttpClient(Timeout = TimeSpan.FromSeconds 30.)

        let! response =
            http.PostAsJsonAsync(
                "https://prod.us.api.archwayportal.com/Sdk/CheckSubscriptionAsync",
                {| TntSubdomain = "yourcompany"
                   CustomerAccountSecretKey = "<customer account secret key>"
                   SubscriptionSecretKey = "<subscription secret key>" |})

        let! result =
            task {
                try
                    let! r = response.Content.ReadFromJsonAsync<SubscriptionCheckResult>()
                    return (match box r with null -> None | _ -> Some r)
                with _ ->
                    return None // not JSON - e.g. a 403 from the firewall
            }

        let isActive =
            response.StatusCode = HttpStatusCode.OK
            && (result |> Option.exists (fun r -> r.IsSubscriptionActive))

        printfn "HTTP %d - active: %b" (int response.StatusCode) isActive
    }

check().GetAwaiter().GetResult()
```

### Java (11+)

Uses the built-in `java.net.http.HttpClient`, and [Jackson](https://github.com/FasterXML/jackson-databind)
(`com.fasterxml.jackson.core:jackson-databind`) for JSON.

```java
import com.fasterxml.jackson.databind.JsonNode;
import com.fasterxml.jackson.databind.ObjectMapper;

import java.net.URI;
import java.net.http.HttpClient;
import java.net.http.HttpRequest;
import java.net.http.HttpResponse;
import java.time.Duration;
import java.util.Map;

public class SubscriptionCheck {
  public static void main(String[] args) throws Exception {
    ObjectMapper mapper = new ObjectMapper();

    String body = mapper.writeValueAsString(Map.of(
        "TntSubdomain", "yourcompany",
        "CustomerAccountSecretKey", "<customer account secret key>",
        "SubscriptionSecretKey", "<subscription secret key>"));

    HttpRequest request = HttpRequest.newBuilder(URI.create("https://prod.us.api.archwayportal.com/Sdk/CheckSubscriptionAsync"))
        .timeout(Duration.ofSeconds(30))
        .header("Content-Type", "application/json")
        .header("Accept", "application/json")
        .POST(HttpRequest.BodyPublishers.ofString(body))
        .build();

    HttpResponse<String> response = HttpClient.newHttpClient().send(request, HttpResponse.BodyHandlers.ofString());

    JsonNode result = null;
    try {
      result = mapper.readTree(response.body());
    } catch (Exception notJson) {
      // e.g. a 403 from the firewall
    }

    boolean isActive = response.statusCode() == 200
        && result != null
        && result.path("IsSubscriptionActive").asBoolean(false);

    System.out.println("HTTP " + response.statusCode() + " - active: " + isActive);
  }
}
```

### Kotlin (JVM)

Uses the built-in `java.net.http.HttpClient`, and
[kotlinx.serialization](https://github.com/Kotlin/kotlinx.serialization) (`kotlinx-serialization-json`, with the
serialization plugin) for JSON.

```kotlin
import kotlinx.serialization.Serializable
import kotlinx.serialization.encodeToString
import kotlinx.serialization.json.Json
import kotlinx.serialization.json.booleanOrNull
import kotlinx.serialization.json.jsonObject
import kotlinx.serialization.json.jsonPrimitive
import java.net.URI
import java.net.http.HttpClient
import java.net.http.HttpRequest
import java.net.http.HttpResponse
import java.time.Duration

@Serializable
data class SubscriptionCheckRequest(
    val TntSubdomain: String,
    val CustomerAccountSecretKey: String,
    val SubscriptionSecretKey: String,
)

fun main() {
    val body = Json.encodeToString(
        SubscriptionCheckRequest(
            TntSubdomain = "yourcompany",
            CustomerAccountSecretKey = "<customer account secret key>",
            SubscriptionSecretKey = "<subscription secret key>",
        )
    )

    val request = HttpRequest.newBuilder(URI.create("https://prod.us.api.archwayportal.com/Sdk/CheckSubscriptionAsync"))
        .timeout(Duration.ofSeconds(30))
        .header("Content-Type", "application/json")
        .header("Accept", "application/json")
        .POST(HttpRequest.BodyPublishers.ofString(body))
        .build()

    val response = HttpClient.newHttpClient().send(request, HttpResponse.BodyHandlers.ofString())

    // null when the body is not JSON - e.g. a 403 from the firewall
    val result = runCatching { Json.parseToJsonElement(response.body()).jsonObject }.getOrNull()

    val isActive = response.statusCode() == 200 &&
        result?.get("IsSubscriptionActive")?.jsonPrimitive?.booleanOrNull == true

    println("HTTP ${response.statusCode()} - active: $isActive")
}
```

### Go

Standard library only.

```go
package main

import (
	"bytes"
	"encoding/json"
	"fmt"
	"net/http"
	"time"
)

type subscriptionCheckRequest struct {
	TntSubdomain             string `json:"TntSubdomain"`
	CustomerAccountSecretKey string `json:"CustomerAccountSecretKey"`
	SubscriptionSecretKey    string `json:"SubscriptionSecretKey"`
}

type subscriptionCheckResult struct {
	IsSubscriptionActive bool   `json:"IsSubscriptionActive"`
	ResponseMessage      string `json:"ResponseMessage"`
}

func main() {
	body, err := json.Marshal(subscriptionCheckRequest{
		TntSubdomain:             "yourcompany",
		CustomerAccountSecretKey: "<customer account secret key>",
		SubscriptionSecretKey:    "<subscription secret key>",
	})
	if err != nil {
		panic(err)
	}

	client := &http.Client{Timeout: 30 * time.Second}

	response, err := client.Post(
		"https://prod.us.api.archwayportal.com/Sdk/CheckSubscriptionAsync",
		"application/json",
		bytes.NewReader(body),
	)
	if err != nil {
		fmt.Println("could not tell:", err) // timeout or no connection - not "not active"
		return
	}
	defer response.Body.Close()

	var result subscriptionCheckResult
	_ = json.NewDecoder(response.Body).Decode(&result) // a body that is not JSON leaves result at its zero value

	isActive := response.StatusCode == http.StatusOK && result.IsSubscriptionActive

	fmt.Printf("HTTP %d - active: %t - %s\n", response.StatusCode, isActive, result.ResponseMessage)
}
```

### Rust

Uses [reqwest](https://crates.io/crates/reqwest), [serde](https://crates.io/crates/serde) and
[tokio](https://crates.io/crates/tokio):

```toml
[dependencies]
reqwest = { version = "0.12", features = ["json"] }
serde = { version = "1", features = ["derive"] }
tokio = { version = "1", features = ["full"] }
```

```rust
use serde::{Deserialize, Serialize};
use std::time::Duration;

#[derive(Serialize)]
#[serde(rename_all = "PascalCase")]
struct SubscriptionCheckRequest<'a> {
    tnt_subdomain: &'a str,
    customer_account_secret_key: &'a str,
    subscription_secret_key: &'a str,
}

#[derive(Deserialize)]
#[serde(rename_all = "PascalCase")]
struct SubscriptionCheckResult {
    is_subscription_active: bool,
    response_message: Option<String>,
}

#[tokio::main]
async fn main() -> Result<(), reqwest::Error> {
    let client = reqwest::Client::builder()
        .timeout(Duration::from_secs(30))
        .build()?;

    let response = client
        .post("https://prod.us.api.archwayportal.com/Sdk/CheckSubscriptionAsync")
        .json(&SubscriptionCheckRequest {
            tnt_subdomain: "yourcompany",
            customer_account_secret_key: "<customer account secret key>",
            subscription_secret_key: "<subscription secret key>",
        })
        .send()
        .await?;

    let status = response.status();
    // None when the body is not JSON - e.g. a 403 from the firewall
    let result = response.json::<SubscriptionCheckResult>().await.ok();

    let is_active = status == reqwest::StatusCode::OK
        && result.as_ref().map_or(false, |r| r.is_subscription_active);

    println!(
        "HTTP {} - active: {} - {}",
        status.as_u16(),
        is_active,
        result.and_then(|r| r.response_message).unwrap_or_default()
    );

    Ok(())
}
```

### Swift (iOS 15+, macOS 12+)

The request field names are not case-sensitive, so ordinary Swift property names can be sent as they are; the
response is mapped back with `CodingKeys`.

```swift
import Foundation

struct SubscriptionCheckRequest: Encodable {
    let tntSubdomain: String
    let customerAccountSecretKey: String
    let subscriptionSecretKey: String
}

struct SubscriptionCheckResult: Decodable {
    let isSubscriptionActive: Bool
    let responseMessage: String?

    enum CodingKeys: String, CodingKey {
        case isSubscriptionActive = "IsSubscriptionActive"
        case responseMessage = "ResponseMessage"
    }
}

func checkSubscription() async throws -> Bool {
    var request = URLRequest(url: URL(string: "https://prod.us.api.archwayportal.com/Sdk/CheckSubscriptionAsync")!)
    request.httpMethod = "POST"
    request.timeoutInterval = 30
    request.setValue("application/json", forHTTPHeaderField: "Content-Type")
    request.setValue("application/json", forHTTPHeaderField: "Accept")
    request.httpBody = try JSONEncoder().encode(
        SubscriptionCheckRequest(
            tntSubdomain: "yourcompany",
            customerAccountSecretKey: "<customer account secret key>",
            subscriptionSecretKey: "<subscription secret key>"
        )
    )

    let (data, response) = try await URLSession.shared.data(for: request)
    let status = (response as? HTTPURLResponse)?.statusCode ?? 0

    // nil when the body is not JSON - e.g. a 403 from the firewall
    let result = try? JSONDecoder().decode(SubscriptionCheckResult.self, from: data)

    return status == 200 && result?.isSubscriptionActive == true
}
```

### Objective-C

```objectivec
#import <Foundation/Foundation.h>

NSURL *url = [NSURL URLWithString:@"https://prod.us.api.archwayportal.com/Sdk/CheckSubscriptionAsync"];

NSMutableURLRequest *request = [NSMutableURLRequest requestWithURL:url];
request.HTTPMethod = @"POST";
request.timeoutInterval = 30;
[request setValue:@"application/json" forHTTPHeaderField:@"Content-Type"];
[request setValue:@"application/json" forHTTPHeaderField:@"Accept"];

NSDictionary *body = @{
  @"TntSubdomain": @"yourcompany",
  @"CustomerAccountSecretKey": @"<customer account secret key>",
  @"SubscriptionSecretKey": @"<subscription secret key>"
};
request.HTTPBody = [NSJSONSerialization dataWithJSONObject:body options:0 error:nil];

NSURLSessionDataTask *task = [[NSURLSession sharedSession] dataTaskWithRequest:request
                                                             completionHandler:^(NSData *data, NSURLResponse *response, NSError *error) {
  if (error != nil) {
    NSLog(@"Could not tell: %@", error.localizedDescription); // timeout or no connection - not "not active"
    return;
  }

  NSInteger status = ((NSHTTPURLResponse *)response).statusCode;

  // nil when the body is not JSON - e.g. a 403 from the firewall
  id json = data != nil ? [NSJSONSerialization JSONObjectWithData:data options:0 error:nil] : nil;
  NSDictionary *result = [json isKindOfClass:[NSDictionary class]] ? json : nil;

  BOOL isActive = status == 200 && [result[@"IsSubscriptionActive"] boolValue];

  NSLog(@"HTTP %ld - active: %@ - %@", (long)status, isActive ? @"YES" : @"NO", result[@"ResponseMessage"]);
}];

[task resume];
```

### C

Uses [libcurl](https://curl.se/libcurl/) for HTTP and [cJSON](https://github.com/DaveGamble/cJSON) for JSON.
Build with, for example, `cc check.c -lcurl -lcjson`.

```c
#include <curl/curl.h>
#include <cjson/cJSON.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

struct buffer {
  char *data;
  size_t size;
};

static size_t on_body(char *ptr, size_t size, size_t nmemb, void *userdata) {
  struct buffer *buf = userdata;
  size_t n = size * nmemb;
  char *grown = realloc(buf->data, buf->size + n + 1);
  if (grown == NULL) {
    return 0;
  }
  buf->data = grown;
  memcpy(buf->data + buf->size, ptr, n);
  buf->size += n;
  buf->data[buf->size] = '\0';
  return n;
}

int main(void) {
  cJSON *request = cJSON_CreateObject();
  cJSON_AddStringToObject(request, "TntSubdomain", "yourcompany");
  cJSON_AddStringToObject(request, "CustomerAccountSecretKey", "<customer account secret key>");
  cJSON_AddStringToObject(request, "SubscriptionSecretKey", "<subscription secret key>");
  char *body = cJSON_PrintUnformatted(request);

  curl_global_init(CURL_GLOBAL_DEFAULT);
  CURL *curl = curl_easy_init();

  struct curl_slist *headers = NULL;
  headers = curl_slist_append(headers, "Content-Type: application/json");
  headers = curl_slist_append(headers, "Accept: application/json");

  struct buffer response = {NULL, 0};

  curl_easy_setopt(curl, CURLOPT_URL, "https://prod.us.api.archwayportal.com/Sdk/CheckSubscriptionAsync");
  curl_easy_setopt(curl, CURLOPT_HTTPHEADER, headers);
  curl_easy_setopt(curl, CURLOPT_POSTFIELDS, body);
  curl_easy_setopt(curl, CURLOPT_TIMEOUT, 30L);
  curl_easy_setopt(curl, CURLOPT_WRITEFUNCTION, on_body);
  curl_easy_setopt(curl, CURLOPT_WRITEDATA, &response);

  CURLcode rc = curl_easy_perform(curl);
  if (rc == CURLE_OK) {
    long status = 0;
    curl_easy_getinfo(curl, CURLINFO_RESPONSE_CODE, &status);

    /* NULL when the body is not JSON - e.g. a 403 from the firewall */
    cJSON *result = response.data != NULL ? cJSON_Parse(response.data) : NULL;
    int is_active = status == 200 && cJSON_IsTrue(cJSON_GetObjectItemCaseSensitive(result, "IsSubscriptionActive"));

    printf("HTTP %ld - active: %s\n", status, is_active ? "true" : "false");
    cJSON_Delete(result);
  } else {
    fprintf(stderr, "Could not tell: %s\n", curl_easy_strerror(rc)); /* not "not active" */
  }

  free(response.data);
  curl_slist_free_all(headers);
  curl_easy_cleanup(curl);
  curl_global_cleanup();
  cJSON_free(body);
  cJSON_Delete(request);
  return 0;
}
```

### C++ (17+)

Uses [libcurl](https://curl.se/libcurl/) for HTTP and [nlohmann/json](https://github.com/nlohmann/json) for JSON.
Build with, for example, `c++ -std=c++17 check.cpp -lcurl`.

```cpp
#include <curl/curl.h>
#include <nlohmann/json.hpp>

#include <iostream>
#include <string>

static size_t on_body(char* ptr, size_t size, size_t nmemb, void* userdata) {
  static_cast<std::string*>(userdata)->append(ptr, size * nmemb);
  return size * nmemb;
}

int main() {
  const std::string body = nlohmann::json{
    {"TntSubdomain", "yourcompany"},
    {"CustomerAccountSecretKey", "<customer account secret key>"},
    {"SubscriptionSecretKey", "<subscription secret key>"},
  }.dump();

  curl_global_init(CURL_GLOBAL_DEFAULT);
  CURL* curl = curl_easy_init();

  curl_slist* headers = nullptr;
  headers = curl_slist_append(headers, "Content-Type: application/json");
  headers = curl_slist_append(headers, "Accept: application/json");

  std::string response;

  curl_easy_setopt(curl, CURLOPT_URL, "https://prod.us.api.archwayportal.com/Sdk/CheckSubscriptionAsync");
  curl_easy_setopt(curl, CURLOPT_HTTPHEADER, headers);
  curl_easy_setopt(curl, CURLOPT_POSTFIELDS, body.c_str());
  curl_easy_setopt(curl, CURLOPT_TIMEOUT, 30L);
  curl_easy_setopt(curl, CURLOPT_WRITEFUNCTION, on_body);
  curl_easy_setopt(curl, CURLOPT_WRITEDATA, &response);

  const CURLcode rc = curl_easy_perform(curl);
  if (rc == CURLE_OK) {
    long status = 0;
    curl_easy_getinfo(curl, CURLINFO_RESPONSE_CODE, &status);

    // parse() with allow_exceptions = false: a body that is not JSON (e.g. a 403 from the firewall) is "discarded"
    const auto result = nlohmann::json::parse(response, nullptr, false);
    const bool isActive = status == 200 && result.is_object() && result.value("IsSubscriptionActive", false);

    std::cout << "HTTP " << status << " - active: " << std::boolalpha << isActive << '\n';
  } else {
    std::cerr << "Could not tell: " << curl_easy_strerror(rc) << '\n'; // not "not active"
  }

  curl_slist_free_all(headers);
  curl_easy_cleanup(curl);
  curl_global_cleanup();
  return 0;
}
```

---

## No-code tools - Zapier, Make, n8n

The check is a plain JSON `POST`, so any tool that can make an HTTP request can use it.

### Zapier

1. Add a **Webhooks by Zapier** action and choose **POST**.
2. **URL:** `https://prod.us.api.archwayportal.com/Sdk/CheckSubscriptionAsync`
3. **Payload Type:** `json`
4. **Data:** three rows - `TntSubdomain`, `CustomerAccountSecretKey` and `SubscriptionSecretKey` - mapped from
   earlier steps or typed in.
5. Add a **Filter** or **Paths** step on `Is Subscription Active` to continue only when it is `true`.

A `200` with `IsSubscriptionActive` `false` is a **successful** step - the subscription is simply not active - so
branch on the field, not on whether the step succeeded. A `401` stops the Zap with an error, which is what you
want: the keys are wrong.

### Make

Use the **HTTP → Make a request** module: method `POST`, the URL above, body type **Raw**, content type
**JSON (application/json)**, the JSON body shown in [The request](#the-request), and **Parse response** switched
on. Add a filter on `IsSubscriptionActive`.

### n8n

Use the **HTTP Request** node: method `POST`, the URL above, **Send Body** on with **JSON**, the three fields as
body parameters. Follow it with an **IF** node on `IsSubscriptionActive`.

---

## Keeping the keys safe

The two keys are **secrets** - together they prove a subscription. Treat them like a password.

- **Make the call from a server or a trusted backend**, not from a web page or a browser-based tool, where
  anyone can read the keys from the page.
- **Store them securely** - an encrypted settings store or secrets manager, not a plain file or source code.
- **Don't log them**, and don't put them in URLs or support emails.
- **If a key may have leaked, regenerate it** with **Regenerate Key** in Archway, then give the new key to
  whoever needs it. The old one stops working at once.

---

## Good to know

- **Regenerating a key breaks existing integrations immediately.** Anything still using the old key gets `401`
  until it is updated.
- **How fresh the answer is.** A subscription that is not active is re-checked with the payment provider every
  time. An active one is re-checked at least every 12 hours, so a change made outside Archway - for example
  directly in Stripe - can take up to 12 hours to show here.
- **How often to call.** There is no need to check on every request your application handles. Checking when a
  customer signs in, when your application starts, or a few times a day is plenty.
- **Rate limiting.** Many requests from one address in a short time are temporarily blocked with `403` - see
  [Status codes](#status-codes). Normal use will never come close.
- **Timeouts.** Answers normally arrive within a second or two. Allow up to 30 seconds before treating a call as
  failed, as an occasional check with the payment provider can take longer.
