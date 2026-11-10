

```powershell

Import-Module ActiveDirectory

# Distinguished Name der OU
$OU = "OU=Gruppen,DC=firma,DC=local"

# CSV-Zieldatei
$ExportPath = "C:\Temp\AD_Gruppen.csv"

# Gruppen aus der OU abfragen und exportieren
Get-ADGroup -Filter * -SearchBase $OU -SearchScope Subtree -Properties Description, GroupScope, GroupCategory |
    Select-Object Name, SamAccountName, GroupCategory, GroupScope, Description, DistinguishedName |
    Export-Csv -Path $ExportPath -Delimiter ";" -Encoding UTF8 -NoTypeInformation

Write-Host "Export abgeschlossen: $ExportPath"

```
