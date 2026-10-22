
```powershell

$InputCsv = "C:\Temp\Benutzer.csv"

$VergleichCsv = "C:\Temp\Vergleichstabelle.csv"

$OutputCsv = "C:\Temp\Ergebnis.csv"

# Spaltennamen
$BenutzerSpalte = "Benutzername"
$AustrittsSpalte = "Austrittsdatum"

# ============================================
# CSV-Dateien einlesen
# ============================================

$BenutzerListe = Import-Csv -Path $InputCsv -Delimiter ";"
$VergleichListe = Import-Csv -Path $VergleichCsv -Delimiter ";"

# Hashtable für den schnellen Abgleich erstellen
$VergleichHash = @{}

foreach ($Eintrag in $VergleichListe) {
    $Name = $Eintrag.$BenutzerSpalte

    if (-not [string]::IsNullOrWhiteSpace($Name)) {
        $VergleichHash[$Name.Trim()] = $Eintrag.$AustrittsSpalte
    }
}

# ============================================
# Benutzer abgleichen
# ============================================

$Ergebnis = foreach ($Benutzer in $BenutzerListe) {

    $Name = $Benutzer.$BenutzerSpalte

    if ([string]::IsNullOrWhiteSpace($Name)) {
        continue
    }

    $Name = $Name.Trim()

    if ($VergleichHash.ContainsKey($Name)) {

        [PSCustomObject]@{
            Benutzername   = $Name
            Gefunden       = "Ja"
            Austrittsdatum = $VergleichHash[$Name]
        }

    }
    else {

        [PSCustomObject]@{
            Benutzername   = $Name
            Gefunden       = "Nein"
            Austrittsdatum = ""
        }
    }
}

# ============================================
# Ergebnis ausgeben und exportieren
# ============================================

$Ergebnis | Format-Table -AutoSize

$Ergebnis | Export-Csv `
    -Path $OutputCsv `
    -Delimiter ";" `
    -NoTypeInformation `
    -Encoding UTF8

Write-Host "`nAbgleich abgeschlossen."
Write-Host "Ergebnis gespeichert unter: $OutputCsv"

```
