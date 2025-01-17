# AT Driver

A [WebSocket](https://developer.mozilla.org/en-US/docs/Web/API/WebSocket)
server which exposes the speech being vocalized by various screen readers
running locally on a Microsoft Windows system.

## Requirements

- Microsoft Windows
- [Node.js](https://nodejs.org), including "Tools for Native Modules" as
  offered by the Node.js installer

<!--
  "Tools for Native Modules" is required to install the "robotjs" npm module,
  which is a dependency of this project.
-->

## Installation

1. Install the project by executing the following command:

       npm install -g @bocoup/windows-sapi-tts-engine-for-automation

   If prompted for system administration permission, grant permission.

2. Start the server by executing the following command in a terminal:

       at-driver

   The process will write a message to the standard error stream when the
   WebSocket server is listening for connections. The `--help` flag will cause
   the command to output advanced usage instructions (e.g. `at-driver --help`).

3. Configure any screen reader to use the synthesizer named "Microsoft Speech
   API version 5" and the text-to-speech voice named "Bocoup Automation Voice."

4. Use any WebSocket client to connect to the server specifying
   `v1.aria-at.bocoup.com` as [the
   sub-protocol](https://datatracker.ietf.org/doc/html/rfc6455#section-1.9).
   The protocol is described below. (The server will print protocol messages to
   its standard error stream for diagnostic purposes only. Neither the format
   nor the availability of this output is guaranteed, making it inappropriate
   for external use.)

## Terminology

- **message** - a [JSON](https://www.json.org)-formatted string that describes
  some occurrence of interest, emitted at the moment it occurred; the message
  should be a JSON object value with two string properties: `type` and `data`
- **message type** - one of `"lifecycle"`, `"speech"`, or `"error"`
  - `"lifecycle"` - signifies that the message data is an expected lifecycle of
    the automation voice (e.g. initialization and destruction)
  - `"speech"` - signifies that the message data is text which a screen reader
    has requested the operating system annunciate
  - `"error"` - signifies that an exceptional circumstances has occurred
- **message data** - information which refines the meaning of the message type

## Protocol

This project uses an application-level protocol named `v1.aria-at.bocoup.com`
to communicate with clients via a WebSocket connection. All messages are
encoded as JSON text.

```typescript
// Clients may send Command messages to the server at any time. The server will
// respond to every Command it receives with a corresponding Response whose
// `id` value matches that of the Command which initiated it. The client may
// use any numeric value to uniquely identify the Command and to correlate the
// eventual Response.
interface PressKeyCommand {
  type: 'command';
  id: number;
  name: 'pressKey';
  params: [string];
}

interface ReleaseKeyCommand {
  type: 'command';
  id: number;
  name: 'releaseKey';
  params: [string];
}

interface SuccessResponse {
  type: 'response';
  id: number;
  result: any;
}

interface ErrorResponse {
  type: 'response';
  id: number;
  error: string;
  message: string;
}

interface SpeechEvent {
  type: 'event';
  name: 'speech';
  data: string;
}

interface LifecycleEvent {
  type: 'event';
  name: 'lifecycle';
  data: string;
}

interface InternalErrorEvent {
  type: 'event';
  name: 'internalError';
  data: string;
}
```

## Architecture

This tool is comprised of two main components: a text-to-speech voice and a
WebSocket server.

### Text-to-speech voice

The text-to-speech voice is written in C++ and integrates with the [Microsoft
Speech API
(SAPI)](https://docs.microsoft.com/en-us/previous-versions/windows/desktop/ee125663(v=vs.85)).
Because it interfaces with the Windows operating system (that is: "below" the
screen reader in the metaphorical software stack), it can observe speech from
many screen readers without coupling to any particular screen reader.

The voice has two responsibilities. First, it emits the observed speech data
and related events to a [Windows named
pipe](https://docs.microsoft.com/en-us/windows/win32/ipc/named-pipes). This
allows the second component to present a robust public interface for
programmatic consumption of the data. (The named pipe is an implementation
detail. Neither its content nor its presence is guaranteed, making it
inappropriate for external use.)

Second, the voice annunciates speech data. It does this by forwarding speech
data to the system's default text-to-speech voice. This ensures that a system
configured to use the voice remains accessible to screen reader users.

### WebSocket server

The WebSocket server is written in Node.js and allows an arbitrary number of
clients to observe events on a standard interface. It has been designed as an
approximation of an interface that may be exposed directly by screen readers in
the future.

## Contribution Guidelines

For details on contributing to this project, please refer to the file named
`CONTRIBUTING.md`.

## License

Licensed under the terms of the MIT Expat License; the complete text is
available in the LICENSE file.

Copyright for portions of AT Driver are held by Microsoft as part of the
"Sample Text-to-Speech Engine and MakeVoice" project. All other copyright for
AT Driver are held by Bocoup.
