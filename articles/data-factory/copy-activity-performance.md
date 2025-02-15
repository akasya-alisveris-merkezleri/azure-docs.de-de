---
title: Handbuch zur Leistung und Skalierbarkeit der Kopieraktivität
titleSuffix: Azure Data Factory & Azure Synapse
description: Hier erfahren Sie, welche Faktoren sich entscheidend auf die Leistung auswirken, wenn Sie in Azure Data Factory- und Azure Synapse Analytics-Pipelines Daten mithilfe der Kopieraktivität verschieben.
services: data-factory
documentationcenter: ''
ms.author: jianleishen
author: jianleishen
manager: shwang
ms.service: data-factory
ms.subservice: data-movement
ms.workload: data-services
ms.topic: conceptual
ms.custom: synapse
ms.date: 09/09/2021
ms.openlocfilehash: c66e7a1d19ecf392c5f990bcaa31ba506cf0d7f2
ms.sourcegitcommit: 0770a7d91278043a83ccc597af25934854605e8b
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 09/13/2021
ms.locfileid: "124767411"
---
# <a name="copy-activity-performance-and-scalability-guide"></a>Handbuch zur Leistung und Skalierbarkeit der Kopieraktivität

> [!div class="op_single_selector" title1="Wählen Sie die von Ihnen verwendete Version von Azure Data Factory aus:"]
> * [Version 1](v1/data-factory-copy-activity-performance.md)
> * [Aktuelle Version](copy-activity-performance.md)

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

Gelegentlich möchten Sie eine umfangreiche Datenmigration von Data Lake oder Enterprise Data Warehouse (EDW) zu Azure durchführen. Ein anderes Mal möchten Sie für Big Data-Analysen große Datenmengen aus verschiedenen Quellen in Azure erfassen. In jedem Fall ist es entscheidend, eine optimale Leistung und Skalierbarkeit zu erreichen.

Azure Data Factory- und Azure Synapse Analytics-Pipelines bieten einen Mechanismus zum Erfassen von Daten mit den folgenden Vorteilen:

* Verarbeitet große Datenmengen
* Ist äußerst leistungsfähig
* Ist kostengünstig

Dadurch können technische Fachkräfte für Daten skalierbare und leistungsstarke Pipelines für die Datenerfassung erstellen.

Nach dem Lesen dieses Artikels können Sie die folgenden Fragen beantworten:

* Welche Leistung und Skalierbarkeit kann ich mit der Kopieraktivität in Datenmigrations- und Datenerfassungsszenarios erzielen?
* Welche Schritte muss ich ausführen, um die Leistung der Kopieraktivität zu optimieren?
* Welche Leistungsoptimierungen kann ich für eine einzelne Kopieraktivitätsausführung nutzen?
* Welche anderen externen Faktoren müssen bei der Optimierung der Kopierleistung berücksichtigt werden?

> [!NOTE]
> Falls Sie mit dem grundlegenden Konzept der Kopieraktivität nicht vertraut sind, finden Sie weitere Informationen unter [Kopieraktivität – Übersicht](copy-activity-overview.md), bevor Sie sich mit diesem Artikel beschäftigen.

## <a name="copy-performance-and-scalability-achievable-using-azure-data-factory-and-synapse-pipelines"></a>Mit Azure Data Factory- und Synapse-Pipelines erreichbare Kopierleistung und Skalierbarkeit

Azure Data Factory- und Synapse-Pipelines bieten eine serverlose Architektur, die Parallelität auf verschiedenen Ebenen ermöglicht.

Diese Architektur ermöglicht es Ihnen, Pipelines zu entwickeln, die den Datenverschiebungsdurchsatz für Ihre Umgebung maximieren. Diese Pipelines nutzen die folgenden Ressourcen vollständig:

* Netzwerkbandbreite zwischen den Quell- und Zieldatenspeichern
* Eingabe/Ausgabe-Vorgänge pro Sekunde (IOPS) und Bandbreite des Quell- oder Zieldatenspeichers

Diese volle Auslastung bedeutet, dass Sie den Gesamtdurchsatz durch Messung des Mindestdurchsatzes einschätzen können, der mit den folgenden Ressourcen zur Verfügung steht:

* Quelldatenspeicher
* Zieldatenspeicher
* Netzwerkbandbreite zwischen den Quell- und Zieldatenspeichern

Die folgende Tabelle zeigt die Berechnung der Dauer der Datenverschiebung. Die Dauer in jeder Zelle wird basierend auf einer bestimmten Netzwerk- und Datenspeicherbandbreite und einer bestimmten Größe der Datennutzlast berechnet.

> [!NOTE]
> Die unten angegebene Dauer soll die erreichbare Leistung in einer umfassenden Datenintegrationslösung darstellen. Dazu wird mindestens eine der Leistungsoptimierungstechniken verwendet, die unter [Features für die Leistungsoptimierung von Kopiervorgängen](#copy-performance-optimization-features) beschrieben sind, einschließlich der Verwendung von ForEach zum Partitionieren und Erzeugen mehrerer gleichzeitiger Kopieraktivitäten. Es wird empfohlen, die Schritte unter [Schritte zur Optimierung der Leistung](#performance-tuning-steps) zu befolgen, um die Kopierleistung für Ihr spezifisches Dataset und Ihre Systemkonfiguration zu optimieren. Sie sollten die in den Leistungsoptimierungstests ermittelten Zahlen für die Planung der Produktionsbereitstellung, die Kapazitätsplanung und die Abrechnungsprognose verwenden.

&nbsp;

| Datengröße/ <br/> bandwidth | 50 MBit/s    | 100 MBit/s  | 500 MBit/s  | 1 GBit/s   | 5 GBit/s   | 10 GBit/s  | 50 GBit/s   |
| --------------------------- | ---------- | --------- | --------- | -------- | -------- | -------- | --------- |
| **1 GB**                    | 2,7 min    | 1,4 min   | 0,3 min   | 0,1 min  | 0,03 min | 0,01 min | 0,0 min   |
| **10 GB**                   | 27,3 min   | 13,7 min  | 2,7 min   | 1,3 min  | 0,3 min  | 0,1 min  | 0,03 min  |
| **100 GB**                  | 4,6 h    | 2,3 h   | 0,5 h   | 0,2 h  | 0,05 h | 0,02 h | 0,0 h   |
| **1 TB**                    | 46,6 h   | 23,3 h  | 4,7 h   | 2,3 h  | 0,5 h  | 0,2 h  | 0,05 h  |
| **10 TB**                   | 19,4 Tage  | 9,7 Tage  | 1,9 Tage  | 0,9 Tage | 0,2 Tage | 0,1 Tage | 0,02 Tage |
| **100 TB**                  | 194,2 Tage | 97,1 Tage | 19,4 Tage | 9,7 Tage | 1,9 Tage | 1 Tag    | 0,2 Tage  |
| **1 PB**                    | 64,7 Monate    | 32,4 Monate   | 6,5 Monate    | 3,2 Monate   | 0,6 Monate   | 0,3 Monate   | 0,06 Monate   |
| **10 PB**                   | 647,3 Monate   | 323,6 Monate  | 64,7 Monate   | 31,6 Monate  | 6,5 Monate   | 3,2 Monate   | 0,6 Monate    |
| | |  | | |  | | |

Die Kopieraktivität kann auf verschiedenen Ebenen skaliert werden:

:::image type="content" source="media/copy-activity-performance/adf-copy-scalability.png" alt-text="Skalieren der Kopieraktivität":::

* Mit der Ablaufsteuerung können mehrere Kopieraktivitäten parallel gestartet werden (z. B. per [ForEach-Schleife](control-flow-for-each-activity.md)).

* Eine einzelne Kopieraktivität kann skalierbare Computeressourcen nutzen.
  * Bei Verwendung der Azure Integration Runtime (IR) können Sie [bis zu 256 Datenintegrationseinheiten (DIUs)](#data-integration-units) für jede Kopieraktivität auf serverlose Weise angeben.
  * Bei der Verwendung einer selbst gehosteten IR können Sie einen der folgenden Ansätze verfolgen:
    * Skalieren Sie den Computer manuell hoch.
    * Skalieren Sie auf mehrere Computer ([ bis zu 4 Knoten](create-self-hosted-integration-runtime.md#high-availability-and-scalability)) auf, und eine einzelne Kopieraktivität partitioniert ihre Dateigruppe über alle Knoten.

* Für eine einzelne Kopieraktivität werden mehrere Threads [parallel](#parallel-copy) genutzt, um für den Datenspeicher Lese- und Schreibvorgänge durchzuführen.

## <a name="performance-tuning-steps"></a>Schritte zur Optimierung der Leistung

Führen Sie die folgenden Schritte aus, um die Leistung Ihres Diensts mit der Kopieraktivität zu verbessern:

1. **Wählen Sie ein Testdataset aus, und legen Sie eine Baseline fest.**

    Testen Sie Ihre Pipeline mit der Kopieraktivität während der Entwicklung mit repräsentativen Beispieldaten. Das von Ihnen gewählte Dataset sollte Ihre typischen Datenmuster zusammen mit den folgenden Attributen darstellen:

    * Ordnerstruktur
    * Dateimuster
    * Datenschema

    Und Ihr Dataset sollte ausreichend groß sein, um die Kopierleistung auswerten zu können. Bei einer geeigneten Größe dauert es mindestens 10 Minuten, bis die Kopieraktivität abgeschlossen ist. Sammeln Sie Ausführungsdetails und Leistungsmerkmale, und folgen Sie dabei der [Überwachung der Kopieraktivität](copy-activity-monitoring.md).

2. **So maximieren Sie die Leistung einer einzelnen Kopieraktivität**:

    Wir empfehlen, dass Sie zunächst die Leistung mit einer einzelnen Kopieraktivität maximieren.

    * **Bei Ausführung der Kopieraktivität in einer _Azure_ Integration Runtime:**

        Beginnen Sie mit den Standardwerten für [Datenintegrationseinheiten (DIU)](#data-integration-units) und [Paralleles Kopieren](#parallel-copy).

    * **Bei Ausführung der Kopieraktivität in einer _selbstgehosteten_ Integration Runtime:**

        Es wird empfohlen, IR auf einem dedizierten Computer zu hosten. Der Computer sollte vom Server, auf dem sich der Datenspeicher befindet, getrennt sein. Beginnen Sie mit den Standardwerten für [Paralleles Kopieren](#parallel-copy), und verwenden Sie einen einzelnen Knoten für die selbstgehostete Integration Runtime.

    Führen Sie einen Leistungstestlauf durch. Notieren Sie sich die erzielte Leistung. Schließen Sie die tatsächlich verwendeten Werte ein, z. B. DIUs und parallele Kopien. Einzelheiten zum Erfassen von Testlaufergebnissen und zu verwendeten Leistungseinstellungen finden Sie unter [Überwachen der Kopieraktivität](copy-activity-monitoring.md). Erfahren Sie, wie Sie [Probleme bei der Leistung der Kopieraktivität beheben können](copy-activity-performance-troubleshooting.md), um den Engpass zu identifizieren und zu beheben.

    Führen Sie weitere Leistungstests durch, indem Sie den Leitfaden für die Problembehandlung und die Optimierung verwenden. Wenn für Ausführungen einer einzelnen Kopieraktivität kein besserer Durchsatz mehr erzielt werden kann, sollten Sie eine Maximierung des aggregierten Durchsatzes erwägen, indem Sie mehrere Kopiervorgänge gleichzeitig ausführen. Diese Option wird im nächsten nummerierten Aufzählungspunkt erörtert.

3. **So maximieren Sie den aggregierten Durchsatz durch paralleles Ausführen mehrerer Kopiervorgänge:**

    Mittlerweile haben Sie die Leistung einer einzelnen Kopieraktivität maximiert. Wenn Sie die Durchsatzobergrenzen Ihrer Umgebung noch nicht erreicht haben, können Sie mehrere Kopieraktivitäten parallel ausführen. Mithilfe von Ablaufsteuerungskonstrukten sind parallele Ausführungen möglich. Ein solches Konstrukt ist die [„For Each“-Schleife](control-flow-for-each-activity.md). Weitere Informationen finden Sie in den folgenden Artikeln über Lösungsvorlagen:

    * [Kopieren von Dateien aus mehreren Containern](solution-template-copy-files-multiple-containers.md)
    * [Migrieren von Daten aus Amazon S3 zu ADLS Gen2](solution-template-migration-s3-azure.md)
    * [Massenkopieren mit einer Steuertabelle](solution-template-bulk-copy-with-control-table.md)

4. **Erweitern der Konfiguration auf das gesamte Dataset.**

    Wenn Sie mit den Ausführungsergebnissen und mit der Leistung zufrieden sind, können Sie die Definition und die Pipeline auf Ihr gesamtes Dataset erweitern.

## <a name="troubleshoot-copy-activity-performance"></a>Problembehandlung für die Leistung der Kopieraktivität

Führen Sie die [Schritte zur Optimierung der Leistung](#performance-tuning-steps) aus, um den Leistungstest für Ihr Szenario zu planen und durchzuführen. Informieren Sie sich auch unter [Problembehandlung für die Leistung der Kopieraktivität](copy-activity-performance-troubleshooting.md) darüber, wie Sie Leistungsprobleme bei der Ausführung von einzelnen Kopieraktivitäten beheben.

## <a name="copy-performance-optimization-features"></a>Features für die Leistungsoptimierung von Kopiervorgängen

Der Dienst bietet die folgenden Leistungsoptimierungsfeatures:

* [Datenintegrationseinheiten](#data-integration-units)
* [Skalierbarkeit der selbstgehosteten Integration Runtime (IR)](#self-hosted-integration-runtime-scalability)
* [Parallele Kopie](#parallel-copy)
* [Gestaffeltes Kopieren](#staged-copy)

### <a name="data-integration-units"></a>Datenintegrationseinheiten

Eine Datenintegrationseinheit (DIU) ist ein Measure, das die Leistung einer einzelnen Einheit in Azure Data Factory- und Synapse-Pipelines darstellt. Leistung ist eine Kombination aus CPU-, Arbeitsspeicher- und Netzwerkressourcenzuordnung. DIU gilt nur für [Azure Integration Runtime](concepts-integration-runtime.md#azure-integration-runtime). DIU gilt nicht für die [selbst gehostete Integration Runtime](concepts-integration-runtime.md#self-hosted-integration-runtime). [Hier erhalten Sie weitere Informationen](copy-activity-performance-features.md#data-integration-units).

### <a name="self-hosted-integration-runtime-scalability"></a>Skalierbarkeit der selbstgehosteten Integration Runtime (IR)

Vielleicht möchten Sie eine zunehmende gleichzeitige Workload hosten. Oder Sie möchten vielleicht eine höhere Leistung bei Ihrer derzeitigen Workload erreichen. Sie können den Umfang der Verarbeitung durch die folgenden Ansätze erweitern:

* Sie können die selbst gehostete IR _hochskalieren_, indem Sie die Anzahl der [gleichzeitigen Aufträge](create-self-hosted-integration-runtime.md#scale-up) erhöhen, die auf einem Knoten ausgeführt werden können.  
Das Hochskalieren funktioniert nur, wenn der Prozessor und der Arbeitsspeicher des Knotens nicht voll ausgelastet sind.
* Sie können die selbst gehostete IR _aufskalieren_, indem Sie weitere Knoten (Computer) hinzufügen.

Weitere Informationen finden Sie unter

* [Features für die Leistungsoptimierung bei Kopieraktivitäten: Skalierbarkeit der selbstgehosteten Integration Runtime (IR)](copy-activity-performance-features.md#self-hosted-integration-runtime-scalability)
* [Erstellen und Konfigurieren einer selbstgehosteten Integration Runtime: Aspekte der Skalierung](create-self-hosted-integration-runtime.md#scale-considerations)

### <a name="parallel-copy"></a>Parallele Kopie

Sie können die `parallelCopies`-Eigenschaft festlegen, um die Parallelität anzugeben, die für die Kopieraktivität verwendet werden soll. Betrachten Sie diese Eigenschaft als die maximale Anzahl von Threads innerhalb der Kopieraktivität. Die Threads arbeiten parallel. Die Threads lesen entweder aus Ihrer Quelle oder schreiben in Ihre Senkendatenspeicher. [Weitere Informationen](copy-activity-performance-features.md#parallel-copy).

### <a name="staged-copy"></a>gestaffeltem Kopieren

Ein Datenkopiervorgang kann die Daten _direkt_ an den Senkendatenspeicher senden. Alternativ können Sie Blobspeicher auch als _Stagingzwischenspeicher_ verwenden. [Weitere Informationen](copy-activity-performance-features.md#staged-copy)

## <a name="next-steps"></a>Nächste Schritte

Weitere Informationen finden Sie in den anderen Artikeln zur Kopieraktivität:

* [Kopieraktivität – Übersicht](copy-activity-overview.md)
* [Troubleshoot copy activity performance](copy-activity-performance-troubleshooting.md) (Problembehandlung bei der Leistung der Kopieraktivität)
* [Features für die Leistungsoptimierung bei Kopieraktivitäten](copy-activity-performance-features.md)
* [Verwenden von Azure Data Factory zum Migrieren von Daten aus einem Data Lake oder Data Warehouse zu Azure](data-migration-guidance-overview.md)
* [Migrieren von Daten aus Amazon S3 zu Azure Storage](data-migration-guidance-s3-azure-storage.md)
