# 389 Directory Server

## CREATE FILE INSTANCE

````
# /root/instance.inf
[general]
config_version = 2

[slapd]
root_password = pass1234

[backend-userroot]
sample_entries = yes
suffix = dc=inno,dc=local
````

## CREATE INSTANCE COMMAND

````
dscreate from-file /root/instance.inf
## OR
dscreate -v from-file /root/instance.inf
````


## CHECK STATUS
````
dsctl localhost status
````


## FOR CLIENT

````
nano ~/.dsrc

# cat ~/.dsrc
[localhost]
uri = ldaps://localhost
basedn = dc=inno,dc=local
binddn = cn=Directory Manager
# You need to copy /etc/dirsrv/slapd-localhost/ca.crt to your host for this to work.
# Then run /usr/bin/c_rehash /etc/openldap/certs
tls_cacertdir = /etc/openldap/certs/
````

## FOR SERVER

````
nano ~/.dsrc

# cat ~/.dsrc
[localhost]
# Note that '/' is replaced to '%%2f'.
uri = ldapi://%%2fvar%%2frun%%2fslapd-localhost.socket
basedn = dc=inno,dc=local
binddn = cn=Directory Manager
````


## ADD USER
````
dsidm localhost user create --uid sadmin --cn sadmin --displayName 'sadmin User' \
--uidNumber 1000 --gidNumber 1000 --homeDirectory /home/sadmin
````

## GET USER sadmin
````
dsidm localhost user get sadmin
````

### result
````
[root@fedora-ansible ~]# dsidm localhost user get sadmin
dn: uid=sadmin,ou=people,dc=inno,dc=local
cn: sadmin
displayName: sadmin User
gidNumber: 1000
homeDirectory: /home/sadmin
objectClass: top
objectClass: nsPerson
objectClass: nsAccount
objectClass: nsOrgPerson
objectClass: posixAccount
uid: sadmin
uidNumber: 1000

[root@fedora-ansible ~]#
````

## RESET PASSWORD
````
[root@fedora-ansible ~]# dsidm localhost account reset_password uid=sadmin,ou=people,dc=inno,dc=local
Enter new password for uid=sadmin,ou=people,dc=inno,dc=local :
CONFIRM - Enter new password for uid=sadmin,ou=people,dc=inno,dc=local :
reset password for uid=sadmin,ou=people,dc=inno,dc=local
[root@fedora-ansible ~]#
````

## CREATE GROUP

````
[root@fedora-ansible ~]# dsidm localhost group create
Enter value for cn : serveradmin
Successfully created serveradmin
[root@fedora-ansible ~]#
````

## ADD USER TO GROUP

````
[root@fedora-ansible ~]# dsidm localhost group add_member serveradmin uid=sadmin,ou=people,dc=inno,dc=local
added member: uid=sadmin,ou=people,dc=inno,dc=local
[root@fedora-ansible ~]# dsidm localhost group add_member serveradmin uid=wachira,ou=people,dc=inno,dc=local
added member: uid=wachira,ou=people,dc=inno,dc=local
[root@fedora-ansible ~]#
````


## TEST CONNECT 
````
[root@fedora-ansible ~]# ldapwhoami -H ldaps://localhost -D uid=sadmin,ou=people,dc=inno,dc=local -W -x
Enter LDAP Password:
ldap_sasl_bind(SIMPLE): Can't contact LDAP server (-1)
[root@fedora-ansible ~]# LDAPTLS_CACERT=/etc/dirsrv/slapd-localhost/ca.crt ldapwhoami -H ldaps://localhost -D uid=sadmin,ou=people,dc=inno,dc=local -W -x
Enter LDAP Password:
dn: uid=sadmin,ou=people,dc=inno,dc=local
[root@fedora-ansible ~]#
````

#### To make this permanent, put 'TLS_CACERT /etc/dirsrv/slapd-localhost/ca.crt' into '/etc/openldap/ldap.conf'

## ADD DESCRIPTION 'sadmin administrator user'
````
dsidm localhost user modify sadmin "add:description:sadmin administrator user"

dsidm localhost user get sadmin
````


