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
	instead of choosing an action.
  - Each time a turn happens, the server sends an
    [Update message](#update-message) to both agents, which has the history of
	[turns](#turn-object) taken so far, the current [state](#state-object), and
	the [choices](#turn-object) that agent can make. An agent knows it is their
	turn if their [Update](#update-message) has choices listed.
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

## Connect Message

This section coming soon.

## Join Message

This section coming soon.

## Start Message

This section coming soon.

## Update Message

This section coming soon.

## Choice Message

This section coming soon.

## Report Message

This section coming soon.

## Stop Message

This section coming soon.

## End Message

This section coming soon.

## Error Message

This section coming soon.

---

## State Object

This section coming soon.

## Status Object

This section coming soon.

## Turn Object

This section coming soon.

## World Object

This section coming soon.