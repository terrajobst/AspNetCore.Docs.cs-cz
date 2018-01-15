---
uid: web-forms/overview/ajax-control-toolkit/nobot/fighting-bots-cs
title: "Požárů robotů (C#) | Microsoft Docs"
author: wenz
description: "Automatizované robotů sádra weblogů a další weby s nevyžádanou poštou, odesílání formulářů komentář bez nutnosti zásahu uživatele. NoBot ovládacího prvku ASP.NET AJAX Con..."
ms.author: aspnetcontent
manager: wpickett
ms.date: 06/02/2008
ms.topic: article
ms.assetid: 0a1917e0-884a-4576-8e93-9ed660faae51
ms.technology: dotnet-webforms
ms.prod: .net-framework
msc.legacyurl: /web-forms/overview/ajax-control-toolkit/nobot/fighting-bots-cs
msc.type: authoredcontent
ms.openlocfilehash: b8eedff4691c1115e242be884f9e74663dc0b4f9
ms.sourcegitcommit: 9a9483aceb34591c97451997036a9120c3fe2baf
ms.translationtype: MT
ms.contentlocale: cs-CZ
ms.lasthandoff: 11/10/2017
---
<a name="fighting-bots-c"></a>Požárů robotů (C#)
====================
podle [Christian Wenz](https://github.com/wenz)

[Stáhněte si kód](http://download.microsoft.com/download/9/3/f/93f8daea-bebd-4821-833b-95205389c7d0/NoBot0.cs.zip) nebo [stáhnout PDF](http://download.microsoft.com/download/b/6/a/b6ae89ee-df69-4c87-9bfb-ad1eb2b23373/nobot0CS.pdf)

> Automatizované robotů sádra weblogů a další weby s nevyžádanou poštou, odesílání formulářů komentář bez nutnosti zásahu uživatele. NoBot ovládacího prvku ASP.NET AJAX Control Toolkit může pomoci boji s těmito robotů.


## <a name="overview"></a>Přehled

Automatizované robotů sádra weblogů a další weby s nevyžádanou poštou, odesílání formulářů komentář bez nutnosti zásahu uživatele. NoBot ovládacího prvku ASP.NET AJAX Control Toolkit může pomoci boji s těmito robotů.

## <a name="steps"></a>Kroky

Do jaké míry robotů jeden běžný postup je použití CAPTCHAs zcela automatizovat veřejné Turing testu s oznámením, počítače a od sebe člověka. Turing test byl původně testu kde někdo třeba rozhodnout, zda je komunikace partnera lidské nebo počítač. Na webu test CAPTCHA obvykle obsahuje bitovou kopii s některé zkreslený písmena na něm. Cílem je, že pouze lidské může číst písmena na bitovou kopii, zatímco algoritmy rozpoznávání znaků se nezdaří.

Existuje několik výhod a nevýhod tohoto přístupu, avšak informace o to jsou nad rámec tohoto návodu. Ale ovládací prvek v sadě nástrojů ovládacího prvku ASP.NET AJAX, která poskytuje podobný postup: `NoBot`. Je snazší překonat než test CAPTCHA, ale je velmi snadno použitelný a tarify velmi dobře na webech jako blogy, kde považuje úspěšné, pokud většinu spamu pokusy jsou nepotlačí, což `NoBot` ovládacího prvku provést.

`NoBot`zpětné volání aktuálního webové formuláře ASP.NET zachytí při splnění alespoň jeden z těchto podmínek:

- V prohlížeči se nepodaří vyřešit stavebnice JavaScript (například při deaktivaci JavaScript)
- Odeslání formuláře pro rychlé uživatele
- IP adresa klienta odeslání formuláře příliš často v určitou dobu.

Chcete-li zkontrolovat pro tyto podmínky `NoBot` ovládacího prvku vyžaduje tyto atributy (všechny z nich volitelný):

- `ResponseMinimumDelaySeconds`minimální množství sekund mezi zpětná vystavení
- `CutoffWindowSeconds`Délka Časový interval, ve kterém jsou postback z jedna IP adresa míry
- `CutoffMaximumInstances`maximální množství za časový interval v sekundách

Následující kód požadavky tohoto nejméně dvou sekund uplynout mezi postback a že jsou pouze pět postback nebo méně v intervalu 30 sekund:

[!code-aspx[Main](fighting-bots-cs/samples/sample1.aspx)]

Potom obvyklým nezapomeňte zahrnout `ScriptManager` na stránce tak, aby se načíst knihovnu ASP.NET AJAX a dá se použít Toolkitu:

[!code-aspx[Main](fighting-bots-cs/samples/sample2.aspx)]

Protože většina kontrol `NoBot` provádí dojít na straně serveru, je potřeba zkontrolovat výsledek těchto ověření. To lze provést pomocí volání `NoBot`na `IsValid()` metoda. Má jeden argument (jako `out` parametr /`ByRef` parametr) je typu `NoBotState`. Jeho řetězcovou reprezentaci obsahuje z důvodu selhání kontroly a `Valid` jinak. Následující kód výstupy zprávu podle `NoBot`je způsobit:

[!code-aspx[Main](fighting-bots-cs/samples/sample3.aspx)]

Nakonec budete potřebovat formuláře pro odeslání a elementu label na výstup zprávu, a dokončení!

[!code-aspx[Main](fighting-bots-cs/samples/sample4.aspx)]

Při spuštění tohoto skriptu a deaktivovat JavaScript nebo odesláním formuláře v rámci prvních dvou sekund nebo odesláním formuláře sedm časy během 30 sekund, zobrazí se chybová zpráva. Ale pomocí tohoto ovládacího prvku dobře, vzhledem k tomu, že pouze přibližně 90 95 % uživatelé mají JavaScript aktivovat, proto se nezdaří 5 až 10 % uživatelů `NoBot`je testování.


[![Tato chybová zpráva by mohla být způsobena robotu](fighting-bots-cs/_static/image2.png)](fighting-bots-cs/_static/image1.png)

Tato chybová zpráva by mohla být způsobena robotu ([Kliknutím zobrazit obrázek v plné velikosti](fighting-bots-cs/_static/image3.png))

>[!div class="step-by-step"]
[Další](fighting-bots-vb.md)