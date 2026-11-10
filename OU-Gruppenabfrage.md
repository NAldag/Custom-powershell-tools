

```powershell

Import-Module ActiveDirectory

$OU = "OU=Gruppen,DC=firma,DC=local"
$ExportPath = "C:\Temp\AD_Gruppen.csv"

Get-ADGroup -Filter * -SearchBase $OU -SearchScope Subtree -Properties Description, GroupScope, GroupCategory |
    Select-Object Name, SamAccountName, GroupCategory, GroupScope, Description, DistinguishedName |
    Export-Csv -Path $ExportPath -Delimiter ";" -Encoding UTF8 -NoTypeInformation

Write-Host "Export abgeschlossen: $ExportPath"

```
