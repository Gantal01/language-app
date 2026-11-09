# Language App

A mobile-friendly app that makes vocabulary learning faster and more fun. Photograph the glossary at the end of a language textbook chapter, and the app turns it into a structured word list you can practice.

> **Status:** work in progress. Features and stack below are partly planned and may change.

## Idea

Textbook glossaries are great, but typing them into flashcards by hand is boring. This app:

- extracts words from a photo of a glossary (AI-based recognition instead of classic OCR),
- groups them by book chapter,
- lets you practice them, with extra focus on the words you got wrong.

It is built mainly for **Spanish, English and German**, but the data model is language-independent.

## Planned features

- **Photo → word list**: AI recognizes the words and translations from a glossary photo. The photo itself is not stored, only the extracted data.
- **Multiple word sources**: book chapters, custom word lists, words extracted from pasted song lyrics.
- **Smart grouping**: sort words by topic and by word type (noun, adjective, ...).
- **Grammar details**: grammatical gender/article, separately stored German plural forms.
- **Mistake tracking**: the app remembers the words you got wrong.
- **On-demand explanations**: AI explains a word only when you ask for it, nothing is pre-generated.
- **Pronunciation** without storing audio files.
- **Shared vocabulary**: one base meaning linked to words in several languages (hu/en/de/es), so you can learn from any language, with synonym support.
- **Multi-user** with authentication from the first version.

## Tech stack (planned)

| Layer | Technology |
|-------|-----------|
| Frontend / mobile | Angular (mobile-first) |
| Backend | .NET (ASP.NET Core) |
| Database | PostgreSQL (server), SQLite (optional local storage on device) |
| AI | Image-to-text word recognition (free options preferred) |

A key constraint: the whole thing should be runnable in a **free environment** (hosting and tooling).

## Getting started

Setup instructions will be added as the project takes shape.

## License

TBD
