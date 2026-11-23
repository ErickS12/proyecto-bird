# ¿Quién canta? — Recuperación de aves mexicanas por su canto y su imagen

## Problema (How Might We)
¿Cómo podríamos ayudar a turistas y cuidadores del bosque a saber qué ave escuchan, aunque no alcancen a verla, mostrándoles **quién es, cómo se ve y por qué el sistema lo cree**?

## Dirección elegida
Página web donde el usuario sube o graba un canto. El sistema corta el audio en ventanas de 3 s, se queda con las que tienen sonido, busca las grabaciones más parecidas en una colección de especies mexicanas y decide la especie por votación de los k vecinos más cercanos. La búsqueda es el clasificador: no se entrenan modelos. Muestra la especie, sus fotos, un % de confianza y **las grabaciones de referencia que más se parecieron, para escucharlas** como evidencia.

Une las dos prácticas del curso. **Audio:** MFCC (práctica) contra embeddings de BirdNET. **Imagen:** CBIR clásico (color + HOG, práctica) contra CLIP, y fusión de las puntuaciones de audio y foto, con pesos elegidos en un conjunto de validación separado. Eso corrige la limitación que anotamos en la práctica de CBIR. La evaluación usa P@k, mAP y curvas PR (Labs 6–7).

Frase de venta: *No competimos con Merlin. Construimos un buscador transparente para aves de México: te dice qué ave es, te muestra por qué, te avisa si no es un ave y aprende de tu opinión.*

## Auditoría de apps similares
Revisadas con capturas y pruebas propias (22 nov 2026). Capturas en `docs/referencias/`.

| App | Qué hace | Hueco que aprovechamos |
|---|---|---|
| **Merlin Bird ID** (Cornell) | Graba el tiempo que quieras y analiza en ventanas de 3 s, con espectrograma reproducible. Photo ID con 5 candidatos. Ficha completa: fotos, hábitats, sonidos de referencia, mapa, frecuencia por mes en tu localidad. Paquete México con CONABIO. Funciona sin internet. | No muestra % de confianza ni las grabaciones que llevaron a la decisión. Ante un ladrido responde "sin sugerencias" con un aviso genérico (micrófono tapado, especie no incluida, poca confianza), sin decir a qué se parece. **No combina foto + canto**, aunque su propia ficha dice que el tirano cuír y el tirano tropical "se distinguen por su vocalización". |
| **BirdNET** (Cornell + TU Chemnitz) | Solo sonido. Grabar, seleccionar a mano un intervalo del espectrograma y ver una lista de especies ordenada por probabilidad con etiqueta cualitativa ("Highly likely", "Uncertain", "Highly uncertain"). | La confianza es solo una etiqueta, sin explicar de dónde sale. No muestra grabaciones de referencia similares. Obliga a elegir el intervalo a mano. No tiene foto ni fusión. Acertó un ladrido, pero clasificó un maullido como perro. |
| **Picture Bird** | Foto y sonido. Consejos para tomar la foto. Candidatos 1, 2, 3, ?. Ficha muy completa (nombres regionales, dieta, anidación, cantos, clasificación). Pide mínimo 5 s de audio. | Es de pago (las identificaciones ilimitadas y el mapa están bloqueados). No muestra % de confianza. Sus cantos de referencia están grabados en EE. UU., aunque la propia app advierte que "la misma especie puede sonar diferente debido al dialecto". No combina foto + canto. Identificó un ladrido como búho y una imitación humana como ave. |

Las pruebas propias son de una grabación cada una: son exploratorias, no conclusiones.

## Diferenciadores (MVP)
1. **Evidencia escuchable y confianza explicable:** las k grabaciones más parecidas para escucharlas, las ventanas usadas marcadas en el espectrograma y un % de confianza basado en votos ("4 de 5 grabaciones más parecidas son de esta especie").
2. **Rechazo honesto "esto no es un ave":** umbral de similitud + sonidos de ESC-50 (perro, gato, ranas, grillos, insectos, lluvia, cláxon, motosierra). En lugar de un aviso genérico, dice a qué se parece: "se parece más a un ladrido que a cualquier ave de la colección". Se mide la tasa de falsos positivos.
3. **Retroalimentación de relevancia (Rocchio):** botones "se parece / no se parece" en cada grabación de evidencia. La consulta se ajusta (q' = α·q + β·media(relevantes) − γ·media(no relevantes)) y se vuelve a buscar. Si el audio es malo (similitud muy baja con todo), se sugiere volver a grabar con consejos concretos.
4. **Fusión foto + canto:** cuando el usuario también alcanzó a ver al ave.
5. **Especies mexicanas con grabaciones hechas en México** (nombres comunes según la lista de CONABIO).

**Duración del audio:** se acepta cualquier duración ≥ 3 s, sin selección manual. El sistema descarta las ventanas silenciosas con energía (RMS) y vota la especie entre las demás. Con menos de 3 s se pide volver a grabar.

**Extras si hay tiempo:** fusión con foto dentro de la página, alerta de motosierra o disparo para guardaparques y línea de tiempo de grabaciones largas con conteos.

## Datos
- **Cantos:** xeno-canto (API; verificar si pide llave) e iNatSounds (audios de iNaturalist etiquetados por especie, para investigación). Se priorizan las grabaciones hechas en México.
- **Fotos:** NaturaLista/iNaturalist (solo licencias CC).
- **Sonidos de "no ave":** ESC-50 (el mismo de la práctica).
- **Modelos preentrenados:** BirdNET (audio), CLIP (imagen). Alternativa de audio: Perch v2.
- **Especies candidatas (12, se confirman en la Fase 0):** zanate mayor, gorrión doméstico, paloma huilota, tórtola coquita, luis bienteveo, mosquero cardenal, pinzón mexicano, chachalaca oriental, carpintero cheje, urraca hermosa cara blanca, momoto cejiazul, colibrí pico ancho. El cenzontle se excluye porque imita a otras aves.
- **Par difícil (experimento estrella):** tirano cuír (*Tyrannus couchii*) contra tirano tropical (*T. melancholicus*). Son casi idénticos en foto y se distinguen por el canto. Se compara foto sola, canto solo y fusión.

## Supuestos a validar
- [ ] Hay ≥ 30 grabaciones por especie en xeno-canto/iNatSounds: contar con la API (Fase 0).
- [ ] BirdNET corre en las laptops o en Colab: extraer embeddings de 10 audios (Fase 0).
- [ ] Las fotos con licencia CC alcanzan (≥ 20 por especie): consultar la API de iNaturalist (Fase 0).
- [ ] Un audio de celular funciona igual que los de xeno-canto: probar con grabaciones propias fuera de la colección. Ya hay una: el mosquero cardenal del 22 nov, identificado así por Merlin y por BirdNET (guardar en `datos/prueba_campo/`).
- [ ] Los embeddings superan a MFCC con un margen claro: P@5 y mAP (Fase 2).

## MVP
**Entra:**
- Colección de 10–15 especies.
- Búsqueda por canto: MFCC contra BirdNET, votación kNN, % de confianza y evidencia.
- Rechazo con "se parece a…" y retroalimentación de relevancia.
- Experimentos de imagen: color + HOG contra CLIP, y fusión (en el reporte).
- Página Gradio: audio → especie + fotos + evidencia + botones.
- Evaluación con P@k, mAP y curvas PR; reporte; repo en GitHub.

**Ficha de especie (complemento, versión básica):** nombre común, nombre científico, 3–5 fotos, 1–2 cantos de referencia y una descripción corta.

## No hacemos (y por qué)
- **Ficha rica (dieta, anidación, mapas, estacionalidad):** Merlin y Picture Bird ya lo hacen bien y no aporta a la recuperación.
- **App móvil:** no cabe en el tiempo; la página web funciona en el celular.
- **Entrenar o afinar modelos:** los preentrenados bastan y mantienen el proyecto en recuperación de información.
- **Búsqueda foto → canto:** si ya ves al ave, ya resolviste lo difícil.
- **Todas las especies de México (1100+):** trabajo futuro.
- **Cuentas, mapas, comunidad:** no aportan a la evaluación.
- **Buscar silbando, modo quiz:** vistosos pero arriesgados; trabajo futuro.

## Preguntas abiertas
- ¿Qué pide exactamente la rúbrica de la profesora (peso del reporte, la demo y el código)?
- ¿Publicar en Hugging Face Spaces o correr la demo local?
- ¿La lista final de especies se enfoca en una región (por ejemplo, el centro de México)?
