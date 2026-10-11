# Language Learning App

Vocabulary learning app. Words come from photographed book glossaries, pasted song lyrics, or manual entry. Languages: English, German, Spanish, Hungarian; practice works from any language to any other. Multi-user from v1.

**This is a learning project.** The developer is practicing Angular, .NET and PostgreSQL. Teaching matters as much as shipping.

## Learning mode (important)
- Explain in **Hungarian**; code, comments, commits and identifiers stay in English.
- Don't write large chunks of code unasked. For multi-file changes, propose a plan first and wait for approval.
- When introducing a new concept, pattern or library, briefly explain what it is and why.
- If a request leads to bad practice, say so and suggest the better option.
- In reviews, check clean code, not just correctness.

## Stack
- **Frontend:** Angular (standalone components, signals). Mobile packaging: _undecided_
- **Backend:** ASP.NET Core Web API, EF Core, ASP.NET Core Identity (`AddIdentityApiEndpoints`)
- **Database:** PostgreSQL only. v1 is online-only; offline storage/sync is a later task.
- **AI:** word extraction from glossary photos and lyrics (not classic OCR)
- **Constraint:** everything must run on free tiers/tooling

## Data model
Source of truth: @docs/adr/0001-adatmodell.md. Read it before touching entities or migrations. Key invariants:
- Shared zone (`Concept`, `Word`, `ConceptWord`) vs. personal zone (`AppUser`, `Source`, `WordList`, `WordListEntry`). References go personal → shared only.
- Shared data is append-only for users. No update/delete endpoints for it.
- Personal data is filtered by current user in ONE place (EF Core global query filter), never by hand per method.
- AI output is a suggestion; the service matches existing shared entries before saving, the user confirms.
- Don't store photos, lyrics, or where a shared word came from.
- Enums stored as strings.

## Structure
```
/frontend   Angular app
/backend    ASP.NET Core solution
/docs/adr   architecture decision records
```

## Commands
```
cd frontend && npm start
cd frontend && npm test
cd backend && dotnet run --project LanguageApp.Api
cd backend && dotnet test
cd backend && dotnet ef migrations add <Name> --project LanguageApp.Api
```

## Clean code rules
- Descriptive English names. If a name needs a comment, rename it.
- Small functions, one job each. Early returns over nested ifs.
- No magic numbers/strings: constants or enums.
- Comments explain *why*, not *what*.
- Backend layers: thin controller → service holds business logic → EF Core for data. Controllers never touch `DbContext`.
- API never exposes entities; use DTOs.
- Angular: components render, services talk to the backend. No HTTP calls in components.
- No `any` in TypeScript; nullable reference types on in C#.
- New business logic gets a unit test.
- Prefer simple and pragmatic over elaborate architecture (YAGNI).
- New architectural decision → new ADR in `/docs/adr`.

## Open questions
- Mobile packaging (Ionic/Capacitor or other)
- Which free AI model/API for extraction
- Copyright of shared glossary data
