# MCP requestState Token Designer

Design the binding set that goes inside an MCP `requestState` token, mint one with real WebCrypto HMAC-SHA-256 in the browser, then read the residual table: which attack each binding stops, and the one attack that none of them stop.

**Live demo:** https://0xelitesystem.github.io/mcp-state-token-designer/

`webhook-signature-verifier` checks a signature somebody else produced, against a published vendor scheme. This tool does the opposite job. It designs the binding set, mints the token in front of you, and computes what each choice does and does not prevent. Its entire point is a case where every signature check is correct and a correct signature is still not the answer.

## Use

1. Open the page. A 32 byte session key comes from `crypto.getRandomValues` and is imported non-extractable through `crypto.subtle.importKey`.
2. Leave the four bindings on: HMAC integrity, authenticated principal, short expiry, request digest. Edit the principal, method, params and TTL if you want your own request.
3. Read the **residual table**. It is the primary output. Every row is computed live by minting and verifying real tokens with the current binding set.
4. Press **Replay 5 times**. The identical token, same principal, same request, inside the TTL, verifies five times out of five. Nothing is wrong with the signature. Nothing is wrong with the checks. The coupon is redeemed five times.
5. Turn one binding off and watch exactly one row of the table flip. That is the point of the toggles: the page teaches which binding stops which attack instead of handing you a recipe.
6. Turn on the **server-side consumption ledger** and replay again. Attempt 1 passes, attempts 2 to 5 are rejected. That control is not in the token, and it cannot be.

The table with all four bindings on:

| Attack | Result |
| --- | --- |
| cross-user, same token presented by a different principal | PREVENTED |
| cross-request, same method, different params | PREVENTED |
| cross-method, same params, different method | PREVENTED |
| tampered body, principal rewritten, signature kept | PREVENTED |
| expired replay, presented after the TTL lapses | PREVENTED |
| same principal, same request, inside the TTL | NOT PREVENTED |

## Why this exists

Multi Round-Trip Requests are new in the 2026-07-28 version of the Model Context Protocol specification. When a server needs more input it returns an `InputRequiredResult` carrying an opaque `requestState`, the client echoes that value back on the retry, and the server reconstructs its context without keeping any state of its own. That design is what lets a server answer a retry on a different instance with no shared storage and no sticky load balancing. It also means an attacker-controlled string is now the input to your authorization decision.

The specification tells you what to put inside it. Integrity protection is its own requirement, a MUST: protect the state with HMAC or AEAD and reject anything that fails verification. Three further bindings sit under a single SHOULD: the authenticated principal, a short expiry, and an identifier for the originating request. Implement all four, watch every check pass, and it is very easy to write "replay protection" in the design doc and move on.

The specification also says, one sentence later and inside a warning callout, that those measures do not guarantee single-use, and that a server for which a state must be consumed at most once MUST enforce that invariant server-side.

Be clear about what this repo is. It is a faithful reading of normative text made executable, not a discovery. The trap is stated by the specification itself. The value here is that everyone skims one sentence in a callout, and almost nobody skims five PASS results in a row on a coupon they just minted. Pressing the button is more persuasive than reading the sentence, and the page mints the token in front of you rather than asserting properties about one.

The practical line: a retry that only needs the elicited value back is safe to replay. A retry that redeems, provisions, charges, or burns a quota is not, and no binding you can put inside the token can tell those two apart for you.

## Privacy

Everything runs in your browser. There is no network request of any kind: no analytics, no fonts, no frameworks, no error reporting, nothing fetched at runtime. The page is one file with the CSS and the script inline.

The session key never leaves the page and is never persisted. It comes fresh each session from `crypto.getRandomValues`, imported as non-extractable, and dies with the tab. Principals, methods, params and every token you mint stay in memory only. The single thing written to `localStorage` is your light or dark theme choice.

## Run locally

    git clone https://github.com/0xelitesystem/mcp-state-token-designer.git
    cd mcp-state-token-designer

Open `index.html` in a browser. It works from `file://`, because there is nothing to fetch.

If you prefer a server:

    python -m http.server 8000

Then open http://localhost:8000/ .

## Build

There is no build. One file, `index.html`, with inline CSS and inline JavaScript, no dependencies and no toolchain. Edit it and reload.

## Third-party notices

- **Project:** Model Context Protocol specification, Multi Round-Trip Requests page, version 2026-07-28.
  **Copyright holder,** copied verbatim from that project's `LICENSE` file: `Copyright (c) 2024-2025 Model Context Protocol a Series of LF Projects, LLC.`
  **Licence:** that same `LICENSE` file states that the project is moving from the MIT License to the Apache License, Version 2.0, that "All new code and specification contributions to the project are licensed under Apache-2.0", and that contributions whose authors have not granted relicensing consent remain under the MIT License.
  **URL:** https://github.com/modelcontextprotocol/modelcontextprotocol/blob/main/LICENSE and https://modelcontextprotocol.io/specification/2026-07-28/basic/patterns/mrtr
  **What was taken:** five short passages of normative text, quoted verbatim and attributed, in the "What the specification actually says" section of `index.html`, plus, in the AEAD section, two short inline quotations and the four-word placeholder value the specification uses in its own example. No code from that project is reproduced here. The token construction is an independent implementation of the stated requirements.

This project is independent and unaffiliated. It is not associated with, endorsed by, or connected to the Model Context Protocol project or its maintainers.

## Related

- https://github.com/0xelitesystem/multi-provider-webhook-verifier
- https://github.com/0xelitesystem/mcp-server-privilege-inventory
- https://github.com/0xelitesystem/jwt-inspector

## License

MIT. Copyright (c) 2026 RBL Industries LLC.
