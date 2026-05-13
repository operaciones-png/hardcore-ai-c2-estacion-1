# Presentación Estación 1 — Green Power Célula AI Fase 1

**Expositor:** Juanda — Gerente de Operaciones, Green Power S.A.S.
**Duración estimada:** 8–10 minutos
**Formato:** guion de exposición con notas de soporte

---

## 1. Contexto en una frase (30 seg)

> "Soy ingeniero mecánico, Gerente de Operaciones de Green Power, una empresa de exploración y explotación de hidrocarburos en los Llanos Orientales con 90 empleados. La iniciativa que traigo es montar una **Célula AI interna**, y la Fase 1 ataca el dolor más caro y repetitivo del back-office: la causación de facturas de proveedores y los reportes financieros mensuales."

---

## 2. El problema en números (1 min)

- **50 a 60 facturas mensuales** procesadas manualmente en Excel, sin ERP.
- **3 personas dedicadas** a digitar, validar y consolidar.
- **70% de su tiempo** en tareas repetitivas de bajo valor.
- **Costo escondido estimado: COP 2.4 a 3.1 millones al mes** según benchmarks APQC y Ardent Partners (USD 10–13 por factura para empresas sin ERP).
- Cierre mensual se entrega **5 a 10 días tarde**.
- Riesgo regulatorio real: una sanción DIAN por información exógena errónea puede llegar a **COP 392 millones** (art. 651 ET).

**Mensaje clave:** "Esto no es un problema de productividad, es un problema de exposición legal y de costo de oportunidad del talento."

---

## 3. Por qué elegí Internal Solution Brief y no Product Vision Board (30 seg)

> "Mi caso no es construir un producto para vender, es resolver un dolor interno de mi empresa. Por eso uso la plantilla ISB. Soy el sponsor técnico y cuento con respaldo del dueño y del área financiera desde el día 1."

---

## 4. La trampa que esquivé — la importancia de la crítica (2 min)

> "Lo más valioso del ejercicio no fue el Deep Research que validó la idea, fue el Deep Research **crítico** que casi mata el proyecto."

**Cinco hallazgos críticos que cambiaron el diseño:**

1. **"No leas la imagen del PDF con un LLM si tienes el XML"** — la factura electrónica colombiana (Resolución DIAN 165/2023) viene en XML UBL 2.1 firmado y validado por DIAN. Procesar la imagen cuando ya tienes el XML estructurado es reemplazar un parser determinístico por una alucinación probabilística. **Corrección:** parser determinístico del XML como fuente primaria, AI con visión solo como fallback para POS, papel y fotos de campo.

2. **"Reemplazar a las 3 personas = sabotaje pasivo garantizado + pérdida de conocimiento tácito"** — En upstream petrolero el auxiliar contable sabe distinguir OPEX vs CAPEX, costos recuperables ANH, retenciones por municipio (Yopal, Aguazul, Tauramena son distintos). Eso no lo aprende un LLM mirando un PDF. **Corrección:** las 3 personas pasan de digitadoras a **validadoras/tutoras del modelo**, no se eliminan roles.

3. **"La fe pública contable no es delegable a un algoritmo"** — Ley 43 de 1990, artículo 10. El contador firma con su tarjeta profesional y un error de causación es responsabilidad personal independiente del medio que lo produjo. **Corrección:** el contador titular firma cada lote, el sistema solo pre-llena propuestas, comité semanal formal con Operaciones + Contador + Revisor Fiscal + Dueño.

4. **"4 semanas es delirante para producción contable"** — En contabilidad "producción" significa firma del contador, obligaciones tributarias reales, conservación 10 años. No hay versión beta legal de eso. **Corrección:** el MVP entrega prototipo con human-in-the-loop al 100%, NO producción. Roadmap explícito de 3 a 5 meses para Fase 2.

5. **"Construir vs comprar — Alegra y N1 ya hacen esto"** — Alegra Contabilidad cuesta COP 17.900/mes, N1 cobra COP 600 por factura (~COP 36.000/mes para 60 facturas). **Mi respuesta:** justifico construir porque las herramientas comerciales no manejan out-of-the-box el dominio upstream — centros de costo por pozo, contratos JOA con ANH, costos recuperables, retención de timbre en obra. Si solo necesitara causación genérica, compraría. Necesito el dominio sectorial.

**Mensaje clave:** "Esta es la diferencia entre un MVP de demo y un MVP que aguanta una auditoría DIAN."

---

## 5. La solución re-encuadrada — qué construyo en 4 semanas (1.5 min)

> "Un **copiloto AI de pre-causación con humano en el loop**."

**Flujo:**
1. Llega una factura por email, carpeta compartida o WhatsApp.
2. **n8n** detecta el tipo:
   - Si tiene XML UBL → parser determinístico extrae todo lo firmado por DIAN.
   - Si no → Claude Sonnet 4.6 con visión extrae los campos vía prompt estructurado.
3. **Validación cruzada obligatoria:** NIT contra RUT DIAN en línea + histórico del proveedor + reglas regex.
4. **Aplicación de reglas fiscales:** retenciones por municipio, calidad tributaria del proveedor, clasificación OPEX/CAPEX por centro de costo.
5. Se genera una **propuesta de causación pre-llenada** en el Excel actual.
6. Una de las 3 personas (ahora validadora) revisa, corrige si necesario, aprueba.
7. Se registra evento auditable: hash del documento + JSON del modelo + firma del validador.
8. Al cierre de mes, n8n consolida los 3 reportes recurrentes para que el contador titular firme.

**Stack:** n8n (ya operativo en Green Power) + Claude Sonnet 4.6 vía nodo nativo + Excel actual.

**Costo operativo proyectado: USD 55 a 130 al mes.** Esto sustituye un costo manual de COP 2.4 a 3.1 millones al mes.

---

## 6. Cómo se ve el éxito (1 min)

| Métrica | Hoy | Target MVP (4 sem) | Target 6 meses |
|---|---|---|---|
| Tiempo por factura | 15–25 min | ≤5 min | ≤3 min |
| Facturas con XML procesadas sin AI | 0% | ≥80% | ≥95% |
| Accuracy extracción (no-XML) | N/A | ≥90% en piloto | ≥95% en producción |
| Cierre mensual | 5–10 días | ≤5 días | ≤3 días |
| Trazabilidad auditable por causación | 0% | 100% | 100% |
| FTE liberados a control y auditoría | 0 | 0.5 | 1.5 |

**Definición de éxito del MVP:** prototipo corriendo sobre 20–30 facturas reales, 100% de validación humana, comité semanal funcionando, DPIA en borrador, carta del contador titular para avanzar a Fase 2.

---

## 7. Riesgos que asumo conscientemente (1 min)

> "No vengo a vender humo. Documenté 10 riesgos con probabilidad, impacto y mitigación. Los tres que más me preocupan:"

1. **Alucinación silenciosa del LLM** (probabilidad media, impacto alto). Mitigación: validación cruzada obligatoria + revisión humana 100% en el MVP.
2. **Sanción DIAN por información exógena errónea** (probabilidad baja en MVP / media en producción sin controles, impacto catastrófico). Mitigación: trazabilidad inmutable + escalamiento gradual.
3. **Transferencia internacional de datos personales** (Ley 1581/2012). Mitigación: DPIA antes de producción, regiones LATAM, DPA con Anthropic.

---

## 8. Roadmap multi-fase (30 seg)

> "Soy explícito en que esto es Fase 1. La Célula AI va a tener más fases:"

- **Fase 1 — MVP:** lo que les estoy presentando hoy (4 semanas).
- **Fase 2 — Producción:** integración ERP, escalamiento gradual, DPIA aprobado (3–5 meses).
- **Fase 3 — Expansión finanzas:** conciliación bancaria, exógena, nómina electrónica.
- **Fase 4 — Otras áreas:** vigilancia regulatoria de vencimientos, aplicaciones de campo, yacimientos.

---

## 9. Cierre (30 seg)

> "Tres aprendizajes me llevo de esta Estación 1:"

1. **El Deep Research crítico vale más que el de validación.** Sin él habría llegado con un proyecto que el primer error mata.
2. **El alcance correcto no es el más grande, es el más defendible.** Pasé de 'AI para todo el back-office en 4 semanas' a 'copiloto con humano en el loop para causación, con roadmap explícito'.
3. **AI en contabilidad colombiana no es un problema técnico, es un problema de gobierno.** La tecnología existe. Lo que define el éxito es quién firma, quién revisa, quién audita.

> "Listo para Estación 2. Gracias."

---

## Anexo: preguntas probables del tutor y respuestas

**P: ¿Por qué no compraste Alegra/Siigo y ya?**
R: Lo evalué. Alegra y N1 hacen causación genérica muy bien. No manejan dominio upstream (centros de costo por pozo, contratos JOA, costos recuperables ANH). Mi MVP construye esa capa sectorial sobre el flujo genérico. Si la capa sectorial fuera trivial, compraría.

**P: ¿Qué pasa si en 6 meses Microsoft Copilot hace esto nativo en Excel?**
R: El MVP construye el **dato auditable, los logs, las reglas fiscales colombianas y el dominio upstream**, no el motor LLM. Si emerge mejor herramienta, los assets construidos siguen siendo válidos. Cambio el motor, conservo el resto.

**P: ¿Cómo van a reaccionar las 3 personas?**
R: Conversación abierta desde día 1: no se eliminan roles, se transforman a validadoras/tutoras. El dueño respalda continuidad laboral durante Fase 1 y Fase 2. El que no se sienta cómodo con el nuevo rol tiene apoyo de la empresa, pero la decisión técnica fue mantener al equipo, no reemplazarlo.

**P: ¿Y si el contador no firma?**
R: El proyecto no avanza a Fase 2. Por eso el comité semanal lo incluye desde la semana 1 del MVP y la entrega de Fase 1 contempla explícitamente una carta de respaldo del contador titular como criterio de éxito.

**P: ¿Cuál es tu mayor incertidumbre?**
R: La accuracy real del LLM sobre facturas de proveedores rurales con fotos torcidas, manuscritas o escaneadas con celular. Por eso el piloto de las 4 semanas es sobre 20–30 facturas reales del último mes, mezclando todos los tipos de documento. Si el accuracy en no-XML cae por debajo de 80%, replanteamos el alcance de Fase 2.

---

*Hardcore AI by 30X — Cohorte 2 — Estación 1 — Mayo 2026*
