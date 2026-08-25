# Run record — Phase 2 flagship, 2026-08-18

Agent Relay side. Deployment `ratify.agentrelay.com`, image
`sha256:cacefe2bdcecebc5618e61da85e66cf54a869769b11ae3a1ef49435a6049d739`
(verified by Identities AI on 08-17), verifier `0c7afc4cb45cc3987cf1003f7957af5d`.
All times UTC.

## Go / no-go

`17:30:01Z` — `scripts/go-no-go.sh` → **GO 9/9**: workspace key 200, peer record
`bearer|SET|BorealisWorker`, `preflight-trust` armed on the running image, smoke
8/8, revocation-attribution 4/4, adversarial 6/6, a2a-cases 18/18, loopback 5/5,
agent token re-minted and verified on the heartbeat route.

## Step 5 — the gated write

Root delegation `relay-run-1787074501`, issued by Ratify Protocol's root
`345140967b9b99a16983cdfeb8acc807` to our contractor lead
`5a727611736ec8902a419e04fc91d816`, scope `files:write` + `identity:delegate`,
bound to `git:github.com/identities-ai/ratify-agent-relay-engagement` under
`/docs`, expiring `21:35:01Z`.

Checked before use against `trusted-issuers.json`: the certificate carries the
same ed25519 key we have established out of band for that issuer id.

The lead narrowed it for the write itself — `files:write` alone, no onward
delegation, same repository and prefix. That narrowed grant is what the verifier
answered on.

```
docs/handoff-note.md   authorized_agent   scope [files:write]
handoff-note.md        constraint_denied — requested path "/handoff-note.md"
                       is outside the authorized prefix "/docs"
```

The control ran **before any bytes were written**. Then:

```
committed  b1b430359dab  AgentRelayBot <311762608+AgentRelayBot@users.noreply.github.com>
pushed     AgentRelayBot:ratify-run-2026-08-18
PR         #5 -> identities-ai:main
```

Verified with a second credential (`khaliqgant`'s `gh`, not the bot token): PR
opened by `AgentRelayBot`, commit `author.login` **and** `committer.login` both
`AgentRelayBot`, upstream `main`'s `/docs` still holding `index.md` alone.

`lint` ran and passed — one check run, `success`, PR `mergeable_state: clean`.
PR #4 had had zero check runs because GitHub holds fork-PR workflows from
first-time contributors; merging it rather than closing it is what disarmed that.

## Kill switch — Procedure B

```
his mark    18:22:00Z  (spoken, as Enter was pressed)
applied     18:22:04.716Z   issuer 345140967b…  revoked ["relay-run-1787074501"]
                            applied_count 1  from ext-ratify-federation-run-03331b48
```

Read from adapter metadata (`ratify_last_revocation`), not container stdout —
stdout is unreadable on this deployment.

```
18:14:55Z  baseline  relay-run-1787074501  authorized_agent [files:write]   12009s left
18:22:17Z  B3        relay-run-1787074501  revoked  []  "delegation certificate
                     has been revoked"                                      11566s left
18:22:24Z  B4        relay-run-1787075619  authorized_agent [files:write]    1877s left
```

### Chain of custody — four stamps, two systems

Identities AI's side stamps the envelope independently, so the crossing is
bracketed at both ends by logs neither party controls alone:

```
signed      18:21:03        (theirs)
published   18:22:02.571    (theirs)
routed      18:22:03.318    (their engine log)
applied     18:22:04.716    (our adapter metadata)
```

**B3 cannot be read as expiry** — 11566s (3.2h) remained on the certificate.
**B4 makes it targeted rather than an outage** — the control differs from the run
certificate only in `cert_id`, `issued_at`, `expires_at` and `signature`; issuer,
issuer key, subject, subject key, scope and constraints are byte-identical.

⚠️ **The number is a bound, not a measurement.** ~4.7s from a spoken mark to
applied, against a 3s drain poll. Agreed with Identities AI to describe it as
*inside 5s, bounded by the poll interval*. Same caveat as the 1.296s and 2.53s
samples on 08-13, which were two ends of the same poll.

## Things in the record that look wrong and are not

- The verifier's trust set reads **three** issuers during the run: the two
  configured ones plus `fc1a08a5193d541f1d08000ab574bb38` marked `chain`. That is
  the ephemeral stand-in root left behind by the morning's `smoke` run — in
  memory, gone on the next deploy, and its private half died with the process
  that made it. Recorded here deliberately rather than left for someone to spot
  in the video.

- Identities AI's relaycast log repeats `provider not delivery-ready; delivery
  deferred` every 30s from `18:22:03` for `del_215535733371478016`. That is their
  **local agent inbox** not draining — not the DM to us, which plainly arrived,
  since we applied it 1.4s later. Named here for the same reason as `fc1a08a5…`:
  better in the record than spotted in the video.

## Fail-closed

`18:32:50Z` — the same driver, the same certificate and the same path as PR #5,
run again after the revocation:

```
asking the verifier about docs/handoff-note.md
  verdict  revoked — delegation certificate has been revoked  scope []
REFUSED — nothing was written, no fork, no commit, no pull request.
```

Confirmed by effect rather than by the tool's own report: no
`fail-closed-2026-08-18` branch on our fork, no pull request above #5 (so none
was opened and closed), upstream `main`'s `/docs` still `index.md` alone, and #5
still open and unmerged. The only difference between the two runs is the
delegation's state.

## The merge

`18:35:10Z` — PR #5 merged by `chuks`. `main` head `21dbaa55c000`, author
`AgentRelayBot`; `docs/handoff-note.md` now on `main` at 1307 bytes, alongside
`index.md`. Confirmed with `khaliqgant`'s credential, not the bot's.

## The narrowing hop, and what could not be quoted afterwards

Identities AI asked for the hop as two certificates with two scopes rather than
as prose, and the ask exposed a gap: the child certificate is minted in-process
by `subDelegate` and was never logged or persisted, and its `cert_id` is random
hex (`mintCertId('work')`). **So the child id from the PR #5 run is not
recoverable, and no re-minted one may be quoted in its place.**

Everything else about that child is deterministic and can be stated:

```
issuer       5a727611736ec8902a419e04fc91d816   (our contractor lead)
subject      caf08a851c78d77799bfc7bd2bc90f9d   (the adapter's worker agent)
scope        [files:write]        from parent [files:write, identity:delegate]
constraints  identical to the parent's — /docs on the engagement repo
ttl          min(600s, parent remaining)
```

**Fixed the same evening:** `ratify-publish --emit-grant <dir>` now prints the
child's id, issuer, subject, scope and parent scope, and writes the encoded
certificate. Demonstrated as `child-work-43528b2bab594b4d.json` — but note that
one was minted under the **now-revoked** root, so it documents the *shape* of the
hop, not the certificate that authorized the merged write. A child under a live
root needs one more short-TTL mint from Identities AI.

### The narrowing demonstration, under a live root

`18:50:40Z` — gate only, `--dry-run --emit-grant`, under a third certificate
Identities AI minted for the purpose: `relay-run-1787078590`, expiring
`18:58:10Z`. **Labelled by both sides as a narrowing demonstration, explicitly
not the certificate behind the merged write.**

```
parent   relay-run-1787078590   345140967b… -> 5a727611…   [files:write, identity:delegate]

child    work-a925b80b3684b8e5  5a727611… -> caf08a851c78d77799bfc7bd2bc90f9d
         scope [files:write]                        -> authorized_agent
child    work-f9bb76e93710b6f6  5a727611… -> caf08a851c78d77799bfc7bd2bc90f9d
         scope [files:write]  path outside /docs    -> constraint_denied
```

Both children expire `18:58:10Z` — the same instant as their parent, not
600s out. `subDelegate` clamps the child's TTL to the parent's remaining window,
so a child can never outlive the certificate it derives from.

Nothing was written: `DRY RUN — no fork, no commit, no pull request`.

## Adversarial annex

`18:35Z`, against the deployed verifier, re-run after the revocation rather than
quoted from the morning:

```
6/6    adversarial cases refused
18/18  serve-authority cases passed
```

Full output in `adversarial-annex.txt`. It includes Gate 3 — the paired-receipt
check — three ways: stripping the separately signed deployment decision refuses
rather than reading as an accept, a served decision lifted onto a refused receipt
is rejected, and `served=true` flipped without re-signing is rejected.

## Artifacts in this directory

| file | what it is |
|---|---|
| `delegation-live-2026-08-18.json` | the run certificate, their root |
| `delegation-control-2026-08-18.json` | the B4 control certificate |
| `adapter-metadata-post-revocation.json` | trust set + `ratify_last_revocation` as captured at 18:26Z |
| `pr-5.json`, `pr-5-commits.json` | PR #5 and its commit, as GitHub reports them |
| `handoff-note.md` | the bytes written into `/docs` |
| `adversarial-annex.txt` | the annex, 6/6 + 18/18, run after the revocation |
| `child-work-43528b2bab594b4d.json` | a child minted under the revoked root, kept only as the first `--emit-grant` output |
| `child-work-a925b80b3684b8e5.json` | the narrowing demonstration's child, live root, `authorized_agent` |
| `child-work-f9bb76e93710b6f6.json` | its control child, same scope, refused outside `/docs` |
| `delegation-narrowing-2026-08-18.json` | `relay-run-1787078590`, the root behind those two |
