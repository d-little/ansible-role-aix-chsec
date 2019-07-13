# IBM AIX CHSEC WRAPPER ROLE

## Purpose

Wrap the AIX [chsec](https://www.ibm.com/support/knowledgecenter/en/ssw_aix_72/com.ibm.aix.cmds1/chsec.htm) command line utility.

## Configuration

### chsec yaml Format

```yaml
---
chsec:
  - file: filename
    stanza: stanzaname
    key: key
    value: value
  - file: filename
    stanza: stanzaname
    key: key
    value: value
```

### Limitations

We're bound by the same limitations as found in IBM's documentation, as below.  This role *does not check for valid inputs nor does it santitise inputs. We rely on `chsec` to know if the command worked; if the chsec fails, the task will fail.*

The role is idempontent in that it at least checks to see if the value is already set using `lssec` before attempting to run `chsec`. This does not guarentee that it will work though, you still need to follow IBM's rules:

- When modifying attributes in the **/etc/security/environ**, **/etc/security/lastlog**, **/etc/security/limits**, **/etc/security/passwd**, and **/etc/security/user** files, the stanza name specified by the Stanza parameter must either be a valid user name or default.
- When modifying attributes in the **/etc/security/group** file, the stanza name specified by the Stanza parameter must either be a valid group name or default.
- When modifying attributes in the **/usr/lib/security/mkuser.default** file, the Stanza parameter must be either admin or user.
- When modifying attributes in the **/etc/security/portlog** file, the Stanza parameter must be a valid port name.
- When modifying attributes in the **/etc/security/login.cfg** file, the Stanza parameter must either be a valid port name, a method name, or the usw attribute.
- When modifying attributes in the **/etc/security/login.cfg** or **/etc/security/portlog** file in a stanza that does not already exist, the stanza is automatically created by the chsec command.
- You cannot modify the password attribute of the **/etc/security/passwd** file using the chsec command.
- Only the root user or a user with an appropriate authorization can change administrative attributes. For example, to modify administrative group data, the user must be root or have GroupAdmin authorization.

#### Examples

- Set 'good' ulimits defaults

```yaml
---
chsec:
  - file: /etc/security/limits
    stanza: default
    key: 'fsize'
    value: '-1'
  - file: /etc/security/limits
    stanza: default
    key: core
    value: 2097151
  - file: /etc/security/limits
    stanza: default
    key: cpu
    value: -1
  - file: /etc/security/limits
    stanza: default
    key: data
    value: 262144
  - file: /etc/security/limits
    stanza: default
    key: rss
    value: 65536
  - file: /etc/security/limits
    stanza: default
    key: stack
    value: 65536
  - file: /etc/security/limits
    stanza: default
    key: nofiles
    value: 2000
```

- Change the /dev/tty0 port to automatically lock if 5 unsuccessful login attempts occur within 60 seconds:

```yaml
---
chsec:
  - file: /etc/security/login.cfg
    stanza: /dev/tty0
    key: logindisable
    value: 5
  - file: /etc/security/login.cfg
    stanza:  /dev/tty0
    key: logininterval
    value: 60
```

- Unlock the /dev/tty0 port after it has been locked by the system:

```yaml
---
chsec:
  - file: /etc/security/portlog
    stanza: /dev/tty0
    key: locktime
    value: 0
```

- Allow logins from 8:00 a.m. until 5:00 p.m. for all users
- Change the CPU time limit of user joe AND chrlie to 1 hour (3600 seconds):

```yaml
---
chsec:
  - file: /etc/security/user
    stanza: default
    key: logintimes
    value: ":0800-1700"
  - file: /etc/security/limits
    stanza: joe
    key: cpu
    value: 3600
  - file: /etc/security/limits
    stanza: charlie
    key: cpu
    value: 3600
```

### Modifyable Files

| Item                                | Description | IBM Docs
| -                                   | -           | -
| /etc/nscontrol.conf                 | Contains the configuration information of some name services. | [7.1](https://www.ibm.com/support/knowledgecenter/en/ssw_aix_71/com.ibm.aix.files/nscontrol.conf.htm) - [7.2](https://www.ibm.com/support/knowledgecenter/en/ssw_aix_72/com.ibm.aix.files/nscontrol.conf.htm)
| /etc/secvars.cfg                    | Contains a stanza file. | [7.1]((https://www.ibm.com/support/knowledgecenter/en/ssw_aix_71/com.ibm.aix.files/secvars.cfg.htm)) - [7.2](https://www.ibm.com/support/knowledgecenter/en/ssw_aix_72/com.ibm.aix.files/secvars.cfg.htm)
| /etc/security/domains               | Contains the valid domain definitions for the system. | [7.1](https://www.ibm.com/support/knowledgecenter/en/ssw_aix_71/com.ibm.aix.files/domainfile.htm) - [7.2](https://www.ibm.com/support/knowledgecenter/en/ssw_aix_72/com.ibm.aix.files/domainfile.htm)
| /etc/security/environ               | Contains the environment attributes of users. | [7.2](https://www.ibm.com/support/knowledgecenter/en/ssw_aix_72/com.ibm.aix.files/environ.htm)
| /etc/security/group                 | Contains extended attributes of groups. / Defines the last login attributes for users. | [7.2](https://www.ibm.com/support/knowledgecenter/en/ssw_aix_72/com.ibm.aix.files/group.htm)
| /etc/security/limits                | Defines resource quotas and limits for each user. | [7.2](https://www.ibm.com/support/knowledgecenter/en/ssw_aix_72/com.ibm.aix.files/limits.htm)
| /etc/security/login.cfg             | Contains port configuration information. | [7.2](https://www.ibm.com/support/knowledgecenter/en/ssw_aix_72/com.ibm.aix.files/login.cfg.htm)
| /etc/security/passwd                | Contains password information. | [7.2](https://www.ibm.com/support/knowledgecenter/en/ssw_aix_72/com.ibm.aix.files/passwd.htm)
| /etc/security/portlog               | Contains unsuccessful login attempt information for each port. | [7.2](https://www.ibm.com/support/knowledgecenter/en/ssw_aix_72/com.ibm.aix.files/portlog.htm)
| /etc/security/pwdalg.cfg            | Contains the configuration information for loadable password algorithms (LPA). | [7.2](https://www.ibm.com/support/knowledgecenter/en/ssw_aix_72/com.ibm.aix.files/pwdalg.cfg.htm)
| /etc/security/roles                 | Contains a list of valid roles. | [7.2](https://www.ibm.com/support/knowledgecenter/en/ssw_aix_72/com.ibm.aix.files/roles.htm)
| /etc/security/smitacl.group         | Contains group ACL definitions. | [7.2](https://www.ibm.com/support/knowledgecenter/en/ssw_aix_72/com.ibm.aix.files/smitacl.group.htm)
| /etc/security/smitacl.user          | Contains user ACL definitions. | [7.2](https://www.ibm.com/support/knowledgecenter/en/ssw_aix_72/com.ibm.aix.files/smitacl.user.htm)
| /etc/security/user                  | Contains the extended attributes of users. | [7.2](https://www.ibm.com/support/knowledgecenter/en/ssw_aix_72/com.ibm.aix.files/user.htm)
| /etc/security/user.roles            | Contains a list of roles for each user. | [7.2](https://www.ibm.com/support/knowledgecenter/en/ssw_aix_72/com.ibm.aix.files/user.roles.htm)
| /etc/security/audit/hosts           | Contains host and processor IDs. | *N/A*
| /etc/security/enc/LabelEncodings    | Contains label definitions for the Trusted AIX system. | [7.2](https://www.ibm.com/support/knowledgecenter/en/ssw_aix_72/com.ibm.aix.security/taix_cust_le_file.htm)
| /etc/security/rtc/rtcd_policy.conf  | Contains the configuration information for the rtcd daemon. | [7.2](https://www.ibm.com/support/knowledgecenter/en/ssw_aix_72/com.ibm.aix.files/rtcd.policy.cnf.htm)
| /usr/lib/security/mkuser.default    | Contains the default values for new users. | [7.2](https://www.ibm.com/support/knowledgecenter/en/ssw_aix_71/com.ibm.aix.files/mkuser.default.htm)
# ansible-role-aix-chsec
