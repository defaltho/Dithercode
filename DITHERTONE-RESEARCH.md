# DitherCode — Research: DITHERTONE PRO 1.1.0

**Data:** 2026-06-13
**Fonte:** `C:\Program Files\Adobe\Adobe Photoshop 2025\Plug-ins\DITHERTONE-PRO-1.1.0`

---

## O que o DITHERTONE tem (33 algoritmos)

| Categoria | Algoritmos |
|-----------|-----------|
| Diffusion | Floyd-Steinberg, **JJN**, Stucki, Burkes, Atkinson |
| Modulation | Row, Column, Dispersed, Medium, Heavy, Circuit, Tilt |
| Pattern | Matrix, Knitt, Cross Square, Serpentine, Rekt Block, Variable Hatch |
| Bayer | 2×2, 4×4, 8×8 |
| Sierra | Sierra, Sierra Two-Row, Sierra Lite |
| Halftone | Halftone 0°, **Halftone 22.5°**, **Halftone 45°** |
| Other | Star, Grid, Vertical Stitch, Horizontal Stitch, Clock, Cyber |

---

## O que o DitherCode já tem

- Diffusion: Floyd-Steinberg, Atkinson, Stucki, Burkes, Sierra, Sierra Lite
- Ordered: Bayer 2×2, 4×4, 8×8
- Render modes: halftone (dots variáveis), circles, cross, ascii

---

## O que está em falta (prioridade)

### Alta prioridade — algoritmos clássicos em falta

**Jarvis-Judice-Ninke (JJN):**
```javascript
// Kernel para adicionar em KERNELS:
jjn: { div: 48, k: [
  [1, 0, 7], [2, 0, 5],
  [-2, 1, 3], [-1, 1, 5], [0, 1, 7], [1, 1, 5], [2, 1, 3],
  [-2, 2, 1], [-1, 2, 3], [0, 2, 5], [1, 2, 3], [2, 2, 1]
]}
```
Qualidade entre Floyd-Steinberg e Stucki. Excelente para retratos.

**Sierra Two-Row:**
```javascript
sierra2row: { div: 16, k: [
  [1, 0, 4], [2, 0, 3],
  [-2, 1, 1], [-1, 1, 2], [0, 1, 3], [1, 1, 2], [2, 1, 1]
]}
```

---

### Média prioridade — pipeline de pré-processamento

O DITHERTONE aplica filtros ANTES de ditherar. Isto melhora significativamente a qualidade:

**Sharpen (unsharp mask):**
- Parâmetros: strength (0-200, default 1), radius (0-100, default 1)
- Aplica antes de ditherar → mais detalhe nos contornos

**Noise:**
- Adiciona ruído aleatório (0-50) antes de ditherar
- Evita padrões regulares em gradientes

**Blur:**
- Suaviza antes de ditherar para superfícies uniformes

Todos aplicados na ordem: sharpen → denoise → noise → blur → brightness/contrast → dither

---

### Média prioridade — Modulation patterns

**Row Modulation** (o mais simples):
```javascript
// threshold varia sinusoidalmente por linha
const t = 0.5 + Math.sin(y * freq * 2 * Math.PI / WH) * amplitude;
const result = lum > t ? 255 : 0;
```

**Column Modulation:** idem mas por coluna (x)

**Dispersed Modulation:** threshold = pseudo-aleatório por posição (diferente de Bayer)

---

### Baixa prioridade — padrões especiais

- Halftone 22.5° e 45° — rotação da grelha de halftone
- Stitch (vertical/horizontal) — padrão em ponto de cruz
- Grid — crosshatch threshold

---

## Parâmetros de tonal mapping (DITHERTONE)

O DITHERTONE tem controlo fino sobre highlights/midtones/shadows com ranges separados:
- Highlights: 0-170, default 50
- Midtones: 0-128, default 70
- Shadows: 0-78, default 78

Aplicado como curve: mapeia os níveis de luminância antes de ditherar.
No DitherCode, isto poderia ser um slider "curve" ou 3 sliders separados.

---

## Resumo — o que adicionar a seguir

1. **JJN kernel** — 1 linha de código (just add to KERNELS object)
2. **Sierra Two-Row kernel** — 1 linha
3. **Sharpen filter** — pré-processamento via convolução 3×3 ou ImageData unsharp mask
4. **Noise slider** — adiciona `Math.random() * noise - noise/2` ao canal de lum antes de ditherar
5. **Row/Column Modulation** — threshold sinusoidal, novo modo de ordered dithering
