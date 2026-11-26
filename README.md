# Experimental GBL chainloader changes for myron

This repository publishes a source-only, curated set of committed changes to the GBL chainloader for review and further development. It includes the B1, B2 and B2b integration changes. These safeguards are **experimental**, not a stable release or a claim of device safety. No EFI binary or release asset is distributed here.

The changes include a Fastboot escape path and lifecycle handling, cached ABL PE and patch-site checks, firmware identity checks, protocol-hook rollback and Block I/O protections, along with related host tests. The included DICE patch comes from [an upstream commit](https://github.com/1vivy/gbl-chainload/commit/dc73ef8475d641b6fb3e104b646e0352c19f2875) authored by SuperTurtle (1vivy), built on NullCode1337's earlier myron Mode 2 work. These changes interact with boot and verified-boot behavior; review the code and applicability to the exact device and firmware before considering a build.

## Contents and provenance

- [`patches/chainloader-b2b.patch`](patches/chainloader-b2b.patch) is the selected source, build-script and test diff against [NullCode's gbl-chainload-myron](https://github.com/NullCode1337/gbl-chainload-myron) commit `3f19d7afe3b14ef836a0804ea2d67987866cde09`. It reflects committed integration tip `5de970ccfd23883772fa1b200111d72dbca48d12`. The patch also retains an ignore rule for the local-only myron firmware fixture. Private investigation notes, CI and release workflows, existing firmware images and uncommitted work are outside this publication.
- The patch updates the EDK2 submodule to the public [EDK2 fork](https://github.com/xistanz83/edk2-gbl-chainload) commit `a85473857c2fa0afd29717e2615253e47f1e2897`. [`patches/edk2-fastboot.patch`](patches/edk2-fastboot.patch) separately shows the two-file EDK2 change from `cd85753b97a9cd48a362f29ebaee486bcf4060bb` to that commit. Full copies of those two modified files, including their copyright and license notices, are in [`edk2-source/`](edk2-source/).
- The two patches are a reviewable source contribution, not a standalone build tree. The EDK2 patch is already represented by the updated submodule pin in the chainloader patch: do not apply it again after checking out that pin.

The chainloader patch deliberately excludes the private repository's CI and release workflow changes and investigation documentation. The upstream checkout contains its own files and workflows, and may contain firmware fixtures; they are not distributed by this repository.

## Applying the chainloader patch

Start from a clean checkout of the exact upstream commit, save this repository separately, and use the absolute path to the patch:

```sh
git clone https://github.com/NullCode1337/gbl-chainload-myron.git chainloader
git -C chainloader checkout --detach 3f19d7afe3b14ef836a0804ea2d67987866cde09
git -C chainloader apply --index /absolute/path/to/patches/chainloader-b2b.patch
git -C chainloader submodule update --init --recursive
```

`--index` is needed to update the EDK2 submodule entry as well as the regular files. Review the inherited upstream build and release scripts before using them. This source publication does not assert that a build reproduces any particular EFI.

## Testing and limitations

On the original B2b integration commit, private CI reported successful host tests, a firmware smoke build, an Android recovery cross-build, and host-tool build/smoke jobs. These are build and host-test results, not hardware validation of the integrated changes. The private firmware identity fixture was unavailable in CI, so fixture-dependent tests returned early.

For this publication, the patch was applied to a clean upstream checkout and checked locally. The Rust patch-engine host suite completed successfully, but the myron firmware-identity and Tier 1b cases requiring the local-only fixture returned early even though the test runner reports them as passed. The DICE tests used a fixture already present in the upstream checkout. The Block I/O rollback mock passed 33 checks; the Fastboot lifecycle mocks and command tests passed. A Fastboot transfer-wait characterization observed premature returns for stale or error completions and a wait that did not exit on disconnect during its bounded observation. Those observations remain unresolved and the mocks do not establish real USB controller safety. No new device testing or full firmware build was performed for the curated patch.

**Known verifier issue:** the committed `scripts/verify-safeguards.sh` is not validated here and its AArch64 check expands `pe_off` in the outer shell before that variable is set. With `set -u` this can terminate the script; do not rely on it as proof of safeguards.

## Credits and licenses

Thank you to NullCode, SuperTurtle (1vivy), and the upstream contributors for the original chainloader, myron work, and related contributions. This is a separate collection of modifications built on their work. The chainloader license and copyright notice are preserved in [`LICENSE`](LICENSE); the EDK2 license is preserved in [`LICENSE-EDK2.txt`](LICENSE-EDK2.txt), and individual EDK2 source files retain additional notices.
