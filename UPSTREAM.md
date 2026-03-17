# Upstream Tracking

Track upstream changes to decide if our fork needs updating.

## Sources

| Source | Location | What to check |
|--------|----------|---------------|
| **Plugin** | [mem0ai/mem0/openclaw/](https://github.com/mem0ai/mem0/tree/main/openclaw) | New commits to `openclaw/` directory |
| **SDK** | [mem0ai on npm](https://www.npmjs.com/package/mem0ai) | New versions (we vendor v2.2.2) |
| **Issues** | [mem0ai/mem0 issues](https://github.com/mem0ai/mem0/issues?q=openclaw) | New issues mentioning openclaw |

## How to Check

```bash
# 1. Plugin commits since last check
gh api "repos/mem0ai/mem0/commits?path=openclaw&per_page=10" \
  --jq '.[] | "\(.sha[:7]) \(.commit.author.date[:10]) \(.commit.message | split("\n")[0])"'

# 2. Latest mem0ai npm version
npm view mem0ai version

# 3. Open issues mentioning openclaw
gh search issues "openclaw" --repo mem0ai/mem0 --state open --json number,title

# 4. Compare upstream index.ts against ours
gh api repos/mem0ai/mem0/contents/openclaw/index.ts --jq '.content' | base64 -d > /tmp/upstream-index.ts
diff /tmp/upstream-index.ts index.ts
```

## Decision Framework

When upstream changes, ask:

1. **Does the change fix a bug we already fixed?** Skip it — our fix is already in the vendored file or index.ts.
2. **Does the change add a new feature?** Evaluate if we want it. Port to our index.ts if yes.
3. **Does the SDK version bump?** Check the changelog. If it fixes bugs in `dist/oss/index.mjs`, re-vendor with our patches.
4. **Does the change conflict with our patches?** If upstream fixes the same code we patched (baseURL, auto-recall, etc.), we can drop our fix and use theirs.

### Re-vendoring the SDK

When `mem0ai` releases a new version:

```bash
# 1. Install fresh version
mkdir /tmp/mem0-update && cd /tmp/mem0-update
npm install mem0ai@<new-version>

# 2. Run the patch script (update /tmp/patch-mem0-oss.mjs if needed)
node /tmp/patch-mem0-oss.mjs

# 3. Diff against current vendored file
diff vendor/mem0-oss.mjs /tmp/mem0-oss-vendored.mjs

# 4. If clean, copy it in
cp /tmp/mem0-oss-vendored.mjs vendor/mem0-oss.mjs
```

## Check Log

| Date | Checked by | Plugin commit | SDK version | Issues | Action |
|------|-----------|---------------|-------------|--------|--------|
| 2026-02-15 | Arc | `3d3e875` (2026-02-02, initial) | 2.2.2 | #4037, #4050, #4056, #3998 | No action — all issues already fixed in our fork |

## Issues We Track

| Issue | Title | Status | Our status |
|-------|-------|--------|------------|
| [#4037](https://github.com/mem0ai/mem0/issues/4037) | Auto-recall silently broken (wrong property name) | Open | Fixed in our index.ts (prependContext) |
| [#4040](https://github.com/mem0ai/mem0/issues/4040) | OpenAI Embedder ignores baseURL | Open | Fixed in vendored mem0-oss.mjs |
| [#4048](https://github.com/mem0ai/mem0/issues/4048) | ESM build missing baseURL passthrough | Open | Fixed in vendored mem0-oss.mjs |
| [#4050](https://github.com/mem0ai/mem0/issues/4050) | ARM64 sqlite3 binding error on Node v22 | Open | N/A (we run x86) |
| [#4056](https://github.com/mem0ai/mem0/issues/4056) | Qdrant local error | Open | Fixed (checkCompatibility disabled) |
| [#3998](https://github.com/mem0ai/mem0/issues/3998) | Per-agent memory isolation (feature request) | Open | Not implemented |
