# Device firmware description

* **Version:** 0.1.0
* **Date:** 01/19
* **Author:** FastyBird dev

This extension add support for device firmware description

----

## Extension identifier

Reserved ID for this extension is `com.fastybird.firmware`

## Extension datatypes

This extension defines no new datatypes.

## Extension attributes

This extension defines nested **required** attribute:

<table style="display: table">
    <tr>
        <th align="left">Topic</th>
        <th align="left">Direction</th>
        <th align="left">Description</th>
        <th align="center">Payload</th>
        <th align="center">Retained</th>
        <th align="center">Required</th>
    </tr>
    <tr>
        <td align="left">$fw / name</td>
        <td align="left">Device → Controller</td>
        <td align="left">
            Friendly name of the device's firmware
        </td>
        <td align="center">String</td>
        <td align="center">Yes</td>
        <td align="center">Yes</td>
    </tr>
    <tr>
        <td align="left">$fw / manufacturer</td>
        <td align="left">Device → Controller</td>
        <td align="left">
            Firmware manufacturer name
        </td>
        <td align="center">String</td>
        <td align="center">Yes</td>
        <td align="center">Yes</td>
    </tr>
    <tr>
        <td align="left">$fw / version</td>
        <td align="left">Device → Controller</td>
        <td align="left">
            Firmware released version
        </td>
        <td align="center">String</td>
        <td align="center">Yes</td>
        <td align="center">Yes</td>
    </tr>
</table>

#### Example

```
/fb/v1/room-smart-switch/$fw/name → "smart-switch-rev.1"
/fb/v1/room-smart-switch/$fw/manufacturer → "Firmware creator"
/fb/v1/room-smart-switch/$fw/version → "1.0.0"
```
