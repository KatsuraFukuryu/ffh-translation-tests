# FFH R115 — TAGALOG MEMORY HEADROOM RECOVERY — READ FIRST
Date: 2026-10-16. Branch: `ffh-r115-tagalog-memory-profiles-20261016`, secondary GitHub account only.
Production remains untouched; no Render changes, Netlify deploys, Supabase writes or approvals.

## Verified GitHub outcome
**BOTH RESOURCE PROFILES PASSED**:
https://github.com/KatsuraFukuryu/ffh-translation-tests/actions/runs/38839956954

Exactly the same installed English→Tagalog model in ALL 3 experiments (R113 original and both R115 profiles): Argos `en→tl` v1.9, `.argos/packages` manifest SHA256 **cb9898879580af23ef686f19b6844d1cc5e789904db8969cc72cd907656d5146**, total installed model directory **81,433,234 bytes**.

| Configuration | Argos batch size | Beam size | Peak cgroup MiB | 512 MiB headroom | Actual outcome |
|---|---:|---:|---:|---:|---|
| R113 baseline | 8 | default 4 | 443.23 | 13.43% | **HOLD** (below 15%) |
| R115 reduced batch | 1 | 4 | 208.35 | 59.31% | **PASS** |
| R115 reduced batch + beam | 4 | 2 | 210.02 | 58.98% | **PASS** |

Each R115 profile: disposable real 512 MiB Docker memory cgroup, runtime networking disabled, real lightweight Argos + Flask/Gunicorn one worker/two threads, 10 invented single-sentence batch inputs plus 664-char paragraph, concurrent health HTTP 200, no OOM kills, correct API input limits, cache disabled and 0 production writes. No reviewed translations.

Performance on this short synthetic workload:
- `batch1-beam4`: 10-item batch 3.311 seconds; long paragraph 2.073 seconds.
- `batch4-beam2`: 10-item batch 1.264 seconds; long paragraph 1.055 seconds.
Performance is exploratory; tests ran on different GitHub runners and must not be used as a production SLA.

## Interpretation and release hold
The memory reduction is measured against **the identical model files**, not a replacement model. The `batch1-beam4` profile keeps the original beam setting, making it a better starting point for independent translation-quality comparisons, but no semantic equivalence has been proven. `batch4-beam2` explicitly changes beam search; output wording and quality may change. Compare native-language/fermentation safety quality before selecting a configuration.

Both profiles still corrupted the raw placeholder fixture before protection (`{{SAMPLE_ID}}`, one literal placeholder mismatch per 10-item batch). Combine with experimentally verified R114 placeholder shielding, fail-closed numbers/units checking, and human review only AFTER those components undergo real-service integration testing. Filipino `fil` locale versus Tagalog `tl` still needs qualified linguistic verification.

**Technical milestone: all nine FFH mapped target languages now have at least one measured single-model configuration below 512 MiB, with 15% headroom in isolated testing.** This is NOT a multi-model-on-one-instance result and NOT a production release or language-quality approval.

## Software and next task
- `tests/r115.Dockerfile` has configurable `ARGOS_BATCH_SIZE`/`ARGOS_BEAM_SIZE`.
- `tests/r113_single_model_matrix.py` was hardened so post-test memory gate FAIL/HOLD overrides preliminary API PASS in saved `summary.json`; fixes R113 Tagalog report-order bug.
- `.github/workflows/ffh-r115-tagalog-profiles.yml` runs only on isolated branch.
- Further: add a longer Tagalog stress profile; compare quality with original beam4 settings; verify models' remote download package digest before installation; test supported upstream source variant, deployment package, Render-equivalent health under load, privacy/rate controls, genuine human safety review and staged rollback.
- Never merge into `main` without separate owner authorization because Render auto-deploys it.
