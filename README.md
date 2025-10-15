
# KAIROS Guardian — Resiliencia para plantas off-grid con Gemelo Digital

Solución open-source basada en **pipeline de datos**, **Gemelo Sombra + Gemelo Digital** y **Guardián** con métricas de confianza para operar en **modo seguro**, reducir **downtime** y extender la **vida útil** de equipos.

> Este repo acompaña la propuesta presentada por el **Equipo Algarroba LID** en el **1er Hackatón Industrial del mundo**.

![Arquitectura](docs/diagramas/arquitectura-kairos.png)

## ¿Qué incluye?
- **Sensores (OT):** filtros de plausibilidad, redundancia (termocupla + IR), resincronización temporal y trazabilidad.
- **Pipeline (IF → ID → EX → MEM/StateLog → WB):** ingesta, normalización, features, bitácora y “write-back” de flags de calidad.
- **Twins:** Digital y **Shadow** en paralelo para detectar inconsistencias.
- **Guardián:** decide según **Métricas de Confianza** → *Degrade/Human/Safety* o *Control Predictivo + Gate*.
- **Procesos energéticos:** SoC/SoH, buffer con **supercapacitores**, **PMUs** y pronóstico meteo para transiciones suaves.

## Quickstart
```bash
python -m venv .venv && source .venv/bin/activate   # (Windows: .venv\Scripts\activate)
pip install -r requirements.txt
python demo/run_demo.py --data data/samples/sensors.csv
