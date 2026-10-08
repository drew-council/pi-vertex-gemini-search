# @bizmyth/pi-vertex-gemini-search

Web search and research tools for the [Pi coding agent](https://pi.dev/), backed by Vertex AI Gemini with Google Search grounding through Pi's built-in `google-vertex` provider.

Forked from [`@nilskluewer/pi-vertex-gemini-search`](https://github.com/nilskluewer/pi-vertex-gemini-search). This fork drops its own token handling, endpoint, and environment configuration and uses whatever Vertex AI setup Pi's chat models use.

## Install

```sh
pi install npm:@bizmyth/pi-vertex-gemini-search
```

Restart Pi after installation. Review extensions before installing them; Pi extensions run with your full user permissions.

## Tools

- **`web_search`**: quick fact-check or verification. Returns a concise answer and source URLs.
- **`web_research`**: in-depth research on complex topics. Returns a structured answer and source URLs.

Both tools:

- Ground answers with Google Search and read URLs passed in the query via Gemini `url_context`.
- Resolve `vertexaisearch.cloud.google.com` redirect links to their destination URLs.
- Return the answer and source URLs to the model, and structured results (`answer`, `sources`, `searchQueries`, `model`, `location`, `serviceTier`, `costUsd`) to codemode scripts.
- Stream the answer into the transcript while Gemini writes it, then show a model/cost/pricing-tier summary with a collapsed preview; expand the result for the full answer and sources.
- Report their token and grounding cost as tool usage, so searches count toward the session cost.

## Vertex AI setup

The tools need Pi's `google-vertex` provider to be configured, exactly as for Vertex AI chat models. See [Google Vertex AI](https://github.com/earendil-works/pi/blob/main/packages/coding-agent/docs/providers.md#google-vertex-ai) in Pi's provider docs. For example, with Application Default Credentials:

```sh
export GOOGLE_CLOUD_PROJECT=your-project
export GOOGLE_CLOUD_LOCATION=global
gcloud auth application-default login
```

A Vertex AI API key (`GOOGLE_CLOUD_API_KEY`) or a credential stored with `/login google-vertex` works too. Searches run in that project and location, and fail with Pi's auth error when Vertex AI is not configured.

The current Gemini models are served from the `global` location.

## Configuration

- `PI_VERTEX_SEARCH_MODEL`: the `google-vertex` model to search with. Default: `gemini-3.8-flash`. It must be in Pi's model catalog (`pi --list-models google-vertex`).

Pi prices tokens from its model catalog. Google Search grounding is estimated at $14 per 1,000 queries.

## Pricing tier

```text
/search-pricing            # show the current preference
/search-pricing standard   # default
/search-pricing flex       # Flex PayGo, falling back to standard
```

Standard is the default because Flex requests can queue until they time out, and grounding dominates the cost of a search. When a Flex request fails or times out, the search is retried at standard, and later searches stay on standard until `/search-pricing flex` is run again. The preference does not persist across Pi sessions.

## Development

```sh
bun install
bun run check
```

`bun run check` runs Biome, TypeScript 7 (`tsc`, the Go-based compiler), and Bun tests.

## Releasing

Trusted publishing must already be configured for `publish.yml`. To release a new version:

```sh
bun pm version patch # or: minor / major
git push origin main --follow-tags
gh release create "$(git describe --tags --exact-match)" --generate-notes
```

Publishing the GitHub release runs the checks and publishes the matching package version to npm using OIDC. No long-lived `NPM_TOKEN` is required.

## License

MIT
