
```powershell

# Pfade anpassen
$InputFile  = "C:\Temp\GefundeneAccounts.csv"
$OutputFile = "C:\Temp\Exchange_Abgleich.csv"

# CSV einlesen
$Accounts = @(Import-Csv -Path $InputFile -Delimiter ';')

Write-Host "Lade Exchange-Empfaenger einmalig ..."

# Exchange-Empfaenger nur einmal abrufen
$AllRecipients = @(Get-Recipient -ResultSize Unlimited -ErrorAction Stop)

Write-Host "Exchange-Empfaenger geladen: $($AllRecipients.Count)"
Write-Host "Starte Abgleich fuer $($Accounts.Count) Accounts ..."

$Results = foreach ($Account in $Accounts) {

    $SamAccountName = $Account.SamAccountName

    Write-Progress `
        -Activity "Exchange-Abgleich" `
        -Status "Pruefe $SamAccountName" `
        -PercentComplete (
            [array]::IndexOf($Accounts, $Account) /
            [math]::Max($Accounts.Count, 1) * 100
        )

    # Leere Accountnamen direkt ausgeben
    if ([string]::IsNullOrWhiteSpace($SamAccountName)) {
        [PSCustomObject]@{
            Nachname        = $Account.Nachname
            Vorname         = $Account.Vorname
            SamAccountName  = ""
            ExchangeTyp     = "Kein AD-Account"
            PrimaereSMTP    = ""
            GALAusgeblendet = ""
        }
        continue
    }

    # Lokaler Abgleich, keine erneute Exchange-Abfrage
    $Recipients = @(
        $AllRecipients | Where-Object {
            $_.SamAccountName -eq $SamAccountName
        }
    )

    if ($Recipients.Count -eq 0) {
        [PSCustomObject]@{
            Nachname        = $Account.Nachname
            Vorname         = $Account.Vorname
            SamAccountName  = $SamAccountName
            ExchangeTyp     = "Nicht gefunden"
            PrimaereSMTP    = ""
            GALAusgeblendet = ""
        }
        continue
    }

    foreach ($Recipient in $Recipients) {
        [PSCustomObject]@{
            Nachname        = $Account.Nachname
            Vorname         = $Account.Vorname
            SamAccountName  = $SamAccountName
            ExchangeTyp     = $Recipient.RecipientTypeDetails
            PrimaereSMTP    = $Recipient.PrimarySmtpAddress
            GALAusgeblendet = $Recipient.HiddenFromAddressListsEnabled
        }
    }
}

Write-Progress -Activity "Exchange-Abgleich" -Completed

# CSV exportieren
$Results | Export-Csv -Path $OutputFile `
    -Delimiter ';' -NoTypeInformation -Encoding UTF8

Write-Host ""
Write-Host "Abgleich abgeschlossen."
Write-Host "Accounts verarbeitet: $($Accounts.Count)"
Write-Host "Ergebnis: $OutputFile"

```


