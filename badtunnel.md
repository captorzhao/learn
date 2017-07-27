本文提出了一种新的攻击模型，可以跨网段劫持TCP/IP广播协议，我们把它命名为“BadTunnel”。

利用这种方法，可以实现跨网段的NetBIOS Name Service Spoofing攻击。无论攻击者和用户是否在同一网段，甚至中间存在防火墙或NAT，只要用户打开IE或Edge浏览器访问一个恶意页面，或打开一个特殊构造的Office文档，攻击者就可以劫持用户系统对任意NetBIOS名称的解析，从而实现仿冒本地网络的打印服务器、文件服务器等。

通过劫持“WPAD”名称，还可以进一步实现劫持用户的所有网络通信，包括一般网络访问，和Windows Update service以及Microsoft Crypto API 更新Certificaterevocation list的通信等。而一旦能劫持网络通信，配合类似Evilgrade的工具（参考链接[1]），也很容易在系统上运行任意程序。

可以通过所有版本的IE和Edge、所有版本的MS Office、以及大量第三方软件触发。事实上只要存在能嵌入file URI scheme或UNC path的地方，就可以触发BadTunnel攻击。如果在一个快捷方式中将图标路径设置为恶意file URI scheme或UNC path，只要用户在资源管理器看见这个快捷方式，就会触发BadTunnel攻击。所以BadTunnel可以通过网页、邮件、U盘等多种手段进行利用。甚至还可能威胁WEB服务器和SQL服务器等（参考链接[2]）

背景知识

NetBIOS是一套古老的协议。1987年IETF发布RFC 1001与RFC 1002，定义了NetBIOS over TCP/IP，简称NBT。NetBIOS包含三种服务，其中之一是名称服务（Name service），即NetBIOS-NS，简称NBNS。NBNS可以通过发送局域网内广播来实现本地名称解析。

当你试图访问\\Tencent\XuanwuLab\tk.txt时，NBNS会向广播地址发出NBNS NB query：

谁是“Tencent”？

而本地局域网内的任何主机都可以回应：

192.168.2.9是“Tencent”。

然后你的电脑就会接受这个回应，然后去访问\\192.168.2.9\XuanwuLab\tk.txt。

这套机制谈不上安全，但由于发生在局域网内，而局域网通常被认为是相对可信的环境。所以虽然很早就有人意识到可以在局域网内假冒任意主机，但这并不被认为是漏洞——就像ARP Spoofing并不被认为是漏洞一样。

WPAD（Web Proxy Auto-Discovery Protocol）是另一套有超过二十年历史的古老协议，用于自动发现和配置系统的代理服务器。几乎所有操作系统都支持WPAD，但只有Windows系统默认启用这个协议。按照WPAD协议，系统会试图访问http://WPAD/wpad.dat，以获取代理配置脚本。

在Windows上，对“WPAD”这个名称的请求很自然会由NBNS来处理。而如前所述，在局域网内，任何主机都可以声称自己是“WPAD”。所以，这套机制也谈不上安全，但由于同样发生在局域网内，而局域网通常被认为是相对可信的环境，所以虽然十几年前就有人意识到可以在局域网内利用WPAD劫持假冒任意主机，2012年被发现的Flame蠕虫也使用了这种攻击方式，但这并不被认为是漏洞——就像ARP Spoofing并不被认为是漏洞一样。

接下来还得再提一下TCP/IP协议。NBNS是用UDP实现的。UDP协议最主要的特点是无会话。无论是防火墙、NAT还是任何其它网络设备，都无法分辨一个UDP包属于哪个会话。只要网络设备允许IP1:Port1->IP2:Port2，就必然同时允许IP2:Port2->IP1:Port1。

刚才我们说过NBNS使用广播协议，通过向本地广播地址发送查询来实现名称解析。但NBNS和绝大多数使用广播协议的应用一样，并不会拒绝来自本网段之外的回应。也就是说，如果192.168.2.2向192.168.2.255发送了一个请求，而10.10.10.10及时返回了一个回应，也会被192.168.2.2接受。在某些企业网络里，这个特性是网络结构所需要的。

1] Evilgrade
https://github.com/infobyte/evilgrade/

[2] 10 Places to Stick Your UNC Path
https://blog.netspi.com/10-places-to-stick-your-unc-path/

[3] Web Proxy Auto-Discovery Protocol
http://tools.ietf.org/html/draft-ietf-wrec-wpad-01

[4] NetBIOS Over TCP/IP
https://technet.microsoft.com/en-us/library/cc940063.aspx

[5] Disable WINS/NetBT name resolution
https://technet.microsoft.com/en-us/library/cc782733(v=ws.10).aspx

[6] MS99-054, CVE-1999-0858
https://technet.microsoft.com/en-us/library/security/ms99-054.aspx

[7] MS09-008, CVE-2009-0093, CVE-2009-0094
https://technet.microsoft.com/en-us/library/security/ms09-008.aspx

[8] MS12-074, CVE-2012-4776
https://technet.microsoft.com/en-us/library/security/ms12-074.aspx

[9]MS16-063, CVE-2016-3213
https://technet.microsoft.com/en-us/library/security/ms16-063.aspx

[10]MS16-077, CVE-2016-3213, CVE-2016-3236
https://technet.microsoft.com/en-us/library/security/ms16-077.aspx