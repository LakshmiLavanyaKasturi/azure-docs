---
# Mandatory fields.
title: Ingest telemetry from IoT Hub
titleSuffix: Azure Digital Twins
description: Learn how to ingest device telemetry messages from Azure IoT Hub to digital twins in an instance of Azure Digital Twins.
author: baanders
ms.author: baanders # Microsoft employees only
ms.date: 02/22/2022
ms.topic: how-to
ms.service: digital-twins

# Optional fields. Don't forget to remove # if you need a field.
# ms.custom: can-be-multiple-comma-separated
# ms.reviewer: MSFT-alias-of-reviewer
# manager: MSFT-alias-of-manager-or-PM-counterpart
---

# Ingest IoT Hub telemetry into Azure Digital Twins

This guide walks through the process of writing a function that can ingest telemetry from IoT Hub and send it to an instance of Azure Digital Twins.

Azure Digital Twins is driven with data from IoT devices and other sources. A common source for device data to use in Azure Digital Twins is [IoT Hub](../iot-hub/about-iot-hub.md).

The process for ingesting data into Azure Digital Twins is to set up an external compute resource, such as a function that's made by using [Azure Functions](../azure-functions/functions-overview.md). The function receives the data and uses the [DigitalTwins APIs](/rest/api/digital-twins/dataplane/twins) to set properties or fire telemetry events on [digital twins](concepts-twins-graph.md) accordingly. 

This how-to document walks through the process for writing a function that can ingest telemetry from IoT Hub.

## Prerequisites

Before continuing with this example, you'll need to set up the following resources as prerequisites:
* An IoT hub. For instructions, see the [Create an IoT Hub section of this IoT Hub quickstart](../iot-hub/quickstart-send-telemetry-cli.md).
* An Azure Digital Twins instance that will receive your device telemetry. For instructions, see [Set up an Azure Digital Twins instance and authentication](./how-to-set-up-instance-portal.md).

This article also uses Visual Studio. You can download the latest version from [Visual Studio Downloads](https://visualstudio.microsoft.com/downloads/).

## Example telemetry scenario

This how-to outlines how to send messages from IoT Hub to Azure Digital Twins, using a function in Azure. There are many possible configurations and matching strategies you can use for sending messages, but the example for this article contains the following parts:
* A thermostat device in IoT Hub, with a known device ID
* A digital twin to represent the device, with a matching ID

> [!NOTE]
> This example uses a straightforward ID match between the device ID and a corresponding digital twin's ID, but it is possible to provide more sophisticated mappings from the device to its twin (such as with a mapping table).

Whenever a temperature telemetry event is sent by the thermostat device, a function processes the telemetry and the `Temperature` property of the digital twin should update. This scenario is outlined in a diagram below:

:::image type="content" source="media/how-to-ingest-iot-hub-data/events.png" alt-text="Diagram of IoT Hub device sending Temperature telemetry to a function in Azure, which updates a temperature property on a twin in Azure Digital Twins." border="false":::

## Add a model and twin

In this section, you'll set up a [digital twin](concepts-twins-graph.md) in Azure Digital Twins that will represent the thermostat device and will be updated with information from IoT Hub.

To create a thermostat-type twin, you'll first need to upload the thermostat [model](concepts-models.md) to your instance, which describes the properties of a thermostat and will be used later to create the twin.

[!INCLUDE [digital-twins-thermostat-model-upload.md](../../includes/digital-twins-thermostat-model-upload.md)]

You'll then need to create one twin using this model. Use the following command to create a thermostat twin named thermostat67, and set 0.0 as an initial temperature value.

```azurecli-interactive
az dt twin create  --dt-name <instance-name> --dtmi "dtmi:contosocom:DigitalTwins:Thermostat;1" --twin-id thermostat67 --properties '{"Temperature": 0.0}'
```

>[!NOTE]
>If you're using anything other than Cloud Shell in the Bash environment, you may need to escape certain characters in the inline JSON so that it's parsed correctly. 
>
>For more information, see [Use special characters in different shells](concepts-cli.md#use-special-characters-in-different-shells).

When the twin is created successfully, the CLI output from the command should look something like this:
```json
{
  "$dtId": "thermostat67",
  "$etag": "W/\"0000000-9735-4f41-98d5-90d68e673e15\"",
  "$metadata": {
    "$model": "dtmi:contosocom:DigitalTwins:Thermostat;1",
    "Temperature": {
      "lastUpdateTime": "2021-09-09T20:32:46.6692326Z"
    }
  },
  "Temperature": 0.0
}
```

## Create a function

In this section, you'll create an Azure function to access Azure Digital Twins and update twins based on IoT telemetry events that it receives. Follow the steps below to create and publish the function.

1. First, create a new function app project in Visual Studio. For instructions on how to do so, see [Develop Azure Functions using Visual Studio](../azure-functions/functions-develop-vs.md#create-an-azure-functions-project).

2. Add the following packages to your project:
    * [Azure.DigitalTwins.Core](https://www.nuget.org/packages/Azure.DigitalTwins.Core/)
    * [Azure.Identity](https://www.nuget.org/packages/Azure.Identity/)
    * [Microsoft.Azure.WebJobs.Extensions.EventGrid](https://www.nuget.org/packages/Microsoft.Azure.WebJobs.Extensions.EventGrid/)

3. Rename the *Function1.cs* sample function that Visual Studio has generated to *IoTHubtoTwins.cs*. Replace the code in the file with the following code:

    :::code language="csharp" source="~/digital-twins-docs-samples/sdks/csharp/IoTHubToTwins.cs":::

    Save your function code.

4. Publish the project with the *IoTHubtoTwins.cs* function to a function app in Azure. For instructions on how to do so, see [Develop Azure Functions using Visual Studio](../azure-functions/functions-develop-vs.md#publish-to-azure).

[!INCLUDE [digital-twins-verify-function-publish.md](../../includes/digital-twins-verify-function-publish.md)]

To access Azure Digital Twins, your function app needs a system-managed identity with permissions to access your Azure Digital Twins instance. You'll set that up next.

### Configure the function app

Next, assign an access role for the function and configure the application settings so that it can access your Azure Digital Twins instance.

[!INCLUDE [digital-twins-configure-function-app.md](../../includes/digital-twins-configure-function-app.md)]

## Connect your function to IoT Hub

In this section, you'll set up your function as an event destination for the IoT hub device data. Setting up your function in this way will ensure that the data from the thermostat device in IoT Hub will be sent to the Azure function for processing.

In the [Azure portal](https://portal.azure.com/), navigate to your IoT Hub instance that you created in the [Prerequisites](#prerequisites) section. Under **Events**, create a subscription for your function.

:::image type="content" source="media/how-to-ingest-iot-hub-data/add-event-subscription.png" alt-text="Screenshot of the Azure portal that shows Adding an event subscription.":::

In the **Create Event Subscription** page, fill the fields as follows:
  1. For **Name**, choose whatever name you want for the event subscription.
  2. For **Event Schema**, choose **Event Grid Schema**.
  3. For **System Topic Name**, choose whatever name you want.
  1. For **Filter to Event Types**, choose the **Device Telemetry** checkbox and uncheck other event types.
  1. For **Endpoint Type**, Select **Azure Function**.
  1. For **Endpoint**, use the **Select an endpoint** link to choose what Azure Function to use for the endpoint.
    
:::image type="content" source="media/how-to-ingest-iot-hub-data/create-event-subscription.png" alt-text="Screenshot of the Azure portal to create the event subscription details.":::

In the **Select Azure Function** page that opens up, verify or fill in the below details.
 1. **Subscription**: Your Azure subscription.
 2. **Resource group**: Your resource group.
 3. **Function app**: Your function app name.
 4. **Slot**: **Production**.
 5. **Function**: Select the function from earlier, *IoTHubtoTwins*, from the dropdown.

Save your details with the **Confirm Selection** button.            
      
:::image type="content" source="media/how-to-ingest-iot-hub-data/select-azure-function.png" alt-text="Screenshot of the Azure portal to select the function.":::

Select the **Create** button to create the event subscription.

## Send simulated IoT data

To test your new ingress function, use the device simulator from [Connect an end-to-end solution](./tutorial-end-to-end.md). That tutorial is driven by this [Azure Digital Twins end-to-end sample project written in C#](/samples/azure-samples/digital-twins-samples/digital-twins-samples). You'll be using the *DeviceSimulator* project in that repository.

In the end-to-end tutorial, complete the following steps:
1. [Register the simulated device with IoT Hub](./tutorial-end-to-end.md#register-the-simulated-device-with-iot-hub)
2. [Configure and run the simulation](./tutorial-end-to-end.md#configure-and-run-the-simulation)

## Validate your results

While running the device simulator above, the temperature value of your digital twin will be changing. In the Azure CLI, run the following command to see the temperature value.

```azurecli-interactive
az dt twin query --query-command "select * from digitaltwins" --dt-name <Digital-Twins-instance-name>
```

Your output should contain a temperature value like this:

```json
{
  "result": [
    {
      "$dtId": "thermostat67",
      "$etag": "W/\"dbf2fea8-d3f7-42d0-8037-83730dc2afc5\"",
      "$metadata": {
        "$model": "dtmi:contosocom:DigitalTwins:Thermostat;1",
        "Temperature": {
          "lastUpdateTime": "2021-06-03T17:05:52.0062638Z"
        }
      },
      "Temperature": 70.20518558807913
    }
  ]
}
```

To see the value change, repeatedly run the query command above.

## Next steps

Read about data ingress and egress with Azure Digital Twins:
* [Data ingress and egress](concepts-data-ingress-egress.md)
