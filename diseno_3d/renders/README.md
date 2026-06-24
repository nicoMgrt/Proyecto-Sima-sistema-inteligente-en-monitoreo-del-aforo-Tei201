# Renders — Visualización del encapsulado

Esta carpeta contiene los renders del encapsulado generados desde el workspace Render de Autodesk Fusion 360 con materiales aplicados e iluminación configurada.

## Archivos

| Archivo | Descripción |
|---|---|
| `render_exterior.png` | Vista isométrica exterior con tapa cerrada |
| `render_interior.png` | Vista superior con tapa removida mostrando componentes internos |
| `render_explosionado.png` | Vista explosionada mostrando ensamble de todas las piezas |

## Configuración de render utilizada

- **Software:** Autodesk Fusion 360 — workspace Render
- **Motor de render:** Fusion 360 Cloud Render
- **Resolución:** 1920 × 1080 px
- **Iluminación:** Soft Box
- **Fondo:** Color sólido blanco

## Materiales aplicados

| Componente | Material en Fusion |
|---|---|
| Carcasa y tapa | ABS Plastic — gris claro |
| ESP32-S3 | PCB — verde |
| Sensores HC-SR04 | ABS Plastic — verde |
| Batería NCR18650B | Steel — acabado metálico |
| Shield de batería | PCB — verde oscuro |
| Postes espaciadores | Aluminio |

## Vistas incluidas y su propósito

**Render exterior:** Muestra el encapsulado terminado tal como se vería el producto final. Permite evaluar las proporciones, acceso a puertos y acabado general.

**Render interior:** Con la tapa removida se aprecia el layout de componentes y verifica que todos caben sin interferencias. Demuestra el criterio de diseño para la reparación — los componentes son accesibles sin destruir el encapsulado.

**Render explosionado:** Muestra la relación entre cada pieza del ensamble y el orden de montaje. Útil para entender cómo se arma el dispositivo en campo.
