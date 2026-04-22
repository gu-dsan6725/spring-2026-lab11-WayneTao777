# Lab 11 Observations

## A2A Messages Exchanged

During the booking workflow, the Travel Assistant Agent first searched for a remote agent that could handle flight reservations, confirmations, and payments. The service log showed:

```text
Tool called: discover_remote_agents(query='flight reservation booking confirmation payment processing', max_results=5)
Discovery successful: found 1 agents, cached 1 new
Invoking agent: Flight Booking Agent
```

The Travel Assistant then sent an A2A JSON-RPC request to the Flight Booking Agent:

```json
{
  "jsonrpc": "2.0",
  "method": "message/send",
  "params": {
    "message": {
      "role": "user",
      "parts": [
        {
          "kind": "text",
          "text": "I need to book flight ID 1 with 2 seats. Please reserve the seats, confirm the reservation, and process the payment."
        }
      ],
      "messageId": "99a13c280b204feb8d002fe04fe5c73b"
    }
  }
}
```

The final Task 3 test output showed that the booking agent was discovered and invoked. The response preview included flight ID 1, United flight UA101, available seats, and the total price for 2 seats.

## How Discovery Worked

The Travel Assistant used the registry stub at `http://127.0.0.1:7861`. It sent a semantic search query for booking-related capabilities. The registry returned one matching agent: the Flight Booking Agent.

The saved discovery API response in `task3_discover_agents_response.json` included:

```json
{
  "query": "book flights",
  "agents_found": 1,
  "agents": [
    {
      "name": "Flight Booking Agent",
      "url": "http://127.0.0.1:10002",
      "tags": ["booking", "flights", "reservations"],
      "trust_level": "verified",
      "relevance_score": 0.95
    }
  ]
}
```

## JSON-RPC Format Observed

The A2A request used JSON-RPC 2.0 with these important fields:

- `jsonrpc`: identifies the JSON-RPC version.
- `method`: `message/send`, the A2A method used to send a message.
- `params.message.role`: `user`, meaning the Travel Assistant sent a user-style request to the remote agent.
- `params.message.parts`: a list of content parts. In this lab, the part was text.
- `params.message.messageId`: a unique identifier for the message.

The test outputs also showed A2A responses returning a `result` object with `artifacts`. Each artifact contained `parts`, and the text part held the agent's natural-language response.

## Agent Card Information

Before invoking the Flight Booking Agent, the Travel Assistant fetched the agent card from:

```text
http://127.0.0.1:10002/.well-known/agent-card.json
```

The agent card contained:

- Agent name: `Flight Booking Agent`
- Description: `Flight booking and reservation management agent`
- URL: `http://127.0.0.1:10002/`
- Protocol: JSON-RPC transport
- Input and output modes: text
- Skills: `check_availability`, `reserve_flight`, `confirm_booking`, `process_payment`, and `manage_reservation`

The Travel Assistant used this card to initialize an A2A client and send the booking request to the correct endpoint.

## Benefits and Limitations

Benefits:

- The Travel Assistant did not need to hardcode every booking capability itself.
- The registry made it possible to discover another agent by capability.
- The agent card provided a standard way to describe an agent's endpoint and skills.
- JSON-RPC gave the agents a consistent request and response shape.

Limitations:

- The registry in this lab is a stub, so it always returns the Flight Booking Agent rather than doing real semantic ranking.
- All three services must be running locally for the workflow to work.
- Debugging requires checking multiple logs because discovery, orchestration, and booking happen in different services.
- The model can sometimes produce partial or cautious responses even when the tool calls succeed, so logs are important for verifying the actual A2A flow.
