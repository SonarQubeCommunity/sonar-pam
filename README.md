PAM Authentication Plugin
=========================
[![Build Status](https://api.travis-ci.org/SonarQubeCommunity/sonar-pam.svg)](https://travis-ci.org/SonarQubeCommunity/sonar-pam)

This plugin enables the delegation of SonarQube authentication to underlying PAM subsystem. 
The plugin works on *nix boxes with the Pluggable Authentication Module (PAM).
Only password-checking is done against PAM. Authorization (access control) is still fully managed in SonarQube. 
During the first authentication trial, if the password is correct, the SonarQube database is automatically populated with the new user.  The System administrator should assign the user to the desired groups in order to grant him necessary rights. If a password exists in the SonarQube database, it will be ignored because the external system password will override it.

Download and Version information: http://update.sonarsource.org/plugins/pam-confluence.html

Requirements
============

| OS and Architecture | Works |
|---------------------|-------|
| Linux AMD64 | Yes, tested |
| Linux i386 | Yes, tested |
| Mac OS X PPC | Yes, not tested |
| Solaris sparc | Yes, not tested |
| Windows all flavours | No |

Usage & Installation
====================

1. Install jpam
 1. Download jpam for your system from http://jpam.sourceforge.net/
 2. Alternatively:
   * Copy the jpam's native library following http://jpam.sourceforge.net/documentation/getting_started.html
    * Or check `java.library.path` in `settings->system` info page anche copy jpam's native library to one of those directorys
2. Install the plugin through the Update Center or download it into the `SONARQUBE_HOME/extensions/plugins` directory
3. Make sure that at least one user with global administration role exists in SonarQube as well as in the external system
4. Update the `SONARQUBE_HOME/conf/sonar.properties` file by adding the following lines:
```
sonar.properties
sonar.security.realm: PAM
pam.serviceName=system-auth
# Automatically create users.
# When set to true, user will be created after successful authentication, if doesn't exists.
# The default group affected to new users can be defined online, in SonarQube general settings. The default value is "sonar-users".
# Default is false.
sonar.authenticator.createUsers: true
```

Restart SonarQube and check logs for:
```
2012.11.24 20:32:34 INFO  org.sonar.INFO  Security realm: PAM
2012.11.24 20:32:34 INFO  org.sonar.INFO  Security realm started
```

Now you can log in to SonarQube.


Technical Users
===============
Since SonarQube 4.2, technical users can be set. Technical users are authenticated against SonarQube's own database of users, rather than against any external tool (LDAP, Active Directory, Crowd, etc.).
Similarly, all accounts not flagged as local will be authenticated only against the external tool. By default `admin` is a technical account. Technical accounts are configured in `SONARQUBE_HOME/conf/sonar.properties` in the `sonar.security.localUsers` (`default value = admin`) property as a comma-separated list.

Known Issues
============
Crash using PAM winbind authentication (`pam_winbind.so`)
In case of an unsucessful login for a bad password or a locked account (a bad username does not produce the same issue) you may get this kind of error while using pam winbind authentication:
```
pam_winbind.so error
INFO   | jvm 1    | 2011/03/18 10:06:10 | *** glibc detected *** java: free(): invalid pointer: 0x00002aaadc000168 ***
INFO   | jvm 1    | 2011/03/18 10:06:10 | ======= Backtrace: =========
INFO   | jvm 1    | 2011/03/18 10:06:10 | /lib64/libc.so.6[0x3b9527245f]
INFO   | jvm 1    | 2011/03/18 10:06:10 | /lib64/libc.so.6(cfree+0x4b)[0x3b952728bb]
INFO   | jvm 1    | 2011/03/18 10:06:10 | /lib64/security/pam_winbind.so[0x2aaadaddc8f9]
INFO   | jvm 1    | 2011/03/18 10:06:10 | /lib64/security/pam_winbind.so[0x2aaadaddee4c]
INFO   | jvm 1    | 2011/03/18 10:06:10 | /lib64/security/pam_winbind.so(pam_sm_authenticate+0x304)[0x2aaadaddf9e4]
INFO   | jvm 1    | 2011/03/18 10:06:10 | /lib64/libpam.so.0(_pam_dispatch+0x277)[0x3b97e02dc7]
INFO   | jvm 1    | 2011/03/18 10:06:10 | /lib64/libpam.so.0(pam_authenticate+0x42)[0x3b97e026d2]
```

In this case SonarQube crashes and restarts automatically.
It appears to be a `pam_winbind.so` issue. This workaround should fix it.

Edit `/etc/security/pam_winbind.conf` and set Kerberos authentication:
```
/etc/security/pam_winbind.conf
#
# pam_winbind configuration file
#
# /etc/security/pam_winbind.conf
#
 
[global]
 
# turn on debugging
#debug = yes
 
# request a cached login if possible
# (needs "winbind offline logon = yes" in smb.conf)
cached_login = yes
 
# authenticate using kerberos
krb5_auth = yes
 
# when using kerberos, request a "FILE" krb5 credential cache type
# (leave empty to just do krb5 authentication but not have a ticket
# afterwards)
;krb5_ccache_type = FILE
 
# make successful authentication dependend on membership of one SID
# (can also take a name)
;require_membership_of =
```
