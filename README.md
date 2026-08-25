# Ratify × Agent Relay — Phase 2 live evidence (contractor half)

This is Agent Relay's half of the evidence from two live Ratify Protocol runs, on
2026-08-18 and 2026-08-19. Identities AI holds the other half in
[`identities-ai/ratify-agent-relay-harness`](https://github.com/identities-ai/ratify-agent-relay-harness)
under `evidence-live-2026-08/client/`, along with the runnable reproduction.

In both sessions Ratify Protocol held the root. They issued delegation
certificates to an agent running on Agent Relay, that agent did real work in a
real repository, and they revoked a live certificate mid-run while the agent
still had hours left on it.

## What is here

```
evidence-live-2026-08/contractor/
  2026-08-18/    flagship run — gated write (PR #5), kill switch, adversarial annex
  2026-08-19/    session 2 — federation beat live, receipts persisted, kill switch re-shot
```

Each session directory contains its `RUN-RECORD.md` and the artifacts it refers
to: delegation certificates as issued, the child certificates minted by
sub-delegation, and for each verification a signed **receipt** and a separately
signed **deployment decision**.

A receipt has no identifier of its own. It is named by its hash, `prev_hash`
chains it to the one before it, and the deployment decision binds to it by
`receipt_hash`. That pair is the per-call handle, and neither half alone is an
authorization — the receipt carries what Ratify core concluded, the decision
carries whether this deployment agreed to serve it, and a consumer has to read
both. `checkPairedDecision` in the adapter is the fail-closed form of that check.

## The deployment

Everything here was produced by `ratify.agentrelay.com` running

```
sha256:ad6436801571276bb0db21f194e9a92dbb540e20cef86f79f2f08b03cde62e91
```

The image itself is attached as a release asset on this repository, exported with
`docker save` from a registry pull rather than a local build, so the manifest
digest matches the one the deployment was served from. Identities AI verified
this digest independently before the 2026-08-19 session.

Both repositories build against `@identities-ai/ratify-protocol@1.0.0-alpha.16`,
which is what is baked into that image. Identities AI have since published
`alpha.17`; its fixtures are byte-identical and no SDK behaviour changed, but
this evidence was produced under alpha.16 and is pinned to it deliberately.

## The write

The gated write from session 2 is commit `7ec0c5ab0d4c`, merged as PR #7 into
`identities-ai/ratify-agent-relay-engagement`. Its receipt hash is

```
OOnGiGy8HkmT+L5Pyvg8T7R/zqU0Y5rQpsvImFRx0bo=
```

PR #6 was an earlier attempt that failed the engagement repo's markdown lint and
was closed as superseded rather than force-pushed over — hand-pushing a fix would
have put bytes in that repository that were not downstream of a verdict, which is
the one thing the run exists to disprove. Its receipt hash `jGhg7BhL…` belongs to
an abandoned branch and should not be quoted for anything.

## What this does not show

The run records name their own gaps, and those are the honest starting point for
reading any of this:

- **A revoked presentation is not published to the deployment's own metadata.**
  The adapter self-publishes accepts and policy refusals to
  `ratify_last_presentation`, but an invalid inbound presentation returns before
  that publish. So the one verdict the kill switch turns on is missing from the
  place a counterparty would look to check our account of it. The refusal is
  real, signed, and in the receipts here — but it is on our word that the
  deployment's own metadata would agree. Not fixed at time of publication:
  fixing it means a new image, and the digest above would stop matching.
- **The reverse direction has not run.** Their agents presenting to us is not
  demonstrated in either session.
- **The 18-case serve-authority suite runs in-process** against a locally
  generated verifier, not against the deployment. The live federation beat did
  fire under their root in session 2 and is in `2026-08-19/`, but the suite is
  not the deployment.

## On the records themselves

`RUN-RECORD.md` in each directory is reproduced exactly as it was written on the
day, including the findings and the mistakes. Where something in a record was
later corrected — session 2's open list still says PR #7 is awaiting merge, for
example, and it was merged — the correction belongs here in the README, not in
the record. Editing an evidence record after the fact to make it read better is
the specific thing this whole exercise argues against.

Agent Relay's write-up is at
[agentrelay.com/blog/someone-elses-agent-in-your-repo](https://agentrelay.com/blog/someone-elses-agent-in-your-repo).
The protocol-side technical note from Identities AI is at
[ratifyprotocol.com/writing/agent-relay-phase2-technical-note](https://ratifyprotocol.com/writing/agent-relay-phase2-technical-note).
