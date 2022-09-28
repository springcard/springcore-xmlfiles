# Syntax of XML files for Templates

## Overview

There are two files for the Templates

- `springcore-nfc-templates.xml` defines the templates related to the NFC/RFID HF interface,
- `springcore-ble-templates.xml` defines the templates related to the BLE (Bluetooth Low Energy) interface.

NFC templates are enabled when the `templatenfc="true"` attribute is set in the device's XML configuration file, and BLE templates are enabled when the `templateble="true"` attribute is set in the device's XML configuration file.

The root of the XML tree is `<springcard-multiconf-templates>`

In the XML tree, every template is exposed as a set of registers (1 to 16 registers). Childr objects list the underlying fields, and how they shall be described to the end-user.

### Understanding how the Template engine works inside the device

#### NFC Templates

The device supports 4 NFC Template instances.

- Template instance #1 stores its configuration in registers `0310` to `031F`,
- Template instance #2 stores its configuration in registers `0320` to `032F`,
- Template instance #3 stores its configuration in registers `0330` to `033F`,
- Template instance #4 stores its configuration in registers `0340` to `034F`.

The first register of any given Template instance (`0310`, `0320`, `0330` and `0340`) is called the **LKL** register. **LKL** stands for *Lookup List*. This register has two roles at the same time:

- It tells the reader what is the Template **type** (e.g. method): read only the protocol level-ID, or access a memory block, or a file...
- It tells the reader which NFC/RFID HF **technology(ies)** it shall look for on the RF interface.

If the **LKL** register is empty or `00`, the Template instance is disabled.

To configure the reader, the user shall

1. Confirm he wants to use Template instance #1 (otherwise all registers from `0310` to `013F` are kept blank),
2. Select the **type** of Template instance #1,
3. Select the **technology(ies)** it shall look for on the RF interface,
4. Point 2 and 3 define the value for LKL register (going to register `0310`),
5. Edit the configuration of the Template instance (going to registers `0311` to `031F`),
6. Do the same for Template instance #2, Template instance #3 and Template instance #4.

#### BLE Templates

*To be written*

### Understanding how the registers are specified in the XML file

In the XML file, only the offset of every register is specified in the `id` attribute.

For instance, a register with `id="0003"` will go to register `0313` if it is used in Template instance #1, to register `0323` if it is used in Template instance #2, and so on. Generally speaking, on Template instance #*t*, register with offset `i` is stored in address `03ti`.

## The XML root node

The root of the XML tree is `<springcard-nfc-templates>`

The attribute fields of the root object are explained in the list below.

### Attributes

| Name      | Description                                                  |
| --------- | ------------------------------------------------------------ |
| `version` | Version of the firmware when the XML file has been generated |
| `author`  | Person who has approved the configuration                    |

## `template` XML nodes

Under the XML root node, a `<tree>` node contains a collection of `<template>`.

Every `<template>` is a possible Template type. It contains a mixed collection of

1.  `<lkl>` entries, one entry per technology that the Template supports,
2. UI-related entries: `<title>`, `<description`>, `<remark>`, ...
3. `<register>` entries. Every register maps to an offset under the addressing space of the Template instance.

### Attributes fot the `template` XML node

| Name    | Description |
| ------- | ----------- |
| `name`  | Short name  |
| `title` | Long name   |

### `description` XML entry

Under a `<template>` XML node, a `<description>` XML entry contains the description of the Template.

### `remark` XML entry

Under a `<template>` XML node, a `<remark>` XML entry contains any remark to explain or detail the Template.

## `register` XML nodes

Every `<register>` node maps to a single register address in the device's configuration memory.

It contains a mixed collection of

1. UI-related entries: `<title>`, `<description`>, `<remark>`, ...
2. `<field>`. A field is a portion of the register (i.e. a subset of it bits) that is shown to the user as a single, atomic, configuration field.

### Attributes for the `register` XML node

| Name      | Description                                                  |
| --------- | ------------------------------------------------------------ |
| `id`      | Offset of the register under the address space of the Template instance, on two bytes, specified in hexadecimal (allowed values are `0001` to `000F`. The LKL register, at offset `0000`, does not appear in the list) |
| `size`    | Size of the register, in bytes, specified in decimal (allowed values are `1` to `64`) |
| `default` | Default value of the register, specified in hexadecimal. This is typically used by the application to populate the fields in the UI with default state when a new, blank Template instance is added. |

### `title` XML entry

Under a `<register>` XML node, a `<title>` XML entry contains the title of the register.

### `description` XML entry

Under a `<register>` XML node, a `<description>` XML entry contains the description of the register.

### `remark` XML entry

Under a `<register>` XML node, a `<remark>` XML entry contains any remark to explain or detail the register.

## `lookups` and `lkl` XML nodes

Under a `<register>` XML node, a ` <lookup>` XML node contains a set  of `<lkl>`  XML nodes.

The `<lkl>`  XML nodes select technology(ies) the Template instance shall look for.

The selected `value` goes straight into the register at offset `0000` under the address space of the Template instance.

### Attributes for the `lkl` XML node

| Name    | Description                                                  |
| ------- | ------------------------------------------------------------ |
| `value` | Value to be written in the **LKL** register of the Template instance. This is a 1-byte value, in hexadecimal. |
| `title` | Description of the selected technology(ies).                 |

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
| `value`     | Value of the (hidden) field                   | Only present for a field having `type="hidden"`              |
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
| `advanced`  | Is the option reserved for experts? | If true, the option is hidden by default, until the user enables advanced features |
| `visibleif` | Condition to display this option    | See `visibleif` paragraph at the end of the document         |

## `visibleif` attributes

A `visibleif` attribute may be present at the `<field>` and `<option>` XML nodes. The goal is to hide fields and options that are not applicable when other configuration parameters are selected (for instance the fields or options related to PC/SC mode when Smart Reader mode is selected).

The syntax of `visibleif` is a very basic script language.

The script langage has 3 functions:

- **TPLVAL(reg_offset):** read the current value in the register at ***reg_offset*** under the current Template instance,
- **TPLBIT(reg_offset, bit_offset):** read the current value of the bit at position ***bit_offset*** in register at ***reg_offset*** under the current Template instance,
- **TPLBITS(reg_offset, bit_offset, bit_count):** read the current value of the ***bit_count*** bits starting at position ***bit_offset*** in register at ***reg_offset*** under the current Template instance.

**Remark:** *TPLBIT(reg_offset, bit_offset)* is the same as *TPLBITS(reg_offset, bit_offset, 1)*

The script langage has 4 operators:

* **is**: test the equality (right value shall be in decimal)
* **is not**: test the inequality (right value shall be in decimal)
* **and**: logical AND
* **or**: logical OR

The reference parser comes from https://github.com/Bunny83/LSystem/blob/master/LogicExpressionParser.cs



