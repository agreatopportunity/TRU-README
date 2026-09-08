# TRU Token Evolution / Verifiable Attribute History (VAH)

## Status

AI Provenance V1 is the currently enabled writer path. VAH-01 introduces the typed capability vocabulary that future historical writer authorization will use. **VAH-01A-E do not enable HUMAN, SENSOR, DEVICE, or SOFTWARE evolution writes and do not expand the existing V1 AI mutation allowlist. VAH-01D/E add signed authorization records and a historical verifier beneath that still-closed write boundary.**

The governing principle is simple: metadata may describe a capability, but TRU only treats a capability as real when code, authorization, signatures, and (where appropriate) external data actually enforce it.

## What Token Evolution does today

SFT and NCFT tokens can maintain a durable sequence of metadata evolution records. Each persisted epoch links to the previous metadata hash, records the AI request provenance, and is queued for an on-chain `TRU_TOKEN_EVOLVE_V1` anchor. The strict runtime verifier can require the issuance transaction and every anchor transaction to be confirmed and can validate the exact anchor payload.

AI Provenance V1 records use Evolution Record V2 fields including `record_format_version`, `writer_type`, provider/model identity, `request_hash`, `input_metadata_hash`, parent/new metadata hashes, epoch information, the updated fields, and the resulting metadata.

The provenance chain proves what was recorded and when. It does **not** establish that an AI-generated claim, a human assertion, or a sensor measurement is objectively true.

## VAH: Verifiable Attribute History

VAH generalizes token evolution into typed, historically authorized attribute writers. The intended writer classes are:

- `AI` — model-generated descriptive or creative attributes.
- `HUMAN` — creator/operator/certifier assertions.
- `SENSOR` — measurements supplied by an authenticated sensor/oracle path.
- `DEVICE` — authenticated device state and device-originated signals.
- `SOFTWARE` — application-managed references and state produced by an authorized integration.

A future VAH writer record is only valid when its signature is valid, the writer was authorized for that token at that historical epoch, the writer class matches the authorization, and every changed field falls inside the writer's authorized typed capabilities.

A later revocation must not retroactively invalidate an older record that was valid at the time it was written.

## Typed capability classes

### `AI_DESCRIPTIVE`

Descriptive/adaptive AI attributes. The catalog includes `description_ai`, `ai_version`, `learning_mode`, `growth_algorithm`, `adaptation_rate`, and staged `behavior_model`.

### `AI_CREATIVE`

AI-generated presentation/creative evolution. The catalog includes `style_descriptor`, `dynamic_morph`, `update_interval`, and staged `neural_art_evolution`, `dynamic_narrative_link`, and `generative_seed`.

### `SENSOR_MEASUREMENT`

Authenticated measurements or measurement references. Staged fields include `sentiment_score`, `emotional_resonance`, `bio_sensor_trigger`, `emotion_response`, `synaptic_pattern_id`, and `biometric_reference`.

These fields require a real sensor/oracle/data path before they should be presented as live measurements.

### `DEVICE_STATE`

Authenticated device/software state. Staged fields include `operating_state`, `context_signal`, and `environmental_signal`.

### `HUMAN_ASSERTION`

Human-authored statements or references. Staged fields include `privacy_level`, `context_aware_rule`, `certification_reference`, and `creator_attribution`.

### `APPLICATION_LINK`

References to external or TRU applications. Staged fields include `defi_insurance_pool`, `atomic_swap_sft_bundle`, `defi_art_loan_value`, `atomic_swap_ncft_chain`, `virtual_gallery_space`, and `cross_platform_avatar`.

An application-link field does not itself create a swap, insurance pool, loan, or other protocol. The referenced application/protocol must enforce the actual behavior.

## Current V1 writable field matrix

VAH-01ABC deliberately preserves the exact previously enabled AI allowlist.

### SFT + AI

- `description_ai`
- `learning_mode`
- `growth_algorithm`
- `adaptation_rate`
- `ai_version`

### NCFT + AI

- `description_ai`
- `style_descriptor`
- `dynamic_morph`
- `update_interval`

All newly cataloged fields are **registered but inactive** until a later VAH phase explicitly enables their writer path and authorization rules.

## Fields that must not become ordinary mutable VAH metadata

Some names from earlier experimental token designs represent authority or protocol state and therefore must not become free-form writer fields:

- `evolution_epoch` — owned by the evolution engine.
- `creator_signature` — should be an actual cryptographic signature/provenance property, not a decorative string.
- `self_evolution` — should be an authorization/policy decision, not a metadata permission bit.
- ownership, balance, supply, mint/burn authority, spend conditions, prices, consensus state — remain protocol/application controlled.

Likewise, names such as `telepathic_interface`, `interdimensional_id`, `multiverse_presence`, `quantum_art_resonance`, or similar speculative concepts may be stored as creative identifiers if explicitly designed for that purpose, but TRU must not imply that the chain verifies a physical/scientific capability merely because a string is present.

## VAH signed writer authorization model (VAH-01D/E)

The authorization root is the token owner/control path, not the writer itself. A writer must never be able to grant itself authority.

The canonical `TRU_VAH_AUTH_V1` authorization record now binds:

```text
format_version = 1
Token ID = canonical lowercase 16-hex
sequence
Effective epoch
Action = AUTHORIZE / ROTATE / REVOKE
Writer compressed secp256k1 public key
Canonical writer ID derived from that key
Writer class
Sorted typed capabilities
ROTATE replacement writer ID (when applicable)
Previous authorization-record hash
Authorization-root compressed public key
Record hash
Strict-DER low-S authorization-root signature
```

The historical verifier will evaluate the authorization state that existed at the record epoch rather than only consulting the latest registry state.



### Authorization trust boundary

The authorization-root public key contained in a record is **not self-trusting**. Verification requires the caller to supply the independently trusted authorization-root public key for the token. A record signed by an attacker-controlled key fails even if that same attacker key is written into the record as `authorization_root_pubkey_hex`.

`writer_id` is also not independently trusted. It is deterministically derived from the canonical 33-byte compressed writer public key using the `TRU_VAH_WRITER_ID_V1` domain and SHA-256.

### Canonical authorization digest

The signed preimage uses the `TRU_VAH_AUTH_V1` domain, fixed-width big-endian integers, and length-prefixed strings. The capabilities vector must be canonical, sorted, unique, and known. `record_hash` is the SHA-256 digest of this unsigned canonical record. The authorization root signs those exact 32 digest bytes with secp256k1 ECDSA; signatures must be strict DER and low-S.

### AUTHORIZE / ROTATE / REVOKE semantics

`AUTHORIZE` activates a previously inactive writer with one or more capabilities. `ROTATE` atomically removes an active old writer ID and activates a distinct replacement key. Rotation cannot silently change writer class; use a root-signed revoke plus authorize when changing roles. `REVOKE` removes an active writer and carries no capabilities.

Authorization records form their own hash chain. Sequence starts at 1, each record commits the previous record hash, and effective epochs may not move backwards. Broken links, reordered/gapped sequences, invalid signatures, unknown actions, duplicate active authorizations, invalid rotations, unknown capabilities, and token/writer capability mismatches fail closed.

### Historical authorization verification

A later revocation does not rewrite earlier validity. To verify an attribute record at epoch `E`, VAH verifies the complete signed authorization history and replays only authorization records with `effective_epoch <= E`. The writer must be active at `E`, the writer class must match, and every changed field must map to a capability held by that writer at `E`.

Example:

```text
Epoch 5   AUTHORIZE sensor key K1 : SENSOR_MEASUREMENT
Epoch 10  ROTATE K1 -> K2
Epoch 20  REVOKE K2

Epoch 9 record signed by K1   => historically authorized
Epoch 10 record signed by K1  => rejected
Epoch 10 record signed by K2  => authorized
Epoch 19 record signed by K2  => authorized
Epoch 20 record signed by K2  => rejected
```

### What VAH-01D/E still does not do

VAH-01D/E deliberately provides the cryptographic record format and historical verifier **without enabling external writers to originate token-evolution epochs**. Persistence/RPC/wallet management and the canonical multi-node ordering boundary remain later activation work. AI Provenance V1 remains the only live evolution writer path until those gates are completed.

## Why historical authorization matters

Suppose sensor key `K1` is authorized for epochs 5 through 20 and revoked before epoch 21. A valid epoch-12 measurement signed by `K1` remains historically valid. An epoch-21 record signed by `K1` must fail. This is stronger than checking only whether a key is authorized *now*.

## Multi-node ordering rule

External writer classes are not enabled by VAH-01ABC. Before arbitrary external writers are allowed to originate evolution epochs, TRU must define canonical multi-node reconciliation for competing proposals to the same next epoch. Local LevelDB sequencing alone is not sufficient to make independently produced epoch-N records globally canonical.

## Using AI Token Evolution V1

From the advanced wallet, open `TOKEN/AI TOOLS`, select `AI Evolution`, select a confirmed SFT or NCFT, choose the AI provider/trigger, and generate a preview. The preview is non-persistent. Review the token ID, type, provider, epoch transition, parent hash, new hash, and proposed fields. Only the exact literal confirmation `COMMIT` is accepted by the hardened commit path.

After commit, the evolution record is atomically persisted with the latest pointer and anchor queue. The anchor worker prepares and signs the exact anchor transaction, submits it, watches/rebroadcasts as necessary, and records the durable receipt. Confirmation is not claimed merely because the record was persisted.

## Strict verification from `tru-cli`

`verifytokenevolution` is an RPC method, so call it through the raw command:

```bash
cd ~/NEW_TRU/build-native/bin

./tru-cli -json raw verifytokenevolution \
'{"tokenID":"<16-hex-token-id>","require_confirmed":true}'
```

For a fully confirmed history, the important top-level result is:

```json
{
  "runtime_ok": true,
  "history_ok": true,
  "fully_anchored": true,
  "all_anchor_payloads_valid": true,
  "all_anchor_txs_observable": true,
  "all_anchor_txs_confirmed": true
}
```

`runtime_ok=true` with `require_confirmed=true` is the strict native runtime gate. Inspect each anchor entry as well; a confirmed anchor should report `chain_status: "CONFIRMED"` and `payload_valid: true`.

## Anchor signer configuration

The anchor worker reads its configured oracle signer and wallet. The configured `oracle.address` must be owned by the configured `oracle.wallet`. A placeholder or unrelated address prevents signing and leaves the evolution pending/queued rather than falsely reporting confirmation.

Keep RPC private to trusted/local interfaces. P2P and RPC are different services; do not expose RPC merely to make the evolution worker function.

## Security rules

1. AI output is untrusted input.
2. Preview must not persist state.
3. Commit must verify freshness and the exact preview boundary.
4. Epoch, ownership, authorization, balances, supply, and consensus fields are not AI-controlled metadata.
5. All mutable fields pass a typed field policy.
6. Unknown writer classes, unknown fields, invalid record versions, broken hash links, malformed queue state, or invalid anchor payloads fail closed.
7. External writer classes require signed historical authorization before activation.
8. Writer authorization is granted by the token authorization root; a writer cannot self-authorize.
9. Revocation affects future validity and does not rewrite valid history.
10. Provenance proves recorded history, not the truth of the underlying claim.

## VAH implementation sequence

- **VAH-01A** — canonical writer identity and writer classes.
- **VAH-01B** — typed capability registry.
- **VAH-01C** — token-type / writer / field policy matrix; V1 AI allowlist routed through it.
- **VAH-01D** — signed AUTHORIZE / ROTATE / REVOKE records. **Implemented foundation.**
- **VAH-01E** — historical authorization verifier. **Implemented foundation.**
- **VAH-01F** — adversarial authorization tests.
- **VAH-02** — canonical multi-node epoch reconciliation before external writers originate epochs.
- **VAH-03** — controlled activation of HUMAN / SENSOR / DEVICE / SOFTWARE writer paths.

## Development invariant

Do not reset the chain or delete `build-native/bin/data` as part of VAH development. New VAH phases must preserve the confirmed AI Provenance V1 history and fail closed when new authorization data is absent or malformed.


## VAH-01F — Durable Authorization Persistence Boundary

VAH authorization history is persisted under a dedicated `TOKEN:VAH_AUTH` contract-storage namespace. Each append writes the exact signed authorization record and its committed head in one synced LevelDB `WriteBatch`. The durable head is `TRU_VAH_AUTH_HEAD_V1|<sequence>|<record_hash>`.

The persistence reader does not trust the root key contained in stored records. Callers must supply the independently trusted authorization-root public key. A stored history is accepted only when every record parses canonically, the sequence and previous-hash chain are exact, the final record hash matches the durable head, no record exists beyond the committed head, and the complete signed authorization history verifies.

Exact retries are idempotent. If a caller receives an ambiguous failure after the batch became durable, retrying the identical signed record succeeds as `already persisted`; a conflicting record at the same sequence fails closed. This is important for crash/restart recovery.

VAH-01F still does **not** activate HUMAN, SENSOR, DEVICE, or SOFTWARE token-evolution writers. It establishes the durable authorization boundary required before VAH-02 canonical multi-node epoch reconciliation and later external-writer activation.


## VAH-02 — Multi-node canonical epoch reconciliation

### VAH-02A: deterministic canonical epoch election

VAH-02A introduces the pure deterministic election rule used when multiple
nodes observe competing confirmed claims for the same token epoch.

A candidate is eligible only when all of the following are true:

- token ID is the exact canonical 16-lowercase-hex ID;
- epoch is the expected next epoch;
- `previous_metadata_hash` equals the already canonical parent;
- the anchor is confirmed in the active chain;
- the anchor block is at or below the caller's finalized active-chain prefix;
- candidate hashes, block hash and txid are canonical lowercase 64-hex values.

Eligible competing candidates are ordered by this exact tuple:

1. active-chain block height;
2. transaction index inside that block;
3. anchor txid;
4. new metadata hash;
5. durable record hash.

The first tuple is canonical. Arrival order, wall-clock time, peer identity,
local process scheduling and which node authored the candidate are not inputs.

This rule gives two nodes the same answer once they possess the same candidate
set and the same finalized active-chain prefix. Nodes at different chain
prefixes may temporarily have different knowledge; VAH-02B/02C must therefore
supply bounded candidate transport plus active-chain observation/reconciliation
before external writers are activated.

### Important V1 anchor limitation

The existing `TRU_EVOLVE_V1` OP_RETURN commits token ID, token type, epoch,
provider, trigger, previous metadata hash and new metadata hash. It does not
commit the complete durable evolution-record hash. Therefore VAH-02A treats
`new_metadata_hash` plus the confirmed anchor location as the chain-visible
state claim, while `record_hash` remains an exact off-chain transport and
persistence identity.

External-writer activation MUST NOT occur until a later VAH-02 phase defines
and verifies a stronger anchor envelope that commits the exact record identity,
or otherwise proves an equivalent complete record binding.

### VAH-02A safety boundary

VAH-02A is election logic only. It does not:

- add a new P2P message;
- accept remote VAH records;
- mutate `TOKEN_EVOLUTION`;
- change the current AI V1 writer;
- activate HUMAN/SENSOR/DEVICE/SOFTWARE writers;
- alter block validity, fork selection, or reorg policy.

VAH-02B should add bounded candidate transport and exact active-chain anchor
observation. VAH-02C should add durable reconciliation state and restart proof.
Only after the whole VAH-02 gate is closed should VAH-03 consider external
writer activation.

### VAH-02B: bounded transport and confirmed active-chain observation

VAH-02B defines a fixed-size canonical `TRU_VAH_CANDIDATE_V1` binary envelope
for candidate exchange and a strict parser for the existing eight-push
`TRU_EVOLVE_V1` OP_RETURN anchor. Transport decoding proves only canonical
structure; it never proves confirmation or makes a peer claim authoritative.

Canonical observations are independently re-derived from the node's local
active chain. The observer scans a bounded, ascending, parent-linked block
range ending exactly at the caller's finalized view, recomputes each stored
transaction ID in the production adapter, and obtains block height,
transaction index, txid and block hash from that local chain snapshot.
Unconfirmed mempool entries and peer-supplied location claims are not inputs.

The V1 anchor limitation remains explicit: because `TRU_EVOLVE_V1` does not
commit the durable record hash, a chain-derived V1 candidate uses the reserved
all-zero record-hash sentinel. A transported off-chain record hash must not be
silently promoted into chain-committed identity. External-writer activation
still requires a stronger anchor binding in a later VAH-02 phase.

VAH-02B does not add a P2P message handler, mutate `TOKEN_EVOLUTION`, persist a
candidate pool, quarantine a losing record, rebuild `latest`, change block
validity/fork selection/reorg policy, or activate HUMAN/SENSOR/DEVICE/SOFTWARE
writers. Durable reconciliation and restart proof remain VAH-02C.

## VAH-02C — Durable reconciliation + restart proof

VAH-02C adds an append-only durable `TOKEN:VAH_RECON` checkpoint for a canonical
per-token epoch election. The persisted winner is checksum-wrapped by the existing
LevelDB contract storage path, read back immediately, revalidated on restart, and
refuses conflicting rewrites for an already reconciled token/epoch. Exact replay is
idempotent. This stage does not accept VAH candidates over P2P, does not mutate
`TOKEN_EVOLUTION` latest/history, does not activate external writers, and does not
change block validity, fork/reorg policy, or chain state.

## VAH-02C.1 — Strict reconciliation checksum boundary

VAH-02C restart testing exposed that `LevelDBStorage::getContract()` still accepted
legacy single-SHA256 contract envelopes and upgraded them on read. That migration
behavior remains available for older contract namespaces, but it is now forbidden
for `TOKEN:VAH_RECON:*`, which is a new namespace with no valid legacy records.
A checksum mismatch there fails closed and is never rewritten. The VAH-02C matrix
now covers both malformed corruption and a parseable one-byte hash-field mutation,
verifies that the raw tampered bytes are unchanged after the failed read, and keeps
a non-VAH legacy-migration compatibility control.


## VAH-02D — Strong record binding + reconciliation closure

VAH-02D closes the record-identity gap called out in VAH-02A/02B without
activating external writers. It defines a dormant canonical `TRU_EVOLVE_V2`
OP_RETURN envelope containing the V1 fields plus the exact lowercase 64-hex
`record_hash`. A V2 record hash may not be the all-zero sentinel.

The read-only active-chain observer independently parses both versions. V1
continues to derive the reserved all-zero uncommitted record hash for backward
compatibility. V2 derives the exact record hash from the confirmed local-chain
anchor. Transport remains non-authoritative: a peer-supplied candidate never
proves chain inclusion by itself.

VAH-02D adds a closed-election gate that rejects V1/uncommitted candidates
before applying the unchanged deterministic VAH-02A ordering. It also verifies
that a locally observed winner commits the exact expected durable record hash.
The closure matrix proves the full boundary: V2 local observation -> strongly
bound election -> durable `TOKEN:VAH_RECON` checkpoint -> LevelDB close/reopen
-> identical winner and record binding.

This phase still does **not** emit V2 anchors from the production token-evolution
writer, add P2P VAH candidate acceptance, mutate `TOKEN_EVOLUTION`, activate
HUMAN/SENSOR/DEVICE/SOFTWARE writers, alter fork/reorg policy, or reset chain
state. Those remain explicit later activation work under VAH-03.

---

## VAH-03A — Controlled External-Writer Activation Gate

Status: **implemented as a dormant production verifier + DEV matrix; no external ingress yet.**

VAH-03A is the first phase of the roadmap's human / sensor / device writer layer.
It introduces a canonical signed `TRU_VAH_EXTERNAL_WRITER_CLAIM_V1` identity without
allowing that claim to mutate token state or arrive from the network.

Admission requires all of the following:

- canonical token ID / token type / epoch / parent and new metadata hashes,
- writer class exactly `human`, `sensor`, or `device`,
- writer ID derived from the compressed secp256k1 writer public key,
- bounded, sorted, unique changed-field names,
- every changed field structurally compatible with the token type and writer class,
- a complete valid signed authorization history under an independently trusted root,
- the writer active at the claim epoch with every required field capability,
- a canonical non-zero `record_hash` equal to the claim digest, and
- a strict-DER low-S writer signature over that exact digest.

The resulting `record_hash` is designed to be used by the strongly-bound
`TRU_EVOLVE_V2` anchor defined in VAH-02D.

VAH-03A deliberately does **not** add:

- RPC external-writer submission,
- P2P external-writer claim acceptance or relay,
- durable pending-claim storage,
- `TRU_EVOLVE_V2` production writer emission,
- canonical reconciliation materialization into `TOKEN_EVOLUTION`, or
- chain/fork/reorg policy changes.

Those activation steps remain gated behind later VAH-03 phases.

## VAH-03B — Bounded Local Claim Intake + Durable Staging

VAH-03B adds an append-only local pending-claim boundary for claims that have
already passed the VAH-03A signed HUMAN/SENSOR/DEVICE admission gate.

Durable namespace:

- `TOKEN:VAH_PENDING:<token>:epoch:<20-digit-epoch>:claim:<record_hash>`
- strict double-SHA256 contract checksum only; no legacy upgrade-on-read
- maximum 32 staged claims per token/epoch
- maximum 4096-byte canonical durable claim envelope
- exact replay is idempotent
- restart loading re-verifies checksum, canonical envelope, key/record binding,
  writer signature, historical authorization, and typed field capability
- simultaneous local intake is serialized so the resource cap cannot be raced

VAH-03B still does **not** expose RPC or P2P external-writer ingress, emit a
production `TRU_EVOLVE_V2` transaction, modify `TOKEN_EVOLUTION`, change
fork/reorg policy, or reset chain state. Those remain later VAH-03 gates.

## VAH-03C — Production V2 Anchor Emission + Confirmed-Chain Admission

VAH-03C connects the already-verified/durably-staged external writer claim to
TRU_EVOLVE_V2 without yet materializing external-writer state into
TOKEN_EVOLUTION.

A production internal emission primitive may broadcast a transaction only when:

- the claim still passes VAH-03A historical authorization/capability checks;
- the exact claim exists in VAH-03B `TOKEN:VAH_PENDING` durable staging;
- the transaction is non-coinbase, funded/signed by its caller, and has a
  materialized txid matching canonical serialization;
- it contains exactly one evolution anchor;
- that anchor is the zero-value canonical `TRU_EVOLVE_V2` envelope for the
  staged claim's token/type/epoch/parent/new hash/non-zero record hash.

Confirmed-chain admission is stricter than simple inclusion. The local active
chain is observed through the VAH-02B adapter and the VAH-02D strong election is
run for the token/epoch/parent. The exact submitted txid and staged record hash
must be the canonical winner. An earlier competing strongly-bound claim causes
this claim to fail closed rather than mutate token state.

VAH-03C does not expose RPC/P2P external-writer ingress, delete pending claims,
or write TOKEN_EVOLUTION latest/history. Those remain later-phase operations.

Next: VAH-03D canonical external-writer materialization into TOKEN_EVOLUTION +
restart/replay/conflict closeout.

## VAH-03D — Canonical External-Writer Materialization + Restart/Replay/Conflict Closeout

VAH-03D closes the HUMAN/SENSOR/DEVICE writer path from a strongly confirmed
VAH-03C chain winner into the existing `TOKEN_EVOLUTION` epoch/history/latest
state. It does not infer metadata from a hash: the caller must supply the exact
metadata JSON document whose canonical dump hashes to the already-anchored
`new_metadata_hash`.

Materialization rechecks the VAH-03A signed authorization/capability gate and
requires the exact claim to remain present in VAH-03B append-only staging. The
production wrapper independently reruns VAH-03C confirmed-active-chain
admission for the supplied funded/signed prepared transaction before any
TOKEN_EVOLUTION write occurs.

External durable evolution records use `record_format_version=3` with
`status=materialized`. The record stores the external writer identity,
the exact signed claim hash, changed fields and values, the full committed
metadata document, and the canonical winning chain position. Signature bytes
remain in append-only VAH-03B staging and the local finalized view remains in
the strict checkpoint so neither non-unique signatures nor later view height
can make canonical TOKEN_EVOLUTION bytes diverge across nodes. The V2
anchor `record_hash` remains the signed external-claim hash; it is intentionally
separate from the SHA256 of the serialized TOKEN_EVOLUTION V3 record.

The actual metadata delta is recomputed against the canonical parent document.
Exactly the signed `changed_fields` plus the system `evolution_epoch` advance
may differ. Deletions, hidden/unclaimed changes, non-string external field
values, wrong parent/new hashes, and capability-incompatible fields fail closed.
For cross-writer continuity, the parent must already be canonical under the
existing metadata normalization and, after epoch 1, the prior TOKEN_EVOLUTION
history must verify as fully anchored.

The synced atomic batch writes the new epoch record, `latest:<token>`, exact
`anchor_tx`, durable submitted receipt, and a strict append-only companion
checkpoint. If the global `anchor_queue` does not yet exist, an empty canonical
queue is created in the same batch because the historical verifier requires it;
an existing unrelated queue is never rewritten by materialization:

- `TOKEN:VAH_MATERIALIZED:<token>:epoch:<20-digit-epoch>`

That checkpoint commits the signed claim hash, new/parent metadata hashes,
canonical anchor tx/block position, finalized view, SHA256 of the durable V3
TOKEN_EVOLUTION record, and receipt hash. The namespace has
no legacy checksum format; checksum mismatch is corruption and is never
upgrade-on-read.

Exact replay is idempotent. A conflicting same-token/epoch materialization is
refused. An ambiguous caller failure after the synced batch is durable converges
to exact success after close/reopen. Replay of new V3 TOKEN_EVOLUTION state uses
a strict raw canonical-checksum read so a generic legacy-checksum write cannot
be laundered by the older TOKEN_EVOLUTION migration path.

VAH-03D deliberately keeps `TOKEN:VAH_PENDING` append-only as audit evidence.
It does not expose RPC or P2P external-writer ingress, run an automatic wallet
worker, change block/fork/reorg policy, or reset chain state.

With VAH-03A/03B/03C/03D complete, the VAH-03 HUMAN/SENSOR/DEVICE writer
foundation is closed. Next: VAH-04A canonical sensor-batch envelope + Merkle
commitments.

## VAH-04A — Canonical Sensor-Batch Envelope + Merkle Commitments

VAH-04A begins the high-frequency sensor/event-feed layer without turning the
blockchain into a raw telemetry database. A batch contains only bounded event
commitments; raw sensor payloads remain outside the envelope and are represented
by non-zero canonical `payload_hash` values.

`TRU_VAH_SENSOR_BATCH_V1` binds:

- canonical token ID/type and evolution epoch,
- a chained `previous_batch_hash` (`00..00` is structurally allowed only for a
  genesis predecessor; durable continuity is VAH-04B),
- one compressed secp256k1 SENSOR writer identity,
- 1..256 events in canonical vector order,
- contiguous non-zero event sequence numbers,
- non-zero nondecreasing observation timestamps,
- typed SENSOR-capable field names and non-zero payload hashes,
- a domain-separated Merkle root, event count, first/last sequence, and
- a canonical batch hash plus strict-DER low-S sensor signature.

Each Merkle leaf commits the token/type/epoch, previous batch hash, writer ID,
event count, sequence, timestamp, field, and payload hash. Internal Merkle nodes use a
separate domain and odd-width levels duplicate the final node. The signed batch
digest separately commits event count and sequence range, eliminating ambiguity
from duplicate-last tree construction. Inclusion proofs are verified with an
explicit leaf index and event count and cannot be transplanted across another
token, epoch, writer, or previous-batch branch.

Batch verification replays the complete signed VAH authorization history at the
batch epoch and requires the sensor to hold the capability for every unique
field represented by the events.

VAH-04A is pure/non-persistent. It does **not** add a durable accumulator,
sequence head, RPC/P2P sensor ingress, V2 anchor emission, TOKEN_EVOLUTION
materialization, automatic wallet spending, fork/reorg changes, or chain reset.

Next: VAH-04B local batch accumulator + sequence/replay/resource bounds.

## VAH-04B — Local Batch Accumulator + Sequence/Replay/Resource Bounds

VAH-04B persists only sensor batches that already pass the complete VAH-04A
cryptographic, Merkle, historical-authorization, and typed-capability verifier.
The accumulator is local and restart-safe; it still exposes no RPC/P2P sensor
ingress and does not emit an on-chain batch anchor.

Durable namespaces are strict double-SHA256 contract envelopes with no legacy
upgrade-on-read path:

- `TOKEN:VAH_SENSOR_BATCH:<token>:writer:<writer>:first:<20-digit-sequence>:batch:<batch_hash>`
- `TOKEN:VAH_SENSOR_HEAD:<token>:writer:<writer>`

Each append is one synced LevelDB atomic batch containing the append-only sensor
batch record and the mutable per-token/per-sensor feed head. The head tracks the
batch count, latest token epoch, latest batch first/last sequence, last observed
timestamp, writer public key, and latest batch hash. The persisted batch stores
its feed index and predecessor's first sequence so restart validation can exact-
key load both the latest batch and its immediate predecessor without an unbounded
namespace scan.

Feed continuity spans token epochs. The first batch must use the zero predecessor
and begin event sequence 1. Later batches must use exactly the durable
`last_batch_hash`, begin at `last_sequence + 1`, never regress timestamps, keep
the same sensor identity, and may keep or advance the token epoch but never move
it backward. A competing batch from the same durable predecessor fails once one
branch advances the head. Process-local concurrent appends are serialized.

Exact semantic replay is idempotent even after later batches exist. ECDSA
signature bytes are not part of replay identity because a valid signer may
produce different strict-low-S signatures over the same canonical unsigned
batch; the signed `batch_hash`, writer identity, events, Merkle root and feed
context define replay identity. The originally stored signature remains durable
and is re-verified on recovery.

Resource use remains bounded per operation: VAH-04A retains the 256-event hard
limit, a durable accumulated-batch envelope is capped at 128 KiB, the feed head
is capped at 2 KiB, sequence/counter overflow is fail-closed, and normal append
or restart-head recovery performs exact-key reads rather than an unbounded
feed-history scan.

VAH-04B does **not** add RPC/P2P sensor-batch ingress, sensor-batch anchor
emission, TOKEN_EVOLUTION event materialization, automatic wallet spending,
fork/reorg policy changes, batch deletion, or chain reset.

Next: VAH-04C authorized sensor-batch anchor emission + active-chain confirmation.

## VAH-04C.1 — sensor-anchor crash-safety + txid API hardening

VAH-04C already failed closed on a blank or stale caller txid by recomputing the
txid on a temporary transaction and comparing it with the supplied transaction.
VAH-04C.1 therefore does **not** classify that behavior as a live 02B3-style
defect. Instead it removes the caller-populated-txid precondition: the exact
canonical mutable transaction object passed to `Blockchain::broadcastTransaction()`
has `computeTxId()` called on that same object. A non-empty conflicting caller
txid is still rejected.

The substantive 04C.1 change is crash safety. The exact signed transaction
serialization is atomically persisted with strict
`TOKEN:VAH_SENSOR_ANCHOR_PREPARED:*` and `TOKEN:VAH_SENSOR_ANCHOR_WATCH:*`
records before broadcast. Restart recovery deserializes that exact transaction,
recomputes/materializes the same txid on the recovered object, verifies the
canonical `TRU_SENSOR_BATCH_V1` output again, and rebroadcasts without funding or
re-signing. Failed broadcast leaves the durable watch intact, and an existing
checksum-invalid prepared/watch record is never overwritten as though missing.

Public bool/reason verification paths contain canonicalization exceptions and
return fail-closed reasons. Canonical hexadecimal/decimal parsing uses explicit
ASCII ranges. `TRU_SENSOR_BATCH_V1` fields are currently capped below 76 bytes,
so canonical V1 encoding never emits `OP_PUSHDATA1`; parser support remains only
as defensive future support and a non-canonical forced-PUSHDATA1 encoding is
rejected by the DEV matrix.

`TOKEN:VAH_AUTH:*` is strict double-SHA256 from this patch forward. Automatic
legacy single-SHA migration is **forbidden** for authorization records because it
would preserve the checksum-laundering class closed for the new VAH namespaces.
If genuine historical legacy-format authorization data is ever discovered, it
requires an explicit offline migration after cryptographic verification. Non-VAH
legacy checksum compatibility remains unchanged.

This patch still adds no RPC/P2P sensor ingress, automatic anchor worker, wallet
spending, TOKEN_EVOLUTION sensor materialization, batch deletion, reorg policy,
or chain reset behavior.

## VAH-04D — Confirmed Sensor-Batch Recovery + Event/Checkpoint Materialization

VAH-04D closes the sensor batching architecture after 04C.1. A finalized active-chain `TRU_SENSOR_BATCH_V1` proof may materialize only the exact strict-checksum VAH-04B batch it commits to. Materialization writes a strict confirmation receipt, append-only per-sequence event records, and the per-feed checkpoint atomically. Historical exact replay is idempotent; advancing a checkpoint requires exact sequence and previous-batch-hash continuity.

The central disagreement rule is fail-closed: if the chain proves an anchor but the local durable batch is missing, checksum-invalid, malformed, unauthorized, or otherwise unreadable, the node persists `TRU_VAH_SENSOR_RECOVERY_REQUIRED_V1`, preserves prepared/watch and chain evidence, and performs no event/checkpoint materialization, no batch repair, and no automatic re-anchor. If a previously expected confirmation is absent from a later supplied canonical view, the runtime reports recovery-required and automatic re-anchor remains forbidden.

New strict namespaces: `TOKEN:VAH_SENSOR_CONFIRMED:`, `TOKEN:VAH_SENSOR_RECOVERY:`, `TOKEN:VAH_SENSOR_EVENT:`, and `TOKEN:VAH_SENSOR_CHECKPOINT:`. `TOKEN:EVOLUTION:` remains deliberately outside this patch's checksum-policy change; it is the oldest contract namespace and requires a separate inventory/migration decision rather than a blind strictness flip.
