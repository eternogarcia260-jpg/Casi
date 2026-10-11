# Apex GT: Grand Prix Nocturno de Singapur (Marina Bay) & Pipeline Visual Cinemático

Transformación de Apex GT a un simulador de carreras AAA con la incorporación del icónico circuito urbano nocturno de Singapur (Marina Bay Street Circuit), un sistema de iluminación nocturna de alta potencia optimizado para cero latencia en GPU, y un pipeline de post-procesamiento visual cinematográfico que eleva drásticamente la inmersión visual a 60–120 FPS.

## Diagnóstico Técnico Integral & Evaluación del Código Actual

### Nota Global del Código: 8.8 / 10 (Sobresaliente con base de simulación de alto nivel)

- **Arquitectura de Físicas y Bucle (9.5/10)**: Bucle desacoplado a 120 Hz (`FIXED_PHYSICS_DT = 1/120`) con acumulador de tiempo e interpolación sub-frame ($\alpha$) impecable. Modelo de neumáticos no lineal con curva de degradación por compuestos (Soft, Medium, Hard), graining, blistering y flat spots. Cero recolección de basura (GC) en runtime gracias a `MathPool`.
- **Modelado 3D y Cinemática de Vehículos (9.2/10)**: Monoplaza procedural con más de 4,400 líneas en `CarModel.ts` con osciladores aeroelásticos físicos para alerones y DRS, incandescencia de frenos carbocerámicos y deformación de daños.
- **Rendimiento e Interfaz (8.8/10)**: Manipulación directa del DOM en `HUD.tsx` para refrescos de 60/120 FPS sin sobrecarga del Virtual DOM de React.
- **Oportunidades de Mejora Detectadas**:
  1. *Iluminación y Atmósfera*: Actualmente la escena está limitada a una iluminación diurna/atardecer estática con una sola luz direccional principal; carece de iluminación nocturna dinámica con proyectores de estadio y reflejos especulares sobre el asfalto.
  2. *Catálogo de Circuitos*: Solo existen 2 circuitos (Square Apex y Speedway GP); falta un circuito urbano nocturno de clase mundial como Singapur.
  3. *Post-procesamiento*: Efectos visuales de velocidad separados que pueden unificarse en un pase óptico con destellos anamórficos nocturnos, bloom calibrado y contraste de carrera nocturna.

---

## Decisiones Críticas y Preferencias del Usuario

> [!IMPORTANT]
> Decisiones confirmadas en la fase de clarificación:
> - **Pilar Prioritario**: Salto visual integral con post-processing e iluminación avanzada.
> - **Nuevo Circuito & Ambiente**: Circuito urbano de Singapur de F1 en modo nocturno con iluminación artificial de gran potencia.
> - **Rendimiento Objetivo**: Mantener 60 FPS estables en dispositivos móviles y 120 FPS en monitores de alta frecuencia sin penalizaciones de Garbage Collection ni sobrecarga de draw calls.

- **Decisión Confirmada 1**: Integración de `singapore_marina` como tercer circuito oficial en `CircuitRegistry`, con geometría de trazado de 3.2 km inspirada en Marina Bay, edificios emblemáticos (Marina Bay Sands skyline, Helix Bridge, noria Singapore Flyer iluminada) y zonas DRS.
- **Decisión Confirmada 2**: Iluminación nocturna de estadio mediante torres de focos LED de alta descarga mediante `InstancedMesh` y shaders de reflexión especular en asfalto nocturno (evitando cientos de luces dinámicas costosas que hundirían el frametime).
- **Decisión Confirmada 3**: Pipeline de post-procesamiento con aberración cromática periférica, destellos ópticos horizontales de focos (anamorphic streaks), atmósfera de niebla nocturna urbana y adaptación de luminancia.

---

## 1. Visión General del Producto

- **Qué Ofrece**: Una experiencia de carrera nocturna hiperrealista bajo los focos de Singapur, con asfalto que refleja las luminarias, edificios y luces de la ciudad, física de monoplaza F1 afinada para curvas lentas y chicane de 90 grados, y telemetría de competición.
- **Público Objetivo**: Aficionados a la simulación de carreras y Formula 1 que buscan adrenalina visual y precisión técnica directa en el navegador web sin descargas ni instalaciones pesadas.
- **Propuesta de Valor**: Gráficos propios de consola en un navegador web ligero, con carga instantánea y fluidez absoluta.

---

## 2. Experiencia de Usuario y Diseño Visual

### Flujos Clave de Usuario
1. **Selección en Dashboard**: En el menú de circuitos del `MainMenuDashboard`, el usuario visualiza el nuevo circuito "Marina Bay Nocturno (Singapur)", con etiqueta de carrera nocturna, mapa vectorial y estadísticas técnicas (longitud, curvas, zonas DRS).
2. **Carga Inmersiva**: Pantalla de carga con ambientación nocturna y datos técnicos del Gran Premio de Singapur.
3. **Parrilla de Salida Nocturna**: Inicio bajo los focos encendidos, gantry de 5 luces rojas reflejadas sobre el morro del coche y el asfalto húmedo/pulido.
4. **Carrera & Pits Nocturnos**: Conducción a través de sectores con túneles de luz, edificios iluminados en el fondo, paso por boxes iluminados con equipos de mecánicos y zona DRS en la recta de Raffles Boulevard.

### Identidad Visual & Paleta
- **Estilo**: Nocturno eléctrico, alta tecnología automovilística, inspirado en las retransmisiones oficiales de la F1 en Marina Bay.
- **Colores Principales**:
  - Asfalto nocturno: Carbón profundo `#0b0f17` con reflectividad especular.
  - Iluminación de focos: Blanco cálido/frío dual de halogenuro metálico `#f8fafc` y `#fef08a`.
  - Skyline de la ciudad: Acentos azul eléctrico `#0284c7`, púrpura magenta `#c026d3` y cian `#06b6d4` en las torres del fondo.
  - HUD de Telemetría: Blanco puro con contrastes ámbar (`#f59e0b`) y carmesí (`#ef4444`).
- **Tipografía**:
  - Títulos y Modo: `Barlow Condensed` / `Plus Jakarta Sans` con peso audaz.
  - Métricas e instrumentos de carrera: `font-mono tabular-nums` para cronómetros, marchas y velocidad.

---

## 3. Decisiones de Ingeniería y Balance de Rendimiento

### Decisión 1: Iluminación Nocturna de Gran Escala sin Pérdida de FPS
- **Enfoque Elegido**: Arquitectura híbrida de iluminación. En lugar de colocar 400 luces puntuales dinámicas (`PointLight`) que colapsarían el cálculo de sombras y shaders en WebGL, empleamos:
  1. Dos luces direccionales globales con balance de luz nocturna (luz lunar cenital azulada tenue + luz cenital de reflectores).
  2. Torres de focos modeladas con geometrías instanciadas (`InstancedMesh`) con emisivos HDR y conos volumétricos de luz falsos ultra-ligeros con adición aditiva.
  3. Shader de asfalto nocturno con mapa de rugosidad y brillo especular adaptado para capturar la orientación hacia los reflectores de pista.
- **Razón**: Permite simular más de 1,200 focos de circuito con exactamente 1 draw call y coste computacional ínfimo para la GPU.

### Decisión 2: Circuito Marina Bay (Singapur) con Geometría Optimizada
- **Enfoque Elegido**: Construcción modular en `SingaporeTrackBuilder.ts` implementando la interfaz `ITrackWorld`, con:
  - Spline de 16 curvas técnicas con los puntos clave: curva 1-2-3 (Sheares), recta Raffles con DRS, curva de la bahía (Anderson Bridge), sección del Marina Bay Sands y recta principal de boxes.
  - Pistas de piano (kerbs) bicolores azul/blanco con run-offs asfálticos y barreras Tecpro/SAFER de alta resistencia.
  - Elementos icónicos en el fondo: Torres del Marina Bay Sands con pasarela voladiza iluminada, la noria Singapore Flyer giratoria en segundo plano y rascacielos iluminados.
- **Razón**: Recrea con máxima fidelidad la atmósfera del GP de Singapur sin sobrecargar la memoria VRAM (reutilización de materiales y texturas procedurales).

### Decisión 3: Pipeline de Efectos Cinemáticos Nocturnos
- **Enfoque Elegido**: Ampliación de `CinematicPostEffect.ts` con un modo nocturno automático que ajusta el mapeo de tonos ACES Filmic, añade destellos anamórficos sutiles en los reflectores (bloom de bajo coste en plano de cámara) y realza el contraste de la fibra de carbono y los neumáticos bajo los focos.
- **Razón**: Transforma la estética visual haciéndola comparable a un título de PC/consola sin requerir pases de render adicionales de RenderTarget múltiple.

---

## 4. Arquitectura Técnica y Diagrama del Sistema

```
┌────────────────────────────────────────────────────────────────────────┐
│                        Apex GT Racing Engine                          │
└────────────────────────────────────┬───────────────────────────────────┘
                                     │
           ┌─────────────────────────┼─────────────────────────┐
           ▼                         ▼                         ▼
┌─────────────────────┐   ┌─────────────────────┐   ┌─────────────────────┐
│  Circuit Registry   │   │  Dynamic Lighting   │   │  Cinematic Optics   │
│  & World Builder    │   │  & Night Atmosphere │   │  Post-Process Suite │
├─────────────────────┤   ├─────────────────────┤   ├─────────────────────┤
│ • SingaporeConfig   │   │ • Night Stadium Sky │   │ • Night Contrast LUT│
│ • SingaporeTrack-   │   │ • Instanced Floods  │   │ • Anamorphic Flares │
│   Builder (3.2 km)  │   │ • Specular Tarmac   │   │ • Low-Latency Dither│
│ • Waypoints & Sectors│  │ • Emissive Towers   │   │ • Speed Flow Streaks│
└──────────┬──────────┘   └──────────┬──────────┘   └──────────┬──────────┘
           │                         │                         │
           └─────────────────────────┼─────────────────────────┘
                                     ▼
                    ┌─────────────────────────────────┐
                    │  WebGL2 High-Performance Canvas  │
                    │  (60 FPS Mobile / 120 FPS PC)    │
                    └─────────────────────────────────┘
```

### Componentes a Modificar y Crear
1. **`src/game/circuits/singapore/SingaporeCircuitConfig.ts`** *(Nuevo)*: Definición del circuito de Singapur, puntos de control de sectores, longitud (~3,150 m), zonas DRS, cajones de parrilla y zona de pit lane.
2. **`src/game/circuits/singapore/SingaporeWaypoints.ts`** *(Nuevo)*: Trayectoria spline de carrera para Singapur con línea óptima de frenada y aceleración para el jugador y la IA.
3. **`src/game/circuits/singapore/SingaporeTrackBuilder.ts`** *(Nuevo)*: Constructor 3D del circuito de Singapur con asfalto reflectante nocturno, torres de iluminación instanciadas, muros urbanos, skyline con Marina Bay Sands y boxes.
4. **`src/game/circuits/CircuitRegistry.ts`**: Registro del nuevo circuito `singapore_marina`.
5. **`src/game/environment/DynamicSkySystem.ts`**: Soporte para cielo nocturno estrellado y bruma luminosa urbana de Singapur.
6. **`src/game/effects/CinematicPostEffect.ts`**: Calibración para modo noche con destellos anamórficos de reflectores y preservación del contraste negro/luz.
7. **`src/game/RacingGameEngine.ts`**: Detección del modo noche para ajustar la exposición y tono del renderizador ACESFilmic según el circuito activo.
8. **`src/components/MainMenuDashboard.tsx`**: Inclusión de la tarjeta y mapa del GP de Singapur con vista previa nocturna y selección rápida.

---

## 5. Verificación y Criterios de Éxito
- **Compilación sin errores**: `tsc --noEmit` y `npm run build` limpios.
- **Rendimiento**: Ejecución fluida a 60–120 FPS con cero allocations en el bucle principal.
- **Jugabilidad**: Conducción precisa por las 16 curvas del Marina Bay Street Circuit con IA competitiva, detección de sectores, vueltas y paradas en boxes operativas en horario nocturno.
