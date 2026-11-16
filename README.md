# ats-strike-contract

The wire format an ats-strike server and its clients agree on, and nothing
else: the `cv_contracts` package (pydantic models, the response envelope, the
protocol number, the free JD pre-check) and [the protocol](PROTOCOL.md) it
describes.

It knows no pipeline — no prompt names, no model roles, no step names. That is
what lets one client talk to a 0.5, a 1.x and a 2.0 server alike, and why every
release line of the server can share this one repository.

## How it is used

[ats-strike](https://github.com/gcontini/ats-strike) (client and 1.x server)
and its `fix-0.5.x` line both carry this repository as a git submodule at
`contracts/`, where it is a member of their uv workspace. Each of them pins
the commit it was tested with.

```bash
git clone --recurse-submodules https://github.com/gcontini/ats-strike
# or, in a clone made without it:
git submodule update --init
```

To change the contract: edit it under `contracts/`, run that repository's
tests (they drive the real client against the real server), commit here and
push, then commit the new submodule pointer there.

## Rules

- **pydantic and nothing else.** These models are imported by a FastAPI
  server and by a PyInstaller-frozen CLI, and must stay cheap in both.
- **`Field(description=...)` is the wire's documentation.** It ends up in the
  server's OpenAPI page.
- **A new location, type or event is additive.** A client skips one it does
  not know; anything else that changes the wire bumps `PROTOCOL`, and a client
  refuses a server that reports another number.

## Tests

```bash
uv run --no-project --with pydantic --with pytest --with-editable . pytest
```

They need no network, no API key and no server.
