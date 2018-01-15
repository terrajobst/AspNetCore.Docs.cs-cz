---
uid: web-forms/overview/ajax-control-toolkit/reorderlist/drag-and-drop-via-reorderlist-vb
title: "Přetáhnout myší prostřednictvím ReorderList (VB) | Microsoft Docs"
author: wenz
description: /Data-Access/tutorials/Master-Detail-Using-a-bulleted-list-of-Master-records-with-a-details-DataList-VB
ms.author: aspnetcontent
manager: wpickett
ms.date: 06/02/2008
ms.topic: article
ms.assetid: 848e6bcf-4c3f-4d14-974d-e45b9444ab79
ms.technology: dotnet-webforms
ms.prod: .net-framework
msc.legacyurl: /web-forms/overview/ajax-control-toolkit/reorderlist/drag-and-drop-via-reorderlist-vb
msc.type: authoredcontent
ms.openlocfilehash: e193a31fc86b7e8733d0b2fba371d99c62783d6c
ms.sourcegitcommit: 9a9483aceb34591c97451997036a9120c3fe2baf
ms.translationtype: MT
ms.contentlocale: cs-CZ
ms.lasthandoff: 11/10/2017
---
<a name="drag-and-drop-via-reorderlist-vb"></a>Přetáhnout myší prostřednictvím ReorderList (VB)
====================
podle [Christian Wenz](https://github.com/wenz)

[Stáhněte si kód](http://download.microsoft.com/download/9/3/f/93f8daea-bebd-4821-833b-95205389c7d0/ReorderList5.vb.zip) nebo [stáhnout PDF](http://download.microsoft.com/download/2/d/c/2dc10e34-6983-41d4-9c08-f78f5387d32b/reorderlist5VB.pdf)

> ReorderList ovládacího prvku AJAX Control Toolkit poskytuje seznam, který může být přeuspořádány uživatelem pomocí operace přetažení. Aktuální pořadí položek v seznamu se na serveru zachován.


## <a name="overview"></a>Přehled

`ReorderList` Ovládacího prvku AJAX Control Toolkit poskytuje seznam, který může být přeuspořádány uživatelem pomocí operace přetažení. Aktuální pořadí položek v seznamu se na serveru zachován.

## <a name="steps"></a>Kroky

`ReorderList` Řízení podporuje vazba dat z databáze do seznamu. Navíc také podporuje zápis změn pracovního element seznamu zpět do úložiště dat.

Tato ukázka používá jako úložiště dat Microsoft SQL Server 2005 Express Edition. Databáze je volitelné (a free) součástí instalaci sady Visual Studio, včetně express edition. Je také k dispozici jako samostatný soubor ke stažení v rámci [https://go.microsoft.com/fwlink/?LinkId=64064](https://go.microsoft.com/fwlink/?LinkId=64064). Tato ukázka předpokládáme, že je název instance systému SQL Server 2005 Express Edition `SQLEXPRESS` a se nachází na stejném počítači jako webový server; toto je také výchozí nastavení. Pokud vaše instalace se liší, budete muset přizpůsobit informace o připojení pro databázi.

Nejjednodušší způsob, jak nastavení databáze je použití Microsoft SQL Server Management Studio Express ([https://www.microsoft.com/downloads/details.aspx?FamilyID=c243a5ae-4bd1-4e3d-94b8-5a0f62bf7796&amp;DisplayLang = en](https://www.microsoft.com/downloads/details.aspx?FamilyID=c243a5ae-4bd1-4e3d-94b8-5a0f62bf7796&amp;DisplayLang=en) ). Připojení k serveru, dvakrát klikněte na `Databases` a vytvořit novou databázi (klikněte pravým tlačítkem a zvolte `New Database`) názvem `Tutorials`.

V této databázi, vytvořte novou tabulku s názvem `AJAX` obsahuje následující čtyři sloupce:

- `id`(primární klíče, celé číslo, identity, není NULL)
- `char`(char(1), NULL)
- `description`(varchar(50), NULL)
- `position`(int, NULL)


[![Rozložení tabulky AJAX](drag-and-drop-via-reorderlist-vb/_static/image2.png)](drag-and-drop-via-reorderlist-vb/_static/image1.png)

Rozložení tabulky AJAX ([Kliknutím zobrazit obrázek v plné velikosti](drag-and-drop-via-reorderlist-vb/_static/image3.png))


V tabulce v dalším kroku vyplníte několik hodnot. Všimněte si, že `position` sloupec obsahuje pořadí řazení elementů.


[![Počáteční data v tabulce AJAX](drag-and-drop-via-reorderlist-vb/_static/image5.png)](drag-and-drop-via-reorderlist-vb/_static/image4.png)

Počáteční data v tabulce AJAX ([Kliknutím zobrazit obrázek v plné velikosti](drag-and-drop-via-reorderlist-vb/_static/image6.png))


Další krok se vyžaduje k vygenerování `SqlDataSource` řízení ke komunikaci s novou databázi a její tabulkou. Zdroj dat musí podporovat `SELECT` a `UPDATE` příkazy SQL. Když pořadí prvků seznamu je později změnit, `ReorderList` řízení automaticky odesílá dvě hodnoty ke zdroji dat `Update` příkaz: nové pozice a ID elementu. Proto zdroj dat, musí `<UpdateParameters>` části pro tyto dvě hodnoty:

[!code-aspx[Main](drag-and-drop-via-reorderlist-vb/samples/sample1.aspx)]

`ReorderList` Ovládací prvek musí nastavit následující atributy:

- `AllowReorder`: Zda můžete měnit položky seznamu
- `DataSourceID`: ID zdroje dat
- `DataKeyField`: Název sloupec primárního klíče ve zdroji dat.
- `SortOrderField`: Data zdrojový sloupec, který poskytuje pořadí řazení pro položky seznamu

V `<DragHandleTemplate>` a `<ItemTemplate>` částí rozložení seznamu lze podrobně nastavit. Datové vazby je taky možné pomocí `Eval()` metoda, jak je vidět tady:

[!code-aspx[Main](drag-and-drop-via-reorderlist-vb/samples/sample2.aspx)]

Informace o stylu následující CSS (odkazovaná v `<DragHandleTemplate>` části `ReorderList` ovládací prvek) zajišťuje, že když setrvá úchytu změní ukazatel myši správně:

[!code-css[Main](drag-and-drop-via-reorderlist-vb/samples/sample3.css)]

Nakonec `ScriptManager` řízení inicializuje prvku ASP.NET AJAX stránky:

[!code-aspx[Main](drag-and-drop-via-reorderlist-vb/samples/sample4.aspx)]

Spuštění tohoto příkladu v prohlížeči a trochu Změna uspořádání položek seznamu. Potom znovu načíst stránku a podívejte se na databáze. Změnit pozice nebyla zachována a se projeví také podle hodnot v `position` sloupec v databázi a že všechny bez žádný kód, právě pomocí značek.


[![Data v databázi změny v pořadí, nové položky seznamu](drag-and-drop-via-reorderlist-vb/_static/image8.png)](drag-and-drop-via-reorderlist-vb/_static/image7.png)

Data v databázi se změní podle seznamu novou položku pořadí ([Kliknutím zobrazit obrázek v plné velikosti](drag-and-drop-via-reorderlist-vb/_static/image9.png))

>[!div class="step-by-step"]
[Předchozí](using-postbacks-with-reorderlist-vb.md)