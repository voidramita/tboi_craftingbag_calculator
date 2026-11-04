# CHANGES.md — Calculadora de Recetas de Tainted Cain (TBOI Repentance+)

Documento de especificación técnica para la implementación de la herramienta de síntesis y cálculo de recetas de *Bag of Crafting* (Sacabolsas) para Tainted Cain en **The Binding of Isaac: Repentance+**.

---

## 1. Requerimientos del Sistema y Entorno

- **Stack:** HTML5, CSS3 (Tailwind CSS CDN / JIT), JavaScript ES6+ (o TypeScript sin bundle complejo).
- **Estilo:** Modo oscuro continuo (`bg-[#1e1e1e]`), acorde a la interfaz original del juego/referencia.
- **Portabilidad:** Aplicación Single-Page (SPA) autosuficiente o modularizable para OpenCode.

---

## 2. Definición de Datos y Componentes (Pickups)

Crear la estructura de datos con los 29 componentes soportados en *Repentance+*:

| ID | Nombre | Tipo / Categoría | Valor Base / Calidad |
|---|---|---|---|
| `red_heart` | Corazón Rojo | Corazón | 1 |
| `soul_heart` | Corazón de Alma | Corazón | 4 |
| `black_heart` | Corazón Negro | Corazón | 5 |
| `eternal_heart` | Corazón Eterno | Corazón | 5 |
| `gold_heart` | Corazón Dorado | Corazón | 3 |
| `bone_heart` | Corazón de Hueso | Corazón | 5 |
| `rotten_heart` | Corazón Podrido | Corazón | 1 |
| `penny` | Moneda (1c) | Moneda | 1 |
| `nickel` | Níquel (5c) | Moneda | 3 |
| `dime` | Moneda de 10c | Moneda | 8 |
| `lucky_penny` | Moneda de la Suerte | Moneda | 5 |
| `golden_penny` | Moneda Dorada | Moneda | 8 |
| `key` | Llave Normal | Llave | 2 |
| `golden_key` | Llave Dorada | Llave | 5 |
| `charged_key` | Llave Cargada | Llave | 5 |
| `bomb` | Bomba | Bomba | 2 |
| `golden_bomb` | Bomba Dorada | Bomba | 5 |
| `giga_bomb` | Giga Bomba | Bomba | 10 |
| `micro_battery` | Micro Batería | Batería | 2 |
| `battery` | Batería Normal | Batería | 4 |
| `mega_battery` | Mega Batería | Batería | 8 |
| `golden_battery` | Batería Dorada | Batería | 8 |
| `card` | Carta / Tarjeta | Consumible | 2 |
| `pill` | Píldora | Consumible | 2 |
| `rune` | Runa | Consumible | 4 |
| `dice_shard` | Fragmento del Dado | Especial | 4 |
| `soul_stone` | Piedra del Alma | Especial | 4 |
| `golden_pill` | Píldora Dorada | Consumible | 8 |
| `poop` | Caca / Poop | Especial | 0 |

---

## 3. Estructura de la Interfaz (UI Breakdown)

### 3.1. Header & Input de Semilla (Seed)
- **Título principal:** `Sintetiza la Excavadora de Bolsas del Tesoro` (Fuente Serif / Retro).
- **Sección Semilla:** Input de texto bloqueado a 8 caracteres alfanuméricos en mayúsculas, con estilo visual destacado en azul (`Semillas`).

### 3.2. Panel de Selección de Pickups (Grid)
- Cuadrícula responsiva que contiene las 29 tarjetas de componentes.
- Cada tarjeta muestra:
  - Icono o representación visual del pickup.
  - Dropdown (`<select>`) o contador numérico interactivo (0 a 30).

### 3.3. Barra de Filtros y Configuración
- **Límite de Recetas:** Selector mutuo exclusivo para `1`, `10`, `20`, `50` y `100` recetas.
- **Interruptores de Motor:**
  - `Habilitar / Desactivar actualización en tiempo real`
  - `Activar / Desactivar aceleración WASM`
- **Modificadores de Partida (Checkboxes):**
  - `Modo avaricia` (Greed / Greedier Mode item pools).
  - `Retos diarios (incluyendo Después del Capítulo 4)`.
  - `Transformado en un alma errante` (Tainted Forgotten / Lost state).
  - `(Corrompido) dueño de la tienda` (Tainted Keeper state).
- **Accesorios / Modificadores adicionales:**
  - `Objeto: Orbe Sagrado de la Cruz` (Sacred Orb effect: descarta ítems calidad 0 y 1).
  - `(Oro/varios) Accesorios: ¡No!` (No Trinket option).

### 3.4. Botones de Acción Principal
- `Comienza la síntesis` (Botón principal verde largo).
- `Vacía los componentes` (Reinicia el conteo de todos los pickups a 0).

### 3.5. Lista de Resultados
- Visualización en lista de recetas válidas generadas:
  - Icono e ID/Nombre del objeto resultante.
  - Indicador visual de Calidad (Quality 0 a 4 stars).
  - Fila con los 8 iconos exactos de componentes consumidos para la receta.

---

## 4. Tareas de Implementación para OpenCode

### Paso 1: Maquetación HTML & CSS (UI Base)
- Crear `index.html` con la estructura base de Tailwind CSS.
- Definir paleta de colores: fondo `#1e1e1e`, contenedores `#2d2d2d`, bordes `#444444`, acentos en azul (`#2563eb`) y verde (`#15803d`).

### Paso 2: Estado Global y Eventos (JavaScript Core)
- Implementar el objeto `state`:
  ```javascript
  const state = {
    seed: 'ABCD EFG1',
    inventory: {}, // pickup_id: cantidad
    recipeLimit: 100,
    realtime: true,
    wasmEnabled: true,
    modifiers: {
      greedMode: false,
      daily: false,
      taintedKeeper: false,
      taintedForgotten: false,
      sacredOrb: true,
      noTrinkets: true
    }
  };
  ```

### Paso 3: Motor de Combinatoria y Síntesis
- Implementar la función `generateCombinations(availablePickups)` que toma el inventario disponible y extrae combinaciones válidas de 8 elementos.
- Calcular el puntaje acumulado de la combinación $\sum (\text{valor\_pickup})$ para determinar la piscina de calidad del objeto resultante.
- Aplicar la función HASH basada en la Semilla para obtener la semilla pseudo-aleatoria del crafteo.

### Paso 4: Integración y Renderizado Dinámico
- Conectar los eventos de cambio en los inputs al refresco automático (si `realtime === true`).
- Implementar la limpieza total de inputs mediante el botón `Vacía los componentes`.

---

## 5. Plan de Verificación

1. **Validación de Componentes:** Probar que no se puedan generar recetas si el total de pickups seleccionados es menor a 8.
2. **Prueba de Reseteo:** Al presionar `Vacía los componentes`, todos los contadores deben volver a 0 y limpiarse la lista de resultados.
3. **Prueba de Semilla:** Cambiar la semilla y verificar que varíe la lista de ítems devueltos.
4. **Verificación de Modificadores:** Activar *Modo Avaricia* y verificar que el filtrado altere las recetas hacia las piscinas de objetos de Avaricia.