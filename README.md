# -Act.Codigo-Divide-y-Venceras-Enntrega-2
Comparador de Imágenes — README
Aplicación de escritorio (Tkinter) para comparar dos imágenes, encontrar diferencias locales y mostrarlas con cuadros superpuestos. El análisis es adaptativo por regiones usando quadtree + SSIM + percentiles, con igualación de histograma y pre-alineación por traslación para robustecer el resultado ante cambios de iluminación y pequeños desplazamientos.

Características Carga de pares de imágenes (A y B) vía diálogo de archivos. Normalización de brillo/contraste por igualación de histograma (B→A). Pre-alineación por búsqueda de desplazamiento entero (±8 px por defecto). Filtro pasa-altos (opcional) para reducir el efecto de iluminación global. Detección de diferencias por regiones: Recursión quadtree (divide y vencerás). SSIM por bloque calculado en O(1) con imágenes integrales. Heurística de “picos” con percentiles (p95/p99) y desviación estándar. Superposición visual de rectángulos rojos sobre ambas vistas. Interfaz configurable: ssim_min, min_bloque, hp_radius, delta_pix. 🧩 ¿Cómo funciona (resumen técnico)? Pre-proceso Convierte a escala de grises y recorta ambas al menor ancho/alto común. Iguala histogramas para compensar iluminación. Alinea por traslación buscando el desplazamiento (dx, dy) que minimiza el MSE. (Opcional) Pasa-altos con desenfoque gaussiano para resaltar detalle local. Métricas por bloque Construye imágenes integrales de A, B, A², B² y A·B para evaluar SSIM por región en O(1). Calcula percentiles de |A−B| y la desviación estándar dentro del bloque. Quadtree + SSIM + percentiles Si SSIM < ssim_min y hay pico local (p95/p99 altos) → subdivide. Si el bloque es pequeño o muestra diferencia fuerte → marca región. Al final, fusiona rectángulos solapados para simplificar el overlay.

Requisitos
Python 3.8+
Dependencias
pip install pillow numpy


Parámetros (barra inferior)

| Parámetro | Tipo | Por defecto | Qué hace | Consejos |
|---|---:|---:|---|
| ssim_min | float | 0.97 | Umbral de similitud. Más alto ⇒ más estricto (detecta diferencias más sutiles). | Si hay falsos negativos, baja a 0.95–0.96. |
| min_bloque | int | 8 | Tamaño mínimo (px) de bloque en el quadtree. | Si las diferencias son muy pequeñas, baja a 4–6. |
| hp_radius | int | 8 | Radio del desenfoque para pasa-altos (si “robusto luz” está activo). | Subirlo puede ayudar con sombras/desvanecidos. |
| delta_pix | float | 18 | Umbral de magnitud de diferencia (p99) para aceptar un bloque como “distinto”. | Si detecta ruido, súbelo; si omite diferencias, bájalo. |
| robusto luz | bool | True | Activa el pasa-altos para mitigar cambios de iluminación. | Desactívalo si tus imágenes ya están normalizadas. |

Casos soportados y límites Diferencias locales de contenido (píxeles añadidos/quitados, pequeñas ediciones). Pequeñas traslaciones (±8 px por defecto) y cambios globales de brillo. No corrige rotaciones, cambios de escala o perspectiva. (Podrías ampliar la pre-alineación con búsqueda de ángulo/escala si lo necesitas.) En imágenes con ruido o compresión fuerte, quizá requieras ajustar delta_pix y ssim_min. Detalles de implementación (puntos clave) Igualación de histograma: mapea la CDF de B a la de A para homogeneizar brillo/contraste. Alineación por desplazamiento: barrido discreto (entero) con MSE sobre imágenes suavizadas.

Detalles de implementación (puntos clave) Igualación de histograma: mapea la CDF de B a la de A para homogeneizar brillo/contraste. Alineación por desplazamiento: barrido discreto (entero) con MSE sobre imágenes suavizadas. SSIM por bloque: - Usa constantes de estabilización 𝐶 1 , 𝐶 2 C1,C2 con 𝐿 = 255 L=255. - Medias, varianzas y covarianza se obtienen con integrales para O(1) por consulta. Heurística de “picos”: - p95 y p99 de |A−B| evitan subdividir ruido; std exige textura mínima. Fusión de rectángulos: agrupa regiones solapadas para una visualización más limpia.

Solución de problemas “No se pudo abrir la imagen” → Revisa permisos o formato. Soporta: PNG/JPG/JPEG/BMP/TIF/TIFF. Demasiadas falsas alarmas → Sube delta_pix o ssim_min; activa robusto luz. Se me escapan diferencias pequeñas → Baja delta_pix y/o min_bloque; sube ssim_min. Imágenes de distinto tamaño → La app recorta al tamaño común menor; verifica que el contenido relevante quede dentro.
