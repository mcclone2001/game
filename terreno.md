# Terreno

El terreno del juego debe funcionar como una capa tactica, no solo visual. Cada tile logico representa una porcion del campo de batalla y define como se mueven, ven, se cubren o se arriesgan las unidades. El render puede cambiar de escala o estilo, pero la simulacion debe apoyarse en tiles logicos independientes.

Para el vertical slice, cada tile debe tener como minimo un tipo de terreno, un costo de movimiento, una posible restriccion aerea, una regla de bloqueo de movimiento, una regla de bloqueo o modificacion de linea de vision y un valor o comportamiento de cobertura. El dano de terreno y los hazards no se definen como campos separados inicialmente; se infieren desde el tipo de terreno. Esto permite que terrenos como explosivos, zonas inestables o estructuras colapsables produzcan dano sin multiplicar reglas desde el principio.

La infraestructura no debe tratarse como una propiedad del tile. La infraestructura se comporta como una entidad o unidad sin movimiento. Esto permite que tenga estado propio, faccion controladora, vida, reglas de captura, eventos, dano recibido y participacion en condiciones de victoria o derrota.

Los tipos de terreno iniciales del vertical slice seran llano, bosque o ruinas ligeras, escombros, agua, carretera, terreno peligroso o explosivo, y edificio o estructura colapsable.

El llano debe ser el terreno base. No bloquea movimiento, no ofrece cobertura relevante y debe tener costo normal. Sirve para probar movimiento, combate y lectura visual sin efectos especiales.

El bosque o las ruinas ligeras deben representar terreno que modifica la visibilidad y ofrece cobertura. Puede aumentar ligeramente el costo de movimiento terrestre. En fases posteriores sera importante para ocultamiento tactico, pero en el MVP sin niebla de guerra puede servir como cobertura y modificador de combate.

Los escombros deben dificultar el movimiento terrestre y pueden ofrecer cobertura parcial. Son utiles para representar zonas urbanas danadas, restos de edificios y caminos bloqueados parcialmente. No necesariamente bloquean por completo, pero si deben hacer que una ruta por escombros sea peor que una carretera o terreno llano.

El agua debe restringir movimiento terrestre normal. En el MVP, sin unidades maritimas ni anfibias, puede funcionar como bloqueo o borde tactico. Mas adelante servira para unidades maritimas, anfibias, puentes, cruces y rutas especiales.

La carretera debe reducir el costo de movimiento terrestre. Debe crear rutas tacticamente valiosas y tambien vulnerables. En campana y batalla puede ayudar a explicar avances rapidos, convoyes o refuerzos.

El terreno peligroso o explosivo debe inferir dano de terreno. Puede representar minas, combustible, municion inestable, radiacion local, fuego o zonas con riesgo de detonacion. La resolucion del dano debe ocurrir por slot, y el evento debe quedar registrado para replay y autosave.

El edificio o estructura colapsable debe poder bloquear movimiento o linea de vision, ofrecer cobertura y generar dano de terreno si colapsa. Aunque se represente visualmente como parte del mapa, si tiene estado relevante debe modelarse como entidad inmovil. El colapso debe producir eventos por slot para que la animacion continua pueda mostrar derrumbe, explosion o dano aplicado.

El objetivo tecnico inicial es que el terreno permita decisiones simples pero reales: moverse por carretera es rapido, cruzar escombros cuesta mas, el agua bloquea, las ruinas dan cobertura, y el terreno peligroso puede danar unidades. Con eso basta para validar la separacion entre simulacion, render, dano de combate y dano de terreno.
