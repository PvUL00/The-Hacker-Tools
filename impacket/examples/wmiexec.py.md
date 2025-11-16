# wmiexec.py

[wmiexec.py](https://github.com/fortra/impacket/blob/master/examples/wmiexec.py) can be used to execute commands on a remote Windows target through **WMI** (Windows Management Instrumentation) over **DCOM**.  
It provides a semi-interactive shell similar to dcomexec.py, but relies on the **Win32_Process** WMI provider to spawn commands remotely.

This technique does not require installing any service or agent on the target system and runs with the privileges of the authenticated user. It is commonly used for lateral movement when WMI/DCOM is allowed between machines.

{% hint style="warning" %}
### Required privileges

To use `wmiexec.py`, the following prerequisites must be met on the target:

- **Local administrator privileges** (required to remotely create processes via WMI and read/write through `ADMIN$`)
- **WMI/DCOM remote execution permissions** (WMI uses DCOM under the hood)
- **RPC/DCOM ports reachable** (TCP 135 + dynamic RPC ports)
- **SMB reachable** (port 445 for output retrieval through `ADMIN$`)
- **WMI service enabled** (`Winmgmt`) and remote WMI/DCOM not disabled (default on Windows)
{% endhint %}

## Commons

It has the following generic command line arguments, similar to many other Impacket tools:

* required positional argument:  
  `[[domain/]username[:password]@]<targetName or address>`  
  Examples:  
  - `domain.local/user@dc01`  
  - `domain/user:password@10.10.0.1`

![](<../../.gitbook/assets/impacket_positional_arg-with target.png>)

* `-hashes`: the LM and/or NT hash to use for a [pass-the-hash](https://www.thehacker.recipes/ad/movement/ntlm/pth).  
  Format: `[LMhash]:NThash` (LM optional, NT must be prefixed with `:`).

* `-aesKey`: AES128 or AES256 long-term key for [pass-the-key](https://www.thehacker.recipes/ad/movement/kerberos/ptk) (Kerberos).

* `-k`: authenticate using Kerberos. Will use a Ccache from the `KRB5CCNAME` environment variable if present ([pass-the-cache](https://www.thehacker.recipes/ad/movement/kerberos/ptc)), or fall back to password / NT hash for [OPTH](https://www.thehacker.recipes/ad/movement/kerberos/opth).

* `-no-pass`: required when using an empty password or when no password should be requested (useful with `-k`).

* `-dc-ip`: IP address of the domain controller. If omitted, a DNS resolution is performed using the domain in the positional argument.

* `-debug`: increases verbosity and prints tracebacks on errors.

## Specificities

wmiexec.py also has a few specific command line arguments:

* `-share`: the SMB share used to store and retrieve command output.  
  Default: `ADMIN$`

* `-nooutput`: execute commands without retrieving their output.

* `-no-raw`: disable raw output mode.

* `-silentcommand`: execute a single command and exit without launching a semi-interactive shell.

* `-shell-type`: either `cmd` (default) or `powershell`.

```bash
# Execute commands with cleartext credentials
wmiexec.py 'DOMAIN'/'USER':'PASSWORD'@'target_ip'

# Pass-the-hash (NTLM)
wmiexec.py -hashes :'NThash' 'DOMAIN'/'USER'@'target_ip'

# Execute a single command without interactive shell
wmiexec.py -silentcommand 'DOMAIN'/'USER':'PASSWORD'@'target_ip' 'whoami'

# Using Kerberos authentication
wmiexec.py -no-pass -k 'DOMAIN'/'USER'@'target_FQDN'

# Using a custom SMB share for output
wmiexec.py -share C$ 'DOMAIN'/'USER':'PASSWORD'@'target_ip'
```

{% hint style="info" %}
WMI-based execution is extremely common in both offensive and administrative contexts.
If WMI/DCOM is blocked or fails, consider switching to dcomexec.py or smbexec.py depending on the environment.
{% endhint %}
