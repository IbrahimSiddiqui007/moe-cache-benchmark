# moe-cache-benchmark

> **This tool is superseded and no longer maintained.** Benchmarking now lives in moe-cache itself: run
> `python3 bin/moe_cache.py bench-report MODEL.gguf`, or press **Share my numbers** on the moe-cache control page. It starts both arms
> (stock llama.cpp and moe-cache) through the same launcher plan, works on Windows and Linux; from v0.3.8 it can also run a real multi-turn conversation.
> Numbers from this repository used an 8k context with short prompts; do not compare them with moe-cache's current results.
> See [moe-cache](https://github.com/IbrahimSiddiqui007/moe-cache).

Benchmarks for [moe-cache](https://github.com/IbrahimSiddiqui007/moe-cache): stock llama.cpp against llama.cpp with the moe-cache plugin, on **your** machine,
with **your** model and prompts, under a memory limit you choose, with a check that both produce the same tokens.
Split out of the moe-cache repository on 2026-10-14 (from moe-cache commit `b59288f`).

## What you need

- Linux, Python 3 (standard library only), `systemd-run` for the memory limits.
- A `llama-server` built with shared ggml libraries, and a built moe-cache next to this folder:

```bash
git clone https://github.com/IbrahimSiddiqui007/moe-cache && (cd moe-cache && ./build.sh)
git clone https://github.com/IbrahimSiddiqui007/moe-cache-benchmark
```

The tool finds moe-cache (plugin build and planner) in this order: `MOE_CACHE_DIR`, a `moe-cache` folder next to this one, `~/moe-cache-setup/moe-cache`
(where moe-cache's `install.sh` puts it), `~/moe-cache`.

## Contents

| Path | What it is |
|---|---|
| `bin/moe-cache-bench` | web page for runs: speed, identity, your own prompts, deep metrics (http://127.0.0.1:8765) |
| `bin/moe-cache-bench-watch` | terminal view of a running benchmark |
| `bench/standard.py` | the standard protocol STP-1 ([docs/TEST_PROTOCOL.md](docs/TEST_PROTOCOL.md)) |
| `bench/run_matrix.py` | many models x placements x memory limits, publishable table |
| `bench/quality.py` | quality harness: compare, longctx, soak, judge-calibrate, codegen |
| `bench/coding_session.py` | coding-session benchmark (prompt reuse on a large context) |
| `tools/wait_cool.sh` | the clean-run gate as one command |
| [docs/BENCHMARKS.md](docs/BENCHMARKS.md) | what is measured and how to run capability suites |

## Using it

The tool (`bin/moe-cache-bench`) needs only the Python standard library. It measures stock llama.cpp and moe-cache on **your** machine,
with **your** model and **your** prompts, under a memory limit you choose, and checks that both produce the same tokens.

```bash
bin/moe-cache-bench            # starts the tool and opens http://127.0.0.1:8765
```

1. **Model**: give the path of your `.gguf` (the llama-server and plugin paths are filled in when found).
2. **Memory limit (GB)**: the RAM you want to emulate for the model. A model larger than this limit is the case moe-cache is made for; set 2 or 3 limits (for example 12 and 24) to see the curve.
3. **Modes**: tick stock and moe-cache; leave "both orders" on (it runs forward and reverse to cancel drift).
4. **Your own prompts** (optional): paste prompts separated by a line with only `---`; add `EXPECT: word` lines to mark words the answer must contain. They are run on both modes and compared.
5. Press **Start**. Watch it in the browser, or in a terminal with `bin/moe-cache-bench-watch`.
6. Read the result: steady generation speed (mean of requests 2 to 4), time to first token, SSD read per token, cache hit rate, and PASS/FAIL for identical tokens. Runs that started hot or with background load are flagged and should be repeated.

Command line, for a fixed comparable protocol (two placements, two limits, both orders; see [docs/TEST_PROTOCOL.md](docs/TEST_PROTOCOL.md)):

```bash
bin/moe-cache-bench --no-browser &
bench/standard.py MODEL.gguf --limits 12,24 --profiles cpu,gpu
```

The shareable HTML report and JSON export contain no prompts, outputs or local paths; the unfiltered versions are for you only (see [docs/SECURITY.md](https://github.com/IbrahimSiddiqui007/moe-cache/blob/main/docs/SECURITY.md)).
Benchmark every model on your disk (all Ollama models and all GGUF files in a folder, both placements, several memory limits) and get a table to publish:

```bash
bench/run_matrix.py --ollama --dir ~/models           # prints the plan: models, cells, limits
bench/run_matrix.py --ollama --dir ~/models --run     # runs it; writes results-matrix-DATE.md and .csv
```

What the benchmarks mean and how to run the usual capability suites against the same server: [docs/BENCHMARKS.md](docs/BENCHMARKS.md).

## Status

Feature-frozen: bugs are fixed, new features are not planned. Only clean runs are quoted (the tool flags runs that started hot or under load).

## License

MIT, see [LICENSE](LICENSE).
