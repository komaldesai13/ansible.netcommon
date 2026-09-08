<!--
Thanks for contributing to ansible.netcommon!

This template implements the AAP Organization-Level Testing Strategy:
https://github.com/ansible/handbook/blob/main/The%20Ansible%20Engineering%20Handbook/Quality/aap-org-testing-strategy/index.md

Every PR must demonstrate coverage of the relevant integration dimensions.

THE RULE FOR EVERY CHECKBOX IN THIS TEMPLATE: tick it, or give a reason.
Each section below has somewhere to put that reason --
  * Integration dimensions -> the "Reason if not covered" column
  * E2E levels             -> the "Reason if not run / not passing" column,
                              plus the free-text box under that table
  * Checklist              -> the "Reason for any unchecked box above" box
Reviewers block merge on an unexplained gap, not on a justified N/A.
-->

## Overview

**What does this PR do?**

<!-- Brief description of the change and the problem it solves. -->

**Issue / Jira:** <!-- e.g. Fixes #791, AAP-XXXXX -->

**Change type:** <!-- bugfix / feature / refactor / docs / test / release -->

**Components touched:** <!-- e.g. network_cli, netconf, httpapi, cli_parse, restconf, module_utils -->

**Related PRs:** <!-- Links to PRs in other repos (network platform collections, ansible.utils, EE definitions) for the same change -->

---

## Integration Dimension Coverage

Mark `[x]` when the dimension is covered, or leave it unticked and explain in the
**Reason if not covered** column. Name the actual test(s) — "added tests" is not
specific enough and will get a *Request changes*.

| # | Dimension | Covered | Test(s) / evidence | Reason if not covered |
| - | --------- | ------- | ------------------ | --------------------- |
| 1 | 📦 Installation | [ ] | <!-- build-import / galaxy install / EE build --> | |
| 2 | 🔌 API Contracts | [ ] | <!-- argspec, module DOCUMENTATION/RETURN, sanity --> | |
| 3 | 🔐 Auth/AuthZ | [ ] | <!-- connection auth, credentials, become/enable --> | |
| 4 | 💾 Data Persistence | [ ] | <!-- persistent connection state, facts, cache --> | |
| 5 | 📊 Observability | [ ] | <!-- logging, no secret leakage, error messages --> | |
| 6 | 🔄 User-flows | [ ] | <!-- see E2E section below --> | |
| 7 | ⬆️ Upgrade | [ ] | <!-- backwards compat, deprecation, ansible-core range --> | |
| 8 | 🏗️ Topology | [ ] | <!-- platforms / transports / ansible-core versions --> | |

<details>
<summary><b>What each dimension means in this collection</b> (click to expand)</summary>

1. **📦 Installation** — The collection still builds and installs cleanly: `build-import` and
   `ansible-lint` pass, `galaxy.yml` / `meta/runtime.yml` updated if plugins or
   `requires_ansible` changed, and any new Python dependency is added to
   `requirements.txt` / `bindep.txt` so execution environments can build.
2. **🔌 API Contracts** — Module/plugin interfaces are the contract here. Argspec changes are
   backwards compatible or deprecated properly, `DOCUMENTATION`/`EXAMPLES`/`RETURN` are updated
   and pass sanity (validate-modules), and `module_utils` signatures consumed by platform
   collections (`cisco.ios`, `arista.eos`, `junipernetworks.junos`, …) are not broken silently.
3. **🔐 Auth/AuthZ** — Connection-level auth is exercised: SSH key/password, `become`/`enable`,
   `httpapi`/`restconf` token and certificate handling, proxy/jump-host settings. Negative cases
   count — bad credentials must fail cleanly rather than hang or leak.
4. **💾 Data Persistence** — Persistent connection behaviour: the `ansible-connection` socket
   lifecycle, `persistent_connect_timeout` / `persistent_command_timeout`, reconnect after the
   device drops the session, fact and connection-cache contents surviving across tasks.
5. **📊 Observability** — Logging and diagnosability: messages are actionable, `no_log`/`vault`
   values and device credentials never appear in output or in
   `ANSIBLE_LOG_PATH` / `ANSIBLE_PERSISTENT_LOG_MESSAGES` output, and new failure paths raise a
   clear `AnsibleConnectionFailure`/`AnsibleError` instead of a traceback.
6. **🔄 User-flows** — End-to-end usage works. In this repo that is the integration suite under
   `tests/integration/targets/` (`cli_tests`, `netconf_*`, `restconf_*`, `httpapi_tests`,
   `cli_parse`) run against real or simulated devices — the closest equivalent of the strategy's
   Level 1 / Level 2 Platform User-flow tests.
7. **⬆️ Upgrade** — Users upgrading the collection are not broken: no removal without a
   deprecation cycle, changelog fragment added under `changelogs/fragments/`, `requires_ansible`
   still honoured, and behaviour changes called out in the fragment.
8. **🏗️ Topology** — Coverage across the deployment matrix that matters here: connection plugins
   (`network_cli`, `netconf`, `httpapi`, `libssh` vs `paramiko`), network platforms (IOS-XE,
   IOS-XR, NX-OS, EOS, JunOS, …), and supported `ansible-core` versions.

Not every dimension applies to every PR. Bug fixes typically touch 2–4; refactors 3–5. Marking a
dimension as not applicable is fine **with a real reason** ("no API surface change, internal
refactor only"). It is not fine with "no time", "hard to test", or "will test later".

</details>

---

## E2E / Platform User-flow Tests

The strategy makes end-to-end user-flow validation **blocking before merge**. In this repo that is
the CML lab run — an ephemeral multi-node topology
(`tests/integration/labs/multi.yaml`: NX-OS, IOS-XR, IOS-XE, Ubuntu) that the
`Integration tests 💻` workflow builds, runs `ansible-test network-integration` against, and tears
down. Every level below has a reason column: **if it is not ticked, say why in that row.**

| Level | What runs | Passed | Reason if not run / not passing |
| ----- | --------- | ------ | ------------------------------- |
| **0 — Collection gates** | `Collection Tests 🧪`: changelog, build-import, ansible-lint, sanity, unit | [ ] | |
| **1 — Critical path E2E** | `Integration tests 💻` on the CML lab — all targets under `tests/integration/targets/` | [ ] | |
| **2 — Downstream / integrated product** | Same run, with `cisco.ios`, `cisco.nxos`, `cisco.iosxr`, `ansible.utils` cloned from `main` | [ ] | |

**Which E2E targets exercise this change?**

<!-- Tick what your change actually touches; untick the rest. -->

- [ ] `cli_tests` — `network_cli` connect / run command / prompt handling
- [ ] `netconf_get`, `netconf_config`, `netconf_rpc` — NETCONF flows
- [ ] `restconf_get`, `restconf_config` — RESTCONF over `httpapi`
- [ ] `httpapi_tests` — `httpapi` connection flows
- [ ] `cli_parse` — parser engines
- [ ] Other / new target: <!-- name it -->

**Transport & version matrix.** A PR run only covers **`libssh` + ansible-core `devel`**. Anything
else (`paramiko`, or core 2.15–2.20 / `milestone`) needs a manual **Run workflow** dispatch on
`integration-cml.yml` with those inputs selected.

- [ ] Default PR matrix (`libssh` + `devel`) is sufficient for this change
- [ ] Extra matrix run(s) needed and completed — link them: <!-- run URL(s) -->

**If E2E did not run or did not fully pass, explain here:**

<!--
Required if any row or box above is unticked. Common valid reasons:
- Docs-only / changelog-only change — workflow does not trigger on these paths
- Awaiting the `safe to test` label from a maintainer (fork PRs do not run E2E without it)
- CML lab or infrastructure failure unrelated to this change — link the failed run
- Known flake — link the tracking issue
Not valid: "will run later", "passes locally", "too slow".
-->

<details>
<summary><b>How to get E2E to run on your PR</b> (click to expand)</summary>

- The workflow triggers on changes under `plugins/**` or `tests/integration/**`. Other paths never
  run it — that is a legitimate reason to leave Level 1 unticked, just say so.
- Fork PRs are gated: a maintainer must add the **`safe to test`** label before the lab is built.
  Ask for it in a comment if your PR needs E2E.
- Add the **`debug`** label to get pip lists, the rendered inventory, the ansible version, and
  libssh versions printed in the run.
- To cover paramiko or an older ansible-core, use **Actions → Integration tests 💻 → Run workflow**
  and pick `ssh_type` / `ansible_version`, then link that run above.
- **You own failures your PR causes**, including ones that surface in a downstream collection
  (`cisco.ios`, `cisco.nxos`, `cisco.iosxr`) rather than in netcommon itself — those clones are
  there precisely to catch integrated-product breakage before merge. If the failure is a genuine
  flake, link the tracking issue rather than re-running until green.

</details>

---

## Checklist

- [ ] Changelog fragment added under `changelogs/fragments/` (or explain below why not needed)
- [ ] New/changed behaviour is covered by a unit or integration test (or explain below)
- [ ] Module/plugin documentation updated and renders correctly
- [ ] No secrets, device credentials, or customer data in code, tests, or logs
- [ ] Backwards compatible for existing playbooks (or deprecation documented)

**Reason for any unchecked box above:**

<!-- Required if anything above is left unchecked. -->

---

## Breaking Changes

- [ ] This PR contains breaking changes

**If yes:** describe the break, the migration path, and link the decision log / notification to
downstream platform collection maintainers.

<!--
Reviewer note: verify dimension coverage and test evidence before approving.
Challenge any dimension marked as not covered where the diff suggests it should apply —
e.g. a new module option without an API Contracts entry, or a connection change without
an Auth/AuthZ or Data Persistence entry.
-->
