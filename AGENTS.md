# Roomie — catatan untuk agent

Aplikasi reservasi dan monitoring ruang kelas. Repo ini masih skeleton Laravel 13 yang kosong: hanya ada `routes/web.php` (`/` → view `welcome`) dan tabel `users`. Semua fitur domain dibangun dari `PRD.md`.

## Baca PRD.md sebelum mengerjakan fitur apa pun

`PRD.md` (bahasa Indonesia) adalah spesifikasi: FR-01..FR-11, data model, kosakata status, dan spesifikasi QR. Bagian 11 "Assumptions dan Open Questions" memuat keputusan yang sudah disepakati — ikuti, jangan diasumsikan ulang:

- Hanya PJ kelas yang boleh reservasi; satu reservasi untuk satu kelas.
- Reservasi langsung terkonfirmasi, tidak ada approval dan tidak ada status `PENDING`.
- Durasi diturunkan dari SKS yang diinput setelah scan QR: 1 SKS = 50 menit.
- Integrasi ESP32 memakai HTTP dan berada di luar scope MVP; status fisik tetap `UNKNOWN` — jangan membuat data sensor palsu hanya agar UI terisi.
- QR berisi token ruangan yang unik dan sulit ditebak, tanpa kredensial, dan token tidak memberi hak akses apa pun. Untuk testing lokal, URL `localhost` pada QR akan mengarah ke HP pemindai, bukan ke PC pengembang — bangun URL absolut lewat `route()`/`APP_URL`.

## Perintah

- `composer run dev` — satu-satunya jalan dev full-stack: `artisan serve` (port 8000) + `queue:listen --tries=1` + `npm run dev`.
- `php artisan test --compact` (atau `composer test`, yang juga menjalankan `config:clear`) — persempit dengan path file atau `--filter=` sesuai perubahan.
- `vendor/bin/pint --dirty` setelah mengubah file PHP.
- `php artisan make:test --pest Nama` tanpa prefiks suite. `RefreshDatabase` masih dikomentari di `tests/Pest.php`; aktifkan per test saat menambah tes yang menyentuh DB.

## Output tes berupa JSON, bukan teks PHPUnit

`laravel/pao` (dev dependency) mengambil alih output tes begitu mendeteksi agent dan mencetak satu baris JSON. Set `PAO_DISABLE=1` bila membutuhkan output PHPUnit biasa.

## Database dev: MySQL dikonfigurasi tapi tidak jalan

- `.env` mengarah ke MySQL `roomie` di 127.0.0.1:3306 dan tidak ada yang listen di sana. `php artisan migrate`, `db:show`, tinker, dan tool DB Boost gagal dengan `SQLSTATE[HY000] [2002]`. Nyalakan MySQL dulu atau alihkan `.env` ke sqlite — jangan berasumsi migrasi sudah dijalankan.
- `composer run setup` juga menjalankan `migrate --force`, jadi gagal di clone baru karena alasan yang sama.
- `phpunit.xml` memaksa `sqlite` `:memory:` untuk tes, jadi query di dalam test harus tetap kompatibel dengan SQLite (hindari SQL khusus MySQL).

## Konvensi Laravel 13 (berbeda dari Laravel lama)

- Model memakai atribut PHP untuk mass assignment: `#[Fillable([...])]` dan `#[Hidden([...])]`; cast ditulis di method `casts(): array`. Lihat `app/Models/User.php` — jangan kembalikan properti `$fillable`/`$casts`.
- Routing, middleware, dan konfigurasi exception berada di `bootstrap/app.php` (tidak ada `Http/Kernel.php`). Route ada di `routes/web.php`; `routes/api.php` belum ada.
- Belum ada auth scaffolding, kolom `role`, maupun model/tabel Room, Reservation, RoomSession. Buat dengan `php artisan make:*` mengikuti data model di PRD.

## Frontend

- Blade + Tailwind v4 tanpa `tailwind.config.js`: token tema ada di blok `@theme` pada `resources/css/app.css`. Tambah gaya lewat utility class, jangan membuat file config baru.
- Font disuntikkan dari `vite.config.js` (`laravel-vite-plugin/fonts` → Bunny "Instrument Sans"); baru tersedia setelah `npm run dev` atau `npm run build`.
- `.npmrc` menyetel `ignore-scripts=true`, sehingga postinstall tidak berjalan.
- `/public/build` ada di gitignore: kalau perubahan Blade tampil tanpa CSS, jalankan `npm run build`.

## File hasil generate Boost — jangan diedit manual

Blok `<laravel-boost-guidelines>` di AGENTS.md, `CLAUDE.md` (salinan identik), `.agents/skills/`, `.claude/skills/`, dan `.mcp.json` ditulis oleh `laravel/boost`. Regenerate dengan `php artisan boost:update` (otomatis berjalan setelah `composer update`). Simpan catatan repo hanya di file ini, di atas blok tersebut.

- MCP Boost sudah terhubung di `opencode.json` dan `.mcp.json` (`php artisan boost:mcp`); utamakan tool-nya (`database-schema`, `database-query`, `search-docs`, `get-absolute-url`).
- Skill yang terpasang di `.agents/skills/`: `laravel-best-practices`, `testing-best-practices`, `tailwindcss-development`, `deploying-to-cloud`, `infer-conventions`. Muat skill yang relevan sebelum menulis kode di domain tersebut.
- Direktori `.ai/rules/` tidak ada, jadi lewati langkah itu dari pedoman Boost.

## Keputusan yang masih terbuka — tanya, jangan hardcode

- `config/app.php` masih `UTC` sementara PRD mewajibkan `Asia/Jakarta`. Konfirmasi ke user sebelum menambah fitur bertimestamp atau menggeser tanggal secara sepihak.
- Toleransi keterlambatan check-in dan perlakuan terhadap sesi yang terlupa di-checkout belum diputuskan (PRD §11 butir 6–7).

<laravel-boost-guidelines>
=== foundation rules ===

# Laravel Boost Guidelines

## Foundational Context

This application is a Laravel application running on PHP 8.4. Always use the APIs that match the installed major version of each package — do not assume a version.

Before relying on a package's API, confirm its installed version:
- PHP packages: run `composer show --direct` to list direct dependencies with versions, or `composer show <vendor/package>` for a single package.
- JS packages: check `package.json` for the installed versions.

## Skills Activation

This project has domain-specific skills available in `**/skills/**`. You MUST activate the relevant skill whenever you work in that domain—don't wait until you're stuck.

## Conventions

- You must follow all existing code conventions used in this application. When creating or editing a file, check sibling files for the correct structure, approach, and naming.
- Use descriptive names for variables and methods. For example, `isRegisteredForDiscounts`, not `discount()`.
- Check for existing components to reuse before writing a new one.

## Verification Scripts

- Do not create verification scripts or tinker when tests cover that functionality and prove they work. Unit and feature tests are more important.

## Application Structure & Architecture

- Stick to existing directory structure; don't create new base folders without approval.
- Do not change the application's dependencies without approval.

## Frontend Bundling

- If a frontend change doesn't show in the UI or you get a "Unable to locate file in Vite manifest" error, run `npm run build` or ask the user to run `npm run dev` or `composer run dev`.

## Documentation Files

- You must only create documentation files if explicitly requested by the user.

=== boost rules ===

# Laravel Boost

## Tools

- Laravel Boost is an MCP server with tools designed specifically for this application. Prefer Boost tools over manual alternatives like shell commands or file reads.
- Use `database-query` to run read-only queries against the database instead of writing raw SQL in tinker.
- Use `database-schema` to inspect table structure before writing migrations or models.
- Use `get-absolute-url` to resolve the correct scheme, domain, and port for project URLs. Always use this before sharing a URL with the user.
- Use `browser-logs` to read browser logs, errors, and exceptions. Only recent logs are useful, ignore old entries.

## Searching Documentation (IMPORTANT)

- Use `search-docs` before changes that depend on Laravel ecosystem APIs, behavior, configuration, or version-specific syntax. Skip it for copy-only edits and other changes where package documentation is irrelevant. Reuse sufficient results already in context instead of searching again.
- Pass a `packages` array to scope results when you know which packages are relevant.
- Use multiple broad, topic-based queries: `['rate limiting', 'routing rate limiting', 'routing']`. Expect the most relevant results first.
- Do not add package names to queries because package info is already shared. Use `test resource table`, not `filament 4 test resource table`.

### Search Syntax

1. Use words for auto-stemmed AND logic: `rate limit` matches both "rate" AND "limit".
2. Use `"quoted phrases"` for exact position matching: `"infinite scroll"` requires adjacent words in order.
3. Combine words and phrases for mixed queries: `middleware "rate limit"`.
4. Use multiple queries for OR logic: `queries=["authentication", "middleware"]`.

## Project Rules

- This project contains committed, area-grouped rules in `.ai/rules` when that directory exists, including path-scoped framework guidelines under `.ai/rules/boost`. Before you enter plan mode or create/edit any file, you MUST first: open @.ai/rules/index.md (it maps file globs to rule files), read every rule file whose globs cover the path(s) in scope, and run `grep -rin 'keyword' .ai/rules` to catch what a path match alone misses. Do not write code until you have read and are following every matching rule. If `.ai/rules` does not exist, continue without it.

## Artisan

- Run Artisan commands directly via the command line (e.g., `php artisan route:list`). Use `php artisan list` to discover available commands and `php artisan [command] --help` to check parameters.
- Inspect routes with `php artisan route:list`. Filter with: `--method=GET`, `--name=users`, `--path=api`, `--except-vendor`, `--only-vendor`.
- Read configuration values using dot notation: `php artisan config:show app.name`, `php artisan config:show database.default`. Or read config files directly from the `config/` directory.

## Tinker

- Execute PHP in app context for debugging and testing code. Do not create models without user approval, prefer tests with factories instead. Prefer existing Artisan commands over custom tinker code.
- Always use single quotes to prevent shell expansion: `php artisan tinker --execute 'Your::code();'`
  - Double quotes for PHP strings inside: `php artisan tinker --execute 'User::where("active", true)->count();'`

=== php rules ===

# PHP

- Always use curly braces for control structures, even for single-line bodies.
- Use PHP 8 constructor property promotion: `public function __construct(public GitHub $github) { }`. Do not leave empty zero-parameter `__construct()` methods unless the constructor is private.
- Use explicit return type declarations and type hints for all method parameters: `function isAccessible(User $user, ?string $path = null): bool`
- Use TitleCase for Enum keys: `FavoritePerson`, `BestLake`, `Monthly`.
- Prefer PHPDoc blocks over inline comments. Only add inline comments for exceptionally complex logic.
- Use array shape type definitions in PHPDoc blocks.

=== deployments rules ===

# Deployment

- Laravel can be deployed using [Laravel Cloud](https://cloud.laravel.com/), which is the fastest way to deploy and scale production Laravel applications.
- Activate the `deploying-to-cloud` skill whenever deploying to Laravel Cloud, configuring Cloud environments or resources, using the Cloud CLI, or troubleshooting Cloud deployments.

=== laravel/core rules ===

# Do Things the Laravel Way

- Use `php artisan make:` commands to create new files (i.e. migrations, controllers, models, etc.). You can list available Artisan commands using `php artisan list` and check their parameters with `php artisan [command] --help`.
- If you're creating a generic PHP class, use `php artisan make:class`.
- Pass `--no-interaction` to all Artisan commands to ensure they work without user input. You should also pass the correct `--options` to ensure correct behavior.

### Model Creation

- When creating new models, create useful factories and seeders for them too. Ask the user if they need any other things, using `php artisan make:model --help` to check the available options.

## APIs & Eloquent Resources

- For APIs, default to using Eloquent API Resources and API versioning unless existing API routes do not, then you should follow existing application convention.

## URL Generation

- When generating links to other pages, prefer named routes and the `route()` function.

## Testing

- When creating models for tests, use the factories for the models. Check if the factory has custom states that can be used before manually setting up the model.
- Faker: Use methods such as `$this->faker->word()` or `fake()->randomDigit()`. Follow existing conventions whether to use `$this->faker` or `fake()`.
- When creating tests, make use of `php artisan make:test [options] {name}` to create a feature test, and pass `--unit` to create a unit test. Most tests should be feature tests.

=== pint/core rules ===

# Laravel Pint Code Formatter

- If you have modified any PHP files, you must run `vendor/bin/pint --dirty --format agent` before finalizing changes to ensure your code matches the project's expected style.
- Do not run `vendor/bin/pint --test --format agent`, simply run `vendor/bin/pint --format agent` to fix any formatting issues.

=== pest/core rules ===

# Pest

- This project uses Pest. Create tests with `php artisan make:test --pest {name}`.
- Do not include the test suite directory in `{name}`. Use `SomeFeatureTest`, not `Feature/SomeFeatureTest`.
- Read the `testing-best-practices` skill for guidance on coverage, naming, structure, dependency isolation, and review.
- Do not delete tests or test files without approval. They are part of the application.

## Running Tests

- Run the narrowest set of tests that covers the change. Pass a file path or `--filter=testName` to `php artisan test --compact`.
- Rerun a test after each change to it.
- Run `vendor/bin/pest` to call the test runner directly. It accepts the same file path and `--filter=testName` arguments.
- After the feature tests pass, ask the user to run the complete suite with `php artisan test --compact`.

</laravel-boost-guidelines>
