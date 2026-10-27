# A2A Protocol Extension: Skill Invocation Hints (v1 Draft)

- **URI:** `https://a2a-protocol.org/extensions/skill-invocation-hints/v1`
- **Type:** Data-only Extension
- **Version:** 1.0.0 (draft)
- **Depends on:** [Media Type Schemas](../../v1/spec.md) (`https://a2a-protocol.org/extensions/media-type-schemas/v1`)
- **Authors:** Vinay Ramesh (Bloomberg)

## 1. Abstract

The Media Type Schemas extension lets an agent publish a schema for each structured media type it
uses. This extension builds on it. It lets an agent declare, per skill, a canonical media type for
invoking that skill with structured input. A Part carrying that media type is a hint that the client
wants the agent to use the skill. When a structured type carrying all relevant information is
provided, agent harnesses MAY be able to apply optimizations internally. Whether they do is not
exposed to the client.

The per-skill canonical modes are a staging ground. They let implementations evaluate whether the
core specification should define a preferred mode for skill invocation.

### 1.1 Definitions

- Skill entry: an object in `params.skills` (Section 3.1).
- Canonical input mode: the media type a skill entry declares for invoking that skill with
  structured input.
- Skill hint: the signal carried by a typed Part whose media type is a skill's canonical input mode.
- Schema entry and typed Part: as defined by the Media Type Schemas extension.

## 2. Extension URI

- **URI:** `https://a2a-protocol.org/extensions/skill-invocation-hints/v1`
- **Constraint:** This is the only URI accepted for this version of the extension.

The URI is an identifier. HTTP access to it is not expected.

## 3. Extension Capability Declaration

An agent that publishes skill invocation hints MUST declare this extension in
`AgentCard.capabilities.extensions`, and MUST also declare the Media Type Schemas extension. The
card below extends the example card of the Media Type Schemas extension. Its `schemas` and `skills`
are unchanged and are elided here.

```json
{
  "name": "Example Wine Supplier",
  "capabilities": {
    "extensions": [
      {
        "uri": "https://a2a-protocol.org/extensions/media-type-schemas/v1",
        "required": false,
        "params": {
          "schemas": [
            { "mediaType": "application/vnd.example.wine-rfq.v1+json", "url": "..." },
            { "mediaType": "application/vnd.example.wine-quote.v1+json", "schema": {} }
          ]
        }
      },
      {
        "uri": "https://a2a-protocol.org/extensions/skill-invocation-hints/v1",
        "required": false,
        "params": {
          "skills": [
            {
              "skillId": "wine-quote",
              "canonicalInputMode": "application/vnd.example.wine-rfq.v1+json",
              "canonicalOutputMode": "application/vnd.example.wine-quote.v1+json"
            }
          ]
        }
      }
    ]
  }
}
```

### 3.1 Parameters

| Field    | Type                 | Required | Description |
| :------- | :------------------- | :------- | :---------- |
| `skills` | array of skill entry | Yes      | Canonical media types for invoking skills with structured input. |

Skill entry:

| Field                 | Type   | Required | Description |
| :-------------------- | :----- | :------- | :---------- |
| `skillId`             | string | Yes      | The `id` of an `AgentSkill` in the same card. |
| `canonicalInputMode`  | string | Yes      | The media type a client uses to invoke this skill with structured input. |
| `canonicalOutputMode` | string | No       | The media type the skill returns for such a request. |

Rules:

1. Each `skillId` MUST appear in at most one skill entry and MUST match a skill in the card.
2. `canonicalInputMode` MUST appear in that skill's `inputModes` and MUST have a schema entry in
   the Media Type Schemas extension. Two skill entries MUST NOT declare the same
   `canonicalInputMode`.
3. If present, `canonicalOutputMode` MUST appear in that skill's `outputModes` and MUST have a
   schema entry.
4. Agents SHOULD set `required` to `false`.

## 4. Client Extension Request

### 4.1 Extension Activation

Activation is OPTIONAL. A client can send a typed Part in a canonical input mode without activating
this extension, and agents MUST NOT reject a request only because it was not activated.

A client MAY activate this extension through the `A2A-Extensions` service parameter to state that
its typed Part is intended as a skill hint. An agent that receives an activation request SHOULD
echo the URI in its response. Activating this extension does not activate the Media Type Schemas
extension. A client that wants conformance reporting activates both.

### 4.2 Wire Format

A skill hint is a typed Part, as defined by the Media Type Schemas extension, whose media type is a
skill's `canonicalInputMode`. No other field is added to the request.

Clients SHOULD NOT put the canonical input modes of different skills in one Message. Agents MAY
treat such a Message as carrying no skill hint.

### 4.3 Server Behavior

A typed Part whose media type matches a skill's `canonicalInputMode` is a skill hint for that skill.
If the request also selects a skill another way and the two conflict, the agent SHOULD prefer the
explicit selection.

- The agent MAY use the hint to select the skill and route the payload to an internal tool or
  workflow, and MAY instead handle the request through its general logic.
- Either way, the agent MUST return standard A2A responses and MUST apply the same authentication
  and authorization as for any other request to that skill. A hint MUST NOT be treated as a grant
  of permission.
- This extension defines no signal of how a request was executed. Clients MUST NOT depend on a
  particular execution path.
- An agent MUST validate a typed Part, or perform an equivalent check, before routing it directly
  to a tool or workflow.

Errors are those of the Media Type Schemas extension and core A2A.

### 4.4 Example

The request and response are the example of the Media Type Schemas extension, with both extensions
activated:

```http
A2A-Extensions: https://a2a-protocol.org/extensions/media-type-schemas/v1, https://a2a-protocol.org/extensions/skill-invocation-hints/v1
```

The RFQ Part is in the canonical input mode of `wine-quote`. The agent authorizes the caller for
`wine-quote`, validates the Part, and handles it by whatever path it chooses. The response has the
same form either way.

## 5. Security Considerations

- The security considerations of the Media Type Schemas extension apply.
- A skill hint does not grant authorization (Section 4.3). A denied request MUST NOT be retried
  internally through a different path with the same effect.
