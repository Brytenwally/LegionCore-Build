# LegionCore-Build

GitHub Actions workflow that builds **LegionCore-Reforged 7.3.5 (build 26972)** Windows x64
binaries from `Brytenwally/LegionCore-Reforged`, linked against the **MySQL 8.0.44** client.
Modules (the playerbots port, `Brytenwally/LegionCore-Playerbots`) can be checked out into the
core's `modules/` folder; that is off until the core has a module system.

## Running a build

- **Actions tab → Build Windows binaries → Run workflow.** Inputs: core repo/ref, playerbots
  on/off and repo/ref, extractors on/off, MySQL client version.
- **Push a tag** like `v1.0.0`: builds with the defaults and publishes the zip as a GitHub
  Release.

The result is a workflow artifact named
`LegionCore-735-win64-mysql<ver>-<core sha>[-playerbots-<sha>]`.

## What is in the package

- `worldserver.exe`, `bnetserver.exe` (+ `bnetserver.cert.pem` / `.key.pem`), the
  `.conf.dist` files, and with tools on: `mapextractor`, `vmap4extractor`, `vmap4assembler`,
  `mmaps_generator`.
- `libmysql.dll`, `mysql.exe` (the SQL updater runs it) and the OpenSSL 3 DLLs.
- `sql\`: the core's whole `sql` folder (base dumps, updates, archived updates, the
  `Game.Patch` scripts in `optional`).
- `BUILD-INFO.txt` (versions, commits), `CHANGELOG.md` (commits since the previous build),
  `commits.json`.

Needs the [Visual C++ 2015-2022 x64 redistributable](https://aka.ms/vs/17/release/vc_redist.x64.exe).

## Setting up a server from the package

1. Copy `worldserver.conf.dist` → `worldserver.conf` and `bnetserver.conf.dist` →
   `bnetserver.conf`.
2. In both: `SourceDirectory = "."` (the folder holding `sql\`), `MySQLExecutable = "mysql.exe"`,
   and the `*DatabaseInfo` lines set to your MySQL login (defaults `root;admin`).
3. Create the databases (`auth`, `characters`, `world`, `hotfixes`); with
   `Updates.AutoSetup = 1` the servers fill empty ones from `sql\base` and apply `sql\updates`.
4. Extract the client data with the tools (see the core's README) into `ClientData`
   (`DataDir`).
