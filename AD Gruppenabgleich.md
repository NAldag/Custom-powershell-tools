# Werkzeuge zur Erleichterung der Gruppenmigration

Auszug und Abgleich zweier Gruppen

```powershell

# Modul laden
Import-Module ActiveDirectory

# Variablen definieren
$GruppeA = "CN=Gruppe-A,OU=Gruppen,DC=domain,DC=de"
$GruppeB = "CN=Gruppe-B,OU=Gruppen,DC=domain,DC=de"

# Mitglieder abrufen
$MitgliederA = Get-ADGroupMember -Identity $GruppeA | Select-Object -ExpandProperty SamAccountName
$MitgliederB = Get-ADGroupMember -Identity $GruppeB | Select-Object -ExpandProperty SamAccountName

# Abgleich durchführen
Compare-Object -ReferenceObject $MitgliederA -DifferenceObject $MitgliederB

```


Export in CSV


```Powershell

Get-ADGroupMember -Identity "Gruppenname" | Select-Object Name, SamAccountName | Export-Csv -Path "C:\Temp\Gruppenmitglieder.csv" -NoTypeInformation -Encoding UTF8


```


Übertragung der Differenz


```powershell

Get-ADGroupMember -Identity "Gruppe-A" | ForEach-Object {
    Add-ADGroupMember -Identity "Gruppe-B" -Members $_.SamAccountName
}

```

