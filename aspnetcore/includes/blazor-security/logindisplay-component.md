<span data-ttu-id="d126d-101">Součást `LoginDisplay` (*Shared/LoginDisplay. Razor*) je vykreslena ve `MainLayout` komponentě (*Shared/MainLayout. Razor*) a spravuje následující chování:</span><span class="sxs-lookup"><span data-stu-id="d126d-101">The `LoginDisplay` component (*Shared/LoginDisplay.razor*) is rendered in the `MainLayout` component (*Shared/MainLayout.razor*) and manages the following behaviors:</span></span>

* <span data-ttu-id="d126d-102">Pro ověřené uživatele:</span><span class="sxs-lookup"><span data-stu-id="d126d-102">For authenticated users:</span></span>
  * <span data-ttu-id="d126d-103">Zobrazí aktuální uživatelské jméno.</span><span class="sxs-lookup"><span data-stu-id="d126d-103">Displays the current username.</span></span>
  * <span data-ttu-id="d126d-104">Nabízí tlačítko pro odhlášení od aplikace.</span><span class="sxs-lookup"><span data-stu-id="d126d-104">Offers a button to log out of the app.</span></span>
* <span data-ttu-id="d126d-105">Pro anonymní uživatele nabízí možnost přihlásit se.</span><span class="sxs-lookup"><span data-stu-id="d126d-105">For anonymous users, offers the option to log in.</span></span>

```razor
@using Microsoft.AspNetCore.Components.Authorization
@using Microsoft.AspNetCore.Components.WebAssembly.Authentication
@inject NavigationManager Navigation
@inject SignOutSessionStateManager SignOutManager

<AuthorizeView>
    <Authorized>
        Hello, @context.User.Identity.Name!
        <button class="nav-link btn btn-link" @onclick="BeginSignOut">
            Log out
        </button>
    </Authorized>
    <NotAuthorized>
        <a href="authentication/login">Log in</a>
    </NotAuthorized>
</AuthorizeView>

@code {
    private async Task BeginSignOut(MouseEventArgs args)
    {
        await SignOutManager.SetSignOutState();
        Navigation.NavigateTo("authentication/logout");
    }
}
```