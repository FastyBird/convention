<p align="center">
 <img src="https://github.com/FastyBird/convention/blob/master/fastybird_logo.png">
</p>

# The FastyBird IoT convention

A lightweight MQTT convention for the FastyBird IoT ecosystem

----

## Table of Contents

* [Motivation](#motivation)
* [MQTT restrictions](#mqtt-restrictions)
  * [Topic IDs](#topic-ids)
  * [Payload](#payload)
  * [QoS and retained messages](#qos-and-retained-messages)
* [Topology](#topology)
  * [Base topic](#base-topic)
  * [Things](#things)
    * [Thing attributes](#thing-attributes)
    * [Thing behavior](#thing-behavior)
    * [Thing statistics](#thing-statistics)
  * [Channels](#channels)
    * [Channel attributes](#channel-attributes)
  * [Properties](#properties)
    * [Property attributes](#property-attributes)
  * [Arrays](#arrays)
  * [Broadcast channel](#broadcast-channel)


## Motivation

We would like to open FastyBird IoT ecosystem to the wide community of users and therefore we have to provide communication API documentation we are using.

> [MQTT](http://mqtt.org) is a machine-to-machine (M2M)/"Internet of Things" connectivity protocol.
> It was designed as an extremely lightweight publish/subscribe messaging transport.

MQTT supports easy and unrestricted message-based communication.
However, MQTT doesn't define the structure and content of these messages and their relation.
An IoT smart thing publishes data and provides interaction possibilities but a controlling entity will need to be specifically configured to be able to interface with the smart thing.

The FastyBird IoT communication convention defines a **standardized way** of how IoT smart things and services announce themselves and their data on the communication channel.
The FastyBird IoT communication convention is thereby a crucial aspect in the support of **automatic discovery, configuration and usage** of smart things and services over the MQTT protocol.

----

## MQTT Restrictions

FastyBird IoT communicates through [MQTT](http://mqtt.org) and is hence based on the basic principles of MQTT topic publication and subscription.

### Topic IDs

An MQTT topic consists of one or more topic levels, starting with the slash (`/`) and separated by the slash character (`/`).
A topic level ID MAY contain lowercase letters from `a` to `z`, numbers from `0` to `9` as well as the hyphen character (`-`).

A topic level ID MUST NOT start or end with a hyphen (`-`).
The special character `$` is used and reserved for FastyBird IoT *attributes*.
The underscore (`_`) is used and reserved for FastyBird IoT *channels arrays*.

### Payload

Every MQTT message payload MUST be sent as string.
If a value is of a numeric data type, it MUST be converted to string.
Booleans MUST be converted to "true" or "false".
All values MUST be encoded as UTF-8 strings. 

### QoS and retained messages

The nature of the FastyBird IoT convention makes it safe about duplicate messages, so the recommended QoS for reliability is **QoS 1**.
All messages MUST be sent as **retained**, UNLESS stated otherwise.

## Topology

**Smart things:**
An instance of a physical piece of hardware is called a *thing*.
For example, a light switch, an Arduino/ESP8266 or a coffee machine.

**Channels:**
A *thing* can expose multiple *channels*.
Channels are independent or logically separable parts of a thing.
For example, a light might expose a `switch` channel, a `color` channel and a `temperature` channel.

Channels can be **arrays**.
For example, instead of creating two `switches` channels to control two relays independently, we can set the `switches` channel to be an array with two elements.

**Properties:**
A *channel* can have multiple *properties*.
Properties represent basic characteristics of the channel/thing, often given as numbers or finite states.
For example the `thermostat` channel might expose an `temperature` property.
The `environment` channel might expose a `temperature`, `humidity` and `air-quality` property.
The `lights` channel might expose an `intensity` and a `color` property.

Properties can be **settable**.
For example, you don't want your `temperature` property to be settable in case of a temperature sensor (like the environment example), but to be settable in case of a thermostat.

**Attributes:**
*Things, channels and properties* have specific *attributes* characterizing them.
Attributes are represented by topic identifier starting with `$`.
The precise definition of attributes is important for the automatic discovery of things following the FastyBird IoT convention.

Examples: A thing might have an `IP` attribute, a channel will have a `name` attribute, and a property will have a `unit` attribute.

----

### Base Topic

The base topic you will see in the following convention will be `/fastybird/`.

----

### Things

* / `fastybird` / **`thing ID`**: this is the base topic of a smart thing.

Each thing has a unique thing ID which is generated in FastyBird system.

#### Thing Attributes

* / `fastybird` / `thing ID` / **`$thing-attribute`**:

When the MQTT connection to the broker is established or re-established, the thing MUST send its attributes to the broker immediately!

<table>
  <tr>
    <th>Topic</th>
    <th>Direction</th>
    <th>Description</th>
    <th>Retained</th>
    <th>Required</th>
  </tr>
  <tr>
    <td>$name</td>
    <td>Thing → Controller</td>
    <td>Friendly name of the thing</td>
    <td>Yes</td>
    <td>Yes</td>
  </tr>
  <tr>
    <td>$state</td>
    <td>Thing → Controller</td>
    <td>
      See <a href="#thing-behavior">Thing behavior</a>
    </td>
    <td>Yes</td>
    <td>Yes</td>
  </tr>
  <tr>
    <td>$local-ip</td>
    <td>Thing → Controller</td>
    <td>IP addres of the thing on the local network</td>
    <td>Yes</td>
    <td>Yes</td>
  </tr>
  <tr>
    <td>$mac-address</td>
    <td>Thing → Controller</td>
    <td>Mac address of the thing network interface. The format MUST be of the type <code>A1:B2:C3:D4:E5:F6</code></td>
    <td>Yes</td>
    <td>Yes</td>
  </tr>
  <tr>
    <td>$fw/name</td>
    <td>Thing → Controller</td>
    <td>Name of the firmware running on the thing. Allowed characters are the same as the thing ID</td>
    <td>Yes</td>
    <td>Yes</td>
  </tr>
  <tr>
    <td>$fw/version</td>
    <td>Thing → Controller</td>
    <td>Version of the firmware running on the thing</td>
    <td>Yes</td>
    <td>Yes</td>
  </tr>
  <tr>
    <td>$channels</td>
    <td>Thing → Controller</td>
    <td>
      Channels the thing exposes, with format <code>id</code> separated by a <code>,</code> if there are multiple channels.
      To make a channel an array, append <code>[]</code> to the ID.
    </td>
    <td>Yes</td>
    <td>Yes</td>
  </tr>
  <tr>
    <td>$implementation</td>
    <td>Thing → Controller</td>
    <td>An identifier for the FastyBird implementation (example <code>esp8266</code>)</td>
    <td>Yes</td>
    <td>Yes</td>
  </tr>
  <tr>
    <td>$implementation/#</td>
    <td>Controller → Thing or Thing → Controller</td>
    <td>You can use any subtopics of <code>$implementation</code> for anything related to your specific FastyBird IoT implementation.</td>
    <td>Yes or No, depending of your implementation</td>
    <td>No</td>
  </tr>
  <tr>
    <td>$stats</td>
    <td>Thing → Controller</td>
    <td>Specify all optional stats that the thing will announce, with format <code>stats</code> separated by a <code>,</code> if there are multiple stats. See next section for an example</td>
    <td>Yes</td>
    <td>Yes</td>
  </tr>
  <tr>
    <td>$stats/interval</td>
    <td>Thing → Controller</td>
    <td>Interval in seconds at which the thing refreshes its <code>$stats/+</code>: See next section for details about statistical attributes</td>
    <td>Yes</td>
    <td>Yes</td>
  </tr>
</table>

For example, a smart switch thing with an ID of `9222852d-1fc4-447b-9967-f8d82fbc4a6f` that comprises off a `buttons`, `status-led` and a `switches` channels would send:

```java
/fastybird/9222852d-1fc4-447b-9967-f8d82fbc4a6f/$version → "2.0"
/fastybird/9222852d-1fc4-447b-9967-f8d82fbc4a6f/$name → "Smart switch"
/fastybird/9222852d-1fc4-447b-9967-f8d82fbc4a6f/$localip → "192.168.0.10"
/fastybird/9222852d-1fc4-447b-9967-f8d82fbc4a6f/$mac → "DE:AD:BE:EF:FE:ED"
/fastybird/9222852d-1fc4-447b-9967-f8d82fbc4a6f/$fw/name → "smart-switch-firmware"
/fastybird/9222852d-1fc4-447b-9967-f8d82fbc4a6f/$fw/version → "1.0.0"
/fastybird/9222852d-1fc4-447b-9967-f8d82fbc4a6f/$channels → "buttons[],status-led,switches[]"
/fastybird/9222852d-1fc4-447b-9967-f8d82fbc4a6f/$implementation → "esp8266"
/fastybird/9222852d-1fc4-447b-9967-f8d82fbc4a6f/$stats/interval → "60"
/fastybird/9222852d-1fc4-447b-9967-f8d82fbc4a6f/$state → "ready"
```

#### Thing Behavior

The `$state` thing attribute represents, as the name suggests, the current state of the thing.

There are 6 different states:

* **`init`**: this is the state the thing is in when it is connected to the MQTT broker, but has not yet sent all FastyBird IoT messages and is not yet ready to operate.
This is the first message that must that must be sent.
* **`ready`**: this is the state the thing is in when it is connected to the MQTT broker, has sent all FastyBird IoT messages and is ready to operate.
You have to send this message after all other announcements message have been sent.
* **`disconnected`**: this is the state the thing is in when it is cleanly disconnected from the MQTT broker.
You must send this message before cleanly disconnecting.
* **`sleeping`**: this is the state the thing is in when the thing is sleeping.
You have to send this message before sleeping.
* **`lost`**: this is the state the thing is in when the thing has been "badly" disconnected.
You must define this message as LWT.
* **`alert`**: this is the state the thing is when connected to the MQTT broker, but something wrong is happening. E.g. a sensor is not providing data and needs human intervention.
You have to send this message when something is wrong.

#### Thing Statistics

* / `fastybird` / `thing ID` / `$stats`/ **`$thing-statistic-attribute`**:

The `$stats/` hierarchy allows to send thing attributes that change over time. The thing MUST send them every `$stats/interval` seconds.

<table>
  <tr>
    <th>Topic</th>
    <th>Direction</th>
    <th>Description</th>
    <th>Retained</th>
    <th>Required</th>
  </tr>
  <tr>
    <td>$stats/uptime</td>
    <td>Thing → Controller</td>
    <td>Time elapsed in seconds since the boot of the thing</td>
    <td>Yes</td>
    <td>Yes</td>
  </tr>
  <tr>
    <td>$stats/signal</td>
    <td>Thing → Controller</td>
    <td>Signal strength in %</td>
    <td>Yes</td>
    <td>No</td>
  </tr>
  <tr>
    <td>$stats/cpu-temperature</td>
    <td>Thing → Controller</td>
    <td>CPU Temperature in °C</td>
    <td>Yes</td>
    <td>No</td>
  </tr>
  <tr>
    <td>$stats/cpu-load</td>
    <td>Thing → Controller</td>
    <td>
      CPU Load in %.
      Average of last <code>$interval</code> including all CPUs
    </td>
    <td>Yes</td>
    <td>No</td>
  </tr>
  <tr>
    <td>$stats/battery</td>
    <td>Thing → Controller</td>
    <td>Battery level in %</td>
    <td>Yes</td>
    <td>No</td>
  </tr>
  <tr>
    <td>$stats/free-heap</td>
    <td>Thing → Controller</td>
    <td>Free heap in bytes</td>
    <td>Yes</td>
    <td>No</td>
  </tr>
  <tr>
    <td>$stats/supply</td>
    <td>Thing → Controller</td>
    <td>Supply Voltage in V</td>
    <td>Yes</td>
    <td>No</td>
  </tr>
  <tr>
    <td>$stats/ssid</td>
    <td>Thing → Controller</td>
    <td>Wifi network SSID name</td>
    <td>Yes</td>
    <td>No</td>
  </tr>
  <tr>
    <td>$stats/rssi</td>
    <td>Thing → Controller</td>
    <td>Wifi network RSSI</td>
    <td>Yes</td>
    <td>No</td>
  </tr>
</table>

For example, our smart switch `9222852d-1fc4-447b-9967-f8d82fbc4a6f` thing with `$stats/interval` value "60" is supposed to send its current values every 60 seconds:

```java
/fastybird/9222852d-1fc4-447b-9967-f8d82fbc4a6f/$stats → "uptime,cpu-temperature,signal,battery"
/fastybird/9222852d-1fc4-447b-9967-f8d82fbc4a6f/$stats/uptime → "120"
/fastybird/9222852d-1fc4-447b-9967-f8d82fbc4a6f/$stats/cpu-temperature → "48"
/fastybird/9222852d-1fc4-447b-9967-f8d82fbc4a6f/$stats/signal → "24"
/fastybird/9222852d-1fc4-447b-9967-f8d82fbc4a6f/$stats/battery → "80"
```

----

### Channels

* / `fastybird` / `thing ID` / **`channel ID`**: this is the base topic of a channel:

Each channel must have a unique channel ID on a per-thing basis which adhere to the [ID format](#topic-ids).

#### Channel Attributes

* / `fastybird` / `thing ID` / `channel ID` / **`$channel-attribute`**:

A channel attribute MUST be one of these:

<table>
  <tr>
    <th>Topic</th>
    <th>Direction</th>
    <th>Description</th>
    <th>Retained</th>
    <th>Required</th>
  </tr>
  <tr>
    <td>$name</td>
    <td>Thing → Controller</td>
    <td>Friendly name of the Channel</td>
    <td>Yes</td>
    <td>Yes</td>
  </tr>
  <tr>
    <td>$type</td>
    <td>Thing → Controller</td>
    <td>Type of the channel</td>
    <td>Yes</td>
    <td>Yes</td>
  </tr>
  <tr>
    <td>$properties</td>
    <td>Thing → Controller</td>
    <td>
      Properties the channel exposes, with format <code>id</code> separated by a <code>,</code> if there are multiple channels.
    </td>
    <td>Yes</td>
    <td>Yes</td>
  </tr>
    <tr>
       <td>$array</td>
       <td>Thing → Controller</td>
       <td>Range separated by a <code>-</code>. e.g. <code>0-2</code> for an array with the indexes <code>0</code>, <code>1</code> and <code>2</code></td>
       <td>Yes</td>
       <td>Yes, if the channel is an array</td>
    </tr>
</table>

For example, our `status-led` channel would send:

```java
/fastybird/9222852d-1fc4-447b-9967-f8d82fbc4a6f/status-led/$name → "Switch status led"
/fastybird/9222852d-1fc4-447b-9967-f8d82fbc4a6f/status-led/$type → "led"
/fastybird/9222852d-1fc4-447b-9967-f8d82fbc4a6f/status-led/$properties → "status,mode"
```

----

### Properties

* / `fastybird` / `thing ID` / `channel ID` / **`property ID`**: this is the base topic of a property.

Each property must have a unique property ID on a per-channel basis which adhere to the [ID format](#topic-ids).

* A property value (e.g. a sensor reading) is directly published to the property topic, e.g.:

```java
/fastybird/9222852d-1fc4-447b-9967-f8d82fbc4a6f/thermostat/temperature → "21.5"
```

#### Property Attributes

* / `fastybird` / `thing ID` / `channel ID` / `property ID` / **`$property-attribute`**:

A property attribute MUST be one of these:

<table>
    <tr>
        <th>Topic</th>
        <th>Direction</th>
        <th>Description</th>
        <th>Valid values</th>
        <th>Retained</th>
        <th>Required (Default)</th>
    </tr>
    <tr>
       <td>$name</td>
       <td>Thing → Controller</td>
       <td>Friendly name of the property.</td>
       <td>Any String</td>
       <td>Yes</td>
       <td>No ("")</td>
    </tr>
    <tr>
        <td>$settable</td>
        <td>Thing → Controller</td>
        <td>Specifies whether the property is settable (<code>true</code>) or readonly (<code>false</code>)</td>
        <td><code>true</code> or <code>false</code></td>
        <td>Yes</td>
        <td>No (<code>false</code>)</td>
    </tr>
    <tr>
        <td>$unit</td>
        <td>Thing → Controller</td>
        <td>
          A string containing the unit of this property.
          You are not limited to the recommended values, although they are the only well known ones that will have to be recognized by FastyBird controller.
        </td>
        <td>
            Recommended:<br>
            <code>°C</code> Degree Celsius<br>
            <code>°F</code> Degree Fahrenheit<br>
            <code>°</code> Degree<br>
            <code>L</code> Liter<br>
            <code>gal</code> Galon<br>
            <code>V</code> Volts<br>
            <code>W</code> Watt<br>
            <code>A</code> Ampere<br>
            <code>%</code> Percent<br>
            <code>m</code> Meter<br>
            <code>ft</code> Feet<br>
            <code>Pa</code> Pascal<br>
            <code>psi</code> PSI<br>
            <code>#</code> Count or Amount
        </td>
        <td>Yes</td>
        <td>No ("")</td>
    </tr>
    <tr>
       <td>$datatype</td>
       <td>Thing → Controller</td>
       <td>Describes the format of data.</td>
       <td>
         <code>integer</code>,
         <code>float</code>,
         <code>boolean</code>,
         <code>string</code>,
         <code>enum</code>,
         <code>color</code>
       </td>
       <td>Yes</td>
       <td>No (<code>string</code>)</td>
    </tr>
    <tr>
       <td>$format</td>
       <td>Thing → Controller</td>
       <td>
        Describes what are valid values for this property.
       </td>
       <td>
         <ul>
           <li>
             <code>from:to</code> Describes a range of values e.g. <code>10:15</code>.
             <br>Valid for datatypes <code>integer</code>, <code>float</code>
           </li>
           <li>
             <code>value,value,value</code> for enumerating all valid values.
             Escape <code>,</code> by using <code>,,</code>. e.g. <code>A,B,C</code> or <code>ON,OFF,PAUSE</code>.
             <br>Valid for datatypes <code>enum</code>
           </li>
           <li>
             <code>rgb</code> to provide colors in RGB format e.g. <code>255,255,0</code> for yellow.
             <code>hsv</code> to provide colors in HSV format e.g. <code>60,100,100</code> for yellow.
             <br>Valid for datatype <code>color</code>
           </li>
         </ul>
       </td>
       <td>Yes</td>
       <td>No for $datatype <code>string</code>,<code>integer</code>,<code>float</code>,<code>boolean</code>. Yes for <code>enum</code>,<code>color</code></td>
    </tr>
</table>

For example, our `temperature` property would send:

```java
/fastybird/9222852d-1fc4-447b-9967-f8d82fbc4a6f/thermostat/temperature/$name → "Engine temperature"
/fastybird/9222852d-1fc4-447b-9967-f8d82fbc4a6f/thermostat/temperature/$settable → "false"
/fastybird/9222852d-1fc4-447b-9967-f8d82fbc4a6f/thermostat/temperature/$unit → "°C"
/fastybird/9222852d-1fc4-447b-9967-f8d82fbc4a6f/thermostat/temperature/$datatype → "float"
/fastybird/9222852d-1fc4-447b-9967-f8d82fbc4a6f/thermostat/temperature/$format → "-20:120"
/fastybird/9222852d-1fc4-447b-9967-f8d82fbc4a6f/thermostat/temperature → "21.5"
```

* / `fastybird` / `thing ID` / `channel ID` / `property ID` / **`set`**: the thing can subscribe to this topic if the property is **settable** from the controller, in case of actuators.

FastyBird IoT is state-based.
You don't tell your smartlight to `turn on`, but you tell it to put its `power` state to `on`.
This especially fits well with MQTT, because of retained message.

For example, a `kitchen-light` thing exposing a `light` channel would subscribe to `/fastybird/9222852d-1fc4-447b-9967-f8d82fbc4a6f/light/power/set` and it would receive:

```java
/fastybird/9222852d-1fc4-447b-9967-f8d82fbc4a6f/light/power/set ← "true"
```

The thing would then turn on the light, and update its `power` state.
This provides pessimistic feedback, which is important for home automation.

```java
/fastybird/9222852d-1fc4-447b-9967-f8d82fbc4a6f/light/power → "true"
```

### Arrays

A channel can be an array if you've added `[]` to its ID in the `$channels` thing attribute, and if its `$array` attribute is set to the range of the array.
Let's consider we want to control independently the relays of our `smart switch`. Our `switches` channel array would look like this. Note that the topic for an element of the array channel is the name of the channel followed by a `_` and the index getting updated:

```java
/fastybird/9222852d-1fc4-447b-9967-f8d82fbc4a6f/$channels → "switches[]"

/fastybird/9222852d-1fc4-447b-9967-f8d82fbc4a6f/switches/$name → "Relay switches"
/fastybird/9222852d-1fc4-447b-9967-f8d82fbc4a6f/switches/$properties → "state"
/fastybird/9222852d-1fc4-447b-9967-f8d82fbc4a6f/switches/$array → "0-1"

/fastybird/9222852d-1fc4-447b-9967-f8d82fbc4a6f/switches/intensity/$name → "Relay state"
/fastybird/9222852d-1fc4-447b-9967-f8d82fbc4a6f/switches/intensity/$settable → "true"
/fastybird/9222852d-1fc4-447b-9967-f8d82fbc4a6f/switches/intensity/$unit → ""
/fastybird/9222852d-1fc4-447b-9967-f8d82fbc4a6f/switches/intensity/$datatype → "boolean"
/fastybird/9222852d-1fc4-447b-9967-f8d82fbc4a6f/switches/intensity/$format → ""

/fastybird/9222852d-1fc4-447b-9967-f8d82fbc4a6f/switches_0/$name → "Main lights"
/fastybird/9222852d-1fc4-447b-9967-f8d82fbc4a6f/switches_0/state → "true"
/fastybird/9222852d-1fc4-447b-9967-f8d82fbc4a6f/switches_1/$name → "Table lights"
/fastybird/9222852d-1fc4-447b-9967-f8d82fbc4a6f/switches_1/state → "false"
```

Note that you can name each element in your array individually ("Main lights", etc.).

----

### Broadcast Channel

FastyBird IoT defines a broadcast channel, so a controller is able to broadcast a message to every FastyBird things:

* / `fastybird` / `$broadcast` / **`level`**: `level` is an arbitrary broadcast identifier.
It must adhere to the [ID format](#topic-ids).

For example, you might want to broadcast an `alert` event with the alert reason as the payload.
Things are then free to react or not.
In our case, every buzzer of your home automation system would start buzzing.

```java
/fastybird/$broadcast/alert ← "Intruder detected"
```
