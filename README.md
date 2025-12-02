# **README.md for Smart Search Frontend (Blazor)**

````markdown
# Smart Search Frontend (Blazor)

This project is a Blazor frontend for the Smart Search Solution API.  
It demonstrates a clean, strongly-typed implementation for consuming a backend API that searches across multiple domains like software, web design, plumbing, construction, electrical, and networking.  
This project is suitable for teaching students Blazor, HTTP services, and MudBlazor UI components.

---

##  Packages Required

Make sure the following packages are installed:

```bash
dotnet add package MudBlazor
dotnet add package Microsoft.AspNetCore.Components.Web
dotnet add package Microsoft.AspNetCore.Components.WebAssembly
````

> Note: For Blazor Server, you only need `MudBlazor` and `Microsoft.AspNetCore.Components.Web`.

---

## Project Setup

1. **Create a new Blazor Server project** in Visual Studio:

```
File > New > Project > Blazor Server App
```

2. Install **MudBlazor** via NuGet:

```
Tools > NuGet Package Manager > Package Manager Console
Install-Package MudBlazor
```

3. Add **HttpClient support** in `Program.cs`:

```csharp
using SearchSolutionClient.Services;
using MudBlazor.Services;

var builder = WebApplication.CreateBuilder(args);

// MudBlazor services
builder.Services.AddMudServices();

// HttpClient for SmartSearchService
builder.Services.AddHttpClient<SmartSearchService>();

builder.Services.AddRazorPages();
builder.Services.AddServerSideBlazor();

var app = builder.Build();

app.UseStaticFiles();
app.UseRouting();

app.MapBlazorHub();
app.MapFallbackToPage("/_Host");

app.Run();
```

---

## Project Structure

```
SearchSolutionClient/
├─ Models/
│  └─ SearchResponse.cs
├─ Services/
│  └─ SmartSearchService.cs
├─ Pages/
│  └─ Search.razor
├─ Program.cs
├─ _Imports.razor
└─ App.razor
```


## 

**Models/SearchResponse.cs**

```
namespace SearchSolutionClient.Models;

public class SearchResponse
{
    public string Query { get; set; }
    public string Category { get; set; }
    public List<SearchBlock> Results { get; set; } = new();
}

public class SearchBlock
{
    public List<SearchItem> Items { get; set; } = new();
}

public class SearchItem
{
    public string Title { get; set; }
    public string Link { get; set; }
}
```


---

##  Service to Call Backend API

**Services/SmartSearchService.cs**

```
using System.Net.Http.Json;
using SearchSolutionClient.Models;

namespace SearchSolutionClient.Services;

public class SmartSearchService
{
    private readonly HttpClient _http;

    public SmartSearchService(HttpClient http)
    {
        _http = http;
    }

    public async Task<SearchResponse?> SearchAsync(string query)
    {
        if (string.IsNullOrWhiteSpace(query))
            return null;

        try
        {
            return await _http.GetFromJsonAsync<SearchResponse>(
                ""
            );
        }
        catch
        {
            return null;
        }
    }
}
```

---

##  Blazor Page

**Pages/Search.razor**

```razor
@page "/search"
@using SearchSolutionClient.Services
@using SearchSolutionClient.Models
@inject SmartSearchService SearchService
@using MudBlazor
@using Microsoft.AspNetCore.Components.Web

<MudContainer Class="mt-6">

    <MudTextField @bind-Value="query"
                  Placeholder="Search anything..."
                  Immediate="true"
                  DebounceInterval="500"
                  Adornment="Adornment.Start"
                  AdornmentIcon="@Icons.Material.Filled.Search"
                  Class="w-100"
                  OnKeyUp="HandleKeyUp" />

    @if (loading)
    {
        <div class="mt-4 text-center">
            <MudProgressCircular Indeterminate="true" />
        </div>
    }

    @if (!loading && response is not null)
    {
        @foreach (var block in response.Results)
        {
            foreach (var item in block.Items)
            {
                <MudCard Class="mt-4">
                    <MudCardContent>
                        <h4>@item.Title</h4>
                        <p>
                            <a href="@item.Link" target="_blank">@item.Link</a>
                        </p>
                    </MudCardContent>
                </MudCard>
            }
        }
    }

</MudContainer>

@code {
    string query = "";
    bool loading = false;
    SearchResponse? response;

    async Task HandleKeyUp(KeyboardEventArgs e)
    {
        if (e.Key == "Enter")
        {
            await DoSearch();
        }
    }

    async Task DoSearch()
    {
        if (string.IsNullOrWhiteSpace(query))
            return;

        loading = true;
        response = await SearchService.SearchAsync(query);
        loading = false;
    }
}
```

---

