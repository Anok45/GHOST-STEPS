# Ghost Steps

**Ghost Steps** is a hackathon prototype that demonstrates how wearable sensor summaries might help identify step counts that occur during vehicle travel.

## What is included

- **MCP server** (`ghost-steps-mcp/`): `analyze_steps`, `get_ghost_report`, `get_activity_timeline`, `get_truestep_score`, and `detect_transport_mode`.
- **Classifier** (`ghost-steps-mcp/src/classifier.ts`): transparent heuristic baseline using step frequency, vibration variance, jerk, movement magnitude, and optional context.
- **Python ML experiment** (`ml-model/classifier.py`): trains a Random Forest on synthetic data and exports a JSON model structure for future integration.
- **Dashboard** (`dashboard/`): responsive interactive demo with simulated step counts, score, timeline, activity chart, transport classification, and sample map.
- **Bee MCP config example** (`ghost-steps-mcp/bee-config.json`).

## Requirements

- Node.js 20 or newer recommended
- npm
- Python 3.10+ only if you want to run the optional model experiment

## Run the dashboard

From the project root:

```bash
npm install
npm run dashboard
```

Open <http://localhost:4173>. Click **Run day simulation** to update the demo metrics. The chart and map use public CDN assets, so internet access is required for those two visuals.

## Build and run the MCP server

```bash
npm install
npm run build
npm start
```

The MCP server uses stdio transport. It is intended to be launched by an MCP-compatible client; running it in a terminal will wait for the client protocol.

### Example MCP client config

Use an absolute path to this project's compiled `ghost-steps-mcp/dist/index.js` in your MCP client's config:

```json
{
  "mcpServers": {
    "ghost-steps": {
      "command": "node",
      "args": ["/absolute/path/to/ghost-steps/ghost-steps-mcp/dist/index.js"]
    }
  }
}
```

The included `ghost-steps-mcp/bee-config.json` is an example, not proof of a live Bee connection. Confirm the exact config format expected by your version of Bee/your MCP client.

## Optional synthetic model experiment

```bash
python -m venv .venv
# Windows:
.venv\\Scripts\\activate
# macOS/Linux:
source .venv/bin/activate
pip install -r ml-model/requirements.txt
python ml-model/classifier.py
```

This generates `ml-model/model.json` and a scikit-learn `.joblib` model. Training data is synthetic, so the resulting metrics are pipeline checks only—not real-world accuracy. The TypeScript MCP server currently uses the transparent heuristic classifier, not this Python model export.

## MCP tools

- `analyze_steps`: classify one sensor window and estimate potentially ghost steps.
- `get_ghost_report`: summarize supplied daily totals.
- `get_activity_timeline`: label supplied activity segments.
- `get_truestep_score`: compute an illustrative verified-step percentage.
- `detect_transport_mode`: classify sensor features into an activity/transport category.

## Important limitations / next steps

1. **No real wearable data is connected.** Bee and Apple Watch integrations require device/app access, documented APIs, permissions, and a real data flow.
2. **Not a validated ML system.** The current MCP classifier is a heuristic baseline. The Python training script uses synthetic data only.
3. **No medical or fitness claims.** The score is a demo estimate, not a validated health metric.
4. **Transport cannot be reliably inferred from these few summary features alone.** Real development needs labelled, consented accelerometer data across devices, placements, roads, and transport modes.
5. Before submission, verify the MCP server with the actual Bee/MCP client, capture screenshots, and state clearly which portions are simulated.

## Architecture

```text
Demo / Wearable data source
          |
          v
   Sensor feature window
          |
          v
  Ghost Steps MCP server <----> MCP-compatible client
          |
          v
  Heuristic classifier
          |
          v
  Step estimate + activity report

Python synthetic-data trainer (separate experiment)
Dashboard simulator (separate demo UI)
```

## License

Add a license before publishing if you want to specify reuse terms.
