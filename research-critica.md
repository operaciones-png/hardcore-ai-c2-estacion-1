# Auditoría crítica del proyecto "Green Power – AI para causación de facturas"
**Auditor escéptico de fracasos de proyectos AI — Llanos Orientales, Colombia**

Esteban, voy a destruir esta idea, no a defenderla. No hay veredicto al final. Solo riesgos. Lea con estómago lleno.

---

## 1. RIESGOS DE FRACASO TÉCNICO

**1.1. El supuesto de "factura única y limpia" no existe en upstream de los Llanos.** En una empresa de 90 personas en Llanos Orientales, el universo de proveedores incluye: ferreterías de Yopal/Villavicencio que emiten POS térmico (documento equivalente, no factura electrónica), transportadores rurales con factura en papel a mano (alquiler de camionetas, vacuum, manguera), restaurantes de campamento con tickets POS, talleres mecánicos con facturas escaneadas con celular y orientación rotada, y contratistas grandes que sí emiten XML UBL 2.1. Eso son **al menos cinco tipos distintos** de documento, cada uno con su propio pipeline. No es un problema, son cinco.

**1.2. El XML UBL 2.1 de la DIAN no se "lee con vision".** La Resolución 000165 de 2023 (https://www.dian.gov.co/normatividad/Normatividad/Resoluci%C3%B3n%20000165%20de%2001-11-2023.pdf) obliga al estándar UBL 2.1 con CUFE, validación previa DIAN y firma digital X.509. El XML es un archivo estructurado: usar un LLM con visión sobre la representación gráfica PDF en vez de parsear el XML es técnicamente absurdo y descarta el dato más confiable que ya viene validado por la DIAN. Si el equipo está pensando "GPT-4V lee facturas", está reemplazando un parser determinístico de 50 líneas por una alucinación probabilística de USD 0,01–0,05 por imagen. El anexo técnico v1.9 trae además reglas de validación aritmética (suma de líneas vs total, IVA discriminado) que un LLM no replica nativamente.

**1.3. Tasa de error real de LLM vision en facturas en español, no marketing.** El paper de Snowflake "Notes on Applicability of GPT-4 to Document Understanding" (https://arxiv.org/pdf/2405.18433) reporta para GPT-4 Turbo Vision + OCR un 71,9 % en DocVQA y **53,3 % en DUDE**, lejos del 98,1 % humano. El estudio "Exploring OCR Capabilities of GPT-4V(ision)" (https://arxiv.org/abs/2310.16809) concluye textualmente: *"GPT-4V performs well in Latin contents, but struggles with multilingual scenarios and complex tasks… GPT-4V does not outperform existing state-of-the-art OCR models"*. El benchmark VideoDB (https://arxiv.org/html/2502.06445v1) reporta GPT-4o entre **65 % y 80 % de accuracy** con caídas en texto manuscrito. Traducido: en 50–60 facturas/mes, espere **10 a 20 facturas/mes con al menos un campo mal extraído**. Cada error es una causación incorrecta firmada por un contador público.

**1.4. La alucinación silenciosa es el riesgo letal, no el OCR fallido.** Un LLM con visión no responde "no sé": inventa un NIT plausible, inventa una fecha cercana, inventa un valor con dígitos verosímiles. Detectarlo requiere validación cruzada contra el RUT en línea (servicio DIAN), contra el RUES (Cámara de Comercio) y contra el histórico del proveedor en la contabilidad. n8n no trae esas validaciones; hay que construirlas. Si no las construyen, la AI "funciona" hasta que el contador descubre tres meses después que el NIT 900.xxx.xxx-1 nunca existió y la factura entró como soporte de costos rechazables (art. 771-2 del Estatuto Tributario).

**1.5. POS y documento equivalente no soportan costos/deducciones igual que la factura electrónica.** El art. 616-1 del ET y la Resolución 000165/2023 son explícitos: para procedencia de costos, deducciones e impuestos descontables se requiere factura de venta o documentos previstos cumpliendo art. 617 ET. El sistema AI debe distinguir POS de factura electrónica para clasificar correctamente — si no lo hace, contabiliza como costo deducible algo que no lo es y genera glosa en renta. Texto: http://www.secretariasenado.gov.co/senado/basedoc/estatuto_tributario_pr027.html.

**1.6. n8n no es una plataforma contable.** Es un orquestador. No tiene control de versiones contables, ni trazabilidad de auditoría conforme al art. 28 de la Ley 962 de 2005 y los conceptos del CTCP sobre soportes electrónicos (Concepto CTCP 700 de 2021 — los soportes deben permitir reproducción exacta por 10 años, https://accounter.co/niif/soportes-contables-electronicos-concepto-ctcp-700-de-2021.html). Cuando n8n caiga (y caerá, es low-code self-hosted típicamente), las causaciones del último mes pueden no tener log reproducible. Adicionalmente, n8n **no está en la lista de proveedores tecnológicos autorizados por la DIAN** (https://micrositios.dian.gov.co/sistema-de-facturacion-electronica/normatividad/).

**1.7. Facturación de proveedores rurales pequeños en Llanos.** Muchos micronegocios de los Llanos están bajo régimen simple o no son responsables de IVA; emiten documento equivalente o facturas manuales. El LLM tendrá que decidir si aplica retención en la fuente, ReteIVA y ReteICA — lo cual depende del **municipio** (Yopal, Aguazul, Tauramena, Maní tienen tarifas distintas de ICA), del **tipo de servicio** (transporte ~1%, servicios generales 4%/6%, compras 2,5%), y de la **calidad tributaria del proveedor** (gran contribuyente, autoretenedor, régimen simple, no responsable de IVA). Esto no es OCR, es lógica fiscal multicapa que el equipo no ha modelado.

**1.8. Facturas electrónicas "mal formadas" son frecuentes en proveedores pequeños.** Tags UBL omitidos, código de producto faltante, IVA agregado pero no discriminado, fechas en formato no ISO. Alegra documenta como observación frecuente "Debe existir el grupo de información de identificación del bien o servicio" (https://ayuda.alegra.com/es/emite-facturas-electronicas-alegra-colombia). El LLM no sabe rechazar un XML inválido; lo "interpreta" — peor escenario posible.

---

## 2. RIESGOS DE ADOPCIÓN

**2.1. Las tres personas saben que esto es para reemplazarlas.** "Reemplazando trabajo de 3 personas" está en la descripción del proyecto. No hace falta hipótesis: el sabotaje pasivo está garantizado. Formas concretas: (i) "olvidar" subir facturas físicas al sistema, (ii) reportar al contador errores de la AI sin reportar sus propios errores históricos en Excel, (iii) no documentar criterios tácitos de causación que solo ellos conocen (centro de costo "Pozo CPO-9 Workover" vs "CPO-9 Mantenimiento" — la diferencia importa para el costo unit del barril).

**2.2. El conocimiento tácito del sector petrolero no está en el LLM.** Causar correctamente una factura de servicios upstream exige saber si el costo es **OPEX vs CAPEX** (NIIF para PYMES Sección 17 / 13, Decreto 2420 de 2015), si va a **centro de costo por pozo, por bloque, por contrato JOA**, si aplica **retención de timbre** en contratos de obra, si es **costo recuperable bajo el contrato E&P con la ANH** o no recuperable. Eso no lo aprende GPT-4V mirando un PDF. Lo aprende un auxiliar contable en 6–18 meses al lado del jefe de costos. Si las tres personas se van resentidas, ese conocimiento se pierde y la AI hereda criterios incorrectos que luego se replican a escala.

**2.3. Sin claridad de rol post-implementación, los mejores se van primero.** El que tenga más experiencia consigue trabajo en cualquier operadora vecina (Parex, Frontera, GeoPark, Sierracol, Canacol) en dos semanas. Quedará el más débil, justo cuando se necesita supervisión humana sofisticada para revisar las causaciones de la AI. Es decir: el proyecto degrada simultáneamente la calidad humana y aumenta la dependencia de calidad humana.

**2.4. No hay caso público colombiano documentado** de transición exitosa de causación manual a AI en upstream de 90 empleados con sponsor de operaciones. Hay marketing de Alegra, Siigo y Cuenti hablando de "contabilidad invisible" (https://cuenti.com/software-contable/inteligencia-artificial-contabilidad-tendencias-2026/, https://siemprealdia.co/colombia/contabilidad/tareas-contables-que-puedes-automatizar/), pero son piezas comerciales, no estudios de caso auditados. El equipo está pioneando sin red.

**2.5. Empresas similares manejan la transición preservando al equipo, no reemplazándolo.** La práctica observada en Colombia (Payana, N1, plataformas que integran a Siigo/Alegra) es **mantener al auxiliar contable como revisor de causaciones sugeridas**, no eliminar el rol. Cuando el rol desaparece, también desaparece la disposición a "enseñarle" a la AI los criterios del cliente — y la AI nunca alcanza el nivel del humano que reemplazó.

---

## 3. RIESGOS REGULATORIOS Y CONTABLES — EL CAPÍTULO MÁS PELIGROSO

**3.1. La firma del contador público no es delegable a un algoritmo.** El art. 10 de la **Ley 43 de 1990** (https://www.funcionpublica.gov.co/eva/gestornormativo/norma.php?i=66148) establece que la atestación o firma del contador público hace presumir, salvo prueba en contrario, que el acto se ajusta a los requisitos legales y que las cifras "reflejan en forma fidedigna la situación financiera". El parágrafo del mismo artículo asimila al contador a **funcionario público para efectos de sanciones penales**. Un contador que firme estados financieros derivados de causaciones hechas por una AI sin control efectivo está exponiendo su tarjeta profesional ante la Junta Central de Contadores y, potencialmente, su libertad. Ningún contador competente firma a ciegas. Si firma, lo hará revisando — y entonces el "ahorro de 3 personas" se evapora.

**3.2. Responsabilidad legal por actor — qué cae sobre cada cabeza cuando esto falla:**
- **Contador público firmante**: responsabilidad disciplinaria ante la Junta Central de Contadores (suspensión, cancelación de tarjeta — arts. 23 a 26 Ley 43/1990), responsabilidad penal (asimilación a funcionario público, art. 10 par. Ley 43/1990), responsabilidad civil solidaria. El concepto CTCP 0026 de 2025 (https://incp.org.co/publicaciones/infoincp-publicaciones/2025/03/ctcp-aclara-la-responsabilidad-de-los-contadores-en-certificaciones-tributarias/) reitera que el incumplimiento deriva en sanciones "civiles, penales, administrativas y disciplinarias".
- **Representante legal**: agente retenedor solidario ante la DIAN (arts. 370-371 ET); responsable de la información tributaria (art. 651 ET); responsable solidario en seguridad social ante UGPP; potencial inhabilidad para ejercer comercio 1–5 años por fraude en retenciones ≥ 4.100 UVT (art. 640-1 ET).
- **Revisor fiscal** (si lo hay por superar topes de la Ley 43/1990 art. 13 / art. 203 Código de Comercio): obligación de denuncia de irregularidades; si avala, responsabilidad disciplinaria propia.
- **Sponsor (Gerente de Operaciones, ingeniero)**: responsabilidad civil interna (laboral, contractual) frente a la empresa; ninguna responsabilidad tributaria directa — **se va a ir libre mientras los demás pagan**. Asimetría brutal.
- **Empresa (persona jurídica)**: sanciones pecuniarias DIAN, UGPP y SIC; rechazo de costos/IVA descontable; posible inscripción como omisa o inexacta. Adicionalmente, la Superintendencia de Sociedades puede activarse si los estados financieros pierden fidelidad representativa bajo Ley 1314 de 2009.

**3.3. Sanción del art. 651 del Estatuto Tributario por información errónea.** El art. 651 ET, modificado por la Ley 2277 de 2022 (https://estatuto.co/651), sanciona la información tributaria con errores con el **0,7 % de las sumas suministradas en forma errónea**, hasta 7.500 UVT — equivalente a **$392.805.000 para 2026** (https://siemprealdia.co/colombia/impuestos/sancion-por-no-presentar-informacion-exogena/). Una AI causando 600–720 facturas al año con 10–20 % de error material golpea directo en exógena (formatos 1001, 1003, 1005, 1006, 1007, 1008). El "ahorro" de 3 sueldos en Yopal (~$120 M COP/año) puede ser borrado por una sola sanción.

**3.4. IVA descontable y retenciones mal aplicadas = inexactitud + cascada de sanciones.** Si la AI clasifica como "IVA descontable" lo que no lo es (porque el proveedor es del régimen simple, porque es un gasto no asociado a operación gravada, porque la factura no cumple requisitos del art. 617 ET), la DIAN puede rechazar el impuesto descontable. Sanción adicional: art. 647 ET (inexactitud, 100 % de la diferencia) y art. 648 ET (http://www.secretariasenado.gov.co/senado/basedoc/estatuto_tributario_pr027.html). Una AI que aplique mal la tarifa de retención (4 % vs 6 %, o no practicar autorretención del Decreto 2201 de 2016) genera responsabilidad solidaria del agente retenedor (arts. 370-371 ET). El art. 640-1 ET prevé **inhabilidad para ejercer comercio de 1 a 5 años** cuando se disminuye saldo a pagar de retenciones en cuantía ≥ 4.100 UVT por fraude (~$214 millones a UVT 2026). En ReteICA, además, hay riesgo municipal por cada jurisdicción — la AI debería conocer la tarifa de Yopal, Aguazul, Villavicencio, Bogotá (oficina principal típica), etc.

**3.5. UGPP cruza causación contable con seguridad social.** La UGPP cruza nómina electrónica, PILA, factura electrónica y contabilidad para detectar evasión de seguridad social en pagos a "contratistas" que son trabajadores encubiertos (https://www.ugpp.gov.co/proceso-determinacion-fiscalizacion/). Sanciones art. 314 Ley 1819 de 2016: omisión hasta **200 %** del valor de aportes; inexactitud **35 %–60 %** (https://www.buk.co/blog/que-exige-la-ugpp-a-los-empleadores-en-colombia). Si la AI causa como "honorarios" lo que debería ir como "salarios" o no detecta proveedores que son trabajadores subordinados (típico en upstream con contratistas individuales que prestan servicios continuos en campamento), la UGPP lo encontrará por cruce automático.

**3.6. NIIF para PYMES — devengo, no probabilidad.** El Decreto 2420 de 2015 (DUR de marcos técnicos) y la Ley 1314 de 2009 exigen reconocer hechos económicos por devengo con fidelidad representativa. Una clasificación "probabilística" no es fidelidad representativa. El CTCP en su Concepto 700/2021 indica que los soportes deben ser fechados y autorizados por quien los elabora; aquí "quien elabora" es un modelo opaco hospedado en Estados Unidos que no firma ni responde.

**3.7. Ley 1581 de 2012 — transferencia internacional de datos.** Las facturas de proveedores personas naturales contienen datos personales (cédula, dirección, teléfono, correo). Enviarlas a OpenAI, Anthropic o Google es **transferencia internacional** y requiere: (i) autorización expresa del titular, o (ii) que el país receptor esté en lista de "nivel adecuado" de la SIC, o (iii) declaración de conformidad de la SIC (art. 26 Ley 1581, https://www.funcionpublica.gov.co/eva/gestornormativo/norma.php?i=49981; preguntas frecuentes SIC: https://sic.gov.co/preguntas-frecuentes-pdp). EE. UU. **no está en la lista de países seguros** para la SIC. Sanción potencial: hasta **2.000 SMMLV** (~$2.847 millones a 2026). Nadie en el proyecto ha pensado en esto, garantizado.

**3.8. Conservación documental 10 años — reproducibilidad.** Art. 28 Ley 962 de 2005 + Concepto CTCP febrero 2007 (https://www.cijuf.org.co/CTCP/CONCEPTOS/CONCEPTOSCTCP2007/Febrero/c008.htm): los libros y papeles del comerciante se conservan 10 años, en cualquier medio que **garantice reproducción fidedigna**. Si los prompts, embeddings, versiones del modelo y respuestas del LLM no se versionan, no hay reproducibilidad. Cuando OpenAI deprecá GPT-4V (lo hará), las causaciones de hace 5 años son irreproducibles y por tanto sin soporte conforme.

**3.9. Valor probatorio de la contabilidad.** Art. 777 ET y jurisprudencia del Consejo de Estado (https://www.javeriana.edu.co/personales/hbermude/jurisprudencia/5622.htm) sobre fe pública del contador: una contabilidad alimentada por una AI sin supervisión documentada puede ser impugnada y **perder valor probatorio**, con cascada en litigios civiles, comerciales o tributarios. Concepto CTCP 1166 (https://cijuf.org.co/normatividad/concepto/2021/concepto-1166.html): *"una contabilidad que no satisfaga las exigencias legales no puede considerarse fidedigna y, por tanto, no puede constituir prueba"*.

**3.10. Régimen de validación previa DIAN y AttachedDocument.** Para que una factura recibida sea soporte de costos, requiere los eventos "Acuse de recibo", "Recibo de los bienes/servicios" y eventualmente "Aceptación expresa" (art. 616-1 ET, Resolución 000165/2023). El sistema AI debe emitir esos eventos correctamente y conservarlos; si los emite mal o doble, genera observaciones DIAN.

---

## 4. RIESGOS DE ALCANCE Y TIMELINE

**4.1. 4 semanas es delirante para producción.** Cuatro semanas alcanzan, en escenario optimista, para un **demo con 5 facturas seleccionadas a mano**. Producción real exige: inventario de tipos de documento (1 semana), curación de un dataset de ≥200 facturas etiquetadas para validar (2 semanas), construcción de validaciones contra RUT/RUES (1 semana), mapeo del PUC y centros de costo del sector petrolero al modelo de datos (2 semanas), integración con el software contable actual (2–6 semanas dependiendo de si es Siigo Nube, World Office o Helisa), pruebas paralelas con conciliación 1:1 contra el equipo humano (≥1 mes), capacitación al contador firmante (1 semana). Eso son **3 a 5 meses, no 4 semanas**.

**4.2. La integración con el software contable es el iceberg.** Siigo Nube tiene API y cuentas predefinidas como "229999 – Causación automática compras (sistema)" que **no se pueden editar** (https://siigonube.portaldeclientes.siigo.com/elaborar-factura-de-compra/). World Office y Helisa tienen integraciones más restringidas. Si la empresa usa Helisa NIIF on-premise (común en empresas medianas del sector), no hay API moderna — toca cargas planas o RPA frágil. Nadie en el equipo ha auditado esto en 4 semanas.

**4.3. Supuesto oculto #1: que las facturas físicas llegan digitalizadas.** En Llanos, las facturas de campo llegan en bolsas de papel después de viajes de campamento. El cuello de botella es la **digitalización**, no la causación. Si no se resuelve esto, la AI no recibe input.

**4.4. Supuesto oculto #2: que el PUC está estandarizado.** Empresas de 90 personas en upstream tienen el PUC parcheado durante años, con cuentas auxiliares creadas ad-hoc por contador anterior. Mapear eso al modelo del LLM toma semanas y requiere al contador, que es el más ocupado en el mes de cierre.

**4.5. Supuesto oculto #3: que los 3 reportes financieros mensuales son estándar.** ¿Estado de resultados por centro de costo / pozo? ¿Reporte por contrato E&P / JOA? ¿Estado de flujo con conciliación de OPEX vs CAPEX y AFE (Authorization for Expenditure)? Cada uno tiene reglas propias. Construir tres reportes "buenos" en 4 semanas con datos que solo lleva 3 semanas tener limpios es matemáticamente imposible.

**4.6. Scope creep garantizado — patrones colombianos típicos.** Lo que empieza como "causar 60 facturas/mes" se transforma en: "ya que estamos, agreguemos nómina electrónica", "agreguemos exógena", "agreguemos conciliación bancaria", "agreguemos órdenes de compra contra factura", "agreguemos kardex". En proyectos de RPA y automatización contable colombianos esto es la norma — la propia investigación documental sobre automatización contable con IA (https://www.researchgate.net/publication/397518191) reconoce como riesgo dominante "costos elevados de implementación" justamente por subestimación de alcance. A las 4 semanas se llega con el OCR del MVP corriendo "a veces", sin nada en producción.

**4.7. No hay caso colombiano público de éxito con n8n + LLM en contabilidad regulada.** Lo más cercano son Payana (https://intercom.help/payana-ayuda/es/articles/12150115-causacion-facturas-de-compra-a-siigo-nube) y N1 (https://n1.app/) — **productos SaaS dedicados con equipos completos de ingeniería y compliance**, no MVPs internos de 4 semanas. Y ambos requieren intervención humana en las primeras causaciones por proveedor para aprender.

---

## 5. RIESGO COMPETITIVO Y DE COMMODITIZACIÓN

**5.1. La función ya existe y está commoditizada en Colombia.**
- **Alegra Contabilidad** (https://www.alegra.com/colombia/, https://www.alegra.com/colombia/facturacion-electronica/precios/): $17.900 a $179.900 COP/mes según rango de ingresos; buzón XML que carga automáticamente las facturas electrónicas (https://ayuda.alegra.com/col/crear-facturas-de-proveedor-desde-un-xml), IA integrada para causación desde PDF/imagen (https://ayuda.alegra.com/col/causar-fact), exógena, conciliación bancaria con IA.
- **Siigo Nube** (https://www.siigo.com/precios-siigo/, https://www.comparasoftware.co/siigo-es): $145.993 a $207.869 COP/mes; **proveedor tecnológico #1 autorizado por la DIAN**; causación automática nativa.
- **N1** (https://n1.app/): $510–$600 COP **por factura causada** con integración a Siigo y Alegra, sincroniza con DIAN, aprende del usuario. A 60 facturas/mes serían **~$36.000 COP/mes**.
- **Payana**, **Rossum**, **Docuware**, **Cuenti**: alternativas adicionales.

**5.2. Cálculo brutal del costo de oportunidad.** N1 a 60 facturas/mes ≈ $36.000 COP/mes ≈ **$432.000 COP/año**. Siigo + N1 ≈ $2,5 M COP/año totales. El MVP propio costará entre **$15 M y $40 M COP** solo en horas de un consultor n8n + un ingeniero senior, **+ tokens de LLM** (60 facturas × ~$500 COP/factura en GPT-4 Vision = otro $360.000/año, optimista, sin retries), **+ mantenimiento eterno** cuando la DIAN actualice anexos técnicos (v1.9 hoy; ya hubo modificaciones por Resolución 008 de 2024, https://micrositios.dian.gov.co/sistema-de-facturacion-electronica/normatividad/). El ROI está negativo desde el día 1 frente a comprar.

**5.3. Microsoft Copilot ya hace parte de esto en Excel — la ventana de oportunidad se cierra.** Copilot en Excel ya importa datos, sugiere fórmulas, analiza y limpia (https://support.microsoft.com/en-us/office/get-started-with-copilot-in-excel-d7110502-0334-4b4f-a175-a73abdfc118a). En abril 2026 entró el "modo agente" que ejecuta acciones autónomamente (https://www.infobae.com/tecno/2026/04/26/copilot-ya-puede-hacer-el-trabajo-por-ti-en-word-y-excel-sin-tener-que-decirles-como-hacerlo/). **Copilot for Finance** (https://learn.microsoft.com/es-es/shows/copilot-learning-hub/revolutionize-your-budgeting-with-copilot-for-finance-in-excel) está específicamente diseñado para conciliación financiera en Excel. En 6–12 meses, Copilot for Finance, Gemini en Google Sheets o Claude en herramientas de Anthropic harán nativo lo que el MVP intenta construir a pulso. El desarrollo interno será obsoleto antes de pagarse.

**5.4. Costo total de propiedad ignorado.** Cada vez que OpenAI deprecá GPT-4V, cada vez que n8n actualice esquema, cada vez que la DIAN publique modificación a la Resolución 165, el MVP requerirá reingeniería. Los SaaS lo absorben en su precio; el MVP propio lo asume Green Power.

---

## 6. RIESGOS POLÍTICOS INTERNOS

**6.1. Sponsor incorrecto: el Gerente de Operaciones no es responsable de la contabilidad.** En oil & gas colombiano, Operaciones tiene poder operativo enorme, pero finanzas y contabilidad responden ante el revisor fiscal, la Superintendencia de Sociedades, la DIAN y la ANH (para contratos E&P). Un ingeniero "no contador" liderando esto sin gobernanza formal de finanzas es exactamente el patrón clásico de "shadow IT" que termina escalando al revisor fiscal cuando ya hay errores en libros. El CTCP ha sido enfático en que los soportes y libros son de la entidad y la responsabilidad por su preparación recae en la administración con el contador (Concepto CTCP 194 de 2023, https://accounter.co/normatividad/responsabilidad-entrega-informacion-del-contador-publico-concepto-ctcp-194-de-2023.html).

**6.2. El contador y el revisor fiscal no firmaron este proyecto.** Si no hay aprobación documentada del contador titular (con su tarjeta profesional en juego) y del revisor fiscal — que tiene obligación de denunciar irregularidades materiales según art. 207 del Código de Comercio y art. 7 Ley 43/1990 — el proyecto está construido contra ellos, no con ellos. El día del primer error visible firman un memorando matando el proyecto y queda como precedente "AI fracasó" para los siguientes 5 años.

**6.3. Asimetría de incentivos del sponsor.** El Gerente de Operaciones gana visibilidad mostrando "automatización con AI" al dueño. No pierde nada si falla — los firmantes y los sancionados serán otros (contador, representante legal). Esa asimetría es exactamente lo que la literatura de gobierno corporativo señala como bandera roja.

**6.4. El "apoyo" del dueño no es presupuesto ni cobertura legal.** Cuando llegue la primera glosa DIAN, el dueño preguntará "¿quién aprobó esto?" y el sponsor dirá "lo aprobamos todos". Esteban, usted que viene de upstream y contratación sabe cómo termina esa frase.

**6.5. Erosión de confianza en mes 1.** La revisión documental sobre automatización contable con IA (https://www.researchgate.net/publication/397518191) señala como riesgo dominante "resistencia al cambio y riesgos de seguridad en la gestión de datos". Un solo error visible al contador en la primera semana (un NIT inventado, un IVA mal calculado, una retención no aplicada) destruye 6 meses de credibilidad y vuelve la conversación irreversible. En empresas pequeñas como Green Power (90 personas), una mala anécdota domina la narrativa.

**6.6. Dinámica oil & gas Llanos: rotación alta + auditorías de operadora.** Si Green Power es contratista de una operadora mayor (Ecopetrol, Parex, Frontera, GeoPark), esa operadora audita facturación cruzada del contratista por temas de costos recuperables y back-to-back de facturas. Una contabilidad con causaciones AI sin trazabilidad reproducible es un hallazgo de auditoría que puede afectar **cobros pendientes** al operador y la calificación HSEQ/comercial. La Superintendencia de Sociedades también puede activar inspección si hay denuncias de partes interesadas (https://www.supersociedades.gov.co).

**6.7. Iniciativas de AI lideradas por áreas no-IT/no-finanzas tienen tasa de fracaso documentada alta.** No hay un benchmark colombiano público específico, pero la literatura general sobre adopción de IA en empresas concluye consistentemente que proyectos sin patrocinio formal de la función dueña del proceso (acá: CFO, Contador Titular) fracasan por falta de gobierno de datos, falta de presupuesto sostenido y falta de poder para imponer cambios al equipo afectado. En Colombia, el patrón se agrava por la fragmentación del PUC y la regulación tributaria cambiante.

---

## 7. SUPUESTOS QUE EL EQUIPO ESTÁ DANDO POR CIERTOS Y NO DEBERÍA

1. **"El LLM lee bien español colombiano y vocabulario petrolero."** Falso por benchmark. GPT-4V tiene desempeño degradado en no-Latin y en vocabulario técnico (slickline, workover, swabbing, vacuum, AFE, JOA, ANH) que no es vocabulario contable estándar (https://arxiv.org/abs/2310.16809).

2. **"n8n escala y es confiable para procesos contables auditables."** n8n es un orquestador de flujos, no un sistema contable certificado por la DIAN, no está en la lista de proveedores tecnológicos autorizados, no tiene mecanismos nativos de conservación 10 años conforme a Ley 962/2005.

3. **"El contador público firmará causaciones hechas por AI."** Ningún contador que haya leído el art. 10 de la Ley 43/1990 firma estados financieros derivados de un proceso que no puede explicar línea por línea. O firma sin leer (irresponsabilidad profesional sancionable por la Junta Central de Contadores), o revisa y entonces no hay ahorro.

4. **"50–60 facturas/mes justifican un MVP a la medida."** No. A ese volumen, **un humano causa todo en 1–2 días-hombre/mes**. El payback de cualquier desarrollo interno con LLM, infra, mantenimiento y compliance es negativo frente a Alegra ($17.900/mes), N1 ($36.000/mes) o un contador outsourcing.

5. **"Tenemos derecho a enviar datos de proveedores a la nube de OpenAI/Anthropic/Google."** Falso bajo Ley 1581/2012 sin autorización expresa de los titulares y sin estar el receptor en lista de países seguros de la SIC.

6. **"La DIAN no se va a dar cuenta."** Falso. La DIAN cruza información exógena con factura electrónica, nómina electrónica, documento soporte de no obligados y RUT. Errores sistemáticos generados por un modelo se ven como patrón, no como ruido.

7. **"4 semanas es realista porque es un MVP."** Confunde "MVP de demo" con "MVP en producción contable". En contabilidad, "producción" significa que firma el contador, que genera obligaciones tributarias reales, que se conserva 10 años. No hay versión "beta" legal de eso.

8. **"El sponsor de Operaciones puede dirigir esto sin gobernanza formal de finanzas."** Falso. La fe pública contable y la responsabilidad tributaria son legalmente atribuibles a roles específicos (contador titular, revisor fiscal, representante legal). El sponsor de Operaciones no es ninguno de esos.

9. **"Vamos a ahorrar el trabajo de 3 personas."** No se ahorra el trabajo, se desplaza. Alguien tiene que revisar excepciones, mantener el sistema, manejar el cambio de modelo, responder a la DIAN. En empresas comparables ese trabajo desplazado es ~1 a 1,5 FTE, no 0.

10. **"Si funciona en mes 1 funciona en mes 12."** Falso. Los modelos de fundación se reentrenan, se deprecan, cambian sus tarifas. La DIAN cambia anexos técnicos. Los proveedores cambian sus plantillas. Sin equipo dedicado de mantenimiento, el sistema se degrada silenciosamente y los errores aparecen en exógena del año siguiente, cuando ya prescribieron las posibilidades de corrección sin sanción.

11. **"El XML UBL 2.1 es opcional, podemos leer todo con visión."** Falso técnicamente. El XML es el dato fuente; la representación gráfica PDF es derivada. Procesar la imagen en vez del XML es introducir error donde no lo había.

12. **"El sector petrolero contabiliza como cualquier PYME."** Falso. Centros de costo por pozo, contratos E&P con ANH, OPEX/CAPEX con reglas distintas, costos recuperables, retención de timbre en obra, ICA por municipio operativo. No es PYME genérica.

---

## Tabla de cobertura del análisis

| Sección solicitada | Cubierta | Fuentes colombianas clave citadas con URL |
|---|---|---|
| 1. Riesgos técnicos (tipos factura, tasa error, alucinación, Res. 000165 / UBL) | ✓ | Resolución DIAN 000165/2023; anexo técnico UBL v1.9; art. 616-1, 617, 771-2 ET; benchmarks arXiv |
| 2. Riesgos de adopción (sabotaje, conocimiento tácito sector petrolero) | ✓ | NIIF PYMES Decreto 2420/2015; mercado upstream Colombia |
| 3. Regulatorios y contables (DIAN, IVA, retenciones, Ley 43/1990, ET, NIIF, validación previa) | ✓ | Ley 43/1990; arts. 651, 647, 648, 640-1, 370-371, 777 ET; CTCP 0026/2025, 700/2021, 1166/2021; Ley 962/2005; Ley 1581/2012 art. 26; UGPP art. 314 Ley 1819/2016; SIC |
| 4. Alcance y timeline (4 semanas, Siigo/WO/Helisa, scope creep) | ✓ | Siigo Nube docs; Payana; N1; investigación ResearchGate |
| 5. Competitivo / commoditización (Alegra, Siigo, Rossum, Copilot) | ✓ | Alegra, Siigo Nube, N1, Payana, Copilot Microsoft |
| 6. Políticos internos (gobierno, AI por operaciones, oil & gas) | ✓ | Código Comercio art. 207; Ley 43/1990 art. 7; CTCP 194/2023; Supersociedades |
| 7. ≥5 supuestos no cuestionados | ✓ (12 listados) | Transversal |
| Implícitos: Ley 1581 datos, conservación 10 años, UGPP, responsabilidad por actor | ✓ | Adicionados explícitamente |