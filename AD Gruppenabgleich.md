# Werkzeuge zur Erleichterung der Gruppenmigration

Disclaimer: Erstentwürfe, die auf die jeweilige Umgebung angepasst werden müssen

## Auszug und Abgleich zweier Gruppen

```powershell

Import-Module ActiveDirectory

$GruppeA = "CN=Gruppe-A,OU=Gruppen,DC=domain,DC=de"
$GruppeB = "CN=Gruppe-B,OU=Gruppen,DC=domain,DC=de"

# Mitglieder abrufen
$MitgliederA = Get-ADGroupMember -Identity $GruppeA | Select-Object -ExpandProperty SamAccountName
$MitgliederB = Get-ADGroupMember -Identity $GruppeB | Select-Object -ExpandProperty SamAccountName

# Abgleich durchführen
Compare-Object -ReferenceObject $MitgliederA -DifferenceObject $MitgliederB

```


## Export in CSV


```Powershell

Get-ADGroupMember -Identity "Gruppenname" | Select-Object Name, SamAccountName | Export-Csv -Path "C:\Temp\Gruppenmitglieder.csv" -NoTypeInformation -Encoding UTF8


```


## Übertragung der Differenz


```powershell

Get-ADGroupMember -Identity "Gruppe-A" | ForEach-Object {
    Add-ADGroupMember -Identity "Gruppe-B" -Members $_.SamAccountName
}

```

## Alternativ: CSV Vergleich und Übertrag der Variable FehlendeUser

```powershell


$SollMitglieder = Import-Csv -Path "C:\temp\mitarbeiter.csv" | Select-Object -ExpandProperty AccountName
$IstMitglieder = Get-ADGroupMember -Identity "Deine-AD-Gruppe" | Select-Object -ExpandProperty SamAccountName

$FehlendeUser = Compare-Object -ReferenceObject $SollMitglieder -DifferenceObject $IstMitglieder | Where-Object SideIndicator -eq "<="

# Fehlende Benutzer der Gruppe hinzufügen
foreach ($User in $FehlendeUser) {
    Add-ADGroupMember -Identity "Deine-AD-Gruppe" -Members $User.InputObject
}

```

