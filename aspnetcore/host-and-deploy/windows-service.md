---
title: "Hostitele ve službě Windows"
author: tdykstra
description: "Zjistěte, jak hostovat aplikace ASP.NET Core ve službě Windows."
manager: wpickett
ms.author: tdykstra
ms.custom: mvc
ms.date: 01/30/2018
ms.prod: aspnet-core
ms.technology: aspnet
ms.topic: article
uid: host-and-deploy/windows-service
ms.openlocfilehash: c14a1f62bce4d06be3b8e6356f45cd5e330a0751
ms.sourcegitcommit: f2a11a89037471a77ad68a67533754b7bb8303e2
ms.translationtype: MT
ms.contentlocale: cs-CZ
ms.lasthandoff: 02/01/2018
---
# <a name="host-an-aspnet-core-app-in-a-windows-service"></a>Hostování aplikace ASP.NET Core ve službě Windows

podle [tní Dykstra](https://github.com/tdykstra)

Doporučeným způsobem, jak hostovat aplikace ASP.NET Core v systému Windows bez použití služby IIS je chcete-li používat [služba systému Windows](/dotnet/framework/windows-services/introduction-to-windows-service-applications). Pokud je hostováno jako služby systému Windows, aplikace může automaticky spustit po restartování počítače a dojde k chybě bez nutnosti lidského zásahu.

[Zobrazit nebo stáhnout ukázkový kód](https://github.com/aspnet/Docs/tree/master/aspnetcore/host-and-deploy/windows-service/sample) ([stažení](xref:tutorials/index#how-to-download-a-sample)). Pokyny o tom, jak spuštění ukázkové aplikace, naleznete v části tohoto příkladu *README.md* souboru.

## <a name="prerequisites"></a>Požadavky

* Aplikace musíte spustit na modul runtime rozhraní .NET Framework. V *.csproj* souboru, zadejte odpovídající hodnoty pro [TargetFramework](/nuget/schema/target-frameworks) a [RuntimeIdentifier](/dotnet/articles/core/rid-catalog). Tady je příklad:

  [!code-xml[](windows-service/sample/AspNetCoreService.csproj?range=3-6)]

  Při vytváření projektu v sadě Visual Studio, použijte **aplikace ASP.NET Core (rozhraní .NET Framework)** šablony.

* Pokud aplikace přijímá požadavky od Internetu (nikoli pouze z interní sítě), musí používat [HTTP.sys](xref:fundamentals/servers/httpsys) webový server (dříve označovaný jako [WebListener](xref:fundamentals/servers/weblistener) pro aplikace ASP.NET Core 1.x) namísto [Kestrel](xref:fundamentals/servers/kestrel). IIS se doporučuje pro použití jako reverzní proxy server s Kestrel pro nasazení okraj. Další informace najdete v tématu [použití Kestrel s reverzní proxy server](xref:fundamentals/servers/kestrel#when-to-use-kestrel-with-a-reverse-proxy).

## <a name="getting-started"></a>Začínáme

Tato část popisuje minimální změny, které jsou nezbytné k nastavení existujícího projektu ASP.NET Core ke spuštění ve službě.

1. Nainstalujte balíček NuGet [Microsoft.AspNetCore.Hosting.WindowsServices](https://www.nuget.org/packages/Microsoft.AspNetCore.Hosting.WindowsServices/).

1. Proveďte následující změny v `Program.Main`:
  
   * Volání `host.RunAsService` místo `host.Run`.
  
   * Pokud kód volá `UseContentRoot`, použijte cestu k umístění pro publikování místo `Directory.GetCurrentDirectory()`.

   # <a name="aspnet-core-2xtabaspnetcore2x"></a>[ASP.NET základní 2.x](#tab/aspnetcore2x)

   [!code-csharp[](windows-service/sample/Program.cs?name=ServiceOnly&highlight=3-4,7,12)]

   # <a name="aspnet-core-1xtabaspnetcore1x"></a>[ASP.NET základní 1.x](#tab/aspnetcore1x)

   [!code-csharp[](windows-service/sample_snapshot/Program.cs?name=ServiceOnly&highlight=3-4,8,14)]

   ---

1. Publikujte aplikaci do složky. Použití [dotnet publikování](/dotnet/articles/core/tools/dotnet-publish) nebo [profil publikování se Visual Studio](xref:host-and-deploy/visual-studio-publish-profiles) který publikuje do složky.

1. Otestujte vytváření a spouštění služby.

   Otevřete příkazové okno s oprávněním správce k použití [sc.exe](https://technet.microsoft.com/library/bb490995) nástroj příkazového řádku k vytvoření a spuštění služby. Pokud je služba Moje_služba, publikovány do `c:\svc`, a s názvem AspNetCoreService, jsou příkazy:

   ```console
   sc create MyService binPath="c:\svc\aspnetcoreservice.exe"
   sc start MyService
   ```

   `binPath` Hodnota je cesta ke spustitelnému souboru aplikace, která zahrnuje název spustitelného souboru.

   ![Okno konzoly vytvořte a spusťte příklad](windows-service/_static/create-start.png)

   Když se tyto příkazy dokončit, přejděte do stejnou cestu jako při spuštění jako konzolové aplikace (ve výchozím nastavení, `http://localhost:5000`):

   ![Spuštěná ve službě](windows-service/_static/running-in-service.png)

## <a name="provide-a-way-to-run-outside-of-a-service"></a>Zadejte způsob, jak spustit mimo službu

Je snazší testování a ladění při spuštění mimo službu, takže se obvykle přidávat kód, který volá `RunAsService` pouze za určitých podmínek. Například můžete spouštět aplikace jako konzolové aplikace s `--console` argument příkazového řádku nebo pokud je připojen ladicí program:

# <a name="aspnet-core-2xtabaspnetcore2x"></a>[ASP.NET základní 2.x](#tab/aspnetcore2x)

[!code-csharp[](windows-service/sample/Program.cs?name=ServiceOrConsole)]

# <a name="aspnet-core-1xtabaspnetcore1x"></a>[ASP.NET základní 1.x](#tab/aspnetcore1x)

[!code-csharp[](windows-service/sample_snapshot/Program.cs?name=ServiceOrConsole)]

---

## <a name="handle-stopping-and-starting-events"></a>Zpracování ukončení a spuštění události

Pro zpracování `OnStarting`, `OnStarted`, a `OnStopping` události, proveďte následující další změny:

1. Vytvořte třídu, která je odvozena z `WebHostService`:

   [!code-csharp[](windows-service/sample/CustomWebHostService.cs?name=NoLogging)]

1. Vytvoření metody rozšíření pro `IWebHost` , předá vlastní `WebHostService` k `ServiceBase.Run`:

   [!code-csharp[](windows-service/sample/WebHostServiceExtensions.cs?name=ExtensionsClass)]

1. V `Program.Main`, zavolejte metodu nové rozšíření, `RunAsCustomService`, místo `RunAsService`:

   # <a name="aspnet-core-2xtabaspnetcore2x"></a>[ASP.NET základní 2.x](#tab/aspnetcore2x)

   [!code-csharp[](windows-service/sample/Program.cs?name=HandleStopStart&highlight=24)]

   # <a name="aspnet-core-1xtabaspnetcore1x"></a>[ASP.NET základní 1.x](#tab/aspnetcore1x)

   [!code-csharp[](windows-service/sample_snapshot/Program.cs?name=HandleStopStart&highlight=26)]

   ---

Pokud vlastní `WebHostService` kódu vyžaduje službu z vkládání závislostí (například protokolovač), získat ze `Services` vlastnost `IWebHost`:

[!code-csharp[](windows-service/sample/CustomWebHostService.cs?name=Logging&highlight=7)]

## <a name="acknowledgments"></a>Potvrzování

Tento článek byl napsán pomocí publikovaných zdrojů:

* [Hostování ASP.NET Core jako služby systému Windows](https://stackoverflow.com/questions/37346383/hosting-asp-net-core-as-windows-service/37464074)
* [Jak hostovat vaše základní technologie ASP.NET ve službě Windows](https://dotnetthoughts.net/how-to-host-your-aspnet-core-in-a-windows-service/)