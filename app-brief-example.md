# Brief Funcional: Gestión de Facturas Departamental

## 1. Objetivo y Alcance
- **Problema a resolver:** Sustituir las hojas de cálculo actuales para centralizar la gestión, seguimiento y control de facturas emitidas (a clientes) y facturas recibidas (de proveedores), evitando errores manuales, duplicados y falta de trazabilidad en aprobaciones y cobros/pagos.
- **Usuarios objetivo:** Departamento de Administración (perfiles Administrativo, Responsable y Consulta).
- **Fuera de alcance (v1):**
  - Conexión oficial con Hacienda (SII, Verifactu, Facturae).
  - Pasarela de pagos online, domiciliación bancaria (SEPA) y conciliación bancaria automática.
  - Envío automático de facturas por correo electrónico desde la aplicación.
  - Gestión multiempresa o multidivisa (v1 asume una única empresa y moneda EUR).

## 2. Roles y Permisos
| Rol | Dominios accesibles | Permisos (Lectura / Escritura / Borrado / Exportar) |
|---|---|---|
| **Administrativo** | Facturas, Clientes, Proveedores | Lectura, creación y edición de borradores, envío a aprobación, importación CSV. Sin anulación ni exportación avanzada (a confirmar en exploración). |
| **Responsable** | Facturas, Clientes, Proveedores, Informes | Lectura, validación/aprobación (y acciones masivas), marcado de cobro/pago, anulación (borrado lógico), exportación Excel/CSV/PDF. |
| **Consulta** | Facturas, Clientes, Proveedores, Informes | Solo lectura y descarga de PDFs/informes (sin creación, edición, borrado ni importación). |

## 3. Dominios y Modelo de Datos
### Cliente / Proveedor
- **Campos:** `nif_cif` (texto, obligatorio, único), `razon_social` (texto, obligatorio), `email` (texto, opcional), `telefono` (texto, opcional), `direccion` (texto, opcional), `activo` (booleano, obligatorio, por defecto `true`). *(Pueden modelarse como entidades separadas o unificadas con rol Cliente/Proveedor).*
- **Relaciones:** `1:N` con `Factura` (un cliente tiene muchas facturas emitidas; un proveedor tiene muchas facturas recibidas).
- **Estados / Ciclo de vida:** `Activo` ↔ `Inactivo`.
- **Reglas de negocio y borrado:** Borrado lógico mediante desactivación (`activo = false`). No se eliminan registros con historial de facturas. El `nif_cif` debe ser único.

### Factura
- **Campos:** `tipo` (`Emitida` / `Recibida`, obligatorio), `numero` (texto, único por tipo/serie; generado automáticamente de forma correlativa en emitidas al aprobar/emitir), `fecha_emision` (fecha, obligatorio), `fecha_vencimiento` (fecha, obligatorio), `concepto_o_lineas` (texto/líneas, obligatorio), `base_imponible` (numérico, obligatorio), `porcentaje_iva` (numérico, obligatorio), `total` (numérico, calculado/obligatorio), `estado` (enum, obligatorio), `documento_pdf` (archivo, opcional/recomendado en recibidas).
- **Relaciones:** `N:1` con `Cliente` (si es `Emitida`) o `Proveedor` (si es `Recibida`).
- **Estados / Ciclo de vida:** `Borrador` → `Pendiente de aprobación` → `Aprobada / Emitida` → `Pagada / Cobrada` → `Anulada`.
- **Reglas de negocio y borrado:**
  - Borrado lógico mediante transición a estado `Anulada` (nunca borrado físico).
  - Una factura en estado `Aprobada / Emitida` (o posterior) queda **bloqueada para edición**; solo puede pasar a `Pagada / Cobrada` o `Anulada`.
  - Numeración automática correlativa en facturas emitidas.

## 4. Formularios y Pantallas
- **Panel Resumen Inicial (Dashboard):** Vista de entrada con indicadores y accesos rápidos a facturas `Pendiente de aprobación` y facturas vencidas sin cobrar/pagar.
- **Listado de Facturas:** Consulta tabular con filtros por `tipo` (`Emitida`/`Recibida`), `estado`, rango de fechas (`fecha_emision` / `fecha_vencimiento`) y `cliente`/`proveedor`. Incluye selección múltiple y **acciones masivas** (aprobar en lote, marcar como pagada/cobrada en lote).
- **Formulario / Detalle de Factura:** Alta y edición en estado `Borrador`, adjuntado de PDF (en recibidas), visualización de totales y botones de transición de estado según rol (`Enviar a aprobación`, `Aprobar/Emitir`, `Marcar como cobrada/pagada`, `Anular`). Modo solo lectura tras emitir.
- **Directorio de Clientes y Proveedores:** Listado con buscador por `nif_cif` o `razon_social`, formulario CRUD y acción de activar/desactivar.

## 5. Informes y Carga/Descarga de Datos
### Informes
- **Resumen de IVA (Excel/CSV):** Cálculo y desglose de IVA repercutido (emitidas) vs. IVA soportado (recibidas) filtrable por rango de fechas / trimestre.
- **Listado de Facturas (Excel/CSV):** Exportación del listado filtrado respetando los filtros activos de pantalla.
- **Factura Emitida (PDF):** Generación y descarga individual del documento PDF de una factura emitida.

### Importación / Exportación (CSV/Excel)
- **Importación CSV (Clientes, Proveedores y Facturas):**
  - **Separador:** `;` (punto y coma).
  - **Columnas esperadas:** Al menos `nif_cif;razon_social` (para terceros) y `tipo;numero;fecha_emision;fecha_vencimiento;nif_cif;base_imponible;porcentaje_iva` (para facturas).
  - **Política ante duplicados y errores:** Si un cliente/proveedor ya existe por `nif_cif`, se actualizan sus datos. Si falta alguna columna obligatoria o hay cualquier error de validación en el archivo, se **cancela la carga completa** (operación atómica) mostrando el detalle de los errores encontrados.

## 6. Dudas Abiertas y Puntos a Explorar (`openspec-explore`)
- **Numeración correlativa de emitidas:** Definir el formato exacto de serie/año (ej. `FAC-2026-0001`), en qué momento exacto se asigna el número (al aprobar/emitir vs. al crear borrador) para evitar huecos y cómo evitar colisiones concurrentes.
- **Desglose de líneas vs. importe global:** Confirmar si una factura puede tener múltiples líneas con distintos tipos de IVA en v1 o un único tipo de IVA global.
- **Permisos granulares por rol:** Precisar si `Administrativo` puede crear/editar `Clientes` y `Proveedores` y si solo `Responsable` puede `Anular` facturas emitidas.
- **PDFs (adjuntos y generación):** Definir dónde y cómo se almacenan los PDFs de facturas recibidas y qué librería/plantilla se usará para generar el PDF de facturas emitidas.
- **Importación de histórico de facturas:** Definir la clave única de deduplicación en carga CSV de facturas (`numero` + `tipo` + `nif_cif`) y si se permite importar facturas directamente en estado `Pagada / Cobrada`.
- **Autenticación y gestión de usuarios:** Definir si el acceso usará login local o integración con directorio corporativo (SSO).
