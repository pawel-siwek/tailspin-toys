# Tailspin Toys

> 🇵🇱 **Repozytorium szablonowe na GitHub Dev Days Gdańsk 2026.**
> Instrukcja warsztatu po polsku: **https://pawel-siwek.github.io/gdn-dev-days-2026/cli-workshop/real-world-development/cli/**
> Zacznij od kliknięcia **Use this template** → **Create a new repository**.

Tailspin Toys to platforma crowdfundingowa dla gier o tematyce programistycznej. Projekt jest stroną fikcyjnej firmy zbierającej fundusze na gry, zbudowaną jako pojedyncza witryna [Astro](https://astro.build/) (w pełni prerenderowana, wyjście statyczne), ostylowana [Tailwind CSS](https://tailwindcss.com/). Dane leżą w lokalnej bazie SQLite, do której dostęp odbywa się przez [Drizzle ORM](https://orm.drizzle.team/) i wbudowany sterownik SQLite w Node.js; strony odpytują bazę bezpośrednio we frontmatterze w czasie budowania, więc nie ma osobnego backendu.

## Architektura

- **Astro 7** — strony, layouty, komponenty i routing. `output: 'static'`, więc cała witryna jest prerenderowana do HTML w czasie budowania.
- **Drizzle ORM + Node SQLite** — warstwa danych. Schemat w `db/schema.ts`, dane zasilane z `db/games.csv`. Migracjami zarządza `drizzle-kit`.
- **Tailwind CSS v4** — stylowanie klasami narzędziowymi (motyw ciemny).
- **Vitest** — testy jednostkowe warstwy danych i czystych transformacji.
- **Playwright** — testy end-to-end uruchamiane na zbudowanej witrynie statycznej.

Komendy deweloperskie są zdefiniowane w `package.json`. Hooki `predev` i `prebuild` uruchamiają migrację i zasilenie bazy z `db/`, zapisując do ignorowanego przez gita pliku `tailspin.db`.

## Jak korzystać z tego szablonu

To repozytorium jest szablonem GitHuba. Kiedy utworzysz z niego nowe repozytorium, jednorazowy workflow **Bootstrap template issues** (`.github/workflows/bootstrap-issues.yml`) uruchomi się automatycznie przy pierwszym pushu na `main` i **założy 9 zgłoszeń po polsku**, opisujących proponowane pierwsze funkcjonalności. Każde zgłoszenie jest zdefiniowane plikiem Markdown w `.github/bootstrap-issues/` — pierwszy nagłówek staje się tytułem, reszta treścią — więc możesz tam edytować, dodawać i usuwać pliki, żeby sterować tym, co powstanie.

Workflow uruchamia się wyłącznie w repozytoriach utworzonych z szablonu (warunek `if: ${{ !github.event.repository.is_template }}` pomija sam szablon), a po założeniu zgłoszeń usuwa siebie oraz katalog `.github/bootstrap-issues/` w commicie porządkowym, żeby nigdy nie wykonał się ponownie.

## Pierwsze kroki

Zainstaluj zależności — wymagany Node.js 22.13 lub nowszy:

```bash
npm ci
npx playwright install chromium   # potrzebne tylko do testów E2E
```

## Uruchomienie strony

```bash
npm run dev
```

`predev` najpierw zmigruje i zasili lokalną bazę. Następnie otwórz [stronę](http://localhost:4321).

Żeby zamiast tego podejrzeć build produkcyjny:

```bash
npm run build      # prebuild migruje i zasila, potem buduje witrynę statyczną
npm run preview
```

## Pochodzenie

Kopia [`github-samples/tailspin-toys`](https://github.com/github-samples/tailspin-toys)
(commit `0b8cd7a7ba9f26b5b880ff773936860ec670ffcc`, licencja MIT), zamrożona na
**GitHub Dev Days Gdańsk**, 28.10.2026. Przetłumaczono backlog 9 zgłoszeń i tę część README.
Instrukcje dla Copilota (`.github/copilot-instructions.md`, `.github/instructions/`,
`.github/skills/`) **celowo pozostawiono po angielsku** — sterują generowaniem kodu,
a tłumaczenie ich pogorszyłoby wyniki i rozjechało się z konwencjami repozytorium.
Pełna atrybucja: [NOTICE.md](https://github.com/pawel-siwek/gdn-dev-days-2026/blob/main/NOTICE.md).

To nie jest oficjalny materiał GitHuba.

## Database

The SQLite database is built from `db/games.csv` — there is no live data to migrate.

```bash
npm run db:generate   # generate a migration after editing db/schema.ts
npm run db:migrate    # apply migrations
npm run db:seed       # seed from games.csv (idempotent)
npm run db:setup      # migrate + seed (run automatically by predev/prebuild)
npm run db:export     # write the seeded catalog to db/catalog.json
```

> [!NOTE]
> Seeding is idempotent — it skips games that already exist (matched by title) rather than reconciling changed rows. CI always starts from a clean database, so it reflects `games.csv` exactly. Locally, if you edit or remove rows in `games.csv`, delete `tailspin.db` and re-run `npm run db:setup` to fully regenerate.

## Running tests

```bash
npm run test:unit   # Vitest unit tests (transforms + data-access helpers)
npm run test:e2e    # Playwright E2E tests (builds + previews the static site first)
```

## Linting

The frontend uses ESLint to enforce code quality across TypeScript and Astro files. Run it with:

```bash
npm run lint
```

ESLint is also run automatically in CI on pull requests to `main`.

## Type checking

The project runs on **TypeScript 7** (the native Go compiler, `tsgo`) for type checking, adopted side-by-side via the [`@typescript/native-preview`](https://www.npmjs.com/package/@typescript/native-preview) package. The classic `typescript` package is intentionally kept at v6 so ESLint + `typescript-eslint` and `astro check` keep working unchanged — TypeScript 7's programmatic API isn't ready for those tools yet.

```bash
npm run typecheck        # tsgo (TS 7) type-checks the pure TypeScript (db/, src/lib/, src/types/, configs, tests)
npm run typecheck:astro  # astro sync + astro check type-check .astro files (on the classic TypeScript package)
npm run typecheck:all    # both of the above
```

`tsgo` runs against [`tsconfig.tsgo.json`](tsconfig.tsgo.json), a scoped config that excludes `.astro` files (which the native compiler doesn't understand). Type checking runs automatically in CI on pull requests to `main`.

> [!NOTE]
> The native compiler is used only for type checking (`--noEmit`); the site is still built by `astro build` (Vite/esbuild). The classic `typescript` package stays on v6 until `typescript-eslint` and `@astrojs/check` support the native API (~TS 7.1); a Dependabot `ignore` in `.github/dependabot.yml` holds the classic `typescript@7` bump until then.

## License 

This project is licensed under the terms of the MIT open source license. Please refer to the [LICENSE](./LICENSE) for the full terms.

## Maintainers 

Szablon warsztatowy utrzymuje [@pawel-siwek](https://github.com/pawel-siwek).

## Support

This project is provided as-is, and may be updated over time. If you have questions, please open an issue.

## Disclaimer

This app is not intended for use in a production environment, nor is it built as an example of what a production app should look like.
