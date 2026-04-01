# claude-code-sourcemap

This repository is a restored source tree inferred from a published `cli.js.map`.
It is useful for source reading and architecture study, but it is not yet a drop-in
replacement for the official Claude Code build.

## Current status

- The previous README said `bun run index.ts`, but the repository did not contain
  a top-level `index.ts`.
- The closest recovered CLI bootstrap is [`src/entrypoints/cli.tsx`](src/entrypoints/cli.tsx).
- Dependency installation currently stops on unreleased internal packages and
  native bindings that are not published to npm.

The unresolved package set reported in issue `#1` is:

```text
@ant/claude-for-chrome-mcp@workspace:*
@ant/computer-use-mcp@workspace:*
@ant/computer-use-swift@workspace:*
audio-capture-napi@*
image-processor-napi@*
url-handler-napi@*
```

## What you can do today

### Read the reconstructed CLI entrypoint

```bash
rg -n "async function main" src/entrypoints/cli.tsx
```

### Inspect the recovered command registry

```bash
sed -n '1,220p' src/commands.ts
```

### Use the top-level bootstrap alias

`index.ts` now forwards to the recovered CLI entrypoint:

```bash
bun run index.ts --version
```

That command only becomes runnable after the missing private packages are replaced
or stubbed locally.

## Why install currently fails

This repository was reconstructed from source maps, not restored from the original
monorepo with its private workspaces and native modules. As a result:

- `bun install` cannot resolve the unpublished `@ant/*` workspace dependencies.
- The `*-napi` packages are also unavailable from the public registry.
- Build-time macros and the original packaging pipeline are not fully recreated
  here, so successful dependency resolution alone would not guarantee a complete
  production-equivalent CLI.

## Practical next steps

If the goal is study rather than full reproduction, start with:

1. `src/entrypoints/cli.tsx`
2. `src/main.tsx`
3. `src/setup.ts`
4. `src/commands.ts`

If the goal is to make the tree runnable, the next real milestone is not another
README tweak. It is replacing or mocking the six unresolved packages above and
then validating the recovered bootstrap path against Bun.
