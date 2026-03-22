# GLAID — Glucose & Insulin AI Dashboard

A single-file, browser-based dashboard for visualising and analysing Glooko CGM and insulin pump exports. No server, no installation, no data ever leaves your device (unless you actively choose to use a cloud AI provider).
Chat with your data and extract insights!

Should work for most pumps but currently tailored for Tandem (see section [Hypo Treatment Detection](#hypo-treatment-detection)).

![GLAID Dashboard](https://img.shields.io/badge/version-0.26.2--beta2-teal) ![No backend](https://img.shields.io/badge/backend-none-green)

---

## Features

### Data Visualisation

- **Glucose & Insulin plot** — CGM trace, basal rate, and insulin-on-board (IOB) on a single zoomable, pannable chart. Scroll to zoom (slow, fine-grained), drag to pan. Bolus events are rendered as flag annotations directly on the chart, colour-coded by type.
- **Bolus flag types:**
  - 🔵 **Meal bolus** — blue flag showing grams of carbohydrate and units delivered
  - 🔴 **Correction bolus** — red flag showing units delivered
  - 🟠 **Hypoglycaemia treatment** — orange flag (boluses of exactly 0.05 U), labelled with the configured carbohydrate amount
- **Glucose by Hour of Day** — percentile bands (5–95th, 25–75th), smoothed median, raw (unsmoothed) median, and mean glucose curves. Average basal rate and an **Adjusted Basal** suggestion are overlaid on a right-hand axis. A basal info box shows Avg Basal/day, Adjusted Basal/day, and the difference. Hover to highlight the hour slot and see a detailed tooltip. Curves extend to 00:00 for a complete 24-hour view.
- **Glucose Rate of Change by Hour of Day** — a compact chart directly below the glucose/basal plot showing the per-hour distribution of glucose rate of change (mmol/L/h), computed via central differencing. Displays IQR band, smoothed median, and mean. Hover on either plot highlights the same hour in both simultaneously.
- **Insulin, Carbs & Hypoglycaemia Treatments by Hour of Day** — side-by-side bar chart showing average grams of carbohydrate, average bolus units, and hypoglycaemia treatment frequency per hour. Bars are averaged over days with actual events in each slot (not total period days). Hovering highlights the hour slot.
- **Post-Prandial Glucose Excursion by Hour of Day** — for each meal bolus, plots Δ interstitial glucose at +3 h and +4 h after the bolus time, bucketed by the hour the bolus was taken. Column highlight overlay on hover.
- **Post-Prandial Interstitial Glucose Trace (0–4 h)** — overlays individual 4-hour CGM traces for all meal boluses administered within a user-selected daily time window. Displays a bold cohort median trace and an IQR shaded band. Hover tooltip shows median, IQR, mean, and meal count at each 5-minute timepoint. See [Post-Prandial Glucose Trace](#post-prandial-glucose-trace) for details.

### Statistics

Statistics cards are displayed **above** the main Glucose & Insulin chart and always reflect the currently visible chart window (including zoom/pan state).

| Metric | Description |
|--------|-------------|
| Average Glucose | Mean CGM value in mmol/L |
| Time in Range | % of readings within selected target range |
| Time Below Range | % of readings below lower target threshold |
| Time Above Range | % of readings above upper target threshold |
| GMI | Estimated glycated haemoglobin (HbA1c) in mmol/mol and % |
| Glucose Variability | Coefficient of variation (%CV) |
| Total Daily Insulin | Combined basal and bolus insulin in U/day |
| Estimated Insulin Sensitivity | Calculated via the 100/TDD rule (mmol/L/U) |
| Basal / Bolus | Split as % with U/day sub-label |
| Avg Carbs / Day | Mean carbohydrate intake (g/day) from bolus records; a secondary line shows the value including hypoglycaemia treatment carbohydrates when present |
| Total Bolus Doses | Count of all bolus events in the selected period |
| Hypoglycaemia Treatments | Count, frequency per day, and configured carbohydrate dose |

Statistics, the HOD plots, and AI analysis all update live as you pan or zoom the main chart.

### AI Analysis & Chat

- **Structured clinical analysis** — generates a detailed report covering glycaemic control, diurnal patterns, insulin delivery assessment, and evidence-based recommendations, with inline citations to ADA, EASD, and other guidelines.
- **Web search integration** — when using the Anthropic API, the AI can retrieve and cite current peer-reviewed literature to support its analysis.
- **Follow-up chat** — continue the conversation with full context of your data and the prior analysis.
- **Multiple AI providers supported:**

  | Provider | Notes |
  |----------|-------|
  | **Anthropic API** | Claude Sonnet. Supports web search for citing studies. Requires an API key from [console.anthropic.com](https://console.anthropic.com). |
  | **Ollama** | Any locally-running model. Start with `OLLAMA_ORIGINS="*" ollama serve`. |
  | **llama.cpp** | HTTP server mode. Add `--cors-allow-origins "*"` to your server command. |
  | **OpenClaw** | OpenAI-compatible gateway. Run `openclaw gateway --port 18789`. |
  | **LM Studio** | Run LM Studio server with CORS enabled. Optional MCP web search support. |

### Controls

The controls bar is **sticky** — it remains pinned to the top of the viewport as you scroll down through the charts.

- **Range presets** — 1D, 2D, 3D, 1W, 2W, 4W, All
- **◀ ▶ navigation arrows** — shift the current date window backwards or forwards by one full period (e.g. pressing ▶ while viewing a 1-week window advances by 7 days). Arrows are disabled at the data boundaries.
- **Custom date range** — from/to date pickers
- **Target range** — 3.9–7.8 mmol/L (tight glycaemic control) or 4.0–10.0 mmol/L (standard)
- **Target glucose** — target interstitial glucose value (default 6.0 mmol/L) used in the Adjusted Basal calculation
- **DIA** — duration of insulin action for IOB calculation (3–6 h, default 5 h)
- **Age & Weight** — used to contextualise TDD/kg in the AI analysis
- **Hypoglycaemia treatment size** — configurable carbohydrate dose (default 15 g) applied to all 0.05 U boluses

---

## Getting Started

GLAID is a single HTML file with no build step and no dependencies to install.

1. Download `GLAID.html`
2. Open it in any modern browser (Chrome, Firefox, Safari, Edge)
3. Export your data from Glooko in ZIP format
4. Drag and drop the ZIP onto the dashboard, or click to browse

That's it. Everything runs locally in the browser.

---

## Data Format

GLAID reads Glooko's standard CSV export ZIP, which must contain:

| File (matched by name fragment) | Contents |
|---------------------------------|----------|
| `cgm_data*.csv` | Timestamped CGM glucose readings |
| `basal_data*.csv` | Basal rate segments with rate (U/h) and duration |
| `bolus_data*.csv` | Bolus events with insulin delivered and carbohydrate input |

The export format is compatible with all pumps supported by Glooko. Both the **English** and **Swedish** Glooko interfaces are supported — see [Swedish Locale Support](#swedish-locale-support) for details.

---

## Swedish Locale Support

Exports from the Swedish-language Glooko interface (`glooko.com/sv`) are fully supported. The following differences from the English export are handled automatically:

**Column names** — Swedish column headers are recognised alongside their English equivalents:

| English | Swedish |
|---------|---------|
| `Timestamp` | `Tidstämpel` |
| `CGM Glucose Value (mmol/l)` | `CGM-glukosvärde (mmol/l)` |
| `Duration (minutes)` | `Varaktighet (minuter)` |
| `Rate` | `Frekvens` |
| `Insulin Delivered (U)` | `Insulin tillfört (enh.)` |
| `Carbs Input (g)` | `Inmatning av kolhydrater (g)` |

**Decimal separator** — Swedish exports use a comma as the decimal separator (e.g. `"5,8"` for 5.8 mmol/L). GLAID normalises these automatically before parsing.

**Date formats** — the Swedish export uses `YYYY/MM/DD HH:MM` format, which is handled alongside all other supported date formats (see below).

**Basal rate** — the Swedish basal file (`basal_data`) does not include an explicit rate column. GLAID derives the rate from `Insulin tillfört (enh.)` ÷ (`Varaktighet (minuter)` ÷ 60).

**Supported date formats** — GLAID accepts any of the following, with colon or dot as the time separator:

`YYYY-MM-DD HH:MM:SS` · `YYYY/MM/DD HH:MM` · `YYYY.MM.DD HH:MM:SS` · `DD/MM/YYYY HH:MM:SS` · `DD-MM-YYYY HH:MM:SS` · `DD.MM.YYYY HH:MM:SS` · `MM/DD/YYYY HH:MM:SS`

---

## Hypo Treatment Detection

GLAID treats any bolus of **exactly 0.05 U** with no associated carbohydrate input as a hypoglycaemia treatment signal. This is a method we use with Tandem Control-IQ to block the pump from delivering additional boluses during the rapid glucose rise following carbohydrate ingestion, while logging the manual carbohydrate intervention and delivering as little insulin as possible. When such a bolus is detected:

- The chart flag turns **orange** and is labelled with the configured carbohydrate dose rather than the bolus amount
- The statistics card counts these events separately from correction boluses
- The AI analysis receives the hypoglycaemia frequency and carbohydrate dose

The carbohydrate amount per hypoglycaemia treatment (default: 15 g) is configurable in the controls bar.

---

## Post-Prandial Glucose Trace

The **Post-Prandial Interstitial Glucose Trace** panel plots individual 4-hour CGM traces, each normalised so that Δglucose = 0 at the moment of bolus administration (time 0). Only meals (boluses with carbohydrate > 0) within the selected daily time window are included. Traces with fewer than 70% of time points covered by CGM data are excluded.

### Meal Window Selector

A dual-handle range slider lets you select the daily time window for meal inclusion (e.g. 10:00–13:00 to isolate lunchtime meals):

- **Drag either thumb** — adjust the start or end time independently (the nearest thumb to the click point is automatically activated)
- **Drag the teal fill bar** — shift the entire window forwards or backwards while preserving its width
- The chart updates in real time as you adjust the window

### What the chart shows

| Element | Description |
|---------|-------------|
| Faint individual traces | Each line is one meal; opacity scales with meal count so dense data remains legible |
| Shaded IQR band | 25th–75th percentile range across all meal traces at each time point |
| Bold teal median line | Cohort median interstitial glucose change at each 5-minute interval |

A value of 0 mmol/L indicates return to pre-prandial glucose baseline. Positive values indicate post-prandial hyperglycaemia; negative values indicate post-prandial hypoglycaemia relative to the pre-meal level.

---

## IOB Calculation

Insulin-on-board is calculated using a bilinear insulin activity curve. The duration of insulin action (DIA) is user-configurable (3–6 hours, default 5 h). Peak activity is set at 35% of DIA. IOB is computed at every CGM timestamp across the full dataset.

---

## Privacy

- **No data is sent anywhere** unless you enter an API key and click Analyse.
- If you use the Anthropic API, your statistics summary (no raw CGM values) is sent to Anthropic's servers for analysis. No data is retained beyond the API call per Anthropic's standard terms.
- If you use a local provider (Ollama, llama.cpp, OpenClaw, LM Studio), all processing happens entirely on your machine.
- No cookies, no localStorage, no telemetry — all session state is in-memory only.

---

## Medical Disclaimer

GLAID is a personal data visualisation tool intended to help you explore your own continuous glucose monitoring (CGM) and insulin pump therapy data. It is **not a medical device** and does not provide medical advice, clinical diagnosis, or therapeutic recommendations.

All AI-generated analysis — including pattern summaries, insulin dosing commentary, and any content retrieved via web search — is for informational and educational purposes only. Web-retrieved content may be incomplete, out-of-date, or miscontextualised.

**Do not modify your insulin regimen** — including your basal rate programme, carbohydrate-to-insulin ratio (CIR), or insulin sensitivity factor (ISF) — based solely on outputs from this tool. Any changes to your diabetes management plan must be reviewed and approved by a qualified healthcare professional, such as a consultant endocrinologist, diabetologist, or specialist diabetes nurse, before implementation.

---

## Technical Details

- **Zero dependencies to install** — JSZip and Chart.js are loaded from cdnjs.cloudflare.com; fonts from Google Fonts
- **Single HTML file** — all JavaScript, CSS, and logic are fully self-contained
- **No cookies, no localStorage, no telemetry** — all state is held in-memory for the duration of the session
- **Responsive** — designed for desktop and tablet; touch pan and pinch-to-zoom are supported on the main chart; the controls bar sticks to the top of the viewport on scroll

---

## Changelog

### v0.26.2-beta2 (2026-03-22)

**New chart: Glucose Rate of Change by Hour of Day**
- A compact chart sits directly below the Glucose by Hour of Day / Adjusted Basal plot, showing the distribution of glucose rate of change (mmol/L/h) at each hour of the day.
- Rate of change computed via central differencing of individual CGM readings, binned by hour. Displays Gaussian-smoothed median (purple), mean (dashed blue), and IQR band (25th–75th percentile). The 5–95th percentile outer band is omitted to reduce noise.
- **Synced hover** — moving the pointer over either the glucose chart or the ROC chart highlights the same hour column in both simultaneously. The main glucose tooltip also shows the median ROC for the hovered hour.

**Adjusted Basal — formula rework**
- The adjusted basal formula was completely rewritten and now has two independent components:
  1. **Rate-of-change correction** — for each hour h (h = 1…22, non-wrapping to avoid midnight artefacts), the smoothed median ROC is computed via central difference. The correction `ROC / ISF` U/h is split evenly across the two preceding hours (h−1 and h−2), shifting the basal shape to counteract glucose trends.
  2. **Target glucose offset** — `(avg glucose − target glucose) / ISF` units distributed uniformly across all 24 hours, shifting the overall basal level to steer average glucose toward the user-set target.
- Previous formula bugs fixed: circular wrap-around artefact (which injected spurious corrections at midnight boundaries) eliminated by using non-wrapping indices; correction bolus component removed (was always additive, overriding negative ROC corrections).

**Target Glucose input**
- A "Target Glucose" number input (default 6.0 mmol/L, range 3–12) added to the controls bar beside the Target Range selector. Used by the Adjusted Basal level-offset component.

**Statistics**
- **Avg Carbs / Day** card now shows a secondary line (`* Xg incl. hypo Tx`) with the average daily carbohydrate intake including hypoglycaemia treatment doses, when hypo treatments are present in the selected period.

**Bug fixes**
- **Navigation arrows date drift** — pressing ◀ then ▶ (or vice versa) no longer shifts the date window by 1–2 days. Root cause: `fmtDateInput` used `toISOString()` which returns UTC, so in UTC+1/+2 timezones local midnight was formatted as the previous calendar day. Fixed to use local date components (`getFullYear()`, `getMonth()`, `getDate()`).

---

### v0.26.2-beta1 (2026-03-18)

**Layout & UX**
- **Statistics cards moved above the main chart** — key glycaemic metrics are now immediately visible on data load, before scrolling.
- **Controls bar is now sticky** — the range/date/settings bar remains pinned to the top of the viewport while scrolling through charts. A drop-shadow provides visual separation from content below.
- **Navigation arrows ◀ ▶** added to the controls bar. Each click shifts the date window by one full period in either direction (e.g. 1-week window → step by 7 days). Arrows are disabled at the data boundaries.

**New chart: Post-Prandial Interstitial Glucose Trace (0–4 h)**
- Plots all individual meal CGM traces within a user-selected daily time window, normalised to pre-meal interstitial glucose at time 0.
- Bold cohort **median trace** and **IQR shaded band** (25th–75th percentile) with border strokes for clear delineation.
- **Dual-handle range slider** to select the daily meal time window (e.g. 10:00–13:00). The entire window can be panned by dragging the teal fill bar, with individual handles adjustable independently.
- **Hover tooltip** showing median, IQR, mean Δ interstitial glucose, and meal count at each 5-minute post-bolus timepoint.
- **Column highlight overlay** — a teal vertical band tracks the pointer position, consistent with the other HOD charts.

**Post-Prandial Excursion by Hour of Day — interactivity**
- **Column highlight overlay** added: a teal vertical band now highlights the hovered hour column, matching the behaviour of the Insulin, Carbs & Hypoglycaemia Treatments chart.

**Post-prandial plots — visual refinements**
- Removed the target glucose range shading from the Δ-glucose axes (absolute mmol/L bands are not meaningful on a delta scale).
- IQR band opacity increased for greater prominence; upper and lower boundary strokes added.
- Median trace line weight increased and rounded caps applied.
- **±2 mmol/L dashed reference lines** added to both delta plots (amber above zero, red below) to indicate clinically meaningful post-prandial excursion thresholds.
- Positive y-axis tick labels display without a `+` prefix (sign implied); negative labels retain the `−` sign. Tooltip values retain the `+` prefix for signed delta readability.

**Glucose by Hour of Day**
- **Y-axis ceiling tightened** — the maximum is now calculated as the nearest 0.5 mmol/L above the 95th percentile data, eliminating excess headroom above the percentile bands.

**Terminology**
- Medical terminology updated throughout: *post-prandial*, *interstitial glucose*, *hypoglycaemia*, *carbohydrate-to-insulin ratio (CIR)*, *insulin sensitivity factor (ISF)*, *euglycaemic baseline*, *bolus administration*, etc.

**Disclaimer**
- Comprehensive medical and AI disclaimer added as a persistent section at the bottom of the page, covering AI outputs, web search retrieval, and insulin adjustment guidance.
- Inline disclaimers added above the AI Analysis section and the Data Chat interface.

**Insulin, Carbs & Hypoglycaemia Treatments by Hour of Day — averaging fix**
- Carbs and bolus bars are now averaged over the number of **days with actual events in each hour slot**, rather than the total period duration. Previously, a meal eaten on 20 of 28 days was deflated to show 20/28ths of its true average size. Values now correctly reflect the per-occurrence average on days when the meal or bolus occurred. Tooltip shows the day count (`n=X days`) for each slot.

**Swedish locale support**
- Full support for exports from the Swedish-language Glooko interface (`glooko.com/sv`).
- Swedish column names recognised: `Tidstämpel`, `CGM-glukosvärde (mmol/l)`, `Varaktighet (minuter)`, `Frekvens` (basal rate), `Insulin tillfört (enh.)`, `Inmatning av kolhydrater (g)`.
- Comma decimal separators (`"5,8"`) parsed correctly throughout.
- Basal rate derived from insulin delivered ÷ duration when no explicit rate column is present (as in the Swedish basal export format).
- Additional date formats added: `YYYY/MM/DD HH:MM`, `DD.MM.YYYY HH:MM:SS`, `YYYY.MM.DD HH:MM:SS`, and all variants with dot as time separator.
- Error message on file load now lists the CSV files found in the ZIP to aid diagnosis of unrecognised export formats.

---

### v0.26.1-beta3 (2026-03-03 – 2026-03-06)

**Glucose by Hour of Day — new curves**
- Added **mean glucose curve** (dashed blue) alongside the existing smoothed median.
- Added **raw (unsmoothed) median** (faint dashed teal) so the effect of Gaussian smoothing is directly visible. Tooltip shows both values.

**Adjusted Basal suggestion**
- Added an **Adjusted Basal** curve (dashed teal, right axis) to the Glucose by Hour of Day plot. Two correction components are applied to the average basal rate for each hour *h*:
  - *Glucose drift*: `Δ = median[h+3] − median[h]`; add `Δ / (ISF × hours²)` to each of hours h, h+1, h+2. The `1/hours²` factor corrects for compounding: each hour accumulates contributions from 3 overlapping drift windows.
  - *Correction boluses*: insulin-only boluses (excluding 0.05 U hypoglycaemia markers) are averaged per hour and distributed as `C(h) / 3` over the 3 preceding hours.
  - All rates are floored at 0. This is **not a clinical recommendation**.
- Added a **basal info box** showing Avg Basal/day, Adjusted Basal/day, and the difference.
- Removed the **Target BG** input (was only used by the previous adjusted basal formula).

**Post-Meal Glucose Change by Hour of Day (new chart)**
- Replaced the glucose-vs-prior-basal scatter plot with a chart showing Δ interstitial glucose at +3 h and +4 h after each meal bolus, grouped by the hour the bolus was taken.
- Data points placed at the exact fractional bolus timestamp (not bucketed to midpoints).
- Curves connect raw per-hour means (no Gaussian smoothing).
- Tooltip includes Δ at +3 h, Δ at +4 h, meal count, and average **ICR (g/U)** for that hour.

---

## License

This project is licensed under the **GNU Affero General Public License v3.0 (AGPL-3.0)**.

You are free to use, modify, and distribute this software, provided that any modified version that is made available over a network also makes its source code available under the same licence. See [https://www.gnu.org/licenses/agpl-3.0.html](https://www.gnu.org/licenses/agpl-3.0.html) for the full licence text.

---

## Acknowledgements

Built with [Chart.js](https://www.chartjs.org/), [JSZip](https://stuk.github.io/jszip/), and the [Anthropic API](https://www.anthropic.com/). Designed for use with [Glooko](https://www.glooko.com/) exports.
