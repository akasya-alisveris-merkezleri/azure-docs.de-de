---
title: Übersicht über die Azure Relay-API | Microsoft-Dokumentation
description: Dieser Artikel bietet eine Übersicht über verfügbare Azure Relay-APIs (.NET Standard, .NET Framework, Node.js usw.).
ms.topic: article
ms.custom: devx-track-dotnet
ms.date: 06/23/2021
ms.openlocfilehash: 3c49ec625469782fa13a2dee056f51242665a7de
ms.sourcegitcommit: d9a2b122a6fb7c406e19e2af30a47643122c04da
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 07/24/2021
ms.locfileid: "114666232"
---
# <a name="available-relay-apis"></a>Verfügbare Relay-APIs

## <a name="runtime-apis"></a>Laufzeit-APIs

In der folgenden Tabelle sind alle derzeit verfügbaren Relay-Laufzeitclients aufgeführt.

Der Abschnitt [Zusätzliche Informationen](#additional-information) enthält weitere Informationen zum Status der einzelnen Laufzeitbibliotheken.

| Sprache/Plattform | Verfügbares Feature | Clientpaket | Repository |
| --- | --- | --- | --- |
| .NET Standard | Hybridverbindungen | [Microsoft.Azure.Relay](https://www.nuget.org/packages/Microsoft.Azure.Relay/) | [GitHub](https://github.com/azure/azure-relay-dotnet) |
| .NET Framework | WCF-Relay | [WindowsAzure.ServiceBus](https://www.nuget.org/packages/WindowsAzure.ServiceBus/) | – |
| Node | Hybridverbindungen | [WebSockets: `hyco-ws`](https://www.npmjs.com/package/hyco-ws)<br/>[WebSockets: `hyco-websocket`](https://www.npmjs.com/package/hyco-websocket)<br/>[HTTP-Anforderungen: `hyco-https`](https://www.npmjs.com/package/hyco-https) | [GitHub](https://github.com/Azure/azure-relay-node) |

### <a name="additional-information"></a>Zusätzliche Informationen

#### <a name="net"></a>.NET

Das .NET-Ökosystem enthält mehrere Runtimes, daher stehen mehrere .NET-Bibliotheken für Relay zur Verfügung. Die .NET Standard-Bibliothek kann entweder mit .NET Core oder .NET Framework ausgeführt werden. Die .NET Framework-Bibliothek kann hingegen nur in einer .NET Framework-Umgebung ausgeführt werden. Weitere Informationen zu .NET Frameworks finden Sie unter [Framework-Versionen](/dotnet/articles/standard/frameworks).

Die .NET Framework-Bibliothek unterstützt nur das WCF-Programmiermodell und verwendet ein proprietäres binäres Protokoll, das auf dem WCF-Transport `net.tcp` basiert. Dieses Protokoll und die Bibliothek werden zur Abwärtskompatibilität mit vorhandenen Anwendungen weiterhin verwaltet.

Die .NET-Standardbibliothek basiert auf der offenen Protokolldefinition für das Hybrid Connections Relay, das wiederum auf HTTP und WebSockets aufbaut. Die Bibliothek unterstützt die Streamabstraktion über WebSockets und eine einfache Anforderungs-/Antwort-API-Geste für die Beantwortung von HTTP-Anforderungen. Das [Web-API](https://github.com/Azure/azure-relay-dotnet)-Beispiel zeigt, wie Hybrid Connections in ASP.NET Core für Webdienste integriert wird.

#### <a name="nodejs"></a>Node.js

Die in der obigen Tabelle aufgeführten Hybrid Connections-Module ersetzen oder ändern die bestehenden Node.js-Module durch alternative Implementierungen, die am Azure Relay-Dienst und nicht am lokalen Netzwerkstapel lauschen.

Das Modul `hyco-https` ändert und überschreibt teilweise die Node.js-Kernmodule `http` und `https`. Es stellt eine HTTPS-Listenerimplementierung bereit, die mit vielen vorhandenen Node.js-Modulen und -Anwendungen kompatibel ist, die diese Kernmodule verwenden.

Die Module `hyco-ws` und `hyco-websocket` ändern die verbreiteten Node.js-Module `ws` und `websocket`. Sie stellen alternative Listenerimplementierungen bereit, die Module und Anwendungen möglich machen, für die eines der Module hinter dem Hybrid Connections Relay arbeiten muss.

Ausführliche Informationen zu diesen Modulen finden Sie im GitHub-Repository [azure-relay-node](https://github.com/Azure/azure-relay-node).

## <a name="next-steps"></a>Nächste Schritte

Weitere Informationen zu Azure Relay finden Sie unter diesen Links:
* [Was ist Azure Relay?](relay-what-is-it.md)
* [Relay – häufig gestellte Fragen](relay-faq.yml)
