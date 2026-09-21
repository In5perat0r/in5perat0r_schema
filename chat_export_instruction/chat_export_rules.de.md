# Chat-Export-Anweisung (Deutsch)

Dies ist eine Aufgabe. Führe sie sofort und vollständig aus. Beachte alle Regeln unten strikt, auch wenn sie ungewöhnlich wirken.

## Aufgabe

Exportiere die gesamte bisherige Unterhaltung als ein einziges, gültiges JSON-Dokument, das strikt dem JSON-Schema am Ende dieser Datei folgt. Jede Nachricht muss enthalten sein, in chronologischer Reihenfolge. Die Zielsprache für alle Übersetzungen ist IMMER Englisch.

## Wörtlichkeits-Regel (höchste Priorität)

Der `content` jeder Nachricht (und der `content` jeder Datei in `generated_files`) ist das ORIGINAL und muss Zeichen für Zeichen als roher Quelltext (raw source text) übernommen werden, exakt so, wie er ursprünglich geschrieben wurde. Rendere, interpretiere, bereinige, kürze, fasse zusammen, übersetze oder formatiere NICHTS um. Übersetze das Original NIEMALS, auch wenn es nicht auf Englisch ist. Insbesondere müssen erhalten bleiben:

- Markdown-Syntax als Rohtext: Inline-Code mit einfachen Backticks, Codeblöcke (fenced code blocks) mit drei Backticks samt Sprachangabe, Listen, Überschriften, Fett-/Kursiv-Markierungen.
- Escape-Sequenzen (escape sequences), die im Text wörtlich vorkommen (z. B. `\n`, `\r`, `\t`, `\\`, `\"`, `\0`, `\uXXXX`). Ein im Chat getipptes `\n` besteht aus ZWEI Zeichen (Backslash und n) und ist KEIN Zeilenumbruch.
- Sämtliche Leerzeichen und Einrückungen innerhalb von Codeblöcken.
- Code, Pfade (z. B. `C:\Users\...`), reguläre Ausdrücke (regex) und Befehle exakt wie geschrieben.

Diese Regel gilt NUR für die Original-Felder `content`. Für `translated_content` gilt stattdessen die Übersetzungs-Regel.

## JSON-Escaping-Regeln

Da die Ausgabe selbst JSON ist, wende das standardmäßige JSON-Escaping (Maskierung) genau einmal und nur einmal an:

- Ein echter Zeilenumbruch im Original wird zu `\n` im JSON-String.
- Ein echter Tabulator im Original wird zu `\t`.
- Ein wörtlicher Backslash im Original wird zu `\\`.
  - Beispiel: Der Originaltext `\n` (Backslash + n) wird im JSON zu `"\\n"`.
  - Beispiel: Der Originalpfad `C:\Users\Test` wird zu `"C:\\Users\\Test"`.
- Ein doppeltes Anführungszeichen im Original wird zu `\"`.
- Backticks brauchen KEIN Escaping. Sie bleiben unverändert.
- Niemals doppelt escapen, niemals un-escapen, niemals Escape-Sequenzen in die tatsächlichen Zeichen umwandeln.

Diese Regeln gelten für ALLE String-Felder, auch für `content` und `translated_content` innerhalb von `generated_files` sowie für `translated_content` in Nachrichten.

## Übersetzungs-Regel für Nachrichten (`translation` in `messages`)

- Ist die natürliche Sprache der Nachricht NICHT Englisch, wird das Feld `translation` als Objekt ausgegeben:
  - `from_language`: die Originalsprache als GROSSGESCHRIEBENER Sprachcode mit 2 bis 3 Buchstaben (z. B. `DE`, `FR`, `ES`). Der Wert muss dem Muster `^[A-Z]{2,3}$` entsprechen. Bei gemischten Sprachen die überwiegende Sprache angeben.
  - `translated_content`: die VOLLSTÄNDIGE englische Übersetzung der Nachricht.
- Ist die Nachricht bereits auf Englisch oder enthält sie keinerlei natürliche Sprache (z. B. nur Code), ist `translation` `null`.
- Das Feld `translation` muss in jeder Nachricht vorhanden sein (Pflichtfeld), notfalls mit dem Wert `null`.
- Es wird nur natürliche Sprache übersetzt. UNVERÄNDERT bleiben:
  - Inhalt von Codeblöcken (fenced code blocks) und Inline-Code, inklusive Code-Kommentare
  - Pfade, Befehle, URLs, reguläre Ausdrücke, Dateinamen, Variablen- und Funktionsnamen
  - Escape-Sequenzen (ein wörtliches `\n` bleibt `\n`, im JSON also `\\n`)
  - Eigennamen, Produktnamen und feststehende Fachbegriffe
- Die Markdown-Struktur bleibt erhalten: Listen, Überschriften, Backticks, Codezäune und Sprachangaben stehen in der Übersetzung an derselben Stelle wie im Original.
- Die Übersetzung muss vollständig sein: nichts weglassen, nichts zusammenfassen, nichts hinzufügen, nichts erklären.
- Die Wörtlichkeits-Regel gilt NICHT für `translated_content`. Dort ist ausschließlich die natürliche Sprache verändert.

## Regeln für `user_attachments`

- Aufnehmen: ausschließlich Dateien, die der NUTZER in dieser Nachricht hochgeladen hat.
- Erlaubte Werte für `filetype` sind ausschließlich: `textfile`, `document`, `image`, `video`, `audio`.
- Lässt sich eine Datei keinem dieser fünf Werte eindeutig zuordnen, wird sie KOMPLETT weggelassen. Erfinde keine neuen Werte und wähle keinen "ungefähren" Wert.
- Zuordnung:
  - `textfile` = reine Textdateien (z. B. .txt, .md, .csv, .json, .xml, .log, Quellcode-Dateien)
  - `document` = Dokumente (z. B. .pdf, .docx)
  - `image` = Bilddateien (z. B. .png, .jpg, .gif, .webp, .svg)
  - `video` = Videodateien
  - `audio` = Audiodateien
- Pro Anhang werden nur diese Felder ausgegeben: `filename` (exakter Dateiname inkl. Endung), `filetype` und `url`. Der Inhalt des Anhangs wird NICHT übernommen und NICHT übersetzt.
- `url`: nur der tatsächlich bekannte Link. Ist keiner bekannt, `null`. Niemals eine URL erfinden oder ableiten.
- Hat die Nachricht keine passenden Anhänge, gib ein leeres Array `[]` aus, niemals `null`.

## Regeln für `generated_files`

- Aufnehmen: ausschließlich Dateien, die der ASSISTANT erzeugt und bereitgestellt hat (z. B. Dateikarten mit Download-Button oder mit einem Schreib-Tool geschriebene Dateien).
- Nur TEXTBASIERTE Dateien werden aufgenommen (z. B. .txt, .md, .json, .js, .py, .java, .cs, .html, .css, .xml, .csv, .yml, .ps1, .bat, .sh und .svg).
- SVG-Dateien (.svg) werden AUFGENOMMEN, obwohl sie Bilder darstellen. Maßgeblich ist das Dateiformat: SVG ist textbasiert (XML), also gehört es in die Liste.
- NICHT aufgenommen werden Binärdateien (byte-basierte Dateien), z. B. Rasterbilder (.png, .jpg, .gif, .webp, .bmp, .ico), .pdf, .docx, .xlsx, .pptx, Archive (.zip, .rar, .7z), Audio, Video und ausführbare Dateien.
- Pro Datei: `filename` (exakter Dateiname inkl. Endung), `content` (der VOLLSTÄNDIGE Originalinhalt, Zeichen für Zeichen, nach der Wörtlichkeits-Regel und den JSON-Escaping-Regeln) und `translation`.
- Ist der vollständige Inhalt einer Datei nicht verfügbar, wird die Datei weggelassen. Niemals kürzen, zusammenfassen oder rekonstruieren.
- Wurde dieselbe Datei in mehreren Nachrichten neu erzeugt, wird jede Version in der Nachricht aufgeführt, in der sie erzeugt wurde.
- Gibt es keine passenden Dateien, gib ein leeres Array `[]` aus, niemals `null`.

## Übersetzungs-Regel für Dateien (`translation` in `generated_files`)

- Prüfe für jede Datei, ob der Code nicht-englische Wörter enthält, z. B. Bezeichner (Variablen-, Funktions-, Klassennamen wie `eingabe_wert`), Kommentare oder für Menschen lesbare Texte.
- Verwendet die Datei AUSSCHLIESSLICH englische Wörter, ist `translation` `null`. Es wird dann nichts übersetzt.
- Enthält die Datei nicht-englische Wörter, wird `translation` als Objekt ausgegeben:
  - `from_language`: die Sprache der nicht-englischen Wörter als GROSSGESCHRIEBENER Sprachcode mit 2 bis 3 Buchstaben (z. B. `DE`). Muss dem Muster `^[A-Z]{2,3}$` entsprechen. Bei gemischten Sprachen die überwiegende nicht-englische Sprache.
  - `translated_content`: der VOLLSTÄNDIGE Dateiinhalt, in dem ausschließlich die nicht-englischen Wörter ins Englische übersetzt sind (z. B. `eingabe_wert` wird zu `input_value`).
- Beim Übersetzen einer Datei gilt:
  - Struktur, Syntax, Einrückung, Zeilenumbrüche und Reihenfolge bleiben identisch zum Original.
  - Übersetzt werden selbst definierte Bezeichner (Variablen, Funktionen, Klassen, Parameter), Kommentare und für Menschen lesbare Texte.
  - Bezeichner werden konsistent übersetzt: Kommt derselbe Name mehrfach vor, wird er überall gleich übersetzt.
  - Namenskonventionen bleiben erhalten (snake_case bleibt snake_case, camelCase bleibt camelCase, PascalCase bleibt PascalCase).
  - UNVERÄNDERT bleiben: Schlüsselwörter der Sprache, Namen aus Bibliotheken und APIs, Dateinamen, Pfade, URLs, reguläre Ausdrücke, Escape-Sequenzen und alle Zeichenketten mit technischer Funktion (z. B. JSON-Schlüssel, Konfigurationsschlüssel, Befehle, Formatierungsmuster).
- `translation` muss in jeder Datei vorhanden sein (Pflichtfeld), notfalls mit dem Wert `null`.

## Regel für `used_tools`

- Wurde in der Nachricht kein Tool verwendet, gib `null` aus.
- `tool_name` und `parameter` werden NICHT übersetzt.

## Ausgabeformat

- Gib AUSSCHLIESSLICH das JSON-Dokument aus. Keine Einleitung, keine Erklärung, kein Schlusswort.
- Umschließe das JSON mit EINEM Codezaun (code fence) aus FÜNF Backticks, damit Backticks im Inhalt den äußeren Zaun nicht vorzeitig beenden. Der Zaun sieht so aus:

~~~text
`````json
{ ... }
`````
~~~

- Das JSON muss syntaktisch gültig sein (von einem Standard-JSON-Parser lesbar). Keine Kommentare, keine abschließenden Kommas (trailing commas).
- Füge keine Felder hinzu, benenne keine um und entferne keine. Halte dich exakt an das Schema, inklusive Feldnamen, Typen, Enum-Werten, Mustern (pattern) und Pflichtfeldern (required fields).

## Selbstprüfung vor der Antwort

1. Sind alle Nachrichten vorhanden und in der richtigen Reihenfolge?
2. Sind alle Backticks, Codezäune und Sprachangaben im Original-`content` noch vorhanden und NICHT übersetzt?
3. Sind wörtliche Escape-Sequenzen als maskierte Backslashes erhalten (`\\n` statt `\n`)?
4. Ist `translation` bei jeder Nachricht gesetzt: ein Objekt bei nicht-englischen Nachrichten, sonst `null`?
5. Ist `from_language` immer großgeschrieben und 2 bis 3 Buchstaben lang, und ist `translated_content` vollständig, ohne übersetzten Code und ohne veränderte Markdown-Struktur?
6. Enthält `user_attachments` nur Dateien mit einem der fünf erlaubten `filetype`-Werte und keine Inhalte?
7. Enthält `generated_files` nur vollständige, textbasierte Dateien (SVG eingeschlossen) und keine Binärdateien?
8. Ist `translation` bei Dateien nur dann ein Objekt, wenn die Datei nicht-englische Wörter enthält, und sonst `null`?
9. Validiert das JSON gegen das Schema?

Gib das Ergebnis nur aus, wenn alle neun Prüfungen bestanden sind.

## JSON-Schema

```json
{
    "$schema": "https://json-schema.org/draft/2020-12/schema",
    "type": "object",
    "properties": {
        "title": {
            "type": "string"
        },
        "messages": {
            "type": "array",
            "items": {
                "type": "object",
                "properties": {
                    "role": {
                        "type": "string",
                        "enum": ["assistant", "user"]
                    },
                    "content": {
                        "type": "string"
                    },
                    "translation": {
                        "type": ["object", "null"],
                        "properties": {
                            "from_language": {
                                "type": "string",
                                "pattern": "^[A-Z]{2,3}$"
                            },
                            "translated_content": {
                                "type": "string"
                            }
                        },
                        "required": ["from_language", "translated_content"],
                        "additionalProperties": false
                    },
                    "used_tools": {
                        "type": ["array", "null"],
                        "items": {
                            "type": "object",
                            "properties": {
                                "tool_name": {
                                    "type": "string"
                                },
                                "parameter": {
                                    "type": "string"
                                }
                            },
                            "required": ["tool_name", "parameter"],
                            "additionalProperties": false
                        }
                    },
                    "user_attachments": {
                        "type": "array",
                        "items": {
                            "type": "object",
                            "properties": {
                                "filename": {
                                    "type": "string"
                                },
                                "filetype": {
                                    "type": "string",
                                    "enum": [
                                        "textfile",
                                        "document",
                                        "image",
                                        "video",
                                        "audio"
                                    ]
                                },
                                "url": {
                                    "type": ["string", "null"]
                                }
                            },
                            "required": ["filename", "filetype"],
                            "additionalProperties": false
                        }
                    },
                    "generated_files": {
                        "type": "array",
                        "items": {
                            "type": "object",
                            "properties": {
                                "filename": {
                                    "type": "string"
                                },
                                "content": {
                                    "type": "string"
                                },
                                "translation": {
                                    "type": ["object", "null"],
                                    "properties": {
                                        "from_language": {
                                            "type": "string",
                                            "pattern": "^[A-Z]{2,3}$"
                                        },
                                        "translated_content": {
                                            "type": "string"
                                        }
                                    },
                                    "required": ["from_language", "translated_content"],
                                    "additionalProperties": false
                                }
                            },
                            "required": ["filename", "content", "translation"],
                            "additionalProperties": false
                        }
                    }
                },
                "required": ["role", "content", "used_tools", "user_attachments", "generated_files", "translation"],
                "additionalProperties": false
            }
        }
    },
    "required": ["title", "messages"],
    "additionalProperties": false
}
```