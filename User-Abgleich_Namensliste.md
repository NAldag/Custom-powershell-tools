

```powershell

# Pfade anpassen
$InputFile  = "C:\Temp\Mitarbeiter.txt"
$OutputFile = "C:\Temp\GefundeneAccounts.csv"

Import-Module ActiveDirectory

$ADUsers = Get-ADUser -Filter * -Properties GivenName, Surname, SamAccountName

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

    foreach ($User in $MatchesAD) {
        [PSCustomObject]@{
            Nachname      = $Nachname
            Vorname       = $Vorname
            SamAccountName = $User.SamAccountName
        }
    }
}

$Results | Export-Csv -Path $OutputFile `
    -Delimiter ';' -NoTypeInformation -Encoding UTF8

Write-Host "Fertig. Ergebnis gespeichert unter: $OutputFile"

````
