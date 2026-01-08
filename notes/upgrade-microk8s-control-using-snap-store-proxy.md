# Upgrading MicroK8s Control Using Snap Store Proxy

Control revisions and upgrade rollouts.  
Cache snaps in local network, limiting external bandwidth requirements.

## Requirements

- PostgreSQL database
- A Linux machine for Snap Store Proxy deployment
- Ubuntu SSO account

---

## Installation Steps

### 1. Prepare the System

- Fresh Ubuntu (or any other) instance

### 2. Setup PostgreSQL

```bash
sudo apt update
sudo apt install postgresql
```

### 3. Create User and Database for Snap Store Proxy

```bash
echo "
CREATE ROLE \"snapproxy-user\" LOGIN CREATEROLE PASSWORD 'snapproxy-password';
CREATE DATABASE \"snapproxy-db\" OWNER \"snapproxy-user\";
\connect \"snapproxy-db\"
CREATE EXTENSION \"btree_gist\";
" | sudo -u postgres psql
```

### 4. Install Snap Store Proxy

```bash
sudo snap refresh snapd
sudo snap install snap-store-proxy
```

### 5. Connect to the Database

Replace the connection string if using an external PostgreSQL database.

```bash
export POSTGRESQL_CONNECTION_STRING="postgresql://snapproxy-user:snapproxy-password@localhost:5432/snapproxy-db"
snap-proxy config proxy.db.connection="${POSTGRESQL_CONNECTION_STRING}"
```

### 6. Ensure Network Connectivity

```bash
snap-proxy check-connections
```

**Expected output:**

```
http: https://dashboard.snapcraft.io: OK
http: https://login.ubuntu.com: OK
http: https://api.snapcraft.io: OK
postgres: localhost: OK
All connections appear to be accessible
```

### 7. Configure Domain Name

```bash
snap-proxy config proxy.domain=proxy.internal
```

### 8. Configure Proxy Cache Size

Default is 2GB. To increase to 10GB:

```bash
snap-proxy config proxy.cache.size=10240
```

Check current cache size:

```bash
snap-proxy config proxy.cache.size
```

### 9. Register the Snap Store Proxy

```bash
snap-proxy generate-keys
snap-proxy register
```

You will be asked to log in to your Ubuntu SSO account and answer a few questions.

### 10. Verify Configuration

```bash
snap-proxy status
```

---

## Connect to the Proxy

On each MicroK8s node, point snap to the Snap Store Proxy instance.  
Retrieve `$STORE_ID` from `snap-proxy status`.

```bash
export STORE_ID="YKNdvTH4IZfIFGtyaS7DSn6QCwgpgNfh"
curl http://proxy.internal/v2/auth/store/assertions | sudo snap ack /dev/stdin
sudo snap set core proxy.store="${STORE_ID}"
```

---

## Install MicroK8s

After configuring your server to use the proxy, install MicroK8s:

```bash
sudo snap install microk8s --classic
```

The first installation fetches MicroK8s from the snap store. Subsequent installations are faster due to caching.

---

## Control Upgrades

With a snap store proxy, you can pin the snap revision for each channel, controlling upgrade rollouts.

### List Available Channels and Revisions

```bash
snap list microk8s
```

**Sample output:**

```
channels:
  1.22/stable:      v1.22.3         2021-11-14 (2645) 194MB classic
  1.22/candidate:   v1.22.4         2021-11-22 (2695) 194MB classic
  1.22/beta:        v1.22.4         2021-11-22 (2695) 194MB classic
  1.22/edge:        v1.22.4         2021-11-17 (2695) 194MB classic
  latest/stable:    v1.22.3         2021-11-15 (2645) 194MB classic
  latest/candidate: v1.22.4         2021-11-18 (2693) 198MB classic
  latest/beta:      v1.22.4         2021-11-18 (2693) 198MB classic
  latest/edge:      v1.22.4         2021-11-23 (2727) 217MB classic
```

### Pin Channels to a Specific Revision

```bash
snap-proxy override microk8s stable=2645
snap-proxy override microk8s edge=2645
snap-proxy override microk8s 1.22/edge=2645
```

List overrides:

```bash
snap-proxy list-overrides microk8s
```

**Sample output:**

```
microk8s stable amd64 2645 (upstream 2645)
microk8s edge amd64 2645 (upstream 2727)
microk8s 1.22/edge amd64 2645 (upstream 2695)
```

### Verify Pinned Revisions

```bash
sudo snap info microk8s
```

**Sample output:**

```
channels:
  1.22/stable:      v1.22.3         2021-11-24 (2645) 194MB classic
  1.22/candidate:   v1.22.4         2021-11-22 (2695) 194MB classic
  1.22/beta:        v1.22.4         2021-11-22 (2695) 194MB classic
  1.22/edge:        v1.22.3         2021-11-24 (2645) 194MB classic
  latest/stable:    v1.22.3         2021-11-24 (2645) 194MB classic
  latest/candidate: v1.22.4         2021-11-18 (2693) 198MB classic
  latest/beta:      v1.22.4         2021-11-18 (2693) 198MB classic
  latest/edge:      v1.22.3         2021-11-24 (2645) 194MB classic
```

### Install MicroK8s from a Pinned Channel

```bash
sudo snap install microk8s --classic --channel=edge
snap info microk8s | grep installed
```

**Sample output:**

```
installed:          v1.22.3                    (2645) 194MB classic
```

### Delete an Override

```bash
sudo snap-proxy delete-override microk8s 1.22/edge
```

**Sample output:**

```
microk8s 1.22/edge amd64 is tracking upstream (revision 2695)
```
