# Flappy Kiro 🌿

Un juego estilo Flappy Bird de un solo archivo HTML/JS, ambientado en un palacio imperial nocturno. La protagonista es una aprendiz de botica chibi cat-girl que vuela recogiendo hierbas medicinales y esquivando obstáculos.

---

## Cómo jugar

| Acción | Control |
|---|---|
| Saltar / iniciar / reiniciar | `Espacio`, clic o toque |

- Recoge **10 hierbas 🌿** para completar la poción y ganar
- Evita los **frascos de veneno ☠** (restan una vida)
- Tienes **3 vidas ♥**; si las pierdes todas, game over
- El récord de hierbas se guarda automáticamente en el navegador

---

## Descripción técnica

Un solo archivo `index.html` (~1 900 líneas). Sin dependencias externas, sin imágenes, sin archivos de audio — todo se genera con Canvas 2D y Web Audio API.

### Estructura del código

```
1.  CONFIG            — constantes de física, colores, tiempos
2.  state             — fase, score, vidas, hierbas, récord
3.  Entidades         — player, pillarList, itemList, floatingTexts
4.  Sonido            — Web Audio API (osciladores sintetizados)
5.  Parallax          — 3 capas de fondo animadas
6.  Física            — gravedad, salto, movimiento de obstáculos
7.  Spawning ítems    — hierbas y venenos en el espacio libre
8.  Colisiones        — por tipo de obstáculo
9.  Feedback visual   — textos flotantes, flash de pantalla
10. Entrada           — teclado, clic, touch
11. Renderizado       — drawBackground → drawPillars → drawPlayer → HUD
12. Game loop         — requestAnimationFrame
13. Inicialización    — loadRecord, initParallax, arranque
```

---

## Diseño visual

### Fondo — cielo azul oscuro estrellado
- Degradado `#010610 → #061838`
- **60 estrellas** animadas con parpadeo (twinkle) individual; algunas con destello en cruz
- **Luna llena** con halo difuso y cráteres en la esquina superior derecha
- Sombra azulada en el fondo del canvas: `box-shadow: 0 0 40px rgba(60,100,220,0.7)`

### Parallax de 3 capas
| Capa | Velocidad | Contenido |
|---|---|---|
| 0 — lejana | 0.3 px/frame | Tejados triangulares oscuros del palacio |
| 1 — media | 0.8 px/frame | Paredes, tejas, ventanas circulares y linternas azules que se balancean |
| 2 — cercana | 1.4 px/frame | Tallos de bambú con nudos y hojitas |

Las capas usan wrap-around modular (`WIDTH × 2`) para que no haya saltos visuales. La animación corre también en la pantalla de inicio.

### Personaje — chibi cat-girl
Inspirada en el personaje de referencia: chibi con orejas de gato, pelo verde oscuro y ojos azules grandes.

- **Cabeza grande** estilo chibi con cara redondeada y piel clara
- **Orejas de gato** triangulares con interior bicolor y lazo azul
- **Pelo verde oscuro** con flequillo recto y mechones laterales
- **Ojos azules** con iris degradado, doble brillo, pestañas dibujadas y cejas finas
- **Pecas**, mejillas sonrojadas, nariz pequeña y sonrisa chibi
- **Kimono verde** con solapa cruzada y obi oscuro
- **Cola de gato** animada con ondulación senoidal y bolita de color en la punta
- Se inclina según la velocidad vertical (`vy × 2°`)
- Parpadea durante los frames de invulnerabilidad

### Obstáculos

Los obstáculos se alternan: **tela → repisa → tela → repisa...**  
Cada obstáculo es **independiente** (no hay par arriba+abajo simultáneo).

#### Telas de seda colgantes (obstáculo superior)
Cuelgan desde `y = 0` hasta `y = hangH` (altura aleatoria).

- Barra/travesaño azul oscuro con 5 argollas metálicas
- **3 paños superpuestos** (púrpura, azul, rojo-rosa) con transparencias
- Ondulación de viento mediante curvas Bézier cúbicas: cada paño tiene fase independiente para los bordes izquierdo, derecho y centro
- Brillo de seda (reflejo de luz) en el tercio superior
- **Flecos animados** en el borde inferior
- Borla dorada central que se balancea

#### Repisas de madera con frascos (obstáculo inferior)
Suben desde el suelo hasta `y = groundY - riseH` (altura aleatoria).

- Soporte vertical de madera oscura con vetas curvas procedurales
- **Tablón** con borde dorado, tres clavos metálicos y sombra
- **5 frascos** sobre el tablón, cada uno con:
  - Cuerpo de vidrio coloreado (verde, rojo, azul, ámbar, lima)
  - Líquido interior semitransparente a distinto nivel con brillo de superficie
  - Cuello, tapón de corcho con ranura, brillo lateral y destello puntual
  - **Vibración suave individual** animada con `Math.sin`

---

## Sistema de colisiones

```js
// Tela: colisiona si el jugador entra en la zona 0 → hangH
if (p.type === 'curtain') return py1 < p.hangH;

// Repisa: colisiona si el jugador entra en la zona (groundY - riseH) → groundY
if (p.type === 'shelf')   return py2 > groundY - p.riseH;
```

Margen de 5 px en todos los lados del jugador para colisiones más justas.

---

## Sonido (Web Audio API)

Sin archivos externos. Todo se sintetiza en tiempo real con osciladores.  
El `AudioContext` se crea en el primer gesto del usuario (política de autoplay).

| Evento | Forma de onda | Frecuencia |
|---|---|---|
| Salto | `sine` | 320 → 520 Hz (glide) |
| Recoger hierba | `sine` | 520 Hz + 780 Hz (80 ms después) |
| Daño / veneno | `square` | 300 → 80 Hz (descenso) |
| Game over | `square` | 220, 180, 140 Hz |
| Victoria | `triangle` | C5-E5-G5-C6 (fanfarria) |
| Superar obstáculo | `triangle` | 440 Hz (tick) |
| Nuevo récord | `sine` | Escala ascendente 5 notas |

---

## Récord

Guardado en `localStorage` bajo la clave `flappyKiro_record`.  
Se compara al final de cada partida (victoria o game over) y se muestra en el HUD y en todas las pantallas de overlay.

---

## HUD en juego

```
[Obstáculos: N]          [♥ ♥ ♥]
[🏆 récord hierbas]    [🌿 N / 10]
                        [████░░░░░░]  ← barra de progreso
```

---

## Parámetros ajustables (CONFIG)

| Parámetro | Valor | Descripción |
|---|---|---|
| `GRAVITY` | 0.30 | Aceleración hacia abajo (px/frame²) |
| `JUMP_FORCE` | -7.5 | Impulso vertical al saltar |
| `MAX_FALL` | 9 | Velocidad máxima de caída |
| `PILLAR_WIDTH` | 64 | Ancho de los obstáculos (px) |
| `PILLAR_SPEED` | 2.0 | Velocidad horizontal inicial |
| `PILLAR_INTERVAL` | 150 | Frames entre spawn de obstáculos |
| `PILLAR_GAP` | 210 | (legacy, no usado en el nuevo modelo) |
| `MAX_LIVES` | 3 | Vidas iniciales |
| `HERBS_TO_WIN` | 10 | Hierbas necesarias para ganar |
| `INVULN_FRAMES` | 90 | Frames de invulnerabilidad tras daño |
| `HERB_CHANCE` | 0.65 | Probabilidad de spawn de hierba |
| `POISON_CHANCE` | 0.25 | Probabilidad de spawn de veneno |
| `PARALLAX_SPEEDS` | [0.3, 0.8, 1.4] | Velocidades de las 3 capas de fondo |

---

## Fases de desarrollo completadas

- **Fase 1** — Mecánica base: física, pilares, colisiones
- **Fase 2** — Sistema de vidas, hierbas, venenos, feedback visual
- **Fase 3** — Parallax, sonido Web Audio, récord en localStorage
- **Reskin** — Fondo azul oscuro estrellado con luna, personaje chibi cat-girl
- **Obstáculos** — Telas de seda colgantes (arriba) y repisas con frascos (abajo), alternados e independientes
