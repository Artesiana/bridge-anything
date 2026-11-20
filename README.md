<img src="assets/hero.png" alt="Bridge Anything: software without an API, reverse engineered with REA into a bridge with REST, MCP and CLI that your app talks to" width="100%" />

# Bridge Anything

[![skills.sh](https://skills.sh/b/artesiana/bridge-anything)](https://skills.sh/artesiana/bridge-anything)
[![MIT license](https://img.shields.io/badge/license-MIT-f4c430?style=flat-square)](LICENSE)

Your app needs data from software that has no API. This skill builds the bridge.

Doctors' practice software, dealer systems, tax and accounting desktop apps, old ERPs: a lot of the world's data sits in programs that run on a server in the back office and offer no API, or one you only get through a partner programme. Every app, automation and agent that wants that data stops there.

**Bridge Anything** gives your coding agent (Claude Code, Codex, Cursor and others) a method for getting through. It uses [REA (Reverse Engineer Anything)](https://github.com/morluto/rea) to find where the software keeps its data and how it reads and writes it, then builds a small **bridge** that runs next to it and exposes that data as REST, MCP and a CLI. Then it wires your app to the bridge.

Every table, column and write path the bridge relies on is backed by evidence REA found. Nothing is guessed.

## Install

```bash
npx rea-agents setup                          # connect REA to your agent
npx skills add artesiana/bridge-anything      # add this skill
```

Restart your agent afterwards. The skill tells the agent how to work; REA gives it the tools to look inside the software.

## Ask your agent

```text
Use bridge-anything on the test installation in C:\Praxis\TestInstall.
My booking app needs patients and appointments from it, and later has to
create appointments. I hold the licence for this installation.
```

Replace the path, the data and the app with your own.

## How it works

Seven steps. Each one ends on a check the agent has to pass before it moves on.

1. **Establish the right to do this.** Whose installation, who agreed, what the vendor already offers. The bridge uses access a licensed user or administrator already has.
2. **Find where the data lives.** REA inspects the installation: runtime, database engine, connection config, export folders. The skill knows what SQL Server, Firebird, SQL Anywhere, Btrieve, Access, dBase, SQLite and others leave on disk.
3. **Map each data area.** Tables, keys, deletion, change markers, code tables, and how the software writes, read from its data-access code with REA and confirmed by diffing the database around one action. Results are checked against what the software shows on screen.
4. **Choose the access path.** Vendor API first, then interface files, then a read-only database user, then UI automation. Writes only through a path the software itself uses.
5. **Build the bridge.** Contract first, one source layer, thin REST, MCP and CLI adapters, write safeguards, tests against an anonymised snapshot, packaged to run on site.
6. **Connect your app.** A typed client from the contract, or the bridge's MCP server registered in your agent. Your app talks to the bridge, never to the legacy database, so a vendor update only touches the bridge.
7. **Share it (optional).** Keep it inside your product, open-source it, or list it where others look for it.

The agent keeps three files as it goes: `BRIDGE.md` (the map and the evidence), `UNKNOWNS.md` (open questions and how to answer them) and `DATA-PROTECTION.md` (what personal data the bridge touches and where it goes).

## Why use it?

Agents are good at writing a bridge once they know the data model. They are bad at knowing when they have guessed it. Without a method they read three table names, invent the rest, and write into a production database the way they think the software does.

This skill makes the agent prove each mapping, treat every write as dangerous until it has watched the software do it, and stop when the only way in is past an access control.

## Share your bridge

A bridge you built for one customer usually works for every other installation of the same software. If you want others to find it, you can list it on [OpenCoupler](https://opencoupler.com), a marketplace for integrations into software without an open API, where people also vote for the systems they need. Step 7 does this through OpenCoupler's MCP server when you ask for it.

## Responsible use

Use this skill on installations you are licensed for or have written permission to analyse, for interoperability with your own software. It does not help circumvent licence checks, DRM, encrypted credentials or other access controls; when that is the only way in, ask the vendor or the customer's administrator. Analyse a copy, never the live system. Personal data in fixtures, logs and screenshots gets replaced before it leaves the test machine. See [legal and data protection](skills/bridge-anything/references/legal-and-data-protection.md) and REA's own [disclaimer](https://github.com/morluto/rea/blob/main/DISCLAIMER.md).

## Credits

Built on [REA](https://github.com/morluto/rea) by morluto (MIT). Independent project, not affiliated with REA. Made by [Artesiana](https://artesiana.de).

## License

[MIT](LICENSE)
