# smbexec.py

[smbexec.py](https://github.com/fortra/impacket/blob/master/examples/smbexec.py) can be used to execute commands on a remote Windows target by creating and running a temporary service over **SMB**.  
It provides a semi-interactive shell similar to wmiexec.py and dcomexec.py, but the execution mechanism relies on the **Windows Service Control Manager (SCM)**.

This method is similar to SysInternals’ PsExec technique: it creates a remote service, uses it to execute commands, retrieves output via SMB, and then deletes the service.

{% hint style="warning" %}
### Required privileges

To use `smbexec.py`, the following prerequisites must be met on the target:

- **Local administrator privileges** (mandatory to create/modify Windows services)
- **SMB reachable** (TCP 445 is required for service management and output retrieval)
- **Service Control Manager (SCM) accessible**  
  The authenticated user must be allowed to remotely create/start/delete services (default for local admin)
- **ADMIN$ accessible** (used to drop the temporary executable and retrieve output)
- **File and printer sharing enabled** (default on most Windows hosts)
{% endhint %}

## Commons

It has the following generic command line arguments, similar to other Impacket tools:

* required positional argument:  
  `[[domain/]username[:password]@]<targetName or address>`  
  Examples:  
  - `domain.local/user@dc01`  
  - `domain/user:password@10.10.0.1`

![](<../../.gitbook/assets/impacket_positional_arg-with target.png>)

* `-hashes`: the LM and/or NT hash to use for a [pass-the-hash](https://www.thehacker.recipes/ad/movement/ntlm/pth).  
  Format: `[LMhash]:NThash` (LM optional, NT must be prefixed with `:`).

* `-aesKey`: AES128/256 key for [pass-the-key](https://www.thehacker.recipes/ad/movement/kerberos/ptk) authentication (Kerberos).

* `-k`: use Kerberos authentication (via a Ccache if `KRB5CCNAME` is set).

* `-no-pass`: required when no password is provided or when using `-k`.

* `-dc-ip`: IP address of the domain controller to use instead of DNS.

* `-debug`: enables verbose debugging output.

## Specificities

smbexec.py introduces some options specific to the service-based execution model:

* `-codec`: sets the Windows output encoding (default: `latin-1`).  
  Useful for Unicode output on non-English systems.

* `-service-name`: specify a custom service name instead of a random one.

* `-shell-type`: either `cmd` (default) or `powershell`.

* `-nooutput`: execute commands without retrieving their output.

Unlike wmiexec.py, smbexec.py uses a **Windows binary dropped on the target**, executed through a service, which can make it slightly noisier but often more reliable.

```bash
# Cleartext authentication
smbexec.py 'DOMAIN'/'USER':'PASSWORD'@'target_ip'

# Pass-the-hash
smbexec.py -hashes :'NThash' 'DOMAIN'/'USER'@'target_ip'

# Kerberos authentication
smbexec.py -no-pass -k 'DOMAIN'/'USER'@'target_FQDN'

# PowerShell instead of cmd
smbexec.py -shell-type powershell 'DOMAIN'/'USER':'PASSWORD'@'target_ip'

# Custom service name
smbexec.py -service-name MyService 'DOMAIN'/'USER':'PASSWORD'@'target_ip'
```

{% hint style="info" %}
smbexec.py is often more stable than wmiexec.py in restricted environments but is also more detectable, since it creates and starts a Windows service.
If SMB is blocked or service creation is monitored, consider switching to wmiexec.py or dcomexec.py.
{% endhint %}
