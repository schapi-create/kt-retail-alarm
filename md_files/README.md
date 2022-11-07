
[<img align="right" src="https://raw.githubusercontent.com/SE-Analytics-Team/public-images/master/common%20images/btn_subscribe.png">](https://shop.exchange.se.com/api/internal/storefront/v1/cta?productId=87923&type=FREE_TRIAL) 

<img src="https://github.com/schapi-create/public-images/blob/master/bem_api/content.png">


# Overview

The Alarms API is a service that will generate alerts for configured abnormal conditions. The platform will generate alarms itself, and ingest alarms from other subsystems. Automatically generating an alert allows operators to understand the current condition of the control system, knowing that issues will be brought to their attention automatically.

A single alarm is a state machine, where the transitions are recorded in a log. The state machine is discussed in detail in the document.

The Alarms API consists of two endpoints, alarms and events.

- **Events** : The events endpoint is a transaction log of all the transitions in the system
- **Alarms** : The alarms contains a list of all currently unresolved alarms in the platform

## Working with Alarms

The most common usecase is to synchronise with the alarm system, which has two methods available depending on your requirements.

### Synchronise every alarm transition.

1. Set `lastUpdate` to 0
2. Query `/events?lastUpdate={lastUpdate}`
3. Process all the alarms
4. Save the `lastUpdate` value
5. Go to 2

### Get the current alarms and watch future events.

1. Query `/alarms`
2. Process all the alarms
3. Save the `lastUpdate` value
4. Query `/events?lastUpdate={lastUpdate}`
5. Process the alarms
6. Save the `lastUpdate` value
7. Go to 4

## Alarm Properties

Querying the `alarms` or `events` end point will return alarm events. The details of each property is discussed below

### EventID

Unique identifier for each alarm transition. Primarily used to identify which alarms are being acknowledged, and for the alarm system to forward
acknowledgements to the subsystems. Alarm events (EventID) are acknowledged, not alarms (SourceID) themselves. This to avoid race conditions where
an alarm that has recently transitioned can be interacted with before the even is displayed to the operator.
EventID is _always_ prefixed by the subsystem.

### SourceID

Unique identifier of the individual alarm state. An alarm can be in only one state at a time.
If the system is unmanaged, and an incoming alarm events with new EventIDs will replace the alarm that has the same `SourceID`.
If the system is managed, and an incoming alarm event with new States will replace the alarm that has the same `SourceID`.
`SourceID` is _always_ prefixed by the subsystem.

### ValueID

Unique identifier of the monitored cocntrol point. Primarily used to gather additional metadata.
ValueID is _always_ prefixed by the subsystem.

### State

The state is the condition of the alarm The alarm can be in four different states:

| State              | Description                                                  |
| ------------------ | ------------------------------------------------------------ |
| `0` (Invalid)      | Invalid state, transition is ignored                         |
| `1` (Active)       | The monitored status is currently in an abnormal condition   |
| `2` (Acknowledged) | The alarm has been acknowledged                              |
| `3` (Reset)        | The monitored status has returned to normal, but has not been acknowledged |
| `4` (Normal)       | The monitored status has returned to normal, and is not waiting to be acknowledged |

### Alarm Lifecycle

Alarms are configured to have a certain lifecycle. Depending on the configuration, the `Acknowledged` and `Reset` states
may be disabled so that operator interaction is not required to clear the alarms.

| Lifecycle       | Code | Description                                                  |
| --------------- | ---- | ------------------------------------------------------------ |
| `Unmanaged`     | `0`  | The subsystem is responsible for managing the alarm status. Acknowledging an alarm will be forwarded to the subsystem |
| `ManagedAck`    | `1`  | The platform will allow operators to acknowledge an alarm, even if the subsystem does not support it |
| `ManagedRst`    | `2`  | The platofrm will not allow operators to acknowledge an active alarm, and require an acknowledgement to clear the alarm from the system |
| `ManagedAckRst` | `3`  | The platform will allow operators to acknowledge an active alarm, and require an acknowledgement to clear the alarm from the system |

### CanAcknowledge

Can acknowledge indicates whether the alarm can be acknowledged by the operator. The value depends on the alarm lifecycle.

| CanAcknowledge | (State) Alarm Lifecycle                  | Description                                                  |
| -------------- | ---------------------------------------- | ------------------------------------------------------------ |
| `false`        | (any) `Unmanaged`                        | No operation                                                 |
| `true`         | (any) `Unmanaged`                        | Alarm acknowledgement is forwarded to the subsystem. The alarm state transition to `acknowledged` or `normal` must be sent from the subsystem |
| `true`         | (active) `ManagedAck` or `ManagedAckRst` | Alarm acknowledged event is generated in the platform        |
| `true`         | (reset) `ManagedRst` or `ManagedAckRst`  | Alarm normal event is generated in the platform              |

### CanResolve

The alarms from subsystem can get stuck in certain conditions. For example, if an alarm has been activated, then the subsystem is decomissioned - a reset alarm event may never be generated. In this scenario, an operator must manually resolve the alarm state. Typically this is reserved for operators with high level of permissions.

# Code Samples

## Examples
 
 https://github.com/ecostruxure-openapi/devportal-test

# Developer Guide

## How to sign up for the API

Before you can sign up for the `Alarms` API, you must [register or login with an Exchange account](https://exchange.se.com).
Then you can subscribe to the [API product in our Shop](https://shop.exchange.se.com/apps/87923) and you can read about Terms & Conditions. After subscription, you will receive further information on how to access the API including the API Key.

Status of your API consumption will soon be available from the [Exchange cockpit](https://exchange.se.com)

## Authentication guide

This API requires 3 keys to authenticate and allow access to the API.

1. SE Exchange API Subscription Key: 

   This key is delivered once the subscription to the API has been approved.
   This key should be included in all API requests in the **Authorization** header that looks like the following:

   | **Authorization:**      _your_subscription_key_


2. Integrated Management Platform API Account Code

  This UUID should be included in the `rimpAccount` HTTP Header.

3. Integrated Management Platform API Account Key

   This code should be included in the `rimpAPIKey` HTTP Header.

## Response Codes

We follow the error response format proposed in [RFC 7807](https://datatracker.ietf.org/doc/html/rfc7807) also known as Problem Details for HTTP APIs.  As with our normal API responses, your client must be prepared to gracefully handle additional members of the response.

### Unauthorized

<!--<RedocResponse pointer={"#/components/responses/Unauthorized"} />-->

### AccessForbidden

<!--<RedocResponse pointer={"#/components/responses/AccessForbidden"} />-->

### 400 - Bad Request

Returned when a request is not compliant with the API specification

### 401 - Unauthorised

Returned when a request authentication is incorrect

### 403 - Forbidden

Returned when a request authorization is insufficient to meet the request

### 404 - Not Found

The request is asking for a resource that was not found

### 429 - Too Many Requests

The client has exceeded the limit of concurrent requests

### 500 - Internal Server Error

There was a temporary error processing the request

## Support

Contact the Exchange support team at exchange.support@se.com.

In your request please :

	- mention the involved endpoint
	- give the request that generate the error
	- copy past any error message you received
	- add some screen shot

---

## Authentication

Authentication is the act of proving an assertion, such as the identify of a computer system user.

|                           |               |
| ------------------------- | ------------- |
| **Security scheme type**  | API Key       |
| **Header parameter name** | Authorization |

In contrast with identification, the act of indicating a person or thing's identity, authentication is the process of verifying that identify.

---

