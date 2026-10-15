
```powershell

Import-Module ActiveDirectory -ErrorAction Stop

# Konfiguration
$CsvPath  = "C:\Scripts\Import Userliste.csv"
$TargetOU = "OU=Deaktivierte Benutzer,DC=firma,DC=local"

# Ziel-OU prüfen
try {
    Get-ADOrganizationalUnit -Identity $TargetOU -ErrorAction Stop |
        Out-Null
}
catch {
    Write-Error "Ziel-OU nicht gefunden: $TargetOU"
    exit 1
}

# Benutzerliste einlesen
$Benutzer = Import-Csv -Path $CsvPath

foreach ($Eintrag in $Benutzer) {

    $Username = $Eintrag.SamAccountName

    if ([string]::IsNullOrWhiteSpace($Username)) {
        Write-Host "Leerer Benutzername - übersprungen." `
            -ForegroundColor Yellow
        continue
    }

    try {
        # Benutzer suchen
        $ADUser = Get-ADUser -Identity $Username -ErrorAction Stop

        # Prüfen, ob Benutzer bereits in der Ziel-OU liegt
        if ($ADUser.DistinguishedName -like "*,$TargetOU") {
            Write-Host "$Username ist bereits in der Ziel-OU." `
                -ForegroundColor Yellow
            continue
        }

        # Benutzer verschieben
        Move-ADObject `
            -Identity $ADUser.DistinguishedName `
            -TargetPath $TargetOU `
            -ErrorAction Stop

        Write-Host "$Username erfolgreich verschoben." `
            -ForegroundColor Green
    }
    catch {
        Write-Host "$Username - FEHLER: $($_.Exception.Message)" `
            -ForegroundColor Red
    }
}

```
