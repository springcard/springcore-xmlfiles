# SpringCard SpringCore XML Files

**SpringCard SpringCore** is the umbrella name for the firmware that powers the new generation of SpringCard devices.

Devices in this family are very versatile, hence have many configuration settings.

The configuration is generally edited using **SpringCard Companion** cloud-based solution ([companion.springcard.com](https://companion.springcard.com)). Alternatively, the **SpringCoreConfig** command-line tool may be used (see [docs.springcard.com/books/Tools/SpringCore](https://docs.springcard.com/books/Tools/SpringCore/index) for details). Documentation of every configuration register is regularly published on [docs.springcard.com/books/SpringCore/Non-volatile_memory](https://docs.springcard.com/books/SpringCore/Non-volatile_memory/index).

**SpringCard Companion** exposes the device's configuration as user-friendly forms in the web browser. These forms are generated from the device's internals. The XML files published in this repository act as the gateway between the source code of the device's firmware and the web application.

This repository is public, and customers/integrators are free to embed the configuration of the devices they use in their own applications.