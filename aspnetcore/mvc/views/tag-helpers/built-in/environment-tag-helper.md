---
title: Pomocná značka prostředí v ASP.NET Core
author: pkellner
description: ASP.NET Core pomocná podpora značek prostředí, včetně všech vlastností
ms.author: riande
ms.custom: mvc
ms.date: 10/10/2018
uid: mvc/views/tag-helpers/builtin-th/environment-tag-helper
ms.openlocfilehash: 308e7db47104ebd4d6bb8d08c64f14bbd118898b
ms.sourcegitcommit: 9a129f5f3e31cc449742b164d5004894bfca90aa
ms.translationtype: MT
ms.contentlocale: cs-CZ
ms.lasthandoff: 03/06/2020
ms.locfileid: "78663987"
---
# <a name="environment-tag-helper-in-aspnet-core"></a>Pomocná značka prostředí v ASP.NET Core

Od [Petra Kellner](https://peterkellner.net) a [Hisham bin Ateya](https://twitter.com/hishambinateya)

Pomocník pro tag prostředí podmíněně vykreslí svůj obsah na základě aktuálního [hostitelského prostředí](xref:fundamentals/environments). Jeden atribut pomocníka značek prostředí, `names`, je čárkami oddělený seznam názvů prostředí. Pokud se některý z poskytnutých názvů prostředí shoduje s aktuálním prostředím, zobrazí se obsah přiložený k obsahu.

Přehled pomocníků značek naleznete v tématu <xref:mvc/views/tag-helpers/intro>.

## <a name="environment-tag-helper-attributes"></a>Pomocné atributy značek prostředí

### <a name="names"></a>názvy

`names` přijímá jeden název hostitelského prostředí nebo čárkami oddělený seznam názvů hostitelských prostředí, které aktivují vykreslování vloženého obsahu.

Hodnoty prostředí jsou porovnány s aktuální hodnotou vrácenou funkcí [IHostingEnvironment. Environment](xref:Microsoft.AspNetCore.Hosting.IHostingEnvironment.EnvironmentName*). Porovnání ignoruje velikost písmen.

V následujícím příkladu je použita pomocná pomůcka značek prostředí. Obsah se vykreslí, pokud je hostující prostředí pracovní nebo produkční:

```cshtml
<environment names="Staging,Production">
    <strong>HostingEnvironment.EnvironmentName is Staging or Production</strong>
</environment>
```

::: moniker range=">= aspnetcore-2.0"

## <a name="include-and-exclude-attributes"></a>zahrnutí a vyloučení atributů

`include` & `exclude` řízení atributů vykreslování obsahu na základě zahrnutých nebo vyloučených názvů hostitelských prostředí.

### <a name="include"></a>include

Vlastnost `include` vykazuje podobné chování atributu `names`. Prostředí uvedené v hodnotě atributu `include` se musí shodovat s hostitelským prostředím aplikace ([IHostingEnvironment. Environment](xref:Microsoft.AspNetCore.Hosting.IHostingEnvironment.EnvironmentName*)), aby se vygeneroval obsah značky `<environment>`.

```cshtml
<environment include="Staging,Production">
    <strong>HostingEnvironment.EnvironmentName is Staging or Production</strong>
</environment>
```

### <a name="exclude"></a>exclude

Na rozdíl od atributu `include` je obsah značky `<environment>` vykreslen v případě, že se hostitelské prostředí neshoduje s prostředím uvedeným v hodnotě atributu `exclude`.

```cshtml
<environment exclude="Development">
    <strong>HostingEnvironment.EnvironmentName is not Development</strong>
</environment>
```

::: moniker-end

## <a name="additional-resources"></a>Další zdroje

* <xref:fundamentals/environments>
