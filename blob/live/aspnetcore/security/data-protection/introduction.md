---
title: "Úvod do ochrany dat"
author: rick-anderson
description: "Tento dokument zavádí koncepci ochrany dat a popisuje zásady přidružené rozhraní API ASP.NET Core."
ms.author: riande
manager: wpickett
ms.date: 10/14/2016
ms.topic: article
ms.technology: aspnet
ms.prod: asp.net-core
uid: security/data-protection/introduction
ms.openlocfilehash: b98027ee0e7c63bac23054d7623f28294388dede
ms.sourcegitcommit: 3e303620a125325bb9abd4b2d315c106fb8c47fd
ms.translationtype: MT
ms.contentlocale: cs-CZ
ms.lasthandoff: 01/19/2018
---
# <a name="introduction-to-data-protection"></a><span data-ttu-id="1b932-103">Úvod do ochrany dat</span><span class="sxs-lookup"><span data-stu-id="1b932-103">Introduction to Data Protection</span></span>

<span data-ttu-id="1b932-104">Webové aplikace často potřebují k ukládání dat citlivé na zabezpečení.</span><span class="sxs-lookup"><span data-stu-id="1b932-104">Web applications often need to store security-sensitive data.</span></span> <span data-ttu-id="1b932-105">Windows poskytuje rozhraní DPAPI pro aplikací klasické pracovní plochy, ale to není vhodná pro webové aplikace.</span><span class="sxs-lookup"><span data-stu-id="1b932-105">Windows provides DPAPI for desktop applications but this is unsuitable for web applications.</span></span> <span data-ttu-id="1b932-106">Zásobník ochrany dat ASP.NET Core poskytují jednoduchý a snadno použitelný API kryptografických vývojář může použít k ochraně dat, včetně správy klíčů a otočení.</span><span class="sxs-lookup"><span data-stu-id="1b932-106">The ASP.NET Core data protection stack provide a simple, easy to use cryptographic API a developer can use to protect data, including key management and rotation.</span></span>

<span data-ttu-id="1b932-107">Zásobník ochrany dat ASP.NET Core slouží k sloužit jako dlouhodobé náhradou <machineKey> element technologie ASP.NET 1.x - 4.x.</span><span class="sxs-lookup"><span data-stu-id="1b932-107">The ASP.NET Core data protection stack is designed to serve as the long-term replacement for the <machineKey> element in ASP.NET 1.x - 4.x.</span></span> <span data-ttu-id="1b932-108">Byla navržená tak, aby vyřešit řadu nedostatků starý kryptografický zásobníku při současném poskytování out-of-the-box řešení pro většinu případů použití, které moderní aplikace jsou setkat.</span><span class="sxs-lookup"><span data-stu-id="1b932-108">It was designed to address many of the shortcomings of the old cryptographic stack while providing an out-of-the-box solution for the majority of use cases modern applications are likely to encounter.</span></span>

## <a name="problem-statement"></a><span data-ttu-id="1b932-109">Stanovení problému</span><span class="sxs-lookup"><span data-stu-id="1b932-109">Problem statement</span></span>

<span data-ttu-id="1b932-110">Příkaz celkový problém, můžete stručně uvedená v jedné větě: je nutné zachovat důvěryhodné informace pro pozdější načtení, ale I nedůvěřujete mechanismus trvalosti.</span><span class="sxs-lookup"><span data-stu-id="1b932-110">The overall problem statement can be succinctly stated in a single sentence: I need to persist trusted information for later retrieval, but I do not trust the persistence mechanism.</span></span> <span data-ttu-id="1b932-111">V podmínkách webové to může být zapsána jako "Potřebuji odezvy důvěryhodného stavu prostřednictvím nedůvěryhodného klienta."</span><span class="sxs-lookup"><span data-stu-id="1b932-111">In web terms, this might be written as "I need to round-trip trusted state via an untrusted client."</span></span>

<span data-ttu-id="1b932-112">Kanonický příklad tohoto objektu je soubor cookie pro ověřování nebo nosiče tokenu.</span><span class="sxs-lookup"><span data-stu-id="1b932-112">The canonical example of this is an authentication cookie or bearer token.</span></span> <span data-ttu-id="1b932-113">Generuje server "Jsem Groot a mít oprávnění xyz" token a předá ho do klienta.</span><span class="sxs-lookup"><span data-stu-id="1b932-113">The server generates an "I am Groot and have xyz permissions" token and hands it to the client.</span></span> <span data-ttu-id="1b932-114">Někdy v budoucnu nevypnete Klient nabídne tento token zpět na server, ale server potřebuje nějaký druh záruku, že klient nebyl forged token.</span><span class="sxs-lookup"><span data-stu-id="1b932-114">At some future date the client will present that token back to the server, but the server needs some kind of assurance that the client hasn't forged the token.</span></span> <span data-ttu-id="1b932-115">Proto první požadavek: pravosti (také známa jako</span><span class="sxs-lookup"><span data-stu-id="1b932-115">Thus the first requirement: authenticity (a.k.a.</span></span> <span data-ttu-id="1b932-116">integritu, zfalšování kontroly pravopisu systému).</span><span class="sxs-lookup"><span data-stu-id="1b932-116">integrity, tamper-proofing).</span></span>

<span data-ttu-id="1b932-117">Vzhledem k tomu, že trvalého stavu je důvěryhodný pro server, Očekáváme, že tento stav může obsahovat informace, které jsou specifické pro provozní prostředí.</span><span class="sxs-lookup"><span data-stu-id="1b932-117">Since the persisted state is trusted by the server, we anticipate that this state might contain information that is specific to the operating environment.</span></span> <span data-ttu-id="1b932-118">To může být ve tvaru cestu k souboru, oprávnění, popisovač nebo nepřímý odkaz na jiné a některé další část dat specifickou pro server.</span><span class="sxs-lookup"><span data-stu-id="1b932-118">This could be in the form of a file path, a permission, a handle or other indirect reference, or some other piece of server-specific data.</span></span> <span data-ttu-id="1b932-119">Tyto informace by neměl být poskytnuty obecně nedůvěryhodného klienta.</span><span class="sxs-lookup"><span data-stu-id="1b932-119">Such information should generally not be disclosed to an untrusted client.</span></span> <span data-ttu-id="1b932-120">Proto požadavek na druhý: utajení.</span><span class="sxs-lookup"><span data-stu-id="1b932-120">Thus the second requirement: confidentiality.</span></span>

<span data-ttu-id="1b932-121">Nakonec od moderní aplikace jsou komponentizované, co jste viděli je, budou jednotlivé komponenty chcete využít tento systém bez ohledu na ostatní součásti v systému.</span><span class="sxs-lookup"><span data-stu-id="1b932-121">Finally, since modern applications are componentized, what we've seen is that individual components will want to take advantage of this system without regard to other components in the system.</span></span> <span data-ttu-id="1b932-122">Například pokud součást tokenu nosiče používá tento zásobníku, ho pracovat bez rušení mechanismus anti-proti útokům CSRF, která by mohla využívat se stejným zásobníkem.</span><span class="sxs-lookup"><span data-stu-id="1b932-122">For instance, if a bearer token component is using this stack, it should operate without interference from an anti-CSRF mechanism that might also be using the same stack.</span></span> <span data-ttu-id="1b932-123">Proto požadavek na konečné: izolace.</span><span class="sxs-lookup"><span data-stu-id="1b932-123">Thus the final requirement: isolation.</span></span>

<span data-ttu-id="1b932-124">Můžeme poskytnout další omezení chcete-li zúžit rozsah naše požadavky.</span><span class="sxs-lookup"><span data-stu-id="1b932-124">We can provide further constraints in order to narrow the scope of our requirements.</span></span> <span data-ttu-id="1b932-125">Předpokládáme, že jsou všechny služby, které pracují v rámci cryptosystem stejně důvěryhodné a že data nemusí generovat ani spotřebováno mimo služby v našem přímou kontrolu.</span><span class="sxs-lookup"><span data-stu-id="1b932-125">We assume that all services operating within the cryptosystem are equally trusted and that the data does not need to be generated or consumed outside of the services under our direct control.</span></span> <span data-ttu-id="1b932-126">Kromě toho je nutné, operace jsou tak rychlý jako možné vzhledem k tomu, že každý požadavek pro webovou službu projít cryptosystem jeden či více krát.</span><span class="sxs-lookup"><span data-stu-id="1b932-126">Furthermore, we require that operations are as fast as possible since each request to the web service might go through the cryptosystem one or more times.</span></span> <span data-ttu-id="1b932-127">Díky tomu symetrické šifrování ideální pro náš scénář a jsme slevy asymetrické šifrování, dokud například čas, kdy je potřeba.</span><span class="sxs-lookup"><span data-stu-id="1b932-127">This makes symmetric cryptography ideal for our scenario, and we can discount asymmetric cryptography until such a time that it is needed.</span></span>

## <a name="design-philosophy"></a><span data-ttu-id="1b932-128">Filosofie návrhu tříd</span><span class="sxs-lookup"><span data-stu-id="1b932-128">Design philosophy</span></span>

<span data-ttu-id="1b932-129">Spuštění tím, že určíte problémy s existující zásobníku.</span><span class="sxs-lookup"><span data-stu-id="1b932-129">We started by identifying problems with the existing stack.</span></span> <span data-ttu-id="1b932-130">Jakmile jsme měli který, jsme názory povahu existující řešení a dospělo k závěru, že žádné existující řešení poměrně měl možnostem, které jsme žádá o.</span><span class="sxs-lookup"><span data-stu-id="1b932-130">Once we had that, we surveyed the landscape of existing solutions and concluded that no existing solution quite had the capabilities we sought.</span></span> <span data-ttu-id="1b932-131">Potom jsme analyzovány řešení založené na několik hlavních zásad.</span><span class="sxs-lookup"><span data-stu-id="1b932-131">We then engineered a solution based on several guiding principles.</span></span>

* <span data-ttu-id="1b932-132">Systém by měl nabízejí zjednodušení konfigurace.</span><span class="sxs-lookup"><span data-stu-id="1b932-132">The system should offer simplicity of configuration.</span></span> <span data-ttu-id="1b932-133">V ideálním případě systém by bez nutnosti konfigurace a vývojáři mohou dosáhl základů systémem.</span><span class="sxs-lookup"><span data-stu-id="1b932-133">Ideally the system would be zero-configuration and developers could hit the ground running.</span></span> <span data-ttu-id="1b932-134">V situacích, kdy je potřeba nakonfigurovat konkrétní aspekt (například klíče úložiště) vývojáři by měla přihlédnout k provedení těchto konkrétní konfigurace jednoduché.</span><span class="sxs-lookup"><span data-stu-id="1b932-134">In situations where developers need to configure a specific aspect (such as the key repository), consideration should be given to making those specific configurations simple.</span></span>

* <span data-ttu-id="1b932-135">Nabízí jednoduché rozhraní API určených.</span><span class="sxs-lookup"><span data-stu-id="1b932-135">Offer a simple consumer-facing API.</span></span> <span data-ttu-id="1b932-136">Rozhraní API by měl být snadno použitelné správně a obtížně použitelný nesprávně.</span><span class="sxs-lookup"><span data-stu-id="1b932-136">The APIs should be easy to use correctly and difficult to use incorrectly.</span></span>

* <span data-ttu-id="1b932-137">Vývojáři by neměl další zásady správy klíčů.</span><span class="sxs-lookup"><span data-stu-id="1b932-137">Developers should not learn key management principles.</span></span> <span data-ttu-id="1b932-138">Systém by měla řídit výběr algoritmus a životnosti klíče jménem pro vývojáře.</span><span class="sxs-lookup"><span data-stu-id="1b932-138">The system should handle algorithm selection and key lifetime on the developer's behalf.</span></span> <span data-ttu-id="1b932-139">V ideálním případě by měl vývojář nikdy i mít přístup k nezpracované materiál klíče.</span><span class="sxs-lookup"><span data-stu-id="1b932-139">Ideally the developer should never even have access to the raw key material.</span></span>

* <span data-ttu-id="1b932-140">Klíčů by měly být chráněné v klidovém stavu, pokud je to možné.</span><span class="sxs-lookup"><span data-stu-id="1b932-140">Keys should be protected at rest when possible.</span></span> <span data-ttu-id="1b932-141">Systém by měl zjistit mechanismus ochrany odpovídající výchozí a použít automaticky.</span><span class="sxs-lookup"><span data-stu-id="1b932-141">The system should figure out an appropriate default protection mechanism and apply it automatically.</span></span>

<span data-ttu-id="1b932-142">S těmito zásadami pamatovat vyvinuli jednoduchý, [snadno použitelný](using-data-protection.md) data protection zásobníku.</span><span class="sxs-lookup"><span data-stu-id="1b932-142">With these principles in mind we developed a simple, [easy to use](using-data-protection.md) data protection stack.</span></span>

<span data-ttu-id="1b932-143">Ochrana dat ASP.NET Core rozhraní API nejsou primárně určený pro neomezené trvalost důvěrné datové části.</span><span class="sxs-lookup"><span data-stu-id="1b932-143">The ASP.NET Core data protection APIs are not primarily intended for indefinite persistence of confidential payloads.</span></span> <span data-ttu-id="1b932-144">Další technologie, jako [Windows CNG DPAPI](https://msdn.microsoft.com/library/windows/desktop/hh706794%28v=vs.85%29.aspx) a [Azure Rights Management](https://docs.microsoft.com/rights-management/) jsou vhodnější ve scénáři neomezené úložiště, a mají možnosti odpovídajícím způsobem silné správy klíčů.</span><span class="sxs-lookup"><span data-stu-id="1b932-144">Other technologies like [Windows CNG DPAPI](https://msdn.microsoft.com/library/windows/desktop/hh706794%28v=vs.85%29.aspx) and [Azure Rights Management](https://docs.microsoft.com/rights-management/) are more suited to the scenario of indefinite storage, and they have correspondingly strong key management capabilities.</span></span> <span data-ttu-id="1b932-145">Ale nutné dodat, není nic zakazují vývojář pomocí funkce Ochrana dat ASP.NET Core rozhraní API pro dlouhodobou ochranu důvěrných údajů.</span><span class="sxs-lookup"><span data-stu-id="1b932-145">That said, there is nothing prohibiting a developer from using the ASP.NET Core data protection APIs for long-term protection of confidential data.</span></span>

## <a name="audience"></a><span data-ttu-id="1b932-146">Cílová skupina</span><span class="sxs-lookup"><span data-stu-id="1b932-146">Audience</span></span>

<span data-ttu-id="1b932-147">Systém ochrany dat je rozdělené do pěti hlavní balíčky.</span><span class="sxs-lookup"><span data-stu-id="1b932-147">The data protection system is divided into five main packages.</span></span> <span data-ttu-id="1b932-148">Různé aspekty tato rozhraní API cíle tři hlavní cílové skupiny;</span><span class="sxs-lookup"><span data-stu-id="1b932-148">Various aspects of these APIs target three main audiences;</span></span>

1. <span data-ttu-id="1b932-149">[Příjemce rozhraní API přehled](consumer-apis/overview.md) cílové aplikace a framework vývojáři.</span><span class="sxs-lookup"><span data-stu-id="1b932-149">The [Consumer APIs Overview](consumer-apis/overview.md) target application and framework developers.</span></span>

   <span data-ttu-id="1b932-150">"Nechci Další informace o tom, jak funguje v zásobníku nebo o tom, jak je nakonfigurovaný.</span><span class="sxs-lookup"><span data-stu-id="1b932-150">"I don't want to learn about how the stack operates or about how it is configured.</span></span> <span data-ttu-id="1b932-151">Jednoduše chcete provádět některé operace v jako jednoduchý takovým způsobem, který nejdříve s velkou pravděpodobností úspěšně pomocí rozhraní API."</span><span class="sxs-lookup"><span data-stu-id="1b932-151">I simply want to perform some operation in as simple a manner as possible with high probability of using the APIs successfully."</span></span>

2. <span data-ttu-id="1b932-152">[Rozhraní API pro konfiguraci](configuration/overview.md) cílové aplikace vývojáři a správci systému.</span><span class="sxs-lookup"><span data-stu-id="1b932-152">The [configuration APIs](configuration/overview.md) target application developers and system administrators.</span></span>

   <span data-ttu-id="1b932-153">"Potřebuji systém ochrany dat říct, že Moje prostředí vyžaduje nastavení nebo jiné než výchozí cesty."</span><span class="sxs-lookup"><span data-stu-id="1b932-153">"I need to tell the data protection system that my environment requires non-default paths or settings."</span></span>

3. <span data-ttu-id="1b932-154">Rozšiřitelnost vývojáři cílové rozhraní API starosti implementace vlastních zásad.</span><span class="sxs-lookup"><span data-stu-id="1b932-154">The extensibility APIs target developers in charge of implementing custom policy.</span></span> <span data-ttu-id="1b932-155">Využití těchto rozhraní API by omezena na velmi zřídka a došlo, vývojáři vědět zabezpečení.</span><span class="sxs-lookup"><span data-stu-id="1b932-155">Usage of these APIs would be limited to rare situations and experienced, security aware developers.</span></span>

   <span data-ttu-id="1b932-156">"Potřebuji nahradit komponentu celý v rámci systému, protože chování skutečně jedinečné požadavky.</span><span class="sxs-lookup"><span data-stu-id="1b932-156">"I need to replace an entire component within the system because I have truly unique behavioral requirements.</span></span> <span data-ttu-id="1b932-157">Chci se další uncommonly používaných částí plochy rozhraní API k sestavení modulu plug-in, který splňuje mé požadavky."</span><span class="sxs-lookup"><span data-stu-id="1b932-157">I am willing to learn uncommonly-used parts of the API surface in order to build a plugin that fulfills my requirements."</span></span>

## <a name="package-layout"></a><span data-ttu-id="1b932-158">Balíček rozložení</span><span class="sxs-lookup"><span data-stu-id="1b932-158">Package Layout</span></span>

<span data-ttu-id="1b932-159">Zásobník ochrany dat se skládá z pěti balíčky.</span><span class="sxs-lookup"><span data-stu-id="1b932-159">The data protection stack consists of five packages.</span></span>

* <span data-ttu-id="1b932-160">Microsoft.AspNetCore.DataProtection.Abstractions obsahuje základní rozhraní IDataProtectionProvider a IDataProtector.</span><span class="sxs-lookup"><span data-stu-id="1b932-160">Microsoft.AspNetCore.DataProtection.Abstractions contains the basic IDataProtectionProvider and IDataProtector interfaces.</span></span> <span data-ttu-id="1b932-161">Obsahuje taky užitečné rozšiřující metody, které mohou pomoci práci s těmito typy (například přetížení IDataProtector.Protect).</span><span class="sxs-lookup"><span data-stu-id="1b932-161">It also contains useful extension methods that can assist working with these types (e.g., overloads of IDataProtector.Protect).</span></span> <span data-ttu-id="1b932-162">Najdete v části příjemce rozhraní pro další informace.</span><span class="sxs-lookup"><span data-stu-id="1b932-162">See the consumer interfaces section for more information.</span></span> <span data-ttu-id="1b932-163">Pokud někdo jiný zodpovídá za vytvoření instance systému ochrany dat a jednoduše spotřebovávají rozhraní API, budete chtít odkaz Microsoft.AspNetCore.DataProtection.Abstractions.</span><span class="sxs-lookup"><span data-stu-id="1b932-163">If somebody else is responsible for instantiating the data protection system and you are simply consuming the APIs, you'll want to reference Microsoft.AspNetCore.DataProtection.Abstractions.</span></span>

* <span data-ttu-id="1b932-164">Microsoft.AspNetCore.DataProtection obsahuje základní implementace systému ochrany dat, včetně základních kryptografických operací, správu klíčů, konfigurace a rozšíření.</span><span class="sxs-lookup"><span data-stu-id="1b932-164">Microsoft.AspNetCore.DataProtection contains the core implementation of the data protection system, including the core cryptographic operations, key management, configuration, and extensibility.</span></span> <span data-ttu-id="1b932-165">Pokud jste zodpovědný za vytváření instancí systému ochrany dat (například přidáním do IServiceCollection) nebo změnou nebo rozšíření své chování, budete chtít odkaz Microsoft.AspNetCore.DataProtection.</span><span class="sxs-lookup"><span data-stu-id="1b932-165">If you're responsible for instantiating the data protection system (e.g., adding it to an IServiceCollection) or modifying or extending its behavior, you'll want to reference Microsoft.AspNetCore.DataProtection.</span></span>

* <span data-ttu-id="1b932-166">Microsoft.AspNetCore.DataProtection.Extensions obsahuje další rozhraní API, které vývojáři mohou být užitečné, ale které nepatří do základní balíček.</span><span class="sxs-lookup"><span data-stu-id="1b932-166">Microsoft.AspNetCore.DataProtection.Extensions contains additional APIs which developers might find useful but which don't belong in the core package.</span></span> <span data-ttu-id="1b932-167">Například tento balíček obsahuje jednoduché "doložit systému odkazující na adresář konkrétní úložiště klíčů se žádné nastavení vkládání závislostí" rozhraní API (Další informace o).</span><span class="sxs-lookup"><span data-stu-id="1b932-167">For instance, this package contains a simple "instantiate the system pointing at a specific key storage directory with no dependency injection setup" API (more info).</span></span> <span data-ttu-id="1b932-168">Také obsahuje rozšiřující metody pro omezení délky trvání chráněných datových částí (Další informace o).</span><span class="sxs-lookup"><span data-stu-id="1b932-168">It also contains extension methods for limiting the lifetime of protected payloads (more info).</span></span>

* <span data-ttu-id="1b932-169">Microsoft.AspNetCore.DataProtection.SystemWeb lze nainstalovat do stávající aplikaci ASP.NET 4.x přesměrování jeho <machineKey> na místo toho použijte novým zásobníkem ochrany dat operací.</span><span class="sxs-lookup"><span data-stu-id="1b932-169">Microsoft.AspNetCore.DataProtection.SystemWeb can be installed into an existing ASP.NET 4.x application to redirect its <machineKey> operations to instead use the new data protection stack.</span></span> <span data-ttu-id="1b932-170">V tématu [kompatibility](compatibility/replacing-machinekey.md#compatibility-replacing-machinekey) Další informace.</span><span class="sxs-lookup"><span data-stu-id="1b932-170">See [compatibility](compatibility/replacing-machinekey.md#compatibility-replacing-machinekey) for more information.</span></span>

* <span data-ttu-id="1b932-171">Microsoft.AspNetCore.Cryptography.KeyDerivation poskytuje implementaci pro hashování rutiny hesel PBKDF2 a mohou být využívána systémy, které je třeba zpracovat uživatelská hesla bezpečně.</span><span class="sxs-lookup"><span data-stu-id="1b932-171">Microsoft.AspNetCore.Cryptography.KeyDerivation provides an implementation of the PBKDF2 password hashing routine and can be used by systems which need to handle user passwords securely.</span></span> <span data-ttu-id="1b932-172">V tématu [Hashování hesel](consumer-apis/password-hashing.md) Další informace.</span><span class="sxs-lookup"><span data-stu-id="1b932-172">See [Password Hashing](consumer-apis/password-hashing.md) for more information.</span></span>