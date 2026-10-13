
```powershell



# Pfade anpassen
$InputFile  = "C:\Temp\Mitarbeiter.txt"
$OutputFile = "C:\Temp\GefundeneAccounts.csv"

Import-Module ActiveDirectory

# AD-Benutzer inklusive Status und Distinguished Name einlesen
$ADUsers = Get-ADUser -Filter * -Properties `
    GivenName, Surname, SamAccountName, Enabled, DistinguishedName

$Results = foreach ($Line in Get-Content -Path $InputFile) {

    if ($Line -notmatch '^\s*(?<Nachname>[^,]+),\s*(?<Vorname>.+?)\s*$') {
        continue
    }

    $Nachname = $Matches.Nachname.Trim()
    $Vorname  = $Matches.Vorname.Trim()

    $MatchesAD = @(
        $ADUsers | Where-Object {
            $_.Surname -eq $Nachname -and
            $_.GivenName -eq $Vorname
        }
    )

    if ($MatchesAD.Count -eq 0) {
        [PSCustomObject]@{
            Nachname       = $Nachname
            Vorname        = $Vorname
            SamAccountName = ""
            OU             = ""
            Status         = "Nicht gefunden"
        }
        continue
    }

    foreach ($User in $MatchesAD) {

        # OU aus dem Distinguished Name ermitteln
        $OU = ($User.DistinguishedName -split ',', 2)[1]

        # Kontostatus lesbar ausgeben
        $Status = if ($User.Enabled) {
            "Aktiv"
        } else {
            "Inaktiv"
        }

        [PSCustomObject]@{
            Nachname       = $Nachname
            Vorname        = $Vorname
            SamAccountName = $User.SamAccountName
            OU             = $OU
            Status         = $Status
        }
    }
}

$Results | Export-Csv -Path $OutputFile `
    -Delimiter ';' -NoTypeInformation -Encoding UTF8

Write-Host "Fertig. Ergebnis gespeichert unter: $OutputFile"


```
