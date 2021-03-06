---
title: Schémata zásad v ASP.NET Core
author: rick-anderson
description: Schémata zásad ověřování usnadňují jedno logické schéma ověřování.
ms.author: riande
ms.date: 12/05/2019
uid: security/authentication/policyschemes
ms.openlocfilehash: f02d8e5cac20a9b60c5eddbd28253efacf682ea1
ms.sourcegitcommit: 9a129f5f3e31cc449742b164d5004894bfca90aa
ms.translationtype: MT
ms.contentlocale: cs-CZ
ms.lasthandoff: 03/06/2020
ms.locfileid: "78660732"
---
# <a name="policy-schemes-in-aspnet-core"></a>Schémata zásad v ASP.NET Core

Schémata zásad ověřování usnadňují použití jediného logického schématu ověřování, který potenciálně používá více přístupů. Schéma zásad může například používat ověřování Google pro problémy a ověřování souborů cookie pro všechno ostatní. Schémata zásad ověřování ji zavedli:

* Snadné přeposílání jakékoli akce ověřování do jiného schématu.
* Dynamické směrování na základě požadavku.

Všechna schémata ověřování, která používají odvozený <xref:Microsoft.AspNetCore.Authentication.AuthenticationSchemeOptions> a související [AuthenticationHandler\<TOptions >](/dotnet/api/microsoft.aspnetcore.authentication.authenticationhandler-1):

* Jsou automatická schémata zásad v ASP.NET Core 2,1 a novějších.
* Dá se povolit prostřednictvím konfigurace možností schématu.

[!code-csharp[sample](policyschemes/samples/AuthenticationSchemeOptions.cs?name=snippet)]

## <a name="examples"></a>Příklady

Následující příklad ukazuje schéma vyšší úrovně, které kombinuje schémata nižší úrovně. Ověřování Google se používá pro problémy a ověřování souborů cookie se používá pro všechno ostatní:

[!code-csharp[sample](policyschemes/samples/Startup.cs?name=snippet1)]

Následující příklad povoluje dynamický výběr schémat na základě jednotlivých požadavků. To znamená, jak kombinovat soubory cookie a ověřování rozhraní API:

 <!-- REVIEW, missing If set in public Func<HttpContext, string> ForwardDefaultSelector -->

[!code-csharp[sample](policyschemes/samples/Startup.cs?name=snippet2)]
