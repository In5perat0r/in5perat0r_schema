# Chat-Export-Anweisung (Deutsch)

Dies ist eine Aufgabe. Führe sie sofort und vollständig aus. Beachte alle Regeln unten strikt, auch wenn sie ungewöhnlich wirken.

## Aufgabe

Exportiere die gesamte bisherige Unterhaltung als ein einziges, gültiges JSON-Dokument, das strikt dem JSON-Schema am Ende dieser Datei folgt. Jede Nachricht muss enthalten sein, in chronologischer Reihenfolge. Die Zielsprache für alle Übersetzungen ist IMMER Englisch.

## Feldstruktur je nach Rolle (höchste Priorität nach der Wörtlichkeits-Regel)

Welche Felder ein Nachrichten-Objekt enthalten darf, hängt von `role` ab. Die jeweils nicht zutreffenden Felder werden NICHT ausgegeben, auch nicht mit dem Wert `null` — der Schlüssel darf im Objekt schlicht NICHT vorkommen.

- **`role: "user"`**: Das Objekt enthält `role`, `content`, `translation` und `user_attachments`. Die Schlüssel `used_tools` und `generated_files` dürfen bei Nutzer-Nachrichten GAR NICHT vorhanden sein.
- **`role: "assistant"`**: Das Objekt enthält `role`, `content`, `translation`, `used_tools` und `generated_files`. Der Schlüssel `user_attachments` darf bei Assistant-Nachrichten GAR NICHT vorhanden sein.

Diese Trennung ist strikt. Ein Nutzer lädt Anhänge hoch und benutzt keine Tools; ein Assistant benutzt Tools und erzeugt Dateien, lädt aber selbst nichts hoch.

## Wörtlichkeits-Regel

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

Diese Regeln gelten für ALLE String-Werte, auch innerhalb von `generated_files`, `translation` und `used_tools` (inklusive `tool_parameter` und `mcp.result`).

## Übersetzungs-Regel für Nachrichten (`translation` in `messages`)

- Ist die natürliche Sprache der Nachricht NICHT Englisch, wird das Feld `translation` als Objekt ausgegeben:
  - `from_language`: die Originalsprache als GROSSGESCHRIEBENER Sprachcode mit 2 bis 3 Buchstaben (z. B. `DE`, `FR`, `ES`). Der Wert muss dem Muster `^[A-Z]{2,3}$` entsprechen. Bei gemischten Sprachen die überwiegende Sprache angeben.
  - `translated_content`: die VOLLSTÄNDIGE englische Übersetzung der Nachricht.
- Ist die Nachricht bereits auf Englisch oder enthält sie keinerlei natürliche Sprache (z. B. nur Code), ist `translation` `null`.
- Das Feld `translation` muss in jeder Nachricht vorhanden sein (Pflichtfeld), notfalls mit dem Wert `null`. Dies gilt unabhängig von der Rolle.
- Es wird nur natürliche Sprache übersetzt. UNVERÄNDERT bleiben:
  - Inhalt von Codeblöcken (fenced code blocks) und Inline-Code, inklusive Code-Kommentare
  - Pfade, Befehle, URLs, reguläre Ausdrücke, Dateinamen, Variablen- und Funktionsnamen
  - Escape-Sequenzen (ein wörtliches `\n` bleibt `\n`, im JSON also `\\n`)
  - Eigennamen, Produktnamen und feststehende Fachbegriffe
- Die Markdown-Struktur bleibt erhalten: Listen, Überschriften, Backticks, Codezäune und Sprachangaben stehen in der Übersetzung an derselben Stelle wie im Original.
- Die Übersetzung muss vollständig sein: nichts weglassen, nichts zusammenfassen, nichts hinzufügen, nichts erklären.
- Die Wörtlichkeits-Regel gilt NICHT für `translated_content`. Dort ist ausschließlich die natürliche Sprache verändert.

## Regeln für `user_attachments` (NUR bei `role: "user"`)

- Dieses Feld erscheint ausschließlich in Nutzer-Nachrichten. Bei Assistant-Nachrichten fehlt es ganz.
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
- Hat die Nachricht keine passenden Anhänge, gib ein leeres Array `[]` aus, niemals `null`. (Das Feld selbst bleibt trotzdem vorhanden, da es bei `role: "user"` Pflicht ist.)

## Regeln für `generated_files` (NUR bei `role: "assistant"`)

- Dieses Feld erscheint ausschließlich in Assistant-Nachrichten. Bei Nutzer-Nachrichten fehlt es ganz.
- Aufnehmen: ausschließlich Dateien, die der ASSISTANT erzeugt und bereitgestellt hat (z. B. Dateikarten mit Download-Button oder mit einem Schreib-Tool geschriebene Dateien).
- Nur TEXTBASIERTE Dateien werden aufgenommen (z. B. .txt, .md, .json, .js, .py, .java, .cs, .html, .css, .xml, .csv, .yml, .ps1, .bat, .sh und .svg).
- SVG-Dateien (.svg) werden AUFGENOMMEN, obwohl sie Bilder darstellen. Maßgeblich ist das Dateiformat: SVG ist textbasiert (XML), also gehört es in die Liste.
- NICHT aufgenommen werden Binärdateien (byte-basierte Dateien), z. B. Rasterbilder (.png, .jpg, .gif, .webp, .bmp, .ico), .pdf, .docx, .xlsx, .pptx, Archive (.zip, .rar, .7z), Audio, Video und ausführbare Dateien.
- Pro Datei: `filename` (exakter Dateiname inkl. Endung), `content` (der VOLLSTÄNDIGE Originalinhalt, Zeichen für Zeichen, nach der Wörtlichkeits-Regel und den JSON-Escaping-Regeln) und `translation`.
- Ist der vollständige Inhalt einer Datei nicht verfügbar, wird die Datei weggelassen. Niemals kürzen, zusammenfassen oder rekonstruieren.
- Wurde dieselbe Datei in mehreren Nachrichten neu erzeugt, wird jede Version in der Nachricht aufgeführt, in der sie erzeugt wurde.
- Gibt es keine passenden Dateien, gib ein leeres Array `[]` aus, niemals `null`. (Das Feld selbst bleibt trotzdem vorhanden, da es bei `role: "assistant"` Pflicht ist.)

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

## Regeln für `used_tools` (NUR bei `role: "assistant"`)

- Dieses Feld erscheint ausschließlich in Assistant-Nachrichten. Bei Nutzer-Nachrichten fehlt es ganz.
- Wurde in der Nachricht kein Tool verwendet, gib `null` aus (nicht ein leeres Array).
- Wurde mindestens ein Tool verwendet, gib ein Array mit einem Eintrag pro Tool-Aufruf aus, in der Reihenfolge der Aufrufe. Mehrfache Aufrufe desselben Tools erzeugen mehrere Einträge.
- Pro Eintrag:
  - `tool_name`: der exakte Name des aufgerufenen Tools, so wie er technisch verwendet wurde (z. B. `Bash`, `Read`, `WebSearch`, `mcp__claude_ai__image_search`).
  - `tool_parameter`: ein JSON-OBJEKT (kein String) mit den tatsächlich übergebenen Parametern/Argumenten dieses Tool-Aufrufs, als Schlüssel-Wert-Paare. Werte werden nach der Wörtlichkeits-Regel und den JSON-Escaping-Regeln übernommen, NICHT übersetzt. Sind keine Parameter vorhanden, ein leeres Objekt `{}`.
  - `is_mcp`: `true`, wenn es sich um ein MCP-Tool handelt (ein Tool eines verbundenen MCP-Servers, meist erkennbar am Präfix `mcp__` im technischen Namen), sonst `false` für eingebaute/native Tools (z. B. Bash, Read, Edit, Write, WebSearch, WebFetch, Agent, TaskCreate).
  - `mcp`: `null`, wenn `is_mcp` `false` ist. Ist `is_mcp` `true`, ein Objekt mit:
    - `server`: der Name des MCP-Servers. Bei einem technischen Namen nach dem Muster `mcp__<server>__<tool>` ist `<server>` der Teil zwischen dem ersten und dem zweiten doppelten Unterstrich (z. B. bei `mcp__claude_ai__image_search` ist der Server `claude_ai`).
    - `tool`: der Name des konkreten Tools innerhalb dieses Servers, also der Teil nach dem zweiten doppelten Unterstrich (z. B. bei `mcp__claude_ai__image_search` ist das `image_search`).
    - `result`: `null`, wenn das Ergebnis des Tool-Aufrufs nicht bekannt oder nicht sinnvoll rekonstruierbar ist. Ansonsten ein Objekt mit:
      - `is_error`: `true`, wenn der Tool-Aufruf fehlgeschlagen ist bzw. einen Fehler zurückgegeben hat, sonst `false`.
      - `content`: ein Array von Ergebnis-Teilen. Jeder Eintrag hat:
        - `type`: einer von `text`, `image`, `audio`, `resource`, `resource_link`, je nach Art des zurückgegebenen Inhalts.
        - `text`: bei `type: "text"` der VOLLSTÄNDIGE zurückgegebene Text, Zeichen für Zeichen nach der Wörtlichkeits-Regel und den JSON-Escaping-Regeln, NICHT übersetzt. Bei allen anderen Typen (`image`, `audio`, `resource`, `resource_link`) ist `text` `null`, da der eigentliche Inhalt nicht als Text vorliegt.
- Für Tools ohne MCP-Bezug (`is_mcp: false`) wird NICHT versucht, `server`/`tool` künstlich zu befüllen; `mcp` bleibt in diesem Fall strikt `null`.

## Ausgabeformat

- Gib AUSSCHLIESSLICH das JSON-Dokument aus. Keine Einleitung, keine Erklärung, kein Schlusswort.
- Umschließe das JSON mit EINEM Codezaun (code fence) aus FÜNF Backticks, damit Backticks im Inhalt den äußeren Zaun nicht vorzeitig beenden. Der Zaun sieht so aus:

~~~text
`````json
{ ... }
`````
~~~

- Das JSON muss syntaktisch gültig sein (von einem Standard-JSON-Parser lesbar). Keine Kommentare, keine abschließenden Kommas (trailing commas).
- Füge keine Felder hinzu, benenne keine um und entferne keine. Halte dich exakt an das Schema, inklusive Feldnamen, Typen, Enum-Werten, Mustern (pattern), bedingten Regeln (if/then/else) und Pflichtfeldern (required fields).
- Beachte besonders: Felder, die laut Schema bei einer Rolle `false` sind (siehe Feldstruktur je nach Rolle oben), dürfen im jeweiligen Objekt NICHT als Schlüssel auftauchen — auch nicht mit `null`.

## Selbstprüfung vor der Antwort

1. Sind alle Nachrichten vorhanden und in der richtigen Reihenfolge?
2. Enthält jede Nutzer-Nachricht genau `role`, `content`, `translation`, `user_attachments` — und KEINE Schlüssel `used_tools` oder `generated_files`?
3. Enthält jede Assistant-Nachricht genau `role`, `content`, `translation`, `used_tools`, `generated_files` — und KEINEN Schlüssel `user_attachments`?
4. Sind alle Backticks, Codezäune und Sprachangaben im Original-`content` noch vorhanden und NICHT übersetzt?
5. Sind wörtliche Escape-Sequenzen als maskierte Backslashes erhalten (`\\n` statt `\n`)?
6. Ist `translation` bei jeder Nachricht gesetzt: ein Objekt bei nicht-englischen Nachrichten, sonst `null`?
7. Enthält `user_attachments` nur Dateien mit einem der fünf erlaubten `filetype`-Werte und keine Inhalte?
8. Enthält `generated_files` nur vollständige, textbasierte Dateien (SVG eingeschlossen) und keine Binärdateien, mit korrekter `translation`?
9. Ist `used_tools` `null`, wenn kein Tool verwendet wurde, und sonst ein Array mit `tool_name`, `tool_parameter` (Objekt), `is_mcp` und `mcp` je Eintrag?
10. Ist `mcp` bei `is_mcp: false` strikt `null`, und bei `is_mcp: true` ein vollständiges Objekt mit `server`, `tool` und `result`?
11. Validiert das JSON gegen das Schema?

Gib das Ergebnis nur aus, wenn alle elf Prüfungen bestanden sind.

## JSON-Schema

```json
{
    "$id": "https://github.com/In5perat0r/in5perat0r_schema/blob/main/chat_export.json",
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
                                "tool_parameter": {
                                    "type": "object",
                                    "additionalProperties": true
                                },
                                "is_mcp": {
                                    "type": "boolean"
                                },
                                "mcp": {
                                    "type": ["object", "null"],
                                    "properties": {
                                        "server": {
                                            "type": "string"
                                        },
                                        "tool": {
                                            "type": "string"
                                        },
                                        "result": {
                                            "type": ["object", "null"],
                                            "properties": {
                                                "is_error": {
                                                    "type": "boolean"
                                                },
                                                "content": {
                                                    "type": "array",
                                                    "items": {
                                                        "type": "object",
                                                        "properties": {
                                                            "type": {
                                                                "type": "string",
                                                                "enum": ["text", "image", "audio", "resource", "resource_link"]
                                                            },
                                                            "text": {
                                                                "type": ["string", "null"]
                                                            }
                                                        },
                                                        "required": ["type", "text"],
                                                        "additionalProperties": false
                                                    }
                                                }
                                            },
                                            "required": ["is_error", "content"],
                                            "additionalProperties": false
                                        }
                                    },
                                    "required": ["server", "tool", "result"],
                                    "additionalProperties": false
                                }
                            },
                            "required": ["tool_name", "tool_parameter", "is_mcp", "mcp"],
                            "additionalProperties": false,
                            "if": {
                                "properties": { "is_mcp": { "const": true } }
                            },
                            "then": {
                                "properties": { "mcp": { "type": "object" } }
                            },
                            "else": {
                                "properties": { "mcp": { "type": "null" } }
                            }
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
                "required": ["role", "content", "translation"],
                "additionalProperties": false,
                "if": {
                    "properties": { "role": { "const": "assistant" } }
                },
                "then": {
                    "required": ["used_tools", "generated_files"],
                    "properties": { "user_attachments": false }
                },
                "else": {
                    "required": ["user_attachments"],
                    "properties": { "used_tools": false, "generated_files": false }
                }
            }
        }
    },
    "required": ["title", "messages"],
    "additionalProperties": false
}
```