# Hardcore AI Cohorte 2 — Estación 1

**Autor:** ESTEBAN — Gerente de Operaciones, Green Power S.A.S.
**Empresa:** Green Power S.A.S. (E&P hidrocarburos, Llanos Orientales, Colombia, ~90 empleados)
**Plantilla utilizada:** Internal Solution Brief (caso de uso corporativo)
**Fecha de entrega:** 13 de mayo de 2026

---

## Contenido de esta entrega

| Archivo | Descripción |
|---|---|
| [`green-power-isb.md`](./green-power-isb.md) | **Internal Solution Brief completo** — 9 secciones del template + anexos. Es el artefacto principal de la entrega. |
| [`research-validacion.md`](./research-validacion.md) | **Deep Research de validación** — análisis de casos comparables, tecnologías, benchmark financiero, regulación colombiana (DIAN, Habeas Data), y evidencia de adopción en el sector. |
| [`research-critica.md`](./research-critica.md) | **Deep Research de crítica** — auditoría escéptica del proyecto: 7 secciones de riesgos (técnicos, adopción, regulatorios, alcance, competitivo, políticos, supuestos cuestionados) con fuentes legales colombianas citadas. |

---

## Resumen ejecutivo del proyecto

**Iniciativa:** Célula AI Green Power — Fase 1.

**MVP a 4 semanas:** Copiloto AI de pre-causación financiera con humano-en-el-loop. Extrae datos de facturas de proveedores (XML UBL primario, PDF/imagen como fallback con Claude Sonnet 4.6), pre-llena la propuesta de causación en el formato Excel contable de Green Power, y la entrega al equipo financiero para validación y firma del contador titular.

**Volumen objetivo:** 50–60 facturas/mes + 3 reportes financieros mensuales recurrentes.

**Stack técnico:** n8n (orquestación, ya operativo) + Claude Sonnet 4.6 vía nodo nativo + parser determinístico de XML UBL 2.1 + Excel como sistema de registro contable.

**Costo operativo proyectado:** USD 55–130/mes (COP 220.000–520.000/mes).

**Ahorro proyectado vs proceso manual actual:** COP 1.9M–2.6M/mes una vez en producción estable (Fase 2).

---

## Decisión de diseño clave

El MVP fue **re-encuadrado** tras estudiar el Deep Research de crítica para evitar los modos de fallo más caros documentados en la literatura colombiana:

| Crítica recibida | Decisión de diseño |
|---|---|
| "Reemplazar a 3 personas = sabotaje + pérdida de conocimiento tácito" | Las 3 personas se mantienen como **validadoras/tutoras del modelo**, no se eliminan roles. |
| "El XML UBL no se lee con visión, eso es absurdo técnicamente" | Parser determinístico del XML UBL como **fuente primaria**. LLM con visión solo como fallback. |
| "La fe pública contable es indelegable a un algoritmo (Ley 43/1990)" | El contador titular **firma cada lote**. El sistema solo pre-llena propuestas. |
| "Sponsor de Operaciones sin gobernanza de finanzas es shadow IT" | **Comité semanal formal**: Operaciones + Contador + Revisor Fiscal + Dueño. |
| "4 semanas es delirante para producción contable" | El MVP entrega **prototipo con human-in-the-loop al 100%**, no producción. Roadmap explícito de 3–5 meses para Fase 2. |
| "Construir vs comprar — Alegra/N1 ya existen" | Justificación abierta: el MVP construye el dominio upstream (centros de costo por pozo, contratos JOA, costos recuperables ANH) que las herramientas comerciales no cubren out-of-the-box. |

---

## Roadmap multi-fase

| Fase | Alcance | Duración | Estado |
|---|---|---|---|
| **Fase 1 — MVP** | Pre-causación AI con human-in-the-loop, prototipo | 4 semanas | EN ENTREGA |
| **Fase 2 — Producción** | Integración ERP, escalamiento gradual, DPIA aprobado | 3–5 meses | Pendiente cierre Fase 1 |
| **Fase 3 — Expansión finanzas** | Conciliación bancaria, exógena, nómina electrónica | 4–6 meses | Pendiente |
| **Fase 4 — Otras áreas** | Vigilancia regulatoria, aplicaciones de campo, yacimientos | Por definir | Pendiente |

---

## Cómo leer esta entrega

1. **Empezar por** [`green-power-isb.md`](./green-power-isb.md) — es el artefacto principal y autocontenido.
2. **Profundizar en** [`research-validacion.md`](./research-validacion.md) para entender la evidencia que sustenta las decisiones técnicas y los benchmarks financieros.
3. **Cerrar con** [`research-critica.md`](./research-critica.md) para entender los riesgos identificados y por qué el MVP fue re-encuadrado como está.

---

*Entrega para Hardcore AI by 30X — Cohorte 2 — Estación 1*
