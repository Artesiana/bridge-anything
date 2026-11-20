# Find the data

Read for step 2 and step 3. Start from the installation directory, then follow what the app points to.

## Order of work

1. `inspect_desktop_application` on the installation directory. Read its `candidates`, `components` and `next_probes`. They show the runtime (.NET, native, Electron, Java), bundled database engines and config files.
2. Read the config files it lists (`*.config`, `*.ini`, `*.xml`, `*.json`, `*.udl`) for connection strings, data paths and interface folders. On Windows, also note registry keys under the vendor's name in `HKLM\SOFTWARE` and `HKLM\SOFTWARE\WOW6432Node`.
3. Open the data-access code:
   - .NET (`*.exe` or `*.dll` with CLI metadata): `inspect_managed_artifact`. Look for ORM mappings, `SqlCommand` text, table names and connection-string builders.
   - Native (Delphi, C++, VB6): `open_binary` with Ghidra, Hopper, IDA or Kuna, then search strings for `SELECT`, `INSERT INTO`, `UPDATE`, table names and file extensions.
   - Electron or Node: `analyze_javascript_application` on the app directory or ASAR.
   - Client talking to a server over HTTP: record a HAR in a browser or a proxy on the test machine, then `inspect_web_network_capture`.
4. Read the schema from the engine itself (below) and match it with the names found in code.

## Engines and what they leave on disk

| Engine | Files and signs | Read-only access | Watch the app write |
| --- | --- | --- | --- |
| Microsoft SQL Server (often Express) | service `MSSQLSERVER` or `MSSQL$<instance>`, `sqlservr.exe`, `*.mdf` / `*.ldf` | a login with only `db_datareader` on the app database; or restore a `.bak` copy | Extended Events session filtered to the app database |
| Firebird / InterBase | `*.fdb`, `*.gdb`, `firebird.conf` | a user with `SELECT` grants; copy with `gbak` | the Firebird trace API (`fbtracemgr`) |
| SAP SQL Anywhere (Sybase) | `*.db` + `*.log`, `dbsrv*.exe` or `dbeng*.exe` | a user with read grants on a copy | the engine's request logging |
| Actian Zen / Pervasive (Btrieve) | `*.btr`, `*.dat`, `*.mkd`, dictionary files `FILE.DDF`, `FIELD.DDF`, `INDEX.DDF` | ODBC through the DDF dictionary, on a copy | diff the data files before and after one action |
| Microsoft Access | `*.mdb`, `*.accdb`, lock files `*.ldb` / `*.laccdb` | open a copy read-only | diff a copy before and after |
| dBase, FoxPro, Clipper | `*.dbf`, indexes `*.ndx` / `*.cdx` / `*.ntx`, memos `*.dbt` / `*.fpt` | read a copy with any DBF reader | diff a copy before and after |
| SQLite | `*.db`, `*.sqlite`, `*.sqlite3`, with `-wal` / `-shm` | `inspect_sqlite_database` on a snapshot | two snapshots, then compare |
| PostgreSQL, MySQL, MariaDB (bundled) | `pg_hba.conf`, `postgresql.conf`, `my.ini` | a role with `SELECT` only | statement logging on the test installation |
| InterSystems Caché / IRIS | `CACHE.DAT`, `IRIS.DAT` | the vendor's SQL projection, if licensed | ask the vendor; globals are hard to diff |
| Oracle | `tnsnames.ora`, `sqlnet.ora` | a user with `SELECT` grants | unified auditing on the test installation |

Diffing before and after: on the test installation, take a snapshot, perform exactly one action in the app (create one appointment), take a second snapshot, and compare row counts and changed rows per table. One action per diff keeps the evidence clean.

## Interfaces the app may already speak

Check these before reading the database. They are often documented, supported by the vendor, and safer for writes.

- Import and export folders found in the config (watch folders, scheduled exports, print-to-file).
- German medical software: the xDT family maintained by the QMS e.V. (GDT for devices, BDT for practice data transfer, LDT for laboratory data).
- Healthcare in general: HL7 v2 and FHIR.
- German accounting: the DATEV import format.
- QuickBooks Desktop: the official QuickBooks SDK (qbXML).

## Credentials

Use a database user that the customer's administrator creates and controls. When the only working credential is one the vendor embedded or encrypted in the app, stop and ask the administrator or the vendor for access. `audit_release_exposure` reports credential markers by location only; it is for knowing that such a credential exists, not for using it.
