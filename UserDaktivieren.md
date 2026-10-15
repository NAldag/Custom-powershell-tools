
```powershell
Import-Module ActiveDirectory

# Konfiguration
$CsvPath = "C:\Scripts\Benutzer.csv"
$TargetOU = "OU=Deaktivierte Benutzer,DC=firma,DC=local"

# Benutzer aus CSV einlesen
$Benutzer = Import-Csv -Path $CsvPath

foreach ($Eintrag in $Benutzer) {

    $Username = $Eintrag.SamAccountName

    try {
        # Benutzer in AD suchen
        $ADUser = Get-ADUser -Identity $Username -ErrorAction Stop

        # Konto deaktivieren
        Disable-ADAccount -Identity $ADUser -ErrorAction Stop

        # Benutzer in Ziel-OU verschieben
        Move-ADObject -Identity $ADUser.DistinguishedName `
                      -TargetPath $TargetOU `
                      -ErrorAction Stop

        Write-Host "$Username: Deaktiviert und verschoben." -ForegroundColor Green
    }
    catch {
        Write-Host "$Username: Fehler - $($_.Exception.Message)" -ForegroundColor Red
    }
}
```
