# Blazor NET8 Test

This is a test project I created to ask for help.

## Scenario

I'm creating a new website with Blazor Web App with `NET8`. Out of the box, Visual Studio creates 2 projects for the `Interactive Mode Auto`, one for _server_ and another one for _client_.

I created a component with NET7 to display a select with images and the source code is on [GitHub](https://github.com/erossini/BlazorImageSelect). The component is working just fine in a Blazor Web Assembly based on NET7.

Now, I want to use the same component in the Blazor Web App with `NET8`. I added the component to the client project and in the _App.razor_ in the server project the required CSS and the JavaScript.

Then, I created a Razor page, marked as `@attribute [RenderModeInteractiveAuto]`, to display a select using the `ImageSelect`. When I open this page, I get this error

> InvalidOperationException: JavaScript interop calls cannot be issued at this time. This is because the component is being statically rendered. When prerendering is enabled, JavaScript interop calls can only be performed during the OnAfterRenderAsync lifecycle method.

[![enter image description here][1]][1]

So, in the ImageSelect component, I call the JSInterop in order to display the select with this code that raises the error above.

```
protected override async Task OnInitializedAsync()
{
    if (string.IsNullOrEmpty(Id))
        Id = "imageselect-" + Guid.NewGuid().ToString();

    if (Config == null)
        Config = new Config() { };

    if (!string.IsNullOrEmpty(CssClass))
        Config.MainCss += " " + CssClass;

    var dotNetObjectRef = DotNetObjectReference.Create(this);
    JSModule = new ImageSelectJsInterop(JSRuntime);
    await JSModule.Setup(dotNetObjectRef, Id, Config);
}
```

So, I remove this code and I added this code as the screenshot suggests:

```
protected override async Task OnAfterRenderAsync(bool firstRender)
{
    if (string.IsNullOrEmpty(Id))
        Id = "imageselect-" + Guid.NewGuid().ToString();

    if (Config == null)
        Config = new Config() { };

    if (!string.IsNullOrEmpty(CssClass))
        Config.MainCss += " " + CssClass;

    var dotNetObjectRef = DotNetObjectReference.Create(this);
    JSModule = new ImageSelectJsInterop(JSRuntime);
    await JSModule.Setup(dotNetObjectRef, Id, Config);

    await base.OnAfterRenderAsync(firstRender);
}
```

Now, the render of the component has a funny behaviour. It renders the `select` with images (on the left side of the following screenshot) but also displays another `select` only with the text (on the right).

[![enter image description here][2]][2]

At this point, my questions are: 
- in what project can I use components with JavaScript (server or client?)
- how can I use a component that requires JavaScript?

### ImageSelect

When the user clicks on an item, the callback is not working. 

  [1]: https://i.stack.imgur.com/ID7dl.png
  [2]: https://i.stack.imgur.com/4u6tQ.png