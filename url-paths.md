# URL Paths

* Tanzu Application Service like many container platforms, adds many layers of load balancing and request routing
* This can often result in URL mappings adding incorrect hostname, protocol (http/https), port, and prefix

## Examples of bad URL mappings

```bash
# IP address instead of hostname
http://192.168.1.1/resource instead of https://myapp.domain/resource

# Non-standard port in URL
http://myapp.domain:8080/resource instead of https://myapp.domain/resource

# prefix in URL
https://myapp.domain/prefix/resource instead of https://myapp.domain/resource

# http instead of https
http://myapp.domain/resource instead of https://myapp.domain/resource
```

## What causes this?

* Your app will run inside a container running on HTTP and using a different port (typically 8080)
* Your app may be called internally by an IP address or internal hostname
* Your app will always be deployed at the root level with no prefix unless customized
* The Request router in TAS (go router) routes traffic to your container and replaces the port, hostname, and protocol

![Path diagram](images/path_diagram.jpg)

## What to look for?

### URL's generated from Request Context
* URL's generated on the backend where hostname's, ports, and schemes are being assumed from the request context
* The request context is not a reliable source for this information as it will be different inside a container

```csharp
string url = $"{Request.Url.Scheme}://{Request.Url.Host}:{Request.Url.Port}/resource";
```
* Instead, use a relative URL like this:
```csharp
string url = "/resource";
```

### Hardcoded prefixes

* Multi-tenant servers (IIS servers with multiple apps running) tend to split applications by adding a prefix
* Example: /app1/* and /app2/*
* Each TAS container only has to run 1 IIS app per instance so there's no need to prefix
* Apps will always be deployed at the root level unless customized 

```csharp
string url = "/app1/resource";
```
* Remove hardcoded prefixes
```csharp
string url = "/resource";
```
* Alternatively, a prefix can be configured and toggled to support multiple environments
```csharp
string url = $"{ConfigurationManager.AppSettings["urlPrefix"]}/resource";
```