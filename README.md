# Informe de monitorización patrimonial con LLM

[![DOI](https://zenodo.org/badge/DOI/10.5281/zenodo.XXXXXXX.svg)](https://doi.org/10.5281/zenodo.XXXXXXX)

Diseño de prompt y código estadístico que convierte los datos de monitorización instrumental de bienes del patrimonio construido en una nota semanal de monitorización redactada por un LLM.

El módulo calcula estadísticos descriptivos por sensor y los entrega al LLM como un bloque JSON estructurado. Se utilizan instrucciones estrictas que mantienen el informe ceñido a los datos: el modelo no puede inventar valores, separa las observaciones de la interpretación, no emite dictámenes de seguridad estructural y termina con una etiqueta de recomendación fija.

El repositorio contiene el código que genera los *prompts* para el cálculo de los estadísticos por sensor, su serialización como bloque de datos y las instrucciones que guían la redacción del informe por el LLM. Su objetivo es documentar y hacer citable el diseño del *prompt* y el método estadístico, no ofrecer una aplicación ejecutable. Por ello no incluye los modelos de datos (`visualization.models`) ni los clientes de los LLM (`visualization.chatbot`), que forman parte de la plataforma completa del proyecto. Las dependencias que el módulo espera se describen en el apartado [Uso](#uso).

## Funciones

1. **Calcula estadísticos** para cada sensor asociado a un bien:
   - Media y desviación estándar (σ) históricas sobre todas las lecturas almacenadas.
   - Media, mínimo y máximo semanales.
   - Desviaciones estadísticas: lecturas semanales con |x − media| > 3σ.
2. **Construye el prompt**: una instrucción de sistema con reglas obligatorias (fidelidad a los datos, límites de alcance, interpretación estadística, calidad de los datos y marco normativo) y un *prompt* o instrucción de usuario con el bloque de datos JSON y un formato de salida fijo de siete apartados.
3. **Llama al LLM** (Gemini u Ollama, temperatura 0,2) y guarda en caché un resumen por bien y semana.

Los informes se redactan en español y están dirigidos a responsables de conservación del patrimonio.

## Sensores soportados

| Tipo de sensor | Magnitudes                                   |
|----------------|----------------------------------------------|
| `6F08-OT-D100` | Inclinación X/Y, distancia, temperatura      |
| `6306-EO-Tilt` | Inclinación X/Y/Z, temperatura               |
| `650B-Air-TH`  | Temperatura del aire, humedad del aire       |
| `6A07-1xVW`    | Frecuencia, resistencia, temperatura         |

La temperatura interna de los sensores se resume, pero no se evalúa como desviación.

## Marco normativo

El prompt cita las siguientes normas solo como referencia general. Las notas generadas no declaran conformidad con ninguna de ellas:

- [UNE-EN 16096:2016](https://www.une.org/encuentra-tu-norma/busca-tu-norma/norma?c=N0056786): inspección y evaluación del estado del patrimonio construido.
- [ISO 13822:2010](https://www.iso.org/es/contents/data/standard/04/65/46556.html): evaluación de estructuras existentes (Anexo I, estructuras patrimoniales).
- [ISO 7870-2:2013](https://www.iso.org/standard/40174.html): gráficos de control de Shewhart.

## Uso

`prompt.py` forma parte de una aplicación Django y depende de los siguientes elementos de la plataforma, no incluidos en este repositorio:

- Los modelos `Item`, `LeicaTiltReading`, `LeicaTemperatureHumidityReading`, `LeicaPiezometerReading` y `MonitoringSummary`.
- Los ajustes `CHATBOT_BACKEND` (`"ollama"` o Gemini) y `TIME_ZONE`.
- Las funciones `chat_with_gemini` / `chat_with_ollama`, que aceptan `system_prompt_override` y `temperature`.

```python
from prompt import build_monitoring_prompt, generate_monitoring_summary

# Inspect the prompt.
system_instruction, user_prompt = build_monitoring_prompt(item_id)
# Generate the prompt and save it in cache.
result = generate_monitoring_summary(item_id)
```

## Aviso

Las notas generadas son documentos de apoyo automáticos basados en los datos. No sustituyen una inspección del estado ni una evaluación estructural realizadas por personal cualificado.

## Cita

Si utilizas este código, cítalo mediante su DOI de Zenodo (consulta el distintivo de la parte superior o el archivo `CITATION.cff`).

## Licencia

Consulta [LICENSE](LICENSE).
