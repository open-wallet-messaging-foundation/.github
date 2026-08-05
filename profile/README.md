# Open Wallet Messaging

**Your bank cannot prove that a message came from it. You cannot revoke the
SSH key you issued in 2019. Your ticket can be screenshotted, your barcode
can be printed by anyone, and the code your exchange just texted you works
just as well for the person who tricked you into reading it aloud.**

These look like six problems for six industries. They are one problem: we
have no standard way to say *this signed thing, from this identity, means
this, for these people, until this moment* — and no standard way to take it
back.

OWM is an open standard that fixes that in the one place every person
already has a key: their wallet. Messages ride end-to-end encrypted on
XMTP/MLS; identities are wallet-held; and the useful part — the part this
page is actually about — is a small set of verbs that compose.

> **Pull requests are welcome — genuinely, including from people who think
> we are wrong.** Open an issue, send a patch, or claim one of the drafts
> looking for an editor. Typos and clarity fixes need no permission at all;
> [see how to contribute](CONTRIBUTING.md).

> **Support the work → [owm.foundation/#/donate](https://owm.foundation/#/donate)**
> The specification and the code are free forever. What costs money is
> independent security auditing, the pilots that prove this works, and the
> hands to finish it.

---

## Six questions, six verbs

Almost every protocol interaction decomposes into the same six questions.
Make the answers orthogonal and you stop building features; you start
composing them.

| Question | Verb | What it is | Status |
|---|---|---|---|
| Who may act? | **Grant** | scoped, expiring, revocable authority — including delegation to a person or an agent | **ships** (WM-7) |
| Who said what about whom? | **Attestation** | an issuer signs a typed claim about a subject, anchored to a domain or a key | draft (WM-13) |
| How do I prove it to you? | **Presentation** | a holder discloses or proves a signed object to a named audience, bound to a freshness challenge | draft (WM-14) |
| How do I hand it over? | **Transfer** | the holder signs the object to a new holder, under the issuer's policy | draft (WM-15) |
| Who can read it? | **Seal / Group** | encrypt to N recipients, or to a group where membership *is* access and removal *is* revocation | **ships** (WM-1, WM-8) |
| What is the audit trail? | **Log + fold** | append-only signed events; state is the fold, so tampering is visible | **ships** (WM-6, WM-10) |

Underneath them sits one atom — a **signed statement** with strict
validation — and one rule we do not bend: unknown message types degrade to
readable text and are never silently dropped.

**Status is not decoration.** Grant, seal, group, log+fold and the envelope
core ship today with tests and live network smokes. Attestation,
presentation and transfer are drafts written after we noticed we had
re-implemented each of them three times in different features. We would
rather publish that honestly than pretend the whole set is finished.

---

## The payoff: things we did not design for

If the six verbs are really orthogonal, then use cases nobody planned should
fall out as compositions. That is the claim, and here is the test — every
row below came from a real conversation with a real industry, and almost
none of them needed new machinery.

| You want to… | The composition |
|---|---|
| Prove a message really came from your bank | **attestation** (domain-anchored sender) |
| Make "we will never text you" provable, so smishing fails by definition rather than by detection | **grant** (exclusive channel) + **attestation** |
| Confirm a payment against the *true* payee, non-repudiably on both sides | **presentation** (what-you-see-is-what-you-sign) + settlement |
| Replace SMS codes and authenticator apps with something unphishable | **grant** + challenge-bound **presentation** |
| Let your accountant read statements but never move money | **grant** with scope and caveats |
| Send an AI agent to act for you — provably, and revocably | **grant** chain, principal-signed |
| Prove an AI inference happened without revealing the prompt or the answer | **attestation** on both legs, hash-bound |
| Sell a ticket only the buyer can use | **attestation** (paid) + **group** (decrypt) |
| Take a royalty on resale — or forbid it entirely | **transfer** under issuer policy |
| Cut off a leaked stream for one holder, instantly | **group** removal (forward secrecy) |
| Let a turnstile verify a ticket offline, and defeat the screenshot | **presentation**, challenge-bound or short-lived, in a QR |
| Bind a product serial to the brand that made it | **attestation** over a barcode subject |
| Register a product to its owner at the moment of purchase | binding + **attestation** |
| Carry warranty and repair history with the item instead of the receipt | **log + fold** keyed to the serial |
| Prove you are over 18 without showing your birthdate | **presentation** with selective disclosure |
| Issue an SSH principal that expires and can actually be revoked | **attestation** (the certificate) + **grant** (which hosts, how long) |
| Answer "who had access to that host, on that date?" — with proof | **log + fold** over grants |
| Approve a wire M-of-N, verifiable offline months later | **presentation** ×N + a quorum fold |
| Run a paywall with no account system | **attestation** (paid) + **group** |
| Encrypt a file so only one wallet opens it — then send it over WhatsApp | **seal**, any carrier |
| Retire CAPTCHAs without excluding blind users | **grant** + a proof-of-cost ladder |

We did not set out to build a ticketing system, a product-passport scheme,
or an SSH certificate authority. They fell out. That is the entire argument
for building this as a standard rather than a product.

---

## What this is *not*

The fastest way to lose a technical reader is to overclaim, so:

- **Not a video pipe.** For paid broadcast, OWM binds payment to
  decryption rights and distributes keys. A CDN moves the bytes. And no
  cryptography stops a paying viewer re-filming their own screen —
  watermarking makes that traceable, never impossible.
- **Not a replacement for OpenSSH.** OpenSSH already has hardware-backed
  keys and certificate authorities. WM-16 changes the trust root and the
  entitlement layer, and adds no new cryptographic core.
- **Not a substitute for a hardware token.** A wallet key in a secure
  enclave or on a hardware device is comparable to a security key. A hot
  key in a browser is not, and we will not pretend otherwise.
- **Not proof that atoms are genuine.** A signed serial proves the brand
  signed *that serial*. A counterfeiter can print the same code. Closing
  that gap needs a physically unclonable mark; we compose with the people
  who make them.
- **Not a tax on anything.** The foundation meters messages, never value
  moved. Self-hosting is first-class and always will be — if it ever
  becomes second-class, the neutrality that makes this adoptable by
  competitors is gone.

---

## Try it — 60 seconds, no wallet needed

```sh
npm install @open-wallet-messaging/core
```

**Strict validation, and the four outcomes.** Every inbound message
resolves to exactly one of: valid, ordinary text, an unknown kind, or a
known kind whose payload failed. Nothing is ever silently dropped —
that last distinction is the whole philosophy.

```js
import { buildPing, parseMessage } from '@open-wallet-messaging/core';

const ping = buildPing({ purpose: 'attention', ts: Date.now() });

parseMessage(JSON.stringify(ping)).ok;          // true    — valid, kind 'wm-ping'
parseMessage('just a chat message').plain;      // true    — ordinary text
parseMessage('{"_kind":"vnd.acme.x","v":1}').unknown;  // true — render it, never drop it

// Strict means strict: no extra keys, no loose types.
parseMessage(JSON.stringify({ ...ping, surprise: 'hi' })).error;  // 'extra key: surprise'
parseMessage(JSON.stringify({ ...ping, ts: 'yesterday' })).error; // 'type mismatch: ts'
```

**The rule the protocol enforces for you.** Invite tokens ride the URL
fragment, never the query string — because a query string reaches server
logs. Move the token and parsing refuses it:

```js
import { createInvite, buildInviteLink, parseInviteLink } from '@open-wallet-messaging/core';

const invite = createInvite({ roomId: 'room-1', now: Date.now(), ttlMs: 3600e3, maxUses: 5 });
const link = buildInviteLink({
  origin: 'https://example.org', roomId: 'room-1',
  adminInboxId: 'inbox-abc', name: 'Founders', mode: 'chat', token: invite.token,
});
// https://example.org/?room=room-1&admin=inbox-abc&name=Founders&m=chat#t=…

parseInviteLink(link).roomId;                    // 'room-1'
parseInviteLink(link.replace('#t=', '?t='));     // throws — a token in a query is refused
```

Both snippets are copy-paste runnable and produce exactly the output
shown. For the parts that need a wallet — secure contact exchange, signed
institutional senders, approvals, payments — see the runnable examples in
[`owm-ts`](https://github.com/open-wallet-messaging-foundation/owm-ts).

```sh
npm install @open-wallet-messaging/auth   # server-side: wallet 2FA, sign-in, grants
```

---

## Repositories

| Repo | What it holds |
|---|---|
| [`spec`](https://github.com/open-wallet-messaging-foundation/spec) | the standard: WM-0 … WM-16 |
| [`owm-api`](https://github.com/open-wallet-messaging-foundation/owm-api) | kind registry, OpenAPI, JSON Schemas, conformance vectors |
| [`owm-ts`](https://github.com/open-wallet-messaging-foundation/owm-ts) | TypeScript reference implementation + runnable examples |
| [`owm-react-native`](https://github.com/open-wallet-messaging-foundation/owm-react-native) | React Native packages |
| [`owm-relay`](https://github.com/open-wallet-messaging-foundation/owm-relay) | blind relay + rendezvous (Rust) |
| [`owm-notifier`](https://github.com/open-wallet-messaging-foundation/owm-notifier) | notification bridge (Rust) + C++ client |

Start with the spec if you want the argument, `owm-ts` if you want to run
something in the next five minutes.

---

## Contributing

**Yes, we want your pull request.** Issues are open on every repository and
patches are read by a human, usually within a day.

Some things need no permission whatsoever: typos, broken links, unclear
wording, better examples, a new conformance vector, a vendor message kind in
the 1000+ range. Send those straight as a PR.

Some things are specification decisions rather than edits — a new core
protocol kind, a change to what a conforming implementation must do. Those
start as an issue so the discussion happens before you spend an evening on
code. Each repository's `CONTRIBUTING.md` has the table telling you which
is which, precisely so you never write a patch that was never mergeable.

A conformance vector that catches a real bug is worth more to this project
than most code.

We especially want people who will attack this. Several drafts are looking
for editors — non-EVM chain rails in particular.

**Security:** report privately to security@owm.foundation. There is no
bounty yet; we are unfunded for that and say so rather than implying
otherwise.

---

**Not affiliated with the OpenWallet Foundation.** Open Wallet Messaging
(OWM) is not associated or affiliated in any way with the
[OpenWallet Foundation](https://openwallet.foundation), a Linux Foundation
project. Similar name, unrelated organisation.

**Governance positions are open to independents**, and academic scrutiny is
actively invited — see [owm.foundation/#/governance](https://owm.foundation/#/governance).

---

*Open Wallet Messaging (OWM) is an open standard, stewarded by the Open Wallet Messaging Foundation. Everything here follows one
house rule: state the ceiling. Where we do not know, we say so; where a
guarantee has limits, the limits are written next to the guarantee.*
