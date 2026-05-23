# pi-codex-search

[![npm](https://img.shields.io/npm/v/pi-codex-search)](https://www.npmjs.com/package/pi-codex-search)
[![license](https://img.shields.io/npm/l/pi-codex-search)](./LICENSE)

**Give Pi a `web_search` tool through your Codex subscription.**

Pi is the harness. Codex already has a ChatGPT-backed search path. This package connects the two: it adds a normal Pi tool that searches the web through the `openai-codex` account you already use in Pi.

No `ACCESS_TOKEN` env var. No separate login flow. If Pi can use your Codex subscription, this extension can use the same auth.

## Why this exists

Pi keeps the core small on purpose. Search does not have to be baked into the harness; it can be a tool.

This extension is for the cases where your Pi workflow needs fresh or source-backed information:

- **Current docs and release notes.** Ask the model to look up something that changed after its training cutoff.
- **Source-backed answers.** The tool returns citations alongside the text.
- **Codex account reuse.** It uses Pi's existing `openai-codex` OAuth credential instead of asking you to paste tokens.
- **Custom Pi workflows.** Any model in Pi can see the tool once the Codex auth is available at session start.

## What this package adds

- **A `web_search` tool** — query the web from inside Pi.
- **Codex auth reuse** — reads the same `openai-codex` credential that Pi stores after `/login openai-codex`.
- **No manual token handling** — the extension does not read `ACCESS_TOKEN` during normal Pi usage.
- **Model selection that follows Pi** — use an explicit env override, the active Codex model, or the default model from Codex's model list.
- **Streaming updates** — partial answer text is sent back while the search response streams.
- **Structured details** — the final tool result includes citations, search calls, response id, model, and usage.

## Install

From npm:

```bash
pi install npm:pi-codex-search
```

Or load a local checkout without installing:

```bash
pi -e /path/to/pi-codex-search
```

### Install from GitHub Release tarball

If you prefer not to use npm, download the tarball from the [latest release](https://github.com/Leechael/pi-codex-search/releases/latest), extract it, and install from the local path:

```bash
curl -L https://github.com/Leechael/pi-codex-search/releases/latest/download/pi-codex-search.tar.gz | tar -xz -C /tmp
pi install /tmp/pi-codex-search
```

## Sign in

Inside Pi, run:

```text
/login openai-codex
```

Choose `ChatGPT Plus/Pro (Codex Subscription)` if Pi asks which provider to use. Pi stores and refreshes the credential.

At `session_start`, this extension asks Pi for the `openai-codex` token. If no token is available, or if the ChatGPT account id cannot be found from the stored OAuth credential or decoded access token, it does not register `web_search`. In that case the model will not see the tool.

## Tool

The extension registers one tool:

```json
{
  "name": "web_search",
  "arguments": {
    "query": "latest OpenAI Codex release notes",
    "search_context_size": "medium",
    "live": true
  }
}
```

Arguments:

- `query` — required search question.
- `search_context_size` — optional, one of `low`, `medium`, `high`; defaults to `medium`.
- `live` — optional boolean; defaults to live web access.

The tool returns text plus a `Sources:` section when citations are available. The structured `details` object includes:

- `model`
- `responseId`
- `searchCalls`
- `citations`
- `usage`

## Model used for search

The search request needs a Codex model id. The extension chooses it in this order:

1. `PI_CODEX_WEB_SEARCH_MODEL`, if set.
2. The active Pi model, if it comes from the `openai-codex` provider.
3. The default model from Codex's `/codex/models` response.

Most users do not need to set anything.

## Common knobs

Most users only need `/login openai-codex`. These env vars are here for debugging or custom setups:

- `PI_CODEX_WEB_SEARCH_MODEL` — force the Codex model used by the tool.
- `PI_CODEX_WEB_SEARCH_CONTEXT_SIZE` — default search context size: `low`, `medium`, or `high`.
- `PI_CODEX_WEB_SEARCH_LIVE` — set to `false` to use cached web access.
- `PI_CODEX_WEB_SEARCH_BASE_URL` — override the Codex backend base URL.
- `PI_CODEX_WEB_SEARCH_CLIENT_VERSION` — override the `client_version` sent to `/codex/models`.

## Notes

### Codex search vs model search

This does not add browsing to the model provider itself. It adds a Pi tool. The model decides when to call `web_search`, just like any other tool.

### Account id

Codex requests need both the access token and the ChatGPT account id. The extension first checks Pi's stored OAuth credential. If that does not include an account id, it tries to extract one from the access token.

### Headers

The extension builds the Codex request headers itself, including `Authorization`, `chatgpt-account-id`, `originator: pi`, `OpenAI-Beta: responses=experimental`, `accept`, and `content-type`.

## Troubleshooting

### The model does not see `web_search`

The tool is only registered when Codex auth is available at session start. Run:

```text
/login openai-codex
```

Then start a new Pi session. If Pi asks for a provider, choose `ChatGPT Plus/Pro (Codex Subscription)`.

### `web_search` says the account id was not found

The stored OAuth credential did not include an account id, and the extension could not decode one from the access token. Re-run `/login openai-codex` so Pi refreshes the credential.

### Search uses the wrong model

Set `PI_CODEX_WEB_SEARCH_MODEL` to the Codex model id you want. If unset, the extension uses the active Codex model when possible, then falls back to the default model from `/codex/models`.

## Development

```bash
npm install
npm run check
npm test
npm run lint
npm run format:check
```

## Release

This repository follows the same release shape as `pi-provider-kimi-code`:

- `release-naming.env` defines `PKG_NAME=pi-codex-search` and `TAG_PREFIX=v`.
- `scripts/next-version.sh` computes the next semantic version from tags.
- `.github/workflows/release-command.yml` creates release commits and tags.
- `.github/workflows/release-on-tag.yml` publishes to npm with provenance and attaches `pi-codex-search.tar.gz` to the GitHub release.

## References

- Pi: [earendil-works/pi](https://github.com/earendil-works/pi)

## License

MIT
