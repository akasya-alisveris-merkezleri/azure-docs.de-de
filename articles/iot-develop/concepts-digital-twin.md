---
title: Grundlegendes zu digitalen IoT Plug & Play-Zwillingen
description: Hier erfahren Sie, wie IoT Plug & Play digitale Zwillinge verwendet.
author: prashmo
ms.author: prashmo
ms.date: 12/14/2020
ms.topic: conceptual
ms.service: iot-develop
services: iot-develop
ms.openlocfilehash: 5623192cd713f06be24ce6c3f78b91756868db86
ms.sourcegitcommit: 8669087bcbda39e3377296c54014ce7b58909746
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 07/18/2021
ms.locfileid: "114406435"
---
# <a name="understand-iot-plug-and-play-digital-twins"></a>Grundlegendes zu digitalen IoT Plug & Play-Zwillingen

Ein IoT Plug & Play-Gerät implementiert ein Modell, das durch das [Digital Twins Definition Language (DTDL)](https://github.com/Azure/opendigitaltwins-dtdl)-Schema beschrieben wird. Ein Modell beschreibt den Satz von Komponenten, Eigenschaften, Befehlen und Telemetrienachrichten, die ein bestimmtes Gerät aufweisen kann.

IoT Plug & Play verwendet DTDL-Version 2. Weitere Informationen zu dieser Version finden Sie in der Spezifikation [Digital Twins Definition Language (DTDL) – Version 2](https://github.com/Azure/opendigitaltwins-dtdl/blob/master/DTDL/v2/dtdlv2.md) auf GitHub.

> [!NOTE]
> DTDL ist nicht für IoT Plug & Play exklusiv. Andere IoT-Dienste, z. B. [Azure Digital Twins](../digital-twins/overview.md), stellen damit ganze Umgebungen wie Gebäude und Energieversorgungsnetze dar.

Die Azure IoT-Dienst-SDKs enthalten APIs, die einem Dienst das Interagieren mit dem digitalen Zwilling eines Geräts ermöglichen. So kann ein Dienst beispielsweise Geräteeigenschaften aus dem digitalen Zwilling lesen oder mithilfe des digitalen Zwillings einen Befehl auf einem Gerät aufrufen. Weitere Informationen finden Sie unter [Beispiele für digitale IoT Hub-Zwillinge](concepts-developer-guide-service.md#iot-hub-digital-twin-examples).

Das Beispiel für ein IoT Plug & Play-Gerät in diesem Artikel implementiert ein [Modell „Temperaturregler“](https://github.com/Azure/opendigitaltwins-dtdl/blob/master/DTDL/v2/samples/TemperatureController.json) mit [Thermostat](https://github.com/Azure/opendigitaltwins-dtdl/blob/master/DTDL/v2/samples/Thermostat.json)-Komponenten.

## <a name="device-twins-and-digital-twins"></a>Gerätezwillinge und digitale Zwillinge

Genauso wie einen digitalen Zwilling verwaltet Azure IoT Hub auch einen *Gerätezwilling* für jedes verbundene Gerät. Ein Gerätezwilling ähnelt einem digitalen Zwilling darin, dass er die Eigenschaften eines Geräts darstellt. Die Azure IoT-Dienst-SDKs enthalten APIs zum Interagieren mit Gerätezwillingen.

Ein IoT-Hub initialisiert einen digitalen Zwilling und einen Gerätezwilling, wenn ein IoT Plug & Play-Gerät zum ersten Mal eine Verbindung herstellt.

Gerätezwillinge sind JSON-Dokumente, in denen Gerätestatusinformationen gespeichert werden, z. B. Metadaten, Konfigurationen und Bedingungen. Weitere Informationen finden Sie unter [Beispiele für IoT Hub-Dienstclients](concepts-developer-guide-service.md#iot-hub-service-client-examples). Sowohl Geräte- als auch Lösungsentwickler können weiterhin denselben Satz von APIs und SDKs für Gerätezwillinge verwenden, um Geräte und Lösungen mithilfe von IoT Plug & Play-Konventionen zu implementieren.

Die APIs für den digitalen Zwilling arbeiten mit allgemeinen DTDL-Konstrukten wie Komponenten, Eigenschaften und Befehlen. Die APIs für digitale Zwillinge erleichtern Lösungsentwicklern das Erstellen von IoT Plug & Play-Lösungen.

Bei einem Gerätezwilling wird der Zustand einer schreibbaren Eigenschaft auf die Abschnitte für *gewünschte Eigenschaften* und *gemeldete Eigenschaften* aufgeteilt. Alle schreibgeschützten Eigenschaften stehen im Abschnitt für gemeldete Eigenschaften zur Verfügung.

Bei einem digitalen Zwilling gibt es eine einheitliche Ansicht des aktuellen und des gewünschten Zustands der Eigenschaft. Der Synchronisierungsstatus für eine bestimmte Eigenschaft wird im entsprechenden Abschnitt `$metadata` der Standardkomponentenebene gespeichert.

### <a name="device-twin-json-example"></a>JSON-Beispiel für einen Gerätezwilling

Der folgende Codeausschnitt zeigt einen IoT Plug & Play-Gerätezwilling, der als JSON-Objekt formatiert ist:

```json
{
  "deviceId": "sample-device",
  "modelId": "dtmi:com:example:TemperatureController;1",
  "version": 15,
  "properties": {
    "desired": {
      "thermostat1": {
        "__t": "c",
        "targetTemperature": 21.8
      },
      "$metadata": {...},
      "$version": 4
    },
    "reported": {
      "serialNumber": "alwinexlepaho8329",
      "thermostat1": {
        "maxTempSinceLastReboot": 25.3,
        "__t": "c",
        "targetTemperature": {
          "value": 21.8,
          "ac": 200,
          "ad": "Successfully executed patch",
        }
      },
      "$metadata": {...},
      "$version": 11
    }
  }
}
```

### <a name="digital-twin-example"></a>Beispiel für einen digitalen Zwilling

Der folgende Codeausschnitt zeigt den digitalen Zwilling, der als JSON-Objekt formatiert ist:

```json
{
  "$dtId": "sample-device",
  "serialNumber": "alwinexlepaho8329",
  "thermostat1": {
    "maxTempSinceLastReboot": 25.3,
    "targetTemperature": 21.8,
    "$metadata": {
      "targetTemperature": {
        "desiredValue": 21.8,
        "desiredVersion": 4,
        "ackVersion": 4,
        "ackCode": 200,
        "ackDescription": "Successfully executed patch",
        "lastUpdateTime": "2020-07-17T06:11:04.9309159Z"
      },
      "maxTempSinceLastReboot": {
         "lastUpdateTime": "2020-07-17T06:10:31.9609233Z"
      }
    }
  },
  "$metadata": {
    "$model": "dtmi:com:example:TemperatureController;1",
    "serialNumber": {
      "lastUpdateTime": "2020-07-17T06:10:31.9609233Z"
    }
  }
}
```

In der folgenden Tabelle werden die Felder im JSON-Objekt für den digitalen Zwilling beschrieben:

| Feldname | BESCHREIBUNG |
| --- | --- |
| `$dtId` | Eine vom Benutzer angegebene Zeichenfolge zur Darstellung der ID des digitalen Zwillings eines Geräts |
| `{propertyName}` | Der Wert einer Eigenschaft im JSON-Format |
| `$metadata.$model` | [Optional] Die ID der Modellschnittstelle, die diesen digitalen Zwilling kennzeichnet |
| `$metadata.{propertyName}.desiredValue` | [Nur für schreibbare Eigenschaften] Der gewünschte Wert der angegebenen Eigenschaft |
| `$metadata.{propertyName}.desiredVersion` | [Nur für schreibbare Eigenschaften] Die Version des gewünschten Werts, der von IoT Hub verwaltet wird|
| `$metadata.{propertyName}.ackVersion` | [Erforderlich, nur für schreibbare Eigenschaften] Die Version, die von dem Gerät bestätigt wird, das den digitalen Zwilling implementiert; muss größer oder gleich der gewünschten Version sein |
| `$metadata.{propertyName}.ackCode` | [Erforderlich, nur für schreibbare Eigenschaften] Der von der Geräte-App, die den digitalen Zwilling implementiert, zurückgegebene `ack`-Code |
| `$metadata.{propertyName}.ackDescription` | [Optional, nur für schreibbare Eigenschaften] Die von der Geräte-App, die den digitalen Zwilling implementiert, zurückgegebene `ack`-Beschreibung |
| `$metadata.{propertyName}.lastUpdateTime` | IoT Hub verwaltet den Zeitstempel der letzten Aktualisierung der Eigenschaft durch das Gerät. Zeitstempel verwenden UTC und sind im ISO8601-Format (YYYY-MM-DDTHH:MM:SS.mmmZ) codiert |
| `{componentName}` | Ein JSON-Objekt mit den Eigenschaftswerten und Metadaten der Komponente. |
| `{componentName}.{propertyName}` | Der Wert der Komponenteneigenschaft im JSON-Format |
| `{componentName}.$metadata` | Die Metadateninformationen für die Komponente. |

### <a name="properties"></a>Eigenschaften

Eigenschaften sind Datenfelder, die den Zustand einer Entität repräsentieren (wie die Eigenschaften in vielen objektorientierten Programmiersprachen).

#### <a name="read-only-property"></a>Schreibgeschützte Eigenschaft

DTDL-Schema:

```json
{
    "@type": "Property",
    "name": "serialNumber",
    "displayName": "Serial Number",
    "description": "Serial number of the device.",
    "schema": "string"
}
```

In diesem Beispiel ist `alwinexlepaho8329` der aktuelle Wert der schreibgeschützten Eigenschaft `serialNumber`, die vom Gerät gemeldet wird.
Die folgenden Codeausschnitte zeigen die JSON-Darstellung der Eigenschaft `serialNumber` nebeneinander:

:::row:::
   :::column span="":::
      **Gerätezwilling**

```json
"properties": {
  "reported": {
    "serialNumber": "alwinexlepaho8329"
  }
}
```

   :::column-end:::
   :::column span="":::
      **Digitaler Zwilling**

```json
"serialNumber": "alwinexlepaho8329"
```

   :::column-end:::
:::row-end:::

#### <a name="writable-property"></a>Schreibbare Eigenschaft

Die folgenden Beispiele zeigen eine schreibbare Eigenschaft in der Standardkomponente.

DTDL:

```json
{
  "@type": "Property",
  "name": "fanSpeed",
  "displayName": "Fan Speed",
  "writable": true,
  "schema": "double"
}
```

:::row:::
   :::column span="":::
      **Gerätezwilling**

```json
{
  "properties": {
    "desired": {
      "fanSpeed": 2.0,
    },
    "reported": {
      "fanSpeed": {
        "value": 3.0,
        "ac": 200,
        "av": 1,
        "ad": "Successfully executed patch version 1"
      }
    }
  },
}
```

   :::column-end:::
   :::column span="":::
      **Digitaler Zwilling**

```json
{
  "fanSpeed": 3.0,
  "$metadata": {
    "fanSpeed": {
      "desiredValue": 2.0,
      "desiredVersion": 2,
      "ackVersion": 1,
      "ackCode": 200,
      "ackDescription": "Successfully executed patch version 1",
      "lastUpdateTime": "2020-07-17T06:10:31.9609233Z"
    }
  }
}
```

   :::column-end:::
:::row-end:::

In diesem Beispiel ist `3.0` der aktuelle Wert der Eigenschaft `fanSpeed`, die vom Gerät gemeldet wird. `2.0` ist der gewünschte Wert, der durch die Lösung festgelegt ist. Der gewünschte Wert und der Synchronisierungsstatus einer Eigenschaft auf Stammebene werden in `$metadata` auf Stammebene für einen digitalen Zwilling festgelegt. Wenn das Gerät online geschaltet wird, kann es diese Aktualisierung anwenden und den aktualisierten Wert zurückmelden.

### <a name="components"></a>Komponenten

Mit Komponenten kann eine Modellschnittstelle als eine Assembly anderer Schnittstellen erstellt werden.
So kann beispielsweise die Schnittstelle [Thermostat](https://github.com/Azure/opendigitaltwins-dtdl/blob/master/DTDL/v2/samples/Thermostat.json) in Form der Komponenten `thermostat1` und `thermostat2` in das [Modell „Temperaturregler“](https://github.com/Azure/opendigitaltwins-dtdl/blob/master/DTDL/v2/samples/TemperatureController.json) integriert werden.

Bei einem Gerätezwilling wird eine Komponente durch den Marker `{ "__t": "c"}` identifiziert. Bei einem digitalen Zwilling kennzeichnet das Vorhandensein von `$metadata` eine Komponente.

In diesem Beispiel ist `thermostat1` eine Komponente mit zwei Eigenschaften:

- `maxTempSinceLastReboot` ist eine schreibgeschützte Eigenschaft.
- `targetTemperature` ist eine schreibbare Eigenschaft, die erfolgreich vom Gerät synchronisiert wurde. Der gewünschte Wert und der Synchronisierungsstatus dieser Eigenschaften sind in `$metadata` der Komponente enthalten.

Die folgenden Codeausschnitte zeigen die JSON-Darstellung der Komponente `thermostat1` nebeneinander:

:::row:::
   :::column span="":::
      **Gerätezwilling**

```json
"properties": {
  "desired": {
    "thermostat1": {
      "__t": "c",
      "targetTemperature": 21.8
    },
    "$metadata": {
    },
    "$version": 4
  },
  "reported": {
    "thermostat1": {
      "maxTempSinceLastReboot": 25.3,
      "__t": "c",
      "targetTemperature": {
        "value": 21.8,
        "ac": 200,
        "ad": "Successfully executed patch",
        "av": 4
      }
    },
    "$metadata": {
    },
    "$version": 11
  }
}
```

   :::column-end:::
   :::column span="":::
      **Digitaler Zwilling**

```json
"thermostat1": {
  "maxTempSinceLastReboot": 25.3,
  "targetTemperature": 21.8,
  "$metadata": {
    "targetTemperature": {
      "desiredValue": 21.8,
      "desiredVersion": 4,
      "ackVersion": 4,
      "ackCode": 200,
      "ackDescription": "Successfully executed patch",
      "lastUpdateTime": "2020-07-17T06:11:04.9309159Z"
    },
    "maxTempSinceLastReboot": {
       "lastUpdateTime": "2020-07-17T06:10:31.9609233Z"
    }
  }
}
```

   :::column-end:::
:::row-end:::

## <a name="digital-twin-apis"></a>APIs für digitale Zwillinge

Die APIs für digitale Zwillinge enthalten die Vorgänge **Get Digital Twin** (Abrufen des digitalen Zwillings), **Update Digital Twin** (Aktualisieren des digitalen Zwillings), **Invoke Component Command** (Aufrufen des Komponentenbefehls) und **Invoke Command** (Aufrufen des Befehls) zum Verwalten eines digitalen Zwillings. Sie können entweder die [REST-APIs](/rest/api/iothub/service/digitaltwin) direkt oder über ein [Dienst-SDK](../iot-develop/libraries-sdks.md) verwenden.

## <a name="digital-twin-change-events"></a>Änderungsereignisse bei digitalen Zwillingen

Wenn Änderungsereignisse bei digitalen Zwillingen aktiviert sind, wird ein Ereignis ausgelöst, sobald sich der aktuelle oder gewünschte Wert der Komponente oder Eigenschaft ändert. Änderungsereignisse bei digitalen Zwillingen werden im [JSON Patch](http://jsonpatch.com/)-Format generiert. Entsprechende Ereignisse werden im Gerätezwillingsformat generiert, wenn Änderungsereignisse bei Zwillingen aktiviert sind.

Informationen zum Aktivieren des Routings für Ereignisse bei Geräte- und digitalen Zwillingen finden Sie unter [Verwenden des IoT Hub-Nachrichtenroutings zum Senden von D2C-Nachrichten an verschiedene Endpunkte](../iot-hub/iot-hub-devguide-messages-d2c.md#non-telemetry-events). Informationen zum Nachrichtenformat finden Sie unter [Erstellen und Lesen von IoT Hub-Nachrichten](../iot-hub/iot-hub-devguide-messages-construct.md).

Beispielsweise wird das folgende Änderungsereignis bei einem digitalen Zwilling ausgelöst, wenn `targetTemperature` durch die Lösung festgelegt ist:

```json
iothub-connection-device-id:sample-device
iothub-enqueuedtime:7/17/2020 6:11:04 AM
iothub-message-source:digitalTwinChangeEvents
correlation-id:275d463fa034
content-type:application/json-patch+json
content-encoding:utf-8
[
  {
    "op": "add",
    "path": "/thermostat1/$metadata/targetTemperature",
    "value": {
      "desiredValue": 21.8,
      "desiredVersion": 4
    }
  }
]
```

Das folgende Änderungsereignis bei einem digitalen Zwilling wird ausgelöst, wenn das Gerät meldet, dass die oben angegebene gewünschte Änderung angewendet wurde:

```json
iothub-connection-device-id:sample-device
iothub-enqueuedtime:7/17/2020 6:11:05 AM
iothub-message-source:digitalTwinChangeEvents
correlation-id:275d464a2c80
content-type:application/json-patch+json
content-encoding:utf-8
[
  {
    "op": "add",
    "path": "/thermostat1/$metadata/targetTemperature/ackCode",
    "value": 200
  },
  {
    "op": "add",
    "path": "/thermostat1/$metadata/targetTemperature/ackDescription",
    "value": "Successfully executed patch"
  },
  {
    "op": "add",
    "path": "/thermostat1/$metadata/targetTemperature/ackVersion",
    "value": 4
  },
  {
    "op": "add",
    "path": "/thermostat1/$metadata/targetTemperature/lastUpdateTime",
    "value": "2020-07-17T06:11:04.9309159Z"
  },
  {
    "op": "add",
    "path": "/thermostat1/targetTemperature",
    "value": 21.8
  }
]
```

> [!NOTE]
> Änderungsbenachrichtigungen für Zwillinge werden doppelt ausgegeben, wenn sie sowohl in der Gerätekonfiguration als auch in der Änderungsbenachrichtigung für digitale Zwillinge aktiviert sind.

## <a name="next-steps"></a>Nächste Schritte

Nachdem Sie sich mit digitalen Zwillingen vertraut gemacht haben, finden Sie weitere Informationen in folgenden Artikeln:

- [Verwenden von APIs für digitale IoT Plug & Play-Zwillinge](howto-manage-digital-twin.md)
- [Interagieren mit einem Gerät über Ihre Lösung](tutorial-service.md)
- [IoT-REST-API für digitale Zwillinge](/rest/api/iothub/service/digitaltwin)
- [Azure IoT-Explorer](../iot-fundamentals/howto-use-iot-explorer.md)