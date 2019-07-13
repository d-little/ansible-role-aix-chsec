<!--- Verify first that your feature was not already discussed on GitHub -->
<!--- Complete *all* sections as described, this form is processed automatically -->

##### SUMMARY
<!--- Describe the new feature/improvement briefly below -->
I've been trying to write a role to 'easily' wrap up the IBM AIX [chsec](https://www.ibm.com/support/knowledgecenter/en/ssw_aix_72/com.ibm.aix.cmds1/chsec.htm) command and make it idempotent.  I've gotten to the point now that I'm pretty much just writing python and figure that it might be a good idea to just add a new module to handle this task.  This module could be part of a larger module to manage many types of AIX settings (aix_settings ?), or simply focus on the chsec command itself.

##### ISSUE TYPE

- Feature Idea

##### COMPONENT NAME
<!--- Write the short name of the module, plugin, task or feature below, use your best guess if unsure -->
aix_chsec

##### ADDITIONAL INFORMATION
<!--- Describe how the feature would be used, why it is needed and what it would solve -->

We can make the tool idempotent by either first checking the output of [`lssec`](https://www.ibm.com/support/knowledgecenter/en/ssw_aix_72/com.ibm.aix.cmds3/lssec.htm) or checking the target file directly.  But I think using `lssec` makes more sense.

One example implementation with a single key/value being set at a time:

```yaml
- name: Name of Task
  aix_chsec:
    file: filename
    stanza: stanzaname
    key: key
    value: value
    state: ?state?Necessary??
    backup: bool (default no, timestamped backup before making changes.)
```

We could also allow for `keys` in place/instead of `key` to be a dict or a list of key/value tuples instead of just a 1 to 1 ratio. EG:

```yaml
keys: [ (key, value), (key, value), (key, value) ]
```

or:

```yaml
keys:
  - key: value
  - key: value
  - key: value
```

In real world use:

- Allow logins from 8:00 a.m. until 5:00 p.m. for all users
- Change the CPU time limit of user joe AND chrlie to 1 hour (3600 seconds):

```yaml
---
- hosts: targetserver
  gather_facts: no
  vars:
    example_chsec:
      - file: /etc/security/user
        stanza: default
        keys:
          - logintimes: ":0800-1700"
      - file: /etc/security/limits
        stanza: joe
        keys: { cpu: 3600 }
      - file: /etc/security/limits
        stanza: charlie
        key: cpu
        value: 3600
  remote_user: deploy
- tasks:
  name: Example chsec
    aix_chsec:
      file: {{ item.file }}
      stanza: {{ item.stanza }}
      key: {{ item.key }}
      value: {{ item.value }}
    loop:
      - "{{ example_chsec }}"
```
