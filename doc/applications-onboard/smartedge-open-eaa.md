```text
SPDX-License-Identifier: Apache-2.0
Copyright (c) 2020 Intel Corporation
```
<!-- omit in toc -->
# Edge Application Agent (EAA)
- [Edge Application APIs](#edge-application-apis)
- [Edge Application Authentication](#edge-application-authentication)

#### Edge Application API support

Before looking at the APIs that are exposed to the Edge Applications, let's have a look at two types of applications that can be deployed on the Edge Node.
- **Producer Application**: Smart Edge Open Producer application is an edge compute application that provides services to other applications running on the edge compute platform. E.g. Location Services, Mapping Services, Transcoding Services, etc.
- **Consumer Application**: Smart Edge Open Consumer application is an edge compute application that serves end users traffic directly. E.g. CDN App, Augmented Reality App, VR Application, Infotainment Application, etc. Pre-existing cloud applications that do not intend to call the EAA APIs but would like to serve the users (without any changes to the implementation) on the edge also fall into this category.

#### EAA component design
![Edge Application Agent component design](eaa-images/eaa-comp.png)


API endpoint for edge applications is implemented in the EAA (Edge Application Agent) microservice and is implemented in Go lang. APIs are classified into:

| Edge Application API                     | Description                                                                                                                                                                                                                                                 | Example                                                                                                         |
|------------------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|-----------------------------------------------------------------------------------------------------------------|
| **Edge Service Activation/Deactivation** | This API endpoint enables a Producer App on the Edge Node to register and activate on the Edge Node\. After this API execution the Producer App will be discoverable to Consumer Apps on the Edge Node\.                                                    | Location Service Producer app will call this API first after being deployed from the controller\.               |
| **Edge Service Discovery**               | This API Endpoint enables a Consumer application to discover all the active Producer Applications on the Edge Node\.                                                                                                                                          | A CDN App will be able to discover Location Service Application on the Edge Node\.                              |
| **Edge Service Subscription/Unsubscription** | This API Endpoint enables a Consumer application to subscribe to Producer application service and notification updates\.                                                                                                                                      | A CDN application can subscribe to the Location Service application and Notification update from the service\.  |
| **Edge Service Notification update**     | This is a Web socket connection that needs to be created by a Consumer Application which intends to subscribe to services from Producer Applications\. This WebSocket will be used for push\-notification when there is update from the Producer Application\.  | Location update is sent as Push Notification update to the CDN Application\.                                        |
| **Edge Service data update**             | This API endpoint enables a Producer Application to publish the data to the Edge Node when it has an update to its service\.                                                                                                                                   |  Location Service Producer App publishes Location update of a user to the Edge Node\.                           |
| **Edge Service list subscription**       | This API endpoint allows a Consumer Application to get the list of Producer Application services it has availed of\.                                                                                                                                             | CDN Application can call this API to check if it has subscribed to Location and Transcoding services\.          |

### Edge Application APIs
Edge Application APIs are implemented by the EAA. Edge Application APIs are important APIs for Edge application developers. EAA APIs are implemented as HTTPS REST. There are two types of use cases here.
1. **Porting of existing Public/Private Cloud application to the edge compute based on Smart Edge Open**: This is the scenario where a customer wants to run existing apps in public cloud on Smart Edge Open edge without calling any APIs or changing code. In this case, the only requirement is for an Application image (VM/Container) should be available to be deployed using Smart Edge Open Kubernetes Control plane.
3. **Native Edge compute Application calling EAA APIs**: This is the scenario where a customer wants to develop Edge compute applications that take advantage of the Edge compute services resulting in more tactile application that responds to the changing user, network or resource scenarios.

Smart Edge Open supports deployment of both types of applications mentioned above. The Edge Application Agent is a service that runs on the Edge Node and operates as a discovery service and basic message bus between applications via pubsub. The connectivity and discoverability of applications by one another is governed by an entitlement system and is controlled by policies set with the Smart Edge Open Controller. The entitlement system is still in its infancy, however, and currently allows all applications on the executing Edge Node to discover one another as well as publish and subscribe to all notifications. The Figure below provides the sequence diagram of the supported APIs for the application

More details about the APIs can be found here [Edge Application APIs](https://www.openness.org/api-documentation/?api=eaa)

![Edge Application services APIs](eaa-images/eaa_services.svg)

![Edge Application subscription APIs](eaa-images/eaa_subscriptions.svg)

![Edge Application notification APIs](eaa-images/eaa_notifications.svg)

_Figure - Edge Application API Sequence Diagram: Service, Subscription and Notification_

### Edge Application Authentication

Connection to the EAA can be established after performing mutual TLS authentication. To achieve that the application needs to generate its certificate using Certificate Signer and should trust the CA certificate `root.pem` that is stored in `ca-certrequester` Kubernetes Secret.

The details can be found on the [Certificate Signer page](smartedge-open-certsigner.md).
