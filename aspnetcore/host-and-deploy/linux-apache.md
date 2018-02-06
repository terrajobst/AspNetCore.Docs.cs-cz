---
title: "Hostování v systému Linux s Apache ASP.NET Core"
description: "Zjistěte, jak nastavit Apache jako reverzní proxy server na CentOS a přesměrování provozu HTTP na webové aplikace ASP.NET Core systémem Kestrel."
author: spboyer
manager: wpickett
ms.author: spboyer
ms.custom: mvc
ms.date: 10/19/2016
ms.prod: asp.net-core
ms.technology: aspnet
ms.topic: article
uid: host-and-deploy/linux-apache
ms.openlocfilehash: aa55ecd6dc8169e0e77b3899389ec924b1e1ae4a
ms.sourcegitcommit: a510f38930abc84c4b302029d019a34dfe76823b
ms.translationtype: MT
ms.contentlocale: cs-CZ
ms.lasthandoff: 01/30/2018
---
# <a name="host-aspnet-core-on-linux-with-apache"></a>Hostování v systému Linux s Apache ASP.NET Core

Podle [Shayne Boyer](https://github.com/spboyer)

Pomocí tohoto průvodce, zjistěte, jak nastavit [Apache](https://httpd.apache.org/) jako reverzní proxy server na [CentOS 7](https://www.centos.org/) a přesměrování provozu HTTP na webové aplikace ASP.NET Core systémem [Kestrel](xref:fundamentals/servers/kestrel). [Mod_proxy rozšíření](http://httpd.apache.org/docs/2.4/mod/mod_proxy.html) a související moduly vytvořit reverzního proxy serveru.

## <a name="prerequisites"></a>Požadavky

1. Serveru se systémem CentOS 7 a standardní uživatelský účet s oprávněním sudo
2. Aplikace ASP.NET Core

## <a name="publish-the-app"></a>Publikování aplikace

Publikování aplikace jako [samostatná nasazení](/dotnet/core/deploying/#self-contained-deployments-scd) v rámci konfigurace verze pro modul runtime CentOS 7 (`centos.7-x64`). Zkopírujte obsah *bin/Release/netcoreapp2.0/centos.7-x64/publish* složky na serveru pomocí spojovací bod služby, FTP nebo jiné metody přenosu.

> [!NOTE]
> V části nasazení produkční scénář průběžnou integraci pracovní postup funguje publikování aplikace a kopírování prostředků na server. 

## <a name="configure-a-proxy-server"></a>Konfigurace proxy serveru

Reverzní proxy server je běžné instalační program pro obsluhující dynamické webové aplikace. Reverzní proxy server ukončí požadavek HTTP a předává do aplikace ASP.NET.

Proxy server je takovou, která předá požadavky klientů na jiný server místo splnění sám sebe. Reverzní proxy server předává dlouhodobý cíl, obvykle jménem libovolného klientů. V této příručce je Apache nakonfigurovaný jako reverzní proxy server spuštěn na stejném serveru, aby Kestrel obsluhuje aplikace ASP.NET Core.

### <a name="install-apache"></a>Nainstalujte Apache

CentOS balíčky aktualizací pro jejich nejnovější stabilní verze:

```bash
sudo yum update -y
```

Instalace webového serveru Apache na CentOS s jedním `yum` příkaz:

```bash
sudo yum -y install httpd mod_ssl
```

Ukázka výstupu po spuštění příkazu:

```bash
Downloading packages:
httpd-2.4.6-40.el7.centos.4.x86_64.rpm               | 2.7 MB  00:00:01
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
Installing : httpd-2.4.6-40.el7.centos.4.x86_64      1/1 
Verifying  : httpd-2.4.6-40.el7.centos.4.x86_64      1/1 

Installed:
httpd.x86_64 0:2.4.6-40.el7.centos.4

Complete!
```

> [!NOTE]
> V tomto příkladu výstupu odráží httpd.86_64 vzhledem k tomu, že verze CentOS 7 je 64bitový. Pokud chcete ověřit, kde je nainstalován Apache, spusťte `whereis httpd` z příkazového řádku. 

### <a name="configure-apache-for-reverse-proxy"></a>Konfigurace Apache pro reverzní proxy server

Konfigurační soubory pro Apache se nacházejí v rámci `/etc/httpd/conf.d/` adresáře. Všechny soubory s *.conf* rozšíření jsou zpracovávány v abecedním pořadí kromě souborů konfigurace modulu v `/etc/httpd/conf.modules.d/`, která obsahuje všechny konfigurační soubory nezbytné k načtení moduly.

Vytvoření konfiguračního souboru pro aplikaci s názvem `hellomvc.conf`:

```
<VirtualHost *:80>
    ProxyPreserveHost On
    ProxyPass / http://127.0.0.1:5000/
    ProxyPassReverse / http://127.0.0.1:5000/
    ErrorLog /var/log/httpd/hellomvc-error.log
    CustomLog /var/log/httpd/hellomvc-access.log common
</VirtualHost>
```

**VirtualHost** uzlu může vyskytovat více než jednou. v jedné nebo více souborů na serveru. **VirtualHost** je nastaven tak, aby naslouchala na všechny IP adresy pomocí portu 80. Následující dva řádky jsou nastavené na požadavky na proxy serveru v kořenovém adresáři na server v hodnotě 127.0.0.1 na port 5000. Pro obousměrnou komunikaci *ProxyPass* a *ProxyPassReverse* jsou povinné.

Můžete konfigurovat protokolování pro jednotlivé **VirtualHost** pomocí **protokolu chyb** a **CustomLog** direktivy. **Protokolu chyb** je umístění, kam server protokoluje chyby, a **CustomLog** nastaví název souboru a formát souboru protokolu. V takovém případě je kde požadavku je do něj protokolují informace. Je jeden řádek pro každý požadavek.

Uložte tento soubor a otestovat konfiguraci. Pokud vše projde, odpověď by měla být `Syntax [OK]`.

```bash
sudo service httpd configtest
```

Restartujte Apache:

```bash
sudo systemctl restart httpd
sudo systemctl enable httpd
```

## <a name="monitoring-the-app"></a>Monitorování aplikace

Apache je teď instalace předávat požadavky na `http://localhost:80` systémem Kestrel v aplikaci ASP.NET Core `http://127.0.0.1:5000`.  Apache není však nastavit ke správě procesu Kestrel. Použití *systemd* a vytvořte soubor služby spuštění a sledování základní webové aplikace. *systemd* je init systém, který poskytuje mnoho výkonné funkce pro spouštění, zastavování a Správa procesů. 


### <a name="create-the-service-file"></a>Vytvoření souboru služby

Vytvořte soubor definice služby:

```bash
sudo nano /etc/systemd/system/kestrel-hellomvc.service
```

Soubor službu příkladu pro aplikaci:

```
[Unit]
Description=Example .NET Web API App running on CentOS 7

[Service]
WorkingDirectory=/var/aspnetcore/hellomvc
ExecStart=/usr/local/bin/dotnet /var/aspnetcore/hellomvc/hellomvc.dll
Restart=always
# Restart service after 10 seconds if dotnet service crashes
RestartSec=10
SyslogIdentifier=dotnet-example
User=apache
Environment=ASPNETCORE_ENVIRONMENT=Production 

[Install]
WantedBy=multi-user.target
```

> [!NOTE]
> **Uživatel** &mdash; Pokud uživatel *apache* nepoužívá konfigurace, musí být uživatel nejprve a pro soubory zadané správné vlastnictví.

Uložte tento soubor a povolení služby:

```bash
systemctl enable kestrel-hellomvc.service
```

Spusťte službu a ověřte, zda je spuštěna:

```bash
systemctl start kestrel-hellomvc.service
systemctl status kestrel-hellomvc.service

● kestrel-hellomvc.service - Example .NET Web API App running on CentOS 7
    Loaded: loaded (/etc/systemd/system/kestrel-hellomvc.service; enabled)
    Active: active (running) since Thu 2016-10-18 04:09:35 NZDT; 35s ago
Main PID: 9021 (dotnet)
    CGroup: /system.slice/kestrel-hellomvc.service
            └─9021 /usr/local/bin/dotnet /var/aspnetcore/hellomvc/hellomvc.dll
```

S reverzní proxy server nakonfigurovat a spravovat prostřednictvím Kestrel *systemd*, webové aplikace je plně nakonfigurovaná a je přístupný z prohlížeče v místním počítači v `http://localhost`. Probíhá kontrola hlavičky odpovědi **Server** Hlavička uvádí, že je aplikace ASP.NET Core poskytovaný Kestrel:

```
HTTP/1.1 200 OK
Date: Tue, 11 Oct 2016 16:22:23 GMT
Server: Kestrel
Keep-Alive: timeout=5, max=98
Connection: Keep-Alive
Transfer-Encoding: chunked
```

### <a name="viewing-logs"></a>Prohlížení protokolů

Od webové aplikace pomocí Kestrel je spravovat pomocí *systemd*, procesy a událostí jsou zaznamenány do centralizované deníku. Ale tento deník obsahuje položky pro všechny služby a spravuje procesy *systemd*. Chcete-li zobrazit `kestrel-hellomvc.service`-konkrétní položky, použijte následující příkaz:

```bash
sudo journalctl -fu kestrel-hellomvc.service
```

Pro filtrování podle času, zadejte čas možnosti pomocí příkazu. Například použít `--since today` filtrovat aktuální den nebo `--until 1 hour ago` zobrazíte položky do předchozí hodiny. Další informace najdete v tématu [man stránky pro journalctl](https://www.unix.com/man-page/centos/1/journalctl/).

```bash
sudo journalctl -fu kestrel-hellomvc.service --since "2016-10-18" --until "2016-10-18 04:00"
```

## <a name="securing-the-app"></a>Zabezpečení aplikací

### <a name="configure-firewall"></a>Konfigurace brány firewall

*Firewalld* je dynamická démon ke správě brány firewall s podporou zón sítě. Porty a filtrování paketů můžete dál spravovat přes iptables. *Firewalld* by měly být nainstalovány ve výchozím nastavení. `yum`slouží k instalaci balíčku nebo ověřte, zda že je nainstalovaná.

```bash
sudo yum install firewalld -y
```

Použití `firewalld` k otevření pouze portů, které jsou potřebné pro aplikaci. V takovém případě je to port 80 a 443 jsou používány. Následující příkazy trvale nastavte porty 80 a 443 otevřete:

```bash
sudo firewall-cmd --add-port=80/tcp --permanent
sudo firewall-cmd --add-port=443/tcp --permanent
```

Znovu načte nastavení brány firewall. Zkontrolujte dostupné služby a porty ve výchozí zóně. Možnosti jsou k dispozici zkontrolováním `firewall-cmd -h`.

```bash 
sudo firewall-cmd --reload
sudo firewall-cmd --list-all
```

```bash
public (default, active)
interfaces: eth0
sources: 
services: dhcpv6-client
ports: 443/tcp 80/tcp
masquerade: no
forward-ports: 
icmp-blocks: 
rich rules: 
```

### <a name="ssl-configuration"></a>Konfigurace protokolu SSL

Ke konfiguraci Apache pro protokol SSL, *mod_ssl* modul se používá. Když *httpd* byl nainstalován modul, *mod_ssl* také nainstalován modul. Pokud nebyla nainstalovaná, použijte `yum` tím ho přidáte do konfigurace.

```bash
sudo yum install mod_ssl
```
Pokud chcete vynutit SSL, nainstalujte `mod_rewrite` modulu, chcete-li povolit přepisování adres URL:

```bash
sudo yum install mod_rewrite
```

Změnit *hellomvc.conf* souboru povolit přepisování adres URL a zabezpečenou komunikaci na portu 443:

```
<VirtualHost *:80>
    RewriteEngine On
    RewriteCond %{HTTPS} !=on
    RewriteRule ^/?(.*) https://%{SERVER_NAME}/ [R,L]
</VirtualHost>

<VirtualHost *:443>
    ProxyPreserveHost On
    ProxyPass / http://127.0.0.1:5000/
    ProxyPassReverse / http://127.0.0.1:5000/
    ErrorLog /var/log/httpd/hellomvc-error.log
    CustomLog /var/log/httpd/hellomvc-access.log common
    SSLEngine on
    SSLProtocol all -SSLv2
    SSLCipherSuite ALL:!ADH:!EXPORT:!SSLv2:!RC4+RSA:+HIGH:+MEDIUM:!LOW:!RC4
    SSLCertificateFile /etc/pki/tls/certs/localhost.crt
    SSLCertificateKeyFile /etc/pki/tls/private/localhost.key
</VirtualHost>
```

> [!NOTE]
> Tento příklad používá certifikát, místně generované. **SSLCertificateFile** by měl být soubor primární certifikát pro název domény. **SSLCertificateKeyFile** by měl být soubor klíče vygenerován při vytváření zástupce oddělení služeb zákazníkům. **SSLCertificateChainFile** by měla být soubor zprostředkující certifikátu (pokud existuje), byl zadán certifikační autoritou.

Uložte tento soubor a otestovat konfiguraci:

```bash
sudo service httpd configtest
```

Restartujte Apache:

```bash
sudo systemctl restart httpd
```

## <a name="additional-apache-suggestions"></a>Další návrhy Apache

### <a name="additional-headers"></a>Další záhlaví

Chcete-li zabezpečení před útoky se zlými úmysly, jsou několik hlavičky, které by měl být změnit nebo přidat. Ujistěte se, že `mod_headers` je modul nainstalován:

```bash
sudo yum install mod_headers
```

#### <a name="secure-apache-from-clickjacking-attacks"></a>Apache zabezpečení před útoky útoků typu clickjacking

[Útoků typu Clickjacking](https://blog.qualys.com/securitylabs/2015/10/20/clickjacking-a-common-implementation-mistake-that-can-put-your-websites-in-danger), také známé jako *uživatelského rozhraní zjednávání nápravy útoku*, je napadením se zlými úmysly, kde je oklame kliknutím na odkaz nebo tlačítko na stránce jiné než aktuálně navštívený, aby návštěvník webu. Použití `X-FRAME-OPTIONS` k zabezpečení webu.

Upravit *httpd.conf* souboru:

```bash
sudo nano /etc/httpd/conf/httpd.conf
```

Přidejte řádek `Header append X-FRAME-OPTIONS "SAMEORIGIN"`. Uložte soubor. Restartujte Apache.

#### <a name="mime-type-sniffing"></a>Sledování toku dat typ MIME

`X-Content-Type-Options` Záhlaví brání aplikaci Internet Explorer z *sledování toku dat MIME* (určení souboru `Content-Type` z obsahu souboru). Pokud je server nastaví `Content-Type` hlavičky k `text/html` s `nosniff` sadu možností, Internet Explorer vykreslí obsah jako `text/html` bez ohledu na jeho obsahu.

Upravit *httpd.conf* souboru:

```bash
sudo nano /etc/httpd/conf/httpd.conf
```

Přidejte řádek `Header set X-Content-Type-Options "nosniff"`. Uložte soubor. Restartujte Apache.

### <a name="load-balancing"></a>Vyrovnávání zatížení 

Tento příklad ukazuje, jak nainstalovat a nakonfigurovat Apache na CentOS 7 a Kestrel na stejném počítači instance. Chcete-li nemá jediný bod selhání; pomocí *mod_proxy_balancer* a úprava **VirtualHost** by umožnilo pro správu několik instancí webové aplikace za proxy serverem Apache.

```bash
sudo yum install mod_proxy_balancer
```

V konfiguračním souboru vidíte níže, další instanci `hellomvc` aplikace je ke spuštění na portu 5001 instalace. *Proxy* části nastavena konfigurace služby Vyrovnávání se dva členy pro vyrovnávání zatížení *byrequests*.

```
<VirtualHost *:80>
    RewriteEngine On
    RewriteCond %{HTTPS} !=on
    RewriteRule ^/?(.*) https://%{SERVER_NAME}/ [R,L]
</VirtualHost>

<VirtualHost *:443>
    ProxyPass / balancer://mycluster/ 

    ProxyPassReverse / http://127.0.0.1:5000/
    ProxyPassReverse / http://127.0.0.1:5001/

    <Proxy balancer://mycluster>
        BalancerMember http://127.0.0.1:5000
        BalancerMember http://127.0.0.1:5001 
        ProxySet lbmethod=byrequests
    </Proxy>

    <Location />
        SetHandler balancer
    </Location>
    ErrorLog /var/log/httpd/hellomvc-error.log
    CustomLog /var/log/httpd/hellomvc-access.log common
    SSLEngine on
    SSLProtocol all -SSLv2
    SSLCipherSuite ALL:!ADH:!EXPORT:!SSLv2:!RC4+RSA:+HIGH:+MEDIUM:!LOW:!RC4
    SSLCertificateFile /etc/pki/tls/certs/localhost.crt
    SSLCertificateKeyFile /etc/pki/tls/private/localhost.key
</VirtualHost>
```

### <a name="rate-limits"></a>Omezení přenosové rychlosti
Pomocí *mod_ratelimit*, který je součástí *httpd* modulu, šířku pásma klientů může být omezen:

```bash
sudo nano /etc/httpd/conf.d/ratelimit.conf
```
Příklad souboru omezení šířky pásma jako 600 KB/s v části umístění kořenového adresáře:

```
<IfModule mod_ratelimit.c>
    <Location />
        SetOutputFilter RATE_LIMIT
        SetEnv rate-limit 600
    </Location>
</IfModule>
```