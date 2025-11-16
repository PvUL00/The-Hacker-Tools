# psexec.py

[psexec.py](https://github.com/fortra/impacket/blob/master/examples/psexec.py) can be used to execute commands on a remote Windows target by uploading a service binary through **SMB** and executing it via the **Windows Service Control Manager (SCM)**.

It replicates the well-known **PsExec** technique from SysInternals:  
a service is created remotely, used to execute commands under **NT AUTHORITY\SYSTEM**, and then removed.  
This gives a semi-interactive SYSTEM-level shell on the target.

This method is reliable but more detectable than wmiexec.py or dcomexec.py due to the creation of a service and the upload of a temporary executable.

{% hint style="warning" %}
### Required privileges

To use `psexec.py`, the following prerequisites must be met on the target:

- **Local administrator privileges** (mandatory to create/start/delete services remotely)
- **SMB reachable** (TCP 445 is required for file upload and retrieving output)
- **Service Control Manager (SCM) accessible**  
  Local admin users can remotely create services by default.
- **ADMIN$ accessible** (used to upload the service binary and collect output)
- **File & Printer Sharing enabled** (default on Windows)
- **No application control blocking remote service execution** (AppLocker/WDAC can break it)
{% endhint %}

## Commons

It has the following generic Impacket arguments:

* required positional argument:  
  `[[domain/]username[:password]@]<targetName or address>`  
  Examples:  
  - `domain.local/user@dc01`  
  - `domain/user:password@10.10.0.1`

![](<../../.gitbook/assets/impacket_positional_arg-with target.png>)

* `-hashes`: LM and/or NT hash for [pass-the-hash](https://www.thehacker.recipes/ad/movement/ntlm/pth).  
  Format: `[LMhash]:NThash`.

* `-aesKey`: AES128/256 key for [pass-the-key](https://www.thehacker.recipes/ad/movement/kerberos/ptk).

* `-k`: use Kerberos authentication (optionally from Ccache via `KRB5CCNAME`).

* `-no-pass`: required when no password should be prompted (useful with `-k`).

* `-dc-ip`: specifies the Domain Controller IP (instead of DNS lookup).

* `-debug`: increases verbosity and prints stack traces.

## Specificities

psexec.py introduces execution options specific to the PsExec-like technique:

* `-codec`: specify output encoding (default: `latin-1`).

* `-service-name`: define a custom service name.  
  Default is a random alphanumeric string.

* `-remote-binary-name`: specify the name of the service binary uploaded to `ADMIN$`.

* `-shell-type`: choose between `cmd` (default) or `powershell`.

* `-nooutput`: execute the command without retrieving output.

psexec.py uploads a custom executable (`psexecsvc.exe`) to the target, runs it as a SYSTEM service, and deletes it afterward.

```bash
# Authenticate with cleartext credentials
psexec.py 'DOMAIN'/'USER':'PASSWORD'@'target_ip'

# Pass-the-hash
psexec.py -hashes :'NThash' 'DOMAIN'/'USER'@'target_ip'

# Using Kerberos
psexec.py -no-pass -k 'DOMAIN'/'USER'@'target_FQDN'

# Launch PowerShell instead of cmd
psexec.py -shell-type powershell 'DOMAIN'/'USER':'PASSWORD'@'target_ip'

# Custom service name
psexec.py -service-name MyService 'DOMAIN'/'USER':'PASSWORD'@'target_ip'
```

{% hint style="info" %}
psexec.py is one of the most reliable SYSTEM-level remote execution methods in Impacket, but also one of the most detectable due to service creation and binary upload.
If stealth is required, prefer wmiexec.py or dcomexec.py depending on environment controls.
{% endhint %}
