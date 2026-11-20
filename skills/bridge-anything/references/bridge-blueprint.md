# Bridge blueprint

Read for step 5. Language is the user's choice; TypeScript with the official MCP SDK or Python with the official MCP SDK both work. Check the installed SDK's types before using its API.

## Layout

```
<system>-bridge/
  BRIDGE.md            data map, mappings, access paths, evidence
  UNKNOWNS.md          open questions and the probe for each
  DATA-PROTECTION.md   personal data touched, destinations, retention
  contract/            the schemas: one per data area, shared by REST, MCP and CLI
  src/
    source/            the only code that touches the legacy system (SQL, files, UI automation)
    service/           one function per operation; validates input, calls source, maps rows to the contract
    rest.ts            thin adapter over service
    mcp.ts             thin adapter over service
    cli.ts             thin adapter over service (optional)
  fixtures/            a snapshot of the test installation, with personal data replaced
  test/
```

Business logic lives in `service/`. `source/` is the one place that knows table names, so a vendor update changes one folder.

## The contract

- One schema per data area, named in the domain's language (`Appointment`, `Invoice`), never after the legacy table (`TBL_TERM2`).
- IDs are the system's own stable keys, as strings.
- Dates are ISO 8601 with a time zone. Money is an integer in minor units plus a currency code.
- Code tables are decoded to readable values; the raw code stays in a `raw` field for debugging.
- Every list is paginated with an opaque cursor and returns `nextCursor`.
- Changes since a point: expose `changedSince` only where `BRIDGE.md` records a reliable change marker (timestamp, row version, sequence). Otherwise say in the tool description that the client has to poll.

MCP tools: one read tool per question an agent asks (`find_patient`, `list_appointments`, `get_invoice`), with descriptions that say what is returned and what is not. Mark read tools `readOnlyHint: true` and write tools `destructiveHint` as appropriate.

## Reading

- Connect with the read-only user from step 4. The bridge refuses to start with a user that has write rights on a read-only area; check the grants at startup.
- Parameterised queries only. Timeouts on every query. Limit page size.
- Read during opening hours without locking: use the engine's snapshot or read-committed mode, never long transactions.

## Writing (only areas approved as `read_write`)

- Reproduce the app's own write path from `BRIDGE.md` exactly: same statements, same order, same transaction, same sequence and counter updates. Or use the app's import function or interface files.
- `dryRun: true` on every write: validates and returns the statements or file it would produce.
- An idempotency key on every write, stored with the result.
- A safeguard the listing can name honestly: `user_confirmation` (the MCP client asks the human before the call) or `approval_workflow` (writes queue until someone approves them in the bridge).
- An append-only audit log of writes: who, when, what, the result. No personal data in the log beyond record IDs.

## Running on site

- Package as a Windows service or a Linux systemd unit, since the system of record runs on the customer's machine or server.
- Bind REST and MCP to localhost or the practice network by default. Remote access goes through the customer's VPN or an outbound tunnel the administrator turns on.
- API keys for clients, stored hashed; scopes per data area and per read or write.
- Configuration in one file: connection, enabled data areas, write mode, bind address.
- Logs carry request IDs, durations and errors, without field values.

## Tests

- Build `fixtures/` from the test installation, replacing names, birth dates, addresses and free text with generated values.
- Contract tests per data area: the fixture rows map to valid contract objects.
- Comparison tests: the three to five records checked on screen in step 3 come out of the bridge with the same values.
- Write tests run against a disposable copy, then diff the copy against the expected change recorded in step 3.
- An MCP test: an in-process client lists the tools and calls one read tool per area.

## Version drift

Record the app version the mapping was made against. At startup, read the version (file, registry, schema table) and refuse write operations on an unknown version; reads keep working and log a warning.
