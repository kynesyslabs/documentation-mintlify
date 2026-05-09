# Identity Provider Docs Audit (Twitter/GitHub/Discord/Telegram)

**Verdict: 2 PASS, 2 FAIL** — fictional OAuth methods in github.mdx and discord.mdx.

Source-of-truth: `/Users/tcsenpai/kynesys/sdks/src/abstraction/Identities.ts`, `/Users/tcsenpai/kynesys/sdks/src/types/abstraction/index.ts`.

## Per-file verdicts

| File | Verdict | Notes |
|------|---------|-------|
| `sdk/web2/identities/twitter.mdx` | ✓ PASS | Methods + signatures correct |
| `sdk/web2/identities/github.mdx` | ✗ FAIL | References non-existent `addGithubIdentityFromOAuth()` (line 72); OAuth section (lines 16–114) is fictional |
| `sdk/web2/identities/discord.mdx` | ✗ FAIL | References non-existent `addDiscordIdentityFromOAuth()` (line 72); OAuth section (lines 16–114) is fictional |
| `sdk/web2/identities/telegram.mdx` | ✓ PASS | `addTelegramIdentity(demos, payload)` matches; `TelegramSignedAttestation` payload fields (`username`, `telegram_user_id`) verified in types/abstraction/index.ts:128–131 |

## Method drift table

| Doc references | SDK actual (Identities.ts) | Status |
|----------------|---------------------------|--------|
| `addGithubIdentityFromOAuth()` | NOT FOUND | ✗ FAIL — fictional |
| `addGithubIdentity(demos, proof)` | line 243 | ✓ PASS |
| `addTwitterIdentity(demos, proof)` | line 336 | ✓ PASS |
| `addDiscordIdentityFromOAuth()` | NOT FOUND | ✗ FAIL — fictional |
| `addDiscordIdentity(demos, proof)` | line 371 | ✓ PASS |
| `addTelegramIdentity(demos, payload)` | line 411 | ✓ PASS |

## Cross-cutting

- All four files use the canonical `(demos, ...args)` calling convention. ✓
- All four import from `@kynesyslabs/demosdk/abstraction`. ✓
- No references to deprecated global `DemosWebAuth`. ✓

## Recommended actions

For github.mdx and discord.mdx: either remove the OAuth sections (lines 16–114 in each) entirely, or leave them gated behind a `<Warning>` saying the OAuth flow is not implemented and only the proof-based flow works. The non-OAuth flows in both files (gist-based for GitHub, message-based for Discord) are correct as written.

If the OAuth flow IS desired and just unimplemented in the SDK, file an SDK issue — don't paper over it in docs.
