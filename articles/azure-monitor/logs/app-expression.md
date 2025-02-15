---
title: app()-Ausdruck in Azure Monitor-Protokollabfragen | Microsoft-Dokumentation
description: Der app-Ausdruck wird in Azure Monitor-Protokollabfragen verwendet, um Daten aus einer bestimmten Application Insights-App in derselben Ressourcengruppe, einer anderen Ressourcengruppe oder einem anderen Abonnement abzurufen.
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 08/11/2021
ms.openlocfilehash: 1c7659d8b566649291e135c68c677b3f3a074d9f
ms.sourcegitcommit: d43193fce3838215b19a54e06a4c0db3eda65d45
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 08/20/2021
ms.locfileid: "122515469"
---
# <a name="app-expression-in-azure-monitor-query"></a>app()-Ausdruck in Azure Monitor-Abfragen

Der `app`-Ausdruck wird in Azure Monitor-Abfragen verwendet, um Daten aus einer bestimmten Application Insights-Anwendung in der gleichen Ressourcengruppe, einer anderen Ressourcengruppe oder einem anderen Abonnement abzurufen. Dies ist nützlich, um Anwendungsdaten in eine Azure Monitor-Protokollabfrage einzuschließen und Daten in mehreren Anwendungen in einer Application Insights-Abfrage abzufragen.

> [!IMPORTANT]
> Der app()-Ausdruck wird nicht verwendet, wenn Sie eine [arbeitsbereichsbasierte Application Insights-Ressource](../app/create-workspace-resource.md) verwenden, da Protokolldaten in einem Log Analytics-Arbeitsbereich gespeichert werden. Verwenden Sie den workspace()-Ausdruck, um eine Abfrage zu schreiben, die Anwendungen in mehreren Arbeitsbereichen umfasst. Für mehrere Anwendungen im gleichen Arbeitsbereich benötigen Sie keine arbeitsbereichübergreifende Abfrage.

## <a name="syntax"></a>Syntax

`app(`*Bezeichner*`)`


## <a name="arguments"></a>Argumente

- *Bezeichner*: Identifiziert die Anwendung mit einem der Formate in der folgenden Tabelle.

| Bezeichner | BESCHREIBUNG | Beispiel
|:---|:---|:---|
| Ressourcenname | Für Menschen lesbarer Name der Anwendung (auch als „Komponentenname“ bezeichnet) | app("fabrikamapp") |
| Qualifizierter Name | Vollständiger Name der Anwendung in der Form: „Abonnementname/Ressourcengruppe/Komponentenname“ | app('AI-Prototype/Fabrikam/fabrikamapp') |
| id | GUID der Anwendung | app("988ba129-363e-4415-8fe7-8cbab5447518") |
| Azure-Ressourcen-ID | Bezeichner für die Azure-Ressource |app("/subscriptions/7293b69-db12-44fc-9a66-9c2005c3051d/resourcegroups/Fabrikam/providers/microsoft.insights/components/fabrikamapp") |


## <a name="notes"></a>Notizen

* Sie benötigen Lesezugriff auf die Anwendung.
* Bei der Identifizierung einer Anwendung über ihren Namen wird vorausgesetzt, dass er in allen zugänglichen Abonnements eindeutig ist. Sollten mehrere Anwendungen mit dem angegebenen Namen vorhanden sein, ist die Abfrage aufgrund der Mehrdeutigkeit nicht erfolgreich. In diesem Fall muss einer der anderen Bezeichner verwendet werden.
* Verwenden Sie den verwandten Ausdruck [workspace](../logs/workspace-expression.md) für übergreifende Abfragen über Log Analytics-Arbeitsbereiche.
* Der app()-Ausdruck wird derzeit bei Verwendung des Azure-Portals nur dann zum Erstellen einer [benutzerdefinierten Warnungsregel für Protokollabfragen](../alerts/alerts-log.md) in der Protokollabfrage unterstützt, wenn als Ressource für die Warnungsregel eine Application Insights-Anwendung verwendet wird.

## <a name="examples"></a>Beispiele

```Kusto
app("fabrikamapp").requests | count
```
```Kusto
app("AI-Prototype/Fabrikam/fabrikamapp").requests | count
```
```Kusto
app("b438b4f6-912a-46d5-9cb1-b44069212ab4").requests | count
```
```Kusto
app("/subscriptions/7293b69-db12-44fc-9a66-9c2005c3051d/resourcegroups/Fabrikam/providers/microsoft.insights/components/fabrikamapp").requests | count
```
```Kusto
union 
(workspace("myworkspace").Heartbeat | where Computer contains "Con"),
(app("myapplication").requests | where cloud_RoleInstance contains "Con")
| count  
```
```Kusto
union 
(workspace("myworkspace").Heartbeat), (app("myapplication").requests)
| where TimeGenerated between(todatetime("2018-02-08 15:00:00") .. todatetime("2018-12-08 15:05:00"))
```

## <a name="next-steps"></a>Nächste Schritte

- Informationen zum Verweisen auf Log Analytics-Arbeitsbereiche finden Sie unter [workspace-Ausdruck](../logs/workspace-expression.md).
- Erfahren Sie mehr zum Speichern von [Azure Monitor-Daten](./log-query-overview.md).
- Greifen Sie auf die vollständige Dokumentation für die [Abfragesprache Kusto](/azure/kusto/query/) zu.
