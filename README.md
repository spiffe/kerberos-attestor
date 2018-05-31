Kerberos-Attestor
==

Overview
--

The Kerberos-Attestor is a plugin for the [SPIRE][spire] server and agent that allows SPIRE to automatically attest nodes that are joined to a domain backed by the [Kerberos authentication protocol][kerberos].  SPIRE is an open-source implementation of the [SPIFFE][spiffe], which is a set of standards to provide authentication and trust to disparate micro-services operating in heterogeneous cloud-native environments.  The predominant on-premise authentication protocol is Kerberos through Active Directory, and with the Kerberos-Attestor, environments backed by SPIRE can provide trust leveraging existing enterprise identity stacks.

Base SVID SPIFFE ID Format
--

An agent attested by the Kerberos-Attestor plugin will have a base SVID SPIFFE ID in this format:

  `spiffe://<trust_domain>/spire/agent/kerberos_attestor/<KRB_REALM>/<agent_fqdn>`

Pre-Requisites
--

These instructions require a running SPIRE server and agent each on a [PhotonOS 2.0][photonos] host.  Both hosts should also be domain joined or promoted as a domain controller using [Project Lightwave][lightwave].

**Pre-Requisite Installation Guides**

* Follow the [PhotonOS Download Guide][photonos-download] to learn how to obtain the OS and set it up
* Follow the [Project Lightwave README][lightwave-readme] to learn how to install and configure Lightwave, promote domain controllers, and join clients to the domain on PhotonOS 2.0
* Follow [SPIRE README][spire-readme] to learn how to install and configure both the SPIRE Server and Agent

Compilation
--

There are two ways to get the plugin--using `go install` to build and install it or alternatively, build it from source.

**Go Install**

Running the following commands will download, build, and install the Kerberos-Attestor server and agent in your `${GOPATH}/bin` directory by default, or in the path set by the `${GOBIN}` environment variable.

* Server:
  * `go install github.com/spiffe/kerberos-attestor/server`
* Agent:
  * `go install github.com/spiffe/kerberos-attestor/agent`

**Build from Source**

1. Clone this repo:

  ```bash
  git clone https://github.com/spiffe/kerberos-attestor ${GOPATH}/src/github.com/spiffe/kerberos-attestor
  cd ${GOPATH}/src/github.com/spiffe/kerberos-attestor
  ```

2. Install utilities such as [Glide][glide]:

  ```bash
  make utils
  ```

3. Install dependencies:

  ```bash
  glide up
  ```

4. Build the Kerberos-Attestor:

  ```bash
  make build
  ```

5. Binaries for the server and agent should be in the `bin/` directory

Installation and Configuration
--

**Kerberos-Attestor Server Plugin**

1. Edit the SPIRE Server config file to add the Kerberos-Attestor server plugin config:

  ```bash
  vim <SPIRE Installation Directory>/conf/server/server.conf
  ```

2. Add the following [HCL][hcl] blob to the "plugins" section of the config file:

  ```bash
  NodeAttestor "kerberos_attestor" {
      plugin_cmd = "${GOPATH}/src/github.com/spiffe/kerberos-attestor/bin/server"
      enabled = true
      plugin_data {
          krb_realm = "LIGHTWAVE.LOCAL"
          krb_conf_path = "/etc/krb5.conf"
          krb_keytab_path = "/etc/krb5.keytab"
      }
  }
  ```
  * Replace `plugin_cmd` with the path to the Kerberos-Attestor server binary compiled earlier
  * Replace `krb_realm` with the domain that you promoted when configuring Lightwave in _all caps_
  * `krb_conf_path` and `krb_keytab_path` point to the default paths to the Kerberos config file and Keytab that are created during Lightwave promotion/join.  _Do not_ modify these unless you are using a different Kerberos provider, or have changed the default paths for your own purposes

**Kerberos-Attestor Agent Plugin**

1. Edit the SPIRE Agent config file to add the Kerberos-Attestor agent plugin config:

  ```bash
  vim <SPIRE Installation Directory>/conf/agent/agent.conf
  ```

2. Add the following [HCL][hcl] blob to the "plugins" section of the config file:

  ```bash
  NodeAttestor "kerberos_attestor" {
      plugin_cmd = "${GOPATH}/src/github.com/spiffe/kerberos-attestor/bin/agent"
      enabled = true
      plugin_data {
          krb_realm = "LIGHTWAVE.LOCAL"
          krb_conf_path = "/etc/krb5.conf"
          krb_keytab_path = "/etc/krb5.keytab"
          server_fqdn = "<FQDN of SPIRE Server>"
      }
  }
  ```
  * Replace `plugin_cmd` with the path to the Kerberos-Attestor server binary compiled earlier
  * Replace `krb_realm` with the domain that you promoted when configuring Lightwave in _all caps_
  * `krb_conf_path` and `krb_keytab_path` point to the default paths to the Kerberos config file and Keytab that are created during Lightwave promotion/join.  _Do not_ modify these unless you are using a different Kerberos provider, or have changed the default paths for your own purposes
  * Replace `server_fqdn` with the FQDN of the SPIRE Server.  This needs to be in FQDN format (without the final `.`).  For example, `spire-server.lightwave.local`

3. Remove Join-Token NodeAttestor plugin config from this file as a SPIRE Agent can only use one NodeAttestor plugin at-a-time

Start SPIRE with Kerberos-Attestor plugins
--

**SPIRE Server**

```bash
cd <SPIRE Installation Directory>
./spire-server run
```

**SPIRE Server**

```bash
cd <SPIRE Installation Directory>
./spire-agent run
```


[spire]: https://github.com/spiffe/spire
[spire-readme]: https://github.com/spiffe/spire/blob/master/README.md
[spiffe]: https://github.com/spiffe/spiffe
[lightwave]: https://github.com/vmware/lightwave/
[lightwave-readme]: https://github.com/vmware/lightwave/blob/dev/README.md
[photonos]: https://github.com/vmware/photon
[photonos-download]: https://github.com/vmware/photon/wiki/Downloading-Photon-OS
[kerberos]: https://en.wikipedia.org/wiki/Kerberos_(protocol)
[go]: https://golang.org/
[go-install]: https://golang.org/doc/install
[glide]: https://github.com/Masterminds/glide
[hcl]: https://github.com/hashicorp/hcl
