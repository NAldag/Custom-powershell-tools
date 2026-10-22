
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

```powershell


# ============================================
# AD-Benutzerabgleich mit Austrittsdatum
# ============================================

# Dateipfade anpassen
$InputDatei  = "C:\Abgleich\AD-Benutzer.txt"
$VergleichsDatei = "C:\Abgleich\Benutzerliste.csv"
$ErgebnisDatei = "C:\Abgleich\Abgleich-Ergebnis.csv"

# CSV-Trennzeichen: Standardmäßig Semikolon
$Trennzeichen = ";"

# Vergleichstabelle importieren
$Vergleich = Import-Csv -Path $VergleichsDatei `
                        -Delimiter $Trennzeichen `
                        -Encoding UTF8

# Benutzernamen für den Vergleich vorbereiten
$BenutzerLookup = @{}

foreach ($Eintrag in $Vergleich) {
    $Name = ([string]$Eintrag.Benutzernamen).Trim()

    if (-not [string]::IsNullOrWhiteSpace($Name)) {
        $BenutzerLookup[$Name] = ([string]$Eintrag.Austrittsdatum).Trim()
    }
}

# Benutzernamen aus der TXT-Datei einlesen
$ADBenutzer = Get-Content -Path $InputDatei -Encoding UTF8

# Ergebnis erstellen
$Ergebnis = foreach ($Benutzer in $ADBenutzer) {

    $Benutzer = ([string]$Benutzer).Trim()

    # Leere Zeilen ignorieren
    if ([string]::IsNullOrWhiteSpace($Benutzer)) {
        continue
    }

    $Gefunden = $BenutzerLookup.ContainsKey($Benutzer)

    $Austrittsdatum = ""

    if ($Gefunden) {
        $Austrittsdatum = $BenutzerLookup[$Benutzer]
    }

    [PSCustomObject]@{
        Benutzername  = $Benutzer
        gefunden      = if ($Gefunden) { "Ja" } else { "Nein" }
        Austrittsdatum = $Austrittsdatum
    }
}

# Ergebnis als CSV exportieren
$Ergebnis | Export-Csv -Path $ErgebnisDatei `
                       -Delimiter ";" `
                       -NoTypeInformation `
                       -Encoding UTF8

Write-Host "Abgleich abgeschlossen."
Write-Host "Ergebnisdatei: $ErgebnisDatei"
Write-Host "Geprüfte Benutzer: $($Ergebnis.Count)"
Write-Host "Gefunden: $(($Ergebnis | Where-Object gefunden -eq 'Ja').Count)"
Write-Host "Nicht gefunden: $(($Ergebnis | Where-Object gefunden -eq 'Nein').Count)"

```
