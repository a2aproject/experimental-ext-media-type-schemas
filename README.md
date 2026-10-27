# Media Type Schemas Extension

> **Experimental:** This extension currently has experimental status. Breaking changes are possible and community feedback is much appreciated. To learn more about the extension lifecycle, please refer to [A2A Extension and Protocol Binding Governance](https://a2a-protocol.org/latest/topics/extension-and-binding-governance/).

This repository contains the specification for the **Media Type Schemas Extension** for the Agent2Agent (A2A) protocol, identified by `https://a2a-protocol.org/extensions/media-type-schemas/v1`.

## Purpose

A media type alone does not describe the shape of a structured payload. This data-only extension lets an agent advertise, on its Agent Card, a schema (inline or by URL) for each structured media type it uses. Clients use the schemas to construct conforming input and to interpret typed output.

## Specification

The full v1 specification can be found [here](./v1/spec.md).

## Skill Invocation Hints

The **Skill Invocation Hints Extension**, identified by `https://a2a-protocol.org/extensions/skill-invocation-hints/v1`, builds on this extension. It lets an agent declare per skill a canonical media type for invoking that skill with structured input. A Part in that media type is a hint that the client wants the agent to use the skill. How the agent executes the request is not exposed to the client. An agent that declares it MUST also declare the Media Type Schemas extension.

The v1 specification can be found [here](./skill-invocation-hints/v1/spec.md).
