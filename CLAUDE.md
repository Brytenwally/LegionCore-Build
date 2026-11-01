# CLAUDE.md

Notes for Claude sessions working on LegionCore-Build. **Keep this file up to date** with
anything future sessions need, in the same commit as the change.

## What this is

A GitHub Actions workflow (`.github/workflows/build-windows.yml`) that builds
**LegionCore-Reforged 7.3.5** Windows x64 binaries from `Brytenwally/LegionCore-Reforged`,
optionally with `Brytenwally/LegionCore-Playerbots` checked out into
`modules/mod-playerbots`, linked against the **MySQL 8.0.44** client. Usage is in `README.md`.
Plan step 0 of the project (see the core's `CLAUDE.md`, which holds the project-wide notes,
rules and plan). Pattern: `Brytenwally/SkyFire-build` (read-only reference).

## Git rules

- Commit and push straight to **`main`**. No feature branches or PRs unless the owner asks.

## Workflow facts

- Triggers: `workflow_dispatch` (inputs: core repo/ref, playerbots on/off + repo/ref, tools
  on/off, MySQL version) and `v*` tags, which also publish a GitHub Release with the defaults.
- `with_playerbots` defaults to **false** and tag builds leave it off: the core has no module
  system yet (plan step 1) and the module repo is empty (checkout of an empty repo fails).
  Turn the default on once both exist.
- Dependencies follow Reforged's own `.github/workflows/build-win-x64.yml`: Boost **1.84**
  (the core needs >= 1.78; newer Boost removed APIs this old code may use), built here from
  source with the runner's MSVC (static, shared CRT, only the components
  `dep/boost/CMakeLists.txt` asks for), cached; `Boost_DIR` is passed explicitly because the
  core only hints `lib64-msvc-*/cmake` layouts. OpenSSL: newest **3.x** full installer listed
  by `slproweb/opensslhashes`, cached by version. CMake via choco.
- MySQL: the official `mysql-<ver>-winx64.zip` from cdn.mysql.com; include/, `libmysql.lib`/
  `.dll`, **`mysql.exe`** and its OpenSSL 3 DLLs are cached. The core's SQL updater
  (TrinityCore `DBUpdater`) shells out to `mysql` (`MySQLExecutable`), so it ships. MySQL
  paths are passed with forward slashes and checked in `CMakeCache.txt`.
- No `-G`: Visual Studio generator, `-A x64`, `SCRIPTS=static`; outputs land in
  `build/bin/Release` (the core's POST_BUILD copies use `$(ConfigurationName)`).
- Package = `cmake --install` output + exe/dll/conf.dist/pem from `build/bin/Release` + MySQL
  and OpenSSL DLLs + `mysql.exe` + the whole core `sql/` (~330 MB uncompressed: the world and
  hotfixes base dumps are in git) + `BUILD-INFO.txt`.
- Database setup is the core's TrinityCore updater: `SourceDirectory` (default = the
  runner's source path baked in at build time, so it must be set to `"."` in the package),
  `MySQLExecutable`, `Updates.AutoSetup`/`EnableDatabases`. `Game.Patch.ContentScripts` reads
  `<SourceDirectory>/sql/optional`.
- Changelog step: same as SkyFire-build (`commits.json` in the package, previous successful
  run's artifact `LegionCore-735-*`, GitHub compare API), never fails the build.
- The cloud container has no Windows and cannot reach mysql.com: verify changes by running
  the workflow (GitHub MCP `actions_run_trigger`) and reading the logs.
