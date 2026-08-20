# PingFederate Cluster Build Guide

> **Source:** PingFederate PreProd Build Documentation v2.0  
> **Original document version:** 1.0  
> **Original date:** 27 April 2020  
> **Purpose:** Build and configure a PingFederate 9.3.3 clustered deployment for the PreProd environment.

## Overview

This repository contains a GitHub-ready version of the supplied PingFederate PreProd build document. The source document describes a clustered PingFederate deployment consisting of:

- **PingFederate Admin Console / Cluster Console**
- **PingFederate Cluster Engine 1**
- **PingFederate Cluster Engine 2**

The source document identifies the following PreProd topology:

| Server | Role | Cluster Node Index |
|---|---|---:|
| `170.11.196.11` | Admin Console / Cluster Console | `100` |
| `170.11.196.11` | Cluster Engine 1 | `101` |
| `170.11.196.12` | Cluster Engine 2 | `102` |

The document also specifies PingFederate **9.3.3** and JDK **8u202**.

---

## Repository Structure

```text
.
├── README.md
└── assets/
    ├── image1.png   # Cluster Management validation screen
    └── image2.png   # Protocol Settings / Base URL / Entity ID screen
```

---

## 1. Architecture

The intended cluster relationship from the source configuration is:

```text
                         PreProd PingFederate Cluster
                                  |
                    +-------------+-------------+
                    |                           |
          170.11.196.11                 170.11.196.12
                    |                           |
          +---------+---------+             +---+---+
          |                   |             |       |
     Admin Console       Engine 1       Engine 2
      Node Index 100     Node Index 101  Node Index 102
          |                   |             |
          +-------------------+-------------+
                    Cluster Discovery
```

The cluster discovery configuration in the source document uses:

```properties
pf.cluster.tcp.discovery.initial.hosts=170.11.196.11[8600],170.11.196.12[8600]
```

---

## 2. Prerequisites

Before starting the build, ensure that the required installation packages and server access are available.

### Software

- PingFederate `9.3.3`
- JDK `8u202` (`jdk-8u202-linux-x64.tar.gz`)
- Linux server access
- `unzip`
- File transfer capability such as WinSCP

### Directories

The source document uses:

```text
/apps/partner
/apps/partner/jdk/
```

The Admin Console installation uses:

```text
/apps/partner/pingfederate-9.3.3-admin-ptnr
```

The runtime engines use:

```text
/apps/partner/pingfederate-9.3.3-runtime-ptnr
```

---

# 3. Admin Console Installation

The Admin Console is installed on:

```text
170.11.196.11
```

The source document specifies the administrative user:

```text
ping9ptnradmin
```

## 3.1 Create the Installation Directory

```bash
cd /apps/partner
mkdir pingfederate-9.3.3-admin-ptnr
```

## 3.2 Install Java

Copy the JDK archive to:

```text
/apps/partner/jdk/
```

Extract it:

```bash
tar -xvf jdk-8u202-linux-x64.tar.gz
```

The source document configures:

```text
JAVA_HOME=/apps/partner/jdk/jdk1.8.0_202
```

## 3.3 Install PingFederate

Copy:

```text
PingFederate9.3.3.zip
```

to:

```text
/apps/partner/pingfederate-9.3.3-admin-ptnr
```

Then:

```bash
cd /apps/partner/pingfederate-9.3.3-admin-ptnr
unzip PingFederate9.3.3.zip
```

The source document then moves the extracted `pingfederate` directory into the installation directory and removes the temporary extracted version directory.

---

## 3.4 Configure `run.properties`

Navigate to:

```bash
cd /apps/partner/pingfederate-9.3.3-admin-ptnr/pingfederate/bin
```

Configure the Admin Console properties:

```properties
pf.admin.https.port=8888
pf.https.port=6031
pf.operational.mode=CLUSTERED_CONSOLE
pf.cluster.node.index=100
pf.cluster.bind.port=8700
pf.cluster.failure.detection.bind.port=8900
pf.cluster.tcp.discovery.initial.hosts=170.11.196.11[8600],170.11.196.12[8600]
```

### Admin Console Configuration Summary

| Property | Value |
|---|---|
| Admin HTTPS Port | `8888` |
| HTTPS Port | `6031` |
| Operational Mode | `CLUSTERED_CONSOLE` |
| Node Index | `100` |
| Cluster Bind Port | `8700` |
| Failure Detection Port | `8900` |
| Discovery Hosts | `170.11.196.11[8600],170.11.196.12[8600]` |

---

## 3.5 Configure Environment Variables

Edit:

```bash
vi ~/.bash_profile
```

The source document specifies:

```bash
export JAVA_HOME=/apps/partner/jdk/jdk1.8.0_202
export PF_HOME=/apps/partner/pingfederate-9.3.3-admin-ptnr/pingfederate
PATH=$PATH:$HOME/bin:$JAVA_HOME/bin

alias ExecStart="${PF_HOME}/sbin/pingfederate-run.sh"
alias ExecStop="${PF_HOME}/sbin/pingfederate-shutdown.sh"

alias cdlog="cd ${PF_HOME}/log"
alias cdbin="cd ${PF_HOME}/bin"
alias cdping="cd ${PF_HOME}"
alias cdconf="cd ${PF_HOME}/pingfederate/server/default/conf"
alias taillog="tail -f ${PF_HOME}/log/server_srbhppoauthapp2.log"
```

Reload the profile:

```bash
source ~/.bash_profile
```

---

# 4. Cluster Engine 1 Installation

The source document describes Engine 1 as running on:

```text
170.11.196.11
```

The runtime user is:

```text
ping9ptnr
```

## 4.1 Install Java

Use the same JDK archive:

```text
jdk-8u202-linux-x64.tar.gz
```

Extract it under:

```text
/apps/partner/jdk/
```

```bash
tar -xvf jdk-8u202-linux-x64.tar.gz
```

## 4.2 Install PingFederate Runtime

Copy:

```text
PingFederate9.3.3.zip
```

to:

```text
/apps/partner/pingfederate-9.3.3-runtime-ptnr
```

Then:

```bash
cd /apps/partner/pingfederate-9.3.3-runtime-ptnr
unzip PingFederate9.3.3.zip
```

After extraction, place the PingFederate runtime directory under the target runtime directory as described in the source build document.

---

## 4.3 Configure Engine 1 `run.properties`

Navigate to the PingFederate `bin` directory.

The source document specifies:

```properties
pf.https.port=8031
pf.secondary.https.port=8032
pf.operational.mode=CLUSTERED_ENGINE
pf.cluster.node.index=101
pf.cluster.bind.port=8600
pf.cluster.failure.detection.bind.port=8800
pf.cluster.tcp.discovery.initial.hosts=170.11.196.11[8600],170.11.196.12[8600]
```

### Engine 1 Configuration Summary

| Property | Value |
|---|---|
| HTTPS Port | `8031` |
| Secondary HTTPS Port | `8032` |
| Operational Mode | `CLUSTERED_ENGINE` |
| Node Index | `101` |
| Cluster Bind Port | `8600` |
| Failure Detection Port | `8800` |
| Discovery Hosts | `170.11.196.11[8600],170.11.196.12[8600]` |

---

## 4.4 Configure Engine 1 Environment

Edit:

```bash
vi ~/.bash_profile
```

Configure:

```bash
export JAVA_HOME=/apps/partner/jdk/jdk1.8.0_202
export PATH=$JAVA_HOME/bin:$PATH
export PF_HOME=/apps/partner/pingfederate-9.3.3-runtime-ptnr/pingfederate

PATH=$PATH:$HOME/bin:$JAVA_HOME/bin

alias ExecStart="${PF_HOME}/sbin/pingfederate-run.sh"
alias ExecStop="${PF_HOME}/sbin/pingfederate-shutdown.sh"

alias cdlog="cd ${PF_HOME}/log"
alias cdbin="cd ${PF_HOME}/bin"
alias cdping="cd ${PF_HOME}"
alias cdconf="cd ${PF_HOME}/pingfederate/server/default/conf"
alias taillog="tail -f ${PF_HOME}/log/server_srbhppoauthapp2.log"
```

Reload:

```bash
source ~/.bash_profile
```

---

# 5. Cluster Engine 2 Installation

Engine 2 is installed on:

```text
170.11.196.12
```

The runtime user is:

```text
ping9ptnr
```

## 5.1 Install Java

Copy the JDK archive to the required location and extract:

```bash
tar -xvf jdk-8u202-linux-x64.tar.gz
```

## 5.2 Install PingFederate Runtime

Copy:

```text
PingFederate9.3.3.zip
```

to:

```text
/apps/partner/pingfederate-9.3.3-runtime-ptnr
```

Then:

```bash
cd /apps/partner/pingfederate-9.3.3-runtime-ptnr
unzip PingFederate9.3.3.zip
```

Place the extracted PingFederate runtime under the target runtime directory as described in the source document.

---

## 5.3 Configure Engine 2 `run.properties`

The source document specifies:

```properties
pf.https.port=8031
pf.secondary.https.port=8032
pf.operational.mode=CLUSTERED_ENGINE
pf.cluster.node.index=102
pf.cluster.bind.port=8600
pf.cluster.failure.detection.bind.port=8800
pf.cluster.tcp.discovery.initial.hosts=170.11.196.11[8600],170.11.196.12[8600]
```

### Engine 2 Configuration Summary

| Property | Value |
|---|---|
| HTTPS Port | `8031` |
| Secondary HTTPS Port | `8032` |
| Operational Mode | `CLUSTERED_ENGINE` |
| Node Index | `102` |
| Cluster Bind Port | `8600` |
| Failure Detection Port | `8800` |
| Discovery Hosts | `170.11.196.11[8600],170.11.196.12[8600]` |

---

## 5.4 Configure Engine 2 Environment

Edit:

```bash
vi ~/.bash_profile
```

Configure:

```bash
export JAVA_HOME=/apps/partner/jdk/jdk1.8.0_202
export PATH=$JAVA_HOME/bin:$PATH
export PF_HOME=/apps/partner/pingfederate-9.3.3-runtime-ptnr/pingfederate

PATH=$PATH:$HOME/bin:$JAVA_HOME/bin

alias ExecStart="${PF_HOME}/sbin/pingfederate-run.sh"
alias ExecStop="${PF_HOME}/sbin/pingfederate-shutdown.sh"

alias cdlog="cd ${PF_HOME}/log"
alias cdbin="cd ${PF_HOME}/bin"
alias cdping="cd ${PF_HOME}"
alias cdconf="cd ${PF_HOME}/pingfederate/server/default/conf"
alias taillog="tail -f ${PF_HOME}/log/server_srbhppoauthapp2.log"
```

Reload:

```bash
source ~/.bash_profile
```

---

# 6. Systemd Configuration

The source document describes systemd service definitions for both the Admin Console and the Engines.

## 6.1 Admin Console Service

The intended service is:

```text
pingfederateconsole.service
```

The source document specifies:

```ini
[Unit]
Description=PingFederate 9.3.3 Console
Documentation=https://docs.pingidentity.com

[Service]
Type=simple
User=ping9ptnradmin
WorkingDirectory=/apps/partner/pingfederate-9.3.3-admin-ptnr/pingfederate
Environment="JAVA_HOME=/apps/partner/jdk/jdk1.8.0_202"
ExecStart=/apps/partner/pingfederate-9.3.3-admin-ptnr/pingfederate/sbin/pingfederate-run.sh
ExecStop=/apps/partner/pingfederate-9.3.3-admin-ptnr/pingfederate/sbin/pingfederate-shutdown.sh
```

## 6.2 Engine Service

The intended service is:

```text
pingfederateengine.service
```

The source document specifies:

```ini
[Unit]
Description=PingFederate 9.3.3 Engine
Documentation=https://docs.pingidentity.com

[Service]
Type=simple
User=ping9ptnr
WorkingDirectory=/apps/partner/pingfederate-9.3.3-runtime-ptnr/pingfederate
Environment="JAVA_HOME=/apps/partner/jdk/jdk1.8.0_202"
ExecStart=/apps/partner/pingfederate-9.3.3-runtime-ptnr/pingfederate/sbin/pingfederate-run.sh
ExecStop=/apps/partner/pingfederate-9.3.3-runtime-ptnr/pingfederate/sbin/pingfederate-shutdown.sh
```

### Enable / Start

After placing a service file under the systemd unit directory:

```bash
sudo systemctl daemon-reload
sudo systemctl enable <service-name>
sudo systemctl start <service-name>
```

Check status:

```bash
sudo systemctl status <service-name>
```

> **Important:** The original document contains several systemd command/path typos, including `/etc/system/system` and `systemctl ExecStart ...`. The commands above use standard systemd syntax. Verify against your target Linux distribution before production use.

---

# 7. Start and Stop PingFederate

## 7.1 Admin Console — Start

Login as:

```text
ping9ptnradmin
```

Using the configured alias:

```bash
ExecStart
```

Or run directly:

```bash
cd /apps/partner/pingfederate-9.3.3-admin-ptnr/pingfederate/sbin/
./pingfederate-run.sh
```

Expected startup message from the source document:

```text
PingFederate is starting ...
```

## 7.2 Admin Console — Stop

```bash
ExecStop
```

Or:

```bash
cd /apps/partner/pingfederate-9.3.3-admin-ptnr/pingfederate/sbin/
./pingfederate-shutdown.sh
```

Expected message:

```text
PingFederate is shutting down ...
```

---

## 7.3 Engines — Start

Login to the Engine servers as:

```text
ping9ptnr
```

Start using:

```bash
ExecStart
```

Or:

```bash
cd /apps/partner/pingfederate-9.3.3-runtime-ptnr/pingfederate/sbin/
./pingfederate-run.sh
```

## 7.4 Engines — Stop

```bash
ExecStop
```

Or:

```bash
cd /apps/partner/pingfederate-9.3.3-runtime-ptnr/pingfederate/sbin/
./pingfederate-shutdown.sh
```

---

# 8. Cluster Validation

After all nodes are started, validate the cluster from the PingFederate Admin Console.

The source document includes a **Cluster Management** validation screen showing:

- Administrative console node
- Engine node `101`
- Engine node `102`
- Cluster addresses and indexes

![PingFederate Cluster Management](assets/image1.png)

Use this screen to confirm that the expected cluster nodes are visible.

---

# 9. PingFederate Console Settings

The source document specifies the following PreProd base URL and Entity ID configuration.

### Base URL

```text
https://partneridentity-pp.telus.com
```

### SAML 2.0 Entity ID

```text
https://partneridentity-pp.telus.com
```

The source document states that the F5 URL should be added as the PingFederate PreProd base URL and then used as the PreProd Entity ID.

![PingFederate Protocol Settings](assets/image2.png)

---

# 10. Protocol Settings Reference

The supplied validation screenshot shows the following protocol settings:

| Setting | Value |
|---|---|
| Enable OAuth 2.0 Authorization Server | `true` |
| IdP SAML 2.0 Support | `true` |
| SP SAML 2.0 Support | `true` |
| Enable IdP Discovery | `false` |
| Base URL | `https://partneridentity-pp.telus.com` |
| SAML 2.0 Entity ID | `https://partneridentity-pp.telus.com` |

---

# 11. Build Checklist

### Admin Console

- [ ] Confirm server `170.11.196.11` is reachable.
- [ ] Confirm `ping9ptnradmin` access.
- [ ] Install JDK `8u202`.
- [ ] Install PingFederate `9.3.3`.
- [ ] Configure `CLUSTERED_CONSOLE`.
- [ ] Set node index to `100`.
- [ ] Configure Admin HTTPS port `8888`.
- [ ] Configure HTTPS port `6031`.
- [ ] Configure cluster discovery hosts.
- [ ] Configure environment variables.
- [ ] Configure the Admin Console service.
- [ ] Start the Admin Console.

### Engine 1

- [ ] Confirm server `170.11.196.11`.
- [ ] Confirm `ping9ptnr` access.
- [ ] Install JDK `8u202`.
- [ ] Install PingFederate runtime `9.3.3`.
- [ ] Configure `CLUSTERED_ENGINE`.
- [ ] Set node index to `101`.
- [ ] Configure ports `8031` and `8032`.
- [ ] Configure cluster bind port `8600`.
- [ ] Configure failure detection port `8800`.
- [ ] Configure cluster discovery hosts.
- [ ] Configure environment variables.
- [ ] Configure the Engine service.
- [ ] Start Engine 1.

### Engine 2

- [ ] Confirm server `170.11.196.12`.
- [ ] Confirm `ping9ptnr` access.
- [ ] Install JDK `8u202`.
- [ ] Install PingFederate runtime `9.3.3`.
- [ ] Configure `CLUSTERED_ENGINE`.
- [ ] Set node index to `102`.
- [ ] Configure ports `8031` and `8032`.
- [ ] Configure cluster bind port `8600`.
- [ ] Configure failure detection port `8800`.
- [ ] Configure cluster discovery hosts.
- [ ] Configure environment variables.
- [ ] Configure the Engine service.
- [ ] Start Engine 2.

### Validation

- [ ] Confirm all three cluster nodes appear in Cluster Management.
- [ ] Confirm node indexes `100`, `101`, and `102`.
- [ ] Confirm the PreProd Base URL.
- [ ] Confirm the SAML 2.0 Entity ID.
- [ ] Confirm OAuth 2.0 Authorization Server setting.
- [ ] Confirm IdP SAML 2.0 support.
- [ ] Confirm SP SAML 2.0 support.

---

# 12. Source Document Notes

This GitHub page is based on the supplied **PingFederate PreProd Build Documentation v2.0**. The source document contains a few apparent inconsistencies and typographical errors. These should be verified before executing the build in an actual environment.

Examples include:

- `/etc/system/system` appears in the source where a standard systemd unit directory would normally be used.
- The source uses `systemctl ExecStart` / `systemctl ExecStop`; standard systemd commands use actions such as `start` and `stop`.
- Some source paths differ between sections.
- The Engine 1 section contains a server reference that differs from the main topology table.
- The source uses both `jdk1.8.0_202` and `jdk_2020` in different configuration snippets.
- The source contains several spelling/formatting errors such as `Distrubution`, `PinhgFederate`, and `PigFederate`.

These have been called out rather than silently treating them as authoritative values.

---

# 13. Quick Reference

## Cluster Nodes

```text
Console : 170.11.196.11 / Node 100
Engine1 : 170.11.196.11 / Node 101
Engine2 : 170.11.196.12 / Node 102
```

## PingFederate

```text
Version : 9.3.3
Java    : JDK 8u202
```

## Ports

```text
Admin HTTPS       : 8888
Console HTTPS     : 6031

Engine HTTPS      : 8031
Engine Secondary  : 8032

Engine Cluster    : 8600
Engine Failure    : 8800

Console Cluster   : 8700
Console Failure   : 8900
```

## PreProd URL

```text
https://partneridentity-pp.telus.com
```

---

## Disclaimer

This README is a documentation conversion of the supplied build document, not an independent validation of the installation procedure. Confirm software compatibility, operating-system requirements, network/firewall rules, credentials, certificates, and Ping Identity product documentation before applying the procedure to a live environment.
