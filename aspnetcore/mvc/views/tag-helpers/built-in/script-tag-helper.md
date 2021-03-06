---
title: Pomocník značek Script v ASP.NET Core
author: rick-anderson
ms.author: riande
description: Seznamte se s atributy pomocníka značky skriptu ASP.NET Core a role, které každý atribut hraje při rozšiřování chování značky skriptu HTML.
ms.custom: mvc
ms.date: 12/02/2019
uid: mvc/views/tag-helpers/builtin-th/script-tag-helper
ms.openlocfilehash: a037abb6a454e6d06305e7d7f6ecad0c2a0ca717
ms.sourcegitcommit: 9a129f5f3e31cc449742b164d5004894bfca90aa
ms.translationtype: MT
ms.contentlocale: cs-CZ
ms.lasthandoff: 03/06/2020
ms.locfileid: "78659836"
---
# <a name="script-tag-helper-in-aspnet-core"></a>Pomocník značek Script v ASP.NET Core

Autor: [Rick Anderson](https://twitter.com/RickAndMSFT)

[Pomocník značek Script](xref:Microsoft.AspNetCore.Mvc.TagHelpers.ScriptTagHelper) vygeneruje odkaz na primární soubor nebo soubor skriptu. Obvykle je soubor primárního skriptu na [Content Delivery Network](/office365/enterprise/content-delivery-networks#what-exactly-is-a-cdn) (CDN).

[!INCLUDE[](~/includes/cdn.md)]

Pomocná značka skriptu umožňuje zadat síť CDN pro soubor skriptu a zálohu, pokud CDN není k dispozici. Pomocník značek Script poskytuje výkonnou výhodu sítě CDN s odolností místního hostování.

Následující kód Razor ukazuje `script` element s Fallback:

```html
<script src="https://ajax.aspnetcdn.com/ajax/jquery/jquery-3.3.1.min.js"
        asp-fallback-src="~/lib/jquery/dist/jquery.min.js"
        asp-fallback-test="window.jQuery"
        crossorigin="anonymous"
        integrity="sha384-tsQFqpEReu7ZLhBV2VZlAu7zcOV+rXbYlF2cqB8txI/8aZajjp4Bqd+V6D5IgvKT">
</script>
```

Nepoužívejte atribut [odložení](https://developer.mozilla.org/docs/Web/HTML/Element/script) elementu `<script>` pro odložení načtení skriptu CDN. Pomocník značek Script vykresluje JavaScript, který okamžitě spustí výraz [ASP-Fallback-test](#asp-fallback-test) . Pokud je načítání skriptu CDN odložené, výraz se nezdařil.

## <a name="commonly-used-script-tag-helper-attributes"></a>Běžně používané pomocné atributy značek skriptů

Viz [Pomocník značek Script](xref:Microsoft.AspNetCore.Mvc.TagHelpers.ScriptTagHelper) pro všechny pomocné atributy, vlastnosti a metody značky skriptu.

### <a name="asp-fallback-test"></a>ASP – Fallback-test

Metoda skriptu definovaná v primárním skriptu pro použití pro záložní test. Další informace naleznete v tématu <xref:Microsoft.AspNetCore.Mvc.TagHelpers.ScriptTagHelper.FallbackTestExpression>.

### <a name="asp-fallback-src"></a>ASP – Fallback – src

Adresa URL značky skriptu, na kterou se má přejít v případě, že primární z nich selhala Další informace naleznete v tématu <xref:Microsoft.AspNetCore.Mvc.TagHelpers.ScriptTagHelper.FallbackSrc>.

## <a name="additional-resources"></a>Další zdroje

* <xref:mvc/views/tag-helpers/intro>
* <xref:mvc/controllers/areas>
* <xref:razor-pages/index>
* <xref:mvc/compatibility-version>
