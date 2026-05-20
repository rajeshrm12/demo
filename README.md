$cert = Get-ChildItem -Path Cert:\LocalMachine\Root | Where-Object { $_.Subject -notlike "*Microsoft*" -and $_.Subject -notlike "*Symantec*" -and $_.Subject -notlike "*DigiCert*" -and $_.Subject -notlike "*Verisign*" } | Select-Object Subject, Thumbprint
$cert
