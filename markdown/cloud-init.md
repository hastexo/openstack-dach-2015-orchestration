Nova
## user data


```sh
nova boot \
  --image trusty-server-cloudimg-amd64 \
  --key_name mykey \
  --flavor m1.small \
  --user-data userdata.txt \
  --nic net-id=4f0dcc21-4b6c-47db-b283-591fdb9aa5a7 \
  test0
```


# This
is what user-data
## typically
looks like:


```sh
#!/bin/sh -e

# Frobnicate a newly booted box

initialize_box
for foo in frobnications; do
  frobnicate_machine $foo || break
done

exit $?
```
# Nope <!-- .element class="fragment stamp" -->


You can do
# better


Enter
## `cloud-config`


## `cloud-config`
enables you to
## bootstrap
a newly booted VM


`cloud-config` is 100%
# YAML


`cloud-config` is OpenStack's most
## underrated
feature


`cloud-config` is Ubuntu's most
## underrated
feature


What can we
# do
with `cloud-config`?


### `package_update`
### `package_upgrade`
Update system on first boot


```yaml
#cloud-config

package_update: true
package_upgrade: true
```


# `users`
Configure users and groups


```yaml
users:
- default
- name: foobar
  gecos: "Fred Otto Oscar Bar"
  groups: users,adm
  lock-passwd: false
  passwd: $6$rounds=4096$J86aZz0Q$To16RGzWJku0
  shell: /bin/bash
  sudo: "ALL=(ALL) NOPASSWD:ALL"
```


## `ssh_pwauth`
Enable/disable SSH password authentication


```
ssh_pwauth: true
```


## `write_files`
Write arbitrary files


```yaml
write_files:
- path: /etc/hosts
  permissions: '0644'
  content: |
    127.0.0.1 localhost
    ::1       ip6-localhost ip6-loopback
    fe00::0   ip6-localnet
    ff00::0   ip6-mcastprefix
    ff02::1   ip6-allnodes
    ff02::2   ip6-allrouters

    192.168.122.100 deploy.example.com deploy
    192.168.122.111 alice.example.com alice
    192.168.122.112 bob.example.com bob
    192.168.122.113 charlie.example.com charlie
```


# `puppet`
Configure a VM's Puppet agent


```yaml
puppet:
 conf:
   agent:
     server: "puppetmaster.example.org"
     certname: "%i.%f"
   ca_cert: |
     -----BEGIN CERTIFICATE-----
     MIICCTCCAXKgAwIBAgIBATANBgkqhkiG9w0BAQUFADANMQswCQYDVQQDDAJjYTAe
     Fw0xMDAyMTUxNzI5MjFaFw0xNTAyMTQxNzI5MjFaMA0xCzAJBgNVBAMMAmNhMIGf
     MA0GCSqGSIb3DQEBAQUAA4GNADCBiQKBgQCu7Q40sm47/E1Pf+r8AYb/V/FWGPgc
     b014OmNoX7dgCxTDvps/h8Vw555PdAFsW5+QhsGr31IJNI3kSYprFQcYf7A8tNWu
     SIb3DQEBBQUAA4GBAH/rxlUIjwNb3n7TXJcDJ6MMHUlwjr03BDJXKb34Ulndkpaf
     +GAlzPXWa7bO908M9I8RnPfvtKnteLbvgTK+h+zX1XCty+S2EQWk29i2AdoqOTxb
     hppiGMp0tT5Havu4aceCXiy2crVcudj3NFciy8X66SoECemW9UYDCb9T5D0d
     -----END CERTIFICATE-----
```


# `chef`
Configure a VM's Chef client


```yaml
chef:
 install_type: "packages"
 force_install: false
 server_url: "https://chef.yourorg.com:4000"
 node_name: "your-node-name"
 environment: "production"
 validation_name: "yourorg-validator"
 validation_key: |
     -----BEGIN RSA PRIVATE KEY-----
     YOUR-ORGS-VALIDATION-KEY-HERE
     -----END RSA PRIVATE KEY-----
 run_list:
  - "recipe[apache2]"
  - "role[db]"
 initial_attributes:
    apache:
      prefork:
        maxclients: 100
      keepalive: "off"
```


# `packages`
Install packages


```yaml
packages:
  - ansible
  - git
```


Running
## arbitrary commands


# `bootcmd`
Run commands early in the boot sequence


```yaml
bootcmd:
- ntpdate pool.ntp.org
```


# `runcmd`
Run commands late in the boot sequence


```yaml
runcmd:
  - >
    sudo -i -u training
    ansible-pull -v -i hosts 
    -U https://github.com/hastexo/academy-ansible 
    -o site.yml
```