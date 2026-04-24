# jtpounce

Watches [WSJT-X](https://wsjt.sourceforge.io/wsjtx.html) for anyone calling
your callsign and immediately tells WSJT-X to reply, then exits.

Useful when you want to leave WSJT-X running and have it automatically respond
the moment a station calls you on FT8 or FT4.

## Requirements

- Python 3.6+
- No third-party packages — stdlib only (`socket`, `struct`, `argparse`)
- WSJT-X with UDP reporting enabled

## WSJT-X configuration

Enable UDP reporting in WSJT-X under **File → Settings → Reporting**:

| Setting | Value |
|---|---|
| UDP Server | `127.0.0.1` (or `224.0.0.1` for multicast) |
| UDP Server port number | `2237` |
| Accept UDP requests | checked |

## Usage

```
./jtpounce YOURCALL [--port PORT] [--multicast]
```

```
positional arguments:
  callsign      Your callsign, e.g. W1ABC

options:
  --port PORT   UDP port WSJT-X sends to (default: 2237)
  --multicast   Join multicast group 224.0.0.1
                (use when WSJT-X is configured to send to that group)
```

### Examples

```bash
# Unicast (default WSJT-X config)
./jtpounce W1ABC

# Custom port
./jtpounce W1ABC --port 2237

# Multicast
./jtpounce W1ABC --multicast
```

### Example output

```
Monitoring for calls to W1ABC on UDP port 2237
  [ ~]  CQ W2XYZ FN20
  [ ~]  K9ABC W1ABC -07

  *** Heard: K9ABC W1ABC -07
  *** Sending reply to WSJT-X at 127.0.0.1:52341 ***
```

WSJT-X will queue an outgoing reply to that station and transmit it on the
next available TX slot.

## How it works

WSJT-X broadcasts decoded messages over UDP as binary datagrams using Qt's
`QDataStream` format.  `jtpounce` binds to the configured port, parses every
**Decode** message (type 2), and checks whether the second token in the
decoded text matches your callsign — the standard FT8/FT4 directed format is
`THEIRCALL OURCALL REPORT`.

When a match is found it constructs a **Reply** message (type 4) containing
the full decode payload and sends it back to WSJT-X, which interprets it as
"reply to this station."  The program then exits immediately.

## License

MIT
