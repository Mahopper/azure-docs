---
title: Configure API protocols - Azure Event Grid IoT Edge | Microsoft Docs 
description: Learn about the possible protocol configurations of an Event Grid module.  
author: VidyaKukke
manager: rajarv
ms.author: vkukke
ms.reviewer: spelluru
ms.date: 02/15/2022
ms.subservice: iot-edge
ms.topic: article
---

# Configure Event Grid API protocols

This guide gives examples of the possible protocol configurations of an Event Grid module. The Event Grid module exposes API for its management and runtime operations. The following table captures the protocols and ports.

| Protocol | Port | Description |
| ---------------- | ------------ | ------------ |
| HTTP | 5888 | Turned off by default. Useful only during testing. Not suitable for production workloads.
| HTTPS | 4438 | Default

See [Security and authentication](security-authentication.md) guide for all the possible configurations.

> [!IMPORTANT]
> On March 31, 2023, Event Grid on Azure IoT Edge support will be retired, so make sure to transition to IoT Edge native capabilities prior to that date. For more information, see [Transition from Event Grid on Azure IoT Edge to Azure IoT Edge](transition.md). 


## Expose HTTPS to IoT Modules on the same edge network

```json
 {
  "Env": [
    "inbound__serverAuth__tlsPolicy=strict",
    "inbound__serverAuth__serverCert__source=IoTEdge"
  ]
}
 ```

## Enable HTTPS to other IoT modules and non-IoT workloads

```json
 {
  "Env": [
    "inbound__serverAuth__tlsPolicy=strict",
    "inbound__serverAuth__serverCert__source=IoTEdge"
  ],
  "HostConfig": {
    "PortBindings": {
      "4438/tcp": [
        {
          "HostPort": "4438"
        }
      ]
    }
  }
}
 ```

>[!NOTE]
> The **PortBindings** section allows you to map internal ports to ports of the container host. This feature makes it possible to reach the Event Grid module from outside the IoT Edge container network, if the IoT edge device is reachable publicly.

## Expose HTTP and HTTPS to IoT modules on the same edge network

```json
 {
  "Env": [
    "inbound__serverAuth__tlsPolicy=enabled",
    "inbound__serverAuth__serverCert__source=IoTEdge"
  ]
}
 ```

## Enable HTTP and HTTPS to other IoT modules and non-IoT workloads

```json
 {
  "Env": [
    "inbound__serverAuth__tlsPolicy=enabled",
    "inbound__serverAuth__serverCert__source=IoTEdge"
  ],
  "HostConfig": {
    "PortBindings": {
      "4438/tcp": [
        {
          "HostPort": "4438"
        }
      ],
      "5888/tcp": [
        {
          "HostPort": "5888"
        }
      ]
    }
  }
}
 ```

>[!NOTE]
> By default, every IoT Module is part of the IoT Edge runtime created by the bridge network. It enables different IoT modules on the same network to communicate with each other. **PortBindings** allows you to map a container internal port onto the host machine thereby allowing anyone to be able to access Event Grid module's port from outside.

>[!IMPORTANT]
> While the ports can be made accessible outside the IoT Edge network, client authentication enforces who is actually allowed to make calls into the module.
