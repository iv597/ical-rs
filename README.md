
# ical-rs 0.2.1
===============

[![Build Status](https://travis-ci.org/Peltoche/ical-rs.svg?branch=master)](https://travis-ci.org/Peltoche/ical-rs)


This is a library to parse the ICalendar format defined in [RFC5545](http://tools.ietf.org/html/rfc5545), as well asl
similar formats like VCard.

There are probably some issues to be taken care of, but the library should work for most cases. If you like to help out and
would like to discuss any API changes, please [contact me](dev@halium.fr) or create an issue.

The initial goal was to make a port from the [ical.js](https://github.com/mozilla-comm/ical.js) library in JavaScript and
many code/algorithms was taken from it but in order to but more 'Rusty' a complete rewrite have been made.

## [Documentation](https://peltoche.github.io/ical-rs/ical/)

## Installing

Put this in your `Cargo.toml`:

```toml
[dependencies]
ical = "0.2.0"
```


## Overview

There is several ways to use Ical depending on the level of parsing you want. Some new wrapper/formater could appeare in
the next releases.

By default all the features are included but you can choose to include in you project only the needed ones.

#### Warning
  The parsers (LineParser / IcalParser) only parse the content and set to uppercase the case-insensitive fields. No checks
  are made on the fields validity.


### IcalParser / VcardParser

Wrap the result of the LineParser into components.

Each component can contains properties (ie: LineParsed) or sub-components.

* The IcalParser return IcalCalendar
* The VcardParser return VcardContact

Cargo.toml:
```toml
[dependencies.ical]
version = "0.2.*"
default-features = false
features = ["ical-parser", "vcard-parser"]
```

Code:
```rust
extern crate ical;

use std::io::BufReader;
use std::fs::File;

fn main() {
    let buf = BufReader::new(File::open("/tmp/component.ics")
        .unwrap());

    let reader = ical::IcalParser::new(buf);

    for line in reader {
        println!("{:?}", line);
    }
}
```

Output:
```
IcalCalendar {
  properties: [],
  events: [
    IcalEvent {
      properties: [ LineParsed { ... }, ... ],
      alarms: [
        IcalAlarm {
          properties: [ LineParsed { ... } ]
        }
      ]
    }
  ],
  alarms: [],
  todos: [],
  journals: [],
  free_busys: [],
  timezones: []
}
```

### LineParser

Parse the result of LineReader into three parts:

- The name of the line attribute formated in uppercase.
- A vector of `(key/value)` tuple for the parameters. The key is formatted in uppercase and the value is untouched.
- The value stay untouched.

It work for both the Vcard and Ical format.

#### Example:

Cargo.toml:
```toml
[dependencies.ical]
version = "0.2.*"
default-features = false
features = ["line-parser"]
```

Code:
```rust
extern crate ical;

use std::io::BufReader;
use std::fs::File;

fn main() {
    let buf = BufReader::new(File::open("/tmp/component.ics")
        .unwrap());

    let reader = ical::LineParser::from_reader(buf);

    for line in reader {
        println!("{:?}", line);
    }

```

Input -> Output:
```
begin:VCALENDAR                           Ok(LineParsed { name: "BEGIN", params: None, value: "VCALENDAR" })
ATTENDEE;cn=FooBar:mailto:foo3@bar    ->  Ok(LineParsed { name: "ATTENDEE", params: Some([("CN", "FooBar")]), value: "mailto:foo3@bar" })
END:VCALENDAR                             Ok(LineParsed { name: "END", params: None, value: "VCALENDAR" })
```

### LineReader

This is a very low level parser. It clean the empty lines and unfold them.

It work for both the Vcard and Ical format.

#### Example:

Cargo.toml:
```toml
[dependencies.ical]
version = "0.2.*"
default-features = false
features = ["line-reader"]
```

Code:
```rust
extern crate ical;

use std::io::BufReader;
use std::fs::File;

fn main() {
    let buf = BufReader::new(File::open("/tmp/component.ics")
        .unwrap());

    let reader = ical::LineReader::new(buf);

    for line in reader {
        println!("{}", line);
    }
}
```

Input -> Output:

```
BEGIN:VCALENDAR        Line 0: BEGIN:VCALENDAR
BEGIN:VEVENT           Line 1: BEGIN:VEVENT
SUMMARY:foo and   ->   Line 3: SUMMARY:foo andbar
 bar
END:VEVENT             Line 4: END:VEVENT
END:VCALENDAR          Line 5: END:VCALENDAR
```



