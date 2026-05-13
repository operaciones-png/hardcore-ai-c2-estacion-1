# Análisis para Green Power — Célula AI Fase 1: Automatización financiera con n8n

Empresa: Green Power (E&P hidrocarburos, Llanos Orientales, 90 empleados, 50–60 facturas/mes, sin ERP, stack n8n). MVP de 4 semanas: causación AI desde PDF/imagen + 3 reportes mensuales recurrentes.

---

## 1. CASOS COMPARABLES

**Hallazgo principal:** **no se encontró información pública verificable** de una empresa colombiana de E&P de tamaño mediano (50–200 empleados) que haya publicado un caso de éxito con nombres, métricas y fechas sobre causación de facturas con AI. Los casos públicos en oil & gas son principalmente de Norteamérica y de proveedores SaaS verticales (Joltly, Stampli, Lightyear, IFS, Enverus OpenInvoice) que reportan métricas agregadas de sus clientes, no de operadores específicos.

**Casos en oil & gas (Norteamérica, extrapolables como benchmark):**

- **Joltly (oil & gas AP SaaS):** publica que sus clientes reportan reducción del flujo de AP del **80%** (testimonio "EII") y reducción del tiempo de procesamiento de factura del **95%**, con ahorro de **20 horas/semana** en AP/AR a partir de US$500/mes. Cita un caso anónimo de un "midsize oil company" con **2,5× incremento en throughput de facturas y 15% de reducción en DPO en el primer año**, y un rango típico de **ROI 200–300% en 12–18 meses** para operadores energéticos (joltly.io/blog/ai-accounts-payable-automation-for-oil-and-gas-companies; joltly.io). Es marketing del proveedor, no auditado por tercero — tratar como cota superior optimista.
- **Bishop Lifting (industrial / oil & gas servicios) con Stuut (AR, no AP):** 91% de comunicaciones automatizadas, 35% reducción en cuentas vencidas, US$3M de mejora de capital de trabajo, **6 semanas para go-live en 45 sucursales** (stuut.ai/blog/best-ar-automation-software-for-oil-and-gas-companies). Es AR, no AP, pero útil como referente de cronograma piloto→producción.
- **Stampli** y **Lightyear** publican casos en energía sin métricas específicas de empresas colombianas o LATAM nombradas (stampli.com/oil-gas-energy/; lightyear.cloud/for-business/oil-gas-energy/).

**Empresas medianas en Colombia (otras industrias, AP automation con IA + OCR):**

- **Alpes Solutions (Colombia, integrador):** publica resultados agregados de clientes (manufactura, agroindustria, retail): IA+OCR que valida NIT, valores y fechas, con **6–8 horas/día ahorradas**, "cero digitación manual", **60–80% de reducción en tiempos operativos** (alpessolutions.com/servicios/automatizacion-inteligente-para-operaciones-empresariales/). No publica nombres de clientes ni ROI auditado.
- **AI Automation Colombia** (aiautomation.com.co): ofrece extracción de NIT, CUFE, fecha y valor desde PDF/XML y reconciliación contra DIAN; sin métricas de clientes públicas.
- Caso citado por Alpes: "grupo financiero" en Colombia con **70% reducción de tiempo en tareas administrativas** vía ML en gestión documental (sin nombre publicado).

**Tiempo piloto→producción (benchmark global):** los integradores reportan **2–4 semanas para Quick Wins** y **6–10 semanas para Automation-as-a-Service de 3–5 procesos** (alpessolutions.com). Stuut/Bishop Lifting alcanzó producción multi-sitio en 6 semanas. Plataformas tipo HighRadius requieren **3–6 meses** (stuut.ai). Para el MVP de Green Power (volumen bajo, sin ERP), 4 semanas es factible si se acota a extracción + plantilla Excel; integraciones a ERP futuras escalarán a 8–12 semanas.

**Conclusión sobre la pregunta original:** no hay un caso público "Green Power-equivalente" en E&P colombiano. La justificación interna debe construirse con benchmarks de APQC/Ardent (sección 3) y referentes operativos genéricos.

---

## 2. TECNOLOGÍAS VÁLIDAS — COMPARATIVA OPERATIVA

Costos verificados a 2025–2026 directamente con proveedores. Para Green Power, 50–60 facturas/mes ≈ 60–180 páginas/mes.

### Tabla comparativa

| Proveedor | Costo/doc | Precisión (español/invoice) | Integración n8n | Autenticación + infraestructura + certificaciones | Campos colombianos (NIT/IVA/Retenciones/CUFE) |
|---|---|---|---|---|---|
| **Claude API (Anthropic)** | Tokens: $3/$15 por MTok (Sonnet 4.6), $1/$5 (Haiku 4.5); 1 factura ≈ **US$0.008–0.04** (claude.com/pricing; finout.io/blog/anthropic-api-pricing) | Alta con prompt + esquema JSON. Visión 3,75 MP en Opus 4.7. Sin benchmark público específico para facturas colombianas | **Nodo nativo "Anthropic"** con operación Analyze Document (docs.n8n.io/integrations/builtin/app-nodes/n8n-nodes-langchain.anthropic/) | API key bearer; SaaS US (us-east). SOC 2 Type II, HIPAA-eligible. `inference_geo: "us"` con uplift 1,1× | Extracción libre vía prompt: NIT, IVA, ReteIVA, ReteFuente, ReteICA, CUFE textualmente. Requiere validación regex post-modelo |
| **GPT-4o / GPT-4.1 (OpenAI)** | GPT-4o: $2.50/$10 por MTok; GPT-4.1: $2/$8; **≈ US$0.02–0.05/factura** (pricepertoken.com; developers.openai.com/api/docs/pricing). Benchmark: **90,5% accuracy de campos**, ~33 s/página (businesswaretech.com) | 90% campo a campo; débil en tablas line-item | **Nodo nativo "OpenAI"** con visión (docs.n8n.io/integrations/builtin/app-nodes/n8n-nodes-langchain.openai/) | API key; SaaS. SOC 2 Type 2, CSA STAR, EU/US data zones. Regional processing con uplift 10% | Mismo enfoque que Claude vía prompt + JSON schema |
| **Azure AI Document Intelligence** | Prebuilt Invoice: **US$10/1k páginas = $0.01/pág**; Read $1.50/1k; Custom $30/1k; Tier F0 gratis 500 pág/mes (azure.microsoft.com/en-us/pricing/details/document-intelligence/) | Mejor en facturas con layouts europeos/LATAM en benchmark Businesswaretech; soporta español; en 2025 expandió monedas/idiomas (learn.microsoft.com) | **Sin nodo nativo "Document Intelligence"** en n8n. Vía **HTTP Request node** (REST API) o nodo Azure OpenAI Chat Model | API key o Azure AD/SAS; region Brazil South disponible (data residency LATAM). ISO 27001, SOC 1/2/3, HIPAA, FedRAMP High, PCI DSS, residencia de datos garantizada por región | Prebuilt extrae VendorTaxId/CustomerTaxId genérico y TaxItems. **Retenciones colombianas y CUFE NO están en el esquema prebuilt** — requieren Custom Neural entrenado o post-procesamiento. Limitación más relevante para Colombia |
| **Google Document AI — Invoice Parser** | **US$0.10/documento (1–10 páginas)**; OCR $1.50/1k con descuento a $0.60/1k sobre 5M (cloud.google.com/document-ai/pricing) | **Peor performance** en benchmark: ~82% campos, **40% en line-items**, débil en layouts no estándar (businesswaretech.com; braincuber.com) | **Sin nodo nativo**; HTTP Request + auth GCP service account (JWT/OAuth) | Service account JSON; region São Paulo (southamerica-east1). ISO 27001/27017/27018, SOC 1/2/3, HIPAA, PCI DSS | Genérico, sin soporte directo a CUFE/retenciones |
| **AWS Textract — AnalyzeExpense** | **US$0.008–0.01/página** (aws.amazon.com/textract/pricing/). DetectText básico $1.50/1k | Modesta en campos básicos; benchmark 82% line-items con tablas estructuradas; débil en layouts complejos (businesswaretech.com) | Sin nodo nativo en n8n core; existen nodos community + HTTP Request (firma SigV4 requerida) | AWS Access Key + SigV4 o IAM Role; region São Paulo (sa-east-1). SOC 1/2/3, ISO 27001/27017/27018, HIPAA, PCI DSS, FedRAMP | Sin extracción nativa de NIT/CUFE. Custom Queries permite preguntar a $0.015/pág adicional |
| **Rossum** | **Pricing por contrato anual**; benchmark ≈ **US$18.000/año** (gennai.io/blog/best-ocr-software-invoices-2026); contrato mínimo 1 año (rossum.ai/pricing). **Sobredimensionado para 60 facturas/mes** | 98%+ accuracy template-free | Sin nodo n8n nativo; API REST + SFTP. Conectores certificados SAP/Coupa | API key/OAuth; SaaS multi-region (EU, US). ISO 27001, SOC 2, HIPAA, TX-RAMP | No documenta extracción nativa de CUFE/retenciones colombianas; soporta español multilayout |
| **Klippa (Doxis Invoice OCR)** | Pricing por contrato (no público). Tier por volumen | **Soporta español** entre 12+ idiomas latinos; extrae VAT, line items, totals; 2-way matching (klippa.com/en/ocr/financial-documents/invoices/) | Sin nodo n8n nativo; HTTP Request | API key; SaaS NL. ISO 27001, SOC 2, GDPR | Sin mención explícita de NIT/CUFE colombianos |
| **Mindee (Invoice OCR API)** | Plan **gratis 250 páginas/mes**; pago: **US$0.10/pág decreciente a $0.01/pág** según volumen (mindee.com/product/financial-document-ocr-api). **60 facturas/mes caen en tier gratis** | **≥90% global, ≥95% en la mayoría de campos**, entrenado con facturas de 50+ países (mindee.com) | Sin nodo n8n nativo; REST API + SDKs Python/Node/Java/PHP | API key; SaaS EU. SOC 2 Type II, GDPR-ready | Esquema estándar (supplier, total_amount, tax). Extrae VAT pero **no diferencia retenciones colombianas ni CUFE de forma nativa** |

### Recomendación técnica para Green Power (MVP 4 semanas)

Stack óptimo dado el contexto (n8n, sin ERP, 50–60 facturas/mes, requiere CUFE/NIT/retenciones):

1. **Capa de extracción primaria: Claude Sonnet 4.6 o GPT-4.1 con visión** via nodo nativo n8n. Razones: (a) nodos nativos = integración en minutos; (b) prompt libre permite especificar "extrae NIT, CUFE, ReteIVA, ReteFuente, ReteICA del PDF" sin entrenar custom model; (c) costo ≈ US$1–3/mes para 60 facturas; (d) accuracy de campos ≈ 90%.
2. **Capa de validación**: Azure Document Intelligence prebuilt Invoice ($10/1k pág, ≈ US$0,60/mes) para los campos estándar (VendorTaxId, total, fecha); LLM como respaldo en campos faltantes. Esto reduce alucinaciones.
3. **Parsing directo del XML UBL**: si el proveedor entrega el XML adjunto de factura electrónica (obligatorio bajo Res 165/2023), **parsear el XML UBL elimina la OCR**. CUFE, NIT, IVA y retenciones están allí como campos estructurados firmados. AI/OCR queda solo para facturas en papel, no electrónicas (cada vez menos comunes post-2024).

**Costo proyectado total de extracción para 60 facturas/mes: US$2–6/mes** + tier gratis Mindee/Azure si se quiere redundancia.

---

## 3. BENCHMARK FINANCIERO

### Costo de procesar manualmente una factura

| Fuente | Costo manual por factura |
|---|---|
| **APQC Open Standards Benchmarking** (cross-industry, 2023–2024) | Top performers (P25): **US$1,77–2,07**; bottom performers (P75): **US$10–10,89**; mediana cross-industry: **US$5,83** (apqc.org/resources/benchmarking/open-standards-benchmarking/measures/total-cost-perform-process-process-19; cfo.com/news/metric-of-the-month-accounts-payable-cost/659393/; nanonets.com/blog/cost-of-processing-an-invoice/) |
| **Ardent Partners "State of ePayables" 2025** (citado por Parseur) | Best-in-Class: **US$2,78/factura**; resto: **US$12,88/factura**. Tiempo de ciclo: 3,1 vs 17,4 días. Excepciones: 9% vs 22% (parseur.com/blog/ai-invoice-processing-benchmarks) |
| **APQC + Levvel (2024)** | Manual: **US$12–30/factura**; automatizado: **US$1–5/factura** (artsyltech.com/blog/invoice-processing-automation-guide) |

**Conversión a COP** (TRM aprox. mayo 2026: COP ≈ 4.000/USD, varía):
- Manual benchmark típico medio: US$6 ≈ **COP 24.000/factura**
- Manual bottom-quartile (caso Green Power probable: Excel, sin ERP, 3 personas): US$10–13 ≈ **COP 40.000–52.000/factura**
- Para 60 facturas/mes: **COP 2,4M–3,1M/mes en costo escondido de AP manual** (carga laboral + retrabajo + errores).

**Nota:** **no se encontró publicación de APQC, Ardent Partners, Levvel, Deloitte, PwC, EY o KPMG con benchmark separado para Colombia o LATAM** sobre cost-per-invoice. Los datos públicos son cross-industry globales con sesgo norteamericano. Esto es una limitación de las fuentes — los costos LATAM se asumen iguales en USD aunque el costo de mano de obra local es menor (≈ 30–50% menos), por lo que **el costo real manual en Colombia podría estar entre US$3 y US$7/factura**, no en el rango alto US.

### Costo proyectado con AI para 50–60 facturas/mes en Green Power

| Concepto | Costo mensual estimado |
|---|---|
| LLM extracción (Sonnet 4.6 o GPT-4.1, 60 facturas × ~7k tokens) | US$2–6 |
| n8n self-hosted (servidor pequeño) o n8n Cloud Starter | US$0 (self-host) – US$20 |
| Almacenamiento PDFs + logs (S3/Drive) | US$1–3 |
| Validación humana ~20% (excepciones) | 6–12 horas/mes de un contador ≈ US$50–100 |
| **Total operativo AI** | **US$55–130/mes** |

Equivalente APQC para "automatizado": US$1–5/factura × 60 = **US$60–300/mes**. La estimación cae en el extremo inferior porque (a) no hay licencia de software AP dedicada y (b) el LLM con n8n es marginalmente barato a este volumen.

### Rango de ROI razonable

| Horizonte | ROI estimado |
|---|---|
| **6 meses** | Punto de equilibrio o **50–100%** sobre la inversión inicial del MVP, asumiendo costo de implementación US$5.000–15.000 y reasignación de 1 de las 3 personas a tareas de mayor valor. Driver: Ardent reporta tiempo de ciclo 3,1 vs 17,4 días → 82% reducción |
| **12 meses** | **150–300%**, alineado con el rango Joltly "200–300% en 12–18 meses para energía" y Artsyl/Levvel: 60–80% reducción de costo unitario |

**Caveat:** estos rangos son guías globales. Para una operación E&P de 90 empleados sin ERP, el ROI dependerá menos del costo unitario (volumen muy bajo) y más de **liberar capacidad de los 3 FTE actuales** para auditoría tributaria, control de retenciones y soporte a contabilidad regulada por DIAN — actividades de mayor riesgo monetario que la digitación.

---

## 4. REGULACIÓN Y CUMPLIMIENTO COLOMBIA

### 4.1 DIAN — Facturación electrónica y soportes contables

- **Resolución DIAN 000165 de 1 nov 2023** (dian.gov.co/normatividad/Normatividad/Resoluci%C3%B3n%20000165%20de%2001-11-2023.pdf; normograma.dian.gov.co/dian/compilacion/docs/resolucion_dian_0165_2023.htm). Reemplaza la 042 de 2020. Establece:
  - Adopción del **Anexo Técnico 1.9** de Factura Electrónica de Venta (XML UBL 2.1 firmado, CUFE como código único, validación previa de la DIAN).
  - **Anexo Técnico 1.0** del Documento Equivalente Electrónico (POS, transporte, espectáculos, etc.).
  - Plazos modificados por **Resolución 000008 de 31 ene 2024**: adopción Anexo 1.9 al 1 mayo 2024; POS electrónico escalonado entre 1 mayo y 1 jul 2024 (dian.gov.co/normatividad/Normatividad/Resoluci%C3%B3n%20000008%20de%2031-01-2024.pdf).
- **Decreto 358 de 2020** reglamenta los artículos 511, 615, 616-1, 617, 618 y 771-2 del Estatuto Tributario y se codificó en el Decreto Único Reglamentario 1625/2016. Define el **Documento Soporte en Adquisiciones a No Obligados a Facturar** que Green Power debe generar cuando compre a un proveedor no facturador electrónico (proveedores informales, pequeños, no responsables de IVA bajo umbral): es responsabilidad del adquirente. La AI debe estar preparada para generar este documento soporte, no solo causar facturas recibidas.
- **Requisitos prácticos para Green Power como adquiriente** (siigo.com/blog/obligaciones-fiscales/resolucion-165-de-2023/; incp.org.co/publicaciones):
  - Toda factura electrónica recibida lleva XML con CUFE, NIT emisor/adquiriente, IVA discriminado, retenciones cuando aplique. **Para causación con IA la fuente preferida es el XML, no el PDF visible**: es campo estructurado y trazable a la DIAN.
  - Conservar XML + representación gráfica (PDF) por **mínimo 5 años** (art. 632 ET).
  - Soporte de costos, deducciones e IVA descontable requiere acuse de recibido (mensajes CUDE/CUFE) según art. 11 y siguientes de Res 165/2023.

### 4.2 Procesar facturas con IA en nube — Ley 1581 de 2012 (Habeas Data)

- **Ley 1581 de 2012** (suin-juriscol.gov.co/viewDocument.asp?id=1684507; alcaldiabogota.gov.co/sisjur/normas/Norma1.jsp?i=49981) aplica a tratamiento de datos personales. Facturas contienen datos de personas naturales (proveedores PN, representantes legales): es **dato personal**.
- **Transferencia internacional** (Art. 26): se prohíbe a países sin "nivel adecuado". La **SIC** mantiene la lista. EE.UU. **no está en la lista de nivel adecuado**, pero la transferencia es legal si: (a) hay autorización expresa del titular, (b) es necesaria para ejecutar un contrato, (c) se firman cláusulas contractuales tipo, o (d) hay declaración de conformidad de la SIC.
- **Recomendación operativa**: usar regiones LATAM cuando estén disponibles (AWS São Paulo `sa-east-1`; Azure Brazil South `brazilsouth`; Google `southamerica-east1`). OpenAI ofrece **regional processing con uplift de 10%** (developers.openai.com/api/docs/pricing). Anthropic permite `inference_geo: "us"` con multiplicador 1,1× (claude.com/pricing).
- **Circular Externa 002 de 21 agosto 2024 (SIC)** sobre datos personales en sistemas de IA (cuatrecasas.com/es/spain/tecnologia-medios-digitales/art/guia-uso-responsable-ia-datos-personales-colombia; sedeelectronica.sic.gov.co): exige antes del diseño un **Estudio de Impacto de Privacidad (PIA/DPIA)** cuando el tratamiento entrañe alto riesgo. Aplica **idoneidad, necesidad, razonabilidad y proporcionalidad**. Green Power debe documentar el PIA antes de productivizar la Célula AI.
- **CONPES 4144 de 2025** sobre IA en Colombia: marco de política pública no vinculante, referencia obligada de soft-law (blog.arielapp.co/inteligencia-artificial-en-colombia-2025-guia-legal-del-conpes-4144-y-la-sic-para-abogados-y-empresas/).

### 4.3 Trazabilidad y responsabilidad si la AI causa con error

La **DIAN no ha emitido normativa específica sobre AI en causación**. Aplican reglas generales:
- **Responsabilidad del contador público** (Ley 43 de 1990, art. 10): el contador firma con su tarjeta profesional; un error de causación es responsabilidad personal del contador **independientemente del medio que lo produjo**. La IA es herramienta, no sujeto responsable.
- **Soporte documental** (Estatuto Tributario art. 771-2): toda causación debe estar respaldada por factura electrónica/equivalente válida + soporte de pago. En un esquema con AI, esto exige **logs por factura**: (a) hash del PDF/XML original, (b) JSON de salida del modelo con timestamp y versión, (c) firma del contador que valida el asiento.
- En auditoría, la DIAN puede aplicar **sanción por inexactitud** (art. 647 ET, 100% del mayor valor). La defensa estándar es mostrar la trazabilidad y la corrección oportuna vía corrección voluntaria (art. 588 ET).

### 4.4 Ley 2300 de 2023 y normativa reciente de AI

- **Ley 2300 de 2023 ("Dejen de fregar")** (ccb.org.co/es/blog/informacion-importante-sobre-proteccion-de-datos-personales-y-ley-2300-de-2023): regula contacto comercial no deseado (llamadas, SMS). **No aplica directamente** al caso de uso de Green Power (back-office financiero interno).
- **No hay ley vigente específica de IA en Colombia a mayo 2026.** Hay 9+ proyectos en trámite: Proyecto de Ley 091 de 2023 (deber de información sobre IA), Proyecto 130 de 2023 (IA y derecho al trabajo), Proyecto de Ley de IA radicado por MinCiencias el 7 mayo 2025 (minciencias.gov.co/sala_de_prensa/minciencias-lidera-el-proyecto-ley-que-busca-regular-el-desarrollo-etico-seguro-y), Proyecto de Ley 156 de 2023 (actualización Ley 1581, archivado; nueva radicación ago 2025). La regulación aplicable hoy es **Ley 1581 + Circular SIC 002/2024 + CONPES 4144** (soft-law).
- **Sentencia Corte Constitucional T-323/2024** sobre uso de IA en sentencias (cotino.es/colombia/) fija doctrinas relevantes para cualquier uso institucional de IA: transparencia, responsabilidad, privacidad e **identificación de un humano responsable** por cada decisión asistida. Para Green Power: el contador público sigue siendo el "humano responsable" de cada asiento.

---

## 5. EVIDENCIA DE WILLINGNESS TO ADOPT EN COLOMBIA / SECTOR

**Hallazgo central: no se encontró información pública específica de la ACP (Asociación Colombiana del Petróleo y Gas), Campetrol ni ACIPET con casos publicados sobre adopción de IA en back-office financiero del sector hidrocarburos colombiano.** Esta sección es la más débil en evidencia primaria — el presupuesto de búsquedas se agotó antes de profundizar más en estos gremios y en las agendas de eventos sectoriales colombianos.

**Lo que sí se documentó:**

- **Demanda activa por AP automation en Colombia (otras industrias):** existen empresas Colombia-nativas (Alpes Solutions, AI Automation Colombia, Automaxia) que comercializan explícitamente extracción IA+OCR de facturas con validación contra DIAN, NIT y CUFE como diferenciador local (aiautomation.com.co; alpessolutions.com). Indica un mercado de demanda probada, no necesariamente concentrado en oil & gas.
- **Marco soft-law institucional pro-adopción:** CONPES 4144 de 2025 (Marco Nacional de IA) y la postura del MinTIC con la iniciativa "PotencIA Digital" (mintic.gov.co/portal/inicio/Sala-de-prensa/Noticias/382370) crean cobertura política para que empresas del sector real adopten IA dentro de las restricciones del Habeas Data.
- **Big-4 sobre oil & gas + AI:** los reportes públicos de Deloitte, PwC, EY y KPMG sobre AI en oil & gas LATAM 2023–2026 enfocan mayoritariamente upstream operacional (mantenimiento predictivo, optimización de yacimientos, exploración sísmica con ML) y no específicamente AP automation en empresas medianas. **No se encontró un reporte big-4 público dedicado a "AP automation con AI en oil & gas medianas LATAM"** — los reportes existentes (Apptunix, Master of Code, Joltly) tratan AP como un caso entre muchos, dentro de un universo dominado por Shell, Cairn-Vedanta y operadores norteamericanos.
- **Eventos sectoriales Colombia (Colombia Oil & Gas Summit, Andesco, Naturgas, ACIPET Congreso, Campetrol Asamblea):** **no se logró verificar en esta investigación agendas específicas con sesiones dedicadas a AI en finanzas/back-office**. Esta es una **brecha de información** que Green Power debería cubrir directamente: (a) contactar a Campetrol —la cámara con mayor afinidad con empresas medianas de servicios y operadores— y (b) revisar memorias publicadas del Colombia Oil & Gas Summit (sumitenergias.com), del Congreso ACIPET y de Naturgas/Andesco.

**Lo que esto implica para Green Power:**
1. No hay "case study LATAM oil & gas mediano" para mostrar en el comité directivo. La justificación interna debe construirse con benchmarks APQC/Ardent (sección 3), no con análogos del sector.
2. Hay cobertura regulatoria suficiente (CONPES 4144 + Circular SIC 002/2024) para implementar IA en back-office con bajo riesgo regulatorio, siempre que se hagan DPIA y se mantenga trazabilidad.
3. La oportunidad de **posicionamiento gremial** es alta: hay espacio para que Green Power sea uno de los primeros casos documentados de E&P mediana colombiana con Célula AI en finanzas, lo cual tiene valor de marca interno con ACP/Campetrol.

---

## SÍNTESIS EJECUTIVA Y RIESGOS

- **MVP técnico viable en 4 semanas** con stack: n8n (orquestación) + Claude Sonnet 4.6 o GPT-4.1 (extracción vía nodo nativo) + parsing directo del XML UBL cuando esté disponible + Excel como sistema de registro contable. Costo operativo total mensual: **US$55–130**.
- **Benchmarks defendibles** (APQC, Ardent Partners): reducción de costo unitario por factura del **60–80%**, reducción de tiempo de ciclo del **80%+**, ROI **150–300% a 12 meses** para empresas medianas.
- **Riesgos prioritarios** a documentar:
  1. **Regulatorio**: hacer DPIA antes de producción (Circular SIC 002/2024); preferir regiones cloud LATAM o cláusulas DPA con proveedores US; documentar al contador firmante como responsable humano.
  2. **Trazabilidad DIAN**: cada factura causada por AI debe tener log inmutable (hash PDF/XML + JSON del modelo + firma humana). Sin esto hay riesgo de sanción por inexactitud (ET art. 647) si la DIAN audita.
  3. **Falta de análogos sectoriales**: el caso de Green Power **será uno de los primeros documentados** en E&P mediana colombiana. No esperar peer case en ACP/Campetrol/ACIPET.
  4. **CUFE y retenciones colombianas no están en esquemas prebuilt** de Azure/Google/AWS/Mindee; esto refuerza la decisión de usar LLM con prompt libre + validación contra XML de la DIAN cuando exista.
  5. **Sobre-ingeniería**: 50–60 facturas/mes no justifican Rossum ($18k/año) ni custom models de Azure ($30/1k pág + entrenamiento). Mantener el stack mínimo.

**Brechas de información** que requieren follow-up directo del equipo (no resolubles vía búsqueda web pública en este ejercicio):
- Casos específicos de operadores E&P medianos colombianos con AP automation IA (ACP, Campetrol, ACIPET).
- Reportes big-4 dedicados a finanzas digitales en oil & gas LATAM 2023–2026.
- Agendas de Colombia Oil & Gas Summit / Campetrol con sesiones sobre AI en back-office.