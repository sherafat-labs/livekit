# Client Setup & Configuration Guidelines

This directory contains reference client configurations and guide for setting up and connecting LiveKit clients to the LiveKit server.

## Overview

When clients (browsers, mobile applications, SDKs, or AI agents) connect to LiveKit server, they negotiate WebRTC parameters and client settings. LiveKit server provides a rule-matching engine to dynamically or statically apply client configurations based on client characteristics (browser type, browser version, operating system, device model, SDK version, and protocol version).

## Connection Setup

Clients connect to LiveKit server via WebSocket signaling endpoint:
`ws://<server-host>:7880` (or `wss://` with TLS in production).

### Authentication

Clients authenticate using a JSON Web Token (JWT) signed with the server's API key and secret pair.
Tokens encode participant identity, room permissions (join, publish, subscribe, etc.), and token expiration.

You can generate client access tokens using the LiveKit CLI (`lk`):

```bash
lk token create \
    --api-key devkey --api-secret secret \
    --join --room my-room --identity user1 \
    --valid-for 24h
```

## Client Configurations

Client configuration rules allow server operators to selectively customize codec capabilities, hardware encoder settings, and connection behaviors for specific client environments.

### Matching Fields

Rule match expressions use Tengo scripts and evaluate properties of the client object `c`:

| Attribute | Description | Example |
| --- | --- | --- |
| `c.sdk` | SDK name (lowercase) | `"js"`, `"android"`, `"ios"` |
| `c.version` | SDK version string | `"2.17.1"` |
| `c.protocol` | Protocol version integer | `6` |
| `c.os` | Operating system (lowercase) | `"android"`, `"ios"`, `"linux"`, `"windows"`, `"macos"` |
| `c.os_version` | OS version string | `"14.0"` |
| `c.device_model` | Device model name | `"pixel 7"`, `"xiaomi 2201117ti"` |
| `c.browser` | Browser name (lowercase) | `"chrome"`, `"safari"`, `"firefox"`, `"firefox mobile"` |
| `c.browser_version` | Browser version string | `"17.4"` |
| `c.address` | Client IP address | `"192.168.1.5"` |

### Rule Examples

Sample rule definitions can be found in `config-sample.yaml`.

#### Example 1: Disabling AV1 on Safari

```yaml
- match: 'c.browser == "safari"'
  configuration:
    disabled_codecs:
      codecs:
        - mime: "video/av1"
  merge: true
```

#### Example 2: Disabling H.264 publishing on specific Android devices or mobile Firefox

```yaml
- match: '(c.device_model == "xiaomi 2201117ti" && c.os == "android") || ((c.browser == "firefox" || c.browser == "firefox mobile") && c.os == "android")'
  configuration:
    disabled_codecs:
      publish:
        - mime: "video/h264"
  merge: false
```

## Client SDK References

For platform-specific setup instructions and SDK guides, refer to official LiveKit SDK repositories:

- **Web / JS / TS**: [client-sdk-js](https://github.com/livekit/client-sdk-js)
- **Swift / iOS / macOS**: [client-sdk-swift](https://github.com/livekit/client-sdk-swift)
- **Android / Kotlin**: [client-sdk-android](https://github.com/livekit/client-sdk-android)
- **Flutter**: [client-sdk-flutter](https://github.com/livekit/client-sdk-flutter)
- **React Native**: [client-sdk-react-native](https://github.com/livekit/client-sdk-react-native)
- **Rust**: [rust-sdks](https://github.com/livekit/rust-sdks)
- **Python**: [python-sdks](https://github.com/livekit/python-sdks)
