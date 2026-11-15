# Esquema de Aficiones (Huerto y Cocina) — contenido remoto

Paquete preparado el 15 de noviembre de 2026 (hora de Chile). Rama local `staging-aficiones`, **sin publicar**.
Este archivo es para el prompt de Cursor que hará que la app lea Aficiones (Naturaleza, Huerto, Cocina, Niveles).
No está en el manifiesto y la app no lo descarga.

Formato compatible con lo que ya lee la app 1.0.5 (`AficionesContent.kt`, `docs/formato-aficiones.md` del repo de la app):
todo campo nuevo es **opcional y adicional** (la app usa `ignoreUnknownKeys = true`).

## 1. Archivos y manifiesto

| Ruta (relativa a `content/`) | Contenido | `minAppVersionCode` de la entrada |
|---|---|---|
| `huerto/siembras.json` | 21 cultivos | 38 (1.0.5) |
| `huerto/flores.json` | 13 especies, 118 variantes de color | 38 (1.0.5) |
| `cocina/recetas.json` | 234 recetas | 39 (1.0.6, la que muestre avisos de ingredientes pendientes) |
| `huerto/images/siembras/<id>.webp` | 21 dibujos | 39 |
| `huerto/images/flores/<idVariante>.webp` | 118 dibujos | 39 |
| `cocina/images/<id>.webp` | 234 dibujos | 39 |

- `contentVersion` sube de 7 a **8**. El `minAppVersionCode` **global sigue en 25** y las entradas existentes
  (catálogo, clima, eventos, créditos, imágenes de criaturas) no cambian ni llevan mínimo propio:
  1.0.3 y 1.0.4 siguen recibiendo catálogo y calendario.
- Cada entrada nueva lleva `minAppVersionCode` propio (campo ya soportado por `ContentFileEntry` en 1.0.5).

### Avisos para la app (importante)

1. **Apps 1.0.3/1.0.4 (≤37) no conocen el mínimo por archivo**: descargan todos los archivos del manifiesto,
   incluidos los nuevos (~45 MiB de imágenes). No se rompen (no los leen), pero gastan datos y una descarga fallida
   aborta su sincronización entera. Si eso preocupa, ver la alternativa en el informe (imágenes con pérdida, ~9 MiB).
2. **Al actualizar la app con el mismo `contentVersion`**, `ContentPackage.sync()` responde `UNCHANGED` y nunca baja
   los archivos que antes se saltó por `minAppVersionCode`. La 1.0.6 debe corregirlo: si existe en el manifiesto
   instalado algún archivo con `minAppVersionCode <= versionCode` que falte en el overlay, sincronizar igual
   (o guardar el `versionCode` con el que se sincronizó y forzar una sincronización cuando cambie).
3. `CreatureImagePaths` solo acepta rutas que empiezan por `catalog/images/`. Para Aficiones, cargar la imagen con
   `library.effectiveBytes(imagen)` validando `ContentPackage.isSafePath` y los prefijos `huerto/images/` y `cocina/images/`.
   Si el archivo no está (versión que se lo saltó o sin red), mostrar el ícono de la sección, como hoy en criaturas.

## 2. Imágenes

Dibujos originales (acuarela naturalista, no arte del juego), tratados igual que las criaturas publicadas:
**WebP sin pérdida, 512×512, RGBA con fondo transparente**, sin marco horneado. El marco de papel
«cuaderno de naturalista» lo dibuja la app (el mismo componente de las criaturas). Nombre de archivo = `id`.

## 3. `huerto/siembras.json`

Objeto `{ "items": [ ... ] }`.

| Campo | Tipo | Siempre | Notas |
|---|---|---|---|
| `id` | string | sí | slug español (`tomate`, `granos-de-cacao`). Lo usan las recetas en `idSiembra`. |
| `nombre` | string | sí | nombre oficial en español |
| `nivelSiembra` | int 1–13 | sí | |
| `precioSemilla` | int | sí | |
| `fuenteSemilla` | string | sí | `"Tienda de Blanc"` o `"Evento"` |
| `tiempoCrecimientoMinutos` | int > 0 | sí | para el temporizador |
| `precioVentaEstrellas` | int[] | no | venta por estrella, desde 1★. **Puede tener 3 valores** (Trigo, Lechuga: solo hay datos hasta 3★). Falta en eventos, Granos de cacao y Aguacate (conflicto de fuentes). |
| `categoria` | string | sí | `"Común"` o `"Evento"` |
| `evento` | string | si es evento | nombre español del evento |
| `certificable` | bool | si es evento | siempre `false` en eventos; ausente = sin restricción conocida |
| `maestriaCosechas` | string | no | texto listo: `"Novato: 1200 · Experto: 3600 · Maestro: 7200 cosechas"` (7 cultivos) |
| `imagen` | string | sí | `huerto/images/siembras/<id>.webp` |

No existen (y no deben inventarse): rendimiento por cosecha, rebrote, temporada.

## 4. `huerto/flores.json`

| Campo | Tipo | Siempre | Notas |
|---|---|---|---|
| `id`, `nombre` | string | sí | especie (`margarita`, «Margarita») |
| `nivelSiembra` | int | sí | |
| `tiempoCrecimientoMinutos` | int | sí | **nuevo** (la 1.0.5 lo ignora) |
| `precioSemilla` | int | sí | **nuevo**, semilla de tienda |
| `categoria` | string | sí | `"Común"` (las 13 son regulares) |
| `imagen` | string | sí | imagen de la especie = la de su primera variante (no hay archivo aparte) |
| `variantes[]` | lista | sí | |
| `variantes[].id` | string | sí | **nuevo**, `margarita-roja` |
| `variantes[].nombre` | string | sí | **nuevo**, nombre oficial completo («Margarita roja») |
| `variantes[].color` | string | sí | adjetivo oficial concordado («roja», «rosado», «durazno», «crepúsculo», «luminiscente») |
| `variantes[].origen` | string | sí | `"tienda"` o `"cruce"` |
| `variantes[].estrellas` | int | sí | estrella base |
| `variantes[].precioVenta` | int | sí | venta de la flor |
| `variantes[].imagen` | string | sí | **nuevo**, `huerto/images/flores/<idVariante>.webp` |

Kotlin para la 1.0.6: añadir a `FlorVariante` `id: String? = null`, `nombre: String? = null`, `imagen: String? = null`,
y a `FlorEntry` `tiempoCrecimientoMinutos: Int? = null`, `precioSemilla: Int? = null`, `categoria: String? = null`.
La ficha de la flor debe mostrar una fila por variante con su dibujo.

## 5. `cocina/recetas.json`

| Campo | Tipo | Siempre | Notas |
|---|---|---|---|
| `id`, `nombre` | string | sí | nombre oficial español |
| `nivelCocina` | int 1–13 | sí | |
| `energiaEstrellas` | int[5] | no | Heartonote |
| `precioVentaEstrellas` | int[5] | no | Heartonote (falta en Bebida extraña y Comida extraña) |
| `ingredientes` | lista | sí | puede venir vacía si ningún ingrediente tiene nombre español verificado |
| `ingredientesCompletos` | bool | no | **nuevo**. Solo aparece como `false`: faltan ingredientes sin nombre español verificado |
| `ingredientesPendientes` | int | no | **nuevo**. Cuántos faltan |
| `origen` | string | no | **nuevo**. Texto listo para mostrar cuando el plato no tiene ingredientes propios (Bebida extraña, Comida extraña: «Se obtiene al quemar o cocinar mal cualquier bebida/comida»). Con `origen`, `ingredientes` viene vacía y la receta está completa. Fuente: confirmado por el usuario en el juego, 15-11-2026. |
| `categoria` | string | sí | `"Común"` o `"Evento"` |
| `evento` | string | si es evento | nombre del evento como lo da Heartodex en español |
| `certificable` | bool | si es evento | siempre `false` en eventos |
| `maestria` | string | no | `"Novato: 160 · Experto: 480 · Maestro: 960 veces"` |
| `imagen` | string | sí | `cocina/images/<id>.webp` |

Ingrediente fijo: `{ "idSiembra"?: "tomate", "idReceta"?: "mermelada-de-fresa", "nombre": "Tomate", "cantidad"?: 2 }`
- `nombre` siempre viene (aunque haya `idSiembra`).
- `idReceta` es **nuevo**: el ingrediente es otro plato del mismo archivo (para enlazar a su ficha).
- `cantidad` falta si ninguna fuente la da.

Ingrediente «elige N»: `{ "elegir": 2, "opciones": [ { "idSiembra": "rabano", "nombre": "Rábano" }, ... ] }`.

**Regla de UI obligatoria (1.0.6):** si `ingredientesCompletos == false`, mostrar bajo la lista
«Faltan N ingredientes por confirmar» y no presentar la lista como completa. Los ingredientes pendientes
no se muestran ni en inglés ni con un nombre provisional.

Kotlin para la 1.0.6: añadir a `RecetaIngrediente` `idReceta: String? = null` y a `RecetaEntry`
`ingredientesCompletos: Boolean = true`, `ingredientesPendientes: Int = 0`, `certificable: Boolean? = null`, `origen: String? = null` (si viene, mostrarlo en lugar de la lista de ingredientes);
a `SiembraEntry` `certificable: Boolean? = null`.

## 6. Pantalla Aficiones (para el prompt de Cursor)

- **Naturaleza**: lo de hoy (catálogo de criaturas `catalog/creatures.json`).
- **Huerto**: pestañas Siembras / Flores. Filtro por `nivelSiembra` (combo 0–13), marca de evento con
  «No certificable» cuando `certificable == false`. Temporizador con `tiempoCrecimientoMinutos`.
  Ficha de cultivo: «Recetas que lo usan» (`idSiembra` en ingredientes y opciones).
- **Cocina**: lista con filtro por `nivelCocina` (combo 0–13) y por categoría/evento; ficha con energía y venta
  por estrella, ingredientes enlazados (`idSiembra` → cultivo, `idReceta` → receta) y aviso de pendientes.
- **Niveles**: resumen de cuántos ítems quedan bloqueados según el nivel del jugador en Siembra y Cocina
  (`AficionesLevelSummary.lockedCount`).
- Todo el texto visible en español. Ningún campo en inglés en estos archivos; la búsqueda es solo por `nombre`.
