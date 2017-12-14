---
title: Hosting w platformy ASP.NET Core
author: guardrex
description: "Więcej informacji na temat hosta sieci web w ASP.NET Core, która jest odpowiedzialna za uruchamianie i okresem istnienia zarządzania aplikacjami."
keywords: Hosta, IWebHost, WebHostBuilder, IHostingEnvironment, IApplicationLifetime sieci web platformy ASP.NET Core
ms.author: riande
manager: wpickett
ms.date: 09/21/2017
ms.topic: article
ms.technology: aspnet
ms.prod: asp.net-core
uid: fundamentals/hosting
ms.openlocfilehash: 7deccf135ddd21729206ebed58ddc8aca52c1deb
ms.sourcegitcommit: 8f42ab93402c1b8044815e1e48d0bb84c81f8b59
ms.translationtype: MT
ms.contentlocale: pl-PL
ms.lasthandoff: 11/29/2017
---
# <a name="hosting-in-aspnet-core"></a>Hosting w platformy ASP.NET Core

Przez [Luke Latham](https://github.com/guardrex)

Aplikacje platformy ASP.NET Core skonfigurować i uruchomić *hosta*, który jest odpowiedzialny za zarządzanie uruchamiania i okresem istnienia aplikacji. Co najmniej hosta konfiguruje serwer i potoku przetwarzania żądań.

## <a name="setting-up-a-host"></a>Konfigurowanie hosta

# <a name="aspnet-core-2xtabaspnetcore2x"></a>[Program ASP.NET Core 2.x](#tab/aspnetcore2x)

Utwórz hosta za pomocą wystąpienia [WebHostBuilder](/dotnet/api/microsoft.aspnetcore.hosting.webhostbuilder). Jest to najczęściej wykonywane w punkcie wejścia aplikacji, `Main` metody. W szablonach projektu `Main` znajduje się w *Program.cs*. Typowe *Program.cs* wywołania [CreateDefaultBuilder](/dotnet/api/microsoft.aspnetcore.webhost.createdefaultbuilder) do rozpoczęcia konfigurowania hosta:

[!code-csharp[Main](../common/samples/WebApplication1DotNetCore2.0App/Program.cs?name=snippet_Main)]

`CreateDefaultBuilder`wykonuje następujące zadania:

* Konfiguruje [Kestrel](servers/kestrel.md) jako serwera sieci web. Aby Kestrel domyślnych opcji, zobacz [Kestrel opcje sekcji Kestrel implementacja serwera sieci web platformy ASP.NET Core](xref:fundamentals/servers/kestrel#kestrel-options).
* Ustawia zawartości głównego [Directory.GetCurrentDirectory](/dotnet/api/system.io.directory.getcurrentdirectory).
* Konfiguracja opcjonalna obciążeń z:
  * *appSettings.JSON*.
  * *appSettings. {Środowiska} JSON*.
  * [Klucze tajne użytkownika](xref:security/app-secrets) po uruchomieniu aplikacji `Development` środowiska.
  * Zmienne środowiskowe.
  * Argumenty wiersza polecenia.
* Konfiguruje [rejestrowanie](xref:fundamentals/logging/index) dla danych wyjściowych konsoli i debugowania z [filtrowania dziennika](xref:fundamentals/logging/index#log-filtering) reguły określone w sekcji konfiguracji rejestrowania *appsettings.json* lub *appsettings. {Środowiska} JSON* pliku.
* Podczas uruchamiania za usług IIS, umożliwia [integracji usług IIS](xref:publishing/iis) przez skonfigurowanie ścieżki podstawowej i port serwera powinien nasłuchiwać przy użyciu [platformy ASP.NET Core modułu](xref:fundamentals/servers/aspnet-core-module). Moduł tworzy serwer proxy wstecznego między usługami IIS a Kestrel. Konfiguruje również aplikacji na [przechwytywania błędy uruchamiania](#capture-startup-errors). Dla opcji domyślnych usług IIS, zobacz [IIS opcje sekcji hosta platformy ASP.NET Core w systemie Windows z programem IIS](xref:publishing/iis#iis-options).

*Zawartości głównego* Określa, gdzie hosta wyszukuje pliki zawartości, takich jak pliki widoku MVC. Domyślny element zawartości jest [Directory.GetCurrentDirectory](/dotnet/api/system.io.directory.getcurrentdirectory). Powoduje to przy użyciu folderu głównego projektu sieci web jako główny zawartości po uruchomieniu aplikacji z folderu głównego (na przykład wywołanie elementu [dotnet Uruchom](/dotnet/core/tools/dotnet-run) z folderu projektu). Jest to wartość domyślna używana w [programu Visual Studio](https://www.visualstudio.com/) i [dotnet nowe szablony](/dotnet/core/tools/dotnet-new).

Zobacz [konfiguracji w programie ASP.NET Core](xref:fundamentals/configuration/index) uzyskać więcej informacji o konfiguracji aplikacji.

> [!NOTE]
> Alternatywą wobec przy użyciu statycznych `CreateDefaultBuilder` metody tworzenia hosta z [WebHostBuilder](/dotnet/api/microsoft.aspnetcore.hosting.webhostbuilder) jest to obsługiwane podejście z platformy ASP.NET Core 2.x. Zobacz kartę 1.x platformy ASP.NET Core, aby uzyskać więcej informacji.

# <a name="aspnet-core-1xtabaspnetcore1x"></a>[Program ASP.NET Core 1.x](#tab/aspnetcore1x)

Utwórz hosta za pomocą wystąpienia [WebHostBuilder](/dotnet/api/microsoft.aspnetcore.hosting.webhostbuilder). Jest to najczęściej wykonywane w punkcie wejścia aplikacji, `Main` metody. W szablonach projektu `Main` znajduje się w *Program.cs*. Następujące *Program.cs* przedstawiono sposób użycia `WebHostBuilder` tworzenie hosta:

[!code-csharp[Main](../common/samples/WebApplication1/Program.cs)]

`WebHostBuilder`wymaga [serwera, który implementuje IServer](servers/index.md). Wbudowane serwery są [Kestrel](servers/kestrel.md) i [HTTP.sys](servers/httpsys.md) (przed wydaniem programu ASP.NET 2.0 Core została wywołana HTTP.sys [WebListener](xref:fundamentals/servers/weblistener)). W tym przykładzie [— metoda rozszerzenia UseKestrel](/dotnet/api/microsoft.aspnetcore.hosting.webhostbuilderkestrelextensions.usekestrel?view=aspnetcore-1.1) Określa serwer Kestrel.

*Zawartości głównego* Określa, gdzie hosta wyszukuje pliki zawartości, takich jak pliki widoku MVC. Domyślny element zawartości dostarczony do `UseContentRoot` jest [Directory.GetCurrentDirectory](/dotnet/api/system.io.directory.getcurrentdirectory?view=netcore-1.1). Powoduje to przy użyciu folderu głównego projektu sieci web jako główny zawartości po uruchomieniu aplikacji z folderu głównego (na przykład wywołanie elementu [dotnet Uruchom](/dotnet/core/tools/dotnet-run) z folderu projektu). Jest to wartość domyślna używana w [programu Visual Studio](https://www.visualstudio.com/) i [dotnet nowe szablony](/dotnet/core/tools/dotnet-new).

Do używania serwera IIS jako zwrotny serwer proxy, należy wywołać [UseIISIntegration](/aspnet/core/api/microsoft.aspnetcore.hosting.webhostbuilderiisextensions) w ramach tworzenia hosta. `UseIISIntegration`nie Konfiguruj *serwera*, takiej jak [UseKestrel](/dotnet/api/microsoft.aspnetcore.hosting.webhostbuilderkestrelextensions.usekestrel?view=aspnetcore-1.1) jest. `UseIISIntegration`Konfiguruje serwer powinien nasłuchiwać przy użyciu portu i ścieżki bazowej [moduł platformy ASP.NET Core](xref:fundamentals/servers/aspnet-core-module) utworzyć proxy wstecznego między Kestrel i IIS. Aby używać usług IIS z platformy ASP.NET Core, należy określić zarówno `UseKestrel` i `UseIISIntegration`. `UseIISIntegration`aktywuje tylko podczas uruchamiania usług IIS lub usług IIS Express. Aby uzyskać więcej informacji, zobacz [wprowadzenie do platformy ASP.NET Core modułu](xref:fundamentals/servers/aspnet-core-module) i [odwołania konfiguracji platformy ASP.NET Core modułu](xref:hosting/aspnet-core-module).

Minimalny wdrożenia, który konfiguruje hosta (a aplikacji platformy ASP.NET Core) obejmuje określenie serwera i konfiguracji Potok żądań aplikacji:

```csharp
var host = new WebHostBuilder()
    .UseKestrel()
    .Configure(app =>
    {
        app.Run(context => context.Response.WriteAsync("Hello World!"));
    })
    .Build();

host.Run();
```

---

Podczas konfigurowania hosta, możesz podać [Konfiguruj](/dotnet/api/microsoft.aspnetcore.hosting.webhostbuilderextensions.configure?view=aspnetcore-1.1) i [ConfigureServices](/dotnet/api/microsoft.aspnetcore.hosting.webhostbuilder.configureservices?view=aspnetcore-1.1) metody. Jeśli określisz `Startup` klasy, należy zdefiniować `Configure` metody. Aby uzyskać więcej informacji, zobacz [uruchamiania aplikacji w ASP.NET Core](startup.md). Wiele wywołań `ConfigureServices` dołączyć do siebie. Wiele wywołań `Configure` lub `UseStartup` na `WebHostBuilder` zastąpić poprzednie ustawienia.

## <a name="host-configuration-values"></a>Wartości konfiguracji hosta

[WebHostBuilder](/dotnet/api/microsoft.aspnetcore.hosting.webhostbuilder) udostępnia metody do ustawiania większość wartości konfiguracji dostępne dla hosta, który można również ustawić bezpośrednio z [UseSetting](/dotnet/api/microsoft.aspnetcore.hosting.webhostbuilder.usesetting) i skojarzony klucz. Podczas ustawiania wartości z `UseSetting`, ma wartość ciągu (w cudzysłowy) niezależnie od typu.

### <a name="capture-startup-errors"></a>Przechwyć błędy uruchamiania

To ustawienie określa przechwytywania błędy uruchamiania.

**Klucz**: captureStartupErrors  
**Typ**: *bool* (`true` lub `1`)  
**Domyślne**: Domyślnie `false` chyba, że aplikacja jest uruchamiana z Kestrel za usług IIS, gdzie wartość domyślna to `true`.  
**Ustawić za pomocą**:`CaptureStartupErrors`

Gdy `false`, błędy podczas wynik uruchomienia na hoście został zakończony. Gdy `true`, host przechwytuje wyjątków podczas uruchamiania i podejmie próbę uruchomienia serwera.

# <a name="aspnet-core-2xtabaspnetcore2x"></a>[Program ASP.NET Core 2.x](#tab/aspnetcore2x)

```csharp
WebHost.CreateDefaultBuilder(args)
    .CaptureStartupErrors(true)
    ...
```

# <a name="aspnet-core-1xtabaspnetcore1x"></a>[Program ASP.NET Core 1.x](#tab/aspnetcore1x)

```csharp
var host = new WebHostBuilder()
    .CaptureStartupErrors(true)
    ...
```

---

### <a name="content-root"></a>Główny zawartości

To ustawienie określa, gdzie platformy ASP.NET Core rozpocznie się wyszukiwanie plików zawartości, np. widoków MVC. 

**Klucz**: contentRoot  
**Typ**: *ciągu*  
**Domyślna**: domyślne do folderu, w którym znajduje się zestaw aplikacji.  
**Ustawić za pomocą**:`UseContentRoot`

Główny zawartości jest również używany jako podstawowa ścieżka dla [ustawienia sieci Web głównego](#web-root). Jeśli ścieżka nie istnieje, host nie powiedzie się.

# <a name="aspnet-core-2xtabaspnetcore2x"></a>[Program ASP.NET Core 2.x](#tab/aspnetcore2x)

```csharp
WebHost.CreateDefaultBuilder(args)
    .UseContentRoot("c:\\mywebsite")
    ...
```

# <a name="aspnet-core-1xtabaspnetcore1x"></a>[Program ASP.NET Core 1.x](#tab/aspnetcore1x)

```csharp
var host = new WebHostBuilder()
    .UseContentRoot("c:\\mywebsite")
    ...
```

---

### <a name="detailed-errors"></a>Błędy szczegółowe

Określa, czy szczegółowe błędy, które mają być przechwytywane.

**Klucz**: detailedErrors  
**Typ**: *bool* (`true` lub `1`)  
**Domyślna**: false  
**Ustawić za pomocą**:`UseSetting`

Po włączeniu (lub gdy <a href="#environment">środowiska</a> ma ustawioną wartość `Development`), aplikacji znajdują się szczegółowe wyjątki.

# <a name="aspnet-core-2xtabaspnetcore2x"></a>[Program ASP.NET Core 2.x](#tab/aspnetcore2x)

```csharp
WebHost.CreateDefaultBuilder(args)
    .UseSetting(WebHostDefaults.DetailedErrorsKey, "true")
    ...
```

# <a name="aspnet-core-1xtabaspnetcore1x"></a>[Program ASP.NET Core 1.x](#tab/aspnetcore1x)

```csharp
var host = new WebHostBuilder()
    .UseSetting(WebHostDefaults.DetailedErrorsKey, "true")
    ...
```

---

### <a name="environment"></a>Środowisko

Ustawia środowisko aplikacji.

**Klucz**: środowiska  
**Typ**: *ciągu*  
**Domyślna**: produkcji  
**Ustawić za pomocą**:`UseEnvironment`

Można ustawić *środowiska* dowolną wartość. Wartości zdefiniowane w ramach obejmują `Development`, `Staging`, i `Production`. Wartości nie jest uwzględniana wielkość liter. Domyślnie *środowiska* są odczytywane z `ASPNETCORE_ENVIRONMENT` zmiennej środowiskowej. Korzystając z [programu Visual Studio](https://www.visualstudio.com/), zmienne środowiskowe może być ustawiona w *launchSettings.json* pliku. Aby uzyskać więcej informacji, zobacz [Praca w środowiskach wielu](xref:fundamentals/environments).

# <a name="aspnet-core-2xtabaspnetcore2x"></a>[Program ASP.NET Core 2.x](#tab/aspnetcore2x)

```csharp
WebHost.CreateDefaultBuilder(args)
    .UseEnvironment("Development")
    ...
```

# <a name="aspnet-core-1xtabaspnetcore1x"></a>[Program ASP.NET Core 1.x](#tab/aspnetcore1x)

```csharp
var host = new WebHostBuilder()
    .UseEnvironment("Development")
    ...
```

---

### <a name="hosting-startup-assemblies"></a>Zestawy uruchomienia hostingu

Ustawia hostingu zestawy uruchomienia aplikacji.

**Klucz**: hostingStartupAssemblies  
**Typ**: *ciągu*  
**Domyślna**: pusty ciąg  
**Ustawić za pomocą**:`UseSetting`

Ciąg rozdzielany średnikami obsługi zestawów uruchamiania załadować podczas uruchamiania. Ta funkcja jest nowa w programie ASP.NET 2.0 Core.

Mimo że konfiguracja domyślnie przyjmowana jest wartość pustego ciągu, hostingu zestawy uruchamiania zawsze należy uwzględniać zestawu aplikacji. Podając hostingu zestawy uruchamiania, są one dodane do zestawu aplikacji ładowania, gdy aplikacja tworzy jego wspólne usługi podczas uruchamiania.

# <a name="aspnet-core-2xtabaspnetcore2x"></a>[Program ASP.NET Core 2.x](#tab/aspnetcore2x)

```csharp
WebHost.CreateDefaultBuilder(args)
    .UseSetting(WebHostDefaults.HostingStartupAssembliesKey, "assembly1;assembly2")
    ...
```

# <a name="aspnet-core-1xtabaspnetcore1x"></a>[Program ASP.NET Core 1.x](#tab/aspnetcore1x)

Ta funkcja jest niedostępna w ASP.NET Core 1.x.

---

### <a name="prefer-hosting-urls"></a>Preferowane jest Hosting adresy URL

Wskazuje, czy host powinien nasłuchiwać adresy URL skonfigurowano `WebHostBuilder` zamiast ustawień skonfigurowanych z `IServer` implementacji.

**Klucz**: preferHostingUrls  
**Typ**: *bool* (`true` lub `1`)  
**Domyślna**: true  
**Ustawić za pomocą**:`PreferHostingUrls`

Ta funkcja jest nowa w programie ASP.NET 2.0 Core.

# <a name="aspnet-core-2xtabaspnetcore2x"></a>[Program ASP.NET Core 2.x](#tab/aspnetcore2x)

```csharp
WebHost.CreateDefaultBuilder(args)
    .PreferHostingUrls(false)
    ...
```

# <a name="aspnet-core-1xtabaspnetcore1x"></a>[Program ASP.NET Core 1.x](#tab/aspnetcore1x)

Ta funkcja jest niedostępna w ASP.NET Core 1.x.

---

### <a name="prevent-hosting-startup"></a>Zapobiegaj Hosting uruchamiania

Uniemożliwia automatyczne ładowanie hosting zestawy uruchomienia, w tym zestawie aplikacji.

**Klucz**: preventHostingStartup  
**Typ**: *bool* (`true` lub `1`)  
**Domyślna**: false  
**Ustawić za pomocą**:`UseSetting`

Ta funkcja jest nowa w programie ASP.NET 2.0 Core.

# <a name="aspnet-core-2xtabaspnetcore2x"></a>[Program ASP.NET Core 2.x](#tab/aspnetcore2x)

```csharp
WebHost.CreateDefaultBuilder(args)
    .UseSetting(WebHostDefaults.PreventHostingStartupKey, "true")
    ...
```

# <a name="aspnet-core-1xtabaspnetcore1x"></a>[Program ASP.NET Core 1.x](#tab/aspnetcore1x)

Ta funkcja jest niedostępna w ASP.NET Core 1.x.

---

### <a name="server-urls"></a>Adresy URL serwerów

Wskazuje adresy IP lub adresy hostów, porty i protokoły, które serwer powinien nasłuchiwać żądań.

**Klucz**: adresy URL  
**Typ**: *ciągu*  
**Domyślna**: http://localhost: 5000  
**Ustawić za pomocą**:`UseUrls`

Ustaw rozdzielonych średnikami (;) lista adresów URL prefiksy powinny odpowiadać serwera. Na przykład `http://localhost:123`. Użyj "\*" Aby wskazać, że serwer powinien nasłuchiwać żądań adresy IP lub nazwa hosta przy użyciu określonego portu i protokołu (na przykład `http://*:5000`). Protokół (`http://` lub `https://`) musi być dołączony do każdego adresu URL. Obsługiwane formaty różnią się między serwerami.

# <a name="aspnet-core-2xtabaspnetcore2x"></a>[Program ASP.NET Core 2.x](#tab/aspnetcore2x)

```csharp
WebHost.CreateDefaultBuilder(args)
    .UseUrls("http://*:5000;http://localhost:5001;https://hostname:5002")
    ...
```

Kestrel ma własny interfejs API konfiguracji punktu końcowego. Aby uzyskać więcej informacji, zobacz [Kestrel implementacja serwera sieci web platformy ASP.NET Core](xref:fundamentals/servers/kestrel?tabs=aspnetcore2x#endpoint-configuration).

# <a name="aspnet-core-1xtabaspnetcore1x"></a>[Program ASP.NET Core 1.x](#tab/aspnetcore1x)

```csharp
var host = new WebHostBuilder()
    .UseUrls("http://*:5000;http://localhost:5001;https://hostname:5002")
    ...
```

---

### <a name="shutdown-timeout"></a>Limit czasu zamykania

Określa czas oczekiwania na zamknięcie hosta sieci web.

**Klucz**: shutdownTimeoutSeconds  
**Typ**: *int*  
**Domyślna**: 5  
**Ustawić za pomocą**:`UseShutdownTimeout`

Mimo że akceptuje klucz *int* z `UseSetting` (na przykład `.UseSetting(WebHostDefaults.ShutdownTimeoutKey, "10")`), `UseShutdownTimeout` przyjmuje — metoda rozszerzenia `TimeSpan`. Ta funkcja jest nowa w programie ASP.NET 2.0 Core.

# <a name="aspnet-core-2xtabaspnetcore2x"></a>[Program ASP.NET Core 2.x](#tab/aspnetcore2x)

```csharp
WebHost.CreateDefaultBuilder(args)
    .UseShutdownTimeout(TimeSpan.FromSeconds(10))
    ...
```

# <a name="aspnet-core-1xtabaspnetcore1x"></a>[Program ASP.NET Core 1.x](#tab/aspnetcore1x)

Ta funkcja jest niedostępna w ASP.NET Core 1.x.

---

### <a name="startup-assembly"></a>Zestaw startowy

Określa zestaw do wyszukania `Startup` klasy.

**Klucz**: startupAssembly  
**Typ**: *ciągu*  
**Domyślna**: zestaw aplikacji  
**Ustawić za pomocą**:`UseStartup`

Nazwę można odwoływać się do zestawu (`string`) lub typu (`TStartup`). Jeśli wiele `UseStartup` metody są nazywane, pierwszeństwo ma ostatnią.

# <a name="aspnet-core-2xtabaspnetcore2x"></a>[Program ASP.NET Core 2.x](#tab/aspnetcore2x)

```csharp
WebHost.CreateDefaultBuilder(args)
    .UseStartup("StartupAssemblyName")
    ...
```

```csharp
WebHost.CreateDefaultBuilder(args)
    .UseStartup<TStartup>()
    ...
```

# <a name="aspnet-core-1xtabaspnetcore1x"></a>[Program ASP.NET Core 1.x](#tab/aspnetcore1x)

```csharp
var host = new WebHostBuilder()
    .UseStartup("StartupAssemblyName")
    ...
```

```csharp
var host = new WebHostBuilder()
    .UseStartup<TStartup>()
    ...
```

---

### <a name="web-root"></a>Główny sieci Web

Ustawia ścieżkę względną do statycznego zasobów aplikacji.

**Klucz**: webroot  
**Typ**: *ciągu*  
**Domyślne**: Jeśli nie jest określony, wartością domyślną jest "(Content Root)/wwwroot", jeśli ścieżka istnieje. Jeśli ścieżka nie istnieje, jest używany dostawca pliku zerowa.  
**Ustawić za pomocą**:`UseWebRoot`

# <a name="aspnet-core-2xtabaspnetcore2x"></a>[Program ASP.NET Core 2.x](#tab/aspnetcore2x)

```csharp
WebHost.CreateDefaultBuilder(args)
    .UseWebRoot("public")
    ...
```

# <a name="aspnet-core-1xtabaspnetcore1x"></a>[Program ASP.NET Core 1.x](#tab/aspnetcore1x)

```csharp
var host = new WebHostBuilder()
    .UseWebRoot("public")
    ...
```

---

## <a name="overriding-configuration"></a>Zastępowanie konfiguracji

Użyj [konfiguracji](xref:fundamentals/configuration/index) Aby skonfigurować hosta. W poniższym przykładzie konfiguracja hosta jest opcjonalnie określić w *hosting.json* pliku. Wszelkie konfiguracja została załadowana z *hosting.json* plik może być zastąpiona przez argumenty wiersza polecenia. Zbudowany konfiguracji (w `config`) służy do konfigurowania hosta z `UseConfiguration`.

# <a name="aspnet-core-2xtabaspnetcore2x"></a>[Program ASP.NET Core 2.x](#tab/aspnetcore2x)

*hosting.JSON*:

```json
{
    urls: "http://*:5005"
}
```

Zastępowanie konfiguracji dostarczonych przez `UseUrls` z *hosting.json* config argument pierwszą, wiersza polecenia config drugi:

```csharp
public class Program
{
    public static void Main(string[] args)
    {
        BuildWebHost(args).Run();
    }

    public static IWebHost BuildWebHost(string[] args)
    {
        var config = new ConfigurationBuilder()
            .SetBasePath(Directory.GetCurrentDirectory())
            .AddJsonFile("hosting.json", optional: true)
            .AddCommandLine(args)
            .Build();

        return WebHost.CreateDefaultBuilder(args)
            .UseUrls("http://*:5000")
            .UseConfiguration(config)
            .Configure(app =>
            {
                app.Run(context => 
                    context.Response.WriteAsync("Hello, World!"));
            })
            .Build();
    }
}
```

# <a name="aspnet-core-1xtabaspnetcore1x"></a>[Program ASP.NET Core 1.x](#tab/aspnetcore1x)

*hosting.JSON*:

```json
{
    urls: "http://*:5005"
}
```

Zastępowanie konfiguracji dostarczonych przez `UseUrls` z *hosting.json* config argument pierwszą, wiersza polecenia config drugi:

```csharp
public class Program
{
    public static void Main(string[] args)
    {
        var config = new ConfigurationBuilder()
            .SetBasePath(Directory.GetCurrentDirectory())
            .AddJsonFile("hosting.json", optional: true)
            .AddCommandLine(args)
            .Build();

        var host = new WebHostBuilder()
            .UseUrls("http://*:5000")
            .UseConfiguration(config)
            .UseKestrel()
            .Configure(app =>
            {
                app.Run(context => 
                    context.Response.WriteAsync("Hello, World!"));
            })
            .Build();

        host.Run();
    }
}
```

---

> [!NOTE]
> `UseConfiguration` — Metoda rozszerzenia nie jest obecnie stanie podczas analizowania sekcji konfiguracji zwrócony przez `GetSection` (na przykład `.UseConfiguration(Configuration.GetSection("section"))`. `GetSection` — Metoda filtruje klucze konfiguracji do sekcji żądanie, jednak nie pozostawia nazwy sekcji kluczy (na przykład `section:urls`, `section:environment`). `UseConfiguration` Metoda oczekuje klucze do dopasowania `WebHostBuilder` kluczy (na przykład `urls`, `environment`). Występowanie nazwy sekcji kluczy uniemożliwia wartości w sekcji Konfigurowanie hosta. Ten problem zostanie rozwiązany w kolejnej wersji. Aby uzyskać więcej informacji i rozwiązania problemu, zobacz [przekazywanie Sekcja konfiguracyjna do WebHostBuilder.UseConfiguration używa kluczy pełna](https://github.com/aspnet/Hosting/issues/839).

Aby określić hosta, uruchom na określony adres URL, możesz przesłać żądaną wartość z wiersza polecenia podczas wykonywania `dotnet run`. Argument wiersza polecenia zastępuje `urls` wartość z *hosting.json* pliku, a serwer nasłuchuje na porcie 8080:

```console
dotnet run --urls "http://*:8080"
```

## <a name="ordering-importance"></a>Znaczenie kolejności

Niektóre `WebHostBuilder` ustawienia najpierw są odczytywane z zmiennych środowiskowych, jeśli ustawiona. Te zmienne środowiskowe, użyj formatu `ASPNETCORE_{configurationKey}`. Aby ustawić adresów URL, które serwer nasłuchuje na domyślne, należy ustawić `ASPNETCORE_URLS`.

Można zastąpić jedną z tych wartości zmiennych środowiskowych, określając konfiguracji (przy użyciu `UseConfiguration`) lub przez jawne ustawienie wartości (przy użyciu `UseSetting` lub jednej z metod rozszerzenia jawne, takich jak `UseUrls`). Host używa jednego z tych opcji ustawia wartość ostatnio. Jeśli chcesz programowo ustawiony domyślny adres URL na jedną wartość, ale zezwolenie na należy zastąpić konfiguracji, można użyć wiersza polecenia konfigurację po ustawieniu adres URL. Zobacz [konfiguracji zastąpiona](#overriding-configuration).

## <a name="starting-the-host"></a>Uruchamianie hosta

# <a name="aspnet-core-2xtabaspnetcore2x"></a>[Program ASP.NET Core 2.x](#tab/aspnetcore2x)

**Uruchom**

`Run` Metoda uruchamiania aplikacji sieci web i blokuje wątek wywołujący do czasu zamknięcia hosta:

```csharp
host.Run();
```

**Start**

Można uruchomić hosta w sposób nieblokujące przez wywołanie jego `Start` metody:

```csharp
using (host)
{
    host.Start();
    Console.ReadLine();
}
```

W przypadku przekazania listę adresów URL do `Start` metody go nasłuchuje na określone adresy URL:

```csharp
var urls = new List<string>()
{
    "http://*:5000",
    "http://localhost:5001"
};

var host = new WebHostBuilder()
    .UseKestrel()
    .UseStartup<Startup>()
    .Start(urls.ToArray());

using (host)
{
    Console.ReadLine();
}
```

Można zainicjować i rozpocząć nowego hosta za pomocą wstępnie skonfigurowane ustawienia domyślne `CreateDefaultBuilder` przy użyciu metody statycznej wygody. Te metody uruchomienia serwera bez dane wyjściowe konsoli i z [WaitForShutdown](/dotnet/api/microsoft.aspnetcore.hosting.webhostextensions.waitforshutdown) poczekaj, aż podział (Ctrl-C/sigint — lub sigterm —):

**Start (RequestDelegate aplikacji)**

Rozpoczynać `RequestDelegate`:

```csharp
using (var host = WebHost.Start(app => app.Response.WriteAsync("Hello, World!")))
{
    Console.WriteLine("Use Ctrl-C to shutdown the host...");
    host.WaitForShutdown();
}
```

Tworzenie żądania w przeglądarce, aby `http://localhost:5000` odpowiedź "Hello World!" `WaitForShutdown`bloki aż do wystawienia podział (Ctrl-C/sigint — lub sigterm —). Aplikacja jest wyświetlana `Console.WriteLine` komunikat i czeka na keypress zakończyć.

**Start (ciąg adresu url, RequestDelegate aplikacji)**

Uruchom przy użyciu adresu URL i `RequestDelegate`:

```csharp
using (var host = WebHost.Start("http://localhost:8080", app => app.Response.WriteAsync("Hello, World!")))
{
    Console.WriteLine("Use Ctrl-C to shutdown the host...");
    host.WaitForShutdown();
}
```

Tworzy wynik, w postaci **Start (aplikacja RequestDelegate)**, z wyjątkiem aplikacji odpowiada na `http://localhost:8080`.

**Start (akcji<IRouteBuilder> routeBuilder)**

Użyj wystąpienia `IRouteBuilder` ([Microsoft.AspNetCore.Routing](https://www.nuget.org/packages/Microsoft.AspNetCore.Routing/)) do używania oprogramowania pośredniczącego routingu:

```csharp
using (var host = WebHost.Start(router => router
    .MapGet("hello/{name}", (req, res, data) => 
        res.WriteAsync($"Hello, {data.Values["name"]}!"))
    .MapGet("buenosdias/{name}", (req, res, data) => 
        res.WriteAsync($"Buenos dias, {data.Values["name"]}!"))
    .MapGet("throw/{message?}", (req, res, data) => 
        throw new Exception((string)data.Values["message"] ?? "Uh oh!"))
    .MapGet("{greeting}/{name}", (req, res, data) => 
        res.WriteAsync($"{data.Values["greeting"]}, {data.Values["name"]}!"))
    .MapGet("", (req, res, data) => res.WriteAsync("Hello, World!"))))
{
    Console.WriteLine("Use Ctrl-C to shutdown the host...");
    host.WaitForShutdown();
}
```

Użyj następujących żądań przeglądarki z przykładem:

| Żądanie                                    | Odpowiedź                                 |
| ------------------------------------------ | ---------------------------------------- |
| `http://localhost:5000/hello/Martin`       | Witaj, pole!                           |
| `http://localhost:5000/buenosdias/Catrina` | Buenos dias, Catrina!                    |
| `http://localhost:5000/throw/ooops!`       | Zgłasza wyjątek z ciągiem "ooops!" |
| `http://localhost:5000/throw`              | Zgłasza wyjątek z ciągiem "Uh Niestety!" |
| `http://localhost:5000/Sante/Kevin`        | Sante, Jan!                            |
| `http://localhost:5000`                    | Cześć ludzie!                             |

`WaitForShutdown`bloki aż do wystawienia podział (Ctrl-C/sigint — lub sigterm —). Aplikacja jest wyświetlana `Console.WriteLine` komunikat i czeka na keypress zakończyć.

**Start (ciągu adresu url, Akcja<IRouteBuilder> routeBuilder)**

Użyj adresu URL i wystąpienie `IRouteBuilder`:

```csharp
using (var host = WebHost.Start("http://localhost:8080", router => router
    .MapGet("hello/{name}", (req, res, data) => 
        res.WriteAsync($"Hello, {data.Values["name"]}!"))
    .MapGet("buenosdias/{name}", (req, res, data) => 
        res.WriteAsync($"Buenos dias, {data.Values["name"]}!"))
    .MapGet("throw/{message?}", (req, res, data) => 
        throw new Exception((string)data.Values["message"] ?? "Uh oh!"))
    .MapGet("{greeting}/{name}", (req, res, data) => 
        res.WriteAsync($"{data.Values["greeting"]}, {data.Values["name"]}!"))
    .MapGet("", (req, res, data) => res.WriteAsync("Hello, World!"))))
{
    Console.WriteLine("Use Ctrl-C to shutdown the host...");
    host.WaitForShutdown();
}
```

Tworzy wynik, w postaci **Start (akcji<IRouteBuilder> routeBuilder)**, z wyjątkiem aplikacji odpowiada na `http://localhost:8080`.

**StartWith (akcji<IApplicationBuilder> aplikacji)**

Podaj delegat konfigurujący `IApplicationBuilder`:

```csharp
using (var host = WebHost.StartWith(app => 
    app.Use(next => 
    {
        return async context => 
        {
            await context.Response.WriteAsync("Hello World!");
        };
    })))
{
    Console.WriteLine("Use Ctrl-C to shutdown the host...");
    host.WaitForShutdown();
}
```

Tworzenie żądania w przeglądarce, aby `http://localhost:5000` odpowiedź "Hello World!" `WaitForShutdown`bloki aż do wystawienia podział (Ctrl-C/sigint — lub sigterm —). Aplikacja jest wyświetlana `Console.WriteLine` komunikat i czeka na keypress zakończyć.

**StartWith (ciągu adresu url, Akcja<IApplicationBuilder> aplikacji)**

Podaj adres URL i delegat konfigurujący `IApplicationBuilder`:

```csharp
using (var host = WebHost.StartWith("http://localhost:8080", app => 
    app.Use(next => 
    {
        return async context => 
        {
            await context.Response.WriteAsync("Hello World!");
        };
    })))
{
    Console.WriteLine("Use Ctrl-C to shutdown the host...");
    host.WaitForShutdown();
}
```

Tworzy wynik, w postaci **StartWith (akcji<IApplicationBuilder> aplikacji)**, z wyjątkiem aplikacji odpowiada na `http://localhost:8080`.

# <a name="aspnet-core-1xtabaspnetcore1x"></a>[Program ASP.NET Core 1.x](#tab/aspnetcore1x)

**Uruchom**

`Run` Metoda uruchamiania aplikacji sieci web i blokuje wątek wywołujący do czasu zamknięcia hosta:

```csharp
host.Run();
```

**Start**

Można uruchomić hosta w sposób nieblokujące przez wywołanie jego `Start` metody:

```csharp
using (host)
{
    host.Start();
    Console.ReadLine();
}
```

W przypadku przekazania listę adresów URL do `Start` metody go nasłuchuje na określone adresy URL:


```csharp
var urls = new List<string>()
{
    "http://*:5000",
    "http://localhost:5001"
};

var host = new WebHostBuilder()
    .UseKestrel()
    .UseStartup<Startup>()
    .Start(urls.ToArray());

using (host)
{
    Console.ReadLine();
}
```

---

## <a name="ihostingenvironment-interface"></a>Interfejs IHostingEnvironment

[Interfejsu IHostingEnvironment](/aspnet/core/api/microsoft.aspnetcore.hosting.ihostingenvironment) zawiera informacje o aplikacji sieci web środowiska macierzystego. Można użyć [iniekcji konstruktora](xref:fundamentals/dependency-injection) uzyskanie `IHostingEnvironment` aby można było używać ich właściwości i metody rozszerzenia:

```csharp
public class CustomFileReader
{
    private readonly IHostingEnvironment _env;

    public CustomFileReader(IHostingEnvironment env)
    {
        _env = env;
    }

    public string ReadFile(string filePath)
    {
        var fileProvider = _env.WebRootFileProvider;
        // Process the file here
    }
}
```

Można użyć [podejście oparte na Konwencji](xref:fundamentals/environments#startup-conventions) Aby skonfigurować aplikację podczas uruchamiania na podstawie środowiska. Alternatywnie można wstrzyknąć `IHostingEnvironment` do `Startup` konstruktora do użycia w `ConfigureServices`:

```csharp
public class Startup
{
    public Startup(IHostingEnvironment env)
    {
        HostingEnvironment = env;
    }

    public IHostingEnvironment HostingEnvironment { get; }

    public void ConfigureServices(IServiceCollection services)
    {
        if (HostingEnvironment.IsDevelopment())
        {
            // Development configuration
        }
        else
        {
            // Staging/Production configuration
        }

        var contentRootPath = HostingEnvironment.ContentRootPath;
    }
}
```

> [!NOTE]
> Oprócz `IsDevelopment` — metoda rozszerzenia, `IHostingEnvironment` oferuje `IsStaging`, `IsProduction`, i `IsEnvironment(string environmentName)` metody. Zobacz [Praca w środowiskach wielu](xref:fundamentals/environments) szczegółowe informacje.

`IHostingEnvironment` Usługi także mogą zostać dodane bezpośrednio do `Configure` metody do konfigurowania potoku przetwarzania sieci:

```csharp
public void Configure(IApplicationBuilder app, IHostingEnvironment env)
{
    if (env.IsDevelopment())
    {
        // In Development, use the developer exception page
        app.UseDeveloperExceptionPage();
    }
    else
    {
        // In Staging/Production, route exceptions to /error
        app.UseExceptionHandler("/error");
    }

    var contentRootPath = env.ContentRootPath;
}
```

Można wstrzyknąć `IHostingEnvironment` do `Invoke` metody podczas tworzenia niestandardowego [oprogramowanie pośredniczące](xref:fundamentals/middleware#writing-middleware):

```csharp
public async Task Invoke(HttpContext context, IHostingEnvironment env)
{
    if (env.IsDevelopment())
    {
        // Configure middleware for Development
    }
    else
    {
        // Configure middleware for Staging/Production
    }

    var contentRootPath = env.ContentRootPath;
}
```

## <a name="iapplicationlifetime-interface"></a>Interfejs IApplicationLifetime

[Interfejsu IApplicationLifetime](/aspnet/core/api/microsoft.aspnetcore.hosting.iapplicationlifetime) umożliwia wykonywanie czynności po uruchamiania i wyłączania. Trzy właściwości w interfejsie są anulowanie tokenów, które można zarejestrować z `Action` metod, aby określić zdarzenia uruchamiania i wyłączania. Istnieje również `StopApplication` metody.

| Token anulowania    | Wyzwalane podczas &#8230; |
| --------------------- | --------------------- |
| `ApplicationStarted`  | Host pełni została uruchomiona. |
| `ApplicationStopping` | Host wykonuje łagodne zamykanie. Żądania mogą nadal być przetwarzane. Bloki zamknięcia aż do zakończenia tego zdarzenia. |
| `ApplicationStopped`  | Host jest kończonych łagodne zamykanie. Wszystkie żądania należy całkowicie przetworzone. Bloki zamknięcia aż do zakończenia tego zdarzenia. |

| Metoda            | Akcja                                           |
| ----------------- | ------------------------------------------------ |
| `StopApplication` | Zakończenie żądania bieżącej aplikacji. |

```csharp
public class Startup 
{
    public void Configure(IApplicationBuilder app, IApplicationLifetime appLifetime) 
    {
        appLifetime.ApplicationStarted.Register(OnStarted);
        appLifetime.ApplicationStopping.Register(OnStopping);
        appLifetime.ApplicationStopped.Register(OnStopped);

        Console.CancelKeyPress += (sender, eventArgs) =>
        {
            appLifetime.StopApplication();
            // Don't terminate the process immediately, wait for the Main thread to exit gracefully.
            eventArgs.Cancel = true;
        };
    }

    private void OnStarted()
    {
        // Perform post-startup activities here
    }

    private void OnStopping()
    {
        // Perform on-stopping activities here
    }

    private void OnStopped()
    {
        // Perform post-stopped activities here
    }
}
```

## <a name="troubleshooting-systemargumentexception"></a>Rozwiązywanie problemów z System.ArgumentException

**Dotyczy tylko platformy ASP.NET Core 2.0**

W przypadku tworzenia hosta przez wstrzykiwanie `IStartup` bezpośrednio do kontenera iniekcji zależności zamiast wywoływania `UseStartup` lub `Configure`, może wystąpić następujący błąd: `Unhandled Exception: System.ArgumentException: A valid non-empty application name must be provided`.

Dzieje się tak dlatego [applicationName(ApplicationKey)](/aspnet/core/api/microsoft.aspnetcore.hosting.webhostdefaults#Microsoft_AspNetCore_Hosting_WebHostDefaults_ApplicationKey) (bieżącego zestawu) jest wymagane do skanowania pod kątem `HostingStartupAttributes`. Jeśli ręcznie wprowadzić `IStartup` do kontenera iniekcji zależności, Dodaj następujące wywołanie do Twojej `WebHostBuilder` o podanej nazwie zestawu:

```csharp
WebHost.CreateDefaultBuilder(args)
    .UseSetting("applicationName", "<Assembly Name>")
    ...
```

Możesz też dodać manekina `Configure` do Twojej `WebHostBuilder`, który określa `applicationName`(`ApplicationKey`) automatycznie:

```csharp
WebHost.CreateDefaultBuilder(args)
    .Configure(_ => { })
    ...
```

**Uwaga**: jest to tylko wymagane wraz z wydaniem programu ASP.NET 2.0 rdzeni i tylko wtedy gdy zgłaszasz nie `UseStartup` lub `Configure`.

Aby uzyskać więcej informacji, zobacz [anonsów: Microsoft.Extensions.PlatformAbstractions została usunięta (komentarz)](https://github.com/aspnet/Announcements/issues/237#issuecomment-323786938) i [próbki StartupInjection](https://github.com/aspnet/Hosting/blob/8377d226f1e6e1a97dabdb6769a845eeccc829ed/samples/SampleStartups/StartupInjection.cs).

## <a name="additional-resources"></a>Dodatkowe zasoby

* [Publikowanie do systemu Windows za pomocą usług IIS](../publishing/iis.md)
* [Publikowanie w systemie Linux przy użyciu Nginx](../publishing/linuxproduction.md)
* [Publikowanie w systemie Linux przy użyciu Apache](../publishing/apache-proxy.md)
* [Host usługi systemu Windows](xref:hosting/windows-service)