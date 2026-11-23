# ADR-0001: Use dbt Core v1 (1.12) instead of dbt v2

- **Status:** Accepted
- **Date:** 2026-11-22

## Decision

I use **dbt Core 1.12 with the dbt-duckdb adapter**, installed as a regular Python
dependency and pinned below 2.0 (`dbt-core>=1.12,<2` in `pyproject.toml`).
I write the models so that a later move to dbt v2 needs as few changes as possible.

## Context

- dbt v2 (the Rust-based engine formerly called Fusion) was released in
  September 2026, about two months before I started this project. The dbt v1 CLI now
  points new projects to v2.
- This project already depends on Python for everything around dbt: file parsers,
  the fake-data generator, pytest, sqlfluff and the Streamlit dashboard. With v1,
  dbt lives in the same `uv.lock` as the rest, so one command installs the whole
  stack on Windows and on the Linux CI runner, at the same versions.
- The warehouse is a local DuckDB file. I load the upstream files (bank and card
  exports) with Python and DuckDB, so DuckDB-specific features matter more here
  than raw parsing speed.
- I am getting back into day-to-day coding. Most tutorials, answers and
  job descriptions still cover dbt v1.

## Evidence

- **v1 is still supported.** dbt Labs states that v1 remains fully supported;
  v1.12 is listed with active support until July 2027.
  ([About dbt versions](https://docs.getdbt.com/docs/dbt-versions),
  [AceMQ](https://acemq.com/blogs/dbt-core-v1-vs-dbt-v2-licence-and-support-options/))
- **DuckDB support in v2 is not at parity yet.** The official DuckDB setup page for
  v2 says some features of the v1 dbt-duckdb adapter are not supported yet. It also
  says v2 may fail to infer schemas from `read_csv()` / `read_parquet()` sources,
  and that the bundled driver cannot load DuckDB extensions.
  ([dbt DuckDB setup](https://docs.getdbt.com/docs/local/connect-data-platform/duckdb-setup),
  [DuckDB blog](https://duckdb.org/2026/09/22/dbt-fusion))
- **Licensing is simpler on v1.** dbt Core v1 is Apache 2.0. dbt v2 is free to use
  but proprietary; only the `dbt-oss` subset is Apache 2.0.
  ([dbt licensing](https://docs.getdbt.com/docs/dbt-licensing))
- **The v1 stack works on this machine.** `dbt --version` reports
  dbt-core 1.12.7 and dbt-duckdb 1.11.1 on Windows 10 with Python 3.12.

## Discarded alternative

**dbt v2 (or dbt-oss) with its built-in DuckDB adapter.** It is faster, ships as a
single binary and is where new features will land. I did not pick it now because
the DuckDB adapter still has known gaps, the release is very recent, and a second,
non-Python install would complicate CI for little benefit at this data size
(thousands of rows, not millions).

To keep migration cheap:

- keep configs and YAML strict (no unknown keys, no deprecated flags), since v2
  turns these into parse errors;
- *to verify:* run the v2 compatibility check available in 1.12 in CI
  (a third-party guide mentions `dbt-autofix` and `--use-v2-parser`).

**Revisit when:** the DuckDB adapter in v2 reaches feature parity with dbt-duckdb,
when v1.12 support approaches its end (mid-2027), or when a target employer
requires v2.
