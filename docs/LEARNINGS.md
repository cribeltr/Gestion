# LEARNINGS.md — Memoria evolutiva de HHHA

> Esta es la "memoria humana" del proyecto. No es un changelog de features
> (eso vive en build_app.py). Aquí se acumulan **trampas, patrones e
> heurísticas confirmadas** sesión a sesión.
>
> **Reglas:**
> 1. Solo se AÑADE, no se borra. Si algo se invalida, se marca `[REVISADO]` y
>    se explica por qué, pero se conserva.
> 2. Cada sesión que toca el código debe dejar al menos una entrada.
> 3. Antes de codear, lee TODO este archivo.
>
> **Formato de cada entrada:**
> ```
> ## [AAAA-MM-DD] Título corto
> - Disparador: qué cambio/pregunta lo originó.
> - Confirmado: qué se verificó (cómo, con qué prueba).
> - Heurística: la regla generalizable que queda para el futuro.
> - Dónde aplica: archivos/funciones/líneas.
> ```

---

## [2026-05-28] Bug de fechas UTC parcheado a medias

- **Disparador:** revisión general tras el ajuste v0.23 (badges de bitácora).
- **Confirmado:** las fechas se guardan como `YYYY-MM-DD`. El parse seguro
  `new Date(f+'T00:00:00')` se usa en líneas 885, 2092, 3016, 3104, pero se
  omitió en `mpDelMesEjecutada` (línea 1119) y `aplicarEfectosEvento` (línea
  1209), que usan `new Date(f)`. Reproducido en Node con TZ=America/Santiago:
  `new Date('2026-05-01').getMonth()` → 3 (abril) en vez de 4; `'2026-01-01'`
  → año 2025. Impacto: una MP del día 1 se escribe en el mes anterior (1209,
  escritura) y no se detecta como ejecutada en su mes (1119, lectura).
- **Heurística:** cuando arregles un patrón, haz `grep` del patrón completo y
  arréglalo en TODOS los sitios. Un fix aplicado en 4 de 6 lugares es la causa
  raíz de bugs futuros. Para fechas, este repo nunca debe usar `new Date(f)`
  sobre un string `YYYY-MM-DD` crudo.
- **Dónde aplica:** build_app.py líneas 1119 y 1209 (pendientes de fix);
  patrón correcto en 885, 2092, 3016, 3104.

## [2026-05-28] Divergencia recalc vs apply en el estado del equipo

- **Disparador:** misma revisión general.
- **Confirmado:** `recalcEstadoEquipo` (líneas 1097-1100) mapea
  `ev.estado==='baja'` → `'baja'`, pero `aplicarEfectosEvento` (1184-1186) no
  lo contempla y caería en el `else` → `'no_operativo'`. Hoy NO es alcanzable
  porque el formulario de evento no ofrece "baja" como estado resultante; solo
  se llega a baja por la causal MP='Baja' (camino aparte) o por `darDeBaja`.
- **Heurística:** cualquier dato derivado por dos caminos (aplicación en vivo
  vs recálculo tras anular) debe usar la MISMA lógica. Ideal: extraer un solo
  helper `mapearEstado(ev.estado)` y llamarlo desde ambos. Mientras sigan
  duplicados, todo valor nuevo de estado hay que agregarlo en los dos.
- **Dónde aplica:** build_app.py, `recalcEstadoEquipo` y `aplicarEfectosEvento`.

## [2026-05-28] Observación de diseño: badge de lote sin estilo "auto"

- **Disparador:** revisión del cambio v0.23.
- **Confirmado:** en `renderBitacora` (línea 2160) la clase `auto` (borde/fondo
  verde) se aplica a `conciliacion_auto` y `conciliacion`, pero el badge
  ⚡ Lote (`origen==='masivo'`, línea 2167) no la recibe, aun siendo también
  generado por el sistema. Es decisión de diseño, no error.
- **Heurística:** al introducir una distinción visual ("eventos generados por
  el sistema"), decidir explícitamente si TODOS los orígenes automáticos
  (`conciliacion_auto`, `conciliacion`, `masivo`) entran o no, y dejarlo escrito.
- **Dónde aplica:** build_app.py, `renderBitacora`.

## [2026-05-28] El protocolo mismo se endureció

- **Disparador:** el usuario observó que entregué archivos sin confirmar antes
  de actuar — justo lo contrario de lo que pide.
- **Confirmado:** se probó el protocolo nuevo contra el bug de fechas de hoy.
  Resultado: el paso 2 (revisar el programa completo con grep del patrón)
  habría detectado los 2 lugares omitidos. El protocolo, si se sigue, funciona.
- **Heurística:** (1) nunca actuar sin confirmar entendimiento y esperar el "sí"
  del usuario — es la REGLA #0. (2) Hablar siempre en español, breve y sin
  tecnicismos; el usuario es experto en su trabajo, no en computación.
  (3) Simular es obligatorio en cada cambio, no opcional.
- **Dónde aplica:** CLAUDE.md (REGLA #0, pasos 2 y 4, sección de trato).

## [2026-05-28] Regla nueva: pensar como experto, no literal

- **Disparador:** el usuario notó que al pedir "diseño" se le entregaban cambios
  literales y superficiales (ej. intercambiar emojis) en vez de pensamiento de
  diseñador.
- **Confirmado:** se agregó la sección "Pensar como experto, no ejecutar literal"
  a CLAUDE.md y se reforzó el bucle de auto-mejora con el uso diario.
- **Heurística:** ante cualquier pedido (sobre todo diseño), entender el
  propósito real, simular cómo se vive, y proponer con criterio; si la idea del
  usuario no es la mejor para su objetivo, decirlo con respeto. Nunca ejecutar
  la frase al pie de la letra sin comprenderla.
- **Dónde aplica:** CLAUDE.md (sección nueva + "Cómo mejora el sistema con tu uso").

## [2026-05-28] Bug de fechas CORREGIDO (v0.24)

- **Disparador:** instrucción del usuario "revisa, ajusta, implementa, simula".
- **Confirmado:** se aplicó `+'T00:00:00'` en `mpDelMesEjecutada` y
  `aplicarEfectosEvento` (build_app.py y app.html). Simulado en TZ
  America/Santiago: una MP del 2026-05-01 ahora cae en Mayo (antes caía en
  Abril) y se detecta como ejecutada en su mes. Regresión: 6 parseos seguros,
  0 inseguros; los 4 lugares previos quedaron intactos. Versión subida a 0.24.
- **Heurística:** confirmada la regla anterior — al arreglar un patrón, buscarlo
  en todo el archivo. Aquí faltaban 2 de 6 sitios.
- **Dónde aplica:** RESUELTO. Reemplaza el estado "pendiente" de la entrada del
  bug de fechas más arriba.

## [2026-05-28] Integrado el TRASPASO.md del chat constructor

- **Disparador:** el chat que construyó el programa entregó `TRASPASO.md` con
  decisiones de diseño, problemas conocidos y supuestos. Se guardó en
  `docs/TRASPASO.md`.
- **Confirmado:** al cruzarlo con el código, el `TRASPASO.md` NO mencionaba el
  bug de fechas (su hueco principal). Sí aporta info valiosa que no estaba en
  ningún lado, en especial:
  - Eventos auto-completados (v0.21) **no son idempotentes**: reimportar el
    mismo maestro puede duplicar si entremedio se editó el equipo.
  - Sin throttle en clicks → confirmar una MP dos veces seguidas puede crear
    dos registros.
  - Patrón de closures frágil (`VIEWS.equipo` necesita `setTab`); olvidarlo deja
    botones no-op (fue el bug v0.19).
  - 9 supuestos de regla de negocio SIN ratificar (folio opcional, día 15 del
    evento sintético, MP fuera de mes permitida, etc.) → ver §4 de TRASPASO.md.
- **Heurística:** un traspaso escrito por otro agente puede tener huecos;
  siempre cruzarlo contra el código antes de confiar en él.
- **Dónde aplica:** docs/TRASPASO.md (referencia); ítems técnicos pendientes.

## [2026-05-28] Carga de datos reales desde un export XLSX (v0.25)

- **Disparador:** el usuario pidió "entrégame el programa sin ningún dato y usa
  estos", adjuntando un export del propio sistema (hhhaexport…xlsx, hojas
  Eventos / Pendientes / Ciclos correctivos). Tras confirmar (eligió "solo los
  datos del Excel"), se reemplazaron los datos demo por los suyos reales.
- **Confirmado:**
  - El export es una SALIDA con pérdida: NO trae el catálogo de equipos ni la
    programación MP anual. Esos se conservaron del seed previo (893 equipos +
    matriz). Solo se reemplazaron eventos (85) y pendientes (34); tareas a 0.
  - La traducción XLSX→interno invierte EXACTAMENTE el mapeo de `exportExcel()`
    (build_app.py ~4793-4838): cada columna ↔ su campo. Fechas DD-MM-YYYY →
    YYYY-MM-DD por split de texto (NUNCA `new Date` sobre crudo: invariante de
    fechas). Tipo de pendiente: etiqueta → clave (inverso de `TIPO_PENDIENTE`).
  - Los ciclos NO se guardan en el seed: `bootstrap()` los reconstruye desde las
    Solicitudes de trabajo con folio. Verificado en Node: 3 solicitudes → 3
    ciclos abiertos idénticos a la hoja Ciclos. 0 Reparaciones operativas → los
    3 quedan "abierto" (coincide).
  - Los estados de equipo NO se guardan: `recalcEstadoEquipo` los deriva del
    evento de mayor fecha. Simulado: 71 operativo, 5 no_operativo, 817
    desconocido (equipos sin actividad aún); 76 equipos con eventos.
  - Causales C3/C8 (campo `resultado` de MP) se preservan; 'Creado por'=Cristian
    y 'Técnico' vacío en los 85 (sin pérdida).
  - Regresión bug fechas v0.24: 0 MP corridas de mes; una MP del 2026-04-01
    (día 1) se detecta como ejecutada en abril. El fix sigue firme con datos reales.
- **Heurística:**
  1. Un export del sistema sirve para RE-INGESTAR, pero es con pérdida: cruzar
     sus columnas contra la función exportadora y conservar del seed lo que el
     export no trae (catálogo, programación, estados derivados, ciclos).
  2. Preferir reconstruir lo DERIVADO (ciclos, estados) con la lógica del propio
     programa, no inyectarlo, para no divergir.
  3. `init()` ahora respeta `tipo`/`origen` del seed si vienen; antes los derivaba
     del texto y reclasificaba mal 2 pendientes ("Reprogramación MP" con desc
     "abril"). Regla: la carga del seed no debe "adivinar" lo que el dato afirma.
- **Dónde aplica:** seed.json (datos); build_app.py `init()` (respeta
  tipo/origen) + `target` relativo a `__file__` + CHANGELOG v0.25; app.html
  regenerado. Conversor reproducible en `tools/excel_a_seed.py` (verificado:
  reproduce el seed byte a byte); insumo en `docs/origen_datos_20260528.xlsx`.

<!-- Próximas entradas debajo de esta línea -->
