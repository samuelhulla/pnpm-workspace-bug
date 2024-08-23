## This repository reproduces a production bug with pnpm workspaces

This repository reproduces a bug with pnpm workspace resolution of `bin` exports for packages with `@org/` in their name *(org can obviously be any other wild-card text,
under the standard npm organization notation)*

## Project structure

- 📁 packages/pkg - A simple bin export with { "name": "pkg" } in package.json
- 📁 packages/@org - Same bin export with { "name": "@org/pkg" } in package.json
- 📁 packages/debug - A debug package which has
```json
{
  "scripts": {
     "debug": "pnpm exec pkg && pnpm exec @org/pkg"
  },
  "devDependencies": {
     "pkg": "workspace:*",
     "@org/pkg": "workspace:*"
  }
}
```

## What happens

The first half of the debug command executes as expected:

> ```
> > debug@ pkg /Users/samuel/Coding/pnpm-workspace-bug/packages/debug
> pnpm exec pkg
> resolved { "pkg": "workspace:* } ✅
> ```

However the second half with the `pnpm exec @org/pkg` fails with:

> ```
> > pnpm exec @org/pkg
> ERR_PNPM_RECURSIVE_EXEC_FIRST_FAIL  Command "@org/pkg" not found
> ```

## Cause of the bug

I've intentionally uploaded `node_modules` to this github repository as well to see the difference between how the installation of these org binaries happen.

- The `pkg` workspace reference is registered under special `node_modules/.bin/pkg` file,
- meanwhile the `@org/pkg` workspace reference is under `node_modules/@org/pkg` symlink.

The `pnpm exec` command *(least to my knowledge)* scaffolds only the top-level `.bin` node_modules folder and ignores any `bin` exports under the `@org` node modules.

## Expected behaviour

- [X] `pnpm exec pkg` should work 
- [ ] `pnpm exec @org/pkg` should work
