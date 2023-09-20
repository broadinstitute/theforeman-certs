#### Table of Contents

1. [Overview](#overview)
    * [What certs affects](#what-certs-affects)
2. [Reference - An under-the-hood peek at what the module is doing and how](#reference)
3. [Development - Guide for contributing to the module](#development)

## Overview

This module is responsible for generating a CA and certificate used
for communication between services inside the Katello deployment.

### What certs affects

* Installs and deploys a CA
* Deploys certificates generated from the CA

## Reference

* **default CA** - a CA generated by the installer used by the installer
* **server CA** - CA used for issuing the server certificates and it's
    used to verify the server identity; when not specified otherwise,
    the ``default CA`` is used
* **puppet CA** - a CA controlled by Puppet for Puppet Agents
    authentication (not covered by this module)

# Certificates overview

| cert                          | purpose                                                                                    | CA      |
|-------------------------------|--------------------------------------------------------------------------------------------|---------|
| ${hostname}-apache            | a server certificate for Apache https                                                      | server  |
| ${hostname}-foreman-proxy     | a server certificate for Foreman-proxy https                                               | server  |
| ${hostname}-foreman-client    | a client certificate for Foreman -> Foreman-proxy communication                            | default |
| ${hostname}-puppet-client     | a client certificate for Puppet ENC -> Foreman communication                               | default |

# Phases

The certificates are configured in three phases:

1. **generation** - producing a certificate; in this phase, the
`$generate` parameter of the `cert` resources is set to `true`
2. **deployment** - installing a certificate into a system that will
use it; in this phase, a `$deploy` parameter of the `cert` resources
is set to `true`; this allows to generate the certificates on one
machine while deploying on another
3. **configuration** - placing a files with keys to specific locations
where the services will be configured to read them from, using the
`pubkey`, `privkey` and `key_bundle` types, the certs need to be
generated and deployed on given system before being able to use it

# Types and providers

There is a set of custom Puppet types defined for defining the
cert-specific resources:

* **ca** - represents an authority that can be used for issuing
    certificates
* **cert** - represents a certificate, the CA of the certs is
    specified by a `ca` property, where the keys are stored should be
    might be implementation specific and ``pubkey`` and ``privkey``
    should be used for using the cert keys
* **pubkey** - a file to copy a public key of a ``cert`` to. It
    produces event on subscribed resources when a certificate changes
    (useful for restarting a service when the certificate changes)
* **privkey** - a file to copy a private key of a ``cert`` to. It
    produces event on subscribed resources when a certificate changes
    (useful for restarting a service when the certificate changes)
* **key_bundle** - a file to copy both public and private key of a
    cert.

For now, the only implemented provider of the type is
`katello_ssl_tool`. It works as follow:

1. **generation** - the artefact of this phase is an RPM with the keys
for the certificate; the RPMs, as well as other files generated in the
process, are located in `/root/ssl-build` directory

2. **deployment** - installing the RPMs into the system; the
certificates are located in `/etc/pki/katello-certs-tools/` directory

## Development

See the CONTRIBUTING guide for steps on how to make a change and get it accepted upstream.
