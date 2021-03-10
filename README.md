# smpp-terminal-test-harness

A simple Terminal UI application for sending and receiving SMPP messages

## Overview

This repository contains a golang application, which provides a simplistic ESME
(client) and SMSC (server) implementation, as well as a method for launching and
controlling multiple ESME and/or SMSC instances.  Furthermore, it adds a terminal-based
UI for interacting with the launched ESME and SMSC applications.

## Use

In order to build this application:

```bash
mkdir ~/git
cd ~/git
git clone https://github.com/blorticus/smpp-terminal-test-harness.git
cd smpp-teerminal-test-harness
go get -u github.com/blorticus/smpp github.com/blorticus/tpcli github.com/blorticus/smppth
go build -o smpp-terminal-test-harness .
```

To run the executable:

```bash
./smpp-terminal-test-harness run esmes|smscs /path/to/config.yaml
```

An instance of `smpp-terminal-test-harness` runs a set of ESME Agents or a set of SMSC Agents.
The Agents are described in the `config.yaml` file, and if the harness is running
ESME Agents, the `config.yaml` also describes the peers to which each ESME will bind.
The test harness only performs transceiver binds.  The format of `config.yaml` is
described below.

The test harness uses a terminal-based UI.  It has three boxes:
- The *Command History* box at the top;
- A *Command Entry* box in the middle; and
- An *Event Output* box at the bottom.

The Agents will emit events, as described above (on receipt of PDU, on sending PDU,
and on bind completion).  Descriptors for those events -- and any errors -- will 
be delivered to the *Event Output* box.  Commands can be admistered to the
harness by entering them in the *Command Entry* box.  Currently, two commands are
supported:
- `send` - Instructs an Agent to send a PDU to a peer;
- `help` - Prints help text to the *Event Output* box.

The `send` command take parameters, as follows:
```bash
<agent>: send enquire-link to <peer>
<agent>: send submit-sm to <peer> [<params...>]
    where <params> are any combination of:
      source_addr_ton=<ton_int>
      source_addr=<src_addr>
      dest_addr_ton=<ton_int>
      dest_addr=<dest_addr>
      short_message="<short message>"
```

The first will send an enquire-link to from an Agent to a peer.  The second
will send a submit-sm message from an Agent to a peer.  For any parameter not
specific, a default value will be used.

Each Agent will automatically respond to an incoming enquire-link with an
enquire-link-resp, and a submit-sm with a submit-sm-resp.  In both cases,
the sequence number of the request will be used in the response.  In the case
of the submit-sm-resp, the `message_id` field will be set to the name of
the responding Agent.

The command box accepts bash-like emacs control keys, including:
- ^a : move to the start of the line
- ^e : move to the end of the line
- ^d : reverse delete
- ^k : remove all text from cursor through the end of the line

It also support up- and down-cursor readline to move through the command
history.  When a command is entered -- whether it is valid or not -- it
is printed in the *Command History* box.

To exit, use ^c, ^q or &lt;esc&gt;.

## Config Yaml File

The following is an example `config.yaml` file:

```yaml
SMSCs:
  - Name: smsc01
    IP: 127.0.0.1
    Port: 2775
    BindPassword: passwd1
    BindSystemID: smsc01
  - Name: smsc02
    IP: 127.0.0.1
    Port: 2776
    BindPassword: passwd2
    BindSystemID: smsc02
ESMEs:
  - Name: esme01
    IP: 127.0.0.1
    Port: 20775
    BindSystemID: esme01
    BindSystemType: rcs
  - Name: esme02
    IP: 127.0.0.1
    BindSystemID: esme02
    BindSystemType: rcs
TransceiverBinds:
  - ESME: esme01
    SMSC: smsc01
  - ESME: esme01
    SMSC: smsc02
  - ESME: esme02
    SMSC: smsc01
  - ESME: esme02
    SMSC: smsc02
```

When the test harness is run `as smscs`, only the **SMSCSs** section is used.  
When it is run `as emses`, all three sections are used.  For **SMSCs**, both the
*IP* and *Port* are required.  For **ESMEs**, both are optional.  If *Port* is
omitted, then an ephemeral port will be chosen.  The *TransceiverBinds* section
is used to describe to which peers an ESMEs should connect.
