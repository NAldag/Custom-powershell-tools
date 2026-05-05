# Methoden zum Aufräumen der Anmelde Skripte

Status: Erster Entwurf

Zur Vorbereitung der Überführung in GPO Logik und Erleichterung der zwischenzeitlichen Administrierung möchte ich tote/kaputte Skripte, Karteileichen und Altlasten effizient entfernen. Diese Werkzeuge sollen bei der Ergründung helfen.

## Karteileichen finden
Methode A: Last Access Time (sofern aktiv)
Auf dem Fileserver prüfen:
Wann wurde welches Skript zuletzt gelesen?

PowerShell:


```

Get-ChildItem \\domain\netlogon -Recurse |
Where-Object {$_.Extension -eq ".bat"} |
Select-Object Name, LastAccessTime |
Sort-Object LastAccessTime

```

90–180 Tage nicht angefasst > tot

LastAccessTime ist oft deaktiviert 

## Mapping gegen AD (robuster, aber wenn Karteileichen auch in AD vorhanden > ohne effekt)

Skript Schema:

[username].bat
[hostname].bat

Nutzer-Skripte prüfen:

```

$users = Get-ADUser -Filter * -Properties SamAccountName | Select -Expand SamAccountName

Get-ChildItem \\netlogon\*.bat | Where-Object {
    $name = $_.BaseName
    $name -notin $users
}

```

Ergebnis:
Verwaiste User Skripte
Computer-Skripte prüfen:

```

$computers = Get-ADComputer -Filter * -Properties Name | Select -Expand Name

Get-ChildItem \\netlogon\*.bat | Where-Object {
    $name = $_.BaseName
    $name -notin $computers
}

```

Ergebnis:
verwaiste Host-Skripte

## Skripte identifizieren, die abbrechen

Typische Ursachen:
Netzressource existiert nicht mehr/zeigt ins leere > Skript bricht ab

PowerShell: Suche nach Risikobefehlen

```

Get-ChildItem \\netlogon\*.bat -Recurse | ForEach-Object {
    $content = Get-Content $_.FullName
    if ($content -match "net use|rundll32|printui") {
        [PSCustomObject]@{
            File = $_.Name
            Risk = "Contains network/printer ops"
        }
    }
}

```

## Custom Logging einführen (Risikofrei)

### Was wird wirklich aufgerufen?

Per Append an an jedes Skript am Anfang einfügen:

echo %date% %time% %username% %computername% >> \\server\logshare\logon_audit_start.txt

### Welche Skripte starten zwar, brechen aber ab:

Per Append am ENDE jedes Skripts einfügen:

echo %date% %time% %username% %computername% >> \\server\logshare\logon_audit_finish.txt

Nach ein paar Wochen sollten wir nutzbare Information haben.
