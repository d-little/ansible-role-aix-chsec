##### SUMMARY

Below is a IBM AIX [chsec]((https://www.ibm.com/support/knowledgecenter/en/ssw_aix_72/com.ibm.aix.cmds1/chsec.htm)) module design proposal.

I've been trying to write a role to 'easily' wrap up the IBM AIX chsec command and make it idempotent.  While I've got it (mostly) working [here](https://github.com/d-little/ansible-role-aix-chsec), I'd rather help to make a module to perform this work instead of maintaining such a clunky role.

##### COMPONENT NAME

aix_chsec

##### Idempotency

We can make the tool idempotent by first checking the output of [`lssec`](https://www.ibm.com/support/knowledgecenter/en/ssw_aix_72/com.ibm.aix.cmds3/lssec.htm) before making changes.  It should be trivial to use this command in Python.

I'm achieving this in the ansible-role-aix-chsec using something similar to:

```shell
shell: >
  ( lssec -f '{{ file }}' -s '{{ stanza }}' -a '{{ key }}' | grep -q '{{ stanza }} {{ key }}={{ value }}' ) && echo nochange || chsec -f '{{ file }}' -s '{{ stanza }}' -a '{{ key }}={{ value }}'
```

##### Scope

This module could be part of a larger module to manage many types of AIX settings (aix_settings ?), or simply focus on the chsec command itself.  For now we can focus on aix_chsec.

##### Potential Layout(s)

###### 'Simple'

One example implementation with multiple key/value being set within a single files+stanza at a time:

```yaml
- name: Name of Task
  aix_chsec:
    file: filename
    stanza: stanzaname
    attributes:
      key1: value1
      key2: value2
      keyN: valueN
    state: [ present, absent ]
    backup: bool (default no, timestamped backup before making changes.)
```

This would return `Changed` if *any number* of the above key/value pairs were changed.  If *none* were changed it would be `OK`.  Failure to change *any* key:value pair would consitute as a `failure` regardless of how many attributes were successfully set.  In order to check for failure we need to re-check the `lssec` command after chsec is run;  chsec will often return 0 even if nothing changed.

- `attributes` can have an alias `attrs`
- Attributes alternatives:
  - `attributes: [ (key, value), (key, value), (key, value) ]`
  - `attributes: [ key: value, key2: value2 ]`

**Real world use:**

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
        attrs:
          logintimes: ":0800-1700"
      - file: /etc/security/limits
        stanza: joe
        attrs: [ cpu: 3600 ]
      - file: /etc/security/limits
        stanza: charlie
        attrs:
          cpu: 3600
  remote_user: deploy
- tasks:
  name: Example chsec
    aix_chsec:
      file: "{{ item.file }}"
      stanza: "{{ item.stanza }}"
      attrs: "{{ item.attrs }}"
    loop:
      - "{{ example_chsec }}"
```

The example_chsec YAML above converts to the following JSON:

```JSON
{
  "example_chsec": [
    {
      "file": "/etc/security/user",
      "stanza": "default",
      "attrs": {
        "logintimes": ":0800-1700"
      }
    },
    {
      "file": "/etc/security/limits",
      "stanza": "joe",
      "attrs": {
        "cpu": 3600
      }
    },
    {
      "file": "/etc/security/limits",
      "stanza": "charlie",
      "attrs": {
        "cpu": 3600,
        "file": -1
      }
    }
  ]
}
```

###### 'Nested' Stanzas

We could 'nest' stanzas for a single file within a single module call.  This makes configuration slightly more complicated, but can potentially reduce the amount of required tasks/YAML code.  We can keep the `stanza` option available in addition to `stanzas`, mutually exclusive.

```yaml
- name: Name of Task
  aix_chsec:
    file: filename
    stanzas:
      - stanza1:
          key1: value1
          key2: value2
          keyN: valueN
        state: [ present|absent ]
      - stanza2:
          key1: value1
          key2: value2
          keyN: valueN
        state: [ present|absent ]
    backup: bool (default no, timestamped backup before making changes.)
```

Real world use:

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
        attrs:
          logintimes: ":0800-1700"
      - file: /etc/security/limits
        stanzas:
        - joe:
            cpu: 3600
            file: -1
        - charlie:
            cpu: 3600
  remote_user: deploy
- tasks:
  name: Example chsec
    aix_chsec:
      file: "{{ item.file }}"
      stanza: "{{ item.stanza }}"
      attrs: "{{ item.attrs }}"
    loop:
      - "{{ example_chsec }}"
```

The example_chsec YAML above converts to the following JSON:

```JSON
{
  "example_chsec": [
    {
      "file": "/etc/security/user",
      "stanza": "default",
      "attrs": {
        "logintimes": ":0800-1700"
      }
    },
    {
      "file": "/etc/security/limits",
      "stanzas": [
        {
          "joe": {
            "cpu": 3600,
            "file": -1
          }
        },
        {
          "charlie": {
            "cpu": 3600
          }
        }
      ]
    }
  ]
}
```

##### Open Questions

- When setting multiple attributes per stanza, do we:
  - Run one chsec per attribute, or using a single chsec with multiple attributes
  - Fail as soon as one attribute fails, or try to set every attribute before failing the module?
    - ie: IF we're setting the ulimits `file`, `count`, and `size`;  if `file` fails to set, do we continue to try `count` and `size` and then quit with failure, or quit at the point of failure?
- Do we stick with **one-stanza-per-file** or allow **many-stanzas-per-file**?
