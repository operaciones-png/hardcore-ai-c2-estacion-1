# Internal Solution Brief — Hardcore AI Cohorte 2

## OPCIÓN A — Célula AI Green Power — Fase 1: Copiloto AI de Pre-Causación Financiera con Humano-en-el-Loop

> **Nota del autor (Esteban):** Este brief re-encuadra el problema original ("implementar un área de AI que acompañe operaciones, finanzas y yacimientos") en un MVP de 4 semanas defendible, con roadmap multi-fase para los demás frentes. La razón es práctica: el template Hardcore AI mide entregable a 4 semanas; abarcar simultáneamente operaciones de campo + finanzas + yacimientos requiere equipo multidisciplinar y datasets que en este momento no están todos disponibles ni curados.
>
> **Particularidad de la Opción A:** el sponsor técnico es el Gerente de Operaciones y los responsables del MVP son ingenieros (no contadores). El caso de uso es financiero. Esta configuración cross-funcional cuenta con el respaldo explícito del dueño de la empresa y la colaboración activa del área financiera. La tensión natural entre dominios técnico y back-office se mitiga con un **comité de validación semanal** (Operaciones + Contador titular + Revisor Fiscal + Dueño) y con la decisión de diseño de **mantener al equipo financiero como aprobador formal de cada causación generada por AI**, no como dependencia reemplazada.
>
> **Disclaimer técnico-legal:** El MVP aquí descrito NO es un sistema contable certificado ante DIAN. Es una herramienta de productividad que pre-llena propuestas de causación, las cuales son revisadas, aprobadas y firmadas por el contador titular. La fe pública contable y la responsabilidad tributaria recaen en los actores legalmente designados (contador, revisor fiscal, representante legal), no en el sistema AI ni en el sponsor de Operaciones.

---

## SOLUCIÓN

**Nombre de la solución:** Copiloto AI de Pre-Causación Financiera — Green Power Fase 1

**Descripción en una línea:** Asistente de AI que extrae datos de facturas de proveedores (XML UBL primario, PDF/imagen como fallback), pre-llena la propuesta de causación en el formato Excel contable de Green Power, y la entrega al equipo financiero para validación y firma del contador titular.

**Empresa / Organización:** Green Power S.A.S. — exploración y explotación de hidrocarburos, bloques activos en los Llanos Orientales (Colombia), ~90 empleados.

---

## 1. PROBLEMA DE NEGOCIO

### Descripción del problema

El proceso de causación de facturas de proveedores y de elaboración de los 3 reportes financieros mensuales recurrentes en Green Power se ejecuta hoy de forma 100% manual sobre Excel, sin ERP. Tres personas dedican el grueso de su jornada a digitación, validación cruzada de NIT/IVA/retenciones y consolidación de reportes, lo que genera:

- **Cuello de botella en cierre mensual:** los reportes financieros se entregan tarde y con retrabajo recurrente porque la causación no termina a tiempo.
- **Riesgo tributario por errores humanos:** digitación manual de NIT, valores, retenciones por municipio (Yopal, Aguazul, Tauramena) y clasificación OPEX/CAPEX por centro de costo por pozo. Cada error puede derivar en sanciones DIAN (art. 647 ET inexactitud 100%, art. 651 ET hasta $392M COP).
- **Costo de oportunidad del talento:** las 3 personas dedican el ~70% de su tiempo a tareas de bajo valor agregado (digitación) en lugar de actividades de alto valor (control tributario, auditoría de retenciones, soporte a operaciones).
- **Falta de trazabilidad y reproducibilidad:** los Excel no garantizan reproducción fidedigna a 10 años conforme al art. 28 de la Ley 962 de 2005 ni al Concepto CTCP 700/2021.

### Cuantificación del impacto

| Métrica actual | Valor estimado |
|---|---|
| Volumen de facturas procesadas/mes | 50–60 |
| Reportes financieros mensuales recurrentes | 3 |
| Personas dedicadas | 3 FTE |
| Costo escondido del proceso manual (benchmark APQC/Ardent: USD 10–13/factura para empresas sin ERP) | **COP 2.4M–3.1M/mes** (≈ COP 28M–37M/año) |
| % del tiempo de las 3 personas en tareas repetitivas | ~70% |
| Días de retraso típico en cierre mensual | 5–10 días después del corte |
| Errores materiales detectados retroactivamente | No medido formalmente (gap de información identificado) |

### Timeline / urgencia

- **Inmediata:** la carga operativa actual ya genera demoras en reportería al dueño y al contador.
- **A 6 meses:** crecimiento esperado de operación + nuevos contratos = más facturas sin posibilidad de escalar el equipo proporcionalmente.
- **A 12 meses:** ventana competitiva para que Green Power sea uno de los primeros casos documentados de E&P mediano colombiano con Célula AI en finanzas — relevante para posicionamiento gremial (ACP, Campetrol).

---

## 2. STAKEHOLDERS Y SPONSOR

### Sponsor ejecutivo
- **Dueño / Gerente General de Green Power** — respaldo explícito al proyecto y al presupuesto del MVP.

### Sponsor técnico / líder del proyecto
- **Juanda — Gerente de Operaciones** (ingeniero mecánico). Responsable de la ejecución técnica del MVP, articulación con n8n y vendor management.

### Stakeholders críticos (deben aprobar formalmente antes de producción)
- **Contador titular de Green Power** — único responsable legal de la firma con tarjeta profesional (Ley 43/1990 art. 10). Sin su aprobación documentada, el proyecto no avanza a producción.
- **Revisor fiscal** (si aplica por superación de topes art. 13 Ley 43/1990 / art. 203 Cód. Comercio) — obligación legal de denuncia de irregularidades; debe avalar el diseño de controles.
- **Líder del área financiera** — propietario operativo del proceso. Ya manifestó apoyo activo al proyecto.

### Usuarios finales
- **3 personas del equipo financiero/contable** que hoy ejecutan la causación manual. **Decisión de diseño explícita: NO se eliminan roles. Se transforman de "digitadores" a "validadores/tutores del modelo".** Esto es deliberado para (a) preservar el conocimiento tácito sectorial (OPEX vs CAPEX, costos recuperables ANH, retenciones por municipio) y (b) mitigar el riesgo de sabotaje pasivo documentado en proyectos similares.

### Bloqueadores potenciales identificados
- **Asimetría de incentivos del sponsor:** si el proyecto falla, las consecuencias legales recaen sobre contador y representante legal, no sobre Operaciones. Mitigación: cogobierno formal documentado en comité semanal con minutas firmadas.
- **Resistencia del equipo financiero ante percepción de reemplazo:** mitigación con comunicación temprana del rediseño de roles + capacitación en uso del copiloto.
- **Posible inquietud del revisor fiscal por trazabilidad:** mitigación con diseño de logs inmutables (hash + JSON + firma humana) desde el día 1.

---

## 3. ESTADO ACTUAL

### Procesos actuales (causación de facturas)

1. Llegan facturas por email del proveedor (XML UBL + PDF), por POS físico (ferreterías, restaurantes campamento), por papel (transportadores rurales) o por fotos de WhatsApp desde campo.
2. Una de las 3 personas digitaliza/clasifica el documento.
3. Otra digita manualmente los datos en una plantilla Excel: NIT proveedor, fecha, valor antes IVA, IVA, ReteFuente, ReteIVA, ReteICA (variable según municipio), centro de costo (pozo, bloque, contrato), clasificación OPEX/CAPEX.
4. Validan cruzando con RUT físico/correo y con histórico del proveedor en Excel.
5. Al cierre del mes, consolidan manualmente para generar los 3 reportes financieros recurrentes.
6. Contador titular revisa y firma.

### Herramientas existentes

| Herramienta | Uso actual | Limitación |
|---|---|---|
| Excel | Único sistema de registro contable | Sin trazabilidad de auditoría 10 años, sin control de versiones, sin validaciones automáticas |
| Email | Recepción de facturas | Manual, sin extracción automática del XML adjunto |
| WhatsApp | Recepción de fotos de facturas de campo | Sin estructura, alto riesgo de pérdida |
| n8n | **Disponible y operativo** en Green Power (instancia ya conectada) | Sin uso actual en finanzas — oportunidad de aprovechamiento inmediato |

### Lo que funciona bien (preservar)
- Conocimiento profundo del equipo sobre proveedores, centros de costo y particularidades del sector upstream.
- Cercanía del contador titular al proceso → revisión efectiva.
- Simplicidad: no hay deuda técnica de un ERP mal implementado.

### Oportunidades de mejora
- Eliminar la digitación manual del 80% de las facturas (las que llegan con XML UBL).
- Automatizar la consolidación de reportes mensuales recurrentes.
- Liberar 1.5–2 FTE de tareas repetitivas hacia control tributario y auditoría preventiva.
- Construir trazabilidad regulatoria desde el inicio (no como retrofit).

---

## 4. ESTADO FUTURO DESEADO

### Proceso ideal (post-MVP, fin de las 4 semanas)

1. Facturas llegan al buzón del copiloto (email dedicado + carpeta compartida).
2. **Workflow n8n** detecta el tipo de documento:
   - Si tiene **XML UBL adjunto** → parser determinístico extrae todos los campos firmados (CUFE, NIT, IVA, retenciones, líneas) directamente del XML.
   - Si NO tiene XML (POS, papel, foto) → **Claude Sonnet 4.6 con visión** extrae los campos vía prompt estructurado + esquema JSON.
3. n8n cruza el NIT contra el histórico de proveedores y aplica reglas de retención (por municipio + por tipo de servicio + por calidad tributaria del proveedor) cargadas en una tabla maestra.
4. n8n genera una **propuesta de causación pre-llenada** en el formato Excel actual y la coloca en una bandeja "Pendiente de aprobación".
5. El equipo financiero (los 3 actuales, ahora como **validadores**) revisa cada propuesta, corrige si necesario, y aprueba.
6. Cada aprobación se registra como evento auditable: hash del documento original + JSON de salida del modelo + timestamp + usuario aprobador.
7. Al cierre de mes, n8n consolida automáticamente los 3 reportes financieros recurrentes a partir de las causaciones aprobadas, listos para revisión y firma del contador titular.

### Cambios en la experiencia del usuario

| Antes | Después |
|---|---|
| Digitar 50–60 facturas/mes campo por campo | Revisar y aprobar 50–60 propuestas pre-llenadas |
| Reconciliar manualmente para reporte mensual | Reporte pre-generado, solo validar |
| Tiempo promedio por factura: 15–25 min | Tiempo promedio por factura: 2–4 min (revisión) |
| Cierre mensual: 5–10 días tras corte | Cierre mensual: 1–3 días tras corte (objetivo) |
| Tareas repetitivas: ~70% del tiempo | Tareas de control y auditoría: ~70% del tiempo |

---

## 5. CRITERIOS DE ÉXITO

| Métrica | Línea base actual | Target al final del MVP (4 semanas) | Target a 6 meses |
|---|---|---|---|
| Tiempo promedio de causación por factura | 15–25 min | ≤5 min (revisión de propuesta pre-llenada) | ≤3 min |
| % de facturas con XML UBL procesadas sin intervención AI (parser determinístico) | 0% | ≥80% del subset con XML | ≥95% |
| Accuracy de extracción de campos críticos (NIT, valor, IVA, retenciones) en facturas no-XML | N/A | ≥90% en piloto controlado de 20–30 facturas | ≥95% en producción |
| % de causaciones aprobadas sin corrección manual | N/A | ≥60% en MVP | ≥85% en mes 6 |
| Tiempo desde corte a cierre mensual | 5–10 días | Demostrar reducción a ≤5 días en mes piloto | ≤3 días |
| Trazabilidad auditable por causación (hash + JSON + firma) | 0% | 100% desde día 1 | 100% |
| FTE liberados de tareas repetitivas hacia control/auditoría | 0 | 0.5 FTE en MVP | 1.5 FTE en mes 6 |

**Definición de éxito del MVP a 4 semanas:** prototipo corriendo sobre 20–30 facturas reales del último mes, con 100% de validación humana, comité semanal funcionando, DPIA en borrador, trazabilidad implementada, y carta de respaldo formal del contador titular para avanzar a Fase 2 (producción).

---

## 6. RESTRICCIONES

### Técnicas
- **n8n self-hosted o cloud Starter** como única plataforma de orquestación (decisión arquitectónica ya tomada).
- **Excel como sistema de registro contable** durante la Fase 1 (no se introduce ERP en este alcance).
- **No certificación DIAN del MVP**: el copiloto NO emite facturas ni reemplaza un proveedor tecnológico autorizado por la DIAN. Es herramienta interna de productividad pre-firma.

### Datos
- **Habeas Data (Ley 1581 de 2012, art. 26):** las facturas contienen datos personales de proveedores personas naturales. Transferencia internacional de datos requiere autorización expresa o uso de regiones LATAM.
- **Mitigación:** preferir regiones LATAM cuando estén disponibles (Anthropic `inference_geo: "us"` con uplift 1.1× / OpenAI regional processing con uplift 10% / Azure Brazil South / AWS São Paulo). Documentar acuerdos contractuales con proveedores cloud (DPA).
- **Conservación 10 años** (Ley 962/2005 art. 28 + Concepto CTCP 700/2021): logs y artefactos del modelo deben permitir reproducción fidedigna por 10 años.

### Organizacionales
- Equipo del MVP: 1 sponsor (Juanda) + 1 ingeniero senior (n8n + AI) + las 3 personas del equipo financiero como validadoras + contador titular como aprobador formal.
- Comité semanal obligatorio: Operaciones + Contador + Revisor Fiscal (si aplica) + Dueño.
- Sin contratación externa hasta cerrar Fase 1.

### Compliance
- **Estatuto Tributario** arts. 615, 616-1, 617, 647, 648, 651, 771-2 → toda causación debe respaldarse en factura electrónica/equivalente válida.
- **Resolución DIAN 000165 de 2023** + Anexo Técnico 1.9 UBL 2.1.
- **Ley 43 de 1990 art. 10** → fe pública contable indelegable a algoritmo.
- **Circular Externa 002 de 2024 SIC** → Estudio de Impacto de Privacidad (DPIA) obligatorio antes de producción.
- **Decreto 2420 de 2015 + NIIF para PYMES** → reconocimiento por devengo con fidelidad representativa.

---

## 7. ENFOQUE TÉCNICO PROPUESTO

### Capacidad AI aplicada
1. **Parsing estructurado de XML UBL 2.1** (determinístico, sin AI) — fuente preferida para facturas electrónicas.
2. **Extracción de datos vía LLM con visión** (Claude Sonnet 4.6 o GPT-4.1) — solo para POS, papel, fotos sin XML.
3. **Generación de reportes financieros recurrentes** vía n8n + plantillas + lógica determinística sobre causaciones ya aprobadas.

### Justificación de la decisión técnica
- **XML UBL primero**: el dato ya viene firmado y validado por DIAN. Procesar la imagen cuando hay XML disponible es introducir error donde no lo había. Crítica directamente atendida del Deep Research de riesgos (sección 1.2).
- **Claude Sonnet 4.6 / GPT-4.1 vía nodo nativo n8n**: el nodo nativo elimina semanas de integración custom; el costo a 50–60 facturas/mes es marginal (US$2–6/mes); permite extracción libre de campos colombianos (NIT, CUFE, retenciones) vía prompt + esquema JSON sin entrenar modelo custom.
- **Validación cruzada obligatoria**: cada extracción AI se valida contra (a) RUT en línea DIAN, (b) histórico del proveedor en la tabla maestra, (c) reglas regex de formato (NIT, CUFE, fechas ISO, montos coherentes). Sin las 3 validaciones, la propuesta de causación se marca como "requiere atención humana especial".
- **n8n como orquestador, no como sistema contable**: separación explícita de responsabilidades. n8n flujo de datos + LLM extracción + Excel registro contable + contador firma.

### Arquitectura de alto nivel

```
┌──────────────────────────────────────────────────────────────┐
│  ENTRADAS                                                     │
│  • Email buzón facturas (XML UBL + PDF)                      │
│  • Carpeta compartida (POS, papel, fotos campo)              │
└──────────────────────────────────────────────────────────────┘
                            │
                            ▼
┌──────────────────────────────────────────────────────────────┐
│  n8n WORKFLOW PRINCIPAL                                       │
│                                                                │
│  1. Detector de tipo de documento                             │
│     ├─ XML UBL detectado → Parser determinístico              │
│     └─ Solo PDF/imagen → LLM con visión (Claude/GPT-4.1)      │
│                                                                │
│  2. Validaciones cruzadas                                     │
│     ├─ NIT contra RUT DIAN (servicio en línea)               │
│     ├─ Histórico proveedor en tabla maestra                   │
│     └─ Reglas regex/aritméticas (suma líneas vs total, etc.)  │
│                                                                │
│  3. Aplicación de reglas fiscales                             │
│     ├─ Retenciones por municipio (Yopal, Aguazul, etc.)       │
│     ├─ Calidad tributaria proveedor (régimen simple, autoret) │
│     └─ Clasificación OPEX/CAPEX por centro de costo           │
│                                                                │
│  4. Generación propuesta de causación (Excel)                 │
│                                                                │
│  5. Registro de evento auditable                              │
│     • Hash SHA-256 documento original                         │
│     • JSON output del modelo (con versión + timestamp)        │
│     • Push a bandeja "Pendiente aprobación"                   │
└──────────────────────────────────────────────────────────────┘
                            │
                            ▼
┌──────────────────────────────────────────────────────────────┐
│  HUMAN-IN-THE-LOOP (las 3 personas como validadoras)          │
│  • Revisan propuesta, corrigen si necesario, aprueban         │
│  • Firma electrónica de validador                             │
└──────────────────────────────────────────────────────────────┘
                            │
                            ▼
┌──────────────────────────────────────────────────────────────┐
│  CONSOLIDACIÓN MENSUAL                                        │
│  • n8n agrega causaciones aprobadas → 3 reportes recurrentes │
│  • Contador titular revisa, firma con tarjeta profesional    │
└──────────────────────────────────────────────────────────────┘
```

### Stack tecnológico
- **Orquestación**: n8n (instancia ya operativa en Green Power).
- **LLM extracción**: Claude Sonnet 4.6 (vía nodo nativo Anthropic en n8n) con respaldo GPT-4.1.
- **Parser XML UBL**: determinístico en n8n (Code node JavaScript).
- **Almacenamiento**: SharePoint / OneDrive corporativo + S3 LATAM para logs de auditoría.
- **Registro contable**: Excel actual (sin cambio de sistema en Fase 1).

### Costo operativo proyectado

| Concepto | USD/mes | COP/mes (TRM ≈ 4,000) |
|---|---|---|
| LLM extracción (60 facturas × ~7k tokens) | 2–6 | 8.000–24.000 |
| n8n self-hosted (servidor pequeño) | 0 | 0 |
| Almacenamiento + logs | 1–3 | 4.000–12.000 |
| Validación humana ~20% excepciones | 50–100 | 200.000–400.000 |
| **Total operativo** | **55–130** | **220.000–520.000** |

Comparado con el costo manual actual (COP 2.4M–3.1M/mes): **ahorro proyectado COP 1.9M–2.6M/mes** una vez en producción estable.

---

## 8. RIESGOS Y DEPENDENCIAS

### Matriz de riesgos prioritarios

| # | Riesgo | Probabilidad | Impacto | Mitigación |
|---|---|---|---|---|
| R1 | **Alucinación silenciosa del LLM** (NIT inventado, monto verosímil incorrecto) | Media | Alto (sanción DIAN art. 647 hasta 100% diferencia) | Validación cruzada obligatoria contra RUT DIAN + histórico + regex. Toda factura sin XML pasa por revisión humana al 100% en MVP. |
| R2 | **Sabotaje pasivo del equipo financiero** por percepción de reemplazo | Media-Alta | Alto (pérdida de conocimiento tácito + boicot al MVP) | Rediseño explícito de rol: digitadores → validadores/tutores. Comunicación formal del Dueño desde día 1. Garantía de continuidad laboral durante Fase 1 y 2. |
| R3 | **Sponsor incorrecto sin gobierno de finanzas** | Alta si no se mitiga | Catastrófico (proyecto muere al primer error visible) | Comité semanal con Contador + Revisor Fiscal + Dueño. Aprobación formal documentada del contador titular antes de producción. |
| R4 | **Sanción DIAN por información exógena errónea** (art. 651 ET hasta $392M COP) | Baja en MVP (validación 100%) / Media en producción sin controles | Catastrófico | Trazabilidad inmutable por causación + revisión 100% en MVP + escalamiento gradual a producción solo tras 3 meses sin errores materiales. |
| R5 | **Transferencia internacional de datos personales** sin autorización (Ley 1581 art. 26, sanción hasta 2.000 SMMLV ≈ $2.847M COP) | Media | Catastrófico | DPIA antes de producción. Uso de regiones LATAM. DPA contractual con Anthropic/OpenAI. Cláusulas de autorización en contratos con proveedores. |
| R6 | **Conservación 10 años no garantizada** (Ley 962/2005 art. 28) | Alta sin diseño explícito | Alto | Logs inmutables en S3 con object-lock. Versionado de prompts y modelos. Política de archivado de 10 años desde día 1. |
| R7 | **Dependencia de versión específica del modelo** (deprecación GPT-4V, Sonnet 4.6) | Alta a 24 meses | Medio (reproducibilidad histórica) | Versionado de modelo en cada log. Snapshots de prompts. Plan de migración documentado. |
| R8 | **Sub-estimación del alcance** (4 semanas insuficientes para producción real) | **Alta — se reconoce explícitamente** | Medio (gestionado por re-encuadre) | El MVP entrega prototipo con human-in-the-loop, NO producción contable. Roadmap explícito de 3–5 meses para Fase 2 (producción). |
| R9 | **Lógica fiscal multicapa** (ICA por municipio, retenciones por tipo, OPEX/CAPEX) | Alta complejidad | Medio | Tabla maestra de reglas mantenida por el equipo financiero (no por el LLM). Reglas determinísticas, no probabilísticas. Auditoría trimestral. |
| R10 | **Commoditización por Microsoft Copilot for Finance / Gemini** en 6–12 meses | Media | Medio | El MVP construye **el dato auditable, los logs, las reglas fiscales colombianas y el dominio upstream**, no el motor LLM. Si emerge mejor herramienta, los assets construidos siguen siendo válidos. |

### Dependencias externas
- Disponibilidad continua de la API de Claude (Anthropic) y/o OpenAI.
- Disponibilidad del servicio de validación RUT DIAN en línea.
- Compromiso continuo del Dueño con la cobertura política y presupuestal.
- Disponibilidad del contador titular para participar en el comité semanal y validar aprobaciones.

### Dependencias internas
- n8n debe permanecer operativo y mantenido (responsable: Operaciones).
- La tabla maestra de proveedores y reglas fiscales debe mantenerse al día (responsable: equipo financiero).
- El comité semanal debe sesionar sin falta durante las 4 semanas del MVP y mensual durante Fase 2.

---

## 9. LÍMITES DE ALCANCE

### IN-SCOPE para el MVP de 4 semanas
- ✅ Workflow n8n end-to-end para causación pre-llenada.
- ✅ Parser determinístico de XML UBL 2.1 (al menos 1 plantilla de proveedor probada).
- ✅ Extracción AI vía Claude Sonnet 4.6 para PDFs/imágenes de prueba.
- ✅ Validación cruzada NIT contra RUT DIAN.
- ✅ Tabla maestra inicial de 10–15 proveedores frecuentes con reglas de retención.
- ✅ Bandeja de aprobación con human-in-the-loop al 100%.
- ✅ Trazabilidad por causación (hash + JSON + firma).
- ✅ Piloto controlado con 20–30 facturas reales del último mes.
- ✅ DPIA en borrador (Circular SIC 002/2024).
- ✅ Comité semanal de validación documentado en minutas.
- ✅ Documentación técnica del workflow.

### OUT-OF-SCOPE explícito en el MVP (Fase 2 o posterior)
- ❌ Integración con software contable (Siigo, World Office, Helisa) → Fase 2.
- ❌ Causación masiva en producción sin revisión humana → Fase 2 con escalamiento gradual.
- ❌ Generación automática de Documento Soporte para no obligados a facturar → Fase 2.
- ❌ Conciliación bancaria automatizada → Fase 3.
- ❌ Información exógena (formatos 1001, 1003, 1005, 1006, 1007, 1008) → Fase 3.
- ❌ Nómina electrónica → Fase 3.
- ❌ Cuentas por cobrar y facturación a clientes (Ecopetrol, Frontera, GeoPark) → Fase 4.
- ❌ Vigilancia regulatoria de vencimientos (licencias ANH, ANLA, pólizas, contratos) → Iniciativa paralela ya identificada para roadmap.
- ❌ Aplicaciones de AI en operaciones de campo y yacimientos → Roadmap multi-fase de la Célula AI.

### Roadmap multi-fase (referencia, no compromiso del MVP)

| Fase | Alcance | Duración estimada | Criterio para activar |
|---|---|---|---|
| **Fase 1 (MVP)** | Pre-causación AI con human-in-the-loop, prototipo | 4 semanas | EN CURSO |
| **Fase 2 — Producción** | Integración ERP, escalamiento gradual a producción, DPIA aprobado | 3–5 meses | Cierre exitoso de Fase 1 + carta del contador titular |
| **Fase 3 — Expansión finanzas** | Conciliación bancaria, exógena, nómina electrónica | 4–6 meses | Estabilidad operativa Fase 2 ≥ 3 meses |
| **Fase 4 — Otras áreas** | Vigilancia regulatoria, aplicaciones de campo, yacimientos | Por definir | Por definir |

---

## Checklist de entrega

- [x] Problema cuantificado en COP/mes y FTE
- [x] Sponsor ejecutivo y técnico identificados con sus responsabilidades
- [x] Stakeholders críticos mapeados (incluido contador y revisor fiscal)
- [x] Estado actual descrito con herramientas y limitaciones
- [x] Estado futuro descrito con cambios cuantificables en UX
- [x] Criterios de éxito con línea base, target MVP y target a 6 meses
- [x] Restricciones técnicas, datos, organizacionales y de compliance documentadas
- [x] Enfoque técnico justificado con arquitectura de alto nivel y costo operativo
- [x] Riesgos prioritarios con probabilidad, impacto y mitigación
- [x] Límites de alcance in/out explícitos + roadmap multi-fase
- [x] Deep Research de validación completado y estudiado (`research-validacion.md`)
- [x] Deep Research de crítica completado y estudiado (`research-critica.md`)
- [x] Documentación en Markdown lista para entrega
- [x] Listo para presentar en la Estación 2 (miércoles 13 de mayo de 2026)

---

## Anexos

- **Anexo A:** `research-validacion.md` — Deep Research de validación (5 secciones, fuentes citadas).
- **Anexo B:** `research-critica.md` — Deep Research de crítica (7 secciones, 12 supuestos cuestionados, fuentes legales colombianas citadas).

---

*Hardcore AI by 30X — Cohorte 2 — Estación 1 — Mayo 2026*
*Green Power S.A.S. — Llanos Orientales, Colombia*
