<p align="right"><a href="README.md">한국어</a> · <a href="README.en.md">English</a> · <a href="README.ja.md">日本語</a> · <a href="README.zh.md">中文</a> · <a href="README.fr.md">Français</a> · <b>Español</b></p>

# 🔍 Upscaly: ampliador de imágenes que aspira a no perder información (무손실 지향 확대기) `v1.2.1`

Upscaly es una herramienta de ampliación de imágenes que aspira a no perder información. Funciona con un solo archivo (`upscaler.html`) y realiza todos los cálculos dentro del navegador.
No hay nada que instalar ni ningún sitio al que subir nada. La imagen nunca sale del dispositivo.

Versión actual **v1.2.1** · los cambios figuran en [12. Registro de cambios](#12-registro-de-cambios). El código fuente de desarrollo, las pruebas y el material de investigación están en el repositorio [Upscaly-dev](https://github.com/Velvitty/Upscaly-dev).

**Índice**

1. [Introducción sencilla](#1-introducción-sencilla)
2. [Principios](#2-principios)
3. [Funcionamiento](#3-funcionamiento)
4. [Algoritmos y fórmulas](#4-algoritmos-y-fórmulas)
5. [El mundo físico](#5-el-mundo-físico)
6. [Arquitectura](#6-arquitectura)
7. [Problemas y soluciones](#7-problemas-y-soluciones)
8. [Optimización](#8-optimización)
9. [Verificación](#9-verificación)
10. [Puntos fuertes](#10-puntos-fuertes)
11. [Limitaciones](#11-limitaciones)
12. [Registro de cambios](#12-registro-de-cambios)
13. [Referencias](#13-referencias)

---

## 1. Introducción sencilla

### Qué es esta herramienta

Al agrandar una imagen pequeña, lo habitual es que se vuelva borrosa, que aparezcan escalones o que surja un borde blanco alrededor de los contornos. Upscaly amplía cada tipo de imagen —fotografía, ilustración, cómic, documento, logotipo, pixel art— con el método adecuado para ella, reduciendo estos defectos y conservando al máximo la información que contenía la imagen original.

«Aspirar a no perder información» significa que la restauración se ajusta para que, al reducir la imagen ampliada de nuevo a su tamaño original, coincida con el original. La ampliación no inventa imagen que no estuviera en el original; se concentra en desplegar de la forma más verosímil la información que el original contiene. Las correcciones que se añaden encima, como el realce de contornos o la eliminación de escalones, son etapas que pulen esta restauración para que resulte agradable a la vista; al activarlas, el resultado se aparta un poco de la información original ([2.1](#21-ampliar-no-crea-información)).

### Ventajas

- **No hay nada que instalar.** Basta con abrir un único archivo en el navegador. Ni siquiera hace falta conexión a internet.
- **La imagen no sale del dispositivo.** No se usa ningún servidor ni ninguna biblioteca externa.
- **Se ajusta por sí sola.** El botón 「Automático」(자동) identifica el tipo de imagen entre 9 clases y fija los ajustes midiendo directamente sobre esa imagen.
- **Aguanta ampliaciones grandes.** La imagen se calcula por teselas y cada fila de teselas se comprime y se encadena, de modo que, aunque el resultado tenga cientos de millones de píxeles, nunca se carga en memoria el resultado completo sin comprimir. Eso sí, el archivo PNG comprimido se acumula en memoria hasta que termina el guardado.
- **Degradados sin bandas.** Al guardar en 8 bits se mezcla un tramado fino para que no aparezcan bandas escalonadas en cielos y degradados ampliados. También se puede guardar en PNG de 16 bits.
- **Los colores de las fotos de móvil, tal cual.** Las fotos Display P3 de iPhone y otros dispositivos se leen en P3 y se guardan en P3, para que los rojos y verdes intensos no se apaguen.
- **Restauración de textura (opcional).** Se puede volver a añadir una textura fina a piel o tejidos que, al ampliarlos mucho, han quedado lisos como cera. Está desactivada por defecto.
- **Las imágenes transparentes también quedan limpias.** El color no se corre en los bordes semitransparentes.
- **Dos objetivos: precisión y percepción.** Por defecto, 「Precisión」(정확도) amplía lo más cerca posible del original. 「Percepción」(체감) vuelve los contornos más pronunciados y hace aflorar el contraste enterrado, para que la imagen se vea más nítida. No aparecen bordes blancos (halos).

### Modo de uso

1. Abra `upscaler.html` en un navegador actual.
2. Pulse **Abrir imagen** (이미지 열기), arrastre la imagen a la pantalla o péguela. Se pueden abrir hasta 20 imágenes a la vez, en JPEG, PNG, WebP o AVIF.
3. Elija el **factor** (배율). Puede basarse en la longitud de ancho y alto (1–12×) o en el número de píxeles (superficie); si elige 「Automático」, se selecciona el mayor factor (3–8×) que este dispositivo puede procesar hasta el final.
4. Pulse **「Automático: (recomendado)」(자동: (권장, 추천))**. Mide el tipo y el estado de la imagen, rellena todos los ajustes de abajo y anota brevemente en qué se ha basado. Si verse nítido importa más que ser fiel al original, elija 「Percepción」 en **Objetivo** (목표).
5. Pulse **Ejecutar ampliación** (확대 실행). Si **Comparar con el original** (원본 비교) está activado, la mitad izquierda de la pantalla muestra el original ampliado por el navegador y la mitad derecha el resultado. La vista se desplaza arrastrando y se amplía con la rueda; están los botones **Ajustar** (맞춤, tecla F), **1:1** y **Comparar con el original** (tecla C).
6. **Guardar PNG** (PNG 저장) guarda la imagen actual y **Guardar todo** (모두 저장) guarda de una vez todas las imágenes procesadas.

### Si elige los ajustes usted mismo

Los paneles de la derecha de la pantalla están dispuestos en el orden de procesamiento.

| Panel | Qué hace |
|---|---|
| Factor (배율) | Cuántas veces ampliar. 1× solo limpia, sin ampliar |
| Preajuste · automático (프리셋 · 자동) | 33 preajustes o 「Automático」 |
| Objetivo (목표) | 「Precisión」 (lo más cerca del original, por defecto) o 「Percepción」 (sube la pendiente de los contornos y el contraste local para verse más nítido, intensidad 25–150 %). Los preajustes y 「Automático」 no lo cambian |
| Núcleo de reconstrucción (재구성 커널) | Cómo rellenar los píxeles nuevos (DCCI, pixel art, Lanczos, Catmull-Rom, Mitchell, vecino más próximo), ampliación por etapas, bloqueo del ringing, consistencia con el original |
| Limpieza de ruido y manchas (노이즈·잡티 정리) | Elimina el ruido y los artefactos JPEG antes de ampliar |
| Recuperación de nitidez (선명도 복원) | Deshace antes de ampliar el desenfoque producido al fotografiar o escanear |
| Eliminación de escalones (계단 제거) | Elimina los escalones promediando solo en la dirección del contorno, redibujado de contornos, suavizado de bloques JPEG |
| Realce de contornos (윤곽 강조) | Máscara de enfoque con supresión de halos |
| Restauración de textura (질감 되살리기) | Añade una textura fina que no estaba en el original. Desactivada por defecto; se recomienda 20–30 % para fotos y pintura |
| Procesamiento del color (색 처리) | Balance de blancos (función aparte que ni los preajustes ni el modo automático tocan), cálculo en luz lineal, tramado de 8 bits, guardado en 16 bits |

Las casillas y los controles deslizantes funcionan exactamente como se eligen. Por ejemplo, 「Ampliar por etapas」(단계로 나눠 확대), si se activa, divide la ampliación aunque según la medición el resultado sea peor. A cambio, el indicador que hay al lado informa del resultado medido sobre la imagen actual. Justo después de abrir una imagen, el valor inicial de esta casilla se decide midiendo la imagen; si antes de que termine esa medición cambia la casilla o pulsa 「Automático」, la medición que termine tarde no toca esa elección.

### Conviene saber

- Una imagen que ya se ha reducido y vuelto a guardar muchas veces ha perdido los detalles que tenía, y no se pueden recuperar. Upscaly solo despliega de la forma más verosímil la información que queda; no inventa caras ni letras que no existen.
- Para logotipos y planos de los que exista un original vectorial (SVG, AI), siempre es mejor usar ese original.
- Al ampliar 8×, el resultado tiene cientos de megapíxeles y tarda mucho. El indicador de progreso muestra el tiempo restante.

---

## 2. Principios

### 2.1 Ampliar no crea información

Una imagen digital es un conjunto de muestras de una escena continua tomadas a intervalos regulares. Los patrones más finos que el intervalo de muestreo nunca llegaron a registrarse. Por eso la esencia de la ampliación es «cómo rellenar el espacio entre muestras», y según cómo se rellene aparecen defectos como desenfoque, escalones o ringing.

Upscaly sigue tres principios al elegir cómo rellenar. Los dos primeros son restricciones de la etapa de restauración (interpolación → límite de celda → consistencia) y se cumplen cuando están activadas las casillas correspondientes (「Bloquear ringing y halos」(링잉·후광 차단), 「Corrección de consistencia con el original」(원본 정합 보정)).

1. **No crear extremos nuevos.** Cada píxel restaurado queda entre el valor más oscuro y el más claro de la celda 2×2 del original a la que pertenece (límite de celda). El ringing y los halos son extremos que no estaban en el original, así que en esta etapa no aparecen.
2. **Al reducir de nuevo se obtiene el original.** Se ajusta la imagen restaurada para que, al reducirla con el promedio de área de la luz, coincida con el original (corrección de consistencia con el original). Es decir, no contradice la información del original.
3. **Los contornos se rellenan siguiendo su dirección.** Mezclar en dirección transversal al contorno lo desenfoca, y mezclar a lo largo de los ejes crea escalones. Se lee la dirección en la que se extiende el contorno y se rellena solo en esa dirección (DCCI).

Es difícil cumplir las dos restricciones (1 y 2) a la vez: al ajustar una, la otra se desvía un poco. Por eso se usa una proyección alterna que las ajusta por turnos ([4.6](#46-proyección-alterna)).

La máscara de enfoque, el suavizado direccional, el redibujado de contornos y la restauración de textura que vienen después de la restauración son etapas que pulen el resultado para la vista y no vuelven a respetar estas dos restricciones. Por eso, el resultado final con ellas activadas puede salirse un poco del rango de la celda del original o diferir un poco del original al reducirlo. El «error de consistencia de la ampliación» que muestra la pantalla se mide con el valor anterior a estas correcciones. Preajustes como 「Intervención mínima · Lanczos puro」(최소 개입 · 순수 Lanczos) desactivan ambas restricciones.

### 2.2 La luz se mezcla en luz lineal

Los valores guardados para la pantalla (sRGB) están curvados para ajustarse al ojo humano. Para mezclar la luz de dos píxeles hay que hacerlo con la cantidad real de luz (luz lineal), no con estos valores curvados. Si se interpola directamente sobre los valores curvados, el límite entre una zona clara y una oscura se oscurece y se empasta. Upscaly calcula por defecto en luz lineal y solo vuelve a sRGB al guardar. Si se desactiva la casilla 「Calcular en luz lineal」(선형광에서 계산), se calcula directamente en el espacio gamma (no recomendado).

Las imágenes transparentes se calculan con el color multiplicado por el alfa (premultiplicado). Además, en la preparación del original, en las etapas intermedias de la ampliación por etapas y en el cálculo final de teselas, **el alfa pasa exactamente por las mismas operaciones que el color.** Si el color y el alfa se filtraran de forma distinta, el color de los bordes semitransparentes se torcería al dividir después el color entre el alfa.

### 2.3 El desenfoque y el ruido se tratan antes de ampliar, a la resolución original

Las lentes de las cámaras y los escáneres captan la escena algo desenfocada, y los sensores y la compresión añaden ruido. Estos defectos se producen a la resolución original, así que lo correcto es deshacerlos a la resolución original. Si se tratan después de ampliar, los defectos ya han crecido también, y es más difícil y más lento.

- El **desenfoque** se estima a la inversa con la deconvolución de Richardson–Lucy. Casi toda la razón por la que la imagen ampliada parece más nítida que el original está aquí.
- El **ruido** se elimina tomando como referencia el espectro de ruido medido directamente en la imagen. Se conservan las componentes mayores que el ruido (contornos y texturas) y solo se reducen las que no se distinguen del ruido.
- Los **artefactos JPEG** se eliminan leyendo la tabla de cuantización del archivo. Los coeficientes DCT de la luminancia se devuelven para que no salgan del intervalo de cuantización del archivo original, de modo que no se borran los detalles registrados en el archivo. Esta garantía solo se aplica a la luminancia en esta etapa de proyección. Las componentes de color, y el suavizado de bloques, la limpieza de ruido, la deconvolución, la ampliación y el enfoque posteriores, pueden mover los coeficientes fuera del intervalo.

### 2.4 Las líneas se vuelven a dibujar

En los gráficos en los que dos colores se separan de forma tajante (logotipos, dibujo lineal, texto), los propios escalones del original tienden a conservarse como si fueran la «respuesta correcta». Upscaly busca estos límites y los redibuja como contornos de 1 píxel de anchura a la resolución ampliada. Toma la isolínea del 50 % del valor interpolado suave como posición del contorno y levanta el perfil según esa distancia (método de distancia con signo). El suavizado direccional endereza la forma del contorno y el redibujado levanta el perfil, así que usándolos juntos se obtienen líneas suaves, sin escalones y sin difuminarse.

### 2.5 Una textura inventada no es textura, sino grano

La ampliación no puede crear patrones más finos que la separación entre píxeles del original. Por eso la textura de la piel, la trama de los tejidos o las nervaduras de las hojas de una foto muy ampliada desaparecen y se ven lisas como la cera. La **cantidad** de textura perdida sí se puede estimar, porque en las imágenes naturales la energía contenida es parecida cada vez que la frecuencia espacial se duplica (el espectro de potencia es aproximadamente $`1/f^2`$). Con la intensidad de textura medida en la banda fina del original se puede calcular aproximadamente cuánta textura había en las octavas que faltan.

Pero la **posición y la forma** de la textura no están en el original. Lo que añade Upscaly es un ruido ajustado a esa cantidad, es decir, una textura fina como el grano de película. Como su posición no coincide con la respuesta correcta, cuanto más se acerca su energía a la correcta, mayor es la diferencia con el original (PSNR), y a la vista empieza a parecer más ruido que textura. Por eso la restauración de textura es una función opcional que 「Automático」 no toca, y se recomienda usarla a baja intensidad sobre superficies lisas como retratos, piel o tejidos. Para dibujar de forma verosímil incluso la forma de la textura hace falta un modelo generativo (red neuronal) ([11. Limitaciones](#11-limitaciones)).

### 2.6 Los escalones de 8 bits se ocultan con ruido fino

El procesamiento se hace en coma flotante, pero el PNG de 8 bits que se guarda normalmente solo registra 256 niveles de brillo. Si se amplía 8 veces un cielo suave, el brillo que en el original cambiaba un nivel cada pocos píxeles se convierte en bandas de decenas de píxeles de anchura. Si justo antes de redondear se mezcla un ruido menor que medio nivel (tramado), los límites de las bandas se dispersan en una textura fina y dejan de verse. El ruido es un patrón que depende solo de las coordenadas de pantalla, así que es igual en todas las teselas, y se aplica por igual a los tres canales para que el gris siga siendo gris. Los valores que ya caen exactamente, como los de una superficie de color uniforme, no cambian.

### 2.7 El color de las fotos de móvil es más amplio que sRGB

Muchas fotos de móvil actuales se toman en la gama Display P3, más amplia que sRGB. Si se leen con un lienzo normal (sRGB), el navegador comprime dentro de sRGB los rojos y verdes intensos que quedan fuera, y el resultado ampliado sale más apagado que el original. Upscaly lee el perfil de color del archivo original y, si es de gama amplia, lo lee con un lienzo P3 y guarda también el resultado con un perfil P3. Display P3 usa la misma curva de transferencia que sRGB, de modo que el cálculo en luz lineal no cambia; solo difieren los primarios.

### 2.8 Cada imagen tiene su propia respuesta

Los ajustes adecuados para una foto estropean un dibujo lineal, y los adecuados para un dibujo lineal estropean una foto. La consistencia con el original es buena en fotos, pero en un dibujo lineal en el que dos colores se separan de forma tajante restaura como correctos los escalones del original. Por eso 「Automático」 primero identifica el tipo de imagen (clasificador de 9 clases), parte del preajuste adecuado para ese tipo y ajusta los detalles con valores medidos sobre esa imagen. Para ampliaciones grandes usa valores hallados explorando un espacio de 1.52 billones de configuraciones ([9.4](#94-exploración-de-la-restauración-a-gran-escala)).

### 2.9 Precisión y percepción son objetivos distintos

El resultado más cercano al original (el de PSNR más alto) no es el que se ve más nítido. La ampliación embota incluso los contornos que la resolución de origen puede contener (pendiente de contorno 0.81× la de la referencia) y debilita el contraste a la escala de unos pocos píxeles de origen (0.94×). El ojo percibe ambas cosas como nitidez. Por eso hay dos objetivos. 「Precisión」 es la restauración de siempre; 「Percepción」 devuelve, sobre ella, estas dos magnitudes al nivel de la referencia. No crea nuevos extremos, así que no aparecen bordes blancos, ni dibuja textura que el original no tenía. A cambio, las cifras frente al original bajan un poco ([4.21](#421-nitidez-percibida), [9.10](#910-nitidez-percibida)).

---

## 3. Funcionamiento

### 3.1 Flujo general

```
Archivo ─▶ decodificación (orientación EXIF) ─▶ perfil de color (lienzo P3 si es de gama amplia) ─▶ cabecera JPEG (tabla de cuantización, muestreo) ─▶ recuadro del sujeto
                                              │
              「Automático」: imagen de análisis ─▶ clasificación en 9 clases ─▶ preajuste + ajuste medido ─▶ valores explorados de restauración a gran escala / medición del suavizado direccional / medición de la ampliación por etapas
                                              │
Preparación del original (resolución original) : plano en luz lineal (premultiplicado) ─▶ procesamiento del color ─▶ restauración DCT de precisión ─▶ suavizado de bloques ─▶ limpieza de ruido ─▶ deconvolución RL ─▶ etapas intermedias de la ampliación por etapas
                                              │
Cálculo de teselas (grupo de workers)   : interpolación ─▶ límite de celda ─▶ proyección alterna consistencia↔límite de celda ─▶ estadísticas ─▶ máscara de enfoque ─▶ suavizado direccional ─▶ redibujado de contornos ─▶ (textura) ─▶ deshacer premultiplicación ─▶ codificación (tramado de 8 bits)
                                              │
                         caché de teselas (LRU) ─▶ pantalla / guardado PNG por filas de teselas (8 y 16 bits, sRGB o P3)
```

### 3.2 Carga

El archivo se decodifica primero con `createImageBitmap` (aplicando la orientación EXIF); si falla, se recurre a la Blob URL de `<img>` y después a una Data URL, por este orden. Los archivos de gama amplia (apartado 2.7) se detectan antes de decodificar y usan primero la vía `<img>` (Blob URL y, si falla, Data URL); solo si ambas fallan se recurre a `createImageBitmap`. El motivo es que la conversión de color predeterminada de `createImageBitmap` varía de un navegador a otro y puede recortar a sRGB. Con archivos grandes, el navegador puede tardar de verdad decenas de segundos, así que el tiempo límite se fija en proporción al tamaño del archivo (45 s + 9 s por MB, máximo 5 minutos). En ninguna vía queda una espera sin terminar.

Si es JPEG, solo se lee la cabecera para obtener la tabla de cuantización, la relación de muestreo (4:2:0, etc.), la calidad estimada y la orientación EXIF. Los datos entrópicos no se descodifican. Si el decodificador ha aplicado la orientación EXIF se comprueba por el ancho y el alto del resultado decodificado, y la cuadrícula JPEG de 8×8 se vuelve a alinear según la orientación de visualización.

También se busca el perfil de color en el archivo (se siguen las cabeceras de fragmentos, chunks y cajas y solo se lee lo necesario, de modo que se encuentra aunque haya metadatos grandes o cajas de imagen delante). En JPEG es el fragmento APP2 `ICC_PROFILE`; en PNG, los chunks cICP, iCCP y sRGB (con esta prioridad, sea cual sea su orden); en WebP, el chunk ICCP; en AVIF y HEIF, la caja colr asociada a la imagen principal. Si la gama es más amplia que sRGB, el lienzo del original se crea como `display-p3`, se dibuja sobre él y en la línea de información se indica «Display P3». Si el navegador no admite lienzos P3, se lee en sRGB como antes.

Para cada imagen, el original se reduce a unos 360 px y se buscan masas de prominencia (distancia al color medio) para formar el recuadro del sujeto. Se puede ver con el botón 「Sujeto」(피사체) de la pantalla.

### 3.3 Preparación del original (`prepareSource`)

Aquí se hacen todas las correcciones que terminan a la resolución original. El resultado se guarda en caché junto con la clave de los ajustes, hasta los dos más recientes, de modo que aunque el resultado ampliado salga de la caché se reutiliza si los ajustes son los mismos.

1. **Plano en luz lineal.** El sRGB de 8 bits se convierte a luz lineal con una tabla de 256 entradas y, si hay alfa, el color se multiplica por el alfa.
2. **Procesamiento del color.** Solo si la intensidad del balance de blancos no es 0 (por defecto, 0).
3. **Restauración DCT de precisión.** Limpia el original con el espectro de ruido y la tabla de cuantización JPEG. Se reparte en bandas entre los workers y la cuadrícula DCT se fija a coordenadas globales, de modo que no hay costuras en los límites de las bandas.
4. **Suavizado de bloques JPEG.** Solo alisa los saltos en posiciones múltiplos de 8 cuando son menores que la variación del entorno.
5. **Limpieza de ruido.** Promedia solo entre vecinos de brillo parecido (filtro bilateral). El alfa también se promedia con los mismos pesos.
6. **Deconvolución RL.** El alfa también pasa por el mismo número de iteraciones.
7. **Etapas intermedias de la ampliación por etapas.** Si un factor grande se divide en dos o más, aquí se amplía hasta antes de la última etapa. DCCI usa la interpolación direccional también en las etapas intermedias, y el pixel art restaura los contornos en cada etapa. El alfa pasa por los mismos pesos direccionales, límite de celda y enfoque entre etapas que el color. La referencia del límite de celda y de la consistencia de la etapa final es el **original**, no la imagen intermedia.

### 3.4 Cálculo de teselas (`renderTile`)

El resultado se calcula dividido en teselas (de 512 a 1024 px según el factor; con factores enteros, múltiplo del factor). Cada tesela recibe el fragmento del original que necesita (soporte de interpolación + margen de consistencia + margen del campo de direcciones y del modelo de contornos) y se calcula de forma independiente, de modo que el resultado es el mismo sea cual sea la tesela que se calcule primero.

1. **Interpolación.** Con DCCI se amplían todos los canales a la vez (`dcciStageMulti`). Los pesos direccionales se calculan una sola vez en el plano guía (luminancia gamma + término de contorno del alfa) y se comparten entre canales. Después se ajusta al factor exacto con Catmull-Rom.
2. **Límite de celda.** Se recorta con el mínimo y el máximo por canal de la celda 2×2 del original.
3. **Proyección alterna de la corrección de consistencia y el límite de celda.** Se proyecta dos veces por turnos (si el límite de celda está desactivado, una sola vez la consistencia). Cada pasada se resuelve en una región 7 píxeles más ancha que la región de resultado, de modo que ninguna pasada lee fuera de la cuadrícula de coeficientes.
4. **Estadísticas.** Para mostrar en pantalla la diferencia con el original al reducir el resultado (error de consistencia de la ampliación), se mide con el valor anterior a las correcciones.
5. **Máscara de enfoque.** Suprime los halos con un limitador de rodilla proporcional a la amplitud local.
6. **Suavizado direccional.** Promedia solo en la dirección tangente hallada con el tensor de estructura del original.
7. **Redibujado de contornos.** Vuelve a dibujar contornos de 1 píxel de anchura con la distancia con signo.
8. **Nitidez percibida.** Solo cuando el objetivo es 「Percepción」, sube la pendiente de los contornos y el contraste local en la luminancia, sin superar nunca el rango de brillo del entorno ([4.21](#421-nitidez-percibida)).
9. **Restauración de textura.** Solo si está activada: añade al resultado la textura fina que le falta, según la intensidad de textura medida en el original. En la ampliación por etapas se mide en el original, no en la imagen intermedia. No se usa con el vecino más próximo, que replica el original tal cual, ni con el núcleo de pixel art, que conserva la paleta; en esos casos el control deslizante también se desactiva.
10. **Deshacer la premultiplicación y codificar.** Se convierte a 8 bits para la pantalla y a 8 o 16 bits para guardar. En 8 bits, si el tramado está activado (por defecto), justo antes de redondear se mezcla un ruido fino determinado por las coordenadas. La pantalla y el guardado en 8 bits usan las mismas teselas, así que se guarda exactamente lo que se ve.

El grupo de workers lanza tantos como núcleos − 1 (máximo 8), y si no responden en 5 segundos se calcula en el hilo principal con el mismo motor. El motor (`ENGINE_FACTORY`) es un conjunto de funciones de cálculo puras, así que el mismo código se carga también como worker mediante Blob URL.

### 3.5 Ampliación por etapas

Es el método de ampliar un factor grande en varias veces. Se elige entre: primero 2×, primero el factor grande, equilibrado (primero el menor o primero el mayor), uniforme (√×√) y tres etapas. Se puede aplicar enfoque entre etapas. El indicador reduce el centro de la imagen actual por el factor objetivo con el promedio de área de la luz, compara con el original (PSNR) la ampliación de una sola vez y la ampliación por etapas con el método actual, e indica si el estado actual de la casilla es el mejor. El resultado se recuerda por imagen y ajustes.

### 3.6 Automático

1. **Análisis.** Si el lado largo supera 1024, se reduce a la mitad con promedios 2×2 y después con promedios de caja de factor entero hasta 512 o menos, y en esa imagen de análisis se miden 27 características: tensor de estructura (proporción de textura, bordes alineados con los ejes), bordes fuertes y líneas de contorno, periodo de las líneas de texto, rectas largas, composición de color y brillo, residuo plano de bloques 8×8 en un fragmento a resolución original, cuadrícula de ampliación por vecino más próximo, número de colores del original, tramas de puntos, etc.
2. **Clasificación.** Una regresión logística multinomial elige entre fotografía, ilustración, pintura, cómic, dibujo lineal, documento, UI, logotipo y pixel art. El pixel art no se decide por probabilidad sino por estructura (cuadrícula de vecino más próximo de factor entero, o paleta reducida a 1×). En pantalla se muestran los motivos que impulsaron la decisión (contribución de cada característica a la diferencia de logits entre el primero y el segundo).
3. **Preajuste y ajuste medido.** Parte del preajuste adecuado para el tipo y, según el ruido (mediana del laplaciano en zonas planas), la anchura de los bordes, el contraste de la cuadrícula de 8 px y la calidad JPEG, ajusta las iteraciones y la σ de la deconvolución, la restauración JPEG, el suavizado de bloques, etc.
4. **Valores explorados de restauración a gran escala.** Si el tipo y el factor cumplen las condiciones de validación, se usan tal cual los ajustes hallados por exploración (`AUTO_BIG`). Fotografía, dibujo lineal, UI y pintura desde 2×; documento desde 4×. Logotipo y cómic solo desde 2× y cuando el archivo es JPEG, porque con entradas limpias no superaron la validación ([9.4](#94-exploración-de-la-restauración-a-gran-escala)). Para ilustración no se usan.
5. **Medición por imagen.** Si no se usan los valores explorados, en los gráficos (salvo fotografía y pixel art) se elige la intensidad del suavizado direccional midiéndola en esa imagen (`autoAA`) y después se miden todos los métodos de ampliación por etapas y se elige el mejor (`autoStage`).

Las peticiones automáticas llevan un número, de modo que una medición anterior que termine tarde no se aplica a una imagen abierta después. Si se cambia el factor después del modo automático, se avisa para que se vuelva a pulsar.

### 3.7 Factor automático

Elige, entre 3× y 8×, el mayor factor cuyo renderizado está **garantizado**. Garantizado significa que la memoria máxima de todo el proceso está dentro del 80 % del presupuesto de memoria, que lo que sobra es mayor que la caché mínima y que el resultado está dentro del límite de guardado. También se cuentan el PNG comprimido que se acumula en memoria durante el guardado (tasa de compresión máxima medida en 47 imágenes del conjunto de datos, con un margen del 10 %) y el original preparado de una imagen anterior que siga en la caché. Como los ajustes que elige 「Automático」 (preajuste, valores explorados de restauración a gran escala, ampliación por etapas) cambian con el factor, y con ellos el presupuesto, 「Automático」 aplica de verdad, empezando por los factores grandes, los ajustes de cada factor y elige el primero que queda garantizado. Al decidir la ampliación por etapas midiéndola en esta foto, también elige solo métodos dentro de la garantía. Una vez elegido, el límite de la caché de teselas se reduce realmente a ese valor. Si ni siquiera 3× está garantizado, se recurre al mayor factor garantizado por debajo de él.

### 3.8 Guardado

Si el navegador admite `CompressionStream('deflate')`, se comprime cada fila de teselas (en 16 bits, bandas de 128 filas) y el PNG se construye encadenando los fragmentos. En 8 bits se reutilizan las teselas de la caché; en 16 bits se recalcula cada banda. No se crea ningún lienzo grande que contenga el resultado completo, pero los fragmentos comprimidos se acumulan en memoria hasta terminar el guardado y se exportan como un solo archivo. El archivo incluye un chunk de espacio de color y un chunk iTXt `Software` (UTF-8) con el núcleo y el factor. El chunk de espacio de color suele ser sRGB; en las imágenes leídas en P3 es cICP (primarios 12 = Display P3, transferencia 13 = curva sRGB) junto con un perfil ICC Display P3 equivalente (iCCP). Los programas que leen cICP usan ese chunk; los que no, usan el ICC. Los navegadores sin compresión en flujo solo pueden guardar en 8 bits: reúnen el resultado en un único lienzo y lo guardan con el codificador del navegador, y si se supera el límite del lienzo aprenden el límite real de ese dispositivo. En estos navegadores, si se elige guardar en 16 bits, el guardado falla.

### 3.9 Procesamiento del color

El procesamiento del color es una función aparte de la ampliación y su valor predeterminado es 0. Al pulsar 「Color automático」(색 자동), el color de la iluminación se estima con tres métodos (media, bordes, zonas claras) y la intensidad del balance de blancos se fija solo en la medida en que los tres coinciden. Una dominante de iluminación real tiñe incluso las zonas claras, de modo que los tres crecen juntos, mientras que el color propio del sujeto, como un gato naranja o una puesta de sol, solo arrastra la media. Los colores de ilustraciones, pinturas, UI, logotipos y pixel art se consideran intencionados y no se corrigen. El brillo global se mantiene y la ganancia de cada canal se limita a 0.55–1.8.

---

## 4. Algoritmos y fórmulas

En lo que sigue, $`I`$ es el original (luz lineal, premultiplicado), $`U`$ el resultado y $`s_x, s_y`$ los factores horizontal y vertical.

### 4.1 Espacio de color

```math
\mathrm{lin}(u) =
\begin{cases}
u / 12.92 & u \le 0.04045 \\[2pt]
\left(\dfrac{u + 0.055}{1.055}\right)^{2.4} & \text{en otro caso}
\end{cases}
\qquad
\mathrm{srgb}(v) =
\begin{cases}
12.92\,v & v \le 0.0031308 \\[2pt]
1.055\,v^{1/2.4} - 0.055 & \text{en otro caso}
\end{cases}
```

De 8 bits a luz lineal se usa una tabla de 256 entradas, y de luz lineal a 8 bits una tabla de 8192 entradas (error de cuantización inferior a 1 nivel). El codificador de 16 bits construye su tabla sobre el eje $`\sqrt{v}`$, de modo que se ajusta dentro de 1 nivel incluso en el tramo empinado cercano al negro. Si los 256 niveles de 8 bits se convierten a luz lineal y se devuelven, los 256 niveles quedan intactos ([9.1](#91-invariantes)).

Si hay alfa, el cálculo se hace con los valores premultiplicados $`(\alpha R, \alpha G, \alpha B, \alpha)`$ y, justo antes de codificar, si $`\alpha > 10^{-7}`$ se divide entre $`\alpha`$.

### 4.2 Núcleos de reconstrucción

Se aplican de forma separable (horizontal → vertical), alineando los centros de los píxeles ($`u = (X + 0.5)/s - 0.5`$). Al reducir, el núcleo se estira según el factor para evitar el aliasing.

```math
L_a(x) = \mathrm{sinc}(x)\,\mathrm{sinc}(x/a)\ \ (|x| < a),\qquad
k_{B,C}(x) = \tfrac16
\begin{cases}
(12 - 9B - 6C)|x|^3 + (-18 + 12B + 6C)|x|^2 + (6 - 2B) & |x| < 1 \\
(-B - 6C)|x|^3 + (6B + 30C)|x|^2 + (-12B - 48C)|x| + (8B + 24C) & 1 \le |x| < 2
\end{cases}
```

Lanczos-3 y 4 corresponden a $`a = 3, 4`$; Catmull-Rom a $`(B, C) = (0, 0.5)`$; Mitchell–Netravali a $`(1/3, 1/3)`$. El vecino más próximo replica el original tal cual con factores enteros, de modo que al reducir de nuevo coincide bit a bit con el original.

### 4.3 DCCI adaptativa a los bordes

Basada en la interpolación por convolución cúbica direccional de Zhou et al. (2012), amplía 2 veces en cada etapa (alineación por esquinas: salida $`(2i, 2j)`$ = entrada $`(i, j)`$).

**Etapa 1: centro de la celda 2×2.** Se mide la variación en las dos direcciones diagonales en un vecindario 4×4. $`G`$ es el plano guía (luminancia gamma 0–255) y $`n_{\mathrm{off}}`$ el suelo de ruido.

```math
d_1 = \sum_{(p,q) \in \nearrow} |G_p - G_q| + n_{\mathrm{off}},\qquad
d_2 = \sum_{(p,q) \in \searrow} |G_p - G_q| + n_{\mathrm{off}}
```

Cada suma recorre los 9 pares vecinos en la dirección diagonal correspondiente. Los pesos son los siguientes ($`T = 1.15`$).

```math
(w_a, w_b) =
\begin{cases}
(0, 1) & 1 + d_1 > T(1 + d_2) \\
(1, 0) & 1 + d_2 > T(1 + d_1) \\
\left(\dfrac{a}{a+b}, \dfrac{b}{a+b}\right),\ a = \dfrac{1}{1 + d_1^{5}},\ b = \dfrac{1}{1 + d_2^{5}} & \text{en otro caso}
\end{cases}
```

Si la variación en $`\nearrow`$ es grande, el contorno se extiende en la dirección $`\searrow`$, así que se interpola en esa dirección. El valor de cada dirección usa los coeficientes de convolución cúbica $`(-1, 9, 9, -1)/16`$.

```math
p_a = \tfrac{1}{16}\big(9(f_{12} + f_{21}) - (f_{03} + f_{30})\big),\quad
p_b = \tfrac{1}{16}\big(9(f_{11} + f_{22}) - (f_{00} + f_{33})\big),\quad
v = \mathrm{clip}\big(w_a p_a + w_b p_b,\ \min_4 f,\ \max_4 f\big)
```

El `clip` final es un añadido que no está en el trabajo original. Impide que el nuevo valor salga del rango de los 4 vecinos más próximos, de modo que el ringing y los halos no aparecen ya desde esta etapa.

**Etapa 2: píxeles entre horizontales y verticales.** Se aplica el mismo procedimiento en direcciones horizontal y vertical sobre una cuadrícula girada 45°. Los bordes se pliegan por reflexión especular para conservar la paridad.

**Número de etapas.** Se usa una tabla fijada midiendo cada factor (`EDI_PLAN`). Hasta 2×, una etapa con cuadrícula de 2×; por encima, se amplía hasta la cuadrícula de 4× (dos etapas) y luego se ajusta al factor exacto con Catmull-Rom ([8. Optimización](#8-optimización)).

### 4.4 Límite de celda

Se recorta con el rango por canal de la celda del original $`(i..i{+}1,\ j..j{+}1)`$ a la que pertenece la muestra de salida $`U(X, Y)`$.

```math
U(X, Y) \leftarrow \mathrm{clip}\Big(U(X, Y),\ \min_{(p,q) \in \mathrm{cell}} I_{p,q},\ \max_{(p,q) \in \mathrm{cell}} I_{p,q}\Big)
```

La ampliación por vecino más próximo siempre está dentro de este rango, así que existe solución. Los mínimos y máximos de cada celda se calculan una sola vez por plano de canal (caché `cellRange`).

### 4.5 Corrección de consistencia con el original

Si $`D`$ es la operación que reduce el resultado por promedio de área, el objetivo es $`D(U) = I`$. A la estimación inicial $`U_0`$ se le suman unos coeficientes $`c`$ a resolución original, desplegados con la interpolación bilineal $`P`$.

```math
U = U_0 + P(c),\qquad D(U_0 + P c) = I \;\Longrightarrow\; K c = I - D(U_0),\quad K = D P
```

Como $`D`$ y $`P`$ son separables por ejes, $`K = K_x \otimes K_y`$, y la $`K`$ de cada eje es una matriz tridiagonal. Por tanto, $`c`$ se resuelve exactamente a resolución original con el algoritmo de Thomas. Los coeficientes de cada eje se construyen a partir de la huella en el original de cada píxel de salida (incluidos los solapamientos fraccionarios). La atenuación de la matriz inversa es de unas 0.17 veces por píxel, así que la influencia más allá de 7 píxeles es inferior a $`0.17^7 \approx 4 \times 10^{-6}`$. Por eso resolver cada tesela con un margen de 7 píxeles da lo mismo que resolver el conjunto de una vez.

### 4.6 Proyección alterna

La condición de consistencia $`\{U : D U = I\}`$ es un conjunto afín y el límite de celda $`\{U : m \le U \le M\}`$ es un conjunto caja. Ambos son convexos, así que proyectar alternativamente sobre ellos acerca el resultado a su intersección.

```math
U^{(k+1)} = \Pi_{\mathrm{box}}\Big(\Pi_{\mathrm{affine}}\big(U^{(k)}\big)\Big),\qquad k = 0, 1
```

Upscaly proyecta dos veces (si el límite de celda está desactivado, una sola vez la consistencia). La proyección de consistencia no es una proyección ortogonal estricta sino una corrección en la dirección de $`P`$, y como la última proyección es el límite de celda, la promesa de no crear extremos nuevos se cumple siempre.

### 4.7 Deconvolución de Richardson–Lucy

El desenfoque se modela como una gaussiana $`h`$ de desviación típica $`\sigma`$, y la estimación $`e`$ se actualiza de forma multiplicativa a partir de la observación $`o`$.

```math
e^{(t+1)} = e^{(t)} \cdot \left( h \ast \mathrm{clip}\!\left(\frac{o}{h \ast e^{(t)}}\right) \right)
```

El cociente se recorta para que no diverja. La actualización multiplicativa conserva los múltiplos constantes, de modo que si se aplica el mismo número de iteraciones al color premultiplicado y al alfa, la relación color/alfa se mantiene en las zonas del mismo color.

### 4.8 Limpieza de ruido (filtro bilateral)

Se aplica por separado en horizontal y en vertical, y los pesos por diferencia de brillo se toman de una tabla de 1025 entradas para evitar llamadas a la función exponencial.

```math
\hat I_p = \frac{\sum_q w_s(\|p - q\|)\, w_r(|G_p - G_q|)\, I_q}{\sum_q w_s(\|p - q\|)\, w_r(|G_p - G_q|)}
```

$`G`$ es el brillo aproximado en gamma. El plano alfa también se promedia con los mismos pesos.

### 4.9 Restauración DCT de precisión

**Modelo de ruido.** Para cada canal (Y, Cb, Cr), el ruido se representa con la varianza por frecuencia de la DCT 8×8, $`S_c[k]`$, y una ganancia por tramo de brillo, $`g_c(I)`$.

1. Se seleccionan repetidamente bloques con poca energía de alta frecuencia ($`u + v \ge 6`$) y se obtiene la forma del espectro con la media por frecuencia. La energía de la textura se concentra en las bajas frecuencias, así que para la selección y la ganancia solo se usan las altas.
2. Para la luminancia se pone un límite superior con el suelo de ruido PCA: la media de los 3 menores valores propios de la covarianza de 63 dimensiones de los bloques de textura débil. La textura solo se extiende en unas pocas direcciones, mientras que el ruido se extiende en las 63, así que se evita confundir textura con ruido y borrarla. El número finito de muestras se corrige con la cota inferior de Marchenko–Pastur $`\gamma = \sqrt{n / n_{\mathrm{sel}}}`$.
3. En caso de duda, se sesga hacia la subestimación (la que conserva los detalles). Una imagen limpia en la que solo queda el ruido de cuantización de 8 bits ($`1/12`$) se considera prácticamente 0.

**Eliminación del ruido.** Es una estructura de dos etapas sobre una DCT 8×8 deslizante, como BM3D sin la búsqueda de bloques. A partir del brillo medio del bloque $`\bar I`$, la varianza del ruido de cada coeficiente se fija en $`\sigma_k^2 = S[k]\, g(\bar I)\, \lambda^2`$ ($`\lambda`$ es la intensidad). $`F_k`$ son los coeficientes del bloque de entrada y $`B_k`$ los coeficientes del mismo bloque en la imagen resultante de la etapa 1.

```math
\text{Etapa 1: } \hat F_k = F_k \cdot \mathbf{1}\big[\,F_k^2 \ge \tau^2 \sigma_k^2\,\big],\qquad
\text{Etapa 2: } \tilde F_k = F_k \cdot W_k,\quad W_k = \frac{B_k^2}{B_k^2 + \sigma_k^2}
```

Los bloques solapados se combinan con una ventana de Kaiser ($`\beta = 2`$) y, como en BM3D, con un peso por bloque de $`1/(1 + n_{\mathrm{nz}})`$ en la etapa 1 y de $`1/\sum_k W_k^2 \sigma_k^2`$ en la etapa 2 ($`n_{\mathrm{nz}}`$ es el número de coeficientes que quedan). La cuadrícula de bloques se fija a coordenadas globales, así que no hay costuras entre teselas.

**Proyección sobre el intervalo de cuantización JPEG (QCS).** Si es JPEG, la varianza del ruido de cuantización se suma al modelo de ruido usando la tabla de cuantización del archivo $`q_k`$, y los coeficientes DCT de la luminancia se devuelven al intervalo alrededor del valor cuantizado observado. Los datos entrópicos no se descodifican, así que los coeficientes observados se obtienen redondeando con la tabla de cuantización los valores $`G_k`$ de la DCT de la imagen decodificada alineada con la cuadrícula 8×8 del archivo.

```math
c_k = q_k \cdot \mathrm{round}\!\left(\frac{G_k - \ell_k}{q_k}\right) + \ell_k,\qquad
F_k \leftarrow \mathrm{clip}\big(F_k,\ c_k - \Delta_k,\ c_k + \Delta_k\big),\qquad
\Delta_k = \tfrac12 \rho_k q_k + \delta_k
```

$`\ell_k`$ es el desplazamiento de nivel de 128, presente solo en DC. Para los coeficientes AC registrados con un valor distinto de 0, salvo los de frecuencia más baja ($`u + v \ge 2`$, la textura real que dejó el codificador), el intervalo se estrecha con $`\rho_k = \rho`$ (por defecto 0.5) para conservar la intensidad de la textura; para los coeficientes 0, DC y los de frecuencia más baja (origen de los bloques y el ringing) se usa el intervalo completo con $`\rho_k = 1`$ para borrar los artefactos. $`\delta_k = 2.5\sigma_k`$ (solo AC) es el margen para eliminar el ruido cuando la imagen se comprimió con ruido. (Eliminación de ruido → proyección) se repite dos veces, y en la segunda la varianza del ruido de cuantización se reduce a la mitad.

**Color.** Los planos de color se limpian del mismo modo y después, con un filtro guiado, los límites de color que el submuestreo de color (como 4:2:0) ha difuminado se ajustan a los contornos de la luminancia restaurada (localmente, color $`= a \cdot`$ luminancia $`+ b`$).

**Recuperación de detalles.** Si no es JPEG, cuando la energía local de lo eliminado $`v`$ supera la cantidad explicable por el modelo de ruido $`\tau \sigma^2`$, se devuelve la proporción de ese exceso.

```math
\hat I \leftarrow \hat I + \mu \left(1 - \frac{\tau \sigma^2}{v}\right)(I - \hat I)\qquad (v > \tau \sigma^2)
```

### 4.10 Máscara de enfoque (supresión de halos)

Se calculan el detalle $`d = I - G_\sigma \ast I`$ y la amplitud local $`A = G_{2\sigma} \ast |d|`$, y se tratan en este orden.

```math
d \leftarrow d \cdot \frac{d^2}{d^2 + t^2}\ \ (\text{protección de zonas planas}),\qquad
d \leftarrow a\, d,\qquad
d \leftarrow \frac{d}{1 + |d| / (o A + 10^{-4})}\ \ (\text{tolerancia de halo})
```

$`a`$ es la intensidad, $`t`$ la protección de zonas planas y $`o`$ la tolerancia de halo. La última expresión es un limitador de rodilla suave, de modo que en los contornos grandes la sobreoscilación no supera $`o`$ veces la amplitud local. Con radio 0, el radio se fija automáticamente según el factor.

### 4.11 Suavizado direccional (eliminación de escalones)

En el original se calcula el tensor de estructura $`J = G \ast (\nabla Y \nabla Y^{\top})`$; el vector propio del valor propio menor se toma como dirección tangente $`\mathbf t`$, y con el valor propio mayor $`\lambda_1`$ se fija la intensidad $`w = \min(1, \lambda_1 / 0.0016)`$. En la imagen ampliada se promedia solo en la dirección tangente.

```math
\bar U = 0.34\,U(\mathbf x) + 0.22\,\big(U(\mathbf x + \ell \mathbf t) + U(\mathbf x - \ell \mathbf t)\big) + 0.11\,\big(U(\mathbf x + 2\ell \mathbf t) + U(\mathbf x - 2\ell \mathbf t)\big),\qquad
U \leftarrow U + w\,\kappa\,(\bar U - U)
```

El periodo de los escalones es igual al factor, así que el paso $`\ell`$ se fija en 0.8 veces el factor (1–8). Una sola pasada no basta, así que se repite de 1 a 4 veces en proporción a la intensidad. Con cada repetición la oscilación disminuye multiplicativamente y la anchura de la transición no cambia. El campo de direcciones se busca en coordenadas globales, así que da el mismo valor también en los límites de las teselas.

### 4.12 Redibujado de contornos (distancia con signo)

Si los píxeles de una ventana 5×5 del original caen sobre el segmento entre el color oscuro y el claro $`(\mathrm{lo}, \mathrm{hi})`$ y el cambio se produce en 1–2 píxeles, se considera un contorno. La imagen ampliada se convierte en la posición $`t \in [0, 1]`$ sobre el segmento y se redibuja una rampa de 1 píxel de anchura con la distancia con signo hasta la isolínea 0.5.

```math
t' = \mathrm{clip}\!\left(0.5 + \frac{t - 0.5}{|\nabla t|},\ 0,\ 1\right),\qquad
U \leftarrow U + \gamma\,w_{\mathrm{fit}}\big(\mathrm{lo} + t'(\mathrm{hi} - \mathrm{lo}) - U\big)
```

$`|\nabla t|`$ es el gradiente por píxel de salida. En el cálculo real, el objetivo es $`U + (t' - t)(\mathrm{hi} - \mathrm{lo})`$ recortado a $`[\mathrm{lo}, \mathrm{hi}]`$, y el peso de mezcla $`\gamma\,w_{\mathrm{fit}}`$ es un peso construido a resolución original a partir de la intensidad del redibujado y del grado de ajuste (cuánto se aparta del segmento el 3×3 central), desplegado de forma bilineal. El redibujado sobre la isolínea del 50 % solo conserva el área (densidad) en líneas de más de 1 píxel de anchura, así que las líneas finas cuya anchura, medida por la densidad integrada transversalmente a la línea, ronda 1 píxel no se tocan, para evitar que se ondulen.

### 4.13 Restauración de contornos de pixel art

La forma la determina el campo DCCI (los escalones a 45° y 2:1 se convierten en rectas), y el color se fija al más cercano al valor del campo entre los 4 colores originales de la celda 2×2 del original a la que pertenece cada submuestra. Para cada píxel de salida se promedian 4×4 submuestras y solo se suavizan (antialiasing) los contornos, de modo que los colores nuevos solo aparecen en el límite de los contornos. Si los 4 colores de la celda son iguales, se salta por una vía rápida.

### 4.14 Consenso del balance de blancos

「Color automático」 excluye de la imagen de análisis (luz lineal) los píxeles saturados (0.95 o más) y casi negros (0.004 o menos), y obtiene el color de la fuente de luz con tres estimadores.

```math
e^{\mathrm{sog}}_c = \Big(\tfrac1n \sum f_c^{6}\Big)^{1/6},\qquad
e^{\mathrm{ge}}_c = \Big(\tfrac1n \sum |\nabla (G \ast f_c)|^{6}\Big)^{1/6},\qquad
e^{\mathrm{wp}}_c = Q_{0.99}(f_c)
```

Son Shades of Gray (media), Gray Edge (bordes) y White Patch (zonas claras, percentil 99). $`G`$ es un suavizado $`[1,4,6,4,1]/16`$. La intensidad $`a`$ se fija con el ángulo $`\theta`$ que forma cada estimación con el neutro $`(1,1,1)`$.

```math
a =
\begin{cases}
0 & \theta_{\min} < 3^\circ \ \text{o}\ \theta_{\min} > 28^\circ \ \text{o}\ \theta_{\max} - \theta_{\min} > 10^\circ \\[2pt]
\min\!\big(1,\ \theta_{\min} / \theta_{\mathrm{sog}}\big) \cdot h,\quad h = \begin{cases} \tfrac12 & \theta_{\mathrm{wp}} < 0.8\,\theta_{\mathrm{sog}} \\ 1 & \text{en otro caso} \end{cases} & \text{en otro caso}
\end{cases}
```

Las tres condiciones son, por orden: sin sesgo, color propio de la escena difícil de atribuir a la iluminación, e iluminación mixta. Si las zonas claras están menos teñidas que la media, se considera que el color del sujeto ha arrastrado la media y solo se corrige la mitad. La intensidad se redondea en pasos de 0.05. Al procesar, la ganancia de cada canal se obtiene de un máximo de 150 000 píxeles, excluidos los semitransparentes (alfa inferior a 0.5), los saturados y los casi negros; se normaliza para mantener la luminancia global ($`0.2126R + 0.7152G + 0.0722B`$), se mezcla con $`1 + (g - 1)a`$ y se recorta a 0.55–1.8.

### 4.15 Clasificador de 9 clases

Se aplica una regresión logística multinomial a $`\mathbf z`$, las características de 27 dimensiones $`\mathbf x`$ estandarizadas con la media y la desviación típica del entrenamiento.

```math
P(k \mid \mathbf x) = \frac{\exp(\mathbf w_k^{\top} \mathbf z + b_k)}{\sum_j \exp(\mathbf w_j^{\top} \mathbf z + b_j)}
```

Como el número de muestras varía por clase, se usan pesos de equilibrio de clases, y la entropía cruzada con regularización L2 se minimizó con L-BFGS-B. Las características afectadas por la calidad JPEG (bloques de un solo color, bloques sin ruido, suelo de ruido, bloques lisos) se multiplican por la fiabilidad de la calidad $`r = \mathrm{clip}((q - 50)/40,\ 0.05,\ 1)`$. Los motivos de la decisión son las contribuciones por característica $`z_j (w_{k_1 j} - w_{k_2 j})`$ a la diferencia de logits entre el primero $`k_1`$ y el segundo $`k_2`$, ordenadas de mayor a menor.

### 4.16 Presupuesto de renderizado

Con factor $`L`$, número de píxeles del original $`n_0`$ y número de píxeles del resultado $`N`$, la memoria máxima de cada etapa se estima así (en bytes).

```math
\begin{aligned}
M_{\mathrm{prep}} &= \max\big(32\,n_0,\ \ 4 c\, n_0 + 32\, n_0 s_k^2\big) && \text{preparación del original, etapas intermedias de la ampliación por etapas} \\
M_{\mathrm{fixed}} &= M_{\mathrm{other}} + M_{\mathrm{src}}^{1} + M_{\mathrm{keep}} + n_w (T + 64 + 2t + 2p)^2 (48 + 24[t > 0] + 32[p > 0]) + 1.5\,W \max(4T, 1024) + 1.1\,\rho\, c\, b\, N && \text{otros originales y originales preparados, planos retenidos, teselas de workers, banda de guardado, PNG comprimido} \\
\text{garantizado} &\iff M_{\mathrm{other}} + M_{\mathrm{src}}^{2} + M_{\mathrm{prep}} \le 0.8B \ \wedge\ 0.8B - M_{\mathrm{fixed}} \ge 4 \min\big(N, \max(4 \times 10^6, 2WT)\big)
\end{aligned}
```

$`c`$ es el número de canales, $`s_k`$ el factor intermedio de la ampliación por etapas, $`n_w`$ el número de workers, $`T`$ el tamaño de tesela, $`t`$ el margen de resultado de la restauración de textura (0 si está desactivada), $`W`$ el ancho del resultado y $`B`$ el presupuesto de memoria. $`M_{\mathrm{src}}^{2}`$ son todos los demás originales preparados que quedan en la caché (mientras se prepara uno nuevo quedan ambos), y $`M_{\mathrm{src}}^{1}`$ el más reciente de ellos (al meter el nuevo solo queda uno). $`b`$ son los bytes por canal (1 en 8 bits, 2 en 16 bits) y $`\rho`$ la tasa de compresión máxima del PNG guardado. Se midió con el codificador de la aplicación ampliando las 47 imágenes de `data/big` a 1.1, 1.25, 1.5, 2, 3 y 4×. Desde 2× se midieron tanto el caso con solo preajustes como el caso que además usa los valores explorados de restauración a gran escala de 「Automático」 (`AUTO_BIG`), y se toma el mayor; por debajo de 2× no se usan los valores explorados, así que solo preajustes (`research/budget/png_ratio.js`, `research/results/png_ratio.json`). Se usa el mayor valor de los dos factores medidos entre los que queda el factor del resultado (desde 4×, el valor de 4×). El factor por superficie de 1.25× equivale a 1.12× en ancho y alto, dentro del rango medido, y por debajo de 1.1× se cuenta con el tamaño sin comprimir. $`p`$ es el margen de resultado de 「Percepción」 (los dos desenfoques y el mínimo/máximo del entorno; 0 en 「Precisión」). La razón de compresión también se midió en 「Percepción」 (intensidad hasta 150 %, textura desactivada y activada), y se usa el mayor valor de los dos objetivos (`research/results/png_ratio_pop.json`).

| Factor | 8 bits, textura desactivada | 8 bits, textura activada | 16 bits, textura desactivada | 16 bits, textura activada |
|---|---|---|---|---|
| 1.1× | 0.721 | 0.767 | 0.928 | 0.933 |
| 1.25× | 0.686 | 0.766 | 0.913 | 0.928 |
| 1.5× | 0.650 | 0.760 | 0.898 | 0.926 |
| 2× | 0.688 | 0.790 | 0.876 | 0.933 |
| 3× | 0.615 | 0.751 | 0.842 | 0.931 |
| 4× o más | 0.562 | 0.731 | 0.817 | 0.934 |

En 8 bits son valores con el tramado activado, y con textura activada se toma el mayor de 25 % y 100 %. El último término es la caché de dos filas de teselas (mínimo 4 megapíxeles) que necesita el guardado. El presupuesto de memoria $`B`$ es el 15 % de `navigator.deviceMemory` (400–1500 MB); si no existe, el 50 % del límite del heap de JS (máximo 1500 MB); y si tampoco, 600 MB. En móviles es el 12 % (máximo 400 MB; si no existe, 180 MB).

### 4.17 Tramado de 8 bits

Al valor sRGB continuo justo antes de redondear, $`s = 255\,\mathrm{srgb}(v)`$, se le suma un ruido de gradiente entrelazado (IGN, Jimenez 2014) determinado solo por las coordenadas del resultado $`(X, Y)`$.

```math
\nu(X, Y) = \mathrm{frac}\!\big(52.9829189 \cdot \mathrm{frac}(0.06711056\,X + 0.00583715\,Y)\big) - \tfrac12,\qquad
b = \big\lfloor s + \tfrac12 + 0.98\,\nu \big\rfloor
```

Con una amplitud de ±0.49 niveles, los valores con $`|s - k| < 0.01`$ (código exacto $`k`$) siguen dando $`k`$. El valor sRGB continuo se obtiene interpolando la tabla sobre el eje $`\sqrt v`$ del codificador de 16 bits, con un error de interpolación máximo de 0.002 niveles. Se usa la misma $`\nu`$ en los tres canales, y el alfa no se trama.

### 4.18 Restauración de textura

A partir del brillo aproximado en gamma del original (luz lineal, premultiplicado) $`Y = \sqrt{\ell}`$, $`\ell = 0.2126R + 0.7152G + 0.0722B`$, se miden la intensidad local de la segunda banda más fina y el enmascaramiento por contornos.

```math
a = \mathrm{erode}_{5\times5}\!\left(\sqrt{G_{1.5} \ast (G_1 \ast Y - G_2 \ast Y)^2}\right),\qquad
m = \big(1 - c^2\big)^2,\quad c = \frac{\sqrt{(J_{xx} - J_{yy})^2 + 4J_{xy}^2}}{J_{xx} + J_{yy}}
```

$`G_\sigma`$ es una gaussiana, $`J = G_{1.5} \ast (\nabla Y \nabla Y^{\top})`$ el tensor de estructura y $`c`$ la coherencia direccional. El mínimo 5×5 (erosión) borra la intensidad de las estructuras fuertes y estrechas, como contornos, líneas finas y esquinas, y deja solo la textura ampliamente extendida. $`m`$ se acerca a 0 en los contornos que se extienden en una sola dirección.

A la resolución del resultado (factor $`L`$) se genera un ruido de varianza unitaria que conserva solo la banda por encima de la frecuencia de Nyquist del original. $`h(X,Y,s)`$ es un hash entero convertido a [0, 1).

```math
n = \sqrt3\Big(\textstyle\sum_{s=1}^{4} h(X, Y, s) - 2\Big),\qquad
\eta = \frac{n - G_{0.375L} \ast n}{\sqrt{1 - 2k_0^2 + (\sum_i k_i^2)^2}}
```

$`k_i`$ es el núcleo gaussiano unidimensional, y el denominador fija exactamente en 1 la varianza de esta diferencia. $`\sigma = 0.375L`$ es el valor cuya respuesta cae a la mitad en la frecuencia de Nyquist del original. Con la varianza objetivo y la varianza de la misma banda que ya tiene el resultado, solo se añade la parte que falta.

```math
T = \big(\alpha\, C\, \sqrt{\log_2 L}\; a\, m\big)^2,\qquad
E = G_{0.5L} \ast \big(Y_U - G_{0.375L} \ast Y_U\big)^2,\qquad
\Delta = \mathrm{clip}\!\big(\sqrt{\max(0, T - E)}\;\eta,\ \pm 0.4\,Y_U\big)
```

$`\alpha`$ es el control deslizante de intensidad, $`C = 6.256`$ una constante ajustada por medición ([9.8](#98-restauración-de-textura)) y $`Y_U`$ el brillo aproximado en gamma del resultado. Por último, los tres canales (premultiplicados) se multiplican por la misma relación $`f = \big((Y_U + \Delta)/Y_U\big)^2`$, de modo que el tono y el alfa no cambian. Al aclarar ($`f > 1`$), se reduce a $`f \le \max\!\big(1,\ \min(1, A) / \max(R, G, B)\big)`$ para que el canal mayor no supere su límite (el alfa, o 1 si es opaco). Si no, en colores claros y muy saturados solo se recorta un canal y el tono cambia (por ejemplo, 250·127·73 pasaba a 255·130·76; corregido en v1.1.1). El ruido, las estadísticas del original y la ventana de energía del resultado se determinan todos en coordenadas globales, y se calcula el margen adicional correspondiente, así que el resultado es el mismo con independencia de las teselas.

### 4.19 Detección de gama y perfil Display P3

La matriz de primarios del perfil ICC $`(\mathbf r, \mathbf g, \mathbf b)`$ (XYZ referido a D50) se compara con la de sRGB; si alguna componente difiere en más de 0.02, se considera de gama amplia. Los perfiles LUT sin matriz se juzgan por su cadena de descripción (P3, Adobe, 2020, ProPhoto). En el cICP de PNG y el nclx de AVIF, se considera amplia si el número de primarios no es 1 (BT.709 = sRGB) ni 2 (no especificado). En AVIF y HEIF se leen `pitm` (elemento principal) de la caja `meta` y, dentro de `iprp`, `ipma` (elemento ↔ número de propiedad) e `ipco` (propiedades), y solo se tiene en cuenta el `colr` asociado a la imagen principal. Aunque delante haya un `colr` de una miniatura o de una imagen auxiliar, se usa el de la imagen principal. Si no se puede leer esta estructura, se usa el primer `colr` que aparezca.

El perfil Display P3 que se adjunta al guardar lo genera la propia aplicación (ICC v4.3, 532 bytes). Los primarios son la matriz RGB→XYZ obtenida a partir de las coordenadas cromáticas de P3 (0.680, 0.320), (0.265, 0.690), (0.150, 0.060) y el blanco D65, adaptada a D50 con la transformación de Bradford.

```math
\mathbf r = (0.515119,\ 0.241189,\ -0.001050),\quad
\mathbf g = (0.291978,\ 0.692244,\ 0.041879),\quad
\mathbf b = (0.157103,\ 0.066567,\ 0.784071)
```

La curva es la misma curva paramétrica que sRGB (tipo 3: $`\gamma = 2.4`$, $`a = 1/1.055`$, $`b = 0.055/1.055`$, $`c = 1/12.92`$, $`d = 0.04045`$). Estos valores coinciden con los primarios del perfil Display P3 de Apple con una diferencia inferior a $`10^{-5}`$.

### 4.20 Métricas

```math
\mathrm{PSNR} = 10 \log_{10} \frac{255^2}{\frac{1}{3n}\sum (U - I_{\mathrm{gt}})^2},\qquad
E_{\mathrm{dir}} = \frac{\sum_p m_p\, \arccos\!\big(|\hat{\mathbf g}^{U}_p \cdot \hat{\mathbf g}^{\mathrm{gt}}_p|\big)}{\sum_p m_p}\quad (m_p \ge 80)
```

$`E_{\mathrm{dir}}`$ es el error de dirección de los bordes. En los bordes fuertes del original (magnitud de Sobel de 80 o más, un salto de unos 20 niveles), se promedia, ponderado por la intensidad del borde, el ángulo en que difieren las direcciones del gradiente del resultado y del original. Cuando aparecen escalones, la dirección del gradiente se inclina hacia los ejes y el error crece. SSIM sigue la definición de Wang et al. (2004).

### 4.21 Nitidez percibida

En el modo 「Percepción」 se suman dos bandas a la luminancia del resultado $`Y`$ ($`\sqrt{\ell}`$ en luz lineal, $`\ell`$ con valores gamma; $`\ell`$ es la luminancia de los canales recortados entre 0 y el tope). $`L`$ es el factor, $`G_\sigma`$ una gaussiana (aproximación de tres cajas) y $`a`$ la intensidad de percepción.

```math
d_1 = Y - G_{0.5L}Y,\qquad d_2 = G_{0.5L}Y - G_{2L}Y,\qquad
\tilde d = d\,\frac{d^2}{d^2+\tau^2}\quad(\tau = 0.004)
```

```math
\Delta = a\Big(k_1\,\tilde d_1 + k_2\,\frac{\tilde d_2}{1+|\tilde d_2|/c_2}\Big),\qquad
Y' = Y + \Delta\,\frac{h}{h+|\Delta|},\qquad
h = \begin{cases}\max_{\mathcal N}Y - Y & \Delta>0\\ Y-\min_{\mathcal N}Y & \Delta\le 0\end{cases}
```

$`d_1`$ es la pendiente de los contornos cerca de la frecuencia de Nyquist del original y $`d_2`$ el contraste local a la escala de 1–4 píxeles de origen. $`\tilde d`$ es un umbral de la misma forma que la máscara de enfoque: las amplitudes pequeñas, como el ruido y las huellas de JPEG, apenas se suben. El contraste local se sube menos cuanto mayor es su amplitud ($`c_2 = 0.05`$): el contraste ya marcado queda igual y el enterrado aflora. $`\mathcal N`$ es un entorno cuadrado de radio $`\mathrm{round}(0.75L)`$ (solo dentro de la imagen) y $`h`$ el margen que queda hasta el mínimo o el máximo del entorno. $`\Delta\,h/(h+|\Delta|)`$ solo se acerca suavemente a ese margen sin cruzarlo, así que no aparecen nuevos extremos (halos), y como los valores no se pegan al límite, las superficies no se apelmazan (no hay posterización). Una primera versión recortaba exactamente al rango del entorno; las superficies se apelmazaban y parecía una pintura al óleo, por lo que se cambió. La nueva luminancia se aplica como una razón $`s = Y'/Y`$ ($`s^2`$ en luz lineal) multiplicada por igual en los tres canales de color premultiplicados, de modo que se conservan el tono, el gris y el alfa; al aclarar, la razón se reduce para que el canal mayor no supere el tope (el mismo tratamiento que la restauración de textura). Con canal alfa, la luminancia se mide sobre el color sin premultiplicar, los desenfoques son medias ponderadas por la opacidad y los píxeles casi transparentes (alfa por debajo del 2 %) quedan fuera del mínimo/máximo del entorno, de modo que un borde donde solo cambia la opacidad, con el mismo color, no recibe ningún reborde.

$`k_1 = 0.5`$ y $`k_2 = 1.5`$ se eligieron con las imágenes de exploración. Se midieron las 16 combinaciones $`k_1 \in \{0.5, 1, 2, 3\}`$ × $`k_2 \in \{0, 0.5, 1, 1.5\}`$ en 22 imágenes × 3 condiciones (4× limpia, 4× degradada, 8× degradada; 66 casos), comparando la mediana del error logarítmico $`|\ln A| + |\ln C|`$ de la pendiente de contorno $`A`$ y el contraste local $`C`$ frente a la referencia. $`A`$ es la razón de pendientes medidas en los contornos de la referencia (10 % de mayor pendiente) tras desenfocar ambas con $`\sigma = 0.5L`$; $`C`$ es la razón de RMS de la banda $`G_{0.5L}-G_{2L}`$. Las 16 tuvieron un error menor que 「Precisión」 (0.418), entre 0.343 y 0.410; de ellas se eligió la que menos halo y ruido en zonas planas añade (error 0.357, halo +0.20 niveles, ruido en zonas planas ×1.14, SSIM +0.004). Los resultados están en `research/results/pop_grid.json`.

---

## 5. El mundo físico

| Fenómeno físico | Modelo en Upscaly |
|---|---|
| Un píxel del sensor suma toda la luz que cae sobre su área | Modelo de reducción = promedio de área en luz lineal $`D`$. El objetivo de la corrección de consistencia $`DU = I`$, la medición de la ampliación por etapas y la reducción de los datos de exploración siguen todos este modelo |
| Difusión de la óptica de lentes y escáneres | Función de dispersión de punto gaussiana (σ 0.45–0.95). Se estima a la inversa con la deconvolución RL |
| La intensidad del ruido del sensor varía con el brillo | Espectro de ruido con una ganancia por tramo de brillo $`g(I)`$ |
| JPEG redondea los coeficientes DCT a enteros | Para cada coeficiente, un intervalo de cuantización de anchura $`q_k`$. El original verdadero está en algún punto dentro de ese intervalo |
| El submuestreo de color (4:2:0) registra el color a media resolución | Filtro guiado que ajusta los límites de color a los contornos de la luminancia |
| El color de la iluminación tiñe incluso las zonas claras | Regla de consenso de tres estimadores |
| Los bordes semitransparentes mezclan la luz del primer plano y del fondo en proporción a su área | Alfa premultiplicado. Mismas operaciones para el color y el alfa |
| El ojo percibe el brillo de forma casi logarítmica | Guardado en sRGB; las decisiones de dirección y contorno se toman en un plano guía de luminancia gamma |
| El espectro de potencia de las imágenes naturales es aproximadamente $`1/f^2`$ (energía parecida en cada octava) | Estimación de las octavas que faltan en la restauración de textura: intensidad de la banda del original × $`\sqrt{\log_2 L}`$ |
| El grano de película son granos finos en posiciones aleatorias | La restauración de textura es un grano que solo ajusta la cantidad de textura. La posición y la forma de la textura no están en el original |
| El ojo es sensible a los escalones (bandas) en superficies amplias e insensible al ruido muy fino | Tramado de 8 bits: traslada el error de cuantización de baja frecuencia a ruido de alta frecuencia |
| Cada pantalla y cada cámara tiene un rango de colores reproducibles (gama) distinto | Se lee el perfil de color del original, se procesa con lienzo P3 y perfil P3, y se guarda así |

---

## 6. Arquitectura

### Archivos y entorno de ejecución

Toda la herramienta es un único archivo `upscaler.html` (unas 6,140 líneas, 360 KB) que contiene HTML, CSS y JavaScript. No hay bibliotecas externas y no hace falta servidor. Solo la tipografía de la interfaz (IBM Plex Sans KR) se descarga de Google Fonts; si no se puede descargar, se usa la del sistema. Se usa `CompressionStream` para el guardado PNG en flujo y Web Workers para el cálculo en paralelo; si falta alguno, se recurre a una vía más lenta. Si el dispositivo está en modo oscuro, se usa el tema oscuro.

### Componentes

| Capa | Funciones principales | Función |
|---|---|---|
| Espacio de color | `S2L`, `L2S`, `enc16Linear` | Tablas sRGB ↔ luz lineal, codificador de 16 bits |
| Motor | `ENGINE_FACTORY`, `PORT` | Cálculo puro compartido por el hilo principal y los workers |
| Interpolación | `dcciStageMulti`, `ediUpsample`, `resample2D`, `interpRegion` | DCCI, remuestreo con núcleos |
| Restricciones | `cellRange`, `clampRegion`, `consistencyCorrectPlanar` | Límite de celda, consistencia (Thomas) |
| Restauración | `estimateNoise`, `sdctDenoise`, `qcsLuma`, `guidedFilter`, `restoreTile`, `richardsonLucy`, `denoise`, `deblock` | Ruido, JPEG, desenfoque |
| Contornos | `edgeField`, `antialiasRegion`, `crispFields`, `steepenRegion`, `pixelTile` | Suavizado direccional, redibujado, pixel art |
| Teselas | `renderTile`, `tileJob`, `cropRectFor`, `outMarginFor` | Cálculo de una tesela, planificación de márgenes |
| Preparación del original | `prepareSource`, `dctRestorePool` | Correcciones a resolución original, etapas intermedias de la ampliación por etapas |
| Paralelismo | `initPool`, `poolRun`, `runLimited` | Grupo de workers, cola, límite de fragmentos simultáneos |
| Caché | `tcGet`, `tcPut`, `tcMax`, `srcGet` | LRU de teselas de resultado, 2 originales preparados |
| Automático | `analyzeImage`, `autoClassify`, `autoPick`, `AUTO_BIG`, `autoAA`, `autoStage`, `pickAutoScale`, `renderBudget` | Análisis, clasificación, ajustes, factor |
| Procesamiento del color | `estimateWB`, `colorAuto` | Balance de blancos |
| Tramado de 8 bits | `ditherTile`, `srgbF` | Tramado IGN determinado por las coordenadas |
| Textura | `texFields`, `texNoise`, `addTexture`, `texMargin` | Intensidad de textura del original, ruido por bandas, añadir solo la parte que falta |
| Gama de color | `P3_OK`, `sniffWideGamut`, `iccIsWide`, `displayP3Icc`, `p3IccpChunk` | Detección de gama amplia, lienzo P3, perfil P3 |
| Guardado | `createPngEncoder`, `exportPng`, `exportViaCanvas`, `saveAll` | PNG por filas de teselas, guardado por lotes |
| Pantalla | `render`, `drawTiles`, `detectSubjects` | Vista comparativa, recuadro del sujeto |

### Flujo de datos y caché

Para cada imagen se guardan por separado el lienzo del original, la cuadrícula y la calidad JPEG, el modelo de ruido (una vez, la primera vez que hace falta), el recuadro del sujeto y los ajustes. El resultado lleva una instantánea de los ajustes con que se creó, de modo que aunque después se cambien los ajustes, el guardado y el recálculo dan los mismos valores que ese resultado.

Las teselas de resultado se guardan en una caché LRU. El límite de la caché es el 60 % del presupuesto de memoria, y si la creación de un lienzo falla de verdad, en ese momento se aprende el 80 % del tamaño de la caché como límite de este dispositivo. Con el factor elegido por el factor automático, el límite se reduce aún más según la condición de garantía. Si el resultado no cabe entero en la caché, primero se calcula solo la mitad de la caché empezando por el centro, y el resto se calcula cuando aparece en pantalla o al guardar. Las teselas visibles que aún no existen se rellenan con el original ampliado y su cálculo se encarga en segundo plano.

### Determinismo: pantalla = resultado guardado

La pantalla, el guardado y el recálculo usan todos el mismo `renderTile`. Los procesos que miran a los vecinos (interpolación, consistencia, máscara de enfoque, suavizado direccional, redibujado) calculan un margen adicional del radio necesario y luego lo recortan, y la cuadrícula de bloques DCT y el campo de direcciones se fijan a coordenadas globales. Por eso el resultado es el mismo con independencia del tamaño y el orden de las teselas y del número de workers. Aunque se calcule troceando en teselas de 64 px, las costuras entre teselas quedan dentro de 1 nivel como máximo, y la diferencia entre dos cálculos con los mismos ajustes es 0 ([9.1](#91-invariantes)).

---

## 7. Problemas y soluciones

### Calidad de imagen

| Problema | Causa | Solución |
|---|---|---|
| Borde blanco alrededor de los contornos (ringing y halos) | Las colas negativas del núcleo crean extremos que no están en el original | Recorte a los 4 vecinos en DCCI, límite de celda, limitador de rodilla en la máscara de enfoque |
| Al reducir el resultado no coincide con el original | Aunque la interpolación pase por las muestras del original, el promedio de área no cuadra | Resolver exactamente la corrección de consistencia a resolución original (Thomas) |
| La consistencia y el límite de celda se rompen mutuamente | Al ajustar una, la otra se desvía | Dos proyecciones alternas |
| Color corrido en bordes semitransparentes (error de composición de hasta 29 niveles) | El alfa y el color pasaban por operaciones distintas | Hacer que el alfa pase por las mismas operaciones que el color (0.1–4 niveles) |
| La ampliación por etapas empeoraba el resultado (−0.4 a −0.8 dB) | La referencia del límite de celda y de la consistencia era la imagen intermedia | Tomar como referencia el original (+0.03 a 0.06 dB) |
| En dibujo lineal reaparecen los escalones del original | La consistencia toma como correctos los escalones del original | Redibujado y suavizado direccional para dibujo lineal y logotipos; la exploración decide por tipo si se usa la consistencia |
| Las líneas finas se ondulan | El redibujado sobre la isolínea del 50 % no conserva la densidad de las líneas más finas que 1 píxel | Medir la anchura de la línea por densidad integrada y dejar las líneas finas como están |
| Errores de dirección de DCCI con ruido | El ruido altera la variación por dirección | Sumar a la variación un suelo de ruido $`n_{\mathrm{off}}`$ (σ de Immerkær) |
| Se borra textura tomándola por ruido | Los bloques con textura inflan el espectro de ruido | Límite superior con el suelo de ruido PCA, selección solo por alta frecuencia |
| La restauración JPEG borra incluso los detalles del archivo | La eliminación de ruido por sí sola no conoce los coeficientes originales | Proyección sobre el intervalo de cuantización (QCS) |

### Color

| Problema | Causa | Solución |
|---|---|---|
| Los preajustes cambiaban también el color | Antes los preajustes incluían balance de blancos | Separar el procesamiento del color como función aparte, por defecto 0, solo con el botón 「Color automático」 |
| Un gato naranja o una puesta de sol se tomaban por dominante de color | Solo se usaba el estimador de la media | Consenso de tres estimadores (en 54 pares, sesgo de color fuerte 13.9° → 4.0°) |
| Los límites oscuros se empastan | Interpolación en espacio gamma | Calcular en luz lineal |
| Las fotos de móvil salen apagadas tras ampliar | El lienzo sRGB comprimía los colores Display P3 dentro de sRGB (el rojo P3 quedaba en 234,51,35 en valores P3) | Leer el perfil del original, procesar con lienzo P3 y guardar con perfil P3 (cICP, iCCP) (v1.1.0) |
| Bandas escalonadas en cielos muy ampliados | Las bandas del redondeo a 8 bits se ensanchan tanto como el factor | Tramado determinado por las coordenadas (v1.1.0) |
| Al añadir textura aparecen puntos blancos en esquinas y zonas oscuras | Las estructuras fuertes y estrechas tienen mucha intensidad de banda, y en las zonas oscuras el mismo cambio es relativamente mayor | Erosión 5×5 de la intensidad de textura, cambio por píxel limitado a ±40 % del brillo (√) |
| Al desactivar 「Calcular en luz lineal」 la textura se desbordaba 1.8 veces | Se trataban los valores gamma como luz lineal y se aplicaban una raíz y un cuadrado de más | En modo gamma, el eje de brillo es el valor gamma tal cual y la relación tampoco se eleva al cuadrado. R en 12 fotos: 2.24 antes de corregir, 1.32 después (1.22 en modo luz lineal, v1.1.1) |
| La textura cambiaba el tono de colores claros y saturados | Aunque se multipliquen los tres canales por la misma relación, solo el canal mayor se recorta en su límite | Limitar la relación de aclarado al margen del canal mayor (alfa, 1). Rojo claro a 4×: cambio máximo de la proporción de color lineal 14.0 % → 2.9 % (nivel de cuantización de 8 bits, v1.1.1) |
| La intensidad de textura se desborda con entradas limpias y se queda corta con entradas degradadas | La banda más fina es la primera que recortan el desenfoque y JPEG | Medir en la banda una octava por debajo, restar la alta frecuencia que ya tiene el resultado y añadir solo la parte que falta |

### Estabilidad y entrada/salida

| Problema | Causa | Solución |
|---|---|---|
| La aplicación se detiene nada más arrancar | `updateSizeNote`, llamada al arrancar, leía un global `let` declarado tarde: error TDZ | Declarar esos globales con `var` o ponerlos al principio |
| Una medición que termina tarde se aplica a una imagen nueva | Carrera entre mediciones asíncronas | Número de petición automática `AUTO_SEQ` y comprobación `alive()` |
| Un lienzo grande queda en blanco sin lanzar excepción | Límite de superficie oculto del navegador | Si el lienzo del original es totalmente transparente, se lee el original directamente en fragmentos de 1024² para ver si hay píxeles visibles; el ensamblado del guardado busca un píxel visible en el resultado, comprueba esa posición en el lienzo y aprende el límite. Detecta incluso imágenes en las que solo se ve un punto (v1.1.1; antes solo se miraba el píxel de abajo a la derecha, y si ese píxel era transparente de por sí, no se detectaba) |
| La gama amplia de AVIF se detectaba como sRGB | Se usaba la primera caja colr del archivo, que podía ser la de una miniatura | Con `pitm`, `ipma` e `ipco` solo se mira el colr asociado a la imagen principal (v1.1.1) |
| La gama amplia de PNG se detectaba como sRGB | Si un chunk sRGB o iCCP iba antes que cICP, se paraba ahí | Se miran todos los chunks de color antes de IDAT y se aplica cICP > iCCP > sRGB. Si los primarios de cICP son no especificados (2) o un número reservado, se pasa a iCCP (v1.1.1) |
| 「Ampliar por etapas」, cambiado justo después de abrir la imagen, volvía a su valor anterior | La medición que fija el valor inicial de la casilla terminaba tarde y lo sobrescribía | Cada vez que el usuario o 「Automático」 lo fijan se incrementa un número (`STAGE_SEQ`), y si difiere del que había al empezar la medición o han cambiado las condiciones, no se aplica (v1.1.1) |
| Color corrido en los bordes y costuras entre teselas en imágenes transparentes ampliadas por etapas | En las etapas intermedias solo el alfa pasaba por un remuestreo normal | El alfa pasa por las mismas operaciones que el color (color de borde en DCCI 4×, máximo de 19–38 → 1–4 niveles; costura del logotipo 206 → 1; v1.1.1) |
| La decodificación de archivos grandes no termina | Casos en que no llega ni `onload` ni `onerror` | Se prueban por turnos tres vías con tiempos límite proporcionales al tamaño |
| Entornos en los que los workers están bloqueados | Políticas de seguridad | Tras comprobar la respuesta durante 5 segundos, se pasa al hilo principal |
| Se agota la memoria con resultados grandes | Lienzo con el resultado completo | Caché de teselas y PNG en flujo |
| Si se cambia el factor después del modo automático, los ajustes no cuadran | El modo automático decide en función de ese factor | Se avisa con `AUTO_LAST` y se pide volver a pulsar |
| La validación con entradas limpias se procesaba como si fueran JPEG | El script de validación pasaba calidad JPEG 85 incluso a las entradas limpias | Nueva validación sin información JPEG; los valores explorados de logotipo y cómic solo se usan con entradas JPEG (v1.1.0) |

---

## 8. Optimización

Las optimizaciones de velocidad solo se incorporaron cuando la salida es idéntica byte a byte. Se contrastan la salida de 16 bits y las estadísticas de 134 combinaciones de ajustes con la versión de referencia (`tests/engine_regression.js`). La excepción es la primera fila de la tabla de abajo, la tabla de cuadrícula interna de DCCI: es un compromiso adoptado tras medir que gana velocidad a cambio de alterar ligeramente la salida (±0.007 dB), y la versión de referencia actual es posterior a la introducción de esta tabla.

| Optimización | Contenido | Efecto |
|---|---|---|
| Tabla de cuadrícula interna de DCCI | En 6, 8 y 12×, cuadrícula de 4× en lugar de 8×. Dentro de ±0.007 dB en 25 preajustes × 1,800 pares | 25–30 % más rápido |
| Pesos direccionales de DCCI compartidos | Se calculan una vez en el plano guía y se usan para todos los canales | |
| Caché del límite de celda | Mínimos y máximos por celda una sola vez por plano de canal | |
| Canales agrupados en la consistencia | Se comparten huellas, coeficientes y pesos de interpolación sin copiar los planos de canal para entrelazarlos | |
| Total | 1024×768 → 4× 「Estándar」(표준) | 13.9 s → 12.0 s, 134 combinaciones idénticas byte a byte |

Además, las gaussianas se aproximan con tres desenfoques de caja (método de Kutskir), los pesos de brillo del filtro bilateral se toman de una tabla y las bandas de 16 bits se recalculan en 128 filas para ahorrar memoria. También se probó la cuadrícula de 4× por debajo de 2×, pero aunque la media subió +0.013 dB, hubo más casos que empeoraron (24 %) que casos que mejoraron (21 %) y el tiempo se duplicaba, así que no se adoptó.

Los cuellos de botella que quedan son la interpolación DCCI (por canal), la corrección de consistencia y el remuestreo. Es difícil reducirlos mucho sin cambiar la salida.

---

## 9. Verificación

### 9.1 Invariantes

`tests/verify_suite.js` carga el script de la aplicación tal cual en un DOM simulado y recorre de principio a fin la vía de procesamiento real.

| Elemento | Esperado | v1.1.1 |
|---|---|---|
| Costuras entre teselas (29 preajustes, teselas de 64 px) | Máximo 1 como mucho | Todas con máximo 1 como mucho |
| Costuras entre teselas en ampliación por etapas (lanczos3 4×, edi 2×, pixelart 4×) | 0 | 0 · 0 · 0 |
| Determinismo (5 preajustes calculados dos veces) | 0 | 0 |
| Identidad a 1× sin procesar | 0 | 0 |
| Exactitud bit a bit del vecino más próximo a 3× | 0 valores discrepantes | 0 |
| Conservación del blanco y negro (diferencia máxima entre R, G y B en 29 preajustes) | 0 | 0 |
| Color de bordes con alfa (alfa del 5 % o más) | Máximo 4 | 4 |
| Color de bordes con alfa y costuras entre teselas en ampliación por etapas a 4× (foto y logotipo × lanczos3 y edi) | Máximo 4 · 1 como mucho | 4 · 0, 0, 1, 1 (en v1.1.0, color 22–38 y costura del logotipo 206) |
| Ida y vuelta de la tabla de luz lineal (256 niveles) | 0 discrepancias | 0 |

Los invariantes se midieron con el tramado de 8 bits activado (por defecto). Salvo el elemento de alfa en la ampliación por etapas, son iguales que en v1.0.0. Estas son las demás pruebas.

| Prueba | Contenido | v1.2.1 |
|---|---|---|
| `e2e_presets.js` | Procesamiento con 8 preajustes + guardado en 8 y 16 bits | ✓ 8, 45,711 · 239,731 bytes (en 8 bits, +5 % respecto a los 43,446 de v1.0.0 por el tramado) |
| `engine_regression.js` | Salida de 16 bits y estadísticas de 134 combinaciones contrastadas con la versión de referencia | 134/134 idénticas byte a byte |
| `worker_test.js` | Tesela del worker = cálculo en el hilo principal | Idénticas byte a byte |
| `alpha_probe.js` | Error de color en bordes semitransparentes por preajuste | |
| `e2e_autoscale.js` | Factor automático: con presupuesto de 600 MB, 512² y 800×600 → 8×, 4000×3000 → 2×. Con vecino más próximo y DCCI en tres etapas (textura activada), la garantía se mantiene tras pulsar 「Automático」, y los factores mayores no quedan garantizados con los ajustes de 「Automático」 de ese factor (6 casos). 4000×3000 con la clasificación fijada en la UI, 「Automático」 desde 1.25× → 2× | 4000×3000 daba 4× en v1.1.0. A 4×, el resultado tiene 192 MP y solo el PNG comprimido acumula hasta unos 300 MB, así que no queda garantizado. En el caso de la UI, las versiones anteriores a v1.1.1 se quedaban en 1.25× con 「Sin garantía」(보장 없음) |
| `e2e_auto_stale.js` | Aviso de cambio de factor después del modo automático | |
| `e2e_stage_force.js` | Ampliación por etapas forzada e indicador, conservación del cambio del usuario durante la primera medición | ✓ 2 (en v1.1.0 se sobrescribían ambos) |
| `blank_canvas.js` | Detección de lienzo en blanco: 6 casos de lectura del original, 4 de guardado desde lienzo | ✓ 10 (v1.1.0 fallaba 4 casos: esquina inferior derecha transparente y solo un punto visible) |
| `gamut_sniff.js` | Detección de gama amplia: 5 casos de colr de la imagen principal en AVIF, 4 de orden de los chunks de color en PNG, 5 de metadatos o cajas de más de 2 MB antes de la información de color (XMP extendido en JPEG, chunks de texto en PNG, mdat de AVIF antes de meta), 4 de número de primarios en cICP (no especificado, reservado, sRGB) | ✓ 18 (v1.1.0 fallaba 10) |
| `dither_probe.js` | Indicador de bandas del tramado de 8 bits | [9.7](#97-tramado-de-8-bits) |
| `texture_seam.js` | Con textura activada: costuras, determinismo, blanco y negro, tono de colores claros | Costura máxima 1 (igual que con textura desactivada), 0 entre dos cálculos, 0 en blanco y negro, cambio de proporción de color en rojo claro 2.9 % (14.0 % en v1.1.0) |
| `pop_seam.js` | 「Percepción」: uniones de teselas · determinismo · escala de grises · sin halo (dentro del rango del entorno del resultado Precisión) · modo gamma · tono de colores brillantes · bordes donde solo cambia la opacidad | Unión como máximo 1 (igual que Precisión), 0 entre dos cálculos, escala de grises 0, como máximo 0.9 niveles por encima del rango del entorno (redondeo de 8 bits), cambio de razón de color del rojo brillante 2.8 %, diferencia de color en bordes de opacidad como máximo 1 (24 antes de la corrección, 18 en un degradado blanco) |
| `p3_browser.js` | Conservación del color P3 en Chromium real (primarios y colores mezclados, incluida la simulación de entornos en los que se bloquean las imágenes blob: y `createImageBitmap` convierte a sRGB) | [9.9](#99-display-p3) |

### 9.2 Datos

En `data/ds` hay 567 originales divididos en 9 clases. Incluyen también variantes creadas a partir del mismo original con distinta calidad JPEG, reducción, desenfoque, ruido, simulación de escaneo, etc.

| Clase | Fotografía | Ilustración | Pintura | Cómic | Dibujo lineal | Documento | UI | Logotipo | Píxel | Total |
|---|---|---|---|---|---|---|---|---|---|---|
| Imágenes | 135 | 152 | 36 | 45 | 40 | 55 | 38 | 43 | 23 | 567 |

`data/big` es para la exploración de la restauración a gran escala (`research/search/prep_big.py`). Para cada clase se separan 3 imágenes de exploración y 3 de validación, se toma como respuesta correcta el fragmento de 256 px con más detalle de cada original y se crean cuatro entradas.

| Condición | Entrada | Respuesta correcta |
|---|---|---|
| Degradación realista 2, 4 y 8× | Reducción de área en luz lineal → desenfoque σ0.6 → JPEG 85 (4:2:0) | Fragmento de 256 px |
| Entrada limpia 4× | Solo reducción de área en luz lineal | Fragmento de 256 px |
| Imagen grande 4× (solo validación) | Degradación realista de los 384–512 px centrales | Ese fragmento |

### 9.3 Preajustes gráficos y automático para gráficos

Los 11 preajustes celuloide, coloreado digital, póster, cómic, dibujo lineal, documento, manuscrito, plano, UI, texto de pantalla y logotipo activan la consistencia; en celuloide el suavizado direccional baja al 50 %, y en coloreado digital y póster al 75 %. Son ajustes con los que, en 29 imágenes de ajuste y 30 de validación degradadas con desenfoque + JPEG, PSNR y SSIM suben 0.2–0.6 dB y los escalones de los bordes a 45° y similares (error de dirección) se mantienen dentro del 10 % respecto al original. Cuando el modo automático para gráficos pasó a elegir el suavizado direccional midiéndolo en cada imagen, en 59 gráficos la media fue de +0.44 dB y el error de dirección de los bordes de −8.4 %.

### 9.4 Exploración de la restauración a gran escala

**Espacio.** Núcleo 4 × consistencia 2 × límite de celda 2 × cantidad de enfoque 19 × (radio, umbral, sobreoscilación, suavizado direccional, redibujado, multiplicador de deconvolución, multiplicador de ruido, multiplicador de restauración JPEG, ampliación por etapas) 10 cada uno × multiplicador de σ de deconvolución 5 = **1,520,000,000,000 configuraciones**.

**Restricciones.** Respecto al modo automático actual, caída de SSIM de 0.003 como máximo y error de dirección de los bordes de +10 % como máximo.

**Procedimiento.** 1,500 al azar → reevaluación de las 150 mejores → descenso por coordenadas desde 4 puntos de partida (todas las etapas, hasta converger) → los valores pegados al extremo de un control deslizante se vuelven a explorar ampliando el rango. En total, unas 55 000 evaluaciones. El registro (`data/big/search.jsonl`) se va acumulando, así que la exploración continúa donde se dejó.

**Validación.** Diferencia de PSNR (dB) respecto al modo automático anterior en las 3 imágenes de validación apartadas. Solo se adopta en las clases y factores en los que la media sube y mejoran al menos 2 de las 3 imágenes. Toda la tabla se volvió a medir en v1.1.0 (`research/results/val_v3.json`). La validación anterior difería del uso real en dos aspectos. Primero, pasaba calidad JPEG 85 incluso a las entradas limpias, de modo que el modo automático anterior, que es la referencia de comparación, aplicaba restauración JPEG. Segundo, a las entradas JPEG solo les pasaba el número de calidad, sin la tabla de cuantización ni la cuadrícula, de modo que faltaba la restauración del intervalo de cuantización que se usa con archivos JPEG reales. Corregidos ambos, el ámbito de adopción no cambió, salvo la condición JPEG de logotipo y cómic.

| Clase | 2× | 4× | 8× | 4× entrada limpia | 4× imagen grande | Adopción |
|---|---|---|---|---|---|---|
| Fotografía | +1.73 | +0.36 | +0.19 | +0.94 | +0.89 | Desde 2× |
| Dibujo lineal | +1.22 | +1.08 | +0.32 | +0.42 | +1.67 | Desde 2× |
| UI | +0.84 | +0.36 | +0.15 | +0.37 | +0.22 | Desde 2× |
| Cómic | +2.27 | +0.62 | +0.07 | −0.15 | +0.44 | Desde 2×, solo entradas JPEG |
| Pintura | +2.37 | +1.46 | +0.25 | +1.50 | +1.51 | Desde 2× |
| Documento | −0.48 | +0.55 | +0.22 | +0.58 | +0.55 | Solo desde 4× |
| Ilustración | +0.28 | +0.26 | +0.04 | −0.12 | +0.16 | No se usa |
| Logotipo | +2.09 | +3.04 | +1.57 | +0.18 (1/3 imágenes) | +1.36 | Desde 2×, solo entradas JPEG |

En logotipo, al volver a medir las entradas limpias solo mejora 1 de las 3 imágenes y el error de dirección de los bordes, +14.8 %, supera la restricción (+10 %). Con entradas que han pasado por JPEG (degradación realista 2, 4 y 8×, imagen grande 4×) lo supera todo, así que solo se usa con archivos JPEG. En cómic, la media de las entradas limpias medidas de nuevo bajó a −0.15 dB, así que se le aplicó la misma condición. Las cifras de entrada limpia de fotografía y pintura bajaron, pero siguen superando la validación.

### 9.5 Clasificador

Se entrenó con las 567 imágenes usando las 27 características de `research/classifier` (`features2.py`). Las variantes procedentes del mismo original se agrupan y la validación cruzada se hace por grupos, de modo que distintas versiones de la misma imagen no entran a la vez en entrenamiento y prueba. La exactitud de la validación cruzada por grupos es del **92.9 %**. Los pesos están en `AUTO_MODEL` de `upscaler.html` como JSON en línea, y la decisión en JS coincide con la de Python en 567/567.

### 9.6 Procesamiento del color

Con la regla de consenso de tres estimadores, en 54 pares de prueba el sesgo de color fuerte (ángulo respecto al neutro) bajó de 13.9° a 4.0°.

### 9.7 Tramado de 8 bits

Se mide el error de la salida de 8 bits tomando como referencia la salida de 16 bits del mismo procesamiento (`tests/dither_probe.js`). El indicador de bandas es el RMS del error tras desenfocarlo con una gaussiana σ2 (en píxeles del resultado). Las bandas son un error de baja frecuencia que abarca superficies amplias, así que persisten al desenfocar, mientras que la textura fina del tramado desaparece al desenfocar. Son 20 casos: un degradado de cielo sintético y 9 fragmentos de foto, pintura e ilustración de validación de la restauración a gran escala, ampliados 4 y 8 veces.

| Indicador (media de 20 casos, niveles) | Redondeo | Tramado | Cambio |
|---|---|---|---|
| Indicador de bandas σ2 | 0.059 | 0.033 | −44 % (unos −50 % a 8×) |
| Indicador de bandas σ4 | 0.040 | 0.017 | −58 % |
| RMS total | 0.245 | 0.340 | +39 % |

El RMS total aumenta en la medida en que el tramado añade ruido fino. Con el tramado activado, la salida del vecino más próximo a 3× sigue teniendo 0 valores discrepantes y la diferencia máxima entre R, G y B de una entrada en blanco y negro es 0.

### 9.8 Restauración de textura

Con los datos de restauración a gran escala (respuesta correcta de 256 px → reducción 4 y 8 veces → nueva ampliación) se mide la energía de la banda por encima de la frecuencia de Nyquist del original (diferencia de gaussianas con σ = 0.375·factor) (`research/texture/tex_probe.js`, resultados en `research/results/texture*.json`).

- **Relación de energía de alta frecuencia R** = resultado / respuesta correcta. La ampliación no puede crear esta banda, así que R es muy inferior a 1.
- **Error del mapa de textura** = RMS del logaritmo de la relación de energías de alta frecuencia por bloques de 16×16. Indica si la textura está donde debe estar.
- **PSNR y SSIM.** Una textura inventada no coincide en posición con la respuesta correcta, así que lo normal es que bajen.

**Ajuste de la constante.** En 9 casos, 3 imágenes de foto de exploración × (entrada limpia 4×, degradación realista 4× y 8×), se buscó con el método de la secante la constante que da R = 1, y la mediana de los 8 casos en los que se pudo ajustar R, $`C = 6.256`$, se fijó como 100 % (en v1.1.0 era 5.915; aumentó un poco al pasar a reducir el aclarado según el margen de los canales). Los ajustes son el preajuste y los valores explorados elegidos clasificando la entrada reducida, igual que 「Automático」. Varía mucho de una imagen a otra, entre 2.4 y 12, porque la cantidad de textura difiere entre fotos y una sola banda del original no basta para estimar con exactitud la energía que falta. Comprobado con las imágenes de respuesta correcta, la segunda banda del original × número de octavas fue, de las cuatro fórmulas de predicción probadas, la más exacta en fotos (energía ×÷1.45).

**Validación (3 imágenes de foto de validación × 4 condiciones = 12 casos).**

| Intensidad | R | Error del mapa de textura | Casos con mejora en ambos | PSNR | SSIM |
|---|---|---|---|---|---|
| Desactivada | 0.11 | 1.04 | | | |
| 25 % | 0.27 | 0.70 | 12/12 | −0.39 dB | −0.05 |
| 100 % | 1.22 | 0.57 | 12/12 | −2.24 dB | −0.18 |

**Otras clases.** Valores medidos con la misma constante (`research/results/texture.log`, `texture_25.log`). Los ajustes son los mismos que en 「Automático」: se clasifica la entrada reducida para elegir el preajuste, y los valores explorados de restauración a gran escala se leen tal cual de `AUTO_BIG` de la aplicación y solo se usan en sus condiciones de clase, factor y JPEG (la medición de v1.1.0 elegía el preajuste con la imagen de respuesta correcta y la clasificación de los datos, y con entradas JPEG usaba los valores explorados también en ilustración, así que difería de la aplicación real).

| Clase (casos) | Intensidad | R | Error del mapa de textura | Casos con mejora en ambos | PSNR |
|---|---|---|---|---|---|
| Pintura (12) | 25 % | 0.18 → 0.78 | 0.88 → 0.41 | 12/12 | −0.63 dB |
| Pintura (12) | 100 % | 0.18 → 8.51 | 0.88 → 0.94 | 7/12 | −4.50 dB |
| Ilustración (12) | 25 % | 0.07 → 0.11 | 1.12 → 1.01 | 10/12 | −0.12 dB |
| Ilustración (12) | 100 % | 0.07 → 0.41 | 1.12 → 0.99 | 6/12 | −1.02 dB |
| Gráficos planos (24) | 25 % | 0.27 → 0.28 | 0.67 → 0.65 | 14/24 | −0.04 dB |
| Gráficos planos (24) | 100 % | 0.27 → 0.63 | 0.67 → 0.56 | 12/24 | −0.94 dB |

En pintura todo mejora al 25 %, pero al 100 % R se desborda hasta 8.5 veces y el error del mapa de textura incluso empeora. Las pinturas a pincel tienen mucha energía en las bandas intermedias, y una constante ajustada para fotos se excede. En ilustración mejoran 10/12 al 25 %, pero R apenas sube (0.07 → 0.11), así que el efecto es pequeño. En gráficos planos (logotipos, UI, dibujo lineal, documentos) solo mejora algo más de la mitad. No se recomienda en ninguno de los dos casos.

**Resultado a la vista.** Las cifras están más cerca de la respuesta correcta al 100 %, pero en las imágenes comparativas (`research/results/texture_samples/`, de izquierda a derecha: respuesta correcta, textura desactivada, textura activada) el 100 % parece ruido en la mayoría de los casos. En escenas con textura densa (un motor de motocicleta, una torre de alta tensión) se ven puntos finos incluso al 25 %. En superficies lisas como rostros o piel, entre el 20 y el 30 % añade una textura sutil. Por eso el texto de ayuda recomienda 20–30 % para fotos y pintura.

### 9.9 Display P3

Se mide en Chromium real (Playwright) (`tests/p3_browser.js`). Se dibujan parches de primarios P3 en un lienzo display-p3 de Chromium, se convierten a JPEG y PNG (el codificador del navegador añade el perfil P3), se cargan en la aplicación, se amplían 2× → se guardan en PNG → se vuelve a leer el archivo guardado en un lienzo P3 y se mira el color del centro del parche.

| Parche (valores P3) | Resultado guardado en v1.0.0 | Resultado guardado en v1.1.0 |
|---|---|---|
| Rojo 255, 0, 0 | 234, 51, 35 | 255, 0, 0 (entrada JPEG 254, 0, 0) |
| Verde 0, 255, 0 | 117, 251, 76 | 0, 255, 0 (entrada JPEG 0, 255, 1) |
| Gris 128, 128, 128 | 127, 127, 127 | 127, 127, 127 |

El gris 127 es el valor de la propia imagen de prueba. El perfil ICC Display P3 generado por la aplicación también se comprobó con LittleCMS (Pillow ImageCms): los valores de P3 (200, 100, 50) y (60, 180, 220) convertidos a sRGB coinciden con los calculados con la matriz de primarios. Los JPEG y PNG con perfil sRGB y los archivos del conjunto de datos sin perfil se detectaron como sRGB, y las imágenes de prueba P3 y los archivos guardados por v1.1.0 como de gama amplia.

### 9.10 Nitidez percibida

Las 23 imágenes de validación (sin pixel art) × 3 condiciones (4× limpia, 4× degradada, 8× degradada) = 70 casos se amplían con los mismos ajustes que 「Automático」, comparando 「Precisión」 y 「Percepción」 (`research/pop/pop_probe.js`, `research/results/pop_val.json`). El error es el $`|\ln A| + |\ln C|`$ de [4.21](#421-nitidez-percibida), que cuenta por igual quedarse más romo que la referencia y pasarse de ella. El halo es la cantidad media en que el resultado supera el rango del entorno de la referencia (radio = factor) alrededor de sus contornos (a menos de 2× el factor); el ruido en zonas planas es el RMS de alta frecuencia del resultado donde la referencia es plana (30 % de menor pendiente).

| Percepción 100 % | Error (Precisión → Percepción) | Mejorados | Pendiente de contorno A | Contraste local C | Halo | Ruido en zonas planas | PSNR | SSIM |
|---|---|---|---|---|---|---|---|---|
| Fotos, 9 casos | 0.303 → 0.226 | 6/9 | 0.849 → 0.949 | 0.967 → 1.088 | +0.16 niveles | ×1.09 | −0.22 dB | −0.0003 |
| Pintura · ilustración · manga, 26 casos | 0.676 → 0.529 | 20/26 | 0.731 → 0.825 | 0.865 → 0.983 | +0.12 niveles | ×1.16 | −0.16 dB | +0.006 |
| Dibujo lineal · documentos · UI · logotipos, 35 casos | 0.621 → 0.559 | 20/35 | 0.865 → 0.949 | 0.982 → 1.078 | +0.52 niveles | ×0.95 | −0.06 dB | +0.012 |
| Total, 70 casos | 0.600 → 0.505 | 46/70 | 0.813 → 0.903 | 0.937 → 1.044 | +0.33 niveles | ×1.05 | −0.12 dB | +0.008 |

| Intensidad (total, 70 casos) | Error | Mejorados | Halo | Ruido en zonas planas | PSNR | SSIM |
|---|---|---|---|---|---|---|
| 50 % | 0.526 | 49/70 | +0.22 niveles | ×1.00 | −0.06 dB | +0.006 |
| 100 % | 0.505 | 46/70 | +0.33 niveles | ×1.05 | −0.12 dB | +0.008 |
| 150 % | 0.497 | 46/70 | +0.40 niveles | ×1.09 | −0.17 dB | +0.008 |

La pendiente de los contornos y el contraste local suben hacia la referencia, mientras que el halo y el ruido en zonas planas apenas aumentan. Las fotos y las imágenes mejoran en dos tercios de los casos o más, pero el dibujo lineal, los documentos, la UI y los logotipos en 20/35, poco más de la mitad: sus contornos ya son pronunciados y el contraste local se pasa con facilidad de la referencia. Las imágenes comparativas están en `research/results/pop_samples/` (de izquierda a derecha: referencia · Precisión · Percepción 50 % · 100 % · 150 %; 4× degradada, y una referencia de 500 px reducida a la mitad y ampliada 2×).

Al principio, 「Percepción」 añadía también un grano fino (restauración de textura, 25 % con intensidad 100 %). La medida de textura mejoraba (razón de energía de alta frecuencia R 0.13 → 0.30), pero el ruido en zonas planas se multiplicaba por 4.0 y el SSIM bajaba 0.044 (`pop_val_grain.json`), con puntos visibles en las superficies lisas. Por eso 「Percepción」 sube solo los contornos y el contraste local, y el grano fino se elige aparte con el control 「Restauración de textura」.

---

## 10. Puntos fuertes

**Restauración honesta con la información.** En la etapa de restauración no se crean extremos nuevos (límite de celda) y se ajusta el resultado para que, al reducirlo de nuevo, se obtenga el original (consistencia). Las correcciones que van encima solo se añaden en la medida en que el usuario las activa.

**Un modo automático que conoce el tipo de imagen.** Decide con una clasificación en 9 clases (92.9 %), una exploración de 1.52 billones de configuraciones y mediciones hechas sobre cada imagen. En pantalla indica en qué se ha basado.

**Restauración completada a la resolución original.** Antes de ampliar se realizan la restauración DCT, que mide el espectro de ruido para eliminarlo, la restauración JPEG, que respeta la tabla de cuantización del archivo, y la deconvolución RL.

**Gráficos con líneas vivas.** El suavizado direccional y el redibujado por distancia con signo producen líneas nítidas sin escalones. En el pixel art, cada submuestra solo elige colores de la paleta original, y los colores nuevos solo aparecen en el antialiasing de los contornos.

**Ajustes del usuario honestos.** Las casillas y los controles deslizantes funcionan exactamente como se eligen (la excepción justo después de abrir una imagen se describe en [11. Limitaciones](#11-limitaciones)). Solo el botón 「Automático」 automatiza, y el indicador se limita a informar del resultado de la medición.

**Sin ataduras de tamaño.** Gracias a la caché de teselas y a la compresión PNG por filas de teselas, se guardan incluso resultados de cientos de millones de píxeles sin cargar en memoria el resultado completo sin comprimir. El factor automático solo elige factores que se pueden procesar hasta el final según el presupuesto de memoria.

**Pantalla = guardado.** Con el mismo núcleo, los mismos márgenes y la misma cuadrícula global, lo que se ve en pantalla es exactamente lo que se guarda.

**8 bits sin bandas.** Con un tramado determinado por las coordenadas, el indicador de bandas se reduce casi a la mitad, sin alterar los colores exactos, los grises ni las costuras entre teselas.

**Los colores del móvil, tal cual.** Se lee el perfil del original, las fotos Display P3 se procesan en P3 y se guardan con un perfil P3 generado por la propia aplicación y cICP.

**Privacidad y portabilidad.** No hay servidor ni instalación, y la imagen no sale del dispositivo. Al ser un único archivo, es fácil de conservar y trasladar.

---

## 11. Limitaciones

**No inventa.** No puede recuperar detalles que no quedaron registrados en el original (un solo pelo, letras pequeñas). No dibuja formas verosímiles como la ampliación con redes neuronales, así que un original muy borroso sigue borroso aunque se amplíe mucho.

**La restauración de textura es grano.** Ajusta la cantidad de textura, pero la posición y la forma son ruido inventado, así que al subir la intensidad parece ruido. Al 100 %, en pintura se desborda mucho. En ilustración el efecto es pequeño, y en gráficos planos solo mejora aproximadamente la mitad con cualquier intensidad. Solo se recomienda a baja intensidad en fotos y pintura ([9.8](#98-restauración-de-textura)).

**「Percepción」 no añade información.** Vuelve a dibujar de forma más visible los contornos y el contraste que el original contiene, sin crear detalle que no tenía. En imágenes de contornos ya pronunciados, como dibujo lineal, documentos, UI y logotipos, mejora poco más de la mitad de los casos (20/35). Un resultado borroso, como el de un original diminuto muy ampliado, sigue borroso con 「Percepción」 ([9.10](#910-nitidez-percibida)).

**Logotipo y cómic solo con entradas JPEG.** Al volver a medir las entradas limpias no superaron la regla de adopción, así que con entradas limpias como PNG no se usan los valores explorados de restauración a gran escala. Una imagen que originalmente era JPEG y se volvió a guardar como PNG se considera entrada limpia aunque tenga artefactos JPEG.

**Ilustración no adoptada.** Los resultados son contradictorios según las condiciones, así que no se usan los valores explorados de restauración a gran escala. Queda margen para revisarla con otras degradaciones (ruido fuerte, otros desenfoques, originales nítidos de partida) y con imágenes más grandes.

**Ámbito de la exploración.** La exploración se basa en fragmentos pequeños (entrada de 64 px) y 3 imágenes por clase. La cantidad de enfoque, la sobreoscilación y otros parámetros tendían a pegarse al extremo del rango, y al ampliar el rango la mejora no pasó de +0.1 dB.

**Factor automático que solo mira la memoria.** No hay límite de tiempo de procesamiento. A 8×, el resultado tiene cientos de megapíxeles. El presupuesto de memoria es una estimación de la memoria del dispositivo, así que la 「Garantía」(보장) se refiere a este presupuesto, porque el navegador no informa con exactitud de la memoria libre.

**Gradación de entradas de 8 bits.** La entrada se lee en 8 bits a través del lienzo del navegador. El tramado y el guardado en 16 bits solo conservan los valores intermedios que la ampliación crea; no eliminan las bandas que ya había en el original (sobre todo a 1× o en superficies muy planas). Si se desactiva el tramado, el guardado en 8 bits es un simple redondeo y las bandas se ensanchan tanto como el factor.

**Gestión del color.** La gama amplia solo se conserva hasta Display P3. Los originales con colores fuera de P3, como Adobe RGB, BT.2020 o ProPhoto, se comprimen dentro de P3, y el PNG guardado lleva un perfil P3. Esto se debe a que el lienzo solo ofrece sRGB y Display P3 como espacios de color de trabajo. Aun así, la compresión es menor que si se leyera en sRGB (en superficie CIE xy del triángulo BT.2020, sRGB cubre el 53 % y P3 el 72 %). Conservar el perfil original tal cual exigiría leer sin conversión de color, procesar en esa codificación y volver a adjuntar el perfil, así que queda como función aparte. Los navegadores que no admiten lienzos P3 (por ejemplo, Firefox actualmente) leen en sRGB como antes. Las imágenes pegadas se consideran siempre sRGB. Si un AVIF guarda la información de color solo dentro del flujo de bits AV1 (cabecera de secuencia), sin caja `colr`, no se lee y se considera sRGB (los codificadores habituales, como libavif, escriben `colr`). Los navegadores sin `DecompressionStream` (Safari 15.2–16.3) no pueden descomprimir el perfil iCCP comprimido de PNG, así que los PNG que indican gama amplia solo con iCCP, sin cICP, se consideran sRGB. El análisis de 「Automático」 (clasificación, mediciones de detalle, color automático) recibe del navegador también las imágenes P3 como valores sRGB, de modo que se mantiene la referencia sRGB con la que se entrenó el clasificador. La vía de guardado desde lienzo de los navegadores sin compresión en flujo no adjunta directamente el perfil P3.

**Supuesto de tasa de compresión.** El factor automático cuenta la tasa de compresión del PNG guardado como el máximo medido por factor en 47 imágenes del conjunto de datos más un margen del 10 % (en 16 bits con textura activada, 0.934 × 1.1, el 102 % del tamaño sin comprimir). Las imágenes con mucho más ruido que esas 47 se comprimen menos de lo supuesto, así que incluso con un factor 「Garantía」 podría faltar memoria durante el guardado.

**Datos ausentes.** La caché RGBA para entrenar el clasificador (unos 1.3 GB) y los datos del experimento de cuadrícula DCCI (unos 1.3 GB) no están en el repositorio por su tamaño. Ambos se pueden regenerar a partir de los originales de `data/ds`.

---

## 12. Registro de cambios

### v1.2.1 — Nombre y versión visibles

- El título de la aplicación pasa a ser **Upscaly: 무손실 지향 확대기** (Upscaly: ampliador de imágenes que aspira a no perder información), y la versión (v1.2.1) aparece junto al nombre en la barra superior. En pantallas estrechas (720 px o menos) solo se muestran «Upscaly» y la versión.
- Los cálculos y los archivos guardados son idénticos a los de la v1.2.0.

### v1.2.0 — Dos objetivos: precisión y percepción

- **「Objetivo: Precisión · Percepción」.** Por defecto, 「Precisión」 amplía lo más cerca del original como antes (la salida es idéntica byte a byte a la v1.1.1). 「Percepción」 lleva la pendiente de los contornos y el contraste local al nivel de la referencia para verse más nítido. Nunca supera el rango de brillo del entorno, así que no aparecen bordes blancos. En 70 casos de validación, el error frente a la referencia bajó de 0.600 a 0.505, con ruido en zonas planas ×1.05 y PSNR −0.12 dB ([2.9](#29-precisión-y-percepción-son-objetivos-distintos), [9.10](#910-nitidez-percibida)).
- Intensidad de percepción 25–150 %; los preajustes y 「Automático」 no cambian esta elección. El factor automático cuenta el margen, la memoria y la razón de compresión del PNG guardado en 「Percepción」.

### v1.1.1 — Corrección de los problemas detectados en la revisión de código

- **Imágenes transparentes ampliadas por etapas.** En las etapas intermedias, el alfa pasa por los mismos pesos direccionales, límite de celda y enfoque entre etapas que el color. En DCCI 4×, el error de color de los bordes semitransparentes bajó de un máximo de 19–38 niveles a 1–4 niveles, y desaparecieron las costuras entre teselas (hasta 206 niveles) del preajuste de logotipo a 4×.
- **Valor inicial de 「Ampliar por etapas」.** Si antes de que termine la medición posterior a abrir la imagen se cambia la casilla o se pulsa 「Automático」, la medición que termine tarde no sobrescribe esa elección.
- **Comprobación de lienzo en blanco.** Si el lienzo del original es totalmente transparente, se lee el original directamente por fragmentos para ver si hay píxeles visibles, y el guardado desde lienzo busca un píxel visible en el resultado y comprueba esa posición. Se detecta el límite del lienzo incluso en imágenes cuya esquina inferior derecha es transparente de por sí o en las que solo se ve un punto.
- **Colores claros en la restauración de textura.** La relación de aclarado se limita al margen del canal mayor, de modo que el tono de los colores claros y saturados no cambia. En consecuencia se reajustó la constante (5.915 → 6.256) y se volvió a medir corrigiendo la medición para que use las mismas condiciones que 「Automático」. También se corrigió que en el modo gamma, con 「Calcular en luz lineal」 desactivado, la textura se desbordara 1.8 veces.
- **Detección de gama.** Aunque haya metadatos o cajas de más de 2 MB antes de la información de color (XMP extendido en JPEG, chunks de texto en PNG, mdat de AVIF antes de meta), se siguen las cabeceras para encontrarla. En AVIF se mira la información de color (colr) asociada a la imagen principal, no a miniaturas ni imágenes auxiliares; en PNG se mira primero cICP sea cual sea el orden de los chunks, pero si los primarios son no especificados (2), se decide por el perfil iCCP. Los archivos de gama amplia se leen con un `<img>` de Data URL aunque la Blob URL esté bloqueada, para conservar el color.
- **Presupuesto tras 「Automático」.** 「Automático」 aplica, empezando por los factores grandes, los ajustes de 「Automático」 de cada factor y elige el primero que queda garantizado. Ya no ocurre que con vecino más próximo y textura activada eligiera 8× para 1200×900 y faltara memoria para preparar el original. Tampoco ocurre ya que se quedara en un factor menor cuando los ajustes se dividían en baratos y caros según el factor (imagen de UI de 4000×3000, factor actual 1.25×, ampliación por etapas en tres etapas: 「Sin garantía」 → 2×).
- **Cálculo de memoria del factor automático.** Se cuentan el PNG comprimido que se acumula durante el guardado (tasa de compresión máxima por factor medida ampliando 47 imágenes del conjunto de datos de 1.1 a 4×, más un margen del 10 %) y los demás originales preparados que quedan en la caché. Por eso, con fotos grandes el factor elegido puede bajar (con un presupuesto de 600 MB, 4000×3000 pasa de 4× a 2×). Al cambiar la profundidad de bits de guardado o la textura, también se recalcula el aviso.

### v1.1.0 — 8 bits sin bandas, colores del móvil, restauración de textura, nueva validación de logotipos

- **Tramado de 8 bits (activado por defecto).** Al convertir a 8 bits se mezcla un tramado fino determinado por las coordenadas, lo que reduce las bandas de los degradados muy ampliados (indicador de bandas −44 %). Se puede desactivar con la casilla 「Tramado de 8 bits」(8비트 디더링). La salida de 16 bits es idéntica byte a byte a la de v1.0.0.
- **Conservación de Display P3.** Si el perfil de color del archivo original es más amplio que sRGB, se lee con un lienzo P3, la pantalla también se dibuja en P3 y el PNG guardado lleva un perfil Display P3 (cICP, iCCP). Los rojos y verdes P3 ya no se comprimen como antes.
- **Restauración de textura (desactivada por defecto).** Se añadió un control deslizante que añade la textura fina perdida según la intensidad de textura del original. Se recomienda 20–30 % para fotos y pintura.
- **Nueva validación de logotipo y cómic.** Se corrigió el error de la validación con entradas limpias y se volvió a medir. Ahora logotipo usa los valores explorados de restauración a gran escala con entradas JPEG, y cómic solo los usa con entradas JPEG.
- El README se reescribió en formato de artículo.

### v1.0.0 — Primera versión trasladada a Claude Code

- El proyecto, que se desarrollaba en local con Claude, se trasladó al repositorio. Se subieron tal cual la aplicación (`upscaler.html`), la versión de referencia, las pruebas, los scripts y resultados de investigación y los datos (567 originales, fragmentos de exploración).
- Los resultados de conversión son idénticos byte a byte a los de la versión de referencia en 134 combinaciones.

---

## 13. Referencias

1. D. Zhou, X. Shen, W. Dong. "Image zooming using directional cubic convolution interpolation." *IET Image Processing* 6(6), 2012.
2. R. Keys. "Cubic convolution interpolation for digital image processing." *IEEE Trans. ASSP* 29(6), 1981.
3. D. P. Mitchell, A. N. Netravali. "Reconstruction filters in computer graphics." *SIGGRAPH* 1988.
4. C. E. Duchon. "Lanczos filtering in one and two dimensions." *J. Applied Meteorology* 18(8), 1979.
5. W. H. Richardson. "Bayesian-based iterative method of image restoration." *JOSA* 62(1), 1972.
6. L. B. Lucy. "An iterative technique for the rectification of observed distributions." *Astronomical Journal* 79, 1974.
7. C. Tomasi, R. Manduchi. "Bilateral filtering for gray and color images." *ICCV* 1998.
8. K. Dabov, A. Foi, V. Katkovnik, K. Egiazarian. "Image denoising by sparse 3-D transform-domain collaborative filtering." *IEEE TIP* 16(8), 2007.
9. A. Zakhor. "Iterative procedures for reduction of blocking effects in transform image coding." *IEEE TCSVT* 2(1), 1992.
10. Y. Yang, N. P. Galatsanos, A. K. Katsaggelos. "Regularized reconstruction to reduce blocking artifacts of block discrete cosine transform compressed images." *IEEE TCSVT* 3(6), 1993.
11. K. He, J. Sun, X. Tang. "Guided image filtering." *IEEE TPAMI* 35(6), 2013.
12. J. Immerkær. "Fast noise variance estimation." *CVIU* 64(2), 1996.
13. V. A. Marchenko, L. A. Pastur. "Distribution of eigenvalues for some sets of random matrices." *Mat. Sbornik* 72(4), 1967.
14. J. Bigün, G. H. Granlund. "Optimal orientation detection of linear symmetry." *ICCV* 1987.
15. R. Achanta, S. Hemami, F. Estrada, S. Süsstrunk. "Frequency-tuned salient region detection." *CVPR* 2009.
16. Z. Wang, A. C. Bovik, H. R. Sheikh, E. P. Simoncelli. "Image quality assessment: from error visibility to structural similarity." *IEEE TIP* 13(4), 2004.
17. G. D. Finlayson, E. Trezzi. "Shades of gray and colour constancy." *Color and Imaging Conference* 2004.
18. J. van de Weijer, T. Gevers, A. Gijsenij. "Edge-based color constancy." *IEEE TIP* 16(9), 2007.
19. H. H. Bauschke, J. M. Borwein. "On projection algorithms for solving convex feasibility problems." *SIAM Review* 38(3), 1996.
20. L. H. Thomas. "Elliptic problems in linear difference equations over a network." Watson Sci. Comput. Lab. Report, Columbia University, 1949.
21. I. Kutskir. "Fastest Gaussian blur (in linear time)." 2013. Entrada de blog.
22. IEC 61966-2-1:1999. Multimedia systems and equipment — Colour measurement and management — Part 2-1: Default RGB colour space — sRGB.
23. J. Jimenez. "Next generation post processing in Call of Duty: Advanced Warfare." Ponencia en SIGGRAPH 2014 Advances in Real-Time Rendering. (Ruido de gradiente entrelazado)
24. D. J. Field. "Relations between the statistics of natural images and the response properties of cortical cells." *JOSA A* 4(12), 1987.
25. A. van der Schaaf, J. H. van Hateren. "Modelling the power spectra of natural images: statistics and information." *Vision Research* 36(17), 1996.
26. International Color Consortium. "ICC.1:2022 Image technology colour management — Architecture, profile format, and data structure." 2022.
27. W3C. "Portable Network Graphics (PNG) Specification (Third Edition)." 2025. (Chunk cICP)
28. SMPTE EG 432-1:2010. Digital Source Processing — Color Processing for D-Cinema. (Primarios P3)
29. R. A. Ulichney. *Digital Halftoning*. MIT Press, 1987.
