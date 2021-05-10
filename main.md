# PowerShell tools




base64

```powershell
function Base64 {
    param (
        [Parameter(Mandatory,ValueFromPipeline)]
        [string]$Text,
        [bool]$decode = $false,
        [bool]$Unicode = $false
    )
    if ($decode){
        if ($Unicode){
            [System.Text.Encoding]::Unicode.GetString([System.Convert]::FromBase64String($Text))
        }else{
            [System.Text.Encoding]::UTF8.GetString([System.Convert]::FromBase64String($Text))
        }
    }else{
        if($Unicode){
            [System.Convert]::ToBase64String([System.Text.Encoding]::Unicode.GetBytes($Text))
        }else{
            [System.Convert]::ToBase64String([System.Text.Encoding]::UTF8.GetBytes($Text))
        }
    }
}
```

http client

```
function Open-Url {
    param (
        [Parameter(Mandatory)]
        [string]$Url
    )
    (New-Object System.Net.WebClient).DownloadString($Url)
} 
```

文件下载

```
function Download-File {
    param (
        [Parameter(Mandatory)]
        [string]$Url,
        [Parameter(Mandatory)]
        [string]$OutFile
    )
    (New-Object System.Net.WebClient).DownloadFile($Url,$OutFile)
}
```

IP Scan

```
function Scan-IP {
    param (
        [Parameter(Mandatory)][string]$Network
    )
    $tool = New-Object System.Net.NetworkInformation.Ping
    $Ip, $Subnet = $Network.Split("/")
    $A, $B, $C, $D = $Ip.Split(".")
    $SubnetRange = [System.Math]::Pow(2, 8 - $Subnet % 8)
    if (24 -gt $Subnet ) {
        Write-Error "The range to big !"
    }
    else {
        $NetworkAddr = $D -band (256 - $SubnetRange)
        for ($i = $NetworkAddr + 1; $i -lt $NetworkAddr + $SubnetRange - 1; $i++) {
            $CurrentIP = $A, $B, $C, $i -join "."
            if ($tool.Send($CurrentIP,1).Status -eq "Success"){ $CurrentIP }
        }
    }
}


function ScanIPFromTXT {
    param (
        [Parameter(Mandatory)][string]$filename
    )
    $tool = New-Object System.Net.NetworkInformation.Ping
    
    Get-Content -Path $filename | ForEach-Object {
        $CurrentIP = $_
        if ($tool.Send($CurrentIP, 1).Status -eq "Success") { $CurrentIP }
    }
}
```

http server

```
function Start-HttpServer {
    param (
        # Parameter help description
        [Parameter(Mandatory)]
        [string]$Addr,
        [string]$Port
    )

    $p=Get-Location
    $H=New-Object Net.HttpListener
    $H.Prefixes.Add("http://${Addr}:${Port}/")
    $H.Start()
    While ($H.IsListening) {
        $HC=$H.GetContext()
        $HR=$HC.Response

        $HR.Headers.Add("Content-Type","text/plain")

        $file=Join-Path $p.Path ($HC.Request).RawUrl
        if (Test-Path $file -PathType Leaf){
            $text=[IO.File]::ReadAllText($file)
            $text=[Text.Encoding]::UTF8.GetBytes($text)
            
            $HR.ContentLength64 = $text.Length
            $HR.OutputStream.Write($text,0,$text.Length)
        }
        $HR.Close()

    }
    $H.Stop()
} 
```



从URL安装MSI

```
function Install-MSI {
    param (
        [Parameter(Mandatory)]
        [string]$Url,
        [Parameter(Mandatory)]
        [string]$pkgName
    )

    if (-not (Test-Path $env:TEMP/msipkg)) {
        mkdir -p $env:TEMP/msipkg
    }
    cd $env:TEMP/msipkg

    $fullName = "$env:TEMP/msipkg/" + $pkgName
    (New-Object System.Net.WebClient).DownloadFile($Url, $fullName)
    IEX "$fullName /quiet /norestart"
    sleep 120
    Remove-Item $env:TEMP/msipkg/*
}
```



FTP上传

```
function ftpupload {
    param (
        $ftpurl, $username, $password, $filepath
    )
    $fileinf = New-Object System.IO.FileInfo($filepath)
    $upFTP = [system.net.ftpwebrequest] [system.net.webrequest]::create($ftpurl + $fileinf.name)
    $upFTP.Credentials = New-Object System.Net.NetworkCredential($username, $password)
    $upFTP.Method = [system.net.WebRequestMethods+ftp]::UploadFile
    $upFTP.KeepAlive = $false
    $sourceStream = New-Object System.Io.StreamReader($fileInf.fullname)
    $fileContents = [System.Text.Encoding]::UTF8.GetBytes($sourceStream.ReadToEnd())
    $sourceStream.Close();
    $upFTP.ContentLength = $fileContents.Length;
    $requestStream = $upFTP.GetRequestStream();
    $requestStream.Write($fileContents, 0, $fileContents.Length);
    $requestStream.Close();
    $response = $upFTP.GetResponse();
    $response.StatusDescription
    $response.Close();
}
```
