# MCP Server Docs Audit

3 FAIL, 4 PASS.

| File | Verdict | Notes |
|------|---------|-------|
| `overview.mdx` | ✓ PASS | Dual transport (stdio + SSE) accurate |
| `configuration.mdx` | ✗ FAIL | Lines 10–13 conflate `MCP_ENABLED` + `MCP_SERVER_PORT` with `RPC_MCP_PORT`. Real env mapping: only `MCP_ENABLED` and `MCP_SERVER_PORT` exist in `config/envKeys.ts`. `RPC_MCP_PORT` is loaded into config but NOT used to initialize MCP port. |
| `architecture.mdx` | ✓ PASS | `transport: "sse"` init from `index.ts:626` matches |
| `monitoring-endpoint.mdx` | ✗ FAIL | Lines 10–17 claim `/mcp` endpoint exists returning `{enabled, transport, status}`. **No such endpoint in source** — no route registered, no handler. Pure fiction. |
| `available-tools.mdx` | ✓ PASS | All 8 tools match `demosTools.ts`: `get_network_status`, `get_node_identity`, `get_last_block`, `get_block_by_number`, `get_chain_height`, `get_peer_list`, `get_peer_count` |
| `client-usage.mdx` | ✗ FAIL | Lines 12–19: malformed markdown — double opening code fence (lines 13 and 14 both start with ` ```typescript `). Breaks rendering. Port 3001 itself correct. |
| `troubleshooting.mdx` | ✓ PASS | Port + debug advice sound |

## Top 3 issues

1. **Fictional `/mcp` monitoring endpoint** (monitoring-endpoint.mdx:10–17). Replace or remove.
2. **Env var mismatch** (configuration.mdx:10–13). Doc treats `RPC_MCP_PORT` as alternative to `MCP_SERVER_PORT`. Reality: only `MCP_SERVER_PORT` is used at init. `RPC_MCP_PORT` is dead config.
3. **Markdown syntax error** (client-usage.mdx:12–19). Double code fence.

## Port verification

✓ `.env.example` line 57: `RPC_MCP_PORT=3001` (matches default).
✓ `config/loader.ts`: `mcpServerPort: envInt(EnvKey.MCP_SERVER_PORT, d.server.mcpServerPort)`.

The two env vars conflict: `RPC_MCP_PORT` is set in `.env.example` but `MCP_SERVER_PORT` is what loader.ts reads. This is a node-side bug or naming inconsistency — flag for node team.
