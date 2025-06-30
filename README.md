Resumen General

SuperBaker.py es un complemento (add-on) para Blender diseñado para automatizar un proceso de "horneado" (baking) de texturas complejo y de varias etapas. Su objetivo principal es generar un único mapa de texturas (un "atlas") para escenas de visualización arquitectónica, manejando de forma inteligente objetos que pueden ocluirse entre sí, como las puertas de un armario. Simplifica un flujo de trabajo que, de hacerse manualmente, sería extremadamente tedioso y propenso a errores.

¿Qué Problema Soluciona?
En la visualización 3D y, especialmente, en la creación de activos para motores de videojuegos o visualización en tiempo real, el "horneado" de texturas es un proceso fundamental. Consiste en pre-calcular la iluminación compleja (sombras, luz rebotada, oclusión ambiental) de una escena y guardarla en un archivo de imagen (una textura). Esto permite que la escena se renderice mucho más rápido, ya que la iluminación no necesita ser calculada en tiempo real.

El problema que SuperBaker ataca es el horneado de escenas con objetos que tienen diferentes estados o que se ocultan entre sí. El ejemplo perfecto, para el que fue diseñado, es un armario empotrado en una pared:

Oclusión: Si horneas la pared con las puertas del armario cerradas, el interior del armario quedará negro y sin información de luz.

Fugas de Luz: Si horneas la pared sin las puertas, la luz entrará en el armario, pero luego las puertas no proyectarán las sombras correctas sobre la pared o el suelo cuando estén cerradas.

Flujo de Trabajo Manual: La solución manual implicaría un proceso largo y meticuloso:

Ocultar las puertas, hornear la pared y el interior del armario.

Mostrar las puertas cerradas, hornear solo las puertas.

Mostrar las puertas abiertas, hornear solo esa versión.

Hacer lo mismo con el techo, asegurándose de que todos los demás objetos contribuyan a la iluminación final.

Todo esto, gestionando la visibilidad de docenas de objetos para cada pasada de horneado.

SuperBaker automatiza por completo este complejo proceso. Garantiza que cada parte de la escena se hornee con la visibilidad correcta de los otros objetos para lograr un resultado de iluminación coherente y realista en un único mapa de texturas.

¿Cómo Funciona?
El script opera siguiendo una lógica muy estructurada, que se puede dividir en los siguientes pasos:

1. Interfaz de Usuario (UI) y Configuración
El add-on crea un panel llamado "SuperBaker" en las Propiedades de Renderizado de Blender. Desde aquí, el usuario puede configurar:

Resolución: La calidad de la textura a hornear (1K, 2K, 4K).

Redimensionamiento: Si se desea, la imagen final puede ser reducida al 50% o 25% de su tamaño original para ahorrar espacio.

Guardado Automático: Si la imagen horneada debe guardarse como un archivo externo.

Ruta y Formato: Dónde guardar la imagen (//textures/ por defecto, que es una carpeta "textures" junto al archivo .blend) y en qué formato (JPEG, PNG, EXR, TIFF), con ajustes de calidad específicos para cada uno.

2. Clasificación de Objetos por Nomenclatura (El Paso Clave)
Esta es la parte más importante de la lógica del script. Cuando el usuario hace clic en el botón de hornear, el add-on no hornea todo a la vez. Primero, examina todos los objetos seleccionados y los clasifica en cuatro grupos basándose en una convención de nombres estricta usando expresiones regulares (regex):

grupo_muros_suelos_frames: Objetos cuyos nombres empiezan con wall, interiorwall, floor, o que coinciden con un patrón específico para marcos de puertas de armario (ej: wall01_closet01_door01_frame01).

grupo_paneles_cerrados: Paneles de puertas que están en estado "cerrado" (ej: ..._closedleftpanel01).

grupo_paneles_abiertos: Paneles de puertas que están en estado "abierto" (ej: ..._openleftpanel01).

grupo_techos: Objetos cuyos nombres empiezan con ceiling.

Si un objeto seleccionado no sigue esta nomenclatura, es ignorado.

3. Preparación para el Horneado
Antes de empezar, el script realiza varias acciones preparatorias:

Guarda el estado actual: Memoriza qué objetos están visibles u ocultos en el render para poder restaurarlo todo al final.

Crea una Imagen Global: Genera una única imagen en blanco (llamada AtlasHorneado_SSGB_...) donde se realizarán todos los horneados.

Configura los Materiales: Itera sobre todos los objetos clasificados y, en cada uno de sus materiales, añade un nodo de "Textura de Imagen" apuntando a la imagen global recién creada. Además, lo selecciona como el nodo activo. Esto le dice a Blender que el resultado del horneado debe ir a parar a este nodo y, por extensión, a la imagen global.

4. Proceso de Horneado Multi-Etapa
Aquí es donde ocurre la magia. El script realiza el horneado en una secuencia de etapas, gestionando la visibilidad de los objetos en cada paso para obtener la iluminación correcta.

Etapa 1: Muros, Suelos y Marcos:

Visibilidad: Hace visibles únicamente los objetos del grupo_muros_suelos_frames.

Horneado: Hornea la iluminación sobre estos objetos. El resultado se plasma en la imagen global.

Etapa 2: Paneles Cerrados:

Visibilidad: Hace visibles los objetos de la Etapa 1 y los grupo_paneles_cerrados. Esto es crucial para que las paredes y suelos proyecten luz y sombras sobre las puertas.

Horneado: Hornea únicamente los grupo_paneles_cerrados. El resultado se añade a la misma imagen global sin borrar lo anterior (usando la opción use_clear=False).

Etapa 3: Paneles Abiertos:

Visibilidad: Mantiene visibles los objetos de la Etapa 1 y ahora muestra los grupo_paneles_abiertos.

Horneado: Hornea únicamente los grupo_paneles_abiertos, añadiendo el resultado al atlas.

Etapa 4: Techos:

Visibilidad: Hace visibles todos los grupos anteriores (muros, suelos, puertas abiertas, puertas cerradas y techos). Esto asegura que la iluminación final sobre el techo sea calculada con la contribución de todos los elementos de la escena.

Horneado: Hornea únicamente los grupo_techos.

5. Post-Procesamiento y Limpieza
Una vez finalizadas todas las etapas de horneado:

Redimensiona y Guarda: Si se ha solicitado, la imagen global se redimensiona y se guarda en la ruta y formato especificados. Utiliza una técnica segura creando una escena temporal para no alterar la configuración de renderizado del usuario.

Restaura la Escena: Devuelve todos los objetos a su estado de visibilidad original y restablece la selección del usuario.

Notificación Final: Muestra un mensaje emergente informando que el proceso ha finalizado.

Capacidades del Script
En resumen, las capacidades de SuperBaker son:

Horneado Multi-Etapa Automatizado: Su función principal es coordinar un horneado secuencial y aditivo.

Creación de Atlas de Textura: Combina el resultado del horneado de múltiples objetos en un único archivo de imagen.

Gestión de Visibilidad Inteligente: Oculta y muestra automáticamente los objetos correctos en cada etapa para lograr un resultado de iluminación físicamente correcto.

Flujo de Trabajo Basado en Nomenclatura: Usa una convención de nombres estricta para identificar y clasificar los objetos a hornear.

Configuración Flexible: Permite al usuario controlar la resolución, el formato de archivo de salida y el post-procesamiento (redimensionamiento).

Proceso No Destructivo: No altera permanentemente la escena del usuario; al finalizar, restaura la visibilidad y selección de objetos a su estado original.

Buena Retroalimentación: Informa al usuario sobre el progreso a través de mensajes en la consola y pop-ups en la interfaz.
