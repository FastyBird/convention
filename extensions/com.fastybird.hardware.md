# Device hardware description

* **Version:** 0.1.0
* **Date:** 01/19
* **Author:** FastyBird dev

This extension add support for device hardware description

----

## Extension identifier

Reserved ID for this extension is `com.fastybird.hardware`

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
        <td align="left">$hw / mac-address</td>
        <td align="left">Device → Controller</td>
        <td align="left">
            Device physical address identifier
        </td>
        <td align="center">String</td>
        <td align="center">Yes</td>
        <td align="center">Yes</td>
    </tr>
    <tr>
        <td align="left">$hw / manufacturer</td>
        <td align="left">Device → Controller</td>
        <td align="left">
            Hardware manufacturer name
        </td>
        <td align="center">String</td>
        <td align="center">Yes</td>
        <td align="center">Yes</td>
    </tr>
    <tr>
        <td align="left">$hw / model</td>
        <td align="left">Device → Controller</td>
        <td align="left">
            Hardware model name
        </td>
        <td align="center">String</td>
        <td align="center">Yes</td>
        <td align="center">Yes</td>
    </tr>
    <tr>
        <td align="left">$hw / version</td>
        <td align="left">Device → Controller</td>
        <td align="left">
            Hardware released version
        </td>
        <td align="center">String</td>
        <td align="center">Yes</td>
        <td align="center">Yes</td>
    </tr>
</table>

#### Example

```
/fb/v1/room-smart-switch/$hw/mac-address → "00:0a:95:9d:68:16"
/fb/v1/room-smart-switch/$hw/manufacturer → "Hardware creator"
/fb/v1/room-smart-switch/$hw/model → "awesome light"
/fb/v1/room-smart-switch/$hw/version → "rev.A"
```
