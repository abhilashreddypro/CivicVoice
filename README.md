# CivicVoice

A sample **accessible survey & feedback platform** for public agencies, built
as a portfolio project to demonstrate full-stack .NET + front-end skills:
server-rendered ASP.NET Core MVC, a small enterprise-style design system in
Sass, WCAG 2.1 AA accessibility practices, SQL Server/T-SQL, a REST API, and
an Azure DevOps CI/CD pipeline.

It is deliberately similar in spirit to a real "economic survey publishing"
system — admin users build a survey from configurable question types,
publish it to a public, token-based URL, and respondents fill it out on an
accessible public form — but it's a clean-room project with its own domain
(civic/community feedback surveys) and its own code.

> **Why this project exists:** it's a compact, runnable code sample built to
> show the specific combination of skills asked for in .NET + front-end /
> accessibility roles: C#, ASP.NET, SQL Server, hands-on UI work (not just
> backend), WCAG 2.1 AA with real testing tooling, an enterprise design
> system, and Azure DevOps CI/CD — in one project you can read end to end in
> an interview.

---


## Feature tour

- **Dynamic survey rendering** — one Razor view (`Take.cshtml`) renders 13
  different question "display types" (short text, dropdown, radio, Yes/No,
  Likert 1–5, Likert with Not Sure, 1–10 rating scale, percent entry,
  section headings, labels, etc.) purely from data, the same way a real
  survey-builder tool would.
- **Public vs. admin separation** — `SurveyController` is the only
  unauthenticated surface and only ever returns a flat `SurveyTakeViewModel`
  projection, never the EF entity, so no internal IDs or admin fields can
  leak to a respondent's browser. Public links use an opaque `PublicToken`
  instead of the sequential `SurveyId`.
- **Accessible-by-default forms** — semantic `fieldset`/`legend`, visible
  focus rings, a skip link, an `aria-live` status region, and a
  server-confirmed error-summary pattern (see `docs/ACCESSIBILITY.md` for
  the full WCAG mapping).
- **Validation in two layers** — `survey-validation.js` gives instant
  feedback, but `ResponseService.SubmitResponseAsync` re-checks everything
  (required fields, email format, 1–100 and 1–10 ranges) server-side before
  anything is saved.
- **Survey Builder admin screen** — keyboard-accessible question reordering
  (a typed sort-order field, not just drag-and-drop), soft-delete with a
  "question is in use" guard, and a duplicate-question validation pass
  (`ValidateForDuplicateResponseVariablesAsync`).
- **Small enterprise design system** — a token file (`_variables.scss`)
  defines color/type/spacing once; every component (`_components.scss`)
  consumes the tokens instead of hardcoding values, so the whole UI can be
  re-themed from one file.

## Project structure

```
CivicVoice/
├── azure-pipelines.yml          # CI/CD: build, test, Sass compile, axe-core scan, deploy
├── docs/
│   └── ACCESSIBILITY.md         # WCAG 2.1 AA checklist + how it's tested
├── sql/
│   ├── schema.sql               # Tables, trigger, stored procedure (T-SQL)
│   └── seed.sql                 # Sample survey + questions
└── src/CivicVoice.Web/
    ├── Program.cs
    ├── Controllers/
    │   ├── HomeController.cs
    │   ├── SurveyController.cs       # public, unauthenticated
    │   ├── SurveysAdminController.cs # admin
    │   └── Api/ResponsesApiController.cs
    ├── Services/                     # business logic, thin controllers
    ├── Models/ (+ ViewModels/)
    ├── Data/ApplicationDbContext.cs  # EF Core
    ├── Views/                        # Razor views (public + admin)
    └── wwwroot/
        ├── scss/                     # design system source
        ├── css/site.css              # compiled output (checked in so the
        │                             #   app runs without a Sass step too)
        └── js/                       # accessibility.js, survey-validation.js,
                                       #   survey-builder.js
```

## Running it locally

### Prerequisites
- [.NET 8 SDK](https://dotnet.microsoft.com/download)
- SQL Server (LocalDB, SQL Express, or a full instance) — or skip the DB and
  just browse the Razor views/CSS/JS, which don't require a live database
- Node.js (only needed if you want to re-compile the Sass yourself)

### 1. Database
```bash
sqlcmd -S localhost\SQLEXPRESS -i sql/schema.sql
sqlcmd -S localhost\SQLEXPRESS -i sql/seed.sql
```
Update the connection string in `src/CivicVoice.Web/appsettings.json`
(`ConnectionStrings:CivicVoiceDb`) to match your SQL Server instance.

### 2. Run the app
```bash
cd src/CivicVoice.Web
dotnet restore
dotnet run
```
Then open `https://localhost:5001` (or the port shown in the console).

- Admin list: `/SurveysAdmin`
- Public survey (seeded sample): `/s/a1b2c3d4e5f6a7b8`

### 3. (Optional) Re-compile the design system
The compiled `wwwroot/css/site.css` is already checked in, so this step is
only needed if you edit the `.scss` files:
```bash
npm install -g sass
sass src/CivicVoice.Web/wwwroot/scss/site.scss src/CivicVoice.Web/wwwroot/css/site.css
```

### 4. (Optional) Accessibility scan
```bash
npm install -g @axe-core/cli
axe http://localhost:5001/s/a1b2c3d4e5f6a7b8 --exit
```

## Notes on scope

This is a **portfolio-scale** sample, not a production system — a few
things are intentionally simplified so the whole project stays readable in
one sitting:

- No authentication/authorization is wired up (`SurveysAdminController`
  notes where `[Authorize]` would go in a real deployment).
- The "question search autocomplete" on the Builder screen is stubbed to
  show the intended UX; a real build would hit a scoped `/api` endpoint.
- Unit tests are referenced from the pipeline (`dotnet test`) but a full
  test project isn't included here — happy to add one if useful for
  discussion.

## Tech stack

C# · ASP.NET Core MVC · Entity Framework Core · SQL Server / T-SQL ·
Newtonsoft.Json · HTML5 · Sass (CSS) · vanilla JavaScript · Azure DevOps
(CI/CD) · axe-core (automated accessibility testing)
