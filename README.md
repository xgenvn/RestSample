## RestClient sample

This is a sample of an end to end client experience that we could build with ASP.NET Core using [refit](https://github.com/paulcbetts/refit) as as implementation detail.
The idea here is to make it easy to declare and inject an autogenerated proxy for an HTTP service. That is acheived by registering an
open generic RestClient\<TClient\> that acts like a factory for a TClient. Each TClient must be configured in the ConfigureServices with the url (or http client). Other more advanced settings can also be set there.

### Fundamentals

1. Define an interface for your rest service.
    ````csharp
    public interface IConferencePlannerApi
    {
        [Get("/api/sessions")]
        Task<IEnumerable<JObject>> GetSessionsAsync();
    }
    ````

1. Configure the URL for your service.
    ```csharp
    public void ConfigureServices(IServiceCollection services)
    {
        services.AddRestClient<IConferencePlannerApi>(o =>
        {
            // Configure the URL for your service
            o.Url = "https://conferenceplanner-api.azurewebsites.net/";
        });
    }
    ```
1. Consume the interface via RestClient\<T\>.
    ```csharp
    public class HomeController : Controller
    {
        private readonly IConferencePlannerApi _client;

        public HomeController(RestClient<IConferencePlannerApi> api)
        {
            _client = api.Client;
        }

        [HttpGet("/")]
        public async Task<IActionResult> Get()
        {
            var sessions = await _client.GetSessionsAsync();

            return Ok(sessions);
        }
    }
    ```

### Future Ideas
- Integrate service discovery. Add the ability to name the service that a client represents via an attribute to avoid having to specify a URL in ConfigureServices.

```csharp
[ServiceName("conference-api")]
public interface IConferencePlannerApi
{
    [Get("/api/sessions")]
    Task<IEnumerable<JObject>> GetSessionsAsync();
}
```
- Support the MVC attributes for consistency between client and server.
- Support returning ActionResult\<T\>. It would unify the story between the client and server even more. It could also bring efficiency gains by piping responses directly to the output when making outgoing calls.
- Integrate circuit breakers and other policy into the configuration.
