---
title: Tabellendatentypen in Synapse SQL
description: Empfehlungen zum Definieren von Tabellendatentypen in Synapse SQL
services: synapse-analytics
author: filippopovic
manager: craigg
ms.service: synapse-analytics
ms.topic: conceptual
ms.subservice: sql
ms.date: 04/15/2020
ms.author: fipopovi
ms.reviewer: jrasnick
ms.custom: ''
ms.openlocfilehash: 1e6cd43e61389596be9b134ab2ad62bbf324a5cd
ms.sourcegitcommit: 58d82486531472268c5ff70b1e012fc008226753
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 08/23/2021
ms.locfileid: "122693566"
---
# <a name="table-data-types-in-synapse-sql"></a>Tabellendatentypen in Synapse SQL

In diesem Artikel finden Sie Empfehlungen zum Definieren von Tabellendatentypen in dedizierten Synapse SQL-Pools. 

## <a name="data-types"></a>Datentypen

Dedizierte Synapse SQL-Pools unterstützen die am häufigsten verwendeten Datentypen. Eine Liste der unterstützten Datentypen finden Sie in der CREATE TABLE-Anweisung unter [Datentypen](/sql/t-sql/statements/create-table-azure-sql-data-warehouse#DataTypes&preserve-view=true). Informationen zu Synapse SQL (serverlos) finden Sie im Artikel [Abfragen von Speicherdateien mit einem serverlosen SQL-Pool in Azure Synapse Analytics](./query-data-storage.md) und [Verwenden von OPENROWSET mit einem serverlosen SQL-Pool in Azure Synapse Analytics](./develop-openrowset.md).

## <a name="minimize-row-length"></a>Minimieren der Zeilenlänge

Durch das Minimieren der Größe von Datentypen wird die Zeilenlänge verkürzt. Dies führt zu einer besseren Abfrageleistung. Verwenden Sie den kleinsten Datentyp, der für Ihre Daten funktioniert.

- Vermeiden Sie es, für Zeichenspalten eine hohe Standardlänge zu definieren. Wenn der längste Wert 25 Zeichen umfasst, sollten Sie die Spalte VARCHAR(25) definieren.
- Vermeiden Sie die Verwendung von [NVARCHAR][NVARCHAR], wenn Sie nur VARCHAR benötigen.
- Verwenden Sie anstelle von NVARCHAR(MAX) oder VARCHAR(MAX) nach Möglichkeit NVARCHAR(4000) oder VARCHAR(8000).
- Vermeiden Sie die Verwendung von Gleitkommazahlen und Dezimalzahlen mit 0 (null) Dezimalstellen.  Dafür sollten TINYINT, SMALLINT, INT oder BIGINT verwendet werden.

> [!NOTE]
> Wenn Sie Ihre Synapse SQL-Tabellen mithilfe externer PolyBase-Tabellen laden, darf die definierte Länge der Tabellenzeile 1 MB nicht überschreiten. Wenn eine Zeile mit Daten variabler Länge 1 MB überschreitet, können Sie die Zeile mit BCP, jedoch nicht mit PolyBase laden.

## <a name="identify-unsupported-data-types"></a>Identifizieren nicht unterstützter Datentypen

Wenn Sie die Datenbank von einer anderen SQL-Datenbank migrieren, kommen unter Umständen einige Datentypen vor, die in Synapse SQL nicht unterstützt werden. Verwenden Sie diese Abfrage, um nicht unterstützte Datentypen in Ihrem vorhandenen SQL-Schema zu ermitteln.

```sql
SELECT  t.[name], c.[name], c.[system_type_id], c.[user_type_id], y.[is_user_defined], y.[name]
FROM sys.tables  t
JOIN sys.columns c on t.[object_id]    = c.[object_id]
JOIN sys.types   y on c.[user_type_id] = y.[user_type_id]
WHERE y.[name] IN ('geography','geometry','hierarchyid','image','text','ntext','sql_variant','xml')
 OR  y.[is_user_defined] = 1;
```

## <a name="workarounds-for-unsupported-data-types"></a><a name="unsupported-data-types"></a>Problemumgehungen für nicht unterstützte Datentypen

Die folgende Liste zeigt die Datentypen, die nicht von Synapse SQL unterstützt werden, und bietet Alternativen, die Sie anstelle der nicht unterstützten Datentypen verwenden können.

| Nicht unterstützte Datentypen | Problemumgehung |
| --- | --- |
| [geometry](/sql/t-sql/spatial-geometry/spatial-types-geometry-transact-sql?view=azure-sqldw-latest&preserve-view=true&preserve-view=true) |[varbinary](/sql/t-sql/data-types/binary-and-varbinary-transact-sql?view=azure-sqldw-latest&preserve-view=true) |
| [geography](/sql/t-sql/spatial-geography/spatial-types-geography) |[varbinary](/sql/t-sql/data-types/binary-and-varbinary-transact-sql?view=azure-sqldw-latest&preserve-view=true) |
| [hierarchyid](/sql/t-sql/data-types/hierarchyid-data-type-method-reference) |[nvarchar](/sql/t-sql/data-types/nchar-and-nvarchar-transact-sql?view=azure-sqldw-latest&preserve-view=true)(4000) |
| [image](/sql/t-sql/data-types/ntext-text-and-image-transact-sql?view=azure-sqldw-latest&preserve-view=true) |[varbinary](/sql/t-sql/data-types/binary-and-varbinary-transact-sql?view=azure-sqldw-latest&preserve-view=true) |
| [text](/sql/t-sql/data-types/ntext-text-and-image-transact-sql?view=azure-sqldw-latest&preserve-view=true) |[varchar](/sql/t-sql/data-types/char-and-varchar-transact-sql?view=azure-sqldw-latest&preserve-view=true) |
| [ntext](/sql/t-sql/data-types/ntext-text-and-image-transact-sql?view=azure-sqldw-latest&preserve-view=true) |[nvarchar](/sql/t-sql/data-types/nchar-and-nvarchar-transact-sql?view=azure-sqldw-latest&preserve-view=true) |
| [sql_variant](/sql/t-sql/data-types/sql-variant-transact-sql?view=azure-sqldw-latest&preserve-view=true) |Spalte in mehrere Spalten mit starker Typisierung unterteilen. |
| [Tabelle](/sql/t-sql/data-types/table-transact-sql?view=azure-sqldw-latest&preserve-view=true) |Führen Sie eine Konvertierung in temporäre Tabellen durch, oder erwägen Sie die Speicherung von Daten im Speicher mit [CETAS](../sql/develop-tables-cetas.md). |
| [timestamp](/sql/t-sql/data-types/date-and-time-types) |Überarbeiten Sie Code mit [datetime2](/sql/t-sql/data-types/datetime2-transact-sql?view=azure-sqldw-latest&preserve-view=true) und der [CURRENT_TIMESTAMP](/sql/t-sql/functions/current-timestamp-transact-sql?view=azure-sqldw-latest&preserve-view=true)-Funktion. Nur Konstanten werden als Standardwerte unterstützt. Daher kann „current_timestamp“ nicht als Standardeinschränkung definiert werden. Wenn Sie Zeilenversionswerte aus einer Spalte vom Typ „timestamp“ migrieren müssen, sollten Sie [BINARY](/sql/t-sql/data-types/binary-and-varbinary-transact-sql?view=azure-sqldw-latest&preserve-view=true)(8) oder [VARBINARY](/sql/t-sql/data-types/binary-and-varbinary-transact-sql?view=azure-sqldw-latest&preserve-view=true)(8) für NOT NULL- oder NULL-Zeilenversionswerte verwenden. |
| [xml](/sql/t-sql/xml/xml-transact-sql?view=azure-sqldw-latest&preserve-view=true) |[varchar](/sql/t-sql/data-types/char-and-varchar-transact-sql?view=azure-sqldw-latest&preserve-view=true) |
| [Benutzerdefinierter Typ](/sql/relational-databases/native-client/features/using-user-defined-types) |Führen Sie, wenn möglich, eine Rückkonvertierung in den nativen Datentyp durch. |
| Standardwerte | Standardwerte unterstützen nur Literale und Konstanten. |

## <a name="next-steps"></a>Nächste Schritte

Weitere Informationen zum Entwickeln von Tabellen finden Sie unter [Einführung in das Entwerfen von Tabellen in Azure SQL Data Warehouse](develop-overview.md).
