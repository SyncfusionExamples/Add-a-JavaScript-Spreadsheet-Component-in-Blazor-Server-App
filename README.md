# Add a JavaScript Spreadsheet Component in a Blazor Server App

This repository demonstrates how to integrate a JavaScript Spreadsheet component into a Blazor Server application targeting .NET 6. The sample shows how to reference the Spreadsheet’s JavaScript and CSS assets, initialize it using JavaScript interop, and render it on a Razor page. While the example is aligned with the Syncfusion EJ2 JavaScript Spreadsheet loaded via CDN for convenience, the overall approach works with other JavaScript spreadsheet libraries as well. The goal is to help you understand where to place assets, how to structure initialization, and how to keep the component lifecycle clean in a Blazor Server context.
 
Note: If you use a commercial component, make sure to review and comply with its licensing terms.

## What You’ll Build

A .NET 6 Blazor Server application that:

- Loads a JavaScript spreadsheet component from a CDN or local assets.
- Initializes the spreadsheet via Blazor’s JavaScript interop.
- Displays a small in-memory dataset with basic formatting (e.g., bold headers, currency formatting).
- Configures common features like toolbar items, editing, sorting, and filtering.

## Tech Stack

- **.NET 6 (Blazor Server)**: Core framework for the application.
- **JavaScript Spreadsheet Library**: Example uses Syncfusion EJ2; adaptable to other libraries.
- **JavaScript Interop (`IJSRuntime`)**: Enables communication between Blazor and JavaScript.

## Prerequisites

- **.NET SDK 6.0**: Installed on your system.
- **IDE**: Visual Studio 2022 (v17.0 or newer) or Visual Studio Code with C# extensions.
- **Browser**: A modern browser (Microsoft Edge, Google Chrome, or Firefox).
- **Internet Access**: Required for CDN assets; optional if hosting assets locally.

## Repository Structure

- `Pages/Index.razor`: Hosts the spreadsheet container and triggers initialization via JavaScript interop.
- `Pages/_Layout.cshtml`: Includes stylesheet and script references for the spreadsheet library and interop logic.
- `Program.cs`: Configures the Blazor Server hosting setup for .NET 6.

## Getting Started

1. Clone the repository using your preferred Git client and navigate to the project folder.
2. Restore dependencies and build the solution:

   ```bash
   dotnet restore
   dotnet build
   ```

3. Run the project:

   ```bash
   dotnet run
   ```

4. Open your browser to the URL displayed in the console or IDE output (e.g., `https://localhost:5001`). You should see a spreadsheet populated with sample data. If issues occur, refer to the **Troubleshooting** section.

## Adding Spreadsheet Assets and Interop JavaScript

In `Pages/_Layout.cshtml`, include the spreadsheet library’s stylesheet in the `<head>` tag to ensure styles are available during rendering. Place the spreadsheet script bundle near the end of the `<body>`, after the Blazor Server script (`blazor.server.js`), to ensure proper initialization order. For CDN usage, reference the correct library version and dependencies. If external network access is restricted, download assets and serve them from the `wwwroot` folder, updating paths accordingly. Maintain this inclusion order:

1. Spreadsheet stylesheet (in `<head>`).
2. Blazor Server script.
3. Spreadsheet library bundle.
4. Custom interop script.

Example in `Pages/_Layout.cshtml`:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Blazor Spreadsheet Demo</title>
    <link href="https://cdn.syncfusion.com/ej2/bootstrap5.css" rel="stylesheet" />
</head>
<body>
    <!-- Page content -->
    <script src="_framework/blazor.server.js"></script>
    <script src="https://cdn.syncfusion.com/ej2/dist/ej2.min.js"></script>
    <script src="js/spreadsheet-init.js"></script>
</body>
</html>
```

Create a file named `spreadsheet-init.js` in `wwwroot/js` to handle initialization and disposal:

```javascript
function RenderSpreadsheet(spreadsheetElem, data) {
    new ej.spreadsheet.Spreadsheet({
        sheets: [{
            ranges: [{
                dataSource: data,
                showFieldAsHeader: false
            }]
        }],
        openUrl: 'https://ej2services.syncfusion.com/production/web-services/api/spreadsheet/open',
        saveUrl: 'https://ej2services.syncfusion.com/production/web-services/api/spreadsheet/save'
    }, spreadsheetElem);
}

function DisposeSpreadsheet(spreadsheetElem) {
    const spreadsheet = ej.base.getComponent(spreadsheetElem, 'spreadsheet');
    if (spreadsheet) {
        spreadsheet.destroy();
    }
}
```

## Rendering the Spreadsheet in a Blazor Page

In `Pages/Index.razor`, create a container `<div>` with a unique ID to host the spreadsheet. Use the `OnAfterRenderAsync` lifecycle method to invoke the JavaScript initializer via interop, ensuring it runs only on the first render. Implement disposal logic to clean up the spreadsheet instance when the component is destroyed or during navigation.

Example in `Pages/Index.razor`:

```razor
@page "/"
@inject IJSRuntime JSRuntime

<PageTitle>Spreadsheet Demo</PageTitle>

<div id="spreadsheet" @ref="spreadsheetElem"></div>

@code {
    private ElementReference spreadsheetElem;
    private bool isRendered;

    protected override async Task OnAfterRenderAsync(bool firstRender)
    {
        if (firstRender && !isRendered)
        {
            var data = new[]
            {
                new { OrderID = 1, Item = "Laptop", Price = 999.99 },
                new { OrderID = 2, Item = "Mouse", Price = 29.99 }
            };
            await JSRuntime.InvokeVoidAsync("RenderSpreadsheet", spreadsheetElem, data);
            isRendered = true;
        }
    }

    public async ValueTask DisposeAsync()
    {
        await JSRuntime.InvokeVoidAsync("DisposeSpreadsheet", spreadsheetElem);
    }
}
```

## How It Works (high level)

- **Visual Studio 2022**: Open the solution, set the Blazor Server project as the startup project, and start debugging.
- **.NET CLI**:

  ```bash
  dotnet restore
  dotnet build
  dotnet run
  ```

  Open the browser at the displayed URL (e.g., `https://localhost:5001`).
 
## How to run the project
 
- Visual Studio 2022:
  - Open the solution, set the Blazor Server project as the startup project, and start debugging. The browser should open automatically at the correct URL.
- .NET CLI:
  - From the project folder, run restore, then build, then run. Open the browser at the local HTTPS or HTTP URL printed in the console output.
  
## Contributing
 
Contributions are welcome. Please open an issue describing your scenario or bug, include steps to reproduce, and, if possible, a minimal repro. When submitting a pull request, explain the change and note any testing instructions.
 
## License

This project is licensed under terms outlined by Syncfusion.  
For detailed licensing information, please refer to the official documentation:  
[Syncfusion Document Processing Licensing Overview](https://help.syncfusion.com/document-processing/licensing/overview)
