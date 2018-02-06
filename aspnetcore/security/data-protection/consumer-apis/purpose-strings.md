---
title: "Účel řetězce"
author: rick-anderson
description: "Tento dokument podrobně popisuje, jak používá řetězce účel ochrany dat ASP.NET Core rozhraní API."
manager: wpickett
ms.author: riande
ms.date: 10/14/2016
ms.prod: asp.net-core
ms.technology: aspnet
ms.topic: article
uid: security/data-protection/consumer-apis/purpose-strings
ms.openlocfilehash: b4a0db801ecc1c4ba0762f0c9faf7429b4ac097b
ms.sourcegitcommit: a510f38930abc84c4b302029d019a34dfe76823b
ms.translationtype: MT
ms.contentlocale: cs-CZ
ms.lasthandoff: 01/30/2018
---
# <a name="purpose-strings"></a>Účel řetězce

<a name="data-protection-consumer-apis-purposes"></a>

Součásti, které využívají `IDataProtectionProvider` musí projít jedinečný *účely* parametru `CreateProtector` metoda. Účely *parametr* je vyplývajících pro zabezpečení systému ochrany dat, protože poskytuje izolaci mezi kryptografických příjemci, i v případě kořenové kryptografické klíče.

Pokud příjemce určuje účel, účel řetězec se používá spolu s kryptografickými klíči, které kořenové odvození kryptografických podklíče jedinečný pro tento příjemce. To izoluje příjemce z jiných kryptografických příjemce v aplikaci: žádné další součásti může číst jeho datové části a ho nelze číst datové části všechny ostatní součásti. Tato izolace také vykreslí je nemožné celý kategorie útoky na komponentu.

![Příklad diagramu účel](purpose-strings/_static/purposes.png)

V diagramu výše `IDataProtector` instancí A a B **nelze** číst vzájemně datové části, jenom svoje vlastní.

Účel řetězec nemusí být skrytá. Jednoduše musí být jedinečný. v tom smyslu, že žádné další dobře behaved součást někdy zadejte stejný účel řetězec.

>[!TIP]
> Použití oboru názvů a typ názvu komponenty využívání rozhraními API ochrany dat. je obvykle, stejně jako postup, který tyto informace se nikdy dojít ke konfliktu.
>
>Komponenta vytvořené Contoso, která je zodpovědná za minting nosné tokeny může použít Contoso.Security.BearerToken jako řetězec jeho účel. Nebo – i lépe - může použít Contoso.Security.BearerToken.v1 jako řetězec jeho účel. Připojování číslo verze umožňuje budoucí verze se má použít Contoso.Security.BearerToken.v2 jako její účel a různé verze by naprosto izolované od sebe navzájem, pokud jde o datové části přejděte.

Od parametru účely `CreateProtector` je pole řetězců, výše může místo toho určeny jako `[ "Contoso.Security.BearerToken", "v1" ]`. To umožňuje vytvoření hierarchie pro účely a otevře možnost výskytu nekonzistentních víceklientský scénáře ochrany systémem data.

<a name="data-protection-contoso-purpose"></a>

>[!WARNING]
> Součásti neměli povolit nedůvěryhodné uživatelský vstup jako jediný zdroj vstup pro účely řetězec.
>
>Představte si třeba součást Contoso.Messaging.SecureMessage, která je zodpovědná za ukládání zabezpečených zpráv. Kdyby došlo k volání zabezpečené součástí zasílání zpráv `CreateProtector([ username ])`, pak uživatel se zlými úmysly může vytvořit účet s uživatelským jménem "Contoso.Security.BearerToken" pokus o získání komponentu volat `CreateProtector([ "Contoso.Security.BearerToken" ])`, což nechtěně způsobuje zabezpečené zasílání zpráv systém pro máta datových částí, které může být považována za tokeny ověřování.
>
>Řetěz pro účely lepší pro komponentu zasílání zpráv by `CreateProtector([ "Contoso.Messaging.SecureMessage", "User: username" ])`, který poskytuje správné izolace.

Izolace poskytované a chování `IDataProtectionProvider`, `IDataProtector`, a pro účely jsou následující:

* Pro danou `IDataProtectionProvider` objekt, `CreateProtector` metoda vytvoří `IDataProtector` objekt jednoznačně vázáno na obě `IDataProtectionProvider` objekt, který jej vytvořil a účely parametr, který byl předán do metody.

* Účel parametr nesmí mít hodnotu null. (Pokud účely je zadán jako pole, to znamená, že pole nesmí mít nulovou délku a všechny elementy pole musí obsahovat hodnotu null.) Prázdný řetězec účel technicky je povoleno, ale se nedoporučuje.

* Dva účely argumenty jsou ekvivalentní a pouze v případě obsahují stejné řetězce (s použitím pořadí porovnávače) ve stejném pořadí. Jednoúčelové argument je stejná jako odpovídající pole pro účely jeden element.

* Dva `IDataProtector` jsou ekvivalentní objekty, pokud jste vytvořili z ekvivalentní `IDataProtectionProvider` objekty s parametry ekvivalentní účely.

* Pro danou `IDataProtector` objektu, volání `Unprotect(protectedData)` vrátí původní `unprotectedData` jenom v případě `protectedData := Protect(unprotectedData)` pro ekvivalentní `IDataProtector` objektu.

> [!NOTE]
> Jsme nejsou vzhledem k tomu tento případ, kdy některé součásti záměrně zvolí účel řetězce, který je znám v konfliktu s jinou součástí. Tato součást bude v podstatě považováno za škodlivý a tento systém není určená k poskytování záruky zabezpečení, v případě, že škodlivý kód je již spuštěna v rámci pracovního procesu.