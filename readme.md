# AllowReplay Draft Specification

The `AllowReplay` feature allows users to opt-in to replay protection on a per-signature basis, ensuring that a particular signature is only valid on a single Bitcoin Cash or Bitcoin Cash-like network.

Replay Protection is achieved using the existing "Fork ID" mechanism as is widely supported in Bitcoin Cash libraries and node implementations.

## Motivation

AllowReplay is a form of [Gradually Activated Replay Protection](https://www.truthcoin.info/blog/garp/) (GARP), a strategy with several advantages:

- It is strictly an advantage to the implementing network, and remains valuable when implemented by competing forks.
- Indifferent users transact on both chains by default, reducing their effect on the market.
- Large actors are strongly incentivized to offer support for popular forks, even if those forks fail to attract a majority market price.

If the Bitcoin Cash ecosystem can develop a standard for gradually activated replay protection, it will encourage valuable technical competition and open source development (lowering switching costs), provide stronger market-based checks on ecosystem actors (reducing the risk of future aggression toward miners and developers), and give users greater control over their funds during contentious splits.

With wide support for AllowReplay, future splits of Bitcoin Cash will be more competitive, less political, safer for users, and easier for large actors to support.

## Specification

After activation of AllowReplay, transaction signatures must be validated using the Fork ID specified by their AllowReplay bit.

### Opting-in: the AllowReplay Bit

This specification redefines the `SIGHASH_FORKID` bit as specified in the [Bitcoin Cash signing serialization algorithm](https://github.com/bitcoincashorg/bitcoincash.org/blob/fd304d1612a8495631cc1cfc118119c4e19a4175/spec/replay-protected-sighash.md). Bit `7` (`0x40`) will now be called the **AllowReplay bit**. After activation of the AllowReplay feature, the AllowReplay bit will no longer be required to be set for all Bitcoin Cash signatures.

When a signature's AllowReplay bit is not set, the signature must be validated using the currently active **Network Fork ID** (NFI). When a signature's AllowReplay bit is set, the signature must be validated using the **Re-playable Fork ID** (RFI) of `0x000000`.

### Rotating Network Fork IDs

A new, unique, pseudorandom **Network Fork ID** (NFI) must be chosen for each network upgrade. This value serves as an identifier for the precise consensus version to be activated, and it should be changed every time consensus rules are changed. It is recommended that the NFI value for a particular network upgrade be selected from the hash of a representative specification or implementation (e.g. Git fingerprint).

To allow network participants to plan for upgrades, the deprecation conditions for a Network Fork ID should be specified before its activation.

At the moment a new network upgrade is activated, un-mined transactions containing signatures in which AllowReplay is disabled must be re-validated using the new Network Fork ID. If the transaction is no longer valid, it must be discarded. Wallets, services, and contracts should be written to expect this behavior: any transactions disabling AllowReplay must be mined before the network upgrade to remain valid.

### Known Fork IDs

This table is intended as a reference for Bitcoin Cash's November 2020 upgrade and might not be updated in the [AllowReplay specification repo](https://github.com/bitjson/allow-replay-spec) after MTP `1605441600` (Nov 15, 2020 12:00:00 UTC).

| Fork ID    | Activation       | Deprecation      | Fork ID Reference                                                                                                                                                                                               |
| ---------- | ---------------- | ---------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `0x000000` | N/A              | N/A              | (Re-playable Fork ID)                                                                                                                                                                                           |
| `0xee5d75` | `MTP 1605441600` | `MTP 1621080000` | [bchn-sw/bitcoincash-upgrade-specifications@ee5d75](https://gitlab.com/bitcoin-cash-node/bchn-sw/bitcoincash-upgrade-specifications/-/blob/ee5d75be997f009295c309269184fd2ba8078f59/spec/2020-11-15-upgrade.md) |
| `0xee7782` | `MTP 1605441600` | `MTP 1621080000` | [bitcoincash.org@ee7782](https://github.com/bitcoincashorg/bitcoincash.org/blob/ee7782042641ec17d7d28b633711e1cc1079f894/spec/2020-11-15-upgrade.md)                                                            |

## Deployment

Deployment of this proposal is recommended for the November 2020 upgrade (`BCH_2020_11`).

Activation is triggered via [BIP9](https://github.com/bitcoin/bips/blob/6295c1a095a1fa33f38d334227fa4222d8e0a523/bip-0009.mediawiki) "version bits" signalling, with the name `AllowReplay` and using bit `7` (`0x40`). The `activation threshold` is set to `1344` (which is about 2/3 of the blocks in each 2016 block period). The `starttime` is set to `1589544000` (MTP May 15, 2020 12:00:00 UTC). The `timeout` is set to `1605441600` (Nov 15, 2020 12:00:00 UTC).

## Rationale

This section is non-normative. It is intended to explain the design decisions made in this specification.

### On Using the Fork ID Mechanism

The current [Bitcoin Cash signing serialization algorithm](https://github.com/bitcoincashorg/bitcoincash.org/blob/fd304d1612a8495631cc1cfc118119c4e19a4175/spec/replay-protected-sighash.md) is well known and already has many implementations, so most wallet software will only need to be modified to allow for a variable **Network Fork ID** (NFI). Once this change has been made, wallet software should be capable of accommodating all future NFIs, even those specified after the software was written or released. Even use cases like hardware wallets can reliably ask the user to validate an "unknown Network Fork ID" at signing time, much like users are already asked to validate payment amounts and addresses.

AllowReplay paves the way for easier support of Bitcoin Cash splits in wallets and services. With the same signing method, most future splits will be easily accommodated.

### On Redefining the `SIGHASH_FORKID` Bit

The AllowReplay bit (previously `SIGHASH_FORKID` bit) is no longer necessary to differentiate Bitcoin Cash transactions from BTC transactions. The signing serialization algorithms are already mutually incompatible.

### On Rotating Network Fork IDs

Rotating Network Fork IDs ensures that no more than two fork IDs will be in use on a network – the **Network Fork ID** (varies) and the **Re-playable Fork ID** (`0x000000`) – minimizing future technical debt. Because this behavior can be well-known in advance, users are able to plan split behaviors even for complex situations like assurance contracts or covenants.

This is also a strict upgrade from the current reality, where future splits can be expected, but planning for them is fairly difficult.

## Implementations

Please see the following reference implementations for examples and test vectors:

[TODO]

## Acknowledgements

Thanks to Justus Ranvier who [noted](https://twitter.com/bitjson/status/1299126960409452551) that an earlier version of this proposal could be simplified by redefining the `SIGHASH_FORKID` bit to be the opt-in mechanism.

Thanks to the [bitcoincash.org contributors](https://github.com/bitcoincashorg/bitcoincash.org/commit/ee7782042641ec17d7d28b633711e1cc1079f894) for [the IFP activation specification](https://github.com/bitcoincashorg/bitcoincash.org/blob/master/spec/2020-05-15-ifp.md) from which the activation strategy for AllowReplay was derived.

## Copyright

This document is placed in the public domain.
