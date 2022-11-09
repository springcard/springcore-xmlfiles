---
title: Syntax of XML files for configuration
category: readme
author: johann.d
tags: [XML, SpringCore, metadata]
---

# Syntax of XML files for configuration

## Overview

Every device has its own configuration file.

The name of the configuration file is `<product-slug>.xml` where `product-slug` is the internal value used by the firmware team to uniquely identify the device.

The root of the XML tree is `<springcard-multiconf-registers>`

In the XML tree,

- The root object's attribute fields provide all data required to identify the device, and to describe it to the end-user
- Child nodes expose all the configuration registers that the device features, and detail the underlying fields, and how they shall be described to the end-user.

### Individual registers and their implication to the UI

File `springcard-multiconf-individuals.xml` contains a list of devices, and, for every device, the list of individual registers (using their address or `id` as key).

From the device point of view, individual registers are a part of their configuration. The firmware makes no difference with other registers. But from the end-user point of view, hence for the configuration application, they are two different things.

- **The global configuration** (i.e. all the registers that are not explicitly listed as being individual) **could be copied 'as is' into many devices**. Typically, all devices of the same type belonging to the same company or installed to the same premises shares the same global configuration,
- **The individual configuration** (i.e. the set of a few registers that are explicitly listed as being individual) **is specific to one very device**. Copying this configuration to another device would not work, or break something. This is for instance the case of the static IP address (the device's address shall be unique on the network) or the inventory and location data used for asset management.

The application shall expose the global configuration as a set of parameters that could be applied to many device, and shall provide a mean to edit the individual configuration of every device (with no doubt on the very target it is actually configuring).

## The XML root node

The root of the XML tree is `<springcard-multiconf-registers>`

The attribute fields of the root object are explained in the list below.

Since the root node describes a device, its content is closely related to the DEVICE object that the **SpringCard Companion Service** exposes for an instance of the device that is actually connected to the computer. For reference, see [docs.springcard.com/books/Companion/REST_API/Devices/Objects/DEVICE](https://docs.springcard.com/books/Companion/REST_API/Devices/Objects/DEVICE/index).

### Attributes

| Name           | Description                                                  | Corresponding DEVICE field in Companion                  |
| -------------- | ------------------------------------------------------------ | -------------------------------------------------------- |
| `title`        | Common name of the device                                    | `Name` or `FriendlyName` depending on the OS             |
| `path`         | Family / Product / Variant                                   | *None*                                                   |
| `firmware`     | Unique identifier of the firmware                            | `Firmware`                                               |
| `implements`   | Features that the device has; see below                      | Closely related to `Characteristics`, but not equivalent |
| `hardware`     | Unique identifier of the hardware                            | `Hardware`                                               |
| `icon`         | Product's icon                                               |                                                          |
| `version`      | Version of the firmware when the XML file has been generated | `Version`                                                |
| `author`       | Person who has approved the configuration                    | *None*                                                   |
| `templatesnfc` | Is the device a NFC SmartReader?                             |                                                          |
| `templatesble` | Is the device a NFC SmartReader?                             |                                                          |

### Selecting the right XML file

Given a DEVICE object provided by **SpringCard Companion Service**, the configuration application shall filter the XML files to keep only the one that match on the `firmware`/`Firmware` and `hardware`/`Hardware` fields.

In some situation, the `Hardware` field may be empty in the DEVICE object. In this case, the match is done on the `Firmware` field only. If more than one XML file match, the user shall be prompted to select the right one.

In other situation, an XML file exists with the `hardware` field empty or set to `*` (wildcard). Such XML file is the fallback for all devices with the same `Firmware` but no corresponding `Hardware`, or an empty `Hardware`.

### Dealing with the version fields

The `version` field in the XML file tells for which version of firmware the XML file has been generated, where the `Version` field in the DEVICE object tells the version of firmware the device is actually running.

* ***If both versions are equals*** everything is fine, don't bother the user,
* ***If version in XML is lower than version in device*** user shall be notified: *"This device is running a new version of the firmware that is not supported by this configuration application. Some features may not work as expected until a new version of the configuration application is released."*.
* ***If version in XML is greater than version in device*** user shall be notified and invited to flash the device: *"This device is running an out-dated version of the firmware. Some features exposed by the configuration application may not be supported by the device. For correct operation, please upgrade the device's firmware to latest version before proceeding."*.

### Values for the `implements` attribute

This field is interesting to improve the UI of the configuration application: icons may be shown to describe the device, and/or only the communication interfaces supported by the device may be offered when trying to connect to it.

The features are:

| Slug          | Description                                         | Ideas to improve the UI                                      |
| ------------- | --------------------------------------------------- | ------------------------------------------------------------ |
| `usb`         | The device has a USB interface                      | USB icon, "plug the device on" prompt instead of "connect the device", etc |
| `ble`         | The device has a Bluetooth interface                | Bluetooth icon, "looking for Bluetooth device" prompt, etc   |
| `ccid`        | The device supports PC/SC operation                 | PC/SC icon, PC/SC-related tests and samples                  |
| `rdr`         | The device supports Smart Reader operation          | RDR icon, template screens available                         |
| `micore`      | The device has a NFC interface                      | NFC icon                                                     |
| `7816`        | The device supports ISO 7816                        | *(nothing special)*                                          |
| `7816_id1`    | The device has a ID-1 smart card interface          | smart card icon                                              |
| `7816_sam`    | The device has a ID-000 smart card interface        | SIM/SAM card icon                                            |
| `7816_sam_x4` | The device has up to 4 ID-000 smart card interfaces | 4 x SIM/SAM card icon                                        |
| `samav`       | The device has a NXP SAM AV Secure Element          | Vault icon, Desfire icon                                     |
| `atecc`       | The device has a ATECC Secure Element               | Vault icon, Apple VAS icon, Google Smart Tap icon            |
| `vegas`       | The device has a Vegas UI for LEDs & buzzer         | *(nothing special)*                                          |
| `battery`     | The device has a battery                            | Battery icon, showing battery level                          |

### Example

```xml
<?xml version="1.0" encoding="utf-8"?>
<springcard-multiconf-registers
	title="Puck Ultimate"
	path="SpringCore/Puck/Puck Ultimate"
	firmware="springcore/h518/puck"
	implements="usb,ble,ccid,rdr,micore,7816,7816_sam,samav,atecc,vegas,battery"
	hardware="FPF18170"
	icon="product-puck"
	version="1.23"
	author="JDA"
	templatesnfc="true"
	templatesble="true"
>
    <tree>
        (...)
```

## `modes` and `mode` XML structure

Under the XML root node, an <u>optional</u> `<modes>` node contains a collection of `<mode>`.

The goal is to (possibly) personalize the product's icon based on the operating mode.

### Attributes for the `mode` XML node

| Name    | Description                                                  |
| ------- | ------------------------------------------------------------ |
| `value` | Value of the mode (matching the `Mode` field in Companion's DEVICE object) |
| `icon`  | Badge to add on top of the product's icon if the product is currently operating in this mode |

## `channels` and `channel` XML structure

Under the XML root node, an <u>optional</u> `<channels>` node contains a collection of `<channel>`.

The goal is to (possibly) personalize the product's icon based on the communication channel.

### Attributes for the `channel` XML node

| Name    | Description                                                  |
| ------- | ------------------------------------------------------------ |
| `value` | Value of the channel (matching the `Channel` field in Companion's DEVICE object) |
| `icon`  | Badge to add on top of the product's icon if the product is currently using this channel |

## `tree` and `node` XML structure

Under the XML root node, a `<tree>` node contains a collection of `<node>`.

Every `<node>` is a section of the configuration screen. It contains a collection of `<register>`.

### Attributes for the `node` XML node

| Name    | Description          |
| ------- | -------------------- |
| `title` | Title of the section |

## `register` XML nodes

Every `<register>` node maps to a single register address in the device's configuration memory.

It contains a mixed collection of

1. UI-related entries: `<title>`, `<description`>, `<remark>`, ...
2. `<field>`. A field is a portion of the register (i.e. a subset of it bits) that is shown to the user as a single, atomic, configuration field.

### Attributes for the `register` XML node

| Name      | Description                                                  |
| --------- | ------------------------------------------------------------ |
| `id`      | Address of the register, on two bytes, specified in hexadecimal (allowed values are `0200` to `02FF`) |
| `size`    | Size of the register, in bytes, specified in decimal (allowed values are `1` to `64`) |
| `default` | Default value of the register, specified in hexadecimal. This is typically used by the application to populate the fields in the UI with default state when a new, blank configuration is created. |

### `title` XML entry

Under a `<register>` XML node, a `<title>` XML entry contains the title of the register.

### `description` XML entry

Under a `<register>` XML node, a `<description>` XML entry contains the description of the register.

### `remark` XML entry

Under a `<register>` XML node, a `<remark>` XML entry contains any remark to explain or detail the register.

## `field` XML nodes

Under a `<register>` XML node, a `<field>` XML node details one piece of configuration that shall be exposed to the user as a single field. Type of field (checkbox, radio, free entry...) is specified in the attributes.

The `<field>` XML node may be final, or may contain a collection of `<option>`.

### Attributes for the `field` XML node

| Name        | Description                                   | Remark                                                       |
| ----------- | --------------------------------------------- | ------------------------------------------------------------ |
| `type`      | How shall the field be displayed to the user? | See below for a list                                         |
| `id`        | Unique ID of the field                        | Not relevant for the Companion Service. Can be used by the configuration application for convenience. |
| `title`     | Title of the field                            | Main label to be associated to the field's input area        |
| `name`      | Name of the field                             | Shorter, hence less explicit than the Title. May be used to present the fields as a tree in the UI. |
| `bitmap`    | Relevant bits in the register                 | Value is specified in hexadecimal. This is a bit mask (i.e. valid bits are `1`, unused bits are `0`). If the register has only one field, the bitmap may be missing (it must be assumed that it is all `1`). |
| `value`     | Value of the (hidden) field                   | Only present for a field having `type="hidden"`. |
| `visibleif` | Condition to display this field               | See `visibleif` paragraph at the end of the document         |

### Values for `type` attribute

The `type` attribute selects the way the field shall be displayed. Allowed values are:

| Type value     | Description                                                  |
| -------------- | ------------------------------------------------------------ |
| `hidden`       | The field is not visible                                     |
| `raw`          | The field is a raw input, expecting an hexadecimal value     |
| `string`       | The field is a text input                                    |
| `number-d`     | The field is a numeric input, user shall enter a decimal value |
| `number-h`     | The field is a numeric input, user shall enter an hexadecimal value |
| `number`       | The field is a numeric input, user may select either decimal or hexadecimal |
| `checkbox`     | The field is a single check box (checked / not checked)      |
| `checkboxes`   | The field is a list of check boxes                           |
| `radios`       | The field is a list of radio boxes (only one can be checked) |
| `buttons`      | The field is a list of buttons (only one can be selected)    |
| `dropdownlist` | The field is a list of values, as a drop down list           |
| `ip`           | The field is an IPv4 address                                 |
| `ipv6`         | The field is an IPv6 address                                 |

## `option` XML nodes

Under a `<field>` XML node with a `type` attribute equal to `checkboxes`, `radios` or `dropdownlist`, every possibility is added as an `<option>` XML node.

### Attributes for the `option` XML node

| Name        | Description                         | Remark                                                       |
| ----------- | ----------------------------------- | ------------------------------------------------------------ |
| `value`     | The value of the option             | Value is expressed in decimal                                |
| `title`     | Title of the option                 | Main label to be associated to the value                     |
| `icon`      | Icon of the button                  | Only for `buttons`-type fields                               |
| `advanced`  | Is the option reserved for experts? | If true, the option is hidden by default, until the user enables advanced features |
| `visibleif` | Condition to display this option    | See `visibleif` paragraph at the end of the document         |

## `visibleif` attributes

A `visibleif` attribute may be present at the `<field>` and `<option>` XML nodes. The goal is to hide fields and options that are not applicable when other configuration parameters are selected (for instance the fields or options related to PC/SC mode when Smart Reader mode is selected).

The syntax of `visibleif` is a very basic script language.

The script langage has 3 functions:

- **REGVAL(address):** read the current value in the register at ***address***,
- **REGBIT(address, bit_offset):** read the current value of the bit at position ***bit_offset*** in register at ***address***,
- **REGBITS(address, bit_offset, bit_count):** read the current value of the ***bit_count*** bits starting at position ***bit_offset*** in register at ***address***,

**Remark:** *REGBIT(address, offset)* is the same as *REGBITS(address, offset, 1)*

The script langage has 4 operators:

* **is**: test the equality (right value shall be in decimal)
* **is not**: test the inequality (right value shall be in decimal)
* **and**: logical AND
* **or**: logical OR

The reference parser comes from https://github.com/Bunny83/LSystem/blob/master/LogicExpressionParser.cs

