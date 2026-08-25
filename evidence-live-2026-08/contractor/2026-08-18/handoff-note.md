# Handoff note — Ratify federation run, 2026-08-18

Written by Agent Relay's implementation agent under a Ratify delegation issued by
Ratify Protocol's root, sub-delegated by the contractor lead, and constrained to
this repository under `/docs`.

The root delegation is `relay-run-1787074501`, issued by `345140967b…` to Agent
Relay's contractor lead with `files:write` and `identity:delegate`. The lead
narrowed it for this write: `files:write` alone, no further delegation, bound to
this repository under `/docs`.

The write was authorized by `ratify_task`, which is the surface that passes the
requested resource to the verifier. A request for a path outside `/docs` under
the same certificate is refused `constraint_denied` — the constraint is carried
on every path and checked on this one.

Verified before this commit landed, in this order:

- the delegation chain verifies against an issuer established out of band, and
  against Ratify Protocol's real root rather than a stand-in this side minted
- the same chain is **refused** for a path outside `/docs`, checked before any
  bytes were written
- the commit is authored by AgentRelayBot, and GitHub attributes it to that
  account rather than leaving it unlinked

The verdict and the refusal control are reproduced in the pull request body.
