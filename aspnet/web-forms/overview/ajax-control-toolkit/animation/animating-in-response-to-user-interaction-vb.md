---
uid: web-forms/overview/ajax-control-toolkit/animation/animating-in-response-to-user-interaction-vb
title: "Animace v odezvě na interakci s uživatelem (VB) | Microsoft Docs"
author: wenz
description: "V sadě nástrojů ovládacího prvku ASP.NET AJAX ovládacího prvku animace není právě ovládací prvek ale celé rozhraní pro přidání do ovládacího prvku animace. Můžete hvězdičkami animací..."
ms.author: aspnetcontent
manager: wpickett
ms.date: 06/02/2008
ms.topic: article
ms.assetid: c8204c05-ec27-40fe-933d-88e4e727a482
ms.technology: dotnet-webforms
ms.prod: .net-framework
msc.legacyurl: /web-forms/overview/ajax-control-toolkit/animation/animating-in-response-to-user-interaction-vb
msc.type: authoredcontent
ms.openlocfilehash: 3219e9d126b3225bfc78d08fb3ac7ef4cc3dca75
ms.sourcegitcommit: 9a9483aceb34591c97451997036a9120c3fe2baf
ms.translationtype: MT
ms.contentlocale: cs-CZ
ms.lasthandoff: 11/10/2017
---
<a name="animating-in-response-to-user-interaction-vb"></a>Animace v odezvě na interakci s uživatelem (VB)
====================
podle [Christian Wenz](https://github.com/wenz)

[Stáhněte si kód](http://download.microsoft.com/download/f/9/a/f9a26acd-8df4-4484-8a18-199e4598f411/Animation6.vb.zip) nebo [stáhnout PDF](http://download.microsoft.com/download/6/7/1/6718d452-ff89-4d3f-a90e-c74ec2d636a3/animation6VB.pdf)

> V sadě nástrojů ovládacího prvku ASP.NET AJAX ovládacího prvku animace není právě ovládací prvek ale celé rozhraní pro přidání do ovládacího prvku animace. Animací spustit automaticky nebo může být aktivovány interakci s uživatelem, například kliknutím myší.


## <a name="overview"></a>Přehled

V sadě nástrojů ovládacího prvku ASP.NET AJAX ovládacího prvku animace není právě ovládací prvek ale celé rozhraní pro přidání do ovládacího prvku animace. Animací spustit automaticky nebo může být aktivovány interakci s uživatelem, například kliknutím myší.

## <a name="steps"></a>Kroky

První řadě zahrnují `ScriptManager` na stránce; potom knihovny ASP.NET AJAX je načtena, aby bylo možné použít Toolkitu:

[!code-aspx[Main](animating-in-response-to-user-interaction-vb/samples/sample1.aspx)]

Animace se použijí na panel textu, který vypadá takto:

[!code-aspx[Main](animating-in-response-to-user-interaction-vb/samples/sample2.aspx)]

Související třídy CSS pro panel definovat barvu pozadí dobrý a také nastavit pevnou šířku pro panel:

[!code-css[Main](animating-in-response-to-user-interaction-vb/samples/sample3.css)]

Poté, přidejte `AnimationExtender` na stránku, poskytuje `ID`, `TargetControlID` atribut a povinný údaj `runat="server"`:

[!code-aspx[Main](animating-in-response-to-user-interaction-vb/samples/sample4.aspx)]

V rámci `<Animations>` uzlu pět způsobů, jak pro spuštění animace prostřednictvím zásah uživatele (chybí element je `<OnLoad>` který je proveden po celé stránky se načetl plně):

- `<OnClick>`(myši klikněte na ovládací prvek)
- `<OnHoverOut>`(ukazatel myši opustí ovládací prvek)
- `<OnHoverOver>`(ukazatel myši nachází ovládacího prvku zastavení `<OnHoverOut>` animace)
- `<OnMouseOut>`(myši ponechá ovládací prvek)
- `<OnMouseOver>`(ukazatel myši nachází ovládacího prvku není zastavení `<OnMouseOut>` animace)

V tomto scénáři `<OnClick>` se používá. Když uživatel klikne na panelu, se změnila velikost a setmívá ve stejnou dobu.

[!code-aspx[Main](animating-in-response-to-user-interaction-vb/samples/sample5.aspx)]


[![Animace bude spuštěna, klikněte na tlačítko myši](animating-in-response-to-user-interaction-vb/_static/image2.png)](animating-in-response-to-user-interaction-vb/_static/image1.png)

Animace bude spuštěna, klikněte na tlačítko myši ([Kliknutím zobrazit obrázek v plné velikosti](animating-in-response-to-user-interaction-vb/_static/image3.png))

>[!div class="step-by-step"]
[Předchozí](picking-one-animation-out-of-a-list-vb.md)
[další](disabling-actions-during-animation-vb.md)