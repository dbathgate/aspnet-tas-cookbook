# OpenID Connect Authentication on ASP.NET

## Install Nuget Packages

```powershell
Install-Package Microsoft.Owin.Security.OpenIdConnect
Install-Package Microsoft.Owin.Security.Cookies
Install-Package Microsoft.Owin.Host.SystemWeb
```

## Edit `Web.config`
```xml
<appSettings>
    <add key="oidc:ClientId" value="test-client" />
    <add key="oidc:ClientSecret" value="client-secret" />
    <add key="oidc:Authority" value="http://localhost:8080/realms/test" />
    <add key="oidc:RedirectUri" value="http://localhost:8081/authorization-code/callback" />
</appSettings>
```

### Add `Startup.cs`
```csharp
namespace OidcApp
{
    public class Startup
    {
        private readonly string Authority = ConfigurationManager.AppSettings["oidc:Authority"];
        private readonly string ClientId = ConfigurationManager.AppSettings["oidc:ClientId"];
        private readonly string ClientSecret = ConfigurationManager.AppSettings["oidc:ClientSecret"];
        private readonly string RedirectUri = ConfigurationManager.AppSettings["oidc:RedirectUri"];

        public void Configuration(IAppBuilder app)
        {

            app.SetDefaultSignInAsAuthenticationType(CookieAuthenticationDefaults.AuthenticationType);
            app.UseCookieAuthentication(new CookieAuthenticationOptions());

            app.UseOpenIdConnectAuthentication(new OpenIdConnectAuthenticationOptions
            {
                ClientId = ClientId,
                ClientSecret = ClientSecret,
                Authority = Authority,
                RedirectUri = RedirectUri,
                Scope = OpenIdConnectScope.OpenIdProfile,
                ResponseType = OpenIdConnectResponseType.Code,
                ResponseMode = OpenIdConnectResponseMode.Query,
                RedeemCode = true,
                SaveTokens = true,
                ProtocolValidator = new OpenIdConnectProtocolValidator
                {
                    RequireNonce = false,
                    RequireStateValidation = false
                },
                RequireHttpsMetadata = false, // disable HTTPS for testing, 
            });
        }
    }
}
```

## Executing a Login request

```csharp
HttpContext.Current.GetOwinContext().Authentication.Challenge(
    new Microsoft.Owin.Security.AuthenticationProperties { RedirectUri = "/" }, 
    OpenIdConnectAuthenticationDefaults.AuthenticationType);
```

## Executing a Logout request

```csharp
HttpContext.Current.GetOwinContext().Authentication.SignOut(CookieAuthenticationDefaults.AuthenticationType);
```

## Fetching User Name
```csharp
var name = HttpContext.Current.GetOwinContext().Authentication.User.Claims.Where(c => c.Type == "name").Select(c => c.Value).FirstOrDefault();
```
