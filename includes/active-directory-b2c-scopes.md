---
author: kengaderdus
ms.service: active-directory-b2c
ms.subservice: B2C
ms.topic: include
ms.date: 10/16/2019
ms.author: kengaderdus
ms.openlocfilehash: 9567ec7c71445fb50c19a5a87846dab0a6eef0f0
ms.sourcegitcommit: 91915e57ee9b42a76659f6ab78916ccba517e0a5
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 10/15/2021
ms.locfileid: "130038112"
---
#### <a name="app-registrations"></a>[App-Registrierungen](#tab/app-reg-ga/) 

1. Wählen Sie **App-Registrierungen** aus.
1. Wählen Sie die Anwendung *webapi1* aus, um ihre **Übersicht**-Seite zu öffnen.
1. Wählen Sie unter **Verwalten** die Option **Eine API verfügbar machen** aus.
1. Wählen Sie neben **Anwendungs-ID-URI** den Link **Festlegen** aus.
1. Ersetzen Sie den Standardwert (eine GUID) durch `api`, und wählen Sie dann **Speichern** aus. Der vollständige URI wird angezeigt und sollte das Format `https://your-tenant-name.onmicrosoft.com/api` aufweisen. Wenn Ihre Webanwendung ein Zugriffstoken für die API anfordert, sollte sie diesen URI als Präfix für jeden Bereich hinzufügen, den Sie für die API definieren.
1. Wählen Sie unter **Durch diese API definierte Bereiche** die Option **Bereich hinzufügen** aus.
1. Geben Sie die folgenden Werte ein, um einen Bereich zu erstellen, der Lesezugriff auf die API definiert, und wählen Sie dann **Bereich hinzufügen** aus:
    1. **Bereichsname**: `demo.read`
    1. **Anzeigename der Administratoreinwilligung**: `Read access to demo API`
    1. **Beschreibung der Administratoreinwilligung**: `Allows read access to the demo API`
1. Wählen Sie **Bereich hinzufügen** aus, geben Sie die folgenden Werte ein, um einen Bereich hinzuzufügen, der Schreibzugriff auf die API definiert, und wählen Sie dann **Bereich hinzufügen** aus:
    1. **Bereichsname**: `demo.write`
    1. **Anzeigename der Administratoreinwilligung**: `Write access to demo API`
    1. **Beschreibung der Administratoreinwilligung**: `Allows write access to the demo API`

#### <a name="applications-legacy"></a>[Anwendungen (Legacy)](#tab/applications-legacy/)

1. Wählen Sie **Anwendungen (Legacy)** aus.
1. Wählen Sie die Anwendung *webapi1* aus, um ihre Seite mit den **Eigenschaften** zu öffnen.
1. Wählen Sie **Veröffentlichte Bereiche** aus. Mit den veröffentlichten Bereichen können einer Clientanwendung bestimmte Berechtigungen für die Web-API gewährt werden.
1. Geben Sie unter **BEREICH** den Bereich `demo.read` und als **BESCHREIBUNG**`Read access to the web API` ein.
1. Geben Sie unter **BEREICH** den Bereich `demo.write` und als **BESCHREIBUNG**`Write access to the web API` ein.
1. Wählen Sie **Speichern** aus.