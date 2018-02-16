# Warden Cachet Integration

![Warden](http://spetz.github.io/img/warden_logo.png)

**OPEN SOURCE & CROSS-PLATFORM TOOL FOR SIMPLIFIED MONITORING**

**[getwarden.net](http://getwarden.net)**

|Branch             |Build status                                                  
|-------------------|-----------------------------------------------------
|master             |[![master branch build status](https://api.travis-ci.org/warden-stack/Warden.Integrations.Cachet.svg?branch=master)](https://travis-ci.org/warden-stack/Warden.Integrations.Cachet)
|develop            |[![develop branch build status](https://api.travis-ci.org/warden-stack/Warden.Integrations.Cachet.svg?branch=develop)](https://travis-ci.org/warden-stack/Warden.Integrations.Cachet/branches)

**HttpApiIntegration** can be used for making a POST request to the own/custom API or the Warden Web Panel via the provided helper methods. You may include any data while making the POST request and for example create your own webhooks that will be invoked under specific conditions.

### Installation:

Available as a **[NuGet package](https://www.nuget.org/packages/Warden.Integrations.HttpApi)**. 
```
dotnet add package Warden.Integrations.HttpApi
```

### Configuration:

 - **WithTimeout()** - timeout of the HTTP request.
 - **WithHeaders()** - collection of the HTTP request headers.
 - **WithJsonSerializerSettings()** - custom JSON serializer settings of the Newtonsoft.Json.
 - **FailFast()** - default parameters of the transactional template. 
 - **WithHttpServiceProvider()** - flag determining whether an exception should be thrown on if PostAsync() returns invalid reponse (false by default).

**HttpApiIntegration** can be configured by using the **HttpApiIntegrationConfiguration** class or via the lambda expression passed to a specialized constructor. 

### Initialization:

In order to register and resolve **HttpApiIntegration** make use of the available extension methods while configuring the **Warden**:

```csharp
var wardenConfiguration = WardenConfiguration
    .Create()
    .IntegrateWithHttpApi("http://my-api.com")
    .SetGlobalWatcherHooks((hooks, integrations) =>
    {
        hooks.OnStart(check => GlobalHookOnStart(check))
             .OnFailureAsync(result => integrations.HttpApi().PostAsync(new {failure = true}))
    })
    //Configure watchers, hooks etc..
```

You may also provide the API key (which will set the X-Api-Key request header). On top of that, you may set the organization id (created within the Warden Web Panel) and POST the iteration data directly to the Warden API.
Please note that you may either use the **[Web Panel](https://github.com/spetz/Warden/wiki/Web-Panel)** in **[Azure](https://panel.getwarden.net/)** cloud (https://panel.getwarden.net/api) or your own API e.g. http://localhost/api.

```csharp
var wardenConfiguration = WardenConfiguration
    .Create()
    .IntegrateWithHttpApi("https://panel.getwarden.net/api", 
                          "my-api-key", "my-organization-id")
    .SetHooks((hooks, integrations) =>
    {
        .OnIterationCompletedAsync(iteration => integrations.HttpApi()
                                   .PostIterationToWardenPanelAsync(iteration));
    })
    //Configure watchers, hooks etc..
```

### Custom interfaces:
```csharp
public interface IHttpService
{
   Task<IHttpResponse> ExecuteAsync(string baseUrl, IHttpRequest request, TimeSpan? timeout = null)
}
```

**IHttpService** is responsible for making a POST request. It can be configured via the *WithHttpServiceProvider()* method. By default, it is based on the HttpClient instance.