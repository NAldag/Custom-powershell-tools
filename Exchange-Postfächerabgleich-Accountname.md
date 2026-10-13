


```powershell

# Pfade anpassen
$InputFile  = "Pfad zur InputDatei"
$OutputFile = "Pfad zur OutputDatei"


# In der Exchange Management Shell ausführen
$Accounts = Import-Csv -Path $InputFile -Delimiter ';'

$Results = foreach ($Account in $Accounts) {

    $SamAccountName = $Account.SamAccountName

    # Exchange-Empfänger anhand des AD-Kontonamens suchen
    $Recipients = @(
        Get-Recipient -ResultSize Unlimited |
            Where-Object {
                $_.SamAccountName -eq $SamAccountName
            }
    )

    if ($Recipients.Count -eq 0) {
        [PSCustomObject]@{
            Nachname         = $Account.Nachname
            Vorname          = $Account.Vorname
            SamAccountName   = $SamAccountName
            ExchangeTyp      = "Nicht gefunden"
            PrimaereSMTP     = ""
            GALAusgeblendet  = ""
        }
        continue
    }

    foreach ($Recipient in $Recipients) {
        [PSCustomObject]@{
            Nachname         = $Account.Nachname
            Vorname          = $Account.Vorname
            SamAccountName   = $SamAccountName
            ExchangeTyp      = $Recipient.RecipientTypeDetails
            PrimaereSMTP     = $Recipient.PrimarySmtpAddress
            GALAusgeblendet  = $Recipient.HiddenFromAddressListsEnabled
        }
    }
}

$Results | Export-Csv -Path $OutputFile `
    -Delimiter ';' -NoTypeInformation -Encoding UTF8BOM

Write-Host "Fertig. Ergebnis gespeichert unter: $OutputFile"

```
