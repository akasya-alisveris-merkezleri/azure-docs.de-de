---
title: Visualisieren von Daten mit Apache Spark
description: Erstellen von umfangreichen Datenvisualisierungen mit Apache Spark und Azure Synapse Analytics-Notebooks
services: synapse-analytics
author: midesa
ms.service: synapse-analytics
ms.topic: conceptual
ms.subservice: machine-learning
ms.date: 10/20/2020
ms.author: midesa
ms.openlocfilehash: beef6768f9b2fb05efb77c16c32b0acbe46d1e85
ms.sourcegitcommit: 0046757af1da267fc2f0e88617c633524883795f
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 08/13/2021
ms.locfileid: "122339326"
---
# <a name="analyze-data-with-apache-spark"></a>Analysieren von Daten mit Apache Spark

In diesem Tutorial erfahren Sie, wie Sie mithilfe von Azure Open Datasets und Apache Spark eine explorative Datenanalyse durchführen. Anschließend können Sie die Ergebnisse in Azure Synapse Analytics in einem Synapse Studio-Notebook visualisieren.

In diesem Tutorial wird das Dataset [New York City (NYC) Taxi](https://azure.microsoft.com/services/open-datasets/catalog/nyc-taxi-limousine-commission-yellow-taxi-trip-records/) analysiert. Die Daten sind über Azure Open Datasets verfügbar. Diese Teilmenge des Datasets enthält Informationen zu Taxifahrten von Yellow Cabs: Informationen zu den einzelnen Fahrten, Start- und Endzeiten, Abfahrtsorte und Ziele, die Kosten sowie weitere interessante Attribute.
  
## <a name="before-you-begin"></a>Voraussetzungen
Erstellen Sie einen Apache Spark-Pool, indem Sie das Tutorial [Erstellen eines Apache Spark-Pools](../articles/../quickstart-create-apache-spark-pool-studio.md) absolvieren. 

## <a name="download-and-prepare-the-data"></a>Herunterladen und Vorbereiten der Daten
1. Erstellen Sie ein Notebook unter Verwendung des PySpark-Kernels. Eine entsprechende Anleitung finden Sie unter [Erstellen eines Notebooks](../quickstart-apache-spark-notebook.md#create-a-notebook). 
   
   > [!Note]
   > Durch den PySpark-Kernel müssen Sie keine Kontexte explizit erstellen. Der Spark-Kontext wird automatisch für Sie erstellt, wenn Sie die erste Codezelle ausführen.

2. In diesem Tutorial verwenden wir verschiedene Bibliotheken, um das Dataset zu visualisieren. Für die Analyse müssen die folgenden Bibliotheken importiert werden: 

   ```python
   import matplotlib.pyplot as plt
   import seaborn as sns
   import pandas as pd
   ```

3. Da die Rohdaten im Parquet-Format vorliegen, können Sie den Spark-Kontext verwenden, um die Datei direkt als Datenrahmen in den Arbeitsspeicher zu lesen. Erstellen Sie einen Spark-Datenrahmen, indem Sie die Daten über die Open Datasets-API abrufen. Hier verwenden wir die *Schema beim Lesen*-Eigenschaften des Spark-Datenrahmens, um die Datentypen und das Schema abzuleiten.

   ```python
   from azureml.opendatasets import NycTlcYellow
   from datetime import datetime
   from dateutil import parser

   end_date = parser.parse('2018-06-06')
   start_date = parser.parse('2018-05-01')
   nyc_tlc = NycTlcYellow(start_date=start_date, end_date=end_date)
   df = nyc_tlc.to_spark_dataframe()
   ```

4. Nachdem die Daten gelesen wurden, müssen wir eine erste Filterung durchführen, um das Dataset zu bereinigen. Wir können nicht benötigte Spalten entfernen und Spalten hinzufügen, die wichtige Informationen extrahieren. Darüber hinaus werden auch Anomalien aus dem Dataset herausgefiltert.

   ```python
   # Filter the dataset 
   from pyspark.sql.functions import *

   filtered_df = df.select('vendorID', 'passengerCount', 'tripDistance','paymentType', 'fareAmount', 'tipAmount'\
                                   , date_format('tpepPickupDateTime', 'hh').alias('hour_of_day')\
                                   , dayofweek('tpepPickupDateTime').alias('day_of_week')\
                                   , dayofmonth(col('tpepPickupDateTime')).alias('day_of_month'))\
                               .filter((df.passengerCount > 0)\
                                   & (df.tipAmount >= 0)\
                                   & (df.fareAmount >= 1) & (df.fareAmount <= 250)\
                                   & (df.tripDistance > 0) & (df.tripDistance <= 200))

   filtered_df.createOrReplaceTempView("taxi_dataset")
   ```

## <a name="analyze-data"></a>Daten analysieren
Als Datenanalyst steht Ihnen eine Vielzahl von Tools zur Verfügung, die Ihnen beim Extrahieren von Erkenntnissen aus Daten helfen. In diesem Teil des Tutorials werden einige nützliche Tools erläutert, die in Azure Synapse Analytics-Notebooks verfügbar sind. In dieser Analyse möchten wir die Faktoren verstehen, die im ausgewählten Zeitraum zu höheren Trinkgeldern führen.

### <a name="apache-spark-sql-magic"></a>Apache Spark SQL-Magic-Befehl 
Als Erstes führen wir eine explorative Datenanalyse mithilfe von Apache Spark SQL und Magic-Befehlen mit dem Azure Synapse-Notebook aus. Sobald die Abfrage fertig ist, visualisieren wir die Ergebnisse mithilfe der integrierten ```chart options```-Funktion. 

1. Erstellen Sie eine neue Zelle im Notebook, und kopieren Sie den folgenden Code. Mit dieser Abfrage möchten wir herausfinden, wie sich die durchschnittlichen Trinkgeldbeträge im ausgewählten Zeitraum verändert haben. Mit dieser Abfrage können wir auch andere nützliche Erkenntnisse erzielen, beispielsweise den minimalen und den maximalen Trinkgeldbetrag pro Tag sowie den durchschnittlichen Fahrpreis.
   
   ```sql
   %%sql
   SELECT 
       day_of_month
       , MIN(tipAmount) AS minTipAmount
       , MAX(tipAmount) AS maxTipAmount
       , AVG(tipAmount) AS avgTipAmount
       , AVG(fareAmount) as fareAmount
   FROM taxi_dataset 
   GROUP BY day_of_month
   ORDER BY day_of_month ASC
   ```

2. Nach dem Ausführen der Abfrage können wir die Ergebnisse visualisieren, indem wir zur Diagrammansicht wechseln. In diesem Beispiel wird ein Liniendiagramm erstellt, indem wir das Feld ```day_of_month``` als Schlüssel und ```avgTipAmount``` als Wert angeben. Nachdem Sie eine Auswahl getroffen haben, wählen Sie **Anwenden** aus, um das Diagramm zu aktualisieren. 
   
## <a name="visualize-data"></a>Visualisieren von Daten
Zusätzlich zu den integrierten Diagrammoptionen im Notebook können Sie auch beliebte Open-Source-Bibliotheken verwenden, um eigene Visualisierungen zu erstellen. In den folgenden Beispielen werden Seaborn und Matplotlib verwendet. Dabei handelt es sich um häufig verwendete Python-Bibliotheken für die Datenvisualisierung. 

> [!Note]
> Standardmäßig enthält jeder Apache Spark-Pool in Azure Synapse Analytics eine Reihe gängiger Bibliotheken und Standardbibliotheken. Eine vollständige Liste der Bibliotheken finden Sie in der Dokumentation zur [Azure Synapse-Runtime](../spark/apache-spark-version-support.md). Wenn Sie Code eines Drittanbieters oder lokal erstellten Code für Ihre Anwendungen verfügbar machen möchten, können Sie auch in einem Ihrer Spark-Pools [eine Bibliothek installieren](../spark/apache-spark-azure-portal-add-libraries.md).

1. Um die Entwicklung einfacher und kostengünstiger zu gestalten, führen wir ein Downsampling des Datasets aus. Dabei verwenden wir die integrierte Samplingfunktion von Apache Spark. Darüber hinaus benötigen sowohl Seaborn als auch Matplotlib einen Pandas-Datenrahmen oder ein NumPy-Array. Um einen Pandas-Datenrahmen zu erhalten, verwenden wir den Befehl ```toPandas()``` zum Konvertieren des Datenrahmens.

   ```python
   # To make development easier, faster, and less expensive, downsample for now
   sampled_taxi_df = filtered_df.sample(True, 0.001, seed=1234)

   # The charting package needs a Pandas DataFrame or NumPy array to do the conversion
   sampled_taxi_pd_df = sampled_taxi_df.toPandas()
   ```

1. Zunächst möchten wir uns die Verteilung der Trinkgelder im Dataset ansehen. Wir verwenden Matplotlib, um ein Histogramm zu erstellen, das die Verteilung der Beträge und die Anzahl zeigt. Anhand der Verteilung können wir erkennen, dass Trinkgelder tendenziell nicht mehr als 10 USD betragen.

   ```python
   # Look at a histogram of tips by count by using Matplotlib

   ax1 = sampled_taxi_pd_df['tipAmount'].plot(kind='hist', bins=25, facecolor='lightblue')
   ax1.set_title('Tip amount distribution')
   ax1.set_xlabel('Tip Amount ($)')
   ax1.set_ylabel('Counts')
   plt.suptitle('')
   plt.show()
   ```

   ![Histogramm der Trinkgelder.](./media/apache-spark-machine-learning-mllib-notebook/histogram.png)

1. Als Nächstes möchten wir die Beziehung zwischen Trinkgeld pro Fahrt und Wochentag verstehen. Wir verwenden Seaborn, um ein Boxplotdiagramm zu erstellen, das die Trends für jeden Wochentag zusammenfasst. 

   ```python
   # View the distribution of tips by day of week using Seaborn
   ax = sns.boxplot(x="day_of_week", y="tipAmount",data=sampled_taxi_pd_df, showfliers = False)
   ax.set_title('Tip amount distribution per day')
   ax.set_xlabel('Day of Week')
   ax.set_ylabel('Tip Amount ($)')
   plt.show()

   ```
   ![Diagramm zur Darstellung der Verteilung von Trinkgeldern pro Tag.](./media/apache-spark-data-viz/data-analyst-tutorial-per-day.png)

4. Nun möchten wir eine weitere Hypothese untersuchen, derzufolge möglicherweise ein positiver Zusammenhang zwischen der Anzahl von Fahrgästen und dem gesamten Trinkgeldbetrag besteht. Um diesen Zusammenhang zu überprüfen, führen wir den folgenden Code aus, um ein Boxplotdiagramm zu generieren, das die Verteilung von Trinkgeldern für die jeweilige Anzahl von Fahrgästen zeigt. 
   
   ```python
   # How many passengers tipped by various amounts 
   ax2 = sampled_taxi_pd_df.boxplot(column=['tipAmount'], by=['passengerCount'])
   ax2.set_title('Tip amount by Passenger count')
   ax2.set_xlabel('Passenger count')
   ax2.set_ylabel('Tip Amount ($)')
   ax2.set_ylim(0,30)
   plt.suptitle('')
   plt.show()
   ```
   ![Boxplotdiagramm.](./media/apache-spark-machine-learning-mllib-notebook/box-whisker-plot.png)

1. Zum Schluss möchten wir mehr über den Zusammenhang zwischen Fahrpreis und Trinkgeldbetrag erfahren. Anhand der Ergebnisse können wir feststellen, dass es mehrere Fahrten gab, bei denen die Fahrgäste gar kein Trinkgeld gegeben haben. Wir sehen jedoch auch einen positiven Zusammenhang zwischen Gesamtfahrpreis und Trinkgeldbetrag.
   
   ```python
   # Look at the relationship between fare and tip amounts

   ax = sampled_taxi_pd_df.plot(kind='scatter', x= 'fareAmount', y = 'tipAmount', c='blue', alpha = 0.10, s=2.5*(sampled_taxi_pd_df['passengerCount']))
   ax.set_title('Tip amount by Fare amount')
   ax.set_xlabel('Fare Amount ($)')
   ax.set_ylabel('Tip Amount ($)')
   plt.axis([-2, 80, -2, 20])
   plt.suptitle('')
   plt.show()
   ```
   ![Punktdiagramm zum Trinkgeldbetrag.](./media/apache-spark-machine-learning-mllib-notebook/scatter.png)

## <a name="shut-down-the-spark-instance"></a>Herunterfahren der Spark-Instanz

Nachdem Sie die Anwendung ausgeführt haben, sollten Sie das Notebook herunterfahren, um die Ressourcen freizugeben. Schließen Sie dazu die Registerkarte, oder wählen Sie im Statusbereich am unteren Rand des Notebooks die Option **Sitzung beenden** aus.

## <a name="see-also"></a>Weitere Informationen

- [Übersicht: Apache Spark in Azure Synapse Analytics](apache-spark-overview.md)
- [Erstellen eines Machine Learning-Modells mit Apache SparkML](../spark/apache-spark-machine-learning-mllib-notebook.md)

## <a name="next-steps"></a>Nächste Schritte

- [Azure Synapse Analytics](../index.yml)
- [Offizielle Apache Spark-Dokumentation](https://spark.apache.org/docs/latest/)