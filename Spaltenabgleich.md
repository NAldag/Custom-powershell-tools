


```powershell

$daten = Import-Csv "C:\Temp\vergleich.csv" -Delimiter ";"

# Werte aus Spalte B sammeln
$spalteB = $daten.SpalteB

$ergebnis = foreach ($zeile in $daten) {

    $wert = $zeile.SpalteA

    if ($spalteB -contains $wert) {
        [PSCustomObject]@{
            Wert   = $wert
            Status = "Gefunden"
        }
    }
    else {
        [PSCustomObject]@{
            Wert   = $wert
            Status = "Nicht gefunden"
        }
    }
}

$ergebnis | Export-Csv "C:\Temp\ergebnis.csv" -Delimiter ";" -NoTypeInformation -Encoding UTF8

```
