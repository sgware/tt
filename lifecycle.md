---
title: Tandem Tales Agent Lifecycle
---

A Tandem Tales agent connects to the
[server](https://github.com/sgware/tt-server) via a
[TLS encrypted socket](https://en.wikipedia.org/wiki/Transport_Layer_Security)
(on port 9005, by default) and plays a single interactive story session. All
messages to and from the server are formatted as [JSON](http://www.json.org) and
terminated by a new line character.

- Starting a Session:
  - As soon as a new socket is established, the server sends the
    [Connect message](#connect-message) that lists the available worlds and
    partners.
  - When the agent is ready to play, it sends the
    [Join message](#join-message) with the details of who it is and the settings
    for the game it wants to play.
  - Once the server has found a compatible partner, it starts a new session and
    sends the [Start message](#start-message) to both agents, which tells them
    their roles, about the [story world](#world-object), and the world's
    [initial state](#state-object).
- During a Session:
  - The game master goes first.
  - On the game master's turn, the game master can `SUCCEED` or `FAIL` any
    number of actions that do not require the player's consent. The game master
    can `PASS` control to the player. The game master can `PROPOSE` an action if
    it requires the consent of the player and a non-player character, and then
    play immediately passes to the player.
  - The player always chooses one action and then passes control back to the
    game master. If the game master's last action was to `PROPOSE` an action,
    the player must choose whether it will `SUCCEED` or `FAIL`. If the game
    master's last action was to `PASS`, the player can `PROPOSE` any action that
    requires the player's consent or `PASS` control back to the game master
    without choosing an action.
  - Each time a turn happens, the server sends an
    [Update message](#update-message) to both agents, which has the history of
    [turns](#turn-object) taken so far, the current [state](#state-object), and
    the [choices](#turn-object) that agent can make. An agent knows it is their
    turn if their [Update message](#update-message) has choices listed.
  - When it is an agent's turn, they make a choice by sending a
    [Choice message](#choice-message) to the server with the index of a choice
    from their last [Update](#update-message).
  - At any time during the session, an agent can send a
    [Report message](#report-message) to the server to record thier answer to a
    question.
- Ending a Session:
  - If the story reaches one of its pre-defined endings, the
    [Update message](#update-message) sent to both agents will include an
    [Ending object](#ending-object).
  - Either agent may end the session early by sending a
    [Stop message](#stop-message). Once an agent sends a Stop message, it should
    not send any more messages to the server, but it may stay connected to wait
    for the [End message](#end-message).
  - If an agent disconnects from the server, it is treated as if it had sent a
    [Stop message](#stop-message).
  - The server sends a [Stop message](#stop-message) to the agents when it is
    time for a session to end. This happens when the story reaches a pre-defined
    ending, when one agent stops the session early, or when the server is
    shutting down and needs to end all session early.
  - When an agent receives a [Stop message](#stop-message) (but has not yet sent
    one), it cannot send any more [Choice messages](#choice-message), but it can
    still send [Report messages](#report-message) to answer questions. Once an
    agent sends its [Stop message](#stop-message), it cannot send any more
    messages to the server, but it may stay connected to wait for the
    [End message](#end-message).
  - Once the server has sent a [Stop message](#stop-message) to both agents, and
    once both agents have sent a [Stop message](#stop-message) to the server (or
    effectively sent one by disconnecting), the server sends the
    [End message](#end-message) to both agents to tell them the session ID
    number and let them know if it time to disconnect.

If an agent does something that is not allowed, the server may send an
[Error message](#error-message) explaining the problem. Depending on the error,
it may also disconnect the agent.

---

# Messages

Messages are encoded in JSON. All messages include the key `type` which defines
the type of message as a string. Messages are always a single JSON object
followed by a new line character. If a message contains a new line character, it
should be encoded.

## Connect Message

```
{
    "type": "Connect",
    "version": "1.0.0",
    "worlds": [
        {
            "name": "tutorial",
            "title": "Tutorial",
            "description": "A simple story about buying a drink that will teach you how to play."
        }
    ],
    "agents": [
        {
            "name": "random",
            "title": "Random",
            "description": "An agent that makes random choices."
        }
    ],
    "available": [
        {
            "agent": "random",
            "world": "tutorial"
        }
    ]
}
```
The `Connect` message is sent from the server to the client as soon as a new
socket is opened.
- `version` gives the server's version number as a string.
- `worlds` is an array of objects describing which worlds the server has
  publically listed as available. Each has a `name`, `title`, and `description`.
- `agents` is an array of objects describing which agent types the server has
  publically listed as available. Each has a `name`, `title`, and `description`.
- `available` is an array of objects listing agents who are currently connected
  and looking for partners to play in specific worlds. Each has an `agent` and
  `world`.

## Join Message

```
{
    "type": "Join",
    "name": "web",
    "password": "dummy",
    "world": "tutorial",
    "role": "PLAYER",
    "partner": "random"
}
```
The `Join` message is sent from the client to the server when the client is
ready to start its session.
- `name` gives the client's agent type. It does not need to be unique.
- `password` gives the client's password. A password is only required if the
  client's name is reserved on the server.
- `world` is the name of the story world the client wants to play in. If this
  is missing or `null`, the client is willing to play in any world.
- `role` is the role the client wants to have in the story. It can be `PLAYER`
  or `GAME_MASTER`. If this is missing or `null`, the client is willing to play
  as either role.
- `partner` is the agent type the client wants to play with. If this is missing
  or `null`, the client is willing to play with any partner.

## Start Message

```
{
    "type": "Start",
    "role": "GAME_MASTER",
    "world": # World object #
}
```
The `Start` message is sent from the server to the client once the server has
found a compatible pair of agents and started a new session.
- `role` is the client's role in the story. It will be either `PLAYER` or
  `GAME_MASTER`.
- `world` is a [World object](#world-object).

## Update Message

```
{
    "type": "Update",
    "status": # Status object #
}
```
An `Update` message is sent from the server to the client each time the state of
the world changes.
- `status` is a [Status object](#status-object).

## Choice Message

```
{
    "type": "Choice",
    "index": 3
}
```
A `Choice` message is sent from the client to the server when it is the client's
turn to notify the server which turn the agent wants to take.
- `index` is the array index (starting at 0) of a choice from the most recent
  [Update message](#update-message) sent to the agent.

## Report Message

```
{
    "type": "Report",
    "item": "structure",
    "value": "4",
    "comment": "strongly agree"
}
```
A `Report` message is sent from the client to the server any time after the
server sends the [Start message](#start-message) and before the client sends
their [Stop message](#stop-message). It is used by the client to answer
questions about the story or play experience.
- `item` is the name of the question the client is answering.
- `value` is the client's answer to the question as a string.
- `comment` is a comment on the value as a string.

## Stop Message

```
{
    "type": "Stop",
    "role": "PLAYER",
    "message": "End the session early."
}
```
```
{
    "type": "Stop",
    "message": "The story has ended.",
    "ending": # Ending object
}
```
A `Stop` message can be sent by either the server or client. It can only be sent
by either party after the server has sent the [Start message](#start-message).
The client can end the session early by sending this message. The server sends
this message if the story reaches one of its pre-defined
[endings](#ending-object), if the client's partner ended the session early or
disconnected, or if the server needs to end the session early (i.e. because it
is shutting down).
- `role` identifies which client ended the session early. This should be missing
  or `null` if the story reached a pre-defined ending or if the server is ending
  the session early.
- `message` is an explanation for why the session is ending.
- `ending` is an [Ending object](#ending-object). This should be set if the
  message is coming from the server and the story reached a pre-defined ending;
  otherwise, it should be missing or `null`.

## End Message

```
{
    "type": "End",
    "session": "MWD7D5R590"
}
```
The `End` message is sent by the server to the client once the session is
complete and the client can disconnect.
- `session` is a unique ID string that identifies the session in the server
  logs.

## Error Message

```
{
    "type": "Error",
    "message": "The message could not be parsed as a JSON object."
}
```
An `Error` message is sent from the server to the client if it cannot parse the
client's message or if the message has invalid values (i.e. if the client sends
a [Choice message](#choice-message) at the wrong time or with an invalid index).
- `message` is a string that explains the problem.

---

# Objects

Object are encoded in JSON. If an object is being sent as part of a
[Message](#messages) and contains a new line character, the new line character
should be encoded to avoid terminating the message prematurely.

## State Object

This section coming soon.

## Status Object

This section coming soon.

## Turn Object

This section coming soon.

## World Object

```
{
    "name": "tutorial",
    "entities": [ # Entity objects # ],
    "variables": [ # Variable objects # ],
    "actions": [ # Action objects # ],
    "endings": [ # Ending objects # ]
}
```
A story `World` object defines:
- The `name` of the story world as a string.
- An array of [Entity objects](#entity-object) defining the objects that exist
  in the story world.
- An array of [Variable objects](#variable-object) defining the variables that
  make up the story world's state.
- An array of [Action objects](#action-object) defining the actions which can
  change the story world's state.
- An array of [Ending objects](#ending-object) defining the possible endings
  that a story in this world can reach.