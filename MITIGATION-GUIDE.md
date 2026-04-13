<!--
SPDX-FileCopyrightText: NOI Techpark <digital@noi.bz.it>

SPDX-License-Identifier: CC0-1.0
-->

# Mitigation Guide 

The objective is to migrate the responsibilities of the *EventShort* endpoint to the *Event* endpoint, ensuring that all services depend solely on the latter. To enable this transition, it was necessary to align and adapt the *EventShort* APIs so that they are fully compatible with the *Event* APIs.

## Fiels

The *Event* and *EventShort* entities have different data structures and use different field names. In order to reuse the existing structure, it was necessary to update the code to accommodate the new field names.

| Old Field| New Field |
|-------------------------|----------|
| `RoomStartDate` | `EventDate.FromUTC` |
| `RoomEndDate` | `EventDate.ToUTC` |
| `Subtitle` | `EventDate.EventDateAdditionalInfo.en.Description` |
| `EventWebAddress` | `contactinfo.en.Url` |
| `SpaceDesc` | `EventData.VenueRoomDetailsIds` |

#### <b>Room</b>

In the *Event* endpoint, the room ID is stored as an alphanumeric string. By sending an HTTP request to the Venues collector, it is possible to retrieve the corresponding room name associated with that ID.

## Query parameters
The Event endpoint also introduces different query parameters. The parameters that have changed are:

| Old Field | New Field | Type |
|-------------------------|----------|----------|
| `eventlocation` | `tagfilter` | `string` : `eventlocation:xxx`  |
| `startdate` | `begindate` | `timestamp` |
| `sortorder` | `sort` | `string` |
| `-` | `active` | `boolean` |
| `-` | `optimizedates` | `boolean` |
| `-` | `denormalize` | `boolean` |

## API

#### <b>Base</b>
To retrieve the current events a request is sent to the endoint : 
```
"https://tourism.api.opendatahub.testingmachine.eu/v1/Event?"
```

#### <b>Venues</b>
To obtain the room name of an event it will be sent a request to the endoint : 
```
"https://tourism.api.opendatahub.testingmachine.eu/v1/Venue?"
```
It ensures that the name of a room can be retrieved dynamically