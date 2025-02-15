---
title: Deduplizieren von Zeilen und Suchen nach NULL-Werten mit Datenfluss-Codeausschnitten
description: Erfahren Sie, wie Sie ganz einfach Zeilen deduplizieren und nach NULL-Werten suchen, indem Sie Codeausschnitte in Datenflüssen verwenden.
author: kromerm
ms.author: makromer
ms.service: data-factory
ms.subservice: tutorials
ms.topic: conceptual
ms.date: 09/30/2020
ms.openlocfilehash: 7940d48edb94bfa89ccc3310172a09519ffc729a
ms.sourcegitcommit: 0770a7d91278043a83ccc597af25934854605e8b
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 09/13/2021
ms.locfileid: "124831357"
---
# <a name="dedupe-rows-and-find-nulls-by-using-data-flow-snippets"></a>Deduplizieren von Zeilen und Suchen nach NULL-Werten mit Datenfluss-Codeausschnitten

[!INCLUDE [appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

Mithilfe von Codeausschnitten in Zuordnungsdatenflüssen können Sie häufige Aufgaben wie die Datendeduplizierung oder die Filterung nach NULL-Werten problemlos durchführen. In diesem Artikel wird beschrieben, wie Sie diese Funktionen problemlos Ihren Pipelines hinzufügen können, indem Sie Codeausschnitte von Datenflussskripts verwenden.
<br>
> [!VIDEO https://www.microsoft.com/en-us/videoplayer/embed/RE4GnhH]

## <a name="create-a-pipeline"></a>Erstellen einer Pipeline

1. Wählen Sie **Neue Pipeline** aus.

1. Fügen Sie eine Datenflussaktivität hinzu.

1. Wählen Sie die Registerkarte **Quelleinstellungen** aus, fügen Sie eine Quelltransformation hinzu, und verbinden Sie diese dann mit einem Ihrer Datasets.

    :::image type="content" source="media/data-flow/snippet-adf-2.png" alt-text="Screenshot des Bereichs &quot;Quelleinstellungen&quot; zum Hinzufügen eines Quelltyps":::

    In den Codeausschnitten für das Deduplizieren und das Prüfen auf NULL-Werte werden generische Muster verwendet, die die Datendrift des Datenflussschemas nutzen. Sie können die Codeausschnitte mit jedem beliebigen Schema aus Ihrem Dataset und sogar mit Datasets verwenden, die nicht über ein vordefiniertes Schema verfügen.

1. Kopieren Sie im Abschnitt „Eindeutige Zeile mit allen Spalten“ im [Datenflussskript (DFS)](./data-flow-script.md#distinct-row-using-all-columns) den Codeausschnitt für DistinctRows.

1. [Navigieren Sie zur entsprechenden Seite in der Datenflussskript-Dokumentation, und kopieren Sie den Codeausschnitt für „Distinct Rows“](./data-flow-script.md#distinct-row-using-all-columns).

    :::image type="content" source="media/data-flow/snippet-adf-3.png" alt-text="Screenshot eines Quellcodeausschnitts":::

1. Drücken Sie nach der Definition für `source1` in Ihrem Skript die EINGABETASTE, und fügen Sie den Codeausschnitt ein.

1. Führen Sie eine der folgenden Aktionen aus:

   * Verbinden Sie diesen eingefügten Codeausschnitt mit der Quelltransformation, die Sie im Graphen erstellt haben, indem Sie vor dem eingefügten Code **source1** eingeben.

   * Alternativ können Sie für die neue Transformation im Designer eine Verbindung herstellen, indem Sie über den neuen Transformationsknoten im Graphen den eingehenden Datenstrom auswählen.

     :::image type="content" source="media/data-flow/snippet-adf-4.png" alt-text="Screenshot des Einstellungsbereichs &quot;Bedingtes Teilen&quot;":::

   In Ihrem Datenfluss werden nun doppelte Zeilen mithilfe der Aggregattransformation aus Ihrer Quelle entfernt. Damit wird nach allen Zeilen gruppiert, indem übergreifend für alle Spaltenwerte ein allgemeiner Hash genutzt wird.
    
1. Fügen Sie einen Codeausschnitt zum Aufteilen Ihrer Daten in einen Datenstrom mit NULL-Werten und einen anderen Datenstrom ohne NULL-Werte hinzu. Gehen Sie folgendermaßen vor:

1. [Navigieren Sie zurück zur Bibliothek mit den Codeausschnitten, und kopieren Sie nun den Code für die Überprüfung auf NULL-Werte](./data-flow-script.md#check-for-nulls-in-all-columns).

   b. Wählen Sie im Datenfluss-Designer erneut **Skript** aus, und fügen Sie dann ganz unten den neuen Transformationscode unten ein. Durch diese Aktion wird das Skript mit der vorherigen Transformation verbunden, indem der Name dieser Transformation vor dem eingefügten Codeausschnitt eingefügt wird.

   Ihr Datenflussgraph sollte jetzt in etwa wie folgt aussehen:

    :::image type="content" source="media/data-flow/snippet-adf-1.png" alt-text="Screenshot des Datenflussgraphen":::

  Sie haben damit einen funktionierenden Datenfluss mit generischer Deduplizierung und einer Überprüfung auf NULL-Werte erstellt, indem Sie vorhandene Codeausschnitte aus der Datenflussskript-Bibliothek in Ihren vorhandenen Entwurf eingefügt haben.

## <a name="next-steps"></a>Nächste Schritte

* Erstellen Sie die restliche Datenflusslogik mithilfe von Mapping Data Flow-[Transformationen](concepts-data-flow-overview.md).