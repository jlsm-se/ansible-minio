<p><img src="https://avatars0.githubusercontent.com/u/695951?s=200&v=4" alt="minio logo" title="minio" align="right" height="60" /></p>

# Ansible Role: MinIO

[![Build Status](https://travis-ci.org/atosatto/ansible-minio.svg?branch=master)](https://travis-ci.org/atosatto/ansible-minio)
[![License](https://img.shields.io/badge/license-MIT%20License-brightgreen.svg)](https://opensource.org/licenses/MIT)
[![Ansible Role](https://img.shields.io/badge/ansible%20role-atosatto.minio-blue.svg)](https://galaxy.ansible.com/atosatto/minio/)
[![GitHub tag](https://img.shields.io/github/tag/atosatto/ansible-minio.svg)](https://github.com/atosatto/ansible-minio/tags)

Install and configure the [MinIO](https://minio.io/) S3 compatible object storage server on RHEL/CentOS and Debian/Ubuntu.

## Requirements

None.

## Role Variables

Available variables are listed below, along with default values (see `defaults/main.yml`):

```yaml
minio_server_bin: /usr/local/bin/minio
minio_client_bin: /usr/local/bin/mc
```

Installation path of the MinIO server and client binaries.

```yaml
minio_server_release: "RELEASE.2022-08-25T07-17-05Z"
minio_client_release: "RELEASE.2022-08-23T05-45-20Z"
```

Release to install for both server and client; lastest if the default. Can be 'RELEASE.2022-07-26T00-53-03Z' for instance.

```yaml
minio_user: minio
minio_group: minio
```

Name and group of the user running the minio server.
**NB**: This role automatically creates the minio user and/or group if they do not exist in the system.

```yaml
minio_server_envfile: /etc/default/minio
```

Path to the file containing the minio server configuration ENV variables.

```yaml
minio_server_addr: ":9091"
```

The MinIO server listen address.

```yaml
minio_server_datadirs:
  - /var/lib/minio
```

Directories of the folder containing the minio server data

```yaml
minio_server_make_datadirs: true
```

Create directories from `minio_server_datadirs`

```yaml
minio_server_args: [ ]
```

Set a list of nodes to create a [distributed cluster](https://docs.minio.io/docs/distributed-minio-quickstart-guide).

In this mode, ansible will create your server datadirs, but use this list for the server startup. Note you will need a number of disks to satisfy MinIO's distributed storage requirements.

Example:

```yaml
minio_server_datadirs:
  - '/minio-data'

minio_server_args:
  - 'https://server{1...4}/minio-data'
```

Additional environment variables to be set in MinIO server environment

```yaml
minio_server_env_extra: |
  MINIO_REGION_NAME=us-east-1
  MINIO_API_REQUESTS_MAX=1600
```

Additional CLI options that must be appended to the minio server start command.

```yaml
minio_server_opts: ""
```


MinIO root user and password.

```yaml
minio_root_username: ""
minio_root_password: ""
```

Switches to disable minio server and/or minio client installation.
```yaml
minio_install_server: true
minio_install_client: true
```

### TLS

```yaml
minio_enable_tls: false
```
Turn on or off TLS support. This requires also configuring TLS certificates, but if you just want TLS and don't care about serving multiple domains (SNI) or about proper handling of the private keys, just set `minio_enable_tls: true` and fill out the `minio_tls_public_cert` and `minio_tls_private_key` variables accordingly (you may want to use e.g. [certgen](https://github.com/minio/certgen/) to generate a certificate if you don't have one already).

If you need to serve multiple domains (SNI), you can put the contents of your PEM-encoded certificate files and keys in the Ansible variables documented below, or you can opt to have this role create symlinks to certificate files managed outside of this role. The latter is useful when using automatically renewing certificates such as those from the LetsEncrypt CA.

If you need to specify custom certificate authorities (CAs) you also have a similar choice of storing them in Ansible variables or create symlinks - or both.

It's possible to mix and match these two methods for any combination of certificates/keys/CAs, but keep in mind that for private keys stored in Ansible variables, you should probably use something like [Ansible Vault](https://docs.ansible.com/ansible/latest/cli/ansible-vault.html).

Refer to the MinIO TLS documentation for more information: https://min.io/docs/minio/linux/operations/network-encryption.html

```yaml
minio_certs_dir: "/etc/minio/certs"
```
Directory to store MinIO certificates.

```yaml
minio_certs_dir_basedir_owner: root
```
Owner of the directory above the certs dir (default location /etc/minio)

```yaml
minio_certs_dir_owner: "{{ minio_user }}"
```
Owner of the certs directory. It's recommended to change this to root (or whichever user manages the renewal of certs) in production.

```yaml
minio_certs_owner: "{{ minio_user }}"
```
Owner of the files in the certs directory. It's recommended to change this to root (or whichever user manages the renewal of certs) in production

```yaml
minio_tls_directory_permissions: '0750'
minio_tls_file_permissions: '0640'
```

File and directory permissions for certificates stored under `minio_certs_dir`. There is typically no need to change these and if there is, it is usually to just remove the owner's write permissions on the files.

```yaml
minio_tls_hostnames: []
```
Hostnames to set up for multiple domain-based TLS certificates (SNI). Must match the hostnames in `minio_hostspecific_*` vars (or those part of the paths specified in in externally managed certs)

```yaml
minio_tls_public_cert: ""
```
If you only want to use a single certificate regardless of domain, this is the variable to put the (public) certificate in.
```yaml
minio_tls_private_key: ""
```
If you only want to use a single certificate regardless of domain, this is the variable to put the private key for it it in.

```yaml
minio_tls_hostspecific_public_certs:
  - hostname: example1.lan
    content: |
      -----BEGIN CERTIFICATE-----
      .....
      -----END CERTIFICATE-----
  - hostname: example2.lan
    content: |
      -----BEGIN CERTIFICATE-----
      .....
      -----END CERTIFICATE-----
```
If you want MinIO to serve content over TLS to multiple domains (hostnames) and use Server Name Indication to answer with different certificates for different domains, this is the variable to put the (public) certificates in and map them to the correct domain (hostname). Any hostnames specified here must also be present in the `minio_tls_hostnames` list.
```yaml
minio_tls_hostspecific_private_keys:
  - hostname: example1.lan
    content: |
      -----BEGIN PRIVATE KEY-----
      .....
      -----END PRIVATE KEY-----
  - hostname: example2.lan
    content: |
      -----BEGIN PRIVATE KEY-----
      .....
      -----END PRIVATE KEY-----
```
If you want MinIO to serve content over TLS to multiple domains (hostnames) and use Server Name Indication to answer with different certificates for different domains, this is the variable to put the (private) keys in and map them to the correct domain (hostname). Any hostnames specified here must also be present in the `minio_tls_hostnames` list.

```yaml
minio_tls_cacerts:
  - filename: ca1.crt
    content: |
      -----BEGIN CERTIFICATE-----
      .....
      -----END CERTIFICATE-----
  - filename: ca2.crt
    content: |
      -----BEGIN CERTIFICATE-----
      .....
      -----END CERTIFICATE-----
```

This is for putting CA certs in the `CAs` subdirectory of `minio_certs_dir`. If your certificate(s)
are signed by certificate authorities that the system doesn't already trust, you'll want to put them
here.

```yaml
minio_tls_externally_managed_certs: []
  - crt_src: /usr/local/share/minio/certs/example1.lan.crt
    key_src: /usr/local/share/minio/private/example1.lan.key
    crt_dst: example1.lan/public.crt
    key_dst: example1.lan/private.key
  - crt_src: /usr/local/share/minio/certs/example2.lan.crt
    key_src: /usr/local/share/minio/private/example2.lan.key
    crt_dst: example2.lan/public.crt
    key_dst: example2.lan/private.key
```

If you want to use certificate files (and corresponding private key files) that are stored outside the `minio_certs_dir` (such as if they are managed by something other than this role), you can use this variable to specify a mapping and the role will then create the appropriate symlinks. Just make sure the source files are accessible to the user/group specified in `minio_user`/`minio_group`.

```yaml
minio_tls_externally_managed_cacerts: []
  - ca_src: /usr/local/share/minio/CAs/ca1.crt
    ca_dst: ca1.crt
  - ca_src: /usr/local/share/minio/CAs/ca2.crt
    ca_dst: ca2.crt
```
Like the `minio_tls_externally_managed_certs` variable, this is just for setting up symlinks to externally managed CA certificates located outside the `minio_certs_dir`.

## Dependencies

None.

## Example Playbook

```yaml
- name: "Install MinIO"
  hosts: all
  become: yes
  roles:
    - { role: minio }
  vars:
    minio_server_datadirs: [ "/minio1" ]
```

## License

MIT
