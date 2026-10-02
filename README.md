# Octet Startup Box

**Status: pre-development specification.** This repository documents a proposed locally hosted organizational intelligence system, “Startup in a Box.” It contains a product specification, an implementation plan, agent guidance and an MIT license. There is no application server, frontend, package manifest or test suite in the current source tree.

The design explores context ingestion, AI personas and memory, inference queues, organizational views, meetings and generated artifacts. These are requirements for future implementation, not delivered capabilities. No runnable demo, beta release or completed deployment is established by this repository.

## Read the design

- [Product specification v0.8](octet-startup-box-spec.md): architecture, proposed workflows, state schema and acceptance criteria.
- [Development plan](development-plan.md): staged website/prototype workstreams and implementation dependencies.
- [Agent guidelines](AGENTS.md) and [agent context](CLAUDE.md): constraints to apply when code is built.

The intended architecture is a raw Node.js server and vanilla HTML/CSS/JavaScript, with inference behind an adapter and runtime organizational data in a separate sibling directory. Those architectural seams and file paths are planned; they do not yet exist as modules in this repository.

## Running and verification

There are no current install, run or test commands. Commands such as `node server.js` and Playwright examples in the planning documents describe a future implementation. They will fail against this documentation-only tree.

The first implementation milestone should establish the server foundation and verify a clean start, durable state and observable queue behavior before downstream persona/meeting features. Record checks against the actual source commit and keep generated organizational data, logs and credentials out of this repository.

## Contributing and license

Follow the staged plan, preserve the adapter/data boundaries, and describe each increment's actual behavior and limitations. Specification edits can be reviewed without claiming runtime validation. Licensed under [MIT](LICENSE).
