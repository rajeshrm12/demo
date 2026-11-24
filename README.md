$src = "C:\Program Files\Microsoft SDKs\Azure\CLI2\Lib\site-packages\certifi\cacert.pem"
$dst = "C:\certs\azcli-ca.pem"
New-Item -ItemType Directory -Force C:\certs | Out-Null
Copy-Item $src $dst -Force

Get-ChildItem Cert:\CurrentUser\Root, Cert:\LocalMachine\Root, Cert:\CurrentUser\CA, Cert:\LocalMachine\CA |
  Where-Object { $_.Subject -match "Navy Federal|NFCU" } |
  ForEach-Object {
    "`n-----BEGIN CERTIFICATE-----`n" + [Convert]::ToBase64String($_.RawData, 'InsertLineBreaks') + "`n-----END CERTIFICATE-----" |
      Add-Content $dst -Encoding ascii
  }

setx REQUESTS_CA_BUNDLE $dst


Get-Content C:\certs\azcli-ca.pem -Tail 5


echo %REQUESTS_CA_BUNDLE%
az login
