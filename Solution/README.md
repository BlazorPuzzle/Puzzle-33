# Blazor Puzzle #33

## The Components that Bind

YouTube Video: https://youtu.be/jqzKepOfwzA

Blazor Puzzle Home Page: https://blazorpuzzle.com

### The Challenge:

We want to know why changing a component’s parameter value doesn’t update when bound to a  value in a parent component.

Starting with a .NET 8 Blazor Web App in Global Server interactive mode, we added a component called `Component1`:

*Component1.razor*:

```c#
<input value="@TextValue" @onchange="OnValueChanged" />

@code {

    [Parameter] 
    public string TextValue { get; set; } = string.Empty;

    protected async Task OnValueChanged(ChangeEventArgs? val)
    {
        if (val == null || val.Value == null) return;

        // Why doesn't this update?
        TextValue = ((string)val.Value) ?? string.Empty;
        StateHasChanged();
    }
}
```

The home page shows the component and handles a button click:

*Home.razor*:

```c#
@page "/"

<PageTitle>Home</PageTitle>

<p>
    This is a Global Server Blazor Web App.
    We have defined a component with a text parameter called TextValue.
    The component hosts an input text box that changes the value of the
    TextValue string.
</p>

<p>
    We expected that the value of the TextValue string would be updated
    in the page that hosts the component whenver the user changes the
    value of the input text box, but it doesn't.
</p>

<h3>
    @Text
</h3>

<MyComponent TextValue="@Text" />

@code {

    string Text = "My Text Value";
}
```

The problem is that the page does not update when you click the button.

What's going on and how can we fix it?

### The Solution:

In *Home.razor*, change the parameter assignment to a binding like so:

```c#
@page "/"

<PageTitle>Home</PageTitle>

<p>
    This is a Global Server Blazor Web App.
    We have defined a component with a text parameter called TextValue.
    The component hosts an input text box that changes the value of the
    TextValue string.
</p>

<p>
    We expected that the value of the TextValue string would be updated
    in the page that hosts the component whenver the user changes the
    value of the input text box, but it doesn't.
</p>

<h3>
    @Text
</h3>

<MyComponent @bind-TextValue="@Text" />

@code {

    string Text = "My Text Value";
}
```

And change *MyComponent.razor* to add an Event Callback with the same name as the parameter but ending in "Changed." That convention ensures the binding will work in both directions:

```c#
<input value="@TextValue" @onchange="OnValueChanged" />

@code {
    [Parameter] public string TextValue { get; set; } = string.Empty;
    [Parameter] public EventCallback<string> TextValueChanged { get; set; }

    protected async Task OnValueChanged(ChangeEventArgs? val)
    {
        if (val == null || val.Value == null) return;
        
        string stringValue = ((string)val.Value) ?? string.Empty;
        await TextValueChanged.InvokeAsync(stringValue);
    }
}
```

