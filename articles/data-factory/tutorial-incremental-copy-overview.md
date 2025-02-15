---
title: Inkrementelles Kopieren von Daten aus einem Quelldatenspeicher in einen Zieldatenspeicher
description: In diesen Tutorials wird veranschaulicht, wie Sie Daten inkrementell aus einem Quelldatenspeicher in einen Zieldatenspeicher kopieren. Im ersten Tutorial werden Daten aus einer Tabelle kopiert.
author: dearandyxu
ms.author: yexu
ms.service: data-factory
ms.subservice: tutorials
ms.topic: tutorial
ms.custom: seo-lt-2019
ms.date: 02/18/2021
ms.openlocfilehash: 543acb129d23a0b74434535306aca801d2f5fdd2
ms.sourcegitcommit: 0770a7d91278043a83ccc597af25934854605e8b
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 09/13/2021
ms.locfileid: "124763631"
---
# <a name="incrementally-load-data-from-a-source-data-store-to-a-destination-data-store"></a>Inkrementelles Laden von Daten aus einem Quelldatenspeicher in einen Zieldatenspeicher

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

In einer Datenintegrationslösung ist das inkrementelle Laden (oder Deltaladen) von Daten nach einem anfänglichen vollständigen Ladevorgang ein häufig verwendetes Szenario. In den Tutorials dieses Abschnitts werden verschiedene Möglichkeiten zum inkrementellen Laden von Daten mit Azure Data Factory gezeigt.

## <a name="delta-data-loading-from-database-by-using-a-watermark"></a>Laden von Deltadaten aus der Datenbank mit einem Grenzwert
In diesem Fall definieren Sie einen Grenzwert (Englisch: Watermark) in Ihrer Quelldatenbank. Der Grenzwert ist hier eine Spalte, die den Zeitstempel der letzten Aktualisierung oder einen Inkrementierungsschlüssel enthält. Mit einer Lösung für das Deltaladen werden die geänderten Daten geladen, die zwischen einem alten und einem neuen Grenzwert liegen. Der Workflow für diesen Ansatz ist im folgenden Diagramm dargestellt: 

:::image type="content" source="media/tutorial-incremental-copy-overview/workflow-using-watermark.png" alt-text="Workflow für die Verwendung von Grenzwerten":::

Die folgenden Tutorials enthalten Schritt-für-Schritt-Anleitungen: 
- [Inkrementelles Kopieren von Daten aus einer Tabelle in Azure SQL-Datenbank in Azure Blob Storage](tutorial-incremental-copy-powershell.md)
- [Inkrementelles Kopieren von Daten aus mehreren Tabellen einer SQL Server-Instanz in Azure SQL-Datenbank](tutorial-incremental-copy-multiple-tables-powershell.md)

Informationen zu Vorlagen finden Sie im folgenden Artikel:
- [Deltakopiervorgänge aus einer Datenbank mit einer Steuertabelle](solution-template-delta-copy-with-control-table.md)

## <a name="delta-data-loading-from-sql-db-by-using-the-change-tracking-technology"></a>Laden von Deltadaten aus SQL-Datenbank unter Verwendung der Technologie für die Änderungsnachverfolgung
Die Technologie für die Änderungsnachverfolgung ist eine einfache Lösung in SQL Server und Azure SQL-Datenbank, die über einen effizienten Mechanismus für die Änderungsnachverfolgung für Anwendungen enthält. Hiermit kann eine Anwendung auf einfache Weise Daten identifizieren, die eingefügt, aktualisiert oder gelöscht wurden. 

Der Workflow für diesen Ansatz ist im folgenden Diagramm dargestellt:

:::image type="content" source="media/tutorial-incremental-copy-overview/workflow-using-change-tracking.png" alt-text="Workflow für die Verwendung der Änderungsnachverfolgung":::

Das folgende Tutorial enthält eine Schritt-für-Schritt-Anleitung: <br/>
- [Inkrementelles Laden von Daten aus Azure SQL-Datenbank in Azure Blob Storage mit Informationen der Änderungsnachverfolgung](tutorial-incremental-copy-change-tracking-feature-powershell.md)

## <a name="loading-new-and-changed-files-only-by-using-lastmodifieddate"></a>Ausschließliches Laden neuer und geänderter Dateien unter Verwendung von „LastModifiedDate“
Sie können die neuen und geänderten Dateien nur kopieren, indem Sie „LastModifiedDate“ für den Zielspeicher verwenden. ADF überprüft alle Dateien aus dem Quellspeicher, wendet den Filter auf deren „LastModifiedDate“ an und kopiert nur die Dateien in den Zielspeicher, die neu sind oder seit dem letzten Mal aktualisiert wurden.  Beachten Sie bitte Folgendes: Wenn Sie von ADF große Mengen von Dateien überprüfen lassen, aber nur wenige Dateien in das Ziel kopieren, dauert dies aufgrund des Überprüfungsvorgangs weiterhin lange.   

Das folgende Tutorial enthält eine Schritt-für-Schritt-Anleitung: <br/>
- [Incrementally copy new and changed files based on LastModifiedDate by using the Copy Data tool](tutorial-incremental-copy-lastmodified-copy-data-tool.md) (Inkrementelles Kopieren neuer und geänderter Dateien auf der Grundlage von „LastModifiedDate“ mithilfe des Tools zum Kopieren von Daten)

Informationen zu Vorlagen finden Sie im folgenden Artikel:
- [Kopieren neuer Dateien basierend auf LastModifiedDate](solution-template-copy-new-files-lastmodifieddate.md)

## <a name="loading-new-files-only-by-using-time-partitioned-folder-or-file-name"></a>Ausschließliches Laden neuer Dateien unter Verwendung zeitpartitionierter Ordner- oder Dateinamen
Sie können den Kopiervorgang auf neue Dateien beschränken, wenn Datei- oder Ordnernamen Zeitangaben zur zeitlichen Partitionierung enthalten (Beispiel: /jjjj/mm/tt/Datei.csv). Dies ist der leistungsfähigste Ansatz für inkrementelles Laden neuer Dateien. 

Das folgende Tutorial enthält eine Schritt-für-Schritt-Anleitung: <br/>
- [Incrementally copy new files based on time partitioned file name by using the Copy Data tool](tutorial-incremental-copy-partitioned-file-name-copy-data-tool.md) (Inkrementelles Kopieren neuer Dateien auf der Grundlage zeitpartitionierter Dateinamen mithilfe des Tools zum Kopieren von Daten)

## <a name="next-steps"></a>Nächste Schritte
Fahren Sie mit dem folgenden Tutorial fort: 

> [!div class="nextstepaction"]
>[Inkrementelles Kopieren von Daten aus einer Tabelle in Azure SQL-Datenbank in Azure Blob Storage](tutorial-incremental-copy-powershell.md)
