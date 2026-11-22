---
trigger: model_decision
description: Conecta automáticamente el archivo app-brief.md con openspec-explore cuando se inicia una exploración o se trabaja con requisitos funcionales del brief.
---

# Conexión `app-brief.md` → `openspec-explore`

Cuando el usuario inicie `openspec-explore` (o pida explorar la arquitectura, diseño técnico o especificaciones a partir de requisitos funcionales), sigue estas reglas:

## 1. Detección automática del brief
- Busca `app-brief.md` en la raíz del workspace (o en la ruta que indique el usuario) antes de empezar a hacer preguntas de descubrimiento.
- Si `app-brief.md` existe, léelo completo al inicio de la exploración y úsalo como fuente de verdad funcional.
- Si no existe y tampoco hay requisitos definidos, sugiere ejecutar primero el workflow `dept-app-brief` para recopilar el brief funcional antes de explorar la solución técnica.

## 2. Cómo usar cada sección en `openspec-explore`
No repitas la entrevista funcional ni vuelvas a preguntar lo que ya está definido en `app-brief.md` (salvo contradicciones o ambigüedades bloqueantes). Traduce cada sección a decisiones de exploración:

- **Sección 1 (`Objetivo y Alcance`):** Usa el problema, usuarios objetivo y la lista de *Fuera de alcance (v1)* como límites estrictos para no diseñar ni proponer alcance extra.
- **Sección 2 (`Roles y Permisos`):** Explora la estrategia de autenticación y control de acceso (RBAC/permisos por dominio, lectura, escritura, borrado y exportación).
- **Sección 3 (`Dominios y Modelo de Datos`):** Analiza el modelo de datos, relaciones (`1:N`, `N:M`), restricciones (`obligatorio`, `único`), validaciones, transiciones de estado y política de borrado lógico vs. físico.
- **Sección 4 (`Formularios y Pantallas`):** Define la estructura de vistas (`CRUD`, `Consulta`, `Detalle`), filtros, columnas y acciones masivas o cambios de estado.
- **Sección 5 (`Informes y Carga/Descarga de Datos`):** Explora el diseño técnico de consultas/agregaciones, generación de informes (PDF/Excel) y el flujo de importación/exportación CSV/Excel (validación de columnas, duplicados y manejo de errores).
- **Sección 6 (`Dudas Abiertas y Puntos a Explorar`):** Toma esta lista como la **agenda prioritaria** de `openspec-explore`.

## 3. Criterio de cierre de la exploración
- Verifica que cada punto de la Sección 6 de `app-brief.md` quede resuelto con una decisión técnica explícita o descartado conscientemente.
- Mantén la trazabilidad entre las decisiones técnicas exploradas y las entidades, roles y reglas de negocio de `app-brief.md` para facilitar la siguiente fase de especificación en OpenSpec.
