

```powershell


# Pfad zur CSV aus dem Exchange-Abgleich
$InputFile = "C:\Temp\Exchange_Abgleich.csv"

# CSV einlesen
$Accounts = Import-Csv -Path $InputFile -Delimiter ';'

# Nur tatsächlich gefundene Empfänger berücksichtigen
$Targets = @(
    $Accounts | Where-Object {
        -not [string]::IsNullOrWhiteSpace($_.SamAccountName) -and
        $_.ExchangeTyp -ne "Nicht gefunden" -and
        $_.ExchangeTyp -ne ""
    }
)

if ($Targets.Count -eq 0) {
    Write-Host "Keine passenden Exchange-Empfänger gefunden."
    return
}

# Vorschau
Write-Host "`nFolgende Empfänger werden aus den Adresslisten ausgeblendet:"
$Targets | Format-Table SamAccountName, ExchangeTyp, PrimaereSMTP, GALAusgeblendet -AutoSize

$Confirm = Read-Host "Änderungen durchführen? (JA eingeben)"

if ($Confirm -cne "JA") {
    Write-Host "Abgebrochen. Es wurden keine Änderungen vorgenommen."
    return
}

foreach ($Target in $Targets) {
    try {
        $Recipient = Get-Recipient -Identity $Target.SamAccountName -ErrorAction Stop

        if ($Recipient.HiddenFromAddressListsEnabled -eq $true) {
            Write-Host "Bereits ausgeblendet: $($Target.SamAccountName)"
            continue
        }

        Set-Mailbox -Identity $Target.SamAccountName `
            -HiddenFromAddressListsEnabled $true `
            -ErrorAction Stop

        Write-Host "Ausgeblendet: $($Target.SamAccountName)"
    }
    catch {
        Write-Warning "Fehler bei $($Target.SamAccountName): $($_.Exception.Message)"
    }
}

```

