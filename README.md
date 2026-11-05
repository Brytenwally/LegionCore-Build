# LegionCore-Build

GitHub Actions workflow that builds **LegionCore-Reforged 7.3.5 (build 26972)** Windows x64
binaries from `Brytenwally/LegionCore-Reforged`, linked against the **MySQL 8.0.44** client.
The playerbots module (`Brytenwally/LegionCore-Playerbots`) is checked out into the core's
`modules/mod-playerbots` and compiled in (input `with_playerbots`, on by default).

## What the build uses

| Part | Repository | Branch / version | Where it goes |
| --- | --- | --- | --- |
| Core | [Brytenwally/LegionCore-Reforged](https://github.com/Brytenwally/LegionCore-Reforged) | `main` (input `core_ref`) | the source tree |
| Playerbots module | [Brytenwally/LegionCore-Playerbots](https://github.com/Brytenwally/LegionCore-Playerbots) | `main` (input `playerbots_ref`) | `modules/mod-playerbots` |
| Example module | in the core (`modules/mod-example`) | the core's branch | built always |
| Build workflow | [Brytenwally/LegionCore-Build](https://github.com/Brytenwally/LegionCore-Build) (this repo) | `main` | `.github/workflows/build-windows.yml` |
| Boost | [boost.org](https://www.boost.org/) source, [archives.boost.io](https://archives.boost.io/release/1.84.0/source/) | 1.84.0 (built here with MSVC, cached) | linked in |
| OpenSSL | [slproweb Win64 OpenSSL](https://slproweb.com/products/Win32OpenSSL.html) ([hashes](https://github.com/slproweb/opensslhashes)) | newest 3.x | linked in, DLLs packaged |
| MySQL client | [MySQL Community Server zip](https://dev.mysql.com/downloads/mysql/) ([cdn.mysql.com](https://cdn.mysql.com/)) | 8.0.44 (input `mysql_version`) | `libmysql`, `mysql.exe` packaged |
| Compiler | GitHub's [`windows-2022`](https://github.com/actions/runner-images/blob/main/images/windows/Windows2022-Readme.md) runner | Visual Studio 2022 (v143), CMake | |

Each package's `BUILD-INFO.txt` records the exact commits and versions used.

## Running a build

- **Actions tab → Build Windows binaries → Run workflow.** Inputs: core repo/ref, playerbots
  on/off and repo/ref, extractors on/off, MySQL client version.
- **Push a tag** like `v1.0.0`: builds with the defaults and publishes the zip as a GitHub
  Release.

The result is a workflow artifact named
`LegionCore-735-win64-mysql<ver>-<core sha>[-playerbots-<sha>]`.

## What is in the package

- `worldserver.exe`, `bnetserver.exe` (+ `bnetserver.cert.pem` / `.key.pem`),
  `connection_patcher.exe`, the `.conf.dist` files (incl. `playerbots.conf.dist` and
  `example.conf.dist`; copy to `.conf` to change a module setting), and with tools on: `mapextractor`, `vmap4extractor`, `vmap4assembler`,
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
3. Create the databases `auth`, `characters`, **`world_legion`** and **`hotfixes_legion`**, and
   use those names in `WorldDatabaseInfo` / `HotfixDatabaseInfo`: Reforged's SQL updates
   refer to `world_legion` and `hotfixes_legion` by name, so `world` / `hotfixes` fail with
   "Unknown database 'hotfixes_legion'"; with
   `Updates.AutoSetup = 1` the servers fill empty ones from `sql\base` and apply `sql\updates`.
4. Extract the client data with the tools (see the core's README) into `ClientData`
   (`DataDir`).
