# Introduction #

This is the ldap server, which I used to connect with gerrit server or docker registry

The `Dockerfile` & ldap schema files are copied from https://github.com/rackerlabs/dockerstack/blob/master/keystone/openldap/Dockerfile

The own sample user data `files/more.ldif` is referred to http://www.zytrax.com/books/ldap/ch5/ 

# Install and Start #

    $ docker pull larrycai/openldap
    $ docker run -d -p 389:389 --name ldap -t larrycai/openldap
    $ docker ps
    user@ubuntu:/mnt/git/docker-gerrit/tmp$ docker ps
    CONTAINER ID        IMAGE                      COMMAND                CREATED             STATUS              PORTS                    NAMES
    4248037c0ab6        larrycai/openldap:latest   /bin/sh -c 'slapd -h   22 seconds ago      Up 21 seconds       0.0.0.0:63389->389/tcp   ldap

## Verify the data inside the ldap database ##

Use `ldapsearch` to check the data, 

    $ docker exec -it ldap bash
	# ldapsearch -H ldap://localhost -LL -b ou=Users,dc=openstack,dc=org -x
	version: 1

	dn: ou=Users,dc=openstack,dc=org
	objectClass: organizationalUnit
	ou: Users

	dn: cn=Robert Smith,ou=Users,dc=openstack,dc=org
	objectClass: inetOrgPerson
    .....

## Important data ##

The admin user/passwd and BaseDN list below

    LDAP username                  : cn=admin,dc=openstack,dc=org
    cn=admin,dc=openstack,dc=org's password : password
    Account BaseDN                 [DC=168,DC=56,DC=153:49154]: ou=Users,dc=openstack,dc=org
    Group BaseDN                   [ou=Users,dc=openstack,dc=org]:

### Gerrit integration ###    
If it is configured in gerrit, please update `etc/gerrit.cfg`, `192.168.59.103` is my boot2docker ip address.  

    [auth]
        type = LDAP
    [ldap]
        server = ldap://192.168.59.103
        username = cn=admin,dc=openstack,dc=org
        accountBase = ou=Users,dc=openstack,dc=org
        groupBase = ou=Users,dc=openstack,dc=org
        accountPattern = (&(objectClass=inetOrgPerson)(uid=${username}))
        accountFullName = ${cn}

### Nginx integration ###

See sample in https://github.com/larrycai/nginx-registry, key segment like below. (`ldap` is the ldap server url)

    ldap_server ldap1 {
       url ldap://ldap:389/ou=Users,dc=openstack,dc=org?uid?sub?(objectClass=inetOrgPerson);
       group_attribute uniquemember;
       group_attribute_is_dn on;
       require valid_user;
    }

# Customize your own data #

You can create for your own by checking `files/more.ldif`

    dn: cn=Larry Cai,ou=Users,dc=openstack,dc=org
    objectclass: inetOrgPerson
    cn: Larry Cai
    sn: Cai
    uid: larrycai
    userpassword: LarryCai
    carlicense: HISCAR 123
    homephone: 555-111-2222
    mail: larry.caiyu@gmail.com
    description: hacker guy
    ou: Development Department  

The file will be added by command

    ldapadd -x -D cn=admin,dc=openstack,dc=org -w password -c -f more.ldif

