# Trezor, with the unified opt-in signature hash

An unofficial fork of [Trezor Firmware](https://github.com/trezor/trezor-firmware) that signs with the unified opt-in signature hash defined by the Bitcoin hardfork, as Bitcoin Knots `v29.4.1.knots20260508` specifies it in [doc/unified-sighash.md](https://github.com/bitcoinknots/bitcoin/blob/v29.4.1.knots20260508/doc/unified-sighash.md). It is not affiliated with SatoshiLabs. SatoshiLabs has not adopted the fork, so use their firmware instead if that is what you want.

> **Not audited. Use at your own risk, and no warranty of any kind, see the [LICENSE](LICENSE.md).** Builds here are not signed by the vendor, so a Trezor Model One shows an "Unofficial firmware detected" screen and its fingerprint on every boot and requires two button presses to continue, permanently. Installing it erases the device, and so does going back to official firmware, so you need your recovery seed both times. Everything below the divider is SatoshiLabs' documentation and describes their firmware rather than this fork.

## What differs from Trezor Firmware

A signature that opts in commits to **every** spent amount and scriptPubKey rather than only the input being signed, which closes CVE-2020-14199 at the consensus layer, and it does not verify under the pre-fork rules, so the transaction cannot be replayed onto a chain that has not adopted the change. **Signing is byte-for-byte unchanged unless the host asks for it.**

- `TxInput.unified_sighash` (protobuf field 21) requests hash type `0x21` for that input, the way `PSBT_IN_SIGHASH_TYPE` carries it in a PSBT.
- `Capability_UnifiedSigHash` (28) is reported in `Features`, and `trezorlib` refuses to request the opt-in from a device that does not report it rather than letting the field be ignored.
- The digest itself, in `core` and in `legacy`, taken from the sub-hashes both already stream for BIP-341. Those are single SHA-256 where the BIP-143 digests from the same writers are double.
- An approval screen naming the hash type, in the same register as the existing sighash warnings.
- The opt-in is bound to what the confirmation screen reported, and a replacement transaction may add it but never remove it.

Script types covered are bare and P2SH, segwit v0 including P2SH-wrapped, and taproot key path. Only `SIGHASH_ALL|SIGHASH_UNIFIED` is implemented, because this firmware never signs `SIGHASH_NONE`, `SIGHASH_SINGLE` or `SIGHASH_ANYONECANPAY` and so has no legacy counterpart to give a unified form to. Tapscript is unreachable because there is no script-path taproot here.

Trezor Suite does not know about the field, so opting in needs a host that sets it. The [Shrike](https://github.com/privkeyio/shrike) fork does, through [lark](https://github.com/privkeyio/lark).

Releases carry a Bitcoin-only Model One build with a reproducible build recipe, published hashes and a signed manifest.

---

# Trezor Firmware

![img](https://repository-images.githubusercontent.com/180590388/968e6880-6538-11e9-9da6-4aef78157e94)

## Repository Structure

* **[`ci`](ci/)**: Ancillary files, data, and scripts for the CI pipeline
* **[`common/defs`](common/defs/)**: JSON coin definitions and support tables
* **[`common/protob`](common/protob/)**: Common protobuf definitions for the Trezor protocol
* **[`common/tools`](common/tools/)**: Tools for managing coin definitions and related data
* **[`core`](core/)**: Trezor Core, a firmware implementation for Trezor T, Trezor Safe 3, Trezor Safe 5, and Trezor Safe 7
* **[`crypto`](crypto/)**: Stand-alone cryptography library used by both Trezor Core and the Trezor One firmware
* **[`docs`](docs/)**: Assorted documentation
* **[`legacy`](legacy/)**: Trezor One firmware implementation
* **[`python`](python/)**: Python [client library](https://pypi.org/project/trezor) and the `trezorctl` command
* **[`storage`](storage/)**: NORCOW storage implementation used by both Trezor Core and the Trezor One firmware
* **[`tests`](tests/)**: Firmware unit test suite
* **[`tools`](tools/)**: Miscellaneous build and helper scripts
* **[`vendor`](vendor/)**: Submodules for external dependencies

## Contribute

See [CONTRIBUTING.md](docs/misc/contributing.md).

Using [Conventional Commits](COMMITS.md) is strongly recommended and might be enforced in the future.

Also, please have a look at the docs, either in the `docs` folder or at [docs.trezor.io](https://docs.trezor.io) before contributing. The [misc](docs/misc/index.md) chapter should be read in particular, as it contains useful assorted knowledge.

## Security Vulnerability Disclosure

Please do NOT create publicly viewable issues for suspected security vulnerabilities. See the [Security tab](https://github.com/trezor/trezor-firmware/security/policy) for reporting instructions.

## Documentation

See the `docs` folder or visit [docs.trezor.io](https://docs.trezor.io).
