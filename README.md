# Media Type Schemas Extension

> **Experimental:** This extension currently has experimental status. Breaking changes are possible and community feedback is much appreciated. To learn more about the extension lifecycle, please refer to [A2A Extension and Protocol Binding Governance](https://a2a-protocol.org/latest/topics/extension-and-binding-governance/).

This repository contains the specification for the **Media Type Schemas Extension** for the Agent2Agent (A2A) protocol, identified by `https://a2a-protocol.org/extensions/media-type-schemas/v1`.

## Purpose

A media type alone does not describe the shape of a structured payload. This data-only extension lets an agent advertise, on its Agent Card, a schema (inline or by URL) for each structured media type it uses, and declare per skill a canonical media type for invoking that skill with structured input. A Part in that media type is a hint that the client wants the agent to use the skill. How the agent executes the request is not exposed to the client.

## Specification

The full v1 specification can be found [here](./v1/spec.md).
