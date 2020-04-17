# The FastyBird MQTT convention

A lightweight MQTT convention for the smart IoT ecosystem

----

## Table of Contents

* [Motivation](#motivation)
* [MQTT restrictions](#mqtt-restrictions)
  * [Topic IDs](#topic-ids)
  * [Payload](#payload)
  * [QoS and retained messages](#qos-and-retained-messages)
  * [Base topic](#base-topic)
* [Example](#example)
* [Topology](#topology)
  * [Devices](#devices)
    * [Device attributes](#device-attributes)
    * [Device lifecycle](#device-lifecycle)
  * [Channels](#channels)
    * [Channel attributes](#channel-attributes)
  * [Properties](#properties)
    * [Property attributes](#property-attributes)
  * [Broadcast channel](#broadcast-channel)
* [Extensions](#extensions)

types are UTF-8 encoded string literal representations of 64-bit signed floating point numbers

## Motivation

We would like to open FastyBird IoT ecosystem to the wide community of users and therefore we have to provide communication API documentation we are using.

> [MQTT](http://mqtt.org) is a machine-to-machine (M2M)/"Internet of Things" connectivity protocol.
> It was designed as an extremely lightweight publish/subscribe messaging transport.

MQTT supports easy and unrestricted message-based communication. However, MQTT doesn't define the structure and content of these messages and their relation.
An IoT devices publishes data and provides interaction possibilities but a controlling entity will need to be specifically configured to be able to interface with devices.

The FastyBird MQTT communication convention defines a **standardized way** of how internet connected devices and services announce themselves and their data on the communication channel.
The FastyBird MQTT communication convention is thereby a crucial aspect in the support of **automatic discovery**, **configuration** and **usage** of devices and services over the MQTT protocol.

----

## MQTT Restrictions

FastyBird IoT services communicates through [MQTT protocol](http://mqtt.org) and is hence based on the basic principles of MQTT topic publication and subscription.

### Topic IDs

An MQTT topic consists of one or more topic levels, starting with the slash (`/`) and separated by the slash character (`/`).
A topic level ID MAY contain lowercase letters from `a` to `z`, numbers from `0` to `9` as well as the hyphen character (`-`).

A topic level ID MUST NOT start or end with a hyphen (`-`).

A topic level ID MUST start with a slash (`/`).

> **NOTE:** The special character `$` is used and reserved for *attributes* and topic IDs separation.

### Payload

Every MQTT message payload MUST be sent as **string**. If a value is of a numeric data type, it MUST be converted to string.
Booleans MUST be converted to "true" or "false". All values MUST be encoded as UTF-8 strings. 

### QoS and retained messages

MQTT protocol support 3 levels of QoS from QoS 0 to QoS 2. The nature of the convention makes it safe about duplicate messages, so the recommended QoS for reliability is **QoS 1**.

All messages MUST be sent as **retained**, UNLESS stated otherwise.
 
### Base topic

The base topic you will see in the following convention will be `/fb/v1/`.

* `/fb` is to identify that this topic is according to FastyBird MQTT convention
* `/v1` is to identify what version is used to be able support newer versions

## Example

FastyBird MQTT convetion topic layout follow basic pattern `/fb/v1/device/....`

In the example bellow, device is a thermostat with relays for controlling heater. Temperature can be read from and written to, humidity can be only read and two switches can be read and set. 

Thermostat is a **channel** and temperature and humidity is a **property** of that channel. And switch is channel to with one property switch.

```
  / fb / v1 / device-name / $name → My device
  / fb / v1 / device-name / $properties → state,ip-address,battery
  / fb / v1 / device-name / $channels → thermostat,switch

  / fb / v1 / device-name / $property / state → ready
  / fb / v1 / device-name / $property / ip-address → 192.168.1.2
  / fb / v1 / device-name / $property / battery → 83

  / fb / v1 / device-name / $channel / thermostat / $name → Room thermostat
  / fb / v1 / device-name / $channel / thermostat / $properties → temperature,humidity

  / fb / v1 / device-name / $channel / thermostat / $property / temperature / $name → Temperature
  / fb / v1 / device-name / $channel / thermostat / $property / temperature / $unit → °C
  / fb / v1 / device-name / $channel / thermostat / $property / temperature / $datatype → integer
  / fb / v1 / device-name / $channel / thermostat / $property / temperature / $settable → true
  / fb / v1 / device-name / $channel / thermostat / $property / temperature / $queryable → true

  / fb / v1 / device-name / $channel / thermostat / $property / temperature  → 22

  / fb / v1 / device-name / $channel / thermostat / $property / humidity / $name → Humidity
  / fb / v1 / device-name / $channel / thermostat / $property / humidity / $unit → %
  / fb / v1 / device-name / $channel / thermostat / $property / humidity / $datatype → integer
  / fb / v1 / device-name / $channel / thermostat / $property / humidity / $settable → false
  / fb / v1 / device-name / $channel / thermostat / $property / humidity / $queryable → true

  / fb / v1 / device-name / $channel / thermostat / $property / humidity  → 60

  / fb / v1 / device-name / $channel / switch / $name → Heating switches
  / fb / v1 / device-name / $channel / switch / $properties → relay

  / fb / v1 / device-name / $channel / switch / $property / relay / $name → Realy switch
  / fb / v1 / device-name / $channel / switch / $property / relay / $unit → boolean
  / fb / v1 / device-name / $channel / switch / $property / relay / $datatype → boolean
  / fb / v1 / device-name / $channel / switch / $property / relay / $settable → true
  / fb / v1 / device-name / $channel / switch / $property / relay / $queryable → true

  / fb / v1 / device-name / $channel / switch / $property / relay  → true
```

All this messages are published as retained and with QOS 1.

Any FastyBird MQTT convention compliant controller can now identify device with identifier **device-name** (unique device name) with name *My device*. This device has one *thermostat* channel and one *switch* channel.

----

## Topology

### Devices

An instance of a physical piece of hardware is called a *device*. For example, a light switch, a coffee machinem garden sprinkler or an Arduino or ESP8266 custom device.

##### Topic format

* / `fb` / `v1` / **`device-identifier`**

Each device MUST have a unique ID which adhere to the [ID format](#topic-ids).

#### Device attributes

*Devices* have specific *attributes* characterizing them. Attributes are represented by topic identifier starting with `$`.
The precise definition of attributes is important for the automatic discovery of devices following the convention.
For example a device might have an `state` attribute and a `name` attribute.

##### Topic format

* / `fb` / `v1` / `device-identifier` / **`$device-attribute`**:

When the connection to the broker is established or re-established, device MUST send its attributes to the broker immediately!

<table style="display: table">
    <tr>
        <th align="left">Attribute</th>
        <th align="left">Direction</th>
        <th align="left">Description</th>
        <th align="center">Retained</th>
        <th align="center">Required</th>
    </tr>
    <tr>
        <td align="left">$name</td>
        <td align="left">Device → Controller</td>
        <td align="left">
            Friendly name of the device
        </td>
        <td align="center">Yes</td>
        <td align="center">Yes</td>
    </tr>
    <tr>
        <td align="left">$state</td>
        <td align="left">Device → Controller</td>
        <td align="left">
            Device current state. See <a href="#device-lifecycle">device lifecycle</a>
        </td>
        <td align="center">Yes</td>
        <td align="center">Yes</td>
    </tr>
    <tr>
        <td align="left">$properties</td>
        <td align="left">Device → Controller</td>
        <td align="left">
            Exposed properties, separated by <code>,</code> for multiple ones
        </td>
        <td align="center">Yes</td>
        <td align="center">Yes</td>
    </tr>
    <tr>
        <td align="left">$channels</td>
        <td align="left">Device → Controller</td>
        <td align="left">
            Channels the device exposes, separated by <code>,</code> for multiple ones.
        </td>
        <td align="center">Yes</td>
        <td align="center">Yes</td>
    </tr>
    <tr>
        <td align="left">$extensions</td>
        <td align="left">Device → Controller</td>
        <td align="left">
            Implemented <a href="#extensions">extensions</a>, separated by <code>,</code> for multiple ones.
        </td>
        <td align="center">Yes</td>
        <td align="center">No</td>
    </tr>
</table>

For example, a smart switch device with an ID of `room-smart-switch` that comprises off a `button` and a `switch` channels would send:

```
/fb/v1/room-smart-switch/$state → "init"

/fb/v1/room-smart-switch/$name → "Smart switch"
/fb/v1/room-smart-switch/$properties → "battery,ip-address"
/fb/v1/room-smart-switch/$channels → "button,switch"

/fb/v1/room-smart-switch/$state → "ready"
```

#### Device lifecycle

The `state` device attribute represents, as the name suggests, the current state of the device.

There are 6 different states:

* **`init`**: this is the state the device is in when it is connected to the MQTT broker, but has not yet sent all convention messages and is not yet ready to operate.
This is the first message that must that must be sent.
* **`ready`**: this is the state the device is in when it is connected to the MQTT broker, has sent all convention messages and is ready to operate.
Device have to send this message after all other announcements message have been sent.
* **`disconnected`**: this is the state the device is in when it is cleanly disconnected from the MQTT broker.
Device must send this message before cleanly disconnecting.
* **`sleeping`**: this is the state the device is in when the device is sleeping.
Device have to send this message before sleeping.
* **`lost`**: this is the state the device is in when the device has been "badly" disconnected.
Device must define this message as LWT.
* **`alert`**: this is the state the device is when connected to the MQTT broker, but something wrong is happening. E.g. a sensor is not providing data and needs human intervention.
Device have to send this message when something is wrong.

----

### Channels

A *device* can expose multiple *channels*. Channels are independent or logically separable parts of a device.
For example, a light device might expose a `switch` channel, a `color` channel and a `light-temperature` channel.

##### Topic format

* / `fb` / `v1` / `device-identifier` / `$channel` / **`channel-identifier`**

Each channel MUST be prefixed with `$channel` topic identifier

Each channel MUST have a unique channel ID on a per-device basis which adhere to the [ID format](#topic-ids).

#### Channel Attributes

*Channels* have specific *attributes* characterizing them. Attributes are represented by topic identifier starting with `$`.
The precise definition of attributes is important for the automatic discovery of devices following the convention.
For example a channel might have an `name` attribute and a `properties` attribute.

##### Topic format

* / `fb` / `v1` / `device-identifier` / `$channel` / `channel-identifier` / **`$channel-attribute`**:

A channel attribute MUST be one of these:

<table style="display: table">
    <tr>
        <th align="left">Attribute</th>
        <th align="left">Direction</th>
        <th align="left">Description</th>
        <th align="center">Retained</th>
        <th align="center">Required</th>
    </tr>
    <tr>
        <td align="left">$name</td>
        <td align="left">Device → Controller</td>
        <td align="left">
            Friendly name of the Channel
        </td>
        <td align="center">Yes</td>
        <td align="center">Yes</td>
    </tr>
    <tr>
        <td align="left">$properties</td>
        <td align="left">Device → Controller</td>
        <td align="left">
            Exposed properties, separated by <code>,</code> for multiple ones
        </td>
        <td align="center">Yes</td>
        <td align="center">Yes</td>
    </tr>
</table>

For example, our `light` channel would send:

```
/fb/v1/room-smart-light/$channel/light/$name → "Smart light"
/fb/v1/room-smart-light/$channel/light/$properties → "status,mode"
```

----

### Properties

A *device* and *channel* can have multiple *properties*. Properties represent basic characteristics of the device or channel, often given as numbers or finite states.
For example the `thermostat` channel might expose an `temperature` property. The `environment` channel might expose a `temperature`, `humidity` and `air-quality` property.
The `lights` channel might expose an `intensity` and a `color` property.

* Properties can be **settable**. For example, you don't want your `temperature` property to be settable in case of a temperature sensor (like the environment example), but to be settable in case of a thermostat.
* Properties can be **queryable**. That means that controller will ask device for actual property value.
* Properties values have to be send as **retained** by default. A **non-retained** property would be send for momentary events (button pressed).

##### Topic format

* / `fb` / `v1` / `device-identifier` / `$property` / **`property-identifier`**
* / `fb` / `v1` / `device-identifier` / `$channel` / `channel-identifier` / `$property` / **`property-identifier`**

Each property MUST be prefixed with `$property` topic identifier

Each property MUST have a unique property ID on a per-device|per-channel basis which adhere to the [ID format](#topic-ids).

* A property value (e.g. a sensor reading) is directly published to the property topic, e.g.:

```
/fb/v1/environment-device/$channel/thermostat/$property/temperature → "21.5"
```

```
/fb/v1/environment-device/$property/battery → "low"
/fb/v1/environment-device/$property/ip-address → "192.168.2.2"
```

#### Property Attributes

*Properties* have specific *attributes* characterizing them. Attributes are represented by topic identifier starting with `$`.
The precise definition of attributes is important for the automatic discovery of devices following the convention.
For example a property might have an `name` attribute, a `type` attribute and a `unit` attribute.

##### Topic format

* / `fb` / `v1` / `device-identifier` / `$property` / `property-identifier` / **`$property-attribute`**
* / `fb` / `v1` / `device-identifier` / `$channel` / `channel-identifier` / `$property` / `property-identifier` / **`$property-attribute`**

The following attributes are required:

<table style="display: table">
    <tr>
        <th align="left" valign="top">Topic</th>
        <th align="left" valign="top">Direction</th>
        <th align="left" valign="top">Description</th>
        <th align="center" valign="top">Retained</th>
        <th align="center" valign="top">Required<br><small>(Default)</small></th>
    </tr>
    <tr>
        <td align="left">$name</td>
        <td align="left">Device → Controller</td>
        <td align="left">
            Friendly name of the property
        </td>
        <td align="center">Yes</td>
        <td align="center">No<br><small>[""]</small></td>
    </tr>
    <tr>
        <td align="left">$datatype</td>
        <td align="left">Device → Controller</td>
        <td align="left">
            Describes the format of data
        </td>
        <td align="center">Yes</td>
        <td align="center">No<br><small>[<code>string</code>]</small></td>
    </tr>
</table>

The following attributes are optional:

<table style="display: table">
    <tr>
        <th align="left" valign="top">Topic</th>
        <th align="left" valign="top">Direction</th>
        <th align="left" valign="top">Description</th>
        <th align="center" valign="top">Retained</th>
        <th align="center" valign="top">Required<br><small>(Default)</small></th>
    </tr>
    <tr>
        <td align="left">$settable</td>
        <td align="left">Device → Controller</td>
        <td align="left">
            Specifies whether the property is settable (<code>true</code>) or readonly (<code>false</code>)
        </td>
        <td align="center">Yes</td>
        <td align="center">No<br><small>[<code>false</code>]</small></td>
    </tr>
    <tr>
        <td align="left">$queryable</td>
        <td align="left">Device → Controller</td>
        <td align="left">
            Specifies whether the property value reading could be requested by controller
        </td>
        <td align="center">Yes</td>
        <td align="center">No<br><small>[<code>false</code>]</small></td>
    </tr>
    <tr>
        <td align="left">$unit</td>
        <td align="left">Device → Controller</td>
        <td align="left">
            A string containing the unit of this property.
            You are not limited to the recommended values, although they are the only well known ones that will have to be recognized by FastyBird controller.
        </td>
        <td align="center">Yes</td>
        <td align="center">No<br><small>[<code>""</code>]</small></td>
    </tr>
    <tr>
        <td align="left">$format</td>
        <td align="left">Device → Controller</td>
        <td align="left">
            Describes what are valid values for this property
        </td>
        <td align="center">Yes</td>
        <td align="center"><sup>1</sup>)</td>
    </tr>
</table>

> <sup>1</sup>) No for $datatype <code>string</code>,<code>integer</code>,<code>float</code>,<code>boolean</code>. Yes for <code>enum</code>,<code>color</code>

For example, our channel `temperature` property would send:

```
/fb/v1/environment-device/$channel/thermostat/$property/temperature/$name → "Room temperature"
/fb/v1/environment-device/$channel/thermostat/$property/temperature/$settable → "false"
/fb/v1/environment-device/$channel/thermostat/$property/temperature/$queryable → "true"
/fb/v1/environment-device/$channel/thermostat/$property/temperature/$unit → "°C"
/fb/v1/environment-device/$channel/thermostat/$property/temperature/$datatype → "float"
/fb/v1/environment-device/$channel/thermostat/$property/temperature/$format → "-20:120"

/fb/v1/environment-device/$channel/thermostat/$property/temperature → "21.5"
```

##### Supported datatypes

* `string` for plain strings
* `integer` for plain numbers
* `float` for numbers with defined precision
* `boolean` for two states properties
* `enum`
* `color` special datatype for light properties

##### Supported format definitions

* `from:to` Describes a range of values e.g. `10:15`. Valid for datatypes `integer`, `float`
* `value,value,value` for enumerating all valid values. E.g. `A,B,C` or `ON,OFF,PAUSE`. Valid for datatypes `enum`
* `rgb` to provide colors in RGB format e.g. `255,255,0` for yellow. Valid for datatype `color`
* `hsv` to provide colors in HSV format e.g. `60,100,100` for yellow. Valid for datatype `color`

##### Recommended unit strings

* `°C` Degree Celsius
* `°F` Degree Fahrenheit
* `°` Degree
* `L` Liter
* `gal` Galon
* `V` Volts
* `W` Watt
* `A` Ampere
* `%` Percent
* `m` Meter
* `ft` Feet
* `Pa` Pascal
* `psi` PSI
* `#` Count or Amount

#### Property command topic

Device can subscribe to this topic if the property is **settable** from the controller, in case of actuators.

* / `fb` / `v1` / `device-identifier` / `$property` / `property-identifier` / **`set`**
* / `fb` / `v1` / `device-identifier` / `$channel` / `channel-identifier` / `$property` / `property-identifier` / **`set`**

A controller publishes to the `set` command topic with non-retained messages only.

The assigned and processed payload must be reflected by the device in the property topic / `fb` / `v1` / `device ID` / `$channel` / `channel ID` / `$property` / `property ID` as soon as possible.
This property state update not only informs other devices about the change but closes the control loop for the commanding controller, important for deterministic interaction with the client device.

For example, a `smart-lights` device exposing a `kitchen-light` channel would subscribe to `/fb/v1/smart-lights/$channel/kitchen-light/$property/power/set` and it would receive:

```
/fb/v1/smart-lights/$channel/kitchen-light/$property/power/set ← "true"
```

Device would then turn on the light, and update its `power` state.
This provides pessimistic feedback, which is important for automation.

```
/fb/v1/smart-lights/$channel/kitchen-light/$property/power → "true"
```

----

### Broadcast channel

FastyBird MQTT convention defines a broadcast channel, so a controller is able to broadcast a message to all devices:

* / `fb` / `v1` / `$broadcast` / **`level`**: `level` is an arbitrary broadcast identifier.
It must adhere to the [ID format](#topic-ids).

For example, you might want to broadcast an `alert` event with the alert reason as the payload.
Things are then free to react or not.
In our case, every buzzer of your home automation system would start buzzing.

```
/fb/v1/$broadcast/alert ← "Intruder detected"
```

## Extensions

This convention covers only basic devices description and capabilities. The aim is to have simple and straightforward standardized MQTT topics for all kind of complex scenarios.
A device may therefore support extensions, defined in separate documents. Every extension MUST be identified by a unique ID.

The ID consists of the reverse domain name and a freely chosen suffix.

For example, an organization `smart-example.org` wanting to add a feature `our-feature` would choose the extension ID `org.smart-example.our-feature`.

#### Supported extensions

* [Firmware info](https://github.com/FastyBird/convention/blob/master/extensions/com.fastybird.firmware.md) helps describe device's firmware
* [Hardware info](https://github.com/FastyBird/convention/blob/master/extensions/com.fastybird.hardware.md) helps describe device's hardware

***
Homepage [http://www.fastybird.com](http://www.fastybird.com) and repository [http://github.com/fastybird/mqtt-convention](http://github.com/fastybird/mqtt-convention).
