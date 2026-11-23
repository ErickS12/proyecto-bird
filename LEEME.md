# ¿Quién canta? — Lo que vamos a hacer

Este documento explica el proyecto final en 5 minutos. Los detalles están en `docs/`.

## La idea en una frase
Una página web donde **grabas el canto de un ave y te dice qué especie es, te muestra cómo se ve y te deja escuchar las grabaciones en las que se basó para decidirlo**. Funciona con especies de México.

## ¿Por qué esta idea?
- En el bosque **se oye mucho más de lo que se ve**: las aves se esconden y tomarles una buena foto es difícil.
- La profesora recomendó basarnos en nuestras prácticas de **recuperación de imágenes (CBIR)** y de **recuperación de sonidos (MFCC)**. Este proyecto usa las dos.
- Ya existen apps (Merlin, BirdNET, Picture Bird). Las revisamos con pruebas propias (capturas en `docs/referencias/`) y a todas les falta algo:
  - Ninguna te deja **escuchar las grabaciones** que usó para decidir.
  - Ninguna da un **% de confianza explicado**.
  - Ninguna **combina foto y canto**, aunque Merlin mismo dice que hay aves que solo se distinguen por el canto.
  - Con ruidos que no son aves fallan: Picture Bird dijo que un perro era un búho, y BirdNET confundió un gato con un perro.

Frase para presentarlo: *"No competimos con Merlin. Construimos un buscador transparente para aves de México: te dice qué ave es, te muestra por qué, te avisa si no es un ave y aprende de tu opinión."*

## Cómo funciona (versión sencilla)
1. **Todo se vuelve un vector.** Cada canto se convierte en una lista de números que resume cómo suena, igual que en la práctica de MFCC. También probamos con BirdNET, una red ya entrenada con millones de cantos que genera vectores mejores.
2. **Buscar es medir distancias.** Con similitud coseno se buscan en la colección los cantos más parecidos al tuyo, como en la práctica.
3. **La búsqueda es el clasificador.** Si de los 5 cantos más parecidos 4 son de mosquero cardenal, la respuesta es "mosquero cardenal, 80%". No entrenamos ningún modelo.
4. **Pedazos de 3 segundos.** El audio se corta en ventanas de 3 s y se descartan las silenciosas midiendo la energía (RMS). Se vota entre las que quedan.
5. **"Esto no es un ave".** Si nada se parece lo suficiente, el sistema no inventa. Usamos los sonidos de ESC-50 (perro, gato, ranas, lluvia, cláxon…) para que diga "esto se parece más a un ladrido".
6. **"Se parece / no se parece".** El usuario marca qué grabaciones de la evidencia se parecen a la suya y el sistema vuelve a buscar. Es **retroalimentación de relevancia (Rocchio)**, teoría de RI.
7. **La foto como confirmación.** Si también viste al ave, la foto se compara igual (color + HOG de la práctica de CBIR, contra CLIP) y se combina con el audio: `puntuación = w·audio + (1−w)·foto`. El peso `w` se elige con datos aparte, no con los mismos de la evaluación (el error que anotamos en la práctica de CBIR).
8. **Cómo sabemos que funciona.** Con las métricas del curso (P@5, mAP, curvas PR), siempre comparando **método clásico contra IA**.

## Datos
- **Cantos:** xeno-canto e iNatSounds, de preferencia grabados en México.
- **Fotos:** NaturaLista/iNaturalist (solo con licencia libre).
- **Ruidos que no son aves:** ESC-50, el mismo de la práctica.
- **12 especies candidatas:** zanate, gorrión, paloma huilota, tórtola coquita, luis bienteveo, mosquero cardenal, pinzón mexicano, chachalaca, carpintero cheje, urraca hermosa, momoto cejiazul y colibrí pico ancho. Se confirman cuando veamos cuántos datos hay.
- **Experimento estrella:** tirano cuír contra tirano tropical. Son casi idénticos en foto y se distinguen por el canto. Medimos foto sola, canto solo y la combinación.

## Cómo nos repartimos
Dos papeles. Decidimos juntos quién toma cuál:

| | Persona A: audio | Persona B: imagen y página |
|---|---|---|
| Datos | Descargar y cortar cantos; separar colección y pruebas | Descargar fotos; preparar ESC-50 |
| Recuperación | MFCC contra BirdNET, votación, rechazo, Rocchio | Color + HOG contra CLIP, fusión, par difícil |
| Producto | README y repo | Página en Gradio y publicarla |
| Juntos | Fase 0, evaluación, reporte, diapositivas y ensayo de la demo | |

El plan completo, por actividades y con tiempo estimado, está en `docs/plan-diapositivas.md`. Son unas 42–51 h entre los dos.

## Orden de trabajo
1. **Fase 0: validar** (antes de programar nada): contar cuántas grabaciones y fotos hay por especie, probar que BirdNET corre y cerrar la lista de especies.
2. **Fase 1: datos.**
3. **Fases 2 y 3 en paralelo:** audio (A) e imagen (B).
4. **Fase 4: evaluación.**
5. **Fases 5 y 6:** página web y entrega.

Antes de todo eso: **presentarle la propuesta a la profesora** con las diapositivas.

## Lo que NO vamos a hacer
App móvil, entrenar modelos, buscar canto a partir de foto, todas las especies de México, ni una ficha completa del ave (dieta, mapas). La ficha es básica: nombre, fotos, 1–2 cantos y una descripción corta. **El foco es la recuperación de imagen y sonido.**

## Documentos
| Archivo | Qué tiene |
|---|---|
| `docs/ideas/aves-mexico.md` | La idea completa: auditoría de apps, diferenciadores, datos, supuestos, alcance |
| `docs/plan-diapositivas.md` | Contenido de las 15 diapositivas + plan de actividades |
| `docs/referencias/` | Capturas de Merlin, BirdNET y Picture Bird |

## Pendientes por resolver
- ¿Qué pide exactamente la rúbrica de la profesora?
- ¿Demo publicada en Hugging Face Spaces o local?
- ¿Nos enfocamos en especies de una región en particular?
