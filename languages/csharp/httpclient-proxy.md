[Csharp](/languages/csharp)
# HttpClient - proxy

#### Context
Recently I've been struggling with working with `HttpClient` behind corporate proxy (or rather, fighting with not using proxy at all).

Why? Well, in general internal endpoint-endpoint communication should have as little overhead as possible (and there is little to no reason to use proxy here).

But why have I forgotten about all of these journeys with fighting my beloved proxy? Well...
In a good old days of WCF and WebAPI, you could've added `<system.net>` section to web.config to globally set the rules for entire application.

Since I'm using lots of ASP.NET Core now (with `AddHttpClient`), it's a little more tricky. Mostly because I got used to CI/CD pipeline doing above in web.config for me...

#### Problem under the hood

Let's browse .NET Core sourcecode on GitHub, `System.Net.Http` assembly to be specific: https://github.com/dotnet/corefx/tree/master/src/System.Net.Http/src

As `HttpClient` construction path used with `AddHttpClient` (along with all the builders and factories on the way...) uses default `HttpClientHandler` ctor on Windows (see: `HttpClientHandler.Windows.cs`), we're investigating this route:
```csharp
private HttpClientHandler(bool useSocketsHttpHandler) // used by parameterless ctor and as hook for testing
{
    if (useSocketsHttpHandler)
    {
        _socketsHttpHandler = new SocketsHttpHandler();
        _diagnosticsHandler = new DiagnosticsHandler(_socketsHttpHandler);
        ClientCertificateOptions = ClientCertificateOption.Manual;

    }
    else
    {
        _winHttpHandler = new WinHttpHandler();
        _diagnosticsHandler = new DiagnosticsHandler(_winHttpHandler);

        // Adjust defaults to match current .NET Desktop HttpClientHandler (based on HWR stack).
        AllowAutoRedirect = true;
        AutomaticDecompression = HttpHandlerDefaults.DefaultAutomaticDecompression;
        UseProxy = true;
        UseCookies = true;
        CookieContainer = new CookieContainer();
        _winHttpHandler.DefaultProxyCredentials = null;
        _winHttpHandler.ServerCredentials = null;

        // The existing .NET Desktop HttpClientHandler based on the HWR stack uses only WinINet registry
        // settings for the proxy.  This also includes supporting the "Automatic Detect a proxy" using
        // WPAD protocol and PAC file. So, for app-compat, we will do the same for the default proxy setting.
        _winHttpHandler.WindowsProxyUsePolicy = WindowsProxyUsePolicy.UseWinInetProxy;
        _winHttpHandler.Proxy = null;

        // Since the granular WinHttpHandler timeout properties are not exposed via the HttpClientHandler API,
        // we need to set them to infinite and allow the HttpClient.Timeout property to have precedence.
        _winHttpHandler.ReceiveHeadersTimeout = Timeout.InfiniteTimeSpan;
        _winHttpHandler.ReceiveDataTimeout = Timeout.InfiniteTimeSpan;
        _winHttpHandler.SendTimeout = Timeout.InfiniteTimeSpan;
    }
}
```

As we can see, proxy is on by default, along with all the fancy stuff in "Internet options" from Internet Exploder (pun intended).

If we consider that corporate proxies often use credentials, wheras `HttpClientHandler` sets (obviously) none, we can end up with quite undefined behaviour in the end. There might be another 'why?' you'd ask.
Well, mostly because if your proxy/firewall is slow on figuring it out who you are and what address (probably unknown since it's internal) you're trying to reach, ASP.NET Core 2.X will end up returning code 502.3 - Bad Gateway (see: https://github.com/aspnet/AspNetCoreModule/issues/48).

And since it's 502 and not 504 (Gateway Timeout) you start to investigate in very different direction and all the fun begins... :)

#### Solution

In our case we're using some wrappers over `HttpClient` with custom serialization and reuse of client itself via `AddHttpClient<TClient>`.
Since that's the case, we can just patch things up in one place - on registration in `Startup.ConfigureServices`:
```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddHttpClient<IMyClient>()
        .ConfigurePrimaryHttpMessageHandler(() =>
        {
            return new HttpClientHandler()
            {
                UseProxy = false
            };
        });
}
```

And that's it.

#### References

*ASP.NET Core docs* - https://docs.microsoft.com/en-us/aspnet/core/fundamentals/http-requests?view=aspnetcore-3.0
*.NET Core GitHub* - https://github.com/dotnet/corefx