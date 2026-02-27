# GLAID — Glucose & Insulin AI Dashboard

A single-file, browser-based dashboard for visualising and analysing Glooko CGM and insulin pump exports. No server, no installation, no data ever leaves your device (unless you actively choose to use a cloud AI provider).
Chat with your data and extract insights!

Should work for most pumps but currently tailored for Tandem (see section Hypo Treatment Detection).

![GLAID Dashboard](https://img.shields.io/badge/version-1.0-teal) ![No backend](https://img.shields.io/badge/backend-none-green)

---

## Features

### Data Visualisation
- **Glucose & Insulin plot** — CGM trace, basal rate, and insulin-on-board (IOB) on a single zoomable, pannable chart. Scroll to zoom (slow, fine-grained), drag to pan. Bolus events are rendered as flag annotations directly on the chart, colour-coded by type.
- **Bolus flag types:**
  - 🔵 **Meal bolus** — blue flag showing grams of carbohydrate and units delivered
  - 🔴 **Correction bolus** — red flag showing units delivered
  - 🟠 **Hypo treatment** — orange flag (boluses of exactly 0.05 U), labelled with the configured carbohydrate amount
- **Glucose by Hour of Day** — percentile curves (5th/25th/median/75th/95th) with average basal rate overlay. Hover to highlight the hour slot and see a detailed tooltip. Curves extend to 00:00 for a complete 24-hour view.
- **Insulin, Carbs & Hypos by Hour of Day** — side-by-side bar chart showing average grams of carbohydrate, average bolus units, and hypo treatment frequency per hour. Hovering highlights the hour slot across both HOD plots simultaneously.

### Statistics (always reflect the visible chart window)
| Metric | Description |
|--------|-------------|
| Average Glucose | Mean CGM value in mmol/L |
| Time in Range | % within selected target range |
| Time Below Range | % below lower target |
| Time Above Range | % above upper target |
| GMI | Estimated HbA1c in mmol/mol and % |
| Glucose Variability | Coefficient of variation (%CV) |
| Total Daily Insulin | Basal + bolus U/day |
| Basal / Bolus | Ratio as % with U/day sub-label |
| Avg Carbs / Day | Mean grams logged per day from bolus records |
| Total Bolus Doses | Count of all bolus events |
| Hypo Treatments | Count, frequency, and configured carb dose |

Statistics, the HOD plots, and AI analysis all update live as you pan or zoom the main chart.

### AI Analysis & Chat
- **Structured clinical analysis** — generates a detailed report covering glycaemic control, diurnal patterns, insulin delivery assessment, and evidence-based recommendations, with inline citations to ADA, EASD, and other guidelines.
- **Follow-up chat** — continue the conversation with full context of your data and the prior analysis.
- **Multiple AI providers supported:**
  | Provider | Notes |
  |----------|-------|
  | **Anthropic API** | Claude Sonnet. Supports web search for citing studies. Requires an API key from [console.anthropic.com](https://console.anthropic.com). |
  | **Ollama** | Any locally-running model. Start with `OLLAMA_ORIGINS="*" ollama serve`. |
  | **llama.cpp** | HTTP server mode. Add `--cors-allow-origins "*"` to your server command. |
  | **OpenClaw** | OpenAI-compatible gateway. Run `openclaw gateway --port 18789`. |

### Controls
- **Range presets** — 1D, 2D, 3D, 1W, 2W, 4W, All
- **Custom date range** — from/to date pickers
- **Target range** — 3.9–7.8 mmol/L (tight control) or 4.0–10.0 mmol/L (standard)
- **DIA** — duration of insulin action for IOB calculation (3–6 h, default 5 h)
- **Age & Weight** — used to contextualise TDD/kg in the AI analysis
- **Hypo treatment size** — configurable carbohydrate dose (default 15 g) applied to all 0.05 U boluses

---

## Getting Started

GLAID is a single HTML file with no build step and no dependencies to install.

1. Download `diabetes_dashboard.html`
2. Open it in any modern browser (Chrome, Firefox, Safari, Edge)
3. Export your data from Glooko in zip format
4. Drag and drop the ZIP onto the dashboard, or click to browse

That's it. Everything runs locally in the browser.

---

## Data Format

GLAID reads Glooko's standard CSV export ZIP, which must contain:

| File (matched by name fragment) | Contents |
|---------------------------------|----------|
| `cgm_data*.csv` | Timestamped CGM glucose readings |
| `basal_data*.csv` | Basal rate segments with rate (U/h) and duration |
| `bolus_data*.csv` | Bolus events with insulin delivered and carbs input |

The export format is compatible with pumps supported by Glooko.

---

## Hypo Treatment Detection

GLAID treats any bolus of **exactly 0.05 U** with no associated carbohydrate input as a hypo treatment signal. This convention is used by CamAPS FX to log manual carbohydrate interventions without delivering insulin. When such a bolus is detected:

- The chart flag turns **orange** and is labelled with the configured carb dose rather than the bolus amount
- The statistics card counts these events separately from corrections
- The AI analysis is told the hypo frequency and carb dose

The carbohydrate amount per hypo treatment (default: 15 g) is configurable in the controls bar.

---

## IOB Calculation

Insulin-on-board is calculated using a bilinear insulin activity curve. The duration of insulin action (DIA) is user-configurable (3–6 hours). The peak activity is set at 35% of DIA. IOB is computed at every CGM timestamp across the full dataset.

---

## Privacy

- **No data is sent anywhere** unless you enter an API key and click Analyse.
- If you use the Anthropic API, your statistics summary (no raw CGM values) is sent to Anthropic's servers for analysis. No data is retained beyond the API call per Anthropic's standard terms.
- If you use a local provider (Ollama, llama.cpp, OpenClaw), all processing happens entirely on your machine.

---

## Medical Disclaimer

GLAID is a personal data visualisation tool intended to help you explore your own glucose and insulin data. It is **not a medical device** and does not provide medical advice. All AI-generated analysis is for informational purposes only and must be reviewed with your diabetes care team before making any changes to your insulin settings or management plan.

---

## Technical Details

- **Zero dependencies to install** — JSZip and Chart.js are loaded from cdnjs.cloudflare.com; fonts from Google Fonts
- **Single HTML file** — everything including all JavaScript and CSS is self-contained
- **No cookies, no localStorage, no telemetry** — all state is in-memory only
- **Responsive** — works on desktop and tablet; touch pan and pinch-to-zoom supported on the main chart

---

## License

Currently no-license. Will update when/if this is released to public.

---

## Acknowledgements

Built with [Chart.js](https://www.chartjs.org/), [JSZip](https://stuk.github.io/jszip/), and the [Anthropic API](https://www.anthropic.com/). Designed for use with [Glooko](https://www.glooko.com/) exports
