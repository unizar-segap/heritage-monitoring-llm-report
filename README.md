# LLM-based Heritage Monitoring Report

[![DOI](https://zenodo.org/badge/DOI/10.5281/zenodo.24526424.svg)](https://doi.org/10.5281/zenodo.24526424)

Prompt design and statistical code that transform instrumental monitoring data from built heritage assets into a weekly monitoring report written by a Large Language Model (LLM).

The module calculates descriptive statistics for each sensor and provides them to the LLM as a structured JSON data block. Strict instructions ensure that the report remains grounded in the data: the model cannot fabricate values, distinguishes observations from interpretation, does not issue structural safety assessments, and concludes with a predefined recommendation label.

The repository contains the code that generates the *prompts* for calculating sensor-level statistics, their serialisation into a data block, and the instructions that guide the LLM in drafting the report. Its purpose is to document and make the *prompt* design and statistical methodology citable, rather than to provide an executable application. Therefore, it does not include the data models (`visualization.models`) or the LLM clients (`visualization.chatbot`), which are part of the project's full platform. The dependencies required by the module are described in the [Usage](#usage) section.

## Features

1. **Calculates statistics** for each sensor associated with a heritage asset:
   - Historical mean and standard deviation (σ) across all stored readings.
   - Weekly mean, minimum and maximum values.
   - Statistical deviations: weekly readings where |x − mean| > 3σ.
2. **Builds the prompt**: a system instruction containing mandatory rules (data fidelity, scope limitations, statistical interpretation, data quality and regulatory framework) and a user *prompt* or instruction containing the JSON data block and a fixed seven-section output format.
3. **Calls the LLM** (Gemini or Ollama, temperature 0.2) and caches a summary for each heritage asset and week.

The reports are written in Spanish and are intended for heritage conservation professionals.

## Supported sensors

| Sensor type    | Measured quantities                     |
|----------------|-----------------------------------------|
| `6F08-OT-D100` | X/Y tilt, distance, temperature         |
| `6306-EO-Tilt` | X/Y/Z tilt, temperature                 |
| `650B-Air-TH`  | Air temperature, air humidity           |
| `6A07-1xVW`    | Frequency, resistance, temperature     |

Internal sensor temperature is summarised but is not assessed for statistical deviations.

## Regulatory framework

The prompt references the following standards solely for general guidance. The generated reports do not claim compliance with any of them:

- [UNE-EN 16096:2016](https://www.une.org/encuentra-tu-norma/busca-tu-norma/norma?c=N0056786): condition survey and assessment of built cultural heritage.
- [ISO 13822:2010](https://www.iso.org/es/contents/data/standard/04/65/46556.html): assessment of existing structures (Annex I, heritage structures).
- [ISO 7870-2:2013](https://www.iso.org/standard/40174.html): Shewhart control charts.

## Usage

`prompt.py` is part of a Django application and depends on the following platform components, which are not included in this repository:

- The `Item`, `LeicaTiltReading`, `LeicaTemperatureHumidityReading`, `LeicaPiezometerReading` and `MonitoringSummary` models.
- The `CHATBOT_BACKEND` (`"ollama"` or Gemini) and `TIME_ZONE` settings.
- The `chat_with_gemini` / `chat_with_ollama` functions, which accept `system_prompt_override` and `temperature`.

```python
from prompt import build_monitoring_prompt, generate_monitoring_summary

# Inspect the prompt.
system_instruction, user_prompt = build_monitoring_prompt(item_id)
# Generate the prompt and save it in cache.
result = generate_monitoring_summary(item_id)
```

## Disclaimer

The generated reports are automated, data-driven support documents. They do not replace condition surveys or structural assessments carried out by qualified professionals.

## Citation

If you use this code, please cite it using its Zenodo DOI (see the badge at the top of this page or the `CITATION.cff` file).

## Licence

See [LICENSE](LICENSE).
