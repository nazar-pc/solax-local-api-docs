# Solax local API docs

Reverse-engineered documentation for local API on inverters built by Solax Power.

Manufacturer didn't want to provide documentation (and it would be mess, I can see why), but we (users) want to have
cloud-free access to real-time data using existing APIs (for instance, to use in Home Assistant) and thus need a way to
interpret the data anyway.

The route I took was to extract information from official Android application (version 0.4.3), which turns out is built
with Apache Cordova and contains all the necessary logic in minified JavaScript.

This approach allows us to not guess what values mean what and to support all inverters that official app supports.

The information contained in this repository is the result of un-tangling that minified JavaScript and transforming into
human-readable pseudo-code.

In order to get the data you'll need to make request to your inverter:
```bash
curl -d "?optType=ReadRealTimeData&pwd=PASSWORD" -X POST http://IP_ADDRESS
```

* `PASSWORD` is the password used for local access to the data in the app, it defaults to the serial number of the Pocket
Wi-Fi (I don't have Pocket Lan, but it likely works the same way).
* `IP_ADDRESS` is the IP address of the inverter given by your router, something like `192.168.1.105`

Typical response from API looks something like this (X1-Hybrid G4):
```json
{
  "sn": "SV********",
  "ver": "3.003.02",
  "type": 15,
  "Data": [ A LOT OF NUMBERS HERE ],
  "Information": [ 5.000, 15, "H4************", 8, 1.30, 0.00, 1.28, 1.04, 0.00, 1]
}
```

# Known inverter types

Inverter type is both contained in `type` field of the response and, apparently, in `Information` field as well.
At least this is true for most inverters.

These are the known inverter types based on https://github.com/squishykid/solax, please add yours in PR:
* `8`
  * Solax Power X1-Smart
* `14`
  * Solax Power X3-Hybrid G4
  * Qcells Q.VOLT HYB-G3-3P
* `15`
  * Solax Power X1-Hybrid G4

Here is how they are named in the application's source code:

| Name            | inverterType |
|-----------------|--------------|
| x1hybridg3      | 3            |
| x1boostairmini  | 4            |
| x3hybridg1      | 5            |
| x320k30k        | 6            |
| x3mic           | 7            |
| x1boostpro      | 8            |
| x1ac            | 9            |
| a1hybrid        | 10,11,12     |
| j1ess           | 13           |
| x3hybridg4      | 14           |
| x1hybridg4      | 15           |
| x3micprog2      | 16           |
| x1hybridsplitg4 | 17           |
| x1boostminig4   | 18,22        |
| a1hybridg2      | 19,20,21     |
| x1hybridg5      | 23           |
| x3hybridg5      | 24           |
| x3big           | 100,101      |
| x1hybridlv      | 102          |

## Contents of this repository

There are a few key files listed below that contain mapping and pseudocode explaining usage or parsing algorithm.
Only information in the source code of the app was used to derive mappings, there might be other meaningful fields that
are not present in the app.

There might be bugs, for instance I didn't include main battery serial number parsing because it returns garbage on my
inverter and I can't guarantee that it makes sense on any model (application doesn't show it despite containing code
that parses it).

There are no promises whatsoever, so use this at your own risk.

Application also, naturally, contains code to control the inverter, but due to risks involved with changing those
parameters, they are not (yet?) documented here. It would have been nice to automate things with Home Assistant though.

### Information.txt

This file helps to get some high-level information about inverter and understand how to interpret the rest of the data
based on `Information` field of the response.

By knowing which inverter type it is, you can look into `Data1.txt` or `Data2.txt` for further details.

### Data1.txt

Seems to be older version of the API for some earlier inverter models, has generic mapping for all matching models and
should be assumed to have many of the fields as optional since I was not able to deduce clear mapping between different
models.

### Data2.txt

Seems to be modern version of the API, has clear mapping between inverter type and decoding rules.
The same parameter doesn't always occupy the same offset in `Data` field, hence more boilerplate would be necessary to
parse it.

### DataEvCharger.txt

Serial number can apparently be in either `sn` or `SN` fields of the response.

Serial numbers starting with "SC" and "SQ" appear to be related to EV chargers.

`DataEvCharger.txt` contains mapping to decode real-time data from such EV chargers.

## License

Public Domain
