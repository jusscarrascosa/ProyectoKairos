# KAIROS Guardian — Simulación y Resiliencia para plantas *off‑grid* con Gemelo Digital

> **Contexto:** Propuesta finalista del **1er Hackatón Industrial del mundo**. Objetivo: **“salvar KAIROS”** — estabilizar una planta *off‑grid* ante fallas, eventos tardíos y desalineación del **Gemelo Digital** con la operación real.

Este repositorio (o carpeta de trabajo) contiene una **simulación ejecutable en Jupyter** que integra:
- **Sensores** con ruido/sesgo y **filtros de plausibilidad** (`Sensor`, `PlausibilityFilter`).
- **Nodo de Integración** (`IntegrationNode`) que unifica señales reales/sintéticas, corrige tiempos y aplica limpieza.
- **Buffer por watermark** para **llegadas tardías y reordenamientos** (`HybridWatermarkBuffer`).
- **Pipeline con reloj** (`ClockedPipeline`): etapas **IF → ID → EX → MEM → WB** con métricas de calidad (skew, drift, bubbles, lateness).
- **Gemelo Digital (GD)** y **Gemelo Sombra (GS)** con *scikit‑learn*.
- **Guardian**: auditor/árbitro que combina confianza de GD, **desacuerdo GD/GS** y **métricas del pipeline** para decidir **Control Predictivo** vs **Degrade/Human/Safety**.
- **Procesos energéticos simplificados**: `SolarPanel`, `WindTurbine`, `Battery` con eficiencias de carga/descarga y demanda.

La implementación principal está en el notebook **`Simulación_ (10).ipynb`**.


## Arquitectura (resumen)
1. **Planta real (sintética)** → sensores térmicos/IR/meteorol., generación solar/eólica y **batería**.
2. **`IntegrationNode`** aplica **plausibility filter**, **corrección de timestamp** y empaqueta un **`packet`** por *tick*.
3. **`HybridWatermarkBuffer`** espera lo suficiente (watermark) para **liberar en orden** y tolerar **lateness**.
4. **`ClockedPipeline`** procesa el *packet* por etapas IF/ID/EX/MEM/WB y exporta métricas:
   - `skew_avg`, `drift`, `bubble_ratio`, `lateness_avg`.
5. **`Gemelo` (GD/GS)** predicen sobre el *packet* y **`Guardian.evaluar(...)`** decide:
   - **OK (Control Predictivo)** si confianza alta, cobertura y bajo desacuerdo GD/GS.
   - **Degrade / Human‑in‑the‑Loop / Safety** en caso contrario.
6. **KPIs** (resumen de sesión): `MTTR_s`, `state_ratios`, `valid_rate`, `hash_integrity` (*vía `resiliencia_kairos`*).

> El diagrama de bloques que acompaña la publicación de LinkedIn refleja exactamente este flujo: Planta → Nodo de Integración → Pipeline → GD/GS → Guardian → Gate/Actuadores.


## Qué hay en el código
Clases y funciones principales detectadas en el notebook:
- `Sensor`, `PlausibilityFilter` — sensado con ruido/sesgo y validación por rango.
- `SolarPanel`, `WindTurbine`, `Battery` — fuentes y almacenamiento básicos para el **balance energético**.
- `IntegrationNode.read_step(...)` — lectura unificada (sensor + energía) con **filtros de plausibilidad**.
- `HybridWatermarkBuffer` — buffer con watermark para **lateness y reordenamiento**.
- `ClockedPipeline` — pasos **IF/ID/EX/MEM/WB** y métricas.
- `Gemelo` — envoltorio de modelos **sklearn** (ej.: `RandomForestClassifier` y `LogisticRegression`).
- `Guardian.evaluar(...)` — decisión combinando **confianza**, **ECE/cobertura** (si están), **desacuerdo GD/GS** y métricas del pipeline.
- **KPIs**: `kpi_log_step`, `kpi_resumen()` → *requieren utilidades de* `resiliencia_kairos`.

Parámetros clave que podés ajustar en el notebook:
- `N` (pasos de simulación, p.ej. `N = 200`).
- `late_prob` (probabilidad de llegada tardía).
- `device_drift` (deriva del reloj del dispositivo).
- `WATERMARK_DELAY`, `MAX_LATE` (config del watermark).
- Rango de **plausibilidad** por sensor (`IntegrationNode.plausibility_filters`).
- Eficiencias energéticas `eta_carga`, `eta_descarga` y `demanda_kw` en cada `read_step(...)`.


