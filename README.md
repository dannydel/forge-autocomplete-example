# Forge Autocomplete Issue

A **Blazor WebAssembly** demonstration application showcasing integration with **Tyler Forge** web components (v3.13.1). This project serves as a test case and reference implementation for using the `forge-autocomplete` component in a Blazor WASM environment.

## Prerequisites

- [.NET 10 SDK](https://dotnet.microsoft.com/download/dotnet/10.0)

## Quick Start

```bash
# Clone the repository
git clone <repository-url>
cd ForgeAutocompleteIssue

# Build the project
dotnet build

# Run the application
dotnet run --project src/ForgeAutocompleteIssue.csproj
```

The application will launch at:
- **HTTP**: http://localhost:5166
- **HTTPS**: https://localhost:7272

For development with hot reload:

```bash
dotnet watch --project src/ForgeAutocompleteIssue.csproj
```

## Project Structure

```
src/
├── Components/
│   ├── ForgeAutocomplete.razor      # Generic Blazor wrapper for forge-autocomplete
│   └── ForgeAutocomplete.razor.js   # JS interop bridge for web component APIs
├── Models/
│   ├── AutocompleteFilterData.cs    # Filter callback input model
│   ├── AutocompleteSelectEventData.cs # Select event payload model
│   └── ListDropdownOption.cs        # Dropdown option structure
├── Pages/
│   ├── Home.razor                   # Demo page with fruit autocomplete
│   └── ...
├── wwwroot/
│   └── index.html                   # Entry point with Forge CDN imports
└── Program.cs                       # Blazor WASM bootstrap
```

## Architecture

### Blazor + Web Components Integration Pattern

This project demonstrates a **colocated JavaScript interop** pattern for integrating web components with Blazor:

1. **Razor Component** (`ForgeAutocomplete.razor`) - Renders the web component markup and manages Blazor-side state
2. **JS Module** (`ForgeAutocomplete.razor.js`) - Bridges the web component's JavaScript API to C# via `DotNetObjectReference`

The pattern works as follows:

```
┌─────────────────────────────────────────────────────────────────┐
│                     Blazor Component                            │
│  ┌─────────────────┐      ┌──────────────────────────────────┐  │
│  │ ForgeAutocomplete│◄────►│ DotNetObjectReference           │  │
│  │ .razor           │      │ (exposes C# methods to JS)      │  │
│  └─────────────────┘      └──────────────────────────────────┘  │
│           │                              ▲                      │
│           │ renders                      │ invokes              │
│           ▼                              │                      │
│  ┌─────────────────┐      ┌──────────────────────────────────┐  │
│  │ <forge-         │      │ ForgeAutocomplete.razor.js       │  │
│  │  autocomplete>  │◄────►│ - Sets filter callback           │  │
│  │                 │      │ - Listens to select events       │  │
│  └─────────────────┘      └──────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────┘
```

### Key Integration Points

**Filter Callback**: The autocomplete's `filter` property is set to an async JavaScript function that calls back into C# via `dotNetRef.invokeMethodAsync()`.

**Select Event**: The `forge-autocomplete-select` event is listened to in JavaScript and forwarded to C# for two-way binding updates.

### Tyler Forge Setup

Tyler Forge is loaded via CDN in `src/wwwroot/index.html`:

```html
<!-- CSS -->
<link rel="stylesheet" href="https://cdn.forge.tylertech.com/v1/libs/@tylertech/forge@3.13.1/forge.css" />

<!-- JavaScript Module -->
<script type="module" src="https://cdn.forge.tylertech.com/v1/libs/@tylertech/forge@3.13.1/index.js"></script>
```

## Usage Example

The `ForgeAutocomplete` component is a generic wrapper that can be used with any value type:

```razor
@using ForgeAutocompleteIssue.Components
@using ForgeAutocompleteIssue.Models

<ForgeAutocomplete
    Id="my-autocomplete"
    TValue="string"
    @bind-Value="_selectedValue"
    FilterFunc="@FilterOptions"
    Label="Select a fruit"
    HasPopoverIcon="true"
    ShowClear="true"
    PopoverPlacement="bottom" />

@code {
    private string? _selectedValue;

    private async Task<IList<ListDropdownOption<string>>> FilterOptions(
        AutocompleteFilterData<string> filterData)
    {
        var options = new[] { "Apple", "Banana", "Cherry" };

        return options
            .Where(o => string.IsNullOrEmpty(filterData.FilterText) ||
                        o.Contains(filterData.FilterText, StringComparison.OrdinalIgnoreCase))
            .Select(o => new ListDropdownOption<string> { Label = o, Value = o })
            .ToList();
    }
}
```

## Component Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `Id` | `string?` | Auto-generated GUID | Element identifier |
| `TValue` | Type parameter | Required | The type of the selected value |
| `Value` | `TValue?` | `null` | The currently selected value |
| `ValueChanged` | `EventCallback<TValue?>` | - | Two-way binding callback |
| `FilterFunc` | `Func<AutocompleteFilterData<TValue>, Task<IList<ListDropdownOption<TValue>>>>` | Required | Async function to filter and return options |
| `Label` | `string` | `"Autocomplete Label"` | Input field label |
| `HasPopoverIcon` | `bool` | `false` | Show dropdown indicator icon |
| `ShowClear` | `bool` | `false` | Show clear button |
| `PopoverPlacement` | `string?` | `"bottom"` | Dropdown placement position |

## Technologies

- **Blazor WebAssembly** (.NET 10)
- **Tyler Forge** Web Components (v3.13.1)
- **JavaScript Interop** for web component bridge

## License

[Add your license here]
