# Chat Export Instructions (English)

This is a task. Execute it immediately and completely. Follow all rules below strictly, even if they seem unusual.

## Task

Export the complete conversation so far as a single valid JSON document that strictly follows the JSON schema at the end of this file. Every message must be included, in chronological order. The target language for all translations is ALWAYS English.

## Verbatim Rule (highest priority)

The `content` of every message (and the `content` of every file in `generated_files`) is the ORIGINAL and must be reproduced character-for-character as the raw source text, exactly as originally written. Do NOT render, interpret, clean up, summarize, shorten, translate, or reformat anything. NEVER translate the original, even if it is not in English. Specifically, you must preserve:

- Markdown syntax as raw text: inline code with single backticks, fenced code blocks with triple backticks including their language identifier, lists, headings, bold/italic markers.
- Escape sequences that appear literally in the text (e.g. `\n`, `\r`, `\t`, `\\`, `\"`, `\0`, `\uXXXX`). A literal backslash-n typed in the chat is TWO characters (backslash and n), not a line break.
- All whitespace and indentation inside code blocks.
- Any code, paths (e.g. `C:\Users\...`), regexes and commands exactly as written.

This rule applies ONLY to the original `content` fields. For `translated_content` the Translation Rule applies instead.

## JSON Escaping Rules

Because the output itself is JSON, apply standard JSON string escaping exactly once, and only once:

- A real line break in the original becomes `\n` in the JSON string.
- A real tab in the original becomes `\t`.
- A literal backslash in the original becomes `\\`.
  - Example: original text `\n` (backslash + n) becomes `"\\n"` in JSON.
  - Example: original path `C:\Users\Test` becomes `"C:\\Users\\Test"`.
- A double quote in the original becomes `\"`.
- Backticks need NO escaping. Keep them as they are.
- Never double-escape, never un-escape, never convert escape sequences into their actual characters.

These rules apply to ALL string fields, including `content` and `translated_content` inside `generated_files` and `translated_content` in messages.

## Translation Rule for Messages (`translation` in `messages`)

- If the natural language of the message is NOT English, the field `translation` is output as an object:
  - `from_language`: the original language as an UPPERCASE language code with 2 to 3 letters (e.g. `DE`, `FR`, `ES`). The value must match the pattern `^[A-Z]{2,3}$`. For mixed languages, use the predominant language.
  - `translated_content`: the COMPLETE English translation of the message.
- If the message is already in English or contains no natural language at all (e.g. only code), `translation` is `null`.
- The field `translation` must be present in every message (required field), with the value `null` if necessary.
- Only natural language is translated. These stay UNCHANGED:
  - the content of code blocks (fenced code blocks) and inline code, including code comments
  - paths, commands, URLs, regular expressions, file names, variable and function names
  - escape sequences (a literal `\n` stays `\n`, which is `\\n` in the JSON)
  - proper names, product names and established technical terms
- The Markdown structure is preserved: lists, headings, backticks, code fences and language identifiers appear in the translation at the same places as in the original.
- The translation must be complete: do not omit, summarize, add or explain anything.
- The verbatim rule does NOT apply to `translated_content`. There, only the natural language is changed.

## Rules for `user_attachments`

- Include ONLY files that the USER uploaded in that message.
- The only allowed values for `filetype` are: `textfile`, `document`, `image`, `video`, `audio`.
- If a file cannot be clearly assigned to one of these five values, OMIT it completely. Do not invent new values and do not pick an "approximate" value.
- Mapping:
  - `textfile` = plain text files (e.g. .txt, .md, .csv, .json, .xml, .log, source code files)
  - `document` = documents (e.g. .pdf, .docx)
  - `image` = image files (e.g. .png, .jpg, .gif, .webp, .svg)
  - `video` = video files
  - `audio` = audio files
- For each attachment output only these fields: `filename` (exact file name including extension), `filetype` and `url`. Do NOT include or translate the attachment's content.
- `url`: only the actually known link. If none is known, use `null`. Never invent or derive a URL.
- If the message has no matching attachments, output an empty array `[]`, never `null`.

## Rules for `generated_files`

- Include ONLY files that the ASSISTANT created and provided (e.g. file cards with a download button, or files written with a file-writing tool).
- Include ONLY TEXT-BASED files (e.g. .txt, .md, .json, .js, .py, .java, .cs, .html, .css, .xml, .csv, .yml, .ps1, .bat, .sh and .svg).
- SVG files (.svg) ARE included, even though they represent images. The file format decides: SVG is text-based (XML), so it belongs in the list.
- Do NOT include binary (byte-based) files, e.g. raster images (.png, .jpg, .gif, .webp, .bmp, .ico), .pdf, .docx, .xlsx, .pptx, archives (.zip, .rar, .7z), audio, video and executables.
- For each file: `filename` (exact file name including extension), `content` (the COMPLETE original content, character-for-character, following the verbatim rule and the JSON escaping rules) and `translation`.
- If the complete content of a file is not available, omit the file. Never shorten, summarize or reconstruct it.
- If the same file was re-created in several messages, list each version in the message where it was created.
- If there are no matching files, output an empty array `[]`, never `null`.

## Translation Rule for Files (`translation` in `generated_files`)

- For each file, check whether the code contains non-English words, e.g. identifiers (variable, function, class names such as `eingabe_wert`), comments or human-readable texts.
- If the file uses EXCLUSIVELY English words, `translation` is `null`. Nothing is translated then.
- If the file contains non-English words, `translation` is output as an object:
  - `from_language`: the language of the non-English words as an UPPERCASE language code with 2 to 3 letters (e.g. `DE`). Must match the pattern `^[A-Z]{2,3}$`. For mixed languages, use the predominant non-English language.
  - `translated_content`: the COMPLETE file content in which only the non-English words are translated into English (e.g. `eingabe_wert` becomes `input_value`).
- When translating a file:
  - Structure, syntax, indentation, line breaks and order stay identical to the original.
  - Self-defined identifiers (variables, functions, classes, parameters), comments and human-readable texts are translated.
  - Identifiers are translated consistently: if the same name occurs several times, it is translated the same way everywhere.
  - Naming conventions are preserved (snake_case stays snake_case, camelCase stays camelCase, PascalCase stays PascalCase).
  - These stay UNCHANGED: language keywords, names from libraries and APIs, file names, paths, URLs, regular expressions, escape sequences and all strings with a technical function (e.g. JSON keys, configuration keys, commands, format patterns).
- `translation` must be present in every file (required field), with the value `null` if necessary.

## Rule for `used_tools`

- If no tool was used in the message, output `null`.
- `tool_name` and `parameter` are NOT translated.

## Output Format

- Output ONLY the JSON document. No introduction, no explanation, no closing remarks.
- Wrap the JSON in ONE fenced code block that uses FIVE backticks, so that backticks inside the content cannot break the outer fence. The fence looks like this:

~~~text
`````json
{ ... }
`````
~~~

- The JSON must be syntactically valid (parsable by a standard JSON parser). No comments, no trailing commas.
- Do not add, rename or remove fields. Follow the schema exactly, including field names, types, enum values, patterns and required fields.

## Self-Check Before Answering

1. Is every message present and in the correct order?
2. Are all backticks, code fences and language identifiers still present in the original `content` and NOT translated?
3. Are literal escape sequences preserved as escaped backslashes (`\\n` instead of `\n`)?
4. Is `translation` set for every message: an object for non-English messages, otherwise `null`?
5. Is `from_language` always uppercase and 2 to 3 letters long, and is `translated_content` complete, without translated code and without a changed Markdown structure?
6. Does `user_attachments` contain only files with one of the five allowed `filetype` values, and no file contents?
7. Does `generated_files` contain only complete, text-based files (SVG included) and no binary files?
8. Is `translation` in files an object only if the file contains non-English words, and otherwise `null`?
9. Does the JSON validate against the schema?

Only output the result if all nine checks pass.

## JSON Schema

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