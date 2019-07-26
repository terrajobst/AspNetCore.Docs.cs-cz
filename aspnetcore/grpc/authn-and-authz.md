---
title: Ověřování a autorizace v gRPC pro ASP.NET Core
author: jamesnk
description: Naučte se používat ověřování a autorizaci v gRPC pro ASP.NET Core.
monikerRange: '>= aspnetcore-3.0'
ms.author: jamesnk
ms.date: 06/07/2019
uid: grpc/authn-and-authz
ms.openlocfilehash: 49024295e4db7976924397bb24567d92d6298562
ms.sourcegitcommit: b40613c603d6f0cc71f3232c16df61550907f550
ms.translationtype: MT
ms.contentlocale: cs-CZ
ms.lasthandoff: 07/18/2019
ms.locfileid: "68308815"
---
# <a name="authentication-and-authorization-in-grpc-for-aspnet-core"></a><span data-ttu-id="cb4a3-103">Ověřování a autorizace v gRPC pro ASP.NET Core</span><span class="sxs-lookup"><span data-stu-id="cb4a3-103">Authentication and authorization in gRPC for ASP.NET Core</span></span>

<span data-ttu-id="cb4a3-104">Od [James Newton – král](https://twitter.com/jamesnk)</span><span class="sxs-lookup"><span data-stu-id="cb4a3-104">By [James Newton-King](https://twitter.com/jamesnk)</span></span>

<span data-ttu-id="cb4a3-105">[Zobrazit nebo stáhnout vzorový kód](https://github.com/aspnet/AspNetCore.Docs/tree/master/aspnetcore/grpc/authn-and-authz/sample/) [(stažení)](xref:index#how-to-download-a-sample)</span><span class="sxs-lookup"><span data-stu-id="cb4a3-105">[View or download sample code](https://github.com/aspnet/AspNetCore.Docs/tree/master/aspnetcore/grpc/authn-and-authz/sample/) [(how to download)](xref:index#how-to-download-a-sample)</span></span>

## <a name="authenticate-users-calling-a-grpc-service"></a><span data-ttu-id="cb4a3-106">Ověřování uživatelů volajících služby gRPC</span><span class="sxs-lookup"><span data-stu-id="cb4a3-106">Authenticate users calling a gRPC service</span></span>

<span data-ttu-id="cb4a3-107">gRPC se dá použít s [ověřováním ASP.NET Core](xref:security/authentication/identity) k přidružení uživatele ke každému volání.</span><span class="sxs-lookup"><span data-stu-id="cb4a3-107">gRPC can be used with [ASP.NET Core authentication](xref:security/authentication/identity) to associate a user with each call.</span></span>

<span data-ttu-id="cb4a3-108">Následuje příklad `Startup.Configure` , který používá gRPC a ASP.NET Core ověřování:</span><span class="sxs-lookup"><span data-stu-id="cb4a3-108">The following is an example of `Startup.Configure` which uses gRPC and ASP.NET Core authentication:</span></span>

```csharp
public void Configure(IApplicationBuilder app)
{
    app.UseRouting();
    
    app.UseAuthentication();
    app.UseAuthorization();

    app.UseEndpoints(endpoints =>
    {
        routes.MapGrpcService<GreeterService>();
    });
}
```

> [!NOTE]
> <span data-ttu-id="cb4a3-109">Pořadí, ve kterém zaregistrujete ASP.NET Core middlewaru ověřování.</span><span class="sxs-lookup"><span data-stu-id="cb4a3-109">The order in which you register the ASP.NET Core authentication middleware matters.</span></span> <span data-ttu-id="cb4a3-110">Vždy volejte `UseAuthentication` `UseAuthorization` apřed`UseEndpoints`a před. `UseRouting`</span><span class="sxs-lookup"><span data-stu-id="cb4a3-110">Always call `UseAuthentication` and `UseAuthorization` after `UseRouting` and before `UseEndpoints`.</span></span>

<span data-ttu-id="cb4a3-111">Po nastavení ověřování se k uživateli dá v metodách služby gRPC přístup pomocí `ServerCallContext`.</span><span class="sxs-lookup"><span data-stu-id="cb4a3-111">Once authentication has been setup, the user can be accessed in a gRPC service methods via the `ServerCallContext`.</span></span>

```csharp
public override Task<BuyTicketsResponse> BuyTickets(
    BuyTicketsRequest request, ServerCallContext context)
{
    var user = context.GetHttpContext().User;

    // ... access data from ClaimsPrincipal ...
}

```

### <a name="bearer-token-authentication"></a><span data-ttu-id="cb4a3-112">Ověřování nosných tokenů</span><span class="sxs-lookup"><span data-stu-id="cb4a3-112">Bearer token authentication</span></span>

<span data-ttu-id="cb4a3-113">Klient může pro ověřování poskytnout přístupový token.</span><span class="sxs-lookup"><span data-stu-id="cb4a3-113">The client can provide an access token for authentication.</span></span> <span data-ttu-id="cb4a3-114">Server token ověří a použije ho k identifikaci uživatele.</span><span class="sxs-lookup"><span data-stu-id="cb4a3-114">The server validates the token and uses it to identify the user.</span></span>

<span data-ttu-id="cb4a3-115">Na serveru je ověřování pomocí tokenu nosiče nakonfigurované pomocí [middleware nosiče JWT](/dotnet/api/microsoft.extensions.dependencyinjection.jwtbearerextensions.addjwtbearer).</span><span class="sxs-lookup"><span data-stu-id="cb4a3-115">On the server, bearer token authentication is configured using the [JWT Bearer middleware](/dotnet/api/microsoft.extensions.dependencyinjection.jwtbearerextensions.addjwtbearer).</span></span>

<span data-ttu-id="cb4a3-116">V klientovi .NET gRPC je možné token odeslat s voláními jako hlavičku:</span><span class="sxs-lookup"><span data-stu-id="cb4a3-116">In the .NET gRPC client, the token can be sent with calls as a header:</span></span>

```csharp
public bool DoAuthenticatedCall(
    Ticketer.TicketerClient client, string token)
{
    var headers = new Metadata();
    headers.Add("Authorization", $"Bearer {token}");

    var request = new BuyTicketsRequest { Count = 1 };
    var response = await client.BuyTicketsAsync(request, headers);

    return response.Success;
}
```

### <a name="client-certificate-authentication"></a><span data-ttu-id="cb4a3-117">Ověřování certifikátu klienta</span><span class="sxs-lookup"><span data-stu-id="cb4a3-117">Client certificate authentication</span></span>

<span data-ttu-id="cb4a3-118">Klient může případně poskytnout klientský certifikát pro ověřování.</span><span class="sxs-lookup"><span data-stu-id="cb4a3-118">A client could alternatively provide a client certificate for authentication.</span></span> <span data-ttu-id="cb4a3-119">[Ověřování certifikátu](https://tools.ietf.org/html/rfc5246#section-7.4.4) se provádí na úrovni protokolu TLS dlouho předtím, než se někdy získá ASP.NET Core.</span><span class="sxs-lookup"><span data-stu-id="cb4a3-119">[Certificate authentication](https://tools.ietf.org/html/rfc5246#section-7.4.4) happens at the TLS level, long before it ever gets to ASP.NET Core.</span></span> <span data-ttu-id="cb4a3-120">Když požadavek vstoupí do ASP.NET Core, [balíček pro ověřování certifikátu klienta](xref:security/authentication/certauth) vám umožní tento certifikát přeložit na `ClaimsPrincipal`.</span><span class="sxs-lookup"><span data-stu-id="cb4a3-120">When the request enters ASP.NET Core, the [client certificate authentication package](xref:security/authentication/certauth) allows you to resolve the certificate to a `ClaimsPrincipal`.</span></span>

> [!NOTE]
> <span data-ttu-id="cb4a3-121">Hostitel musí být nakonfigurovaný tak, aby přijímal klientské certifikáty.</span><span class="sxs-lookup"><span data-stu-id="cb4a3-121">The host needs to be configured to accept client certificates.</span></span> <span data-ttu-id="cb4a3-122">Informace o přijímání klientských certifikátů v Kestrel, IIS a Azure najdete v tématu [Konfigurace hostitele pro vyžadování certifikátů](xref:security/authentication/certauth#configure-your-host-to-require-certificates) .</span><span class="sxs-lookup"><span data-stu-id="cb4a3-122">See [configure your host to require certificates](xref:security/authentication/certauth#configure-your-host-to-require-certificates) for information on accepting client certificates in Kestrel, IIS and Azure.</span></span>

<span data-ttu-id="cb4a3-123">V klientovi .NET gRPC se certifikát klienta přidá do `HttpClientHandler` , který pak slouží k vytvoření klienta gRPC:</span><span class="sxs-lookup"><span data-stu-id="cb4a3-123">In the .NET gRPC client, the client certificate is added to `HttpClientHandler` that is then used to create the gRPC client:</span></span>

```csharp
public Ticketer.TicketerClient CreateClientWithCert(
    string baseAddress,
    X509Certificate2 certificate)
{
    // Add client cert to the handler
    var handler = new HttpClientHandler();
    handler.ClientCertificates.Add(certificate);

    // Create the gRPC client
    var httpClient = new HttpClient(handler);
    httpClient.BaseAddress = new Uri(baseAddress);

    return GrpcClient.Create<Ticketer.TicketerClient>(httpClient);
}
```

### <a name="other-authentication-mechanisms"></a><span data-ttu-id="cb4a3-124">Jiné mechanismy ověřování</span><span class="sxs-lookup"><span data-stu-id="cb4a3-124">Other authentication mechanisms</span></span>

<span data-ttu-id="cb4a3-125">Kromě nosných tokenů a ověřování klientských certifikátů je potřeba, aby všechny ASP.NET Core podporované mechanismy ověřování, jako je OAuth, OpenID a Negotiate, pracovaly s gRPC.</span><span class="sxs-lookup"><span data-stu-id="cb4a3-125">In addition to bearer token and client certificate authentication, all ASP.NET Core supported authentication mechanisms such as OAuth, OpenID and Negotiate should work with gRPC.</span></span> <span data-ttu-id="cb4a3-126">Další informace o konfiguraci ověřování na straně serveru najdete na stránce [ASP.NET Core ověřování](xref:security/authentication/identity) .</span><span class="sxs-lookup"><span data-stu-id="cb4a3-126">Visit [ASP.NET Core authentication](xref:security/authentication/identity) for more information for configuring authentication on the server side.</span></span>

<span data-ttu-id="cb4a3-127">Konfigurace na straně klienta bude záviset na mechanismu ověřování, které používáte.</span><span class="sxs-lookup"><span data-stu-id="cb4a3-127">Client side configuration will depend on the authentication mechanism you are using.</span></span> <span data-ttu-id="cb4a3-128">Předchozí tokeny nosiče a příklady ověřování klientského certifikátu ukazují několik způsobů, jak může být klient gRPC nakonfigurovaný tak, aby odesílal metadata ověřování pomocí volání gRPC:</span><span class="sxs-lookup"><span data-stu-id="cb4a3-128">The previous bearer token and client certificate authentication examples show a couple of ways the gRPC client can be configured to send authentication metadata with gRPC calls:</span></span>

* <span data-ttu-id="cb4a3-129">GRPC klienti silného typu `HttpClient` používají interně.</span><span class="sxs-lookup"><span data-stu-id="cb4a3-129">Strongly typed gRPC clients use `HttpClient` internally.</span></span> <span data-ttu-id="cb4a3-130">Ověřování lze nakonfigurovat na [`HttpClientHandler`](/dotnet/api/system.net.http.httpclienthandler)nebo přidáním vlastních [`HttpMessageHandler`](/dotnet/api/system.net.http.httpmessagehandler) instancí do `HttpClient`.</span><span class="sxs-lookup"><span data-stu-id="cb4a3-130">Authentication can be configured on [`HttpClientHandler`](/dotnet/api/system.net.http.httpclienthandler), or by adding custom [`HttpMessageHandler`](/dotnet/api/system.net.http.httpmessagehandler) instances to the `HttpClient`.</span></span>
* <span data-ttu-id="cb4a3-131">Každé volání gRPC má nepovinný `CallOptions` argument.</span><span class="sxs-lookup"><span data-stu-id="cb4a3-131">Each gRPC call has an optional `CallOptions` argument.</span></span> <span data-ttu-id="cb4a3-132">Vlastní záhlaví lze odeslat pomocí kolekce záhlaví možnosti.</span><span class="sxs-lookup"><span data-stu-id="cb4a3-132">Custom headers can be sent using the option's headers collection.</span></span>

## <a name="authorize-users-to-access-services-and-service-methods"></a><span data-ttu-id="cb4a3-133">Autorizace uživatelů přístup k službám a metodám služeb</span><span class="sxs-lookup"><span data-stu-id="cb4a3-133">Authorize users to access services and service methods</span></span>

<span data-ttu-id="cb4a3-134">Ve výchozím nastavení mohou být všechny metody ve službě volány neověřenými uživateli.</span><span class="sxs-lookup"><span data-stu-id="cb4a3-134">By default, all methods in a service can be called by unauthenticated users.</span></span> <span data-ttu-id="cb4a3-135">Chcete-li vyžadovat ověření, použijte pro službu atribut [[autorizovat]](xref:Microsoft.AspNetCore.Authorization.AuthorizeAttribute) :</span><span class="sxs-lookup"><span data-stu-id="cb4a3-135">To require authentication, apply the [[Authorize]](xref:Microsoft.AspNetCore.Authorization.AuthorizeAttribute) attribute to the service:</span></span>

```csharp
[Authorize]
public class TicketerService : Ticketer.TicketerBase
{
}
```

<span data-ttu-id="cb4a3-136">Pomocí argumentů konstruktoru a vlastností `[Authorize]` atributu můžete omezit přístup jenom na uživatele, kteří odpovídají na konkrétní [zásady autorizace](xref:security/authorization/policies).</span><span class="sxs-lookup"><span data-stu-id="cb4a3-136">You can use the constructor arguments and properties of the `[Authorize]` attribute to restrict access to only users matching specific [authorization policies](xref:security/authorization/policies).</span></span> <span data-ttu-id="cb4a3-137">Pokud máte například vlastní zásadu `MyAuthorizationPolicy`autorizace, ujistěte se, že ke službě budou mít přístup jenom uživatelé, kteří mají k této zásadě přístup pomocí následujícího kódu:</span><span class="sxs-lookup"><span data-stu-id="cb4a3-137">For example, if you have a custom authorization policy called `MyAuthorizationPolicy`, ensure that only users matching that policy can access the service using the following code:</span></span>

```csharp
[Authorize("MyAuthorizationPolicy")]
public class TicketerService : Ticketer.TicketerBase
{
}
```

<span data-ttu-id="cb4a3-138">Jednotlivé metody služby mohou mít `[Authorize]` také použit atribut.</span><span class="sxs-lookup"><span data-stu-id="cb4a3-138">Individual service methods can have the `[Authorize]` attribute applied as well.</span></span> <span data-ttu-id="cb4a3-139">Pokud aktuální uživatel neodpovídá zásadám použitým **pro metodu** i třídu, je volajícímu vrácena chyba:</span><span class="sxs-lookup"><span data-stu-id="cb4a3-139">If the current user doesn't match the policies applied to **both** the method and the class, an error is returned to the caller:</span></span>

```csharp
[Authorize]
public class TicketerService : Ticketer.TicketerBase
{
    public override Task<AvailableTicketsResponse> GetAvailableTickets(
        Empty request, ServerCallContext context)
    {
        // ... buy tickets for the current user ...
    }

    [Authorize("Administrators")]
    public override Task<BuyTicketsResponse> RefundTickets(
        BuyTicketsRequest request, ServerCallContext context)
    {
        // ... refund tickets (something only Administrators can do) ..
    }
}
```

## <a name="additional-resources"></a><span data-ttu-id="cb4a3-140">Další zdroje</span><span class="sxs-lookup"><span data-stu-id="cb4a3-140">Additional resources</span></span>

* [<span data-ttu-id="cb4a3-141">Ověřování nosných tokenů v ASP.NET Core</span><span class="sxs-lookup"><span data-stu-id="cb4a3-141">Bearer Token authentication in ASP.NET Core</span></span>](https://blogs.msdn.microsoft.com/webdev/2016/10/27/bearer-token-authentication-in-asp-net-core/)
* [<span data-ttu-id="cb4a3-142">Konfigurace ověřování klientského certifikátu v ASP.NET Core</span><span class="sxs-lookup"><span data-stu-id="cb4a3-142">Configure Client Certificate authentication in ASP.NET Core</span></span>](xref:security/authentication/certauth)