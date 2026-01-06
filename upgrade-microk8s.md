# Upgrading MicroK8s to a New Minor Version

MicroK8s uses snap channels, restricting automatic updates to new versions published in that channel.  
This ensures version upgrades only occur when the user requests them.

MicroK8s channels follow the upstream release version of Kubernetes, making it easy to specify a version and restrict updates to non-breaking changes within the same minor version.  
To upgrade to a new version, refresh the snap to point to a different channel. Additional steps may be required on a running cluster to minimize disruption.

---

## Important Notes

- **Workloads running in the cluster** (including enabled add-ons and the CNI) will **NOT** be upgraded as part of a MicroK8s upgrade.
- Upgrade add-ons by disabling and re-enabling them:

    ```bash
    microk8s disable <add-on>
    microk8s enable <add-on>
    ```

- Read the release notes for specific details about add-ons before upgrading.
- The latest manifest for the default CNI (Calico) is at:

    ```
    /snap/microk8s/current/upgrade-scripts/000-switch-to-calico/resources/calico.yaml
    ```

    Patch it with any customizations before applying.

---

## Holding Upgrades

You can delay or hold snap refreshes:

- **Delay refreshes for a specified period:**

    ```bash
    snap refresh --hold=24hr microk8s
    ```

- **Delay refreshes until a specific date:**

    ```bash
    snap refresh --hold=2023-02-18T15:22:04+00:00
    ```

- **Stop updates altogether:**

    ```bash
    snap refresh --hold microk8s
    ```

> If you hold or turn off snap refreshes, your software may not be the most up-to-date or patched version.

---

## Cluster Upgrade Guidelines

- Kubernetes releases strive for API backward compatibility, but always check release notes for breaking changes or deprecations.
- **MicroK8s upgrade constraints:**
    - "Skip-level" upgrades are NOT tested. Upgrade only one minor release at a time (e.g., 1.19 → 1.20).
    - Downgrading (e.g., 1.20 → 1.19) is not tested or supported.
    - Migrate configuration changes incrementally across versions.
    - Customizations (e.g., service arguments) are carried over, but may be incompatible with the new version.

If an upgrade is not possible, you can re-install MicroK8s targeting the desired version.

---

## Upgrading a Single Cluster Node

Refresh the MicroK8s snap to the desired channel:

```bash
snap refresh microk8s --channel=1.21/stable
```

---

## Upgrading a Multi-Node Cluster

Kubernetes allows some service skew, so you can upgrade nodes one at a time.  
See `./k8s-upgrade.md` for more details.

---

## Reverting a Failed Upgrade

If the upgrade fails, revert the node to its previous state:

```bash
snap revert microk8s
```

For diagnostics, collect a tarball of cluster/node information before reverting:

```bash
microk8s inspect
```

