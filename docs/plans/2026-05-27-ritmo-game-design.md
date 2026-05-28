# Ritmo — Game Design Document

**Fecha:** 2026-05-27  
**Última actualización:** 2026-05-28  
**Estado:** MVP jugable en `index.html` (HTML/JS/CSS puro)

---

## Concepto

Juego rítmico visual para mobile, infinito y competitivo. La pantalla es un campo oscuro donde aparecen figuras geométricas que rotan y se achican hasta desaparecer. El jugador debe tocarlas antes de que desaparezcan, priorizando las de menor vida restante. No hay niveles ni pantallas de carga — una sesión, una vida.

**Referencias de inspiración:** Osu!, Piano Tiles, Tetris, Geometry Dash  
**Pilares del juego:**
- Simple de entender, difícil de dominar
- Sesiones cortas (1–5 minutos), ideal para mobile
- Competitivo con cero pay-to-win
- Sin assets gráficos — todo generado con código

---

## Modos de juego

### Modo Classic (MVP)
- Todas las figuras tienen un color fijo por tipo (ver tabla abajo)
- Mecánica pura: tocar antes de que desaparezcan
- La estrategia viene de la diferencia de duración por tipo

### Modo Color (post-MVP)
- Misma mecánica base + reglas adicionales por color:
  - **Rojo** — NO tocar (trampa, game over inmediato)
  - **Azul** — tap normal
  - **Amarillo** — hold (sostener hasta que suene el beat)
  - **Verde** — doble tap rápido
- Los colores se introducen progresivamente con la dificultad

---

## Mecánica principal

1. Aparecen figuras geométricas (círculos, cuadrados, triángulos) en posiciones aleatorias
2. Cada figura **rota** y **se achica** continuamente — el tamaño actual ES el timer visual
3. **Cada tipo tiene una duración distinta y fija** (ver tabla), lo que crea estrategia de prioridad
4. El jugador decide qué tocar primero basándose en el tamaño relativo de cada tipo
5. Al tocar: sound FX específico por tipo + animación de burst + floating score
6. Beat hipnótico de fondo sintetizado con Web Audio API (sin archivos externos)

**Colores por tipo:**
| Figura | Color | Duración inicio | Duración máx dificultad |
|--------|-------|----------------|------------------------|
| Círculo | Cyan `#00E5FF` | 5.0s | ~2.0s |
| Cuadrado | Blanco `#FFFFFF` | 4.0s | ~1.6s |
| Triángulo | Amarillo `#FFE000` | 3.0s | ~1.2s |

**Game over cuando:**
- Una figura desaparece sin ser tocada
- En Modo Color: se toca una figura roja

---

## Sistema de puntaje

- **Base** — 100 pts por figura tocada
- **Speed bonus** — hasta +80 pts según qué tan temprano se tocó (respecto a su lifetime)
- **Beat bonus** — hasta +80 pts si el tap cae dentro de ±107ms del kick del beat (ventana 22% del beat a 124bpm). El bonus es proporcional: más centrado en el beat = más puntos. Se muestra "BEAT!" en dorado
- **Streak multiplier** — racha sin errores: x1 → x2 (5 aciertos) → x4 (10) → x8 (20)
- Fórmula final: `(100 + speedBonus + beatBonus) × multiplier`
- Dejar desaparecer una figura termina la partida (no resetea multiplicador — mata directamente)

---

## Dificultad (curva logarítmica infinita)

**Sin techo.** La dificultad sube rápido al inicio y se suaviza gradualmente — el cambio nunca es perceptible en un momento dado, solo se acumula. Fórmula base:

```
factor(t) = 1 + 0.55 × ln(1 + t/35)
```

| Parámetro | Fórmula | t=0 | t=35s | t=70s | t=140s | t=300s |
|-----------|---------|-----|-------|-------|--------|--------|
| Spawn interval | 1.5 / factor | 1.5s | 1.09s | 0.94s | 0.80s | 0.68s |
| Max figuras activas | 3 + 1.6×ln(1+t/18) | 3 | 4 | 5 | 6 | 7 |
| Círculo lifetime | 5 / factor | 5.0s | 3.6s | 3.1s | 2.7s | 2.3s |
| Cuadrado lifetime | 4 / factor | 4.0s | 2.9s | 2.5s | 2.1s | 1.8s |
| Triángulo lifetime | 3 / factor | 3.0s | 2.2s | 1.9s | 1.6s | 1.4s |

Los lifetimes tienen un floor de 38% del valor base (para no volverse imposibles).

---

## Sistema de seeds

El juego es determinístico mediante seeds — garantiza fairness en competencia.

- **Free play** — seed aleatoria por sesión (práctica, no rankeable)
- **Daily seed** — calculada en cliente con la fecha (`YYYY-MM-DD` como seed base). Todos los jugadores reciben exactamente la misma secuencia de figuras cada día
- **1v1** — ambos jugadores reciben la misma seed, sesión idéntica
- **Ghost mode** — el recording está atado a una seed; el retador juega esa misma seed

Algoritmo recomendado: `mulberry32` (simple, determinístico, sin dependencias)

---

## Competencia

### MVP
- Leaderboard global diario (daily seed) — separado por Classic / Color
- Leaderboard all-time personal best — separado por Classic / Color
- Al terminar: puntaje propio, posición global, top 10

### Roadmap
- **Ghost mode** — jugás contra la sombra del jugador inmediatamente arriba en el ranking
- **1v1 live** — misma sesión, mismas figuras (misma seed), gana quien dura más

---

## Monetización

### Free tier
- Ad intersticial cada 2–3 muertes (no cada partida)
- "Continue" opcional: ver ad para reaparecer con figuras actuales, pero multiplicador en x1

### Premium pass (pago único, ~$2.99)
- Elimina todos los ads
- Skins de figuras y paletas de colores alternativas
- Ícono de perfil exclusivo en el leaderboard

**Sin pay-to-win.** Ningún beneficio de gameplay se vende.

---

## Proyección económica

| DAU | Ingreso bruto/mes | Infraestructura | Margen neto* |
|-----|-------------------|-----------------|--------------|
| 1,000 | ~$600 | ~$2 | ~75% |
| 10,000 | ~$6,000 | ~$40 | ~78% |
| 100,000 | ~$60,000 | ~$200 | ~80% |

*Después de corte de stores (15–30%) y ad networks (~35%). Sin costos de equipo.

---

## Tech Stack (producción)

### Frontend
- **React Native + Expo** — cross-platform iOS/Android
- **React Native Skia** — renderizado 2D de figuras, animaciones de shrinking y rotación
- **React Native Reanimated** — feedback de tap, transiciones de UI
- **Expo Audio** — sound FX por figura + música de fondo

### Backend (pay-per-use, sin costos fijos)
- **Firebase Auth** — autenticación (gratis hasta 10M usuarios)
- **Firestore** — perfiles de usuario, scores ($0.06/100k reads)
- **Upstash Redis** — leaderboards con sorted sets ($0.20/100k requests)
- **Expo EAS** — builds para stores ($0 en tier gratuito)

### Seeds
- Calculadas en cliente con `mulberry32` — cero costo de servidor

---

## Visual y audio

- **Área de juego:** fondo `#05050F` con grid sutil, figuras geométricas con glow
- **Exterior al área:** fondo `#0d0d1f` con grid CSS y gradientes radiales sutiles
- Figuras con color fijo por tipo + glow que cambia a naranja → rojo pulsante cuando quedan <40% de vida
- Ghost ring (outline al tamaño original) siempre visible como referencia
- Arco de tiempo restante aparece cuando la figura entra en zona de urgencia
- Beat sintetizado con Web Audio API: kick + snare + hat, 124bpm, sin archivos externos
- Sound FX distintos por tipo: círculo (880Hz), cuadrado (500Hz), triángulo (700Hz)

---

## MVP jugable — estado actual

**Archivo:** `index.html` (HTML/CSS/JS puro, sin dependencias)  
**Cómo ejecutar:** Abrir directamente en browser

### Implementado
- [x] Loop de juego con canvas
- [x] 3 tipos de figuras con duraciones distintas por tipo
- [x] Shrinking + rotación como timer visual
- [x] Ghost ring + arco de urgencia + pulso en crítico
- [x] Score con speed bonus + streak multiplier (x1/x2/x4/x8)
- [x] Beat bonus (±107ms ventana, muestra "BEAT!" en dorado)
- [x] Beat sintetizado con Web Audio API (kick/snare/hat, 124bpm)
- [x] Sound FX distintos por tipo al tocar
- [x] Partículas burst al tocar + floating score
- [x] Curva logarítmica de dificultad sin techo
- [x] Screen shake en game over
- [x] Pantalla de inicio, HUD, game over con stats
- [x] Fondo exterior diferenciado del área de juego

### Pendiente (próxima iteración)
- [ ] Sistema de seeds (mulberry32)
- [ ] Modo Color (rojo/azul/amarillo/verde con mecánicas distintas)
- [ ] Leaderboard (requiere backend)
- [ ] Auth básico
- [ ] Daily seed
- [ ] Identidad visual final (3 opciones estéticas a evaluar)
- [ ] Port a React Native + Expo

---

## Decisiones descartadas

- **Mutaciones de mecánica** — descartado por riesgo de caos. La dificultad escala solo por velocidad/cantidad/duración
- **Variación aleatoria en lifetime** — descartado. El lifetime depende solo del tipo de figura + tiempo de partida (determinístico)
- **Mezcla de modos desde el inicio** — los modos son separados: Classic primero, Color después
