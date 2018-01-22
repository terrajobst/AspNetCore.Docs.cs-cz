---
title: "Obrázek značky pomocná | Microsoft Docs"
author: pkellner
description: "Ukazuje, jak pracovat s pomocná značka obrázku"
ms.author: riande
manager: wpickett
ms.date: 02/14/2017
ms.topic: article
ms.technology: aspnet
ms.prod: aspnet-core
uid: mvc/views/tag-helpers/builtin-th/image-tag-helper
ms.openlocfilehash: 438c5afb96dce6d8978d26159a3b460614111988
ms.sourcegitcommit: 3e303620a125325bb9abd4b2d315c106fb8c47fd
ms.translationtype: MT
ms.contentlocale: cs-CZ
ms.lasthandoff: 01/19/2018
---
# <a name="imagetaghelper"></a><span data-ttu-id="b81e3-103">ImageTagHelper</span><span class="sxs-lookup"><span data-stu-id="b81e3-103">ImageTagHelper</span></span>

<span data-ttu-id="b81e3-104">Podle [Petr Kellner](http://peterkellner.net)</span><span class="sxs-lookup"><span data-stu-id="b81e3-104">By [Peter Kellner](http://peterkellner.net)</span></span> 

<span data-ttu-id="b81e3-105">Pomocná značka obrázku rozšiřuje `img` (`<img>`) značky.</span><span class="sxs-lookup"><span data-stu-id="b81e3-105">The Image Tag Helper enhances the `img` (`<img>`) tag.</span></span> <span data-ttu-id="b81e3-106">Vyžaduje `src` značky a taky `boolean` atribut `asp-append-version`.</span><span class="sxs-lookup"><span data-stu-id="b81e3-106">It requires a `src` tag as well as the `boolean` attribute `asp-append-version`.</span></span>

<span data-ttu-id="b81e3-107">Pokud zdroj bitové kopie (`src`) je statický soubor na webovém serveru hostitele se připojí jedinečný mezipaměti nejnovějších řetězec jako parametr dotazu na zdroj bitové kopie.</span><span class="sxs-lookup"><span data-stu-id="b81e3-107">If the image source (`src`) is a static file on the host web server, a unique cache busting string is appended as a query parameter to the image source.</span></span> <span data-ttu-id="b81e3-108">Tím se zajistí, že pokud se změní soubor na webovém serveru hostitele, adresu URL jedinečný požadavku je vygenerováno, která zahrnuje parametr aktualizované žádosti.</span><span class="sxs-lookup"><span data-stu-id="b81e3-108">This ensures that if the file on the host web server changes, a unique request URL is generated that includes the updated request parameter.</span></span> <span data-ttu-id="b81e3-109">Mezipaměť nejnovějších řetězec je jedinečnou hodnotu představující hodnotu hash souboru statické bitové kopie.</span><span class="sxs-lookup"><span data-stu-id="b81e3-109">The cache busting string is a unique value representing the hash of the static image file.</span></span>

<span data-ttu-id="b81e3-110">Pokud zdroj bitové kopie (`src`) není statický soubor (například soubor nebo vzdálenou adresou URL neexistuje na serveru), `<img>` tagu `src` atribut je generována žádné mezipaměti nejnovějších parametr řetězce dotazu.</span><span class="sxs-lookup"><span data-stu-id="b81e3-110">If the image source (`src`) isn't a static file (for example a remote URL or the file doesn't exist on the server), the `<img>` tag's `src` attribute is generated with no cache busting query string parameter.</span></span>

## <a name="image-tag-helper-attributes"></a><span data-ttu-id="b81e3-111">Atributy pomocné rutiny značka obrázku</span><span class="sxs-lookup"><span data-stu-id="b81e3-111">Image Tag Helper Attributes</span></span>


### <a name="asp-append-version"></a><span data-ttu-id="b81e3-112">ASP připojit verze</span><span class="sxs-lookup"><span data-stu-id="b81e3-112">asp-append-version</span></span>

<span data-ttu-id="b81e3-113">-Li zadána spolu s `src` atribut pomocná značka obrázku je volána.</span><span class="sxs-lookup"><span data-stu-id="b81e3-113">When specified along with a `src` attribute, the Image Tag Helper is invoked.</span></span>

<span data-ttu-id="b81e3-114">Příklad platné `img` Pomocník značky:</span><span class="sxs-lookup"><span data-stu-id="b81e3-114">An example of a valid `img` tag helper is:</span></span>

```cshtml
<img src="~/images/asplogo.png" 
    asp-append-version="true"  />
```

<span data-ttu-id="b81e3-115">Pokud se statický soubor existuje v adresáři *... Wwwroot/Images/asplogo.PNG* generovaný kód jazyka html je podobný následujícímu (hodnota hash bude jiný):</span><span class="sxs-lookup"><span data-stu-id="b81e3-115">If the static file exists in the directory *..wwwroot/images/asplogo.png* the generated html is similar to the following (the hash will be different):</span></span>

```html
<img 
    src="/images/asplogo.png?v=Kl_dqr9NVtnMdsM2MUg4qthUnWZm5T1fCEimBPWDNgM"/>
```

<span data-ttu-id="b81e3-116">Hodnota přiřazená k parametr `v` je hodnota hash souboru na disku.</span><span class="sxs-lookup"><span data-stu-id="b81e3-116">The value assigned to the parameter `v` is the hash value of the file on disk.</span></span> <span data-ttu-id="b81e3-117">Pokud webový server nemůže získat přístup pro čtení ke statickému souboru odkazováno, ne `v` parametry se přidá do `src` atribut.</span><span class="sxs-lookup"><span data-stu-id="b81e3-117">If the web server is unable to obtain read access to the static file referenced,  no `v` parameters is added to the `src` attribute.</span></span>

- - -

### <a name="src"></a><span data-ttu-id="b81e3-118">src</span><span class="sxs-lookup"><span data-stu-id="b81e3-118">src</span></span>

<span data-ttu-id="b81e3-119">Pokud chcete aktivovat pomocná značka obrázku, src atribut je požadován na `<img>` elementu.</span><span class="sxs-lookup"><span data-stu-id="b81e3-119">To activate the Image Tag Helper, the src attribute is required on the `<img>` element.</span></span> 

> [!NOTE]
> <span data-ttu-id="b81e3-120">Pomocník značka obrázku používá `Cache` zprostředkovatele na serveru místní web ukládat počítané `Sha512` daného souboru.</span><span class="sxs-lookup"><span data-stu-id="b81e3-120">The Image Tag Helper uses the `Cache` provider on the local web server to store the calculated `Sha512` of a given file.</span></span> <span data-ttu-id="b81e3-121">Pokud je soubor požádat znovu `Sha512` nemusí být přepočítána.</span><span class="sxs-lookup"><span data-stu-id="b81e3-121">If the file is requested again the `Sha512` does not need to be recalculated.</span></span> <span data-ttu-id="b81e3-122">Je mezipaměť zneplatněna pomocí souboru sledovacího procesu, který je připojen k souboru po souboru `Sha512` se počítá.</span><span class="sxs-lookup"><span data-stu-id="b81e3-122">The Cache is invalidated by a file watcher that is attached to the file when the file's `Sha512` is calculated.</span></span>

## <a name="additional-resources"></a><span data-ttu-id="b81e3-123">Další zdroje</span><span class="sxs-lookup"><span data-stu-id="b81e3-123">Additional resources</span></span>

* <xref:performance/caching/memory>