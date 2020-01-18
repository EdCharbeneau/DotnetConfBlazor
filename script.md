# Step 1

The singular alert box

```
<div class="alert alert-danger">
    <span>Test Message</span>
</div>
```

# Step 2

File > New Component
`AlertMessage.razor`

**Add to _Imports.razor**

```
<div class="alert alert-danger">
    @ChildContent
</div>

@code {
    [Parameter] public RenderFragment ChildContent { get; set; }
}
```

# Step 3

Introduce @typeparam TItem

`AlertMessages.razor`
```
@typeparam TItem

<ul>
    @foreach (var item in Items)
    {
        @ItemTemplate(item)
    }
</ul>

@code {
    [Parameter]
    public RenderFragment<TItem> ItemTemplate { get; set; }

    [Parameter]
    public IReadOnlyList<TItem> Items { get; set; }
}
```

Rename parameters & move tags

```
@typeparam TItem

@foreach (var item in Items)
{
    <AlertMessage>
        @AlertTemplate(item)
    </AlertMessage>
}

@code {
    [Parameter]
    public RenderFragment<TItem> AlertTemplate { get; set; }

    [Parameter]
    public IReadOnlyList<TItem> Items { get; set; }
}
```

## Step 3A

Modify Counter.razor
```
<hr />
<AlertMessages Items="alerts" Context="message">
    <AlertTemplate>
        <span>This is an alert @message</span>
    </AlertTemplate>
</AlertMessages>

@code {
    List<int> alerts = new List<int>();

    private void IncrementCount()
    {
        alerts.Add(alerts.Count() + 1);
    }
}
```

# Step 4

Add event callback
```
[Parameter]
public EventCallback<object> OnDelete { get; set; }

void HandleDelete(TItem item)
{
    OnDelete.InvokeAsync(item);
}
```

Add delete button
```
@if (OnDelete.HasDelegate)
{
    <button @onclick="@(_ => HandleDelete(item))" type="button" class="close" data-dismiss="alert" aria-label="Close">
        <span aria-hidden="true">&times;</span>
    </button>
}
```
Consume

```
void DeleteItem(object item)
{
    alerts.Remove((int)item);
}
```

# Step 5

SpaLoader

```
@if (IsLoading)
{
    @LoadingTemplate
}
else
{
    @ContentTemplate
}

@code {
    [Parameter] public bool IsLoading { get; set; }

    [Parameter] public RenderFragment LoadingTemplate { get; set; }

    [Parameter] public RenderFragment ContentTemplate { get; set; }

}
```
Fetch Data
```
<SpaLoader IsLoading="forecasts == null">
    <LoadingTemplate>
        Loading...
    </LoadingTemplate>
    <ContentTemplate>
        <table class="table">
            <thead>
                <tr>
                    <th>Date</th>
                    <th>Temp. (C)</th>
                    <th>Temp. (F)</th>
                    <th>Summary</th>
                </tr>
            </thead>
            <tbody>
                @foreach (var forecast in forecasts)
                {
                    <tr>
                        <td>@forecast.Date.ToShortDateString()</td>
                        <td>@forecast.TemperatureC</td>
                        <td>@forecast.TemperatureF</td>
                        <td>@forecast.Summary</td>
                    </tr>
                }
            </tbody>
        </table>
    </ContentTemplate>
</SpaLoader>
```

# Step 6

SpinLoader

```
<h1>Weather forecast</h1>

<p>This component demonstrates fetching data from the server.</p>
<p>
    The &lt;SpinLoader&gt; component supports three template regions:
    <ul>
        <li>LoadingTemplate (optional) - content to display while the IsLoading property is <b>true</b>.</li>
        <li>ContentTemplate - content to display when the IsLoading is <b>false</b></li>
        <li>FaultedTemplate - content to display when IsFaulted is <b>true</b> and IsLoading is <b>false</b></li>
    </ul>
</p>
<div class="form-row align-items-center">
    <div class="col-auto">
        <div class="form-check mb-2">
            <input id="forceException" type="checkbox" @bind="forceException" class="form-check-input" />
            <label class="form-check-label" for="forceException">Force exception</label>
        </div>
    </div>
    <div class="col-auto">
        <button class="btn btn-primary mb-2" @onclick="LoadData">Retry</button>
    </div>
</div>
<table class="table">
    <thead>
        <tr>
            <th>Date</th>
            <th>Temp. (C)</th>
            <th>Temp. (F)</th>
            <th>Summary</th>
        </tr>
    </thead>
    <tbody>
        <SpinLoader IsLoading="isLoading" IsFaulted="isFaulted">
            <LoadingTemplate>
                <tr>
                    <td colspan="4" style="vertical-align: middle;background-color: rgb(0, 0, 0, .2); height:300px;">
                        <Circle Color="#e67e22" Size="60px" Center="true" />
                    </td>
                </tr>
            </LoadingTemplate>
            <ContentTemplate>
                @foreach (var forecast in forecasts)
                {
                    <tr>
                        <td>@forecast.Date.ToShortDateString()</td>
                        <td>@forecast.TemperatureC</td>
                        <td>@forecast.TemperatureF</td>
                        <td>@forecast.Summary</td>
                    </tr>
                }
            </ContentTemplate>
            <FaultedContentTemplate>
                <tr>
                    <td colspan="4">
                        <div class="alert alert-danger">Fail</div>
                    </td>
                </tr>
            </FaultedContentTemplate>
        </SpinLoader>
    </tbody>
</table>

@code {

    WeatherForecast[] forecasts;
    bool isFaulted = false;
    bool isLoading = true;
    int delay = 2000;
    bool forceException = false;

    protected override async Task OnInitializedAsync()
    {
        await LoadData();
    }

    async Task LoadData()
    {
        await TryLoadingData(
            onSuccess: SuccessPath,
            onFaulted: FaultedPath
            );
    }

    void SuccessPath(WeatherForecast[] data)
    {
        // do work
        forecasts = data;
    }

    void FaultedPath(Exception e)
    {
        // log message, don't share it with the user
        var fakeLog = e.Message;
    }

    async Task TryLoadingData(Action<WeatherForecast[]> onSuccess, Action<Exception> onFaulted)
    {
        isLoading = true;
        await Task.Delay(delay);
        try
        {
            if (forceException)
            {
                throw new NotSupportedException();
            }
            var data = await Http.GetJsonAsync<WeatherForecast[]>("sample-data/weather.json");
            isFaulted = false;
            onSuccess(data);
        }
        catch (Exception e)
        {
            isFaulted = true;
            onFaulted(e);
        }
        finally
        {
            isLoading = false;
        }
    }

    public class WeatherForecast
    {
        public DateTime Date { get; set; }

        public int TemperatureC { get; set; }

        public string Summary { get; set; }

        public int TemperatureF => 32 + (int)(TemperatureC / 0.5556);
    }
}
```

# Step 7

Component Libs

File > New RCL

Move components

Change Libman output

Add Blazor Utilities

Change namespaces

Add project ref

    <link href="_content/Spinkit/spinkit.min.css" rel="stylesheet" />

Update _Imports
