# Run record — session 2, 2026-08-19

Second session of the Ratify Phase 2 run. It existed to close three gaps left by
2026-08-18, **all three found by Chuks auditing his capture list afterwards
rather than by us**: no contractor-side footage, the federation beat never fired
live, and receipts emitted then discarded.

`RUNBOOK-2026-08-19.md` is the procedure. `run-2026-08-18/RUN-RECORD.md` is the
previous session and is not superseded by this one.

| gap from 08-18 | state after today |
|---|---|
| no contractor-side footage | **closed** — kill switch re-shot live, both sides rolling |
| step 4, the federation beat, never fired | **closed** — ran live under their root, both halves |
| receipts + decisions discarded | **closed** — persisted on every call, hash in the PR body |

## Preflight

```
17:33:44Z  go/no-go GO 11/11        scripts/go-no-go.sh
           running image  sha256:ad6436801571276bb0db21f194e9a92dbb540e20cef86f79f2f08b03cde62e91
           adapter        AgentRelayFederationWorker active, last seen 17:33:30Z
```

The 08-19 morning run (11:01Z) was stale by session time and was re-run.

## Certificates

All minted by Identities AI under their real root, all checked with
`scripts/check-cert.py` before driving anything.

| # | cert_id | resource | prefix | issued | expires |
|---|---|---|---|---|---|
| 1 | `relay-run-1787160417` | `relay:v1:ratify.agentrelay.com:channel:213287751714545664` | absent | 17:26:57Z | 19:26:57Z |
| 2 | `relay-run-1787160430` | `relay:v1:relay.ratifyprotocol.com:channel:213287751714545664` | absent | 17:27:10Z | 19:27:10Z |
| 3 | `relay-run-1787160460` | `git:github.com/identities-ai/ratify-agent-relay-engagement` | `/docs` | 17:27:40Z | 21:27:40Z |
| 4 | `relay-run-1787161127` | same repo, the B4 control | `/docs` | 17:38:47Z | 21:38:47Z |

Cert 4 was minted mid-session. The original plan had no control: the kill switch
revokes cert 3, so B4 needed a *different* cert that was still valid, and certs 1
and 2 expire at 19:27 — before the switch would have fired. Caught before the
mark; Chuks had it at 60 minutes and re-minted at 240.

⚠️ **`path_prefix` is absent, not empty, on certs 1 and 2.** Their tooling had
been writing `path_prefix: ""`, which Ratify rejects on the wire — the failure
would have read as a malformed certificate rather than a wrong intent. Fixed
their side before minting, and verified here against the encoded files rather
than console output.

## Step 4 — the federation beat, live for the first time

Same channel identifier under two deployment authorities. Until today these
cases had only run in-process against a locally generated verifier; they had
never touched the deployed adapter or Ratify Protocol's root.

```
served    core authorized_agent · deployment served true
          receipt gwDnZE8NcTSsJz2PSKSwDVZuqBa8HPJ7UbX8keJWdeo=
unserved  core authorized_agent · deployment served FALSE · unserved_authority
          receipt 0rYU5tBGF51ZIRZbKwxZ8s2vzQ+6OUGqWxMexC/rzME=
```

The load-bearing half is what the refusal did **not** do. The core chain result
for the refused presentation is `authorized_agent` and the receipt records it as
such: the delegation is valid and the deployment declined to act on it. Had the
refusal also broken the chain result, `unserved_authority` would be
indistinguishable from a bad delegation, and a deployment could hide policy
behind protocol.

Both verdicts were read back from the deployment's own agent metadata
(`ratify_last_presentation`) within seconds, and the `receipt_hash` there matched
the receipt the driver wrote. That is a check Identities AI could not previously
make about anything we reported.

## Step 5 — the gated write

Two writes, because the first failed the engagement repo's markdown lint.

```
17:45:31.787Z  PR #6  commit d89ba16825ef  receipt jGhg7BhL+9Prl3hJfvRbgqdf+iTAom3xMmzSL8ZcJvI=
               lint RED — 9 markdownlint errors, all ours, none touching Ratify
17:49:28.545Z  PR #7  commit 7ec0c5ab0d4c  receipt OOnGiGy8HkmT+L5Pyvg8T7R/zqU0Y5rQpsvImFRx0bo=
               lint GREEN
```

PR #6 was closed as superseded rather than force-pushed over: the publisher
clones upstream fresh and pushes without `--force`, and hand-pushing a fix would
have put bytes in that repository that were not downstream of a verdict — the
one thing this run exists to disprove. The fix went through the gate as a second
write under the same certificate.

**The receipt hash for the write-up is `OOnGiGy8…`, bound to `7ec0c5ab0d4c`.**
`jGhg7BhL…` belongs to the abandoned branch and must not be quoted.

Gate held on both runs: `constraint_denied` for the same certificate at a path
outside `/docs`, `authorized_agent` inside it. The control runs **before** the
gate so the deployment's last published verdict matches the commit rather than
reading as a refusal.

Four child certificates were persisted with receipt and decision, closing
08-18's gap where the child behind the merged write could not be quoted:

```
work-2fcdfb239fbc1fa1  control, PR #6      work-c7058bd2179ea7b3  write, PR #6
work-de855c0c9e5e67dc  control, PR #7      work-49505dedc088b2ce  write, PR #7
```

## Kill switch — Procedure B

Both sides recording **for this section only** — by agreement the beat and the
write were not filmed, on the grounds that running tape through everything is
what burned two takes on 08-18. Each side confirmed it was rolling by posting a
clock reading in the thread before the mark, ours at 17:58:35Z. That
confirmation is the thing 08-18 lacked, and it is why this session existed.

```
18:00:04.967 → 18:00:08.307   present, cert 3   authorized_agent [files:write]   12455s left
18:02:00Z                     his declared mark (posted in Slack, not spoken)
18:02:09.706Z                 his send      (his stamp, supplied by Identities AI)
18:02:11.123Z                 applied       revoked_certs ["relay-run-1787160460"], applied_count 1
18:03:01.702 → 18:03:05.395   B3 (first)        revoked []                        12278s left
18:03:09.431 → 18:03:12.831   B4 control        authorized_agent [files:write]    12938s left
18:04:27.027 → 18:04:30.080   B3 (second)       revoked []                        12193s left
18:05:40.376 → 18:05:44.799   fail-closed       nothing written
```

Applied time read from `ratify_last_revocation` in agent metadata — container
stdout is unreadable on this deployment.

**B3 cannot be read as expiry**: 12278s — 3.4 hours — remained on a certificate
that does not expire until 21:27:40Z.

**B4 makes it targeted rather than an outage.** Cert 4 differs from cert 3 only
in `cert_id`, `issued_at`, `expires_at` and `signature`; issuer, issuer key,
subject, subject key, scope and constraints are identical.

**B3 ran twice, and the record says so.** The second run is the one that followed
a request for its output, not a re-enactment. It leaves the sequence as
refused → authorized → refused, which brackets the control between two refusals
and closes off any reading that the verifier was transiently unhealthy on one
side of it. The take is not to be cut into a single tidy sequence.

**The 18:00 presentation was fired before the mark, and it is load-bearing.**
`authorized_agent` at 18:00:08 and `revoked` at 18:03:05 — same certificate, same
path, same command, same verifier, with only his revocation in between.

**The crossing is 1.417s**, his send at 18:02:09.706Z to applied at
18:02:11.123Z. Two stamps from two systems, neither party holding both: he
supplied the send time, our adapter metadata holds the applied time. Yesterday's
equivalent was 2.145s.

⚠️ **Do not quote mark-to-applied.** His declared mark at 18:02:00Z to applied
is 11.1s, and that interval is mostly him getting from posting the mark to
pressing the key. The mark was posted in the thread AND spoken aloud on
their side; no audio was agreed for this session and ours has no track, but
theirs does, and they have now listened to it.

**What their audio carries:** the spoken anchor at the start, corroborating the
clock they posted in the thread, and the mark called at the revocation. It also
renders the 11.1s audibly — roughly six seconds of speaking the mark, a pause,
then the send.

⚠️ **It is NOT a third independent stamp on the crossing, and must not be cited
as one.** A transient at 18:02:09.7 lines up with their send stamp and was first
taken for the Enter press; on listening, keystrokes are not captured, and they
record from a coworking space with faint background throughout. Their own
correction, 08-19. It is a round trip with a human keystroke in it, not a revocation
latency, and it must not sit beside yesterday's ~4.7s as though they measure the
same thing. Agreed with Identities AI on 08-19.

Fail-closed verified independently of stdout: `AgentRelayBot/ratify-agent-relay-
engagement` carries no `ratify-run-2026-08-19-failclosed` branch, and no pull
request exists beyond #7.

## Video sync

`05-inset-clean.mov` (26s, no overlays) is the segment cut for the split-screen.

```
frame 0        18:02:58.08Z   (+/- 0.2s)
t=3.6          18:03:01.702   B3 command, stamp printed
t=7.3          18:03:05.395   B3 verdict — revoked
t=11.35        18:03:09.431   control command
t=14.75        18:03:12.831   control verdict — authorized_agent
end (t=26)     18:03:24.1Z
```

Frame 0 was derived from two independent stamps visible in the terminal, which
agree to 0.004s: `18:03:01.702` first appears between t=3.60 and t=3.65, and
`18:03:12.831` between t=14.6 and t=14.9. The residual uncertainty is the frame
interval, not the arithmetic — the source is ~2.5fps, so a frame can be drawn up
to ~0.4s after the text it shows, which makes 18:02:58.08 a lower bound.

**The clip runs 1:1 with wall time**: 11.129s separates those two stamps in
reality and ~11.25s in the clip. Idle time is held rather than compressed, so a
synchronised two-panel cut is possible. ⚠️ That is a property of *this* segment,
not of the source file — see the frame-rate warning below before cutting others.

## Findings

🔴 **A revoked presentation never reaches `ratify_last_presentation`.** In
`ratify_task`, an invalid inbound presentation returns through `reject(...)` at
`repo-2/src/worker.ts:773`, before the publish block at `:839` — whose own
comment is about refusals leaving no trace. So the deployment self-publishes its
accepts and its policy refusals but not the one verdict the demonstration turns
on, and a counterparty checking our account of the kill switch finds nothing.
Sixth of the shape: correct in the code, absent from everything that reaches the
other side.

Mitigated, not fixed: `--emit-grant` persisted signed receipts for the refusal
(`revoked`, "delegation certificate has been revoked", verifier `0c7afc4c…`), so
the refusal is verifiable against the verifier key rather than on our word.

🔴 **The publisher's closing line misdescribes a revocation as a constraint
refusal.** It printed "The delegation does not authorize a write to
docs/federation-note-2026-08-19.md" while both children immediately above read
`revoked — delegation certificate has been revoked`. Same family as
`stale_challenge` firing on a challenge that is too new: the verdict is right and
the sentence points the wrong way. It is also the last line on screen in the
footage.

✅ **The TTL clamp IS in evidence — in 08-18's artifacts, not today's.**
Corrected by Identities AI on 08-19 after this record first called it untested,
and verified here against the committed files rather than taken on trust. Every
child issued today requested 600s against a parent with hours left and got the
full 600s, which is why today's set shows nothing. Yesterday's set carries the
demonstration, with its own negative control:

```
work-43528b2bab594b4d  18:38:16 -> 18:48:16  life 600s  parent had 1194s left  unclamped
work-a925b80b3684b8e5  18:50:37 -> 18:58:10  life 453s  parent had  453s left  clamped
work-f9bb76e93710b6f6  18:50:39 -> 18:58:10  life 451s  parent had  451s left  clamped
```

Same issuance path and the same 600s request: one child received it in full, two
came back cut to `relay-run-1787078590`'s exact expiry instant. That is
`min(requested, parent_remaining)` choosing `parent_remaining`, with a control
showing it is not simply a fixed shorter default.

⚠️ Still to keep separate in the write-up: this bounds a child's window **at
issuance**. A parent cut short afterwards is a different mechanism — the verifier
checks each certificate's own window, and revocation is checked separately.
"A child cannot outlive its parent" describes neither accurately on its own.

## Artifacts

```
beat-served-*        receipt + decision, step 4 served half
beat-unserved-*      receipt + decision, step 4 unserved half
child-work-*         four gated-write children and two fail-closed children,
                     each with receipt and decision
```

## Open

- PR #7 to be merged by Identities AI
- publish the presentation-half refusal; fix the publisher's closing line
- mint a child that exercises the TTL clamp, if the write-up is to claim it
- a public repo to host the image artefact at publication time
- delete `federation-sender`; delete `rehearsal-fork-0817`, `seam-test-0817`,
  `ratify-run-2026-08-18`, `ratify-run-2026-08-19` from the bot fork
- step 3's reverse direction, whenever their image gains inbound A2A
