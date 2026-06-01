# LLDAP Client side
## Package
```
pacman -S sssd openldap
```
## OpenLDAP Domain Component (DC) Configuration
###Understanding Domain Components (DC)

In OpenLDAP, Domain Components (DC) are used to define the directory structure based on an organization's domain name. The domain name is divided into multiple dc attributes, which together form the Base Distinguished Name (Base DN) of the LDAP directory.

For example, if the domain name is:

`local.test`

The corresponding Domain Component structure will be:

`dc=local,dc=test`

This structure serves as the root of the LDAP directory tree, under which users, groups, and other directory objects are stored.

OpenLDAP Client Configuration

Open the OpenLDAP client configuration file:
```
sudo nvim /etc/openldap/ldap.conf
```
Example configuration:
```
#
# LDAP Defaults
#

# See ldap.conf(5) for details
# This file should be world readable but not world writable.
#BASE	dc=example,dc=com
#URI	ldap://ldap.example.com ldap://ldap-provider.example.com:666

#SIZELIMIT	12
#TIMELIMIT	15
#DEREF		never

BASE dc=yuros,dc=org
URI ldap://member.yuros.org:11003

```

The BASE parameter specifies the default Base Distinguished Name (Base DN) used for LDAP searches. All directory entries, including users, groups, and organizational units, are located within this domain hierarchy.
 
In this example, the LDAP directory is organized under the domain:

`yuros.org`

which translates to:

`dc=yuros,dc=org`
URI
`URI ldap://member.yuros.org:11003`

The URI parameter defines the LDAP server endpoint that clients will connect to. It includes the protocol, server address, and listening port.

## NSS (Name Service Switch) Configuration
Name Service Switch (NSS) is a Linux subsystem that determines where the operating system retrieves information such as user accounts, groups, hostnames, and network services.

When integrating LDAP through SSSD, NSS must be configured to allow the system to query both local files and the LDAP directory service.

Editing the NSS Configuration

Open the NSS configuration file:
```
sudo nvim /etc/nsswitch.conf
```
Example configuration:
```
# Name Service Switch configuration file.
# See nsswitch.conf(5) for details.

passwd: files systemd sss
group: files systemd sss
shadow: files sss
gshadow: files systemd

publickey: files

hosts: mymachines resolve [!UNAVAIL=return] files myhostname dns
networks: files

protocols: files
services: files
ethers: files
rpc: files

netgroup: files
```


## SSSD (System Security Services Daemon) Configuration
SSSD (System Security Services Daemon) provides centralized access to identity and authentication services such as LDAP, Active Directory, and Kerberos.

In this setup, SSSD is used to connect a Linux system to an LDAP server, allowing LDAP users and groups to be recognized and authenticated by the operating system.

Create the Configuration Directory and File

Create the SSSD configuration directory:
```
sudo mkdir -p /etc/sssd/
```
Create the configuration file:
```
sudo touch /etc/sssd/sssd.conf
```
Edit the SSSD Configuration

Open the configuration file:
```
sudo nvim /etc/sssd/sssd.conf
```
Add the following configuration:
```
services = nss, pam
domains = lldap
debug_level = 9

[domain/lldap]
id_provider = ldap
auth_provider = ldap

ldap_uri = ldap://member.yuros.org:11003
ldap_search_base = dc=yuros,dc=org

ldap_user_search_base = ou=people,dc=yuros,dc=org
ldap_group_search_base = ou=groups,dc=yuros,dc=org

ldap_schema = rfc2307bis

ldap_default_bind_dn = uid=sultan,ou=people,dc=yuros,dc=org
ldap_default_authtok_type = password
ldap_default_authtok = 15111511

ldap_auth_disable_tls_never_use_in_production = true
ldap_id_use_start_tls = false
ldap_tls_reqcert = never

cache_credentials = true

fallback_homedir = /home/%u
default_shell = /bin/bash
```
### LDAP Connection
```
`ldap_uri = ldap://[ip_server]:[port ldap on server]`
`ldap_search_base = dc=local,dc=test`
```
`ldap_uri` specifies the LDAP server endpoint.
`ldap_search_base` defines the root directory used for LDAP searches.

### User and Group Search Bases
```
`ldap_user_search_base = ou=people,dc=local,dc=test`
`ldap_group_search_base = ou=groups,dc=local,dc=test`
```
User accounts are searched under the people organizational unit.
Group entries are searched under the groups organizational unit.
### LDAP Bind Account
```
ldap_default_bind_dn = uid=[username],ou=people,dc=local,dc=test
ldap_default_authtok_type = password
ldap_default_authtok = [password]
```
These settings specify the account used by SSSD to bind and perform LDAP directory searches.

## Securing the SSSD Configuration File

The SSSD configuration file contains sensitive information such as LDAP bind credentials and authentication settings. To prevent unauthorized access, the file ownership and permissions must be properly configured.

Set File Ownership

Assign ownership of the configuration file to the root user:
```
sudo chown root:root /etc/sssd/sssd.conf
```
This ensures that only the root user and root group own the file.

Set File Permissions

Restrict access to the configuration file:
```
sudo chmod 600 /etc/sssd/sssd.conf
```
Permission 600 means: root can read write and accsess, group cant accsess, other cant accsess
