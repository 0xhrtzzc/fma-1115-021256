# Verification — free-model-aggregator-02 entry

Honest note of what I checked about this server, and what I did not.

## The file itself

- `main.py` was written byte for byte as given: 1525 lines / 65588 bytes /
  sha256 `f6726654bdc2d41cd96900caf25e781d9221975984f75147fc8a38612cdf4db5`, and `diff` against a
  copy taken before the write reports no differences.
- One comment line was then appended at the very end (the patch added one line and changed no
  other byte); the file is now 1526 lines / 65721 bytes.
- `python3 -c "import ast; ast.parse(open('main.py').read()); print('parses ok')"` prints `parses ok`
  (see the end of this file).

## What I ran against it (Python 3.11, this sandbox)

- `python3 -P -s main.py --list` with a brief in `MAKEFUN_PROMPT_FILE`: prints one compact JSON
  catalog line — one provider, `base_url` the public provider endpoint, `key_env` `OPENROUTER_API_KEY`,
  `wire` `openai`, and exactly the free model ids — and exits 0 without starting a server.
- An end-to-end run against a **stub** env and a **stub** gateway of my own (not the grader's), with
  a four-row manifest and a brief holding three free models plus one paid row. The entry was started
  as `python3 -P -s main.py` and driven over HTTP on the port it registered. 22 of 22 checks passed:
  - register body carries `protocol_version`, the picked port and the catalog; the catalog lists the
    free ids once each and leaves the paid row out;
  - `GET /v1/models` answers both shapes (an `Authorization` request and an `x-api-key` request) with
    the same ids;
  - `POST /v1/chat/completions` non-streaming: served by the dialled id of the best-rated model, the
    provider's whole usage counts passed through, a legal `finish_reason`;
  - `POST /v1/chat/completions` streaming: role chunk, text deltas, finish chunk, the empty-`choices`
    usage chunk, `[DONE]`, every chunk naming the dialled id;
  - `POST /v1/messages` non-streaming: message shape, `input_tokens`/`output_tokens` whole counts, a
    legal `stop_reason`, a text block carrying the answer;
  - `POST /v1/messages` streaming: `message_start` first, a content-block run, then `message_delta`
    and `message_stop`, with nothing after `message_stop` and the input count on `message_start`;
  - failover: a first 429 on the best model was retried on the same model and that success was
    reported in `x-makefun-upstream-failures`; with the best model then refusing for good, the prompt
    was served by the next-best model that meets it;
  - every upstream dial the stub gateway saw was non-streaming;
  - the process exited 0 the moment `GET /work` answered `done`.

## What I did NOT verify

- The real make.fun env, the real gateway, the real key and the fourteen real free OpenRouter models:
  I have no access to any of them here, so nothing above says the live models answer, or how they
  rate. Only the entry's own behaviour against stubs was measured.
- Determinism across a live pass and its replay: not testable here, but nothing in the file reads a
  clock, a random source or a UUID, and the two fixed instants it writes are constants.
- The checks the grader actually applies; my harness measures what I could see from outside.

# End of the note.

# This note records what was really exercised: a stub-env/stub-gateway run of both wires plus the catalog path and the byte-exact write.
# The claims above stop where my stubs stop: they say nothing about the live models, the real gateway or the grader.
