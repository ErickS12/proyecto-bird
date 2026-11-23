# Plan de la presentación de propuesta — ¿Quién canta?

Objetivo de la presentación: que la profesora apruebe la idea. Tono: técnico pero fácil de leer. Duración sugerida: 10 min, unas 15 diapositivas.

---

## Diapositivas

### 1. Portada
- **Título:** ¿Quién canta? Recuperación de aves mexicanas por su canto y su imagen
- Integrantes, materia, fecha.
- Imagen de fondo: un ave mexicana (mosquero cardenal).

### 2. El problema
- En el bosque **se oye mucho más de lo que se ve**: las aves se esconden entre las hojas y una buena foto es difícil.
- Turistas y cuidadores del bosque escuchan un canto y no saben de quién es.
- Mensaje: *"Lo escuchaste, pero nunca lo viste."*

### 3. Pregunta y objetivo
- **¿Cómo podríamos** ayudar a identificar un ave solo con su canto, mostrando quién es, cómo se ve y **por qué** el sistema lo cree?
- Objetivo: un sistema de **recuperación por contenido de audio e imagen** para especies mexicanas, evaluado con las métricas del curso.

### 4. Estado del arte
- Capturas de Merlin, BirdNET y Picture Bird (`docs/referencias/`).
- Una línea por app: qué hace bien.

### 5. Lo que les falta (la oportunidad)
Tabla comparativa:

| | Merlin | BirdNET | Picture Bird | **Nuestro** |
|---|---|---|---|---|
| % de confianza explicable | ✗ | Solo etiqueta | ✗ | ✓ |
| Escuchar las grabaciones que coincidieron | ✗ | ✗ | ✗ | ✓ |
| Fusión foto + canto | ✗ | ✗ | ✗ | ✓ |
| Referencias grabadas en México | ? | ? | ✗ | ✓ |
| Rechaza "no es un ave" | ✓ (aviso genérico) | Parcial (gato → perro) | ✗ (perro → búho) | ✓ |
| Dice a qué se parece el ruido | ✗ | Parcial | ✗ | ✓ |
| Corregir resultados y volver a buscar | ✗ | ✗ | ✗ | ✓ |
| Duración del audio | Libre | Selección manual | Mínimo 5 s | Libre (≥ 3 s), selección automática de las partes con sonido |
| Gratis y abierto | ✓ | ✓ | ✗ | ✓ |

Pruebas propias (exploratorias, una grabación cada una) con ladrido, maullido e imitación humana: Merlin no sugiere nada y no explica qué escuchó; BirdNET acierta el perro pero confunde el gato con perro; Picture Bird dice que el perro es un búho.

Tres citas de las propias apps como gancho:
- Merlin: tirano cuír *"casi idéntico al Tropical Kingbird… se distingue por su vocalización"*, pero no combina foto y canto.
- Picture Bird: *"la misma especie puede sonar diferente debido al dialecto"*, pero usa grabaciones de EE. UU.
- BirdNET: "Highly likely / Uncertain", una confianza sin explicación.

### 6. Nuestra propuesta (vista del usuario)
Flujo en 4 pasos con íconos:
1. Grabas o subes el canto.
2. El sistema busca los cantos más parecidos de su colección.
3. Te muestra la especie, sus fotos, el % de confianza y **las grabaciones parecidas para que las escuches**.
4. Si no te convence: marcas "se parece / no se parece" en las grabaciones y el sistema vuelve a buscar con tu opinión.
- Opcional: *"¿Lo alcanzaste a ver?"* Subes una foto y se combina con el canto.
- Si es un ruido: *"Esto se parece más a un ladrido que a cualquier ave"*. Si el audio es malo: consejos para volver a grabar.

### 7. Cómo funciona por dentro
Diagrama del flujo:
```
Audio (≥ 3 s) → ventanas de 3 s → solo las que tienen sonido (RMS) → vector (MFCC | BirdNET) → búsqueda de los k más parecidos
      → votación por especie → ¿similitud suficiente? → sí: especie + fotos + evidencia
                                                      → no: "no es un ave, se parece a: perro"
Usuario marca "se parece / no se parece" → Rocchio ajusta la consulta → nueva búsqueda
Foto (opcional) → vector (color+HOG | CLIP) → puntuación por especie ─┐
                                                                     ├→ fusión con pesos
Audio ───────────────────────────────────────────────────────────────┘
```
- Mensaje clave: **la búsqueda es el clasificador**. No entrenamos modelos; recuperamos y votamos.

### 8. Conexión con el curso
| Práctica previa | Qué hicimos | Qué hacemos ahora |
|---|---|---|
| Recuperación de sonidos (MFCC) | 40 audios, 5 clases, P@5 ≈ 0.57 | Miles de segmentos de aves; MFCC contra BirdNET |
| CBIR tradicional | 48 imágenes sintéticas; color + HOG | Fotos reales; color + HOG contra CLIP |
| Descriptor combinado | Pesos ajustados sobre la misma colección (optimista) | Pesos elegidos con un conjunto de validación separado |
| Métricas (Labs 6–7) | P@k, curvas PR | P@k, mAP, curvas PR, matriz de confusión |
| Modelo vectorial y relevancia (teoría de RI) | Consultas de texto | Retroalimentación de relevancia (Rocchio) aplicada a audio |

Mensaje: *el proyecto lleva a cabo las mejoras que propusimos en nuestras propias prácticas.*

### 9. Datos
- **Cantos:** xeno-canto e iNatSounds, priorizando grabaciones hechas en México.
- **Fotos:** NaturaLista/iNaturalist (licencias CC).
- **Sonidos de "no ave":** ESC-50 (perro, gato, ranas, grillos, lluvia, cláxon, motosierra), el mismo dataset de la práctica.
- **Especies (12):** zanate mayor, gorrión doméstico, paloma huilota, tórtola coquita, luis bienteveo, mosquero cardenal, pinzón mexicano, chachalaca oriental, carpintero cheje, urraca hermosa cara blanca, momoto cejiazul, colibrí pico ancho.
- **Par difícil:** tirano cuír contra tirano tropical.
- Nota: la lista final depende de cuántos datos haya por especie.

### 10. Experimentos y cómo los medimos
| # | Experimento | Pregunta | Métrica |
|---|---|---|---|
| E1 | MFCC contra BirdNET | ¿Cuánto mejora la IA al método clásico en audio? | P@5, mAP, curvas PR |
| E2 | Color + HOG contra CLIP | ¿Y en imagen? | P@5, mAP |
| E3 | Foto sola, canto solo y fusión (par difícil) | ¿Combinar ayuda donde la foto falla? | Exactitud y matriz de confusión |
| E4 | Rechazo de ruidos (ESC-50) | ¿Evita confundir ranas o cláxones con aves? | Tasa de falsos positivos |
| E5 | Retroalimentación de relevancia (Rocchio) | ¿Una ronda de "se parece / no se parece" mejora el resultado? Se simula con las etiquetas. | P@5 y mAP antes y después |
| E6 | Prueba de campo con celular | ¿Funciona con audio real fuera de la colección? | Aciertos en grabaciones propias |

- Regla: las ventanas de una misma grabación nunca quedan repartidas entre colección y consultas, para no hacer trampa.

### 11. Así se verá (maqueta)
- Boceto de la página: zona para subir o grabar audio, espectrograma con las ventanas usadas marcadas, tarjeta de resultado (foto, nombre, % de confianza), lista "escucha las 5 más parecidas" con botones "se parece / no se parece" y botón "confirmar con foto".
- Hecha con Gradio y publicada en Hugging Face Spaces (link para probarla desde el celular).

### 12. Alcance
- **Incluye (MVP):** búsqueda por canto, evidencia, % de confianza, rechazo con "se parece a…", retroalimentación "se parece / no se parece", experimentos de imagen y fusión (en el reporte), ficha básica, evaluación y página web.
- **Extras si hay tiempo:** fusión con foto dentro de la página, alerta de motosierra y línea de tiempo de grabaciones largas.
- **No incluye:** app móvil, entrenar modelos, búsqueda foto → canto, todas las especies de México, ficha rica (dieta, mapas).

### 13. Plan de trabajo
Resumen de las fases de abajo en una línea de tiempo con los colores de cada integrante.

### 14. Riesgos y plan B
| Riesgo | Plan B |
|---|---|
| Pocas grabaciones de alguna especie | Cambiarla por otra común con más datos |
| BirdNET no corre en nuestras máquinas | Perch v2 o PANNs; en último caso MFCC + deltas |
| Falta tiempo para la fusión en la página | Queda como experimento en el reporte |
| La página publicada falla el día de la demo | Demo local + video grabado de respaldo |

### 15. Entregables y cierre
- Página web funcionando (demo en vivo).
- Reporte con metodología, resultados y conclusiones.
- Código en GitHub.
- Diapositivas finales.
- Cierre: *"No competimos con Merlin. Construimos un buscador transparente para aves de México: te dice qué ave es, te muestra por qué, te avisa si no es un ave y aprende de tu opinión."* Preguntas.

---

## Plan de actividades
Tiempo estimado entre paréntesis (por actividad, no por día). **A** = integrante audio, **B** = integrante imagen y página.

### Fase 0: Validar antes de construir
- Contar grabaciones y fotos disponibles por especie (A + B, 1–2 h)
- Probar BirdNET con 10 audios en Colab o en la laptop (A, 1 h)
- Cerrar la lista final de especies (A + B, 30 min)
- Guardar la prueba de campo del mosquero cardenal (A, 10 min)

### Fase 1: Datos
- Descargar cantos, cortarlos en ventanas de 3 s y descartar las silenciosas con RMS (A, 3–4 h)
- Descargar fotos con licencia CC (B, 2–3 h)
- Preparar los negativos de ESC-50 (B, 30 min)
- Separar colección, validación y prueba por grabación (A, 1 h)

### Fase 2: Recuperación de audio
- Línea base MFCC, reutilizando la práctica (A, 1–2 h)
- Embeddings de BirdNET (A, 2–3 h)
- Búsqueda kNN + votación + % de confianza (A, 1–2 h)
- Umbral de rechazo "no es un ave" + "se parece a…" (A, 1–2 h)
- Retroalimentación de relevancia Rocchio + experimento simulado (A, 1–2 h)

### Fase 3: Recuperación de imagen
- Línea base color + HOG, reutilizando la práctica (B, 1–2 h)
- Embeddings de CLIP (B, 1–2 h)
- Fusión audio + foto con pesos elegidos en validación (B, 2–3 h)
- Experimento del par difícil (B, 1 h)

### Fase 4: Evaluación
- P@k, mAP, curvas PR y matriz de confusión, reutilizando los Labs 6–7 (A + B, 2–3 h)
- Prueba de campo con grabaciones propias (A + B, 1 h)

### Fase 5: Página web
- Página Gradio: subir audio, resultado, evidencia, botones "se parece / no se parece" y ficha básica (B, 3–5 h)
- Publicar en Hugging Face Spaces (B, 1 h)

### Fase 6: Entrega
- Reporte (A + B, 4–5 h)
- Diapositivas finales (A + B, 2 h)
- README y limpieza del repo (A, 1 h)
- Ensayo de la demo + video de respaldo (A + B, 1 h)

### Antes de todo: presentación de propuesta
- Armar estas diapositivas (A + B, 2–3 h)

**Total estimado: ~42–51 h** entre los dos. Cabe en 10 días a 2–3 h diarias por persona. Si se aprieta, se recortan primero los extras y después la fusión en la página (queda en el reporte).

**Orden y dependencias:** Fase 0 → Fase 1 → (Fases 2 y 3 en paralelo, una por persona) → Fase 4 → Fases 5 y 6. La página (Fase 5) puede empezar con datos de prueba mientras termina la Fase 4.
