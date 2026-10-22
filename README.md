# Simien permissions

The permissions [Simien](https://github.com/simienapp/runtime) grants to
well-known packages by default, as one file: [`permissions.json`](permissions.json).
The rules behind it are [APS 3](https://github.com/simienapp/aps/issues/3).

Everything Simien runs (install scripts, `s x`) runs in an OS sandbox. With
no grants, a program:

- can read everything except secrets (`~/.ssh`, cloud credentials, other
  tools' tokens);
- can write only its own package folder (install scripts) or the project
  (`s x`), plus a temporary folder;
- has no network during install, and network for `s x`;
- doesn't see environment variables that look like credentials.

A package can declare what more it needs in its own `package.json`
(`"permissions"`); the user approves that. This list is the other way in:
grants Simien approves on everyone's behalf, so it is kept **small**, and
every entry says why it is there.

## The format

```json
"puppeteer": {
  "reason": "postinstall downloads Chrome for Testing into the user's cache",
  "install": {
    "net": ["storage.googleapis.com"],
    "write": ["$CACHE/puppeteer"],
    "env": ["PUPPETEER_CACHE_DIR"]
  }
}
```

- `install` applies to the package's install scripts, `x` to `s x <package>`.
- `dangerouslyUseNode: true` gives the package's programs Node's APIs
  ([APS 2](https://github.com/simienapp/aps/issues/2)): Simien runs them
  itself, still in the sandbox and under every other rule here. Only
  packages whose programs are Node programs people need belong here.
- `net`: hosts (optionally `host:port`). Until Simien filters by host, any
  `net` grant turns the network on; the hosts are recorded so the grant
  narrows automatically when it does.
- `write`, `read`: paths under `$PACKAGE`, `$PROJECT`, `$CACHE` or `$TEMP`.
  `$HOME` paths are not accepted here: this list can never grant secrets.
- `env`: variable names to keep in the environment.

[`schema.json`](schema.json) is checked in CI.

## Changing it

Open a pull request: people and agents both welcome. Say what the package
does that needs the grant, with a link to the code that does it. Keep the
grant as narrow as the code needs.

Simien pins a commit of this repository; a change reaches users when the
runtime bumps the pin.
