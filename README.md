# qubes-bridge-device-tests

Ansible testbed for the Qubes OS bridge device addon, split across two components:

- [qubes-core-admin-addon-bridge-device](https://github.com/fepitre/qubes-core-admin-addon-bridge-device): dom0 extension (device class, lifecycle hooks)
- [qubes-core-agent-linux-addon-bridge-device](https://github.com/fepitre/qubes-core-agent-linux-addon-bridge-device): VM-side scripts (publish-bridge, harden-bridge)

All playbooks run from dom0 against `localhost` and connect to backend VMs via the `qubes` connection plugin.

## Testbed layout

Two backend StandaloneVMs (`net-interfaces`, `net-interfaces-bis`) each host virtual bridges. Frontend AppVMs attach to those bridges as bridge devices.


| Backend | Bridge | Frontend VMs |
|---|---|---|
| net-interfaces | br-dev | testvm1, testvm2 |
| net-interfaces | br-dev-2 | testvm3, testvm4 |
| net-interfaces-bis | br-dev | testvm5, testvm6 |

## Playbooks

| Playbook | Purpose |
|---|---|
| `playbooks/main.yml` | Create and configure the backend StandaloneVMs |
| `playbooks/deploy.yml` | Deploy agent scripts and admin extension, restart qubesd |
| `playbooks/tests.yml` | Run the full test suite |
| `playbooks/cleanup.yml` | Tear down all testbed VMs and dom0 artifacts (dev use) |

## Setup and run

```sh
# From dom0 -- one-time testbed creation
ansible-playbook playbooks/main.yml
ansible-playbook playbooks/deploy.yml

# Run tests
ansible-playbook playbooks/tests.yml

# Selective test tags: discovery attach options persistence lifecycle isolation hardening
ansible-playbook playbooks/tests.yml --tags isolation,hardening
```

Source paths are configured in `group_vars/all.yml` (`core_admin_src`, `core_agent_src`). Override them at the command line with `-e` if the repos are cloned elsewhere.

## CI

Both addon repos run this testbed automatically on every PR via the `r4.3:bridge-device-integration` job. Each job clones the companion repo at the matching branch (falls back to main) and passes the source paths to Ansible.
