# htl-kiosk — map for Claude

The self-service till at the HTL: a Pi 3A+ (`pi@10.72.3.105`, hostname `interface-stock`) with the 7" Touch Display 2 running the kiosk (`kiosk/`, Rust + Slint) and a TV on HDMI showing the htl-tv page in Chromium. Pi OS Lite Trixie 32-bit + labwc, SD card read-only (overlay). Setup/recovery: README.md.

## Where things are
| What | File |
|---|---|
| Scan → cart → pay flow (state machine) | `kiosk/src/state.rs` |
| Cart, part shape (= `barcode_cache.json` entry) | `kiosk/src/cart.rs` |
| InvenTree REST (lookups, stock, prices, booking) | `kiosk/src/inventree.rs`; offline/unknown stock: `backend.rs` |
| Barcode scanner / RFID (keyboard mode) | `kiosk/src/scanner.rs` (scan debounce in `lib.rs`) |
| Payment QR (EPC/Wero) | `kiosk/src/payment.rs` |
| Link to the TV page | `kiosk/src/tv.rs` (other side: htl-tv `hooks/useTvPage.ts`) |
| UI screens / colours + fonts | `kiosk/ui/app.slint` / `kiosk/ui/tokens.slint` |
| Demo shop data | `kiosk/demo-data/`, `kiosk/tools/fetch_demo_data.py` |
| Pi setup, overlay, health line | `pi/setup.sh`, `pi/overlay.sh`, `pi/health.sh`, `pi/labwc/` |
| Watchdog, TV on/off, TV browser | `scripts/kiosk-watchdog.sh`, `scripts/tv-*.sh`, `systemd/` |
| Filament spools into InvenTree | `scripts/seed_bulk_items.py` |

## Run / test / deploy
- Preview on PC: `kiosk/preview.sh` (live window, demo shop) or `kiosk/preview.sh shots` (PNG per screen in `kiosk/target/screens/`).
- Tests: `cd kiosk && cargo test` (`tests/ui.rs` = headless tap-through).
- CI builds armhf → GitHub release `kiosk-latest`; the Pi fetches it (`pi/fetch-kiosk.sh`) via `scripts/autodeploy.sh` + `deploy.sh` every 5 min. Push to master = production.
- Not in git: `.env` (InvenTree token, payment), `barcode_cache.json` → weekly backup to SharePoint `Stock-keeping/Pi-kiosk-backup/`.

## graphify

This project has a knowledge graph at graphify-out/ with god nodes, community structure, and cross-file relationships.

Rules:
- When graphify-out/graph.json exists, run `graphify query "<question>"` before your first grep, search or source-file read for a codebase question, and put this rule in every subagent prompt that explores code. Read raw files only for lines the graph or an earlier read already located. Use `graphify path "<A>" "<B>"` for relationships and `graphify explain "<concept>"` for focused concepts. These return a scoped subgraph, usually much smaller than GRAPH_REPORT.md or raw grep output.
- If graphify-out/wiki/index.md exists, use it for broad navigation instead of raw source browsing.
- Read graphify-out/GRAPH_REPORT.md only for broad architecture review or when query/path/explain do not surface enough context.
- After modifying code, run `graphify update .` to keep the graph current (AST-only, no API cost).
- A committed graphify-out/graph.json is rewritten whole on every build, so a large diff to it in a pull request is expected. Do not flag it as churn or suggest skipping the graph rebuild or its hooks.
