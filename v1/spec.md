# A2A Protocol Extension: Media Type Schemas (v1 Draft)

- **URI:** `https://a2a-protocol.org/extensions/media-type-schemas/v1`
- **Type:** Data-only Extension
- **Version:** 1.0.0 (draft)
- **Authors:** Vinay Ramesh (Bloomberg)

## 1. Abstract

A2A agents declare the media types they accept and produce through `defaultInputModes`,
`defaultOutputModes`, and each skill's `inputModes` and `outputModes`. A media type alone does not
describe the shape of a structured payload, and a vendor media type such as
`application/vnd.example.order.v1+json` has no defined place where its schema can be found.

This extension defines how an agent advertises, in its Agent Card, a schema for each structured
media type it uses. It also lets an agent declare, per skill, a canonical media type for invoking
that skill with structured input. A Part carrying that media type is a hint that the client wants
the agent to use the skill. When a structured type carrying all relevant information is provided,
agent harnesses MAY be able to apply optimizations internally. Whether they do is not exposed to the
client.

The per-skill canonical modes are a staging ground. They let implementations evaluate whether the
core specification should define a preferred mode for skill invocation.

### 1.1 Definitions

- Schema entry: an object in `params.schemas` (Section 3.1).
- Skill entry: an object in `params.skills` (Section 3.1).
- Typed Part: a `Part` whose `mediaType` matches a schema entry (Section 4.2).
- Canonical input mode: the media type a skill entry declares for invoking that skill with
  structured input.
- Skill hint: the signal carried by a typed Part whose media type is a skill's canonical input mode.

## 2. Extension URI

- **URI:** `https://a2a-protocol.org/extensions/media-type-schemas/v1`
- **Constraint:** This is the only URI accepted for this version of the extension.

The URI is an identifier. HTTP access to it is not expected.

## 3. Extension Capability Declaration

An agent that publishes media type schemas MUST declare this extension in
`AgentCard.capabilities.extensions`. The card below is used throughout this document.

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
            {
              "mediaType": "application/vnd.example.wine-rfq.v1+json",
              "url": "https://supplier.example.com/schemas/wine-v1.schema.json#/$defs/rfq",
              "digest": "sha256-PLACEHOLDERPLACEHOLDERPLACEHOLDERPLACEHOLDER="
            },
            {
              "mediaType": "application/vnd.example.wine-quote.v1+json",
              "schema": {
                "type": "object",
                "required": ["quoteId", "totalMinor", "currency"],
                "properties": {
                  "quoteId": { "type": "string" },
                  "totalMinor": { "type": "integer" },
                  "currency": { "const": "USD" }
                }
              }
            }
          ],
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
  },
  "defaultInputModes": ["text/plain"],
  "defaultOutputModes": ["text/plain"],
  "skills": [
    {
      "id": "wine-quote",
      "name": "Wine quote",
      "description": "Quote a price for cases of a wine delivered to a city.",
      "tags": ["wine", "quote"],
      "inputModes": ["text/plain", "application/vnd.example.wine-rfq.v1+json"],
      "outputModes": ["text/plain", "application/vnd.example.wine-quote.v1+json"]
    }
  ]
}
```

### 3.1 Parameters

| Field     | Type                  | Required | Description |
| :-------- | :-------------------- | :------- | :---------- |
| `schemas` | array of schema entry | Yes      | One schema per structured media type. |
| `skills`  | array of skill entry  | No       | Canonical media types for invoking skills with structured input. |

Schema entry:

| Field         | Type              | Required    | Description |
| :------------ | :---------------- | :---------- | :---------- |
| `mediaType`   | string            | Yes         | The media type described, per [RFC 6838](https://www.rfc-editor.org/rfc/rfc6838). MUST NOT contain wildcards. |
| `schema`      | object or boolean | Conditional | An inline schema. |
| `url`         | string            | Conditional | An absolute `https` URI [RFC 3986](https://www.rfc-editor.org/rfc/rfc3986) for the schema, optionally with a JSON Pointer fragment [RFC 6901](https://www.rfc-editor.org/rfc/rfc6901). |
| `dialect`     | string            | No          | URI of the schema language. Defaults to `https://json-schema.org/draft/2020-12/schema` [JSON Schema 2020-12](https://json-schema.org/draft/2020-12). A `$schema` keyword in the schema takes precedence. |
| `digest`      | string            | No          | Integrity metadata for the bytes retrieved from `url`, in Subresource Integrity format [Subresource Integrity](https://www.w3.org/TR/SRI/). |
| `description` | string            | No          | A human-readable description. |

Skill entry:

| Field                 | Type   | Required | Description |
| :-------------------- | :----- | :------- | :---------- |
| `skillId`             | string | Yes      | The `id` of an `AgentSkill` in the same card. |
| `canonicalInputMode`  | string | Yes      | The media type a client uses to invoke this skill with structured input. |
| `canonicalOutputMode` | string | No       | The media type the skill returns for such a request. |

Rules:

1. Each schema entry MUST contain exactly one of `schema` or `url`. Both forms are fully supported.
   Inline schemas suit small schemas specific to one agent and are covered by an Agent Card
   signature. A `url` suits large or shared schemas, and `digest` extends a card signature to the
   referenced document.
2. Agents MUST NOT publish two schema entries that match the same media type.
3. A schema entry describes shape only. Whether a media type is used for input or output is
   determined by where it appears: `defaultInputModes`, `defaultOutputModes`, skill `inputModes`
   and `outputModes`, and the canonical modes.
4. Agents SHOULD NOT publish schemas for generic media types such as `application/json`. Agents
   SHOULD use a specific media type, typically in the vendor tree with a `+json` suffix [RFC 6839](https://www.rfc-editor.org/rfc/rfc6839),
   and SHOULD put a major version in the subtype.
5. Each `skillId` MUST appear in at most one skill entry and MUST match a skill in the card.
6. `canonicalInputMode` MUST appear in that skill's `inputModes` and MUST have a schema entry. Two
   skill entries MUST NOT declare the same `canonicalInputMode`.
7. If present, `canonicalOutputMode` MUST appear in that skill's `outputModes` and MUST have a
   schema entry.
8. Agents SHOULD set `required` to `false`.

When a `url` carries a fragment, clients SHOULD resolve it as a JSON Schema 2020-12 validator
resolves a `$ref` to the same URI: the whole document is the schema resource, and the fragment
selects a sub-schema. `digest` covers the document without the fragment.

## 4. Client Extension Request

### 4.1 Extension Activation

Activation is OPTIONAL. Clients can read the schemas and send typed Parts without activating the
extension, and agents MUST NOT reject a request only because it was not activated.

A client MAY activate the extension through the `A2A-Extensions` service parameter. Activation
states that the client's typed Parts are intended to conform to the published schemas, and that the
client wants to be told when a typed Part is rejected or not used because it does not conform. An
agent that receives an activation request SHOULD echo the URI in its response and SHOULD report such
non-conformance as an error (Section 4.3) or in a task status message.

### 4.2 Wire Format

A client sends typed input as a standard `Part` with `mediaType` set and, for JSON content, the
payload in `data`. Other Parts in the same Message, such as text instructions, are context.
Non-JSON media types use `raw` or `url`, and the schema applies to the resource bytes.

A Part's `mediaType` matches a declared media type when type and subtype are equal,
case-insensitively, and every parameter in the declared media type is present on the Part with the
same value. A Part without `mediaType` matches nothing.

A client that wants typed output lists the output media type in `acceptedOutputModes`. An agent
returning typed output SHOULD place it in an `Artifact` Part with that `mediaType`.

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
  to a tool or workflow. A non-conforming Part MUST NOT be treated as conforming.
- An agent that labels an output Part with a schema-described media type MUST ensure the content
  conforms. If it cannot, it MUST NOT use that label.

Errors reuse core A2A errors:

| Condition | Response |
| :-------- | :------- |
| Media type not accepted | `ContentTypeNotSupportedError` |
| Typed Part fails validation, request rejected | `InvalidParamsError` with a `google.rpc.BadRequest` detail naming the Part and the failing JSON Pointer |
| Typed Part fails validation, correction wanted | `TASK_STATE_INPUT_REQUIRED` with a status message (the [Structured Input extension](https://github.com/a2aproject/experimental-ext-structured-input) MAY be used) |

### 4.4 Example

The client sends an RFQ in the canonical input mode of `wine-quote`, activating the extension.

```http
POST /a2a HTTP/1.1
Host: supplier.example.com
Content-Type: application/json
A2A-Extensions: https://a2a-protocol.org/extensions/media-type-schemas/v1

{
  "jsonrpc": "2.0",
  "id": 1,
  "method": "SendMessage",
  "params": {
    "message": {
      "messageId": "6f1d2c1e-8a4b-4d8e-9f1a-2b7c3d4e5f60",
      "role": "ROLE_USER",
      "parts": [
        {
          "mediaType": "application/vnd.example.wine-rfq.v1+json",
          "data": { "wine": "Barolo", "vintage": 2019, "cases": 12, "deliveryCity": "San Jose" }
        }
      ]
    },
    "configuration": {
      "acceptedOutputModes": ["application/vnd.example.wine-quote.v1+json", "text/plain"]
    }
  }
}
```

The agent authorizes the caller for `wine-quote`, validates the Part, and handles it by whatever
path it chooses. The response has the same form either way.

```http
HTTP/1.1 200 OK
Content-Type: application/json
A2A-Extensions: https://a2a-protocol.org/extensions/media-type-schemas/v1

{
  "jsonrpc": "2.0",
  "id": 1,
  "result": {
    "task": {
      "id": "0b9a7c52-3e1f-4a6d-8c2b-91d4e5f60718",
      "contextId": "4c3b2a19-7d6e-4f5a-b8c9-0a1b2c3d4e5f",
      "status": { "state": "TASK_STATE_COMPLETED" },
      "artifacts": [
        {
          "artifactId": "quote-Q-1042",
          "parts": [
            {
              "mediaType": "application/vnd.example.wine-quote.v1+json",
              "data": { "quoteId": "Q-1042", "totalMinor": 650500, "currency": "USD" }
            }
          ]
        }
      ]
    }
  }
}
```

## 5. Security Considerations

- Schemas are untrusted input. They MUST NOT be executed. Text such as `description` is a prompt
  injection vector when schemas are placed in a model's context. Implementations SHOULD limit
  schema size, depth, and validation time, and SHOULD guard against costly `pattern` expressions.
  A conforming payload can still be harmful for a given operation.
- Fetching a `url` is a request to a location chosen by a remote party. Fetchers MUST only retrieve
  `https` URLs, SHOULD refuse loopback, link-local, and private addresses (including after DNS
  resolution and redirects), SHOULD limit redirects, size, and time, and SHOULD NOT follow remote
  `$ref` values by default. When `digest` is present, clients MUST verify it and MUST NOT use a
  schema that fails.
- A skill hint does not grant authorization (Section 4.3). A denied
  request MUST NOT be retried internally through a different path with the same effect.
