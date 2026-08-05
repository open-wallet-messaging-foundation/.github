# Contributing to Open Wallet Messaging

This is the organisation-wide default. Several repositories carry their own
`CONTRIBUTING.md` with a change-authority table specific to what they hold —
[`spec`](https://github.com/open-wallet-messaging-foundation/spec),
[`owm-api`](https://github.com/open-wallet-messaging-foundation/owm-api) and
[`owm-ts`](https://github.com/open-wallet-messaging-foundation/owm-ts) do.
Where a repository has its own, that one wins.

## Where to start

- **Found a bug or an ambiguity?** Open an issue in the repository it
  affects. For the specification, an issue that ends *"so either the text or
  my reading is wrong, and here is the ambiguity"* is worth more than a
  patch.
- **Want to implement OWM?** Read the spec, then check your work against the
  conformance vectors in `owm-api`. If a vector is missing for something you
  had to guess at, that gap is itself a contribution — tell us.
- **Want to break it?** Please do. Attacks on the ceremonies, the
  authorization flows and the parsers are the most valuable contributions
  available. Report anything with teeth privately: see `SECURITY.md`.

## Who signs for a commit

Every commit in an Open Wallet Messaging repository is authored by an
**identified individual** who takes responsibility for it. Not an anonymous
handle, not a shared organisation mailbox, and not an AI.

Each commit carries a Developer Certificate of Origin sign-off with your
real name and a working address:

```
Signed-off-by: Your Name <you@example.com>
```

`git commit -s` adds it. By adding it you assert the
[Developer Certificate of Origin](https://developercertificate.org): that
you wrote the contribution, or otherwise have the right to submit it under
the repository's licence.

**On tooling and AI assistance:** use whatever helps you write good code —
compilers, linters, language models. Disclose it if you like; several of us
do, in the commit body. What is not negotiable is that a named human has
read the change, understood it, and is accountable for it. An assistant can
draft a commit; it cannot be responsible for one. In a project whose entire
value is that signatures mean something, authorship that means nothing would
be a strange place to start.

## The invariants

These hold across every repository. A change that weakens one will be closed
with a link here — not out of preciousness, but because each is load-bearing
for somebody's safety.

- **Strict validation.** Envelopes reject missing, extra and type-mismatched
  keys. Unknown *kinds* fall back to readable text and are never silently
  dropped — a different rule, equally important.
- **Invite tokens ride the URL fragment only.** A token in a query string
  reaches server logs. Parsing refuses it.
- **No key material anywhere it can leak** — not in logs, errors or
  telemetry, including debugging code you meant to remove.
- **State the ceiling.** If a guarantee has limits, say so next to the
  guarantee. "We have not tested this" is a perfectly good sentence; a claim
  that outruns the evidence is not.
- **Tests are mandatory** — happy path and unhappy path. Anything touching
  the wire needs a conformance vector too.

## Conduct

Be kind, argue about the work, state your ceilings. Full text in
`CODE_OF_CONDUCT.md`.
