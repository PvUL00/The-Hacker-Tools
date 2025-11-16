# dcomexec.py

[dcomexec.py](https://github.com/SecureAuthCorp/impacket/blob/master/examples/dcomexec.py) can be used to execute commands on a remote target via DCOM (Distributed Component Object Model). It provides a semi-interactive shell similar to wmiexec.py, but uses different DCOM endpoints for execution.

The utility currently supports three different DCOM objects for command execution:
* **MMC20.Application** - Default method using the MMC Application object
* **ShellWindows** - Uses the ShellWindows DCOM object
* **ShellBrowserWindow** - Uses the ShellBrowserWindow DCOM object

This technique does not require installing any service or agent on the target system and runs with the privileges of the authenticated user.

{% hint style="warning" %}
### Required privileges

To successfully execute commands via DCOM, the following conditions **must** be met:

**1. Local administrator privileges (mandatory)**  
`dcomexec.py` writes command output to `C:\Windows\` via **ADMIN$**, which requires being **local admin** on the target.

**2. DCOM Remote Activation / Launch permissions**  
The user must have permission to remotely activate the selected DCOM object  
(*MMC20.Application*, *ShellWindows*, *ShellBrowserWindow*).  
Local administrators have these permissions by default.

**3. RPC/DCOM ports accessible**  
- TCP **135** (RPC Endpoint Mapper)  
- RPC dynamic ports (**49152–65535** by default)  
If blocked, DCOM execution will fail.

**4. SMB accessible (TCP 445)**  
SMB is required to retrieve the command output via `ADMIN$`.  
SMBv2/v3 are supported (SMBv1 not required).

**5. DCOM enabled**  
DCOM is enabled by default on Windows.  
If disabled, the technique will not work.
{% endhint %}

## Commons

It has the following generic command line arguments, similar to many other tools:

* required positional argument: `[[domain/]username[:password]@]<targetName or address>` (e.g. `domain.local/user@dc01`, `domain/user:password@10.10.0.1`).

![](<../../.gitbook/assets/impacket\_positional\_arg-with target.png>)

* `-hashes`: the LM and/or NT hash to use for a [pass-the-hash](https://www.thehacker.recipes/ad/movement/ntlm/pth) (NTLM). The format is as follows: `[LMhash]:NThash` (the LM hash is optional, the NT hash must be prepended with a colon (`:`).).
* `-aesKey`: the AES128 or AES256 hexadecimal long-term key to use for a [pass-the-key](https://www.thehacker.recipes/ad/movement/kerberos/ptk) authentication (Kerberos).
* `-k`: this flag must be set when authenticating using Kerberos. The utility will try to grab credentials from a Ccache file which path must be set in the `KRB5CCNAME` environment variable. In this case, the utility will do [pass-the-cache](https://www.thehacker.recipes/ad/movement/kerberos/ptc). If valid credentials cannot be found or if the `KRB5CCNAME` variable is not or wrongly set, the utility will use the password specified in the positional argument for plaintext Kerberos authentication, or the NT hash (i.e. RC4 long-term key) in the `-hashes` argument for [overpass-the-hash](https://www.thehacker.recipes/ad/movement/kerberos/opth). A Kirbi file could also be converted to a Ccache file using [ticketConverter.py](ticketconverter.py.md) in order to be used by the utility (indirect [pass-the-ticket](https://www.thehacker.recipes/ad/movement/kerberos/ptt)).
* `-no-pass`: this flag must be set when an empty password will by used, or no password at all. Without this flag, the user will be prompted for a password when running the utility. This flag is especially useful when using `-k`.
* `-dc-ip`: IP address of the domain controller. If omitted, the positional argument's domain part will be used (in that case, it must be a Fully-Qualified-Domain-Name (FQDN)).
* `-debug`: with this flag set, the utility will be more verbose and will possibly print useful information for debug purposes. With this flag set, the utility will also print tracebacks.

## Specificities

It also has the following specific command line arguments:

* `-object`: the DCOM object to use for command execution. Available options are:
  * `MMC20` - Uses MMC20.Application DCOM object (default)
  * `ShellWindows` - Uses ShellWindows DCOM object
  * `ShellBrowserWindow` - Uses ShellBrowserWindow DCOM object
* `-silentcommand`: executes a single command without launching a semi-interactive shell and exits.
* `-shell-type`: the type of shell to use. Available options are `cmd` (default) or `powershell`.
* `-com-version`: specifies the DCOM version. If omitted, the script will try to determine the version automatically.

```bash
# Execute commands with cleartext credentials (default MMC20 object)
dcomexec.py 'DOMAIN'/'USER':'PASSWORD'@'target_ip'

# Pass-the-hash (with an NT hash)
dcomexec.py -hashes :'NThash' 'DOMAIN'/'USER'@'target_ip'

# Using a different DCOM object (ShellWindows)
dcomexec.py -object ShellWindows 'DOMAIN'/'USER':'PASSWORD'@'target_ip'

# Execute a single command without interactive shell
dcomexec.py -silentcommand 'DOMAIN'/'USER':'PASSWORD'@'target_ip' 'whoami'

# Using Kerberos authentication
dcomexec.py -no-pass -k 'DOMAIN'/'USER'@'target_FQDN'
```

{% hint style="info" %}
The MMC20.Application method is the most commonly used and reliable DCOM execution method. If it fails or is blocked, try the ShellWindows or ShellBrowserWindow alternatives as they may bypass certain security controls.
{% endhint %}

