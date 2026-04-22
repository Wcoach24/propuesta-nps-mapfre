# Propuesta · NPS Predictivo en Contact Center

**Cliente:** Mapfre España
**Referencia pliego:** LSALBER-Data-NPS Predictivo · Doc. 3068906220
**Presentado por:** Capgemini Invent · Data & AI · Iberia
**Fecha:** Abril 2026
**Versión:** v0.2 · borrador para validación interna

> Para una vista renderizada con estilo, abre [`index.html`](./index.html) (descárgalo y ábrelo en el navegador, o habilita GitHub Pages en Settings → Pages).

---

## Índice

1. [Resumen ejecutivo](#01--resumen-ejecutivo)
2. [Entendimiento del reto y propuesta de valor](#02--entendimiento-del-reto-y-propuesta-de-valor)
3. [Enfoque técnico y arquitectura](#03--enfoque-técnico-y-arquitectura)
4. [Roadmap, entregables y aceptación](#04--roadmap-entregables-y-aceptación)
5. [Operación y mantenimiento post-entrega](#05--operación-y-mantenimiento-post-entrega)
6. [Equipo, gobernanza y riesgos](#06--equipo-gobernanza-y-riesgos)
7. [Criterios de éxito](#07--criterios-de-éxito)
8. [Inversión y condiciones contractuales](#08--inversión-y-condiciones-contractuales)
9. [Próximos pasos](#09--próximos-pasos)
10. [Anexos](#10--anexos)

---

## 01 · Resumen ejecutivo

Capgemini Invent propone diseñar, construir y transferir a Mapfre España un agente de IA que infiere el NPS transaccional de cada llamada del Contact Center de Prestaciones Hogar, a partir de su transcripción y de los metadatos operativos asociados, en un plazo de **8 semanas con entrega final el 30 de junio de 2026**.

El sistema de medición de NPS actual de Mapfre cubre el **5–8% de las llamadas** vía encuesta voluntaria. Eso significa que el 92% de los clientes —y casi todos los detractores silenciosos— quedan invisibles. La propuesta no es mejorar la encuesta. Es construir el sistema que permite saber qué siente el otro 92% de clientes con datos que Mapfre ya tiene.

### Propuesta de valor en cuatro puntos

- **Arquitectura híbrida, no caja negra.** El LLM extrae señales semánticas estructuradas; un modelo supervisado clásico (XGBoost) las combina con metadatos operativos para predecir el NPS. Cada predicción es trazable y explicable matemáticamente con SHAP. Diseñado para auditoría EU AI Act desde el primer día.
- **Coste de operación controlado.** El LLM se ejecuta solo en la fase de extracción de señales, en batch nocturno. La predicción posterior es prácticamente gratuita. Coste estimado: **~5.400 €/año** en tokens para procesar las 300K llamadas de Prestaciones Hogar.
- **Land-and-expand desde el diseño.** El mismo pipeline escala a Auto, Multirriesgo y resto del Contact Center con coste lineal predecible.
- **Mapfre opera en autonomía a la entrega.** Repositorio de código, model card, runbook operativo, dataset etiquetado y dos sesiones de transferencia grabadas.

### Cifras clave

| Concepto | Valor |
|---|---|
| Volumen a procesar | 300.000 llamadas/año · servicio Prestaciones Hogar |
| Plazo de ejecución | 8 semanas desde adjudicación · entrega 30/06/2026 |
| Fases | 3 fases solapadas con 2 hitos gating intermedios |
| Coste operativo post-entrega | ~5.400 €/año en tokens LLM (escala lineal) |
| Inversión proyecto | Rango orientativo 80–95 K€ · precio cerrado · pago por hitos · *[a confirmar tras escandallo oficial Capgemini]* |
| Compliance | Diseñado AI Act-ready · vigencia plena 02/08/2026 (33 días tras entrega) |

---

## 02 · Entendimiento del reto y propuesta de valor

### El reto operativo de Mapfre

Mapfre España opera un Contact Center (C@C24) que gestiona aproximadamente **1,37 millones de llamadas/mes**, con NICE como sistema de grabación y transcripción. La calidad percibida del servicio se mide hoy mediante encuesta NPS transaccional, con cuatro limitaciones inherentes:

- **Cobertura limitada.** Solo el 5–8% responde la encuesta. El 92–95% restante es estadísticamente invisible.
- **Sesgo del detractor silencioso.** Los más insatisfechos son los que menos responden. La encuesta sobreestima el NPS real.
- **Latencia operativa.** La respuesta llega días después, cuando la ventana de recovery se ha cerrado.
- **Granularidad agregada.** El NPS da una temperatura global, no identifica palancas operativas concretas.

> El sistema actual funciona para reportar. No funciona para accionar.

### Lo que el pliego pide

El RFP acota el alcance con precisión: **NPS predictivo** —estimación a partir de transcripciones y metadatos—, descartando explícitamente en esta fase el "índice de satisfacción inteligente" más amplio. Traducido operativamente:

1. Un agente IA que, dada una llamada {transcripción + metadatos}, devuelva NPS estimado, clasificación detractor/pasivo/promotor y la explicación de la predicción.
2. Una metodología reproducible y documentada que Mapfre pueda aplicar en autonomía tras la entrega.
3. Una taxonomía de pain points construida con datos reales de Prestaciones Hogar, accionable por Calidad y Operaciones.

### Por qué Capgemini Invent

- **Arquitectura híbrida probada** en banca y seguros bajo regulación europea.
- **Precedente directo en Mapfre.** El stack AWS Bedrock con Claude está aprobado por la DPO en MIA GPT. *[A confirmar con Alberto/Javier]*
- **Equipo senior compacto.** Un único punto focal técnico ante Mapfre, conforme al pliego.
- **Entregables de transferencia reales.** Mapfre opera el día 1 post-cierre sin nosotros.
- **Diseño AI Act-ready desde S1** con plantillas validadas Capgemini.

---

## 03 · Enfoque técnico y arquitectura

### El principio de diseño · separar percepción de decisión

El LLM hace lo que sabe hacer bien — leer una conversación humana en español y describirla en variables estructuradas con quotes literales. Un modelo supervisado clásico hace lo que sabe hacer bien — combinar esas variables con metadatos para predecir el NPS de forma determinista, auditable y reentrenable.

### Por qué híbrido y no LLM directo

| Dimensión | LLM directo | Arquitectura híbrida | Impacto Mapfre |
|---|---|---|---|
| Coste anual | ~13.500 €/año | ~5.400 €/año | ~8.000 € ahorro |
| Determinismo | Variable entre runs | 100% determinista | Auditable |
| Aprendizaje específico | No aprende | XGBoost aprende | Mayor precisión |
| Explicabilidad | Texto generado | SHAP matemático | Defendible AI Act |
| Cambio proveedor LLM | Lock-in | Desacoplado | Optionality |

### Arquitectura en cuatro capas

| Capa | Responsabilidad | Componente |
|---|---|---|
| 01 · Ingesta | Normalización, deduplicación, anonimización PII | Microsoft Presidio + AWS Comprehend |
| 02 · Extracción LLM | JSON con 12 señales + quotes literales | Claude Sonnet 4.6 · AWS Bedrock (EU Frankfurt) |
| 03 · Modelo predictivo | NPS + clase + SHAP | XGBoost + SHAP + scikit-learn + MLflow |
| 04 · Insights | Taxonomía + dashboard | Power BI sobre S3 vía Athena |

### Las 12 señales semánticas

1. Sentimiento global del cliente (float [-1, +1])
2. Esfuerzo percibido CES implícito (float [0, 1])
3. Empatía mostrada por el agente (float [0, 1])
4. Repetición del mismo problema (int)
5. Fricción operativa percibida (float [0, 1])
6. Resolución en primera llamada FCR (bool)
7. Claridad de la respuesta del agente (float [0, 1])
8. Tono cliente al cierre (enum: calmado / frustrado / indignado / satisfecho)
9. Mención de competencia (bool)
10. Intención implícita de cancelación (float [0, 1])
11. Categoría de pain point dominante (enum ≥ 8 categorías)
12. Severidad del pain point dominante (enum: bajo / medio / alto / crítico)

### Ejemplo de explicación SHAP

```
Llamada #47821 → Predicción NPS = 3 (Detractor, prob 0.87)

Base value (NPS medio histórico):    7.2
+ Esfuerzo percibido alto:          -1.8
+ 3 transferencias entre agentes:   -1.2
+ Repetición del problema (2x):     -0.9
- Empatía del agente alta:          +0.4
- Resolución en primera llamada:    +0.3
─────────────────────────────────────
Predicción final:                    4.0 → Clase: Detractor
```

### Stack tecnológico

**Stack A · AWS Bedrock (principal):** Claude Sonnet 4.6 · SageMaker · S3 cifrado KMS · Presidio + Comprehend · XGBoost + SHAP · MLflow · Evidently + CloudWatch · Power BI sobre Athena · Model card Google + DPA AWS + CloudTrail.

**Stack B · Azure OpenAI (alternativa):** GPT-5.4 en Microsoft Foundry + Azure ML + ADLS + Power BI sobre Synapse. Migración entre stacks: ~1 semana. Decisión en Hito 01.

---

## 04 · Roadmap, entregables y aceptación

### Calendario maestro (adjudicación primera semana mayo 2026)

| Sem. | Fechas | Fase | Actividad | Hito |
|---|---|---|---|---|
| S1 | 05–11 may | I | Kick-off · accesos · muestra 100 llamadas | Kick-off cerrado |
| S2 | 12–18 may | I | Entendimiento datos · anotación ground truth | |
| S3 | 19–25 may | I→II | Cierre diseño · variables cerradas · prompt v1 | **H1 · Diseño aprobado** |
| S4 | 26 may–01 jun | II | Pipeline · prompt engineering · anotación | |
| S5 | 02–08 jun | II | Modelo XGBoost · MVP end-to-end | |
| S6 | 09–15 jun | II→III | Validación · calibración Platt | **H2 · Modelo validado** |
| S7 | 16–22 jun | III | Taxonomía · insights · dashboard v1 | |
| S8 | 23–30 jun | III | Cierre · model card · runbook · transferencia | **H3 · Aceptación** |

### Transferencia · seis entregables

1. **Repositorio de código** · pipeline, tests, Terraform
2. **Model card** · estándar Google
3. **Runbook operativo** · reentrenamiento, drift, prompts
4. **Dataset etiquetado** · 1.500–2.000 llamadas ground truth
5. **Sesiones de transferencia grabadas** · técnica (4h) + negocio (2h)
6. **Plan de evolución** · extensión a Auto, Multirriesgo, resto CC

---

## 05 · Operación y mantenimiento post-entrega

### Funcionamiento

Pipeline **batch nocturno**, no tiempo real. Ingesta → Anonimización → Extracción LLM → Predicción → Persistencia → Visualización. Resultados disponibles a primera hora del día siguiente.

### Coste operativo

| Cobertura | Volumen/año | Coste LLM/año |
|---|---|---|
| Prestaciones Hogar (piloto) | ~300.000 | ~5.400 € |
| Hogar + Auto + Multirriesgo | ~900.000 | ~16.000 € |
| Contact Center completo | ~16 M | ~290.000 € |

### Palancas de control

- Procesamiento batch (descuento 50%)
- Filtrado pre-LLM (<60s descartadas, ~15% adicional)
- Muestreo configurable (50–75% manteniendo validez estadística)

### Independencia tecnológica

Abstracción del proveedor LLM. Cambiar a GPT-5, Llama 4 o Mistral requiere modificar configuración. Migración: ~1 semana de un Data Scientist Mapfre.

### Mantenimiento en autonomía

| Actividad | Frecuencia | Esfuerzo | Responsable |
|---|---|---|---|
| Monitorización dashboard | Diaria | Automática | Calidad / Ops |
| Métricas de drift | Mensual | ~2h | Data Scientist |
| Reentrenamiento XGBoost | Cada 3–6 meses | ~1 día | Data Scientist |
| Actualización prompt | Según necesidad | ~1 día | Data Scientist |
| Auditoría AI Act | Anual | ~3 días | Data & IA + Compliance |

### Fuera de alcance

- UI propia distinta al dashboard Power BI
- Integración tiempo real en flujo CC
- Alertas automáticas durante llamada
- API expuesta para otras aplicaciones
- Servicios distintos de Prestaciones Hogar

---

## 06 · Equipo, gobernanza y riesgos

### Equipo (~67 días totales)

| Rol | Perfil | Dedicación | Esfuerzo |
|---|---|---|---|
| Responsable técnico · punto focal | Senior Consultant | 100% · 8 sem | 40 días |
| Apoyo modelado | Data Scientist | 25–35% · S4–S6 | 12 días |
| Anotación ground truth | Annotator | 100% · S2–S4 | 10 días |
| QA, governance, sponsor | Manager / Senior Manager | 10% · 8 sem | 5 días |

### Gobernanza

- **Daily asíncrona** (<15 min) · equipo Capgemini + Data Mapfre
- **Weekly** (30 min) · Senior Consultant + interlocutor único
- **Comité quincenal** (60 min) · sponsors + Manager + responsables
- **Validación hito gating** (90 min × 3) · Comité ampliado + Calidad/Ops + DPO

### Riesgos principales

| # | Riesgo | Prob. | Impacto | Mitigación |
|---|---|---|---|---|
| R1 | NPS real vinculado inexistente | Alta | Media | Plan A (encuesta) / Plan B (anotación manual, Kappa ≥ 0,70) |
| R2 | Calendario apretado 8 semanas | Alta | Alta | Fases solapadas · ground truth desde S2 · contingencia 15% |
| R3 | LOPDGDD · PII en transcripciones | Alta | Media | Anonimización previa · EU Frankfurt · DPA AWS · DPO S1 |
| R4 | Sin criterios cuantitativos en pliego | Media | Alta | Umbrales propuestos en §07 · firmados en H1 |
| R5 | EU AI Act vigente 02/08/2026 | Media | Alta | Documentación alto riesgo desde S1 · plantillas Capgemini |
| R6 | Calidad heterogénea NICE | Media | Media | QC en ingesta · descarte WER alto |
| R7 | Desalineación con Ops (Pablo) | Media | Media | Weekly con Ops desde S5 · validación taxonomía |
| R8 | Sesgos por segmento | Media | Media | Análisis fairness en H2 · ajuste si disparidades |

### Plan de contingencia

- **(a) Todos los umbrales superados** → aceptación formal en 10 días
- **(b) Mayoría superada, alguno al 85–95%** → aceptación con reserva + plan de mejora sin coste 4 semanas
- **(c) Significativamente por debajo** → análisis causas raíz + medidas correctoras (ampliación ground truth, simplificación target, change order)

---

## 07 · Criterios de éxito

| Métrica | Umbral | Por qué |
|---|---|---|
| F1-score · Detractor | ≥ 0,75 | Métrica operativamente más relevante. Realista con transcripciones reales |
| MAE · NPS predicho vs real | ≤ 1,5 | Ranking por NPS útil para priorización |
| Kappa de Cohen · ground truth | ≥ 0,70 | Acuerdo entre anotadores: validez del entrenamiento |
| Trazabilidad por predicción | 100% | SHAP + quotes literales · exigible AI Act |
| Pain points en taxonomía | ≥ 8 | Granularidad mínima accionable |
| Sesiones transferencia | 2 | Técnica (4h) + negocio (2h) grabadas |

### Aceptación formal

1. Modelo supera umbrales acordados en H1
2. Taxonomía validada por Comité
3. Seis entregables en infraestructura Mapfre
4. Dos sesiones de transferencia impartidas
5. Aceptación escrita en 10 días laborables (o tácita)

---

## 08 · Inversión y condiciones contractuales

### Modalidad y rango

**Precio cerrado por entregables.** **80–95 K€** · IVA no incluido · *[a confirmar tras escandallo oficial]*

### Desglose orientativo

| Concepto | Perfil | Días |
|---|---|---|
| Responsable técnico | Senior Consultant | 40 |
| Apoyo modelado | Data Scientist | 12 |
| Anotación | Annotator | 10 |
| QA, governance | Manager / Senior Manager | 5 |
| Infra, licencias, tokens proyecto | — | ~3.500 € |
| **Total** | | **~67 días · 80–95 K€** |

### Formatos alternativos

| Formato | Rango | Incluye |
|---|---|---|
| Básico | ~-20% sobre Estándar | Fases I-II · Fase III simplificada |
| **Estándar (recomendado)** | 80–95 K€ | Alcance completo |
| Ampliado | ~+30% sobre Estándar | Estándar + hyper-care 4 sem + 2 servicios piloto |

### Plan de pagos

| Hito | % | Momento | Condición |
|---|---|---|---|
| Kick-off | 30% | S1 | Firma contrato |
| Cierre Fase II | 40% | S6 | Aprobación H2 |
| Aceptación final | 30% | S8 + 10 días | Aceptación escrita o tácita |

### Condiciones

- **Validez oferta:** 60 días
- **Facturación:** Ariba / Factura Digital Mapfre
- **Cambios de alcance:** change order con respuesta 48h
- **IP:** modelo, dataset, código y documentación son propiedad Mapfre tras pago final. Metodología/frameworks genéricos son Capgemini con licencia perpetua, no exclusiva y libre de royalties
- **Confidencialidad:** Anexo A (NDA)
- **Responsabilidad:** limitada al importe del contrato

### Lo que NO incluye el precio

- Procesamiento masivo producción (~5.400 €/año AWS)
- Integración tiempo real en CC
- Servicios distintos de Prestaciones Hogar
- Tokens LLM si exceden 500 € durante desarrollo

---

## 09 · Próximos pasos

| Paso | Plazo | Acción |
|---|---|---|
| 01 · Anexo C dudas | Esta semana | Sesión 60 min con Data & IA |
| 02 · Adjudicación | Próximos 10 días | NDA · accesos · interlocutor único |
| 03 · Kick-off | Semana 5 mayo | Presencial Madrid · firma H1 antes fin S3 |

### Las 5 preguntas críticas pendientes

1. ¿Existe NPS real (encuesta) vinculado a un subset de las 300K llamadas?
2. ¿Plataforma LLM aprobada en tenant Mapfre: AWS Bedrock, Azure OpenAI o ambas?
3. ¿NICE C@C24 transcribe el 100% de Prestaciones Hogar o un subset?
4. ¿Criterios cuantitativos predefinidos o se fijan en H1?
5. ¿Fecha estimada de adjudicación?

---

## 10 · Anexos

- **Anexo A** · NDA firmado según plantilla Mapfre
- **Anexo B** · Propuesta económica detallada (separado, confidencial)
- **Anexo C** · Plantilla de dudas
- **Anexo D** · CV Senior Consultant
- **Anexo E** · Credenciales Capgemini Invent sector asegurador
- **Anexo F** · FAQ anticipadas (abajo)

### Anexo F · FAQ anticipadas

**1 · ¿Por qué no pedir al LLM el NPS directamente?**
Técnicamente posible pero compromete cinco aspectos críticos: coste ×2,5 (13.500 € vs 5.400 €/año), no determinismo, no aprendizaje específico, explicabilidad no matemática, lock-in. Comparativa en §03.

**2 · ¿Cuánto costará operar anualmente?**
~5.400 €/año para Prestaciones Hogar. Escala lineal: ~16.000 € Hogar+Auto+Multirriesgo · ~290.000 € CC completo.

**3 · ¿Qué tratamiento reciben los datos personales?**
Anonimización total antes del LLM (Presidio + Comprehend en infra Mapfre). PII nunca sale del tenant. EU Frankfurt bajo DPA AWS.

**4 · ¿Podemos cambiar de proveedor LLM?**
Sí. Pipeline con abstracción. Migración GPT-5 / Llama 4 / Mistral: ~1 semana de un Data Scientist.

**5 · ¿Análisis en tiempo real?**
No. Batch nocturno, resultados primera hora del día siguiente. Tiempo real = extensión posterior, fuera del alcance.

**6 · ¿Y si el modelo no alcanza los umbrales?**
Plan de contingencia con 3 escenarios (§06 y §07).

---

*Capgemini Invent · Data & AI · Iberia · Doc. CI-INV-2026-NPS-001 · v0.2 · abril 2026*
