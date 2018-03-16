---
title: "Sdílení souborů cookie mezi aplikace s ASP.NET a ASP.NET Core"
author: rick-anderson
description: "Naučte se sdílet soubory cookie pro ověřování mezi ASP.NET 4.x a ASP.NET Core aplikace."
manager: wpickett
ms.author: riande
ms.custom: mvc
ms.date: 01/19/2017
ms.prod: asp.net-core
ms.technology: aspnet
ms.topic: article
uid: security/cookie-sharing
ms.openlocfilehash: c2d022d8dc625d479bc690f410d4994a1d7e3d74
ms.sourcegitcommit: 493a215355576cfa481773365de021bcf04bb9c7
ms.translationtype: MT
ms.contentlocale: cs-CZ
ms.lasthandoff: 03/15/2018
---
# <a name="sharing-cookies-among-apps-with-aspnet-and-aspnet-core"></a><span data-ttu-id="26ed9-103">Sdílení souborů cookie mezi aplikace s ASP.NET a ASP.NET Core</span><span class="sxs-lookup"><span data-stu-id="26ed9-103">Sharing cookies among apps with ASP.NET and ASP.NET Core</span></span>

<span data-ttu-id="26ed9-104">Podle [Rick Anderson](https://twitter.com/RickAndMSFT) a [Luke Latham](https://github.com/guardrex)</span><span class="sxs-lookup"><span data-stu-id="26ed9-104">By [Rick Anderson](https://twitter.com/RickAndMSFT) and [Luke Latham](https://github.com/guardrex)</span></span>

<span data-ttu-id="26ed9-105">Weby často obsahovat jednotlivé webové aplikace, které pracují společně.</span><span class="sxs-lookup"><span data-stu-id="26ed9-105">Websites often consist of individual web apps working together.</span></span> <span data-ttu-id="26ed9-106">A poskytuje prostředí jednotné přihlašování (SSO), musí webové aplikace v rámci lokality sdílet soubory cookie pro ověřování.</span><span class="sxs-lookup"><span data-stu-id="26ed9-106">To provide a single sign-on (SSO) experience, web apps within a site must share authentication cookies.</span></span> <span data-ttu-id="26ed9-107">Pro podporu tohoto scénáře, zásobník ochrany dat umožňuje sdílení Katana ověřování souborů cookie a ASP.NET Core lístků pro ověřování pomocí souboru cookie.</span><span class="sxs-lookup"><span data-stu-id="26ed9-107">To support this scenario, the data protection stack allows sharing Katana cookie authentication and ASP.NET Core cookie authentication tickets.</span></span>

<span data-ttu-id="26ed9-108">[Zobrazit nebo stáhnout ukázkový kód](https://github.com/aspnet/Docs/tree/master/aspnetcore/security/cookie-sharing/sample/) ([stažení](xref:tutorials/index#how-to-download-a-sample))</span><span class="sxs-lookup"><span data-stu-id="26ed9-108">[View or download sample code](https://github.com/aspnet/Docs/tree/master/aspnetcore/security/cookie-sharing/sample/) ([how to download](xref:tutorials/index#how-to-download-a-sample))</span></span>

<span data-ttu-id="26ed9-109">Ukázka ukazuje soubor cookie sdílení napříč tří aplikací, které používala ověřování souborů cookie:</span><span class="sxs-lookup"><span data-stu-id="26ed9-109">The sample illustrates cookie sharing across three apps that use cookie authentication:</span></span>

* <span data-ttu-id="26ed9-110">Aplikace ASP.NET Core stránky Razor 2.0 bez použití [ASP.NET Core Identity](xref:security/authentication/identity)</span><span class="sxs-lookup"><span data-stu-id="26ed9-110">ASP.NET Core 2.0 Razor Pages app without using [ASP.NET Core Identity](xref:security/authentication/identity)</span></span>
* <span data-ttu-id="26ed9-111">Jádro ASP.NET 2.0 aplikace MVC se službou ASP.NET Core Identity</span><span class="sxs-lookup"><span data-stu-id="26ed9-111">ASP.NET Core 2.0 MVC app with ASP.NET Core Identity</span></span>
* <span data-ttu-id="26ed9-112">Aplikace MVC rozhraní ASP.NET 4.6.1 s ASP.NET Identity</span><span class="sxs-lookup"><span data-stu-id="26ed9-112">ASP.NET Framework 4.6.1 MVC app with ASP.NET Identity</span></span>

<span data-ttu-id="26ed9-113">V příkladech, které následují:</span><span class="sxs-lookup"><span data-stu-id="26ed9-113">In the examples that follow:</span></span>

* <span data-ttu-id="26ed9-114">Název souboru cookie ověřování je nastaven na hodnotu běžné `.AspNet.SharedCookie`.</span><span class="sxs-lookup"><span data-stu-id="26ed9-114">The authentication cookie name is set to a common value of `.AspNet.SharedCookie`.</span></span>
* <span data-ttu-id="26ed9-115">`AuthenticationType` Je nastaven na `Identity.Application` explicitně nebo ve výchozím nastavení.</span><span class="sxs-lookup"><span data-stu-id="26ed9-115">The `AuthenticationType` is set to `Identity.Application` either explicitly or by default.</span></span>
* <span data-ttu-id="26ed9-116">[CookieAuthenticationDefaults.AuthenticationScheme](/dotnet/api/microsoft.aspnetcore.authentication.cookies.cookieauthenticationdefaults.authenticationscheme) slouží jako schéma ověřování.</span><span class="sxs-lookup"><span data-stu-id="26ed9-116">[CookieAuthenticationDefaults.AuthenticationScheme](/dotnet/api/microsoft.aspnetcore.authentication.cookies.cookieauthenticationdefaults.authenticationscheme) is used as the authentication scheme.</span></span> <span data-ttu-id="26ed9-117">Konstanta přeloží na hodnotu `Cookies`.</span><span class="sxs-lookup"><span data-stu-id="26ed9-117">The constant resolves to a value of `Cookies`.</span></span>
* <span data-ttu-id="26ed9-118">Middleware ověřování souborů cookie používá provádění [DataProtectionProvider](/dotnet/api/microsoft.aspnetcore.dataprotection.dataprotectionprovider).</span><span class="sxs-lookup"><span data-stu-id="26ed9-118">The cookie authentication middleware uses an implementation of [DataProtectionProvider](/dotnet/api/microsoft.aspnetcore.dataprotection.dataprotectionprovider).</span></span> <span data-ttu-id="26ed9-119">`DataProtectionProvider` poskytuje služby ochrany dat pro šifrování a dešifrování dat datové části souboru cookie ověřování.</span><span class="sxs-lookup"><span data-stu-id="26ed9-119">`DataProtectionProvider` provides data protection services for the encryption and decryption of authentication cookie payload data.</span></span> <span data-ttu-id="26ed9-120">`DataProtectionProvider` Instance je izolovaná od systému ochrany dat používá dalších částí aplikace.</span><span class="sxs-lookup"><span data-stu-id="26ed9-120">The `DataProtectionProvider` instance is isolated from the data protection system used by other parts of the app.</span></span>
  * <span data-ttu-id="26ed9-121">Společné [klíč ochrany dat](xref:security/data-protection/implementation/key-management) umístění úložiště se používá.</span><span class="sxs-lookup"><span data-stu-id="26ed9-121">A common [data protection key](xref:security/data-protection/implementation/key-management) storage location is used.</span></span> <span data-ttu-id="26ed9-122">Ukázková aplikace používá složku s názvem *Správce klíčů* v kořenovém adresáři řešení pro ochranu dat klíče.</span><span class="sxs-lookup"><span data-stu-id="26ed9-122">The sample app uses a folder named *KeyRing* at the root of the solution to hold the data protection keys.</span></span>
  * <span data-ttu-id="26ed9-123">[DataProtectionProvider.Create(System.IO.DirectoryInfo)](/dotnet/api/microsoft.aspnetcore.dataprotection.dataprotectionprovider.create?view=aspnetcore-2.0#Microsoft_AspNetCore_DataProtection_DataProtectionProvider_Create_System_IO_DirectoryInfo_) přijme [DirectoryInfo](/dotnet/api/system.io.directoryinfo) pro použití s ověřovací soubory cookie.</span><span class="sxs-lookup"><span data-stu-id="26ed9-123">[DataProtectionProvider.Create(System.IO.DirectoryInfo)](/dotnet/api/microsoft.aspnetcore.dataprotection.dataprotectionprovider.create?view=aspnetcore-2.0#Microsoft_AspNetCore_DataProtection_DataProtectionProvider_Create_System_IO_DirectoryInfo_) accepts a [DirectoryInfo](/dotnet/api/system.io.directoryinfo) for use with authentication cookies.</span></span> <span data-ttu-id="26ed9-124">Ukázková aplikace obsahuje cestu *Správce klíčů* složku pro `DirectoryInfo`.</span><span class="sxs-lookup"><span data-stu-id="26ed9-124">The sample app provides the path of the *KeyRing* folder to `DirectoryInfo`.</span></span>
  * <span data-ttu-id="26ed9-125">[DataProtectionProvider](/dotnet/api/microsoft.aspnetcore.dataprotection.dataprotectionprovider) vyžaduje [Microsoft.AspNetCore.DataProtection.Extensions](https://www.nuget.org/packages/Microsoft.AspNetCore.DataProtection.Extensions/) balíček NuGet.</span><span class="sxs-lookup"><span data-stu-id="26ed9-125">[DataProtectionProvider](/dotnet/api/microsoft.aspnetcore.dataprotection.dataprotectionprovider) requires the [Microsoft.AspNetCore.DataProtection.Extensions](https://www.nuget.org/packages/Microsoft.AspNetCore.DataProtection.Extensions/) NuGet package.</span></span> <span data-ttu-id="26ed9-126">Pokud chcete získat tento balíček pro technologii ASP.NET 2.0 jádra a novější aplikace, odkazovat [Microsoft.AspNetCore.All](xref:fundamentals/metapackage) metapackage.</span><span class="sxs-lookup"><span data-stu-id="26ed9-126">To obtain this package for ASP.NET Core 2.0 and later apps, reference the [Microsoft.AspNetCore.All](xref:fundamentals/metapackage) metapackage.</span></span> <span data-ttu-id="26ed9-127">Při cílení na rozhraní .NET Framework, přidat odkaz na balíček `Microsoft.AspNetCore.DataProtection.Extensions`.</span><span class="sxs-lookup"><span data-stu-id="26ed9-127">When targeting .NET Framework, add a package reference to `Microsoft.AspNetCore.DataProtection.Extensions`.</span></span>

## <a name="share-authentication-cookies-among-aspnet-core-apps"></a><span data-ttu-id="26ed9-128">Sdílet soubory cookie pro ověřování mezi aplikací ASP.NET Core</span><span class="sxs-lookup"><span data-stu-id="26ed9-128">Share authentication cookies among ASP.NET Core apps</span></span>

<span data-ttu-id="26ed9-129">Při použití ASP.NET Core Identity:</span><span class="sxs-lookup"><span data-stu-id="26ed9-129">When using ASP.NET Core Identity:</span></span>

# <a name="aspnet-core-2xtabaspnetcore2x"></a>[<span data-ttu-id="26ed9-130">ASP.NET základní 2.x</span><span class="sxs-lookup"><span data-stu-id="26ed9-130">ASP.NET Core 2.x</span></span>](#tab/aspnetcore2x)

<span data-ttu-id="26ed9-131">V `ConfigureServices` metoda, použijte [ConfigureApplicationCookie](/dotnet/api/microsoft.extensions.dependencyinjection.identityservicecollectionextensions.configureapplicationcookie) metody rozšíření pro nastavení služby ochrany dat pro soubory cookie.</span><span class="sxs-lookup"><span data-stu-id="26ed9-131">In the `ConfigureServices` method, use the [ConfigureApplicationCookie](/dotnet/api/microsoft.extensions.dependencyinjection.identityservicecollectionextensions.configureapplicationcookie) extension method to set up the data protection service for cookies.</span></span>

[!code-csharp[](cookie-sharing/sample/CookieAuthWithIdentity.Core/Startup.cs?name=snippet1)]

<span data-ttu-id="26ed9-132">Najdete v článku *CookieAuthWithIdentity.Core* v projektu [ukázkový kód](https://github.com/aspnet/Docs/tree/master/aspnetcore/security/cookie-sharing/sample/) ([stažení](xref:tutorials/index#how-to-download-a-sample)).</span><span class="sxs-lookup"><span data-stu-id="26ed9-132">See the *CookieAuthWithIdentity.Core* project in the [sample code](https://github.com/aspnet/Docs/tree/master/aspnetcore/security/cookie-sharing/sample/) ([how to download](xref:tutorials/index#how-to-download-a-sample)).</span></span>

# <a name="aspnet-core-1xtabaspnetcore1x"></a>[<span data-ttu-id="26ed9-133">ASP.NET základní 1.x</span><span class="sxs-lookup"><span data-stu-id="26ed9-133">ASP.NET Core 1.x</span></span>](#tab/aspnetcore1x)

<span data-ttu-id="26ed9-134">V `Configure` metoda, použijte [CookieAuthenticationOptions](/dotnet/api/microsoft.aspnetcore.builder.cookieauthenticationoptions) nastavit:</span><span class="sxs-lookup"><span data-stu-id="26ed9-134">In the `Configure` method, use the [CookieAuthenticationOptions](/dotnet/api/microsoft.aspnetcore.builder.cookieauthenticationoptions) to set up:</span></span>

* <span data-ttu-id="26ed9-135">Služba ochrany dat pro soubory cookie.</span><span class="sxs-lookup"><span data-stu-id="26ed9-135">The data protection service for cookies.</span></span>
* <span data-ttu-id="26ed9-136">`AuthenticationScheme` Tak, aby odpovídaly ASP.NET 4.x.</span><span class="sxs-lookup"><span data-stu-id="26ed9-136">The `AuthenticationScheme` to match ASP.NET 4.x.</span></span>

```csharp
app.AddIdentity<ApplicationUser, IdentityRole>(options =>
{
    options.Cookies.ApplicationCookie.AuthenticationScheme = 
        "ApplicationCookie";

    var protectionProvider = 
        DataProtectionProvider.Create(
            new DirectoryInfo(@"PATH_TO_KEY_RING_FOLDER"));

    options.Cookies.ApplicationCookie.DataProtectionProvider = 
        protectionProvider;

    options.Cookies.ApplicationCookie.TicketDataFormat = 
        new TicketDataFormat(protectionProvider.CreateProtector(
            "Microsoft.AspNetCore.Authentication.Cookies.CookieAuthenticationMiddleware", 
            "Cookies", 
            "v2"));
});
```

---

<span data-ttu-id="26ed9-137">Při používání souborů cookie přímo:</span><span class="sxs-lookup"><span data-stu-id="26ed9-137">When using cookies directly:</span></span>

# <a name="aspnet-core-2xtabaspnetcore2x"></a>[<span data-ttu-id="26ed9-138">ASP.NET základní 2.x</span><span class="sxs-lookup"><span data-stu-id="26ed9-138">ASP.NET Core 2.x</span></span>](#tab/aspnetcore2x)

[!code-csharp[](cookie-sharing/sample/CookieAuth.Core/Startup.cs?name=snippet1)]

<span data-ttu-id="26ed9-139">Najdete v článku *CookieAuth.Core* v projektu [ukázkový kód](https://github.com/aspnet/Docs/tree/master/aspnetcore/security/cookie-sharing/sample/) ([stažení](xref:tutorials/index#how-to-download-a-sample)).</span><span class="sxs-lookup"><span data-stu-id="26ed9-139">See the *CookieAuth.Core* project in the [sample code](https://github.com/aspnet/Docs/tree/master/aspnetcore/security/cookie-sharing/sample/) ([how to download](xref:tutorials/index#how-to-download-a-sample)).</span></span>

# <a name="aspnet-core-1xtabaspnetcore1x"></a>[<span data-ttu-id="26ed9-140">ASP.NET základní 1.x</span><span class="sxs-lookup"><span data-stu-id="26ed9-140">ASP.NET Core 1.x</span></span>](#tab/aspnetcore1x)

```csharp
app.UseCookieAuthentication(new CookieAuthenticationOptions
{
    DataProtectionProvider = 
        DataProtectionProvider.Create(
            new DirectoryInfo(@"PATH_TO_KEY_RING_FOLDER"))
});
```

---

## <a name="encrypting-data-protection-keys-at-rest"></a><span data-ttu-id="26ed9-141">Šifrování klíče ochrany dat v klidovém stavu</span><span class="sxs-lookup"><span data-stu-id="26ed9-141">Encrypting data protection keys at rest</span></span>

<span data-ttu-id="26ed9-142">Pro nasazení v produkčním prostředí, nakonfigurovat `DataProtectionProvider` k šifrování klíče v klidovém stavu pomocí rozhraní DPAPI nebo certifikátu x 509.</span><span class="sxs-lookup"><span data-stu-id="26ed9-142">For production deployments, configure the `DataProtectionProvider` to encrypt keys at rest with DPAPI or an X509Certificate.</span></span> <span data-ttu-id="26ed9-143">V tématu [klíč šifrování na Rest](xref:security/data-protection/implementation/key-encryption-at-rest) Další informace.</span><span class="sxs-lookup"><span data-stu-id="26ed9-143">See [Key Encryption At Rest](xref:security/data-protection/implementation/key-encryption-at-rest) for more information.</span></span>

# <a name="aspnet-core-2xtabaspnetcore2x"></a>[<span data-ttu-id="26ed9-144">ASP.NET základní 2.x</span><span class="sxs-lookup"><span data-stu-id="26ed9-144">ASP.NET Core 2.x</span></span>](#tab/aspnetcore2x)

```csharp
services.AddDataProtection()
    .ProtectKeysWithCertificate("thumbprint");
```

# <a name="aspnet-core-1xtabaspnetcore1x"></a>[<span data-ttu-id="26ed9-145">ASP.NET základní 1.x</span><span class="sxs-lookup"><span data-stu-id="26ed9-145">ASP.NET Core 1.x</span></span>](#tab/aspnetcore1x)

```csharp
app.UseCookieAuthentication(new CookieAuthenticationOptions
{
    DataProtectionProvider = DataProtectionProvider.Create(
        new DirectoryInfo(@"PATH_TO_KEY_RING"),
        configure =>
        {
            configure.ProtectKeysWithCertificate("thumbprint");
        })
});
```

---

## <a name="sharing-authentication-cookies-between-aspnet-4x-and-aspnet-core-apps"></a><span data-ttu-id="26ed9-146">Sdílení souborů cookie ověřování mezi ASP.NET 4.x a ASP.NET Core aplikace</span><span class="sxs-lookup"><span data-stu-id="26ed9-146">Sharing authentication cookies between ASP.NET 4.x and ASP.NET Core apps</span></span>

<span data-ttu-id="26ed9-147">Aplikace ASP.NET 4.x, která používá Katana middleware ověřování souborů cookie může být nakonfigurováno pro generování ověřovací soubory cookie, které jsou kompatibilní s middleware ověřování souborů cookie ASP.NET Core.</span><span class="sxs-lookup"><span data-stu-id="26ed9-147">ASP.NET 4.x apps which use Katana cookie authentication middleware can be configured to generate authentication cookies that are compatible with the ASP.NET Core cookie authentication middleware.</span></span> <span data-ttu-id="26ed9-148">To umožňuje piecemeal upgrade velké lokality jednotlivými aplikacemi při současném poskytování smooth přihlašováním celém webu.</span><span class="sxs-lookup"><span data-stu-id="26ed9-148">This allows upgrading a large site's individual apps piecemeal while providing a smooth SSO experience across the site.</span></span>

> [!TIP]
> <span data-ttu-id="26ed9-149">Pokud aplikace používá Katana middleware ověřování souborů cookie, zavolá `UseCookieAuthentication` v projektu *Startup.Auth.cs* souboru.</span><span class="sxs-lookup"><span data-stu-id="26ed9-149">When an app uses Katana cookie authentication middleware, it calls `UseCookieAuthentication` in the project's *Startup.Auth.cs* file.</span></span> <span data-ttu-id="26ed9-150">Projekty ASP.NET 4.x webové aplikace vytvořené pomocí sady Visual Studio 2013 a později middleware ověřování souborů cookie Katana ve výchozím nastavení.</span><span class="sxs-lookup"><span data-stu-id="26ed9-150">ASP.NET 4.x web app projects created with Visual Studio 2013 and later use the Katana cookie authentication middleware by default.</span></span>

> [!NOTE]
> <span data-ttu-id="26ed9-151">Aplikace ASP.NET 4.x musí mít jako cíl rozhraní .NET Framework 4.5.1 nebo novější.</span><span class="sxs-lookup"><span data-stu-id="26ed9-151">An ASP.NET 4.x app must target .NET Framework 4.5.1 or higher.</span></span> <span data-ttu-id="26ed9-152">Potřebné balíčky NuGet, jinak hodnota nepodaří nainstalovat.</span><span class="sxs-lookup"><span data-stu-id="26ed9-152">Otherwise, the necessary NuGet packages fail to install.</span></span>

<span data-ttu-id="26ed9-153">Sdílet soubory cookie pro ověřování mezi ASP.NET 4.x aplikace a aplikace ASP.NET Core, konfiguraci aplikace ASP.NET Core, jak je uvedeno výše a potom nakonfigurujte aplikace ASP.NET 4.x podle následujících kroků.</span><span class="sxs-lookup"><span data-stu-id="26ed9-153">To share authentication cookies among ASP.NET 4.x apps and ASP.NET Core apps, configure the ASP.NET Core app as stated above, then configure the ASP.NET 4.x apps by following the steps below.</span></span>

1. <span data-ttu-id="26ed9-154">Nainstalujte balíček [Microsoft.Owin.Security.Interop](https://www.nuget.org/packages/Microsoft.Owin.Security.Interop/) do každé aplikace ASP.NET 4.x.</span><span class="sxs-lookup"><span data-stu-id="26ed9-154">Install the package [Microsoft.Owin.Security.Interop](https://www.nuget.org/packages/Microsoft.Owin.Security.Interop/) into each ASP.NET 4.x app.</span></span>

2. <span data-ttu-id="26ed9-155">V *Startup.Auth.cs*, vyhledejte volání `UseCookieAuthentication` a upravit ho následujícím způsobem.</span><span class="sxs-lookup"><span data-stu-id="26ed9-155">In *Startup.Auth.cs*, locate the call to `UseCookieAuthentication` and modify it as follows.</span></span> <span data-ttu-id="26ed9-156">Změňte název souboru cookie, aby odpovídal názvu používané middleware ověřování souborů cookie ASP.NET Core.</span><span class="sxs-lookup"><span data-stu-id="26ed9-156">Change the cookie name to match the name used by the ASP.NET Core cookie authentication middleware.</span></span> <span data-ttu-id="26ed9-157">Zadejte instanci `DataProtectionProvider` inicializována tak, aby společné umístění dat ochrany úložiště klíčů.</span><span class="sxs-lookup"><span data-stu-id="26ed9-157">Provide an instance of a `DataProtectionProvider` initialized to the common data protection key storage location.</span></span>

# <a name="aspnet-core-2xtabaspnetcore2x"></a>[<span data-ttu-id="26ed9-158">ASP.NET základní 2.x</span><span class="sxs-lookup"><span data-stu-id="26ed9-158">ASP.NET Core 2.x</span></span>](#tab/aspnetcore2x)

[!code-csharp[](cookie-sharing/sample/CookieAuthWithIdentity.NETFramework/CookieAuthWithIdentity.NETFramework/App_Start/Startup.Auth.cs?name=snippet1)]

<span data-ttu-id="26ed9-159">Najdete v článku *CookieAuthWithIdentity.NETFramework* v projektu [ukázkový kód](https://github.com/aspnet/Docs/tree/master/aspnetcore/security/cookie-sharing/sample/) ([stažení](xref:tutorials/index#how-to-download-a-sample)).</span><span class="sxs-lookup"><span data-stu-id="26ed9-159">See the *CookieAuthWithIdentity.NETFramework* project in the [sample code](https://github.com/aspnet/Docs/tree/master/aspnetcore/security/cookie-sharing/sample/) ([how to download](xref:tutorials/index#how-to-download-a-sample)).</span></span>

<span data-ttu-id="26ed9-160">Při generování identitu uživatele, typ ověřování se musí shodovat s typem definovaným v `AuthenticationType` s `UseCookieAuthentication`.</span><span class="sxs-lookup"><span data-stu-id="26ed9-160">When generating a user identity, the authentication type must match the type defined in `AuthenticationType` set with `UseCookieAuthentication`.</span></span>

<span data-ttu-id="26ed9-161">*Models/IdentityModels.cs*:</span><span class="sxs-lookup"><span data-stu-id="26ed9-161">*Models/IdentityModels.cs*:</span></span>

[!code-csharp[](cookie-sharing/sample/CookieAuthWithIdentity.NETFramework/CookieAuthWithIdentity.NETFramework/Models/IdentityModels.cs?name=snippet1)]

# <a name="aspnet-core-1xtabaspnetcore1x"></a>[<span data-ttu-id="26ed9-162">ASP.NET základní 1.x</span><span class="sxs-lookup"><span data-stu-id="26ed9-162">ASP.NET Core 1.x</span></span>](#tab/aspnetcore1x)

<span data-ttu-id="26ed9-163">Nastavte `CookieManager` zprostředkovatele komunikace s objekty `ChunkingCookieManager` tak Formát bloku dat je kompatibilní.</span><span class="sxs-lookup"><span data-stu-id="26ed9-163">Set the `CookieManager` to interop `ChunkingCookieManager` so the chunking format is compatible.</span></span>

```csharp
app.UseCookieAuthentication(new CookieAuthenticationOptions
{
    AuthenticationType = DefaultAuthenticationTypes.ApplicationCookie,
    CookieName = ".AspNetCore.Cookies",
    // CookieName = ".AspNetCore.ApplicationCookie", (if using ASP.NET Identity)
    // CookiePath = "...", (if necessary)
    // ...
    TicketDataFormat = new AspNetTicketDataFormat(
        new DataProtectorShim(
            DataProtectionProvider.Create(
                new DirectoryInfo(@"PATH_TO_KEY_RING_FOLDER"))
            .CreateProtector(
                "Microsoft.AspNetCore.Authentication.Cookies.CookieAuthenticationMiddleware",
                "Cookies", 
                "v2"))),
    CookieManager = new ChunkingCookieManager()
});
```

---

## <a name="use-a-common-user-database"></a><span data-ttu-id="26ed9-164">Použít společnou databázi uživatele</span><span class="sxs-lookup"><span data-stu-id="26ed9-164">Use a common user database</span></span>

<span data-ttu-id="26ed9-165">Potvrďte, že je na stejné uživatele databáze odkazoval systém identit pro každou aplikaci.</span><span class="sxs-lookup"><span data-stu-id="26ed9-165">Confirm that the identity system for each app is pointed at the same user database.</span></span> <span data-ttu-id="26ed9-166">Systém identit, jinak hodnota vytvoří chyby za běhu, když pokusí se porovnat informace v souboru cookie pro ověřování proti informací ve své databázi.</span><span class="sxs-lookup"><span data-stu-id="26ed9-166">Otherwise, the identity system produces failures at runtime when it attempts to match the information in the authentication cookie against the information in its database.</span></span>