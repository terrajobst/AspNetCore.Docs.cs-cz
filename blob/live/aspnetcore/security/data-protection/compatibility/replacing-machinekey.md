---
title: "Nahrazení `<machineKey>` technologie ASP.NET"
author: rick-anderson
description: "Nahrazení `<machineKey>` technologie ASP.NET"
ms.author: riande
manager: wpickett
ms.date: 10/14/2016
ms.topic: article
ms.technology: aspnet
ms.prod: asp.net-core
uid: security/data-protection/compatibility/replacing-machinekey
ms.openlocfilehash: c151a48b22d6d0d6e0a8fa88a9868767d5897b2d
ms.sourcegitcommit: 3e303620a125325bb9abd4b2d315c106fb8c47fd
ms.translationtype: MT
ms.contentlocale: cs-CZ
ms.lasthandoff: 01/19/2018
---
# <a name="replacing-machinekey-in-aspnet"></a><span data-ttu-id="068d0-103">Nahrazení `<machineKey>` technologie ASP.NET</span><span class="sxs-lookup"><span data-stu-id="068d0-103">Replacing `<machineKey>` in ASP.NET</span></span>

<a name="compatibility-replacing-machinekey"></a>

<span data-ttu-id="068d0-104">Implementace `<machineKey>` element technologie ASP.NET [je replaceable](https://blogs.msdn.microsoft.com/webdev/2012/10/23/cryptographic-improvements-in-asp-net-4-5-pt-2/).</span><span class="sxs-lookup"><span data-stu-id="068d0-104">The implementation of the `<machineKey>` element in ASP.NET [is replaceable](https://blogs.msdn.microsoft.com/webdev/2012/10/23/cryptographic-improvements-in-asp-net-4-5-pt-2/).</span></span> <span data-ttu-id="068d0-105">To umožňuje většinu volání kryptografické rutiny ASP.NET prostřednictvím ochranný mechanismus nahrazení data, včetně nového systému ochrany dat směrování.</span><span class="sxs-lookup"><span data-stu-id="068d0-105">This allows most calls to ASP.NET cryptographic routines to be routed through a replacement data protection mechanism, including the new data protection system.</span></span>

## <a name="package-installation"></a><span data-ttu-id="068d0-106">Instalace balíčku</span><span class="sxs-lookup"><span data-stu-id="068d0-106">Package installation</span></span>

> [!NOTE]
> <span data-ttu-id="068d0-107">Nový systém ochrany dat pouze lze nainstalovat do stávající aplikaci ASP.NET cílení na rozhraní .NET 4.5.1 nebo vyšší.</span><span class="sxs-lookup"><span data-stu-id="068d0-107">The new data protection system can only be installed into an existing ASP.NET application targeting .NET 4.5.1 or higher.</span></span> <span data-ttu-id="068d0-108">Instalace se nezdaří, pokud aplikace cílí .NET 4.5 nebo nižší.</span><span class="sxs-lookup"><span data-stu-id="068d0-108">Installation will fail if the application targets .NET 4.5 or lower.</span></span>

<span data-ttu-id="068d0-109">Instalace nového systému ochrany dat do existujícího projektu ASP.NET 4.5.1+, nainstalujte balíček Microsoft.AspNetCore.DataProtection.SystemWeb.</span><span class="sxs-lookup"><span data-stu-id="068d0-109">To install the new data protection system into an existing ASP.NET 4.5.1+ project, install the package Microsoft.AspNetCore.DataProtection.SystemWeb.</span></span> <span data-ttu-id="068d0-110">To vytvoří instanci systému ochrany dat pomocí [výchozí konfigurace](xref:security/data-protection/configuration/default-settings) nastavení.</span><span class="sxs-lookup"><span data-stu-id="068d0-110">This will instantiate the data protection system using the [default configuration](xref:security/data-protection/configuration/default-settings) settings.</span></span>

<span data-ttu-id="068d0-111">Při instalaci balíčku se vloží řádek do *Web.config* použít pro technologii ASP.NET, která oznamuje [nejvíce kryptografické operace](https://blogs.msdn.microsoft.com/webdev/2012/10/23/cryptographic-improvements-in-asp-net-4-5-pt-2/), včetně ověřování pomocí formulářů, zobrazení stavu a volání MachineKey.Protect.</span><span class="sxs-lookup"><span data-stu-id="068d0-111">When you install the package, it inserts a line into *Web.config* that tells ASP.NET to use it for [most cryptographic operations](https://blogs.msdn.microsoft.com/webdev/2012/10/23/cryptographic-improvements-in-asp-net-4-5-pt-2/), including forms authentication, view state, and calls to MachineKey.Protect.</span></span> <span data-ttu-id="068d0-112">Řádek, který je vložen přečte následujícím způsobem.</span><span class="sxs-lookup"><span data-stu-id="068d0-112">The line that's inserted reads as follows.</span></span>

```xml
<machineKey compatibilityMode="Framework45" dataProtectorType="..." />
```

>[!TIP]
> <span data-ttu-id="068d0-113">Můžete zjistit, pokud je nový systém ochrany dat active zkontrolováním pole jako `__VIEWSTATE`, který by měl začínat obráceným "CfDJ8" jako v příkladu níže.</span><span class="sxs-lookup"><span data-stu-id="068d0-113">You can tell if the new data protection system is active by inspecting fields like `__VIEWSTATE`, which should begin with "CfDJ8" as in the example below.</span></span> <span data-ttu-id="068d0-114">"CfDJ8" je base64 reprezentace záhlaví magic "09 F0 C9 F0", který identifikuje chráněného systémem ochrany dat datové části.</span><span class="sxs-lookup"><span data-stu-id="068d0-114">"CfDJ8" is the base64 representation of the magic "09 F0 C9 F0" header that identifies a payload protected by the data protection system.</span></span>

```html
<input type="hidden" name="__VIEWSTATE" id="__VIEWSTATE" value="CfDJ8AWPr2EQPTBGs3L2GCZOpk..." />
```

## <a name="package-configuration"></a><span data-ttu-id="068d0-115">Konfigurace balíčku</span><span class="sxs-lookup"><span data-stu-id="068d0-115">Package configuration</span></span>

<span data-ttu-id="068d0-116">Vytvoření instance systému ochrany dat s výchozí konfigurací nastavení nula.</span><span class="sxs-lookup"><span data-stu-id="068d0-116">The data protection system is instantiated with a default zero-setup configuration.</span></span> <span data-ttu-id="068d0-117">Ale vzhledem k tomu, že ve výchozím nastavení jsou trvalé klíče do místního systému souborů, to nebude fungovat pro aplikace, které jsou nasazené ve farmě.</span><span class="sxs-lookup"><span data-stu-id="068d0-117">However, since by default keys are persisted to the local file system, this won't work for applications which are deployed in a farm.</span></span> <span data-ttu-id="068d0-118">To vyřešit, můžete zadat konfiguraci vytvořením typu které podtřídy DataProtectionStartup a přepíše jeho ConfigureServices metoda.</span><span class="sxs-lookup"><span data-stu-id="068d0-118">To resolve this, you can provide configuration by creating a type which subclasses DataProtectionStartup and overrides its ConfigureServices method.</span></span>

<span data-ttu-id="068d0-119">Dole je příklad typ spuštění ochrany vlastních dat, který nakonfigurovaný, kde jsou klíče nastavené jako trvalé i jak jste zašifrovaná přinejmenším.</span><span class="sxs-lookup"><span data-stu-id="068d0-119">Below is an example of a custom data protection startup type which configured both where keys are persisted and how they're encrypted at rest.</span></span> <span data-ttu-id="068d0-120">Potlačí výchozí zásady izolace aplikací také tím, že poskytuje svůj vlastní název aplikace.</span><span class="sxs-lookup"><span data-stu-id="068d0-120">It also overrides the default app isolation policy by providing its own application name.</span></span>

```csharp
using System;
using System.IO;
using Microsoft.AspNetCore.DataProtection;
using Microsoft.AspNetCore.DataProtection.SystemWeb;
using Microsoft.Extensions.DependencyInjection;

namespace DataProtectionDemo
{
    public class MyDataProtectionStartup : DataProtectionStartup
    {
        public override void ConfigureServices(IServiceCollection services)
        {
            services.AddDataProtection()
                .SetApplicationName("my-app")
                .PersistKeysToFileSystem(new DirectoryInfo(@"\\server\share\myapp-keys\"))
                .ProtectKeysWithCertificate("thumbprint");
        }
    }
}
```

>[!TIP]
> <span data-ttu-id="068d0-121">Můžete také použít `<machineKey applicationName="my-app" ... />` místo explicitní volání SetApplicationName.</span><span class="sxs-lookup"><span data-stu-id="068d0-121">You can also use `<machineKey applicationName="my-app" ... />` in place of an explicit call to SetApplicationName.</span></span> <span data-ttu-id="068d0-122">Jedná se o usnadnění práce mechanismus, aby se zabránilo vynucení vývojáře pro vytvoření typu odvozeného DataProtectionStartup, pokud se všechny chtěli nakonfigurovat nastavení název aplikace.</span><span class="sxs-lookup"><span data-stu-id="068d0-122">This is a convenience mechanism to avoid forcing the developer to create a DataProtectionStartup-derived type if all they wanted to configure was setting the application name.</span></span>

<span data-ttu-id="068d0-123">Pokud chcete povolit tento vlastní konfigurace, přejděte zpět do souboru Web.config a vyhledejte `<appSettings>` element, který nainstalujte balíček přidat do konfiguračního souboru.</span><span class="sxs-lookup"><span data-stu-id="068d0-123">To enable this custom configuration, go back to Web.config and look for the `<appSettings>` element that the package install added to the config file.</span></span> <span data-ttu-id="068d0-124">Ho bude vypadat jako následující kód:</span><span class="sxs-lookup"><span data-stu-id="068d0-124">It will look like the following markup:</span></span>

```xml
<appSettings>
  <!--
  If you want to customize the behavior of the ASP.NET Core Data Protection stack, set the
  "aspnet:dataProtectionStartupType" switch below to be the fully-qualified name of a
  type which subclasses Microsoft.AspNetCore.DataProtection.SystemWeb.DataProtectionStartup.
  -->
  <add key="aspnet:dataProtectionStartupType" value="" />
</appSettings>
```

<span data-ttu-id="068d0-125">Zadejte prázdnou hodnotu s názvem kvalifikovaný sestavení DataProtectionStartup odvozeného typu, který jste právě vytvořili.</span><span class="sxs-lookup"><span data-stu-id="068d0-125">Fill in the blank value with the assembly-qualified name of the DataProtectionStartup-derived type you just created.</span></span> <span data-ttu-id="068d0-126">Pokud název aplikace je DataProtectionDemo, to bude vypadat níže.</span><span class="sxs-lookup"><span data-stu-id="068d0-126">If the name of the application is DataProtectionDemo, this would look like the below.</span></span>

```xml
<add key="aspnet:dataProtectionStartupType"
     value="DataProtectionDemo.MyDataProtectionStartup, DataProtectionDemo" />
```

<span data-ttu-id="068d0-127">Nově nakonfigurovaný data ochrany systému je nyní připravena k použití uvnitř aplikace.</span><span class="sxs-lookup"><span data-stu-id="068d0-127">The newly-configured data protection system is now ready for use inside the application.</span></span>