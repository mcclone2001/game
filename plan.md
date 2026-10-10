# Plan del juego RTS

## Objetivo

Construir un juego de estrategia en tiempo real top-down, inicialmente single-player, con una arquitectura preparada para evolucionar a multiplayer casual. Debe ejecutarse en navegador para PC y movil, y poder empaquetarse primero para Google Play y luego para App Store.

## Decisiones cerradas

- Genero: estrategia en tiempo real.
- Vista: top-down.
- Fantasia/tema: guerra moderna postapocaliptica con elementos sci-fi.
- Duracion objetivo de partida: 60+ minutos.
- Modelo temporal: slots discretos de 5 segundos para envio de ordenes, con animacion continua del combate.
- Limite de ordenes por slot: variable segun tecnologias adquiridas por cada jugador.
- Tamano de mapa promedio: 1024x1024 tiles.
- Soporte de mapas: el motor debe soportar distintos tamanos de mapa.
- Control movil: modo comandante, con navegacion de mapa por arrastre, pinch y scroll.
- Economia: no hay economia durante la batalla; los beneficios se aplican despues segun zonas controladas.
- Tipos de unidades: aereas, terrestres, maritimas, anfibias y airdrops.
- No habra heroes ni comandantes como unidades especiales.
- Construccion en batalla: estructuras defensivas, temporales y despliegues tacticos.
- Edificios permanentes: se crean fuera de batalla durante la fase estrategica.
- Niebla de guerra: tactica, con ocultamiento por terreno y habilidades.
- IA enemiga single-player: adaptativa.
- Guardado/carga: autosave por slots.
- Replays: basados en registro de ordenes.
- Primera version: single-player.
- Evolucion esperada: multiplayer casual asincrono por slots y cooperativo contra IA.
- Campana: persistente tanto en single-player como en multiplayer/cooperativo.
- Estilo visual: top-down pixel art.
- Monetizacion: free-to-play con cosmeticos.
- Offline: todo el single-player debe funcionar offline.
- El progreso offline no afecta el estado del mundo estrategico online/persistente.
- MVP: campana, unidades terrestres y aereas, sin niebla de guerra, IA basica y solo navegador.
- Tecnologias: se investigan en la campana estrategica y tambien se obtienen como recompensas por controlar zonas.
- Arbol tecnologico: comun para todas las facciones.
- Facciones: sin diferencias visuales de unidades; se diferencian por simbolos en el mapa.
- Numero de facciones: variable, con batallas de 2 o mas facciones.
- MVP: 2 facciones.
- Debe existir la posibilidad de que un nuevo atacante entre a media partida.
- Control de unidades: se comandan grupos pequenos, no unidades individuales.
- Los grupos pueden dividirse y fusionarse libremente.
- Reorganizar grupos durante combate consume ordenes/slots.
- Los grupos pueden ser mixtos completos.
- Tamano de grupo: sin maximo fijo.
- Tamano minimo de grupo: 1 unidad.
- Resolucion de combate: por slots de 5 segundos.
- Tipos de dano: dano de terreno y dano de combate.
- Primera meta tecnica: vertical slice.
- Vertical slice: debe incluir pantalla de organizacion de grupos y dos vistas de batalla, nivel tierra y vista aerea.
- Cambio entre vistas de batalla: boton toggle.
- Victoria/derrota: variable segun mision.
- Victoria MVP: controlar infraestructura clave durante un numero variable de slots segun la infraestructura.
- Control de infraestructura: requiere ingresar tropas de infanteria.
- Derrota MVP: variable segun mision.
- Derrota vertical slice: quedarse sin infanteria.
- Mapa de batalla del vertical slice: 128x128 tiles.
- Coordenadas vertical slice: simulacion en tiles logicos y render independiente.
- Tile logico: terreno, costo de movimiento, restriccion aerea, bloqueo de movimiento, vision y cobertura.
- Hazards/dano de terreno: inferidos del tipo de terreno.
- Infraestructura: entidad/unidad sin movimiento.
- Infraestructura: documentada en `infrastructure.md`.
- Terreno: documentado en `terreno.md`.
- Primera pantalla a implementar: mapa estrategico.
- Mapa estrategico del vertical slice: 3 regiones, una del jugador, una neutral y una del enemigo.
- Conexion del mapa estrategico vertical slice: linea jugador -> neutral -> enemigo.
- Primera batalla del vertical slice: region neutral.
- Infraestructura clave de la region neutral: aeropuerto.
- Aeropuerto: permite desplazar tropas rapidamente entre dos zonas.
- Movimiento estrategico base entre regiones sin infraestructura: 5 slots.
- Caminos: reducen 1 slot si existen en una zona, 2 slots si existen en ambas.
- Puente aereo: aeropuerto en origen y destino, reduce 4 slots por cada zona que se cruce.
- Movimiento estrategico minimo despues de reducciones: 1 slot.
- Persistencia de campana: zonas, infraestructura, tecnologias, grupos/unidades, dano/bajas, recursos estrategicos y estado narrativo.
- Mapa estrategico: regiones conectadas.
- Ataques estrategicos: regiones adyacentes, regiones remotas por airdrops si la ruta evita antiaereos o aceptando dano, y ataques especiales por eventos narrativos.
- Medidas antiaereas estrategicas: no bloquean airdrops; aplican probabilidad de perdidas y probabilidad de dano.
- Transportes airdrop: distintos tipos con capacidad por tipo de unidad; cada unidad consume capacidad segun su tamano.
- Tipo inicial de transporte: transporte mixto.
- Capacidades del transporte mixto: infanteria 100, vehiculos 5, aereos ligeros/drones 10, estructuras/despliegues tacticos 3.
- Unidades MVP: infanteria basica, mortero pequeno, vehiculo ligero, tanque, drone reconocimiento y drone ataque.
- Escala objetivo: 1000+ unidades simultaneas.
- Plataformas objetivo:
  - PC en navegador.
  - Movil en navegador o WebView.
  - Android via Google Play primero.
  - iOS via App Store despues.
- Preferencia tecnica: TypeScript puro esta bien.

## Stack propuesto

- Lenguaje: TypeScript.
- Build/dev server: Vite.
- Motor 2D: Phaser.
- Empaquetado movil: Capacitor.
- Simulacion: core independiente de Phaser.
- Backend futuro: Node.js con WebSocket.
- Persistencia futura: PostgreSQL si hay cuentas, progreso, rankings o partidas persistidas.
- Cache/tiempo real futuro: Redis si hay matchmaking, presencia o colas.

## Arquitectura inicial

```text
packages/
  game-core/
    simulacion, comandos, entidades, combate, IA, pathfinding
  game-client/
    Phaser, render, camara, input, escenas, audio, UI in-game
  game-server/
    futuro servidor multiplayer, reutiliza game-core
  game-mobile/
    Capacitor wrapper para Android/iOS
```

La regla principal es mantener la logica del juego fuera de Phaser. Phaser debe encargarse de render, input, camara, escenas y audio. La simulacion debe vivir en `game-core` para permitir testing, replay, guardado y una migracion posterior a multiplayer.

## Modelo de simulacion

- Fixed timestep para la simulacion.
- Render desacoplado de la simulacion.
- El tiempo de juego se divide en slots de 5 segundos.
- En cada slot, el jugador puede enviar hasta un numero limitado de ordenes.
- El limite de ordenes por slot no es fijo: depende de las tecnologias adquiridas por cada jugador.
- La capacidad de ordenes debe modelarse como una propiedad de faccion/jugador para que pueda cambiar durante la partida.
- Las ordenes se resuelven dentro de la simulacion discreta, pero el cliente debe interpolar y animar el resultado de forma continua.
- Este modelo debe tratarse como turnos simultaneos discretizados con presentacion en tiempo real.
- Comandos de jugador como entrada principal:
  - `select_units`
  - `move_units`
  - `attack_target`
  - `build_structure`
  - `queue_unit`
- Registro de comandos para replay/debug y futura sincronizacion multiplayer.
- Simulacion optimizada por lotes, no todo cada frame.

## Resolucion de combate

- La resolucion de combate se realiza por slots de 5 segundos.
- Durante cada slot se resuelven eventos de dano y resultados tacticos.
- La animacion del cliente debe presentar el combate como continuo, aunque la resolucion de reglas ocurra por slot.
- Hay dos tipos principales de resolucion de dano:
  - Dano de terreno.
  - Dano de combate.
- El dano de terreno representa efectos originados por el mapa, estructuras o eventos ambientales, por ejemplo explosivos, derrumbes de edificios, incendios, radiacion, minas o hazards temporales.
- El dano de combate representa ataques directos entre unidades o grupos, por ejemplo proyectiles, lasers, misiles, fuego automatico o ataques especiales.
- El motor debe mantener separados los origenes de dano para balance, replay, debug, resistencias, mitigaciones y reportes post-batalla.
- Los eventos de dano deben quedar registrados por slot para que autosave, replay y multiplayer futuro puedan reconstruir la resolucion.

## Controles

- En movil se usara un modo comandante.
- La navegacion del mapa debe soportar arrastre, pinch zoom y scroll.
- La emision de ordenes debe favorecer decisiones tacticas por slot y no microgestion precisa por frame.
- La UI movil debe mostrar claramente:
  - slot actual y proximo slot,
  - ordenes disponibles,
  - ordenes restantes del slot,
  - tecnologias o efectos que modifican el limite de ordenes.

## Economia y capa estrategica

- No hay economia activa durante una batalla.
- La batalla se enfoca en combate, posicionamiento, control tactico y cumplimiento de objetivos.
- Al concluir una batalla, si el jugador decide controlar una zona, los beneficios de esa zona se aplican a la economia.
- La economia pertenece a una capa estrategica entre batallas.
- Las zonas controladas pueden otorgar beneficios como recursos, tecnologias, capacidad logistica, informacion, produccion o modificadores militares.
- Esta separacion reduce la carga de microgestion durante el combate y encaja con el modo comandante.
- Los edificios permanentes pertenecen a la fase estrategica, no a la batalla.

## Tecnologias

- Las tecnologias pertenecen a la campana estrategica.
- El jugador puede investigar tecnologias fuera de batalla.
- Controlar zonas puede otorgar tecnologias, progreso tecnologico o bonificaciones relacionadas.
- Las tecnologias pueden modificar el limite de ordenes por slot.
- Las tecnologias tambien pueden desbloquear unidades, estructuras defensivas, despliegues tacticos, sensores, mejoras de movilidad, mejoras de combate o ventajas logisticas.
- El sistema debe soportar tecnologias permanentes y bonificaciones ligadas al control de zonas.
- El arbol tecnologico sera igual para todas las facciones.
- Las diferencias entre facciones, si existen, no deben depender de arboles tecnologicos distintos en la primera version.

## Persistencia de campana

- La campana debe persistir zonas controladas.
- La campana debe persistir infraestructura construida.
- La campana debe persistir tecnologias investigadas.
- La campana debe persistir grupos y unidades sobrevivientes.
- La campana debe persistir dano, bajas permanentes o desgaste acumulado cuando aplique.
- La campana debe persistir recursos estrategicos.
- La campana debe persistir estado narrativo y misiones desbloqueadas.
- La persistencia offline local no debe modificar el mundo estrategico online/persistente.

## Mapa estrategico

- El mapa estrategico de campana estara dividido en regiones conectadas.
- Cada region puede contener infraestructura, beneficios, riesgos, misiones, estado de control y conexiones hacia otras regiones.
- Las conexiones entre regiones deben influir en avance, defensa, rutas logisticas, ataques y entrada de nuevos atacantes.
- Controlar una region puede otorgar beneficios economicos, tecnologicos, logisticos, narrativos o militares.
- Desde el mapa estrategico, el jugador debe poder seleccionar zonas y entrar a batallas relacionadas.
- Las facciones pueden atacar regiones adyacentes.
- Las facciones pueden atacar regiones remotas mediante airdrops.
- Para un airdrop remoto, la ruta no debe caer bajo cobertura de medidas antiaereas emplazadas, salvo que el jugador decida aceptar el dano por pasar.
- Las medidas antiaereas emplazadas deben influir en rutas, riesgo, dano esperado y decisiones estrategicas.
- Eventos narrativos pueden abrir ataques especiales fuera de las reglas normales.

## Medidas antiaereas estrategicas

- Las medidas antiaereas no bloquean airdrops.
- Las medidas antiaereas aplican probabilidad de perdidas.
- Si se pierde un transporte, se pierde todo lo que trae.
- Las medidas antiaereas tambien aplican probabilidad de dano.
- Si ocurre dano sin perdida total del transporte, algunas unidades transportadas pueden recibir dano.
- No es posible sabotear medidas antiaereas en esta regla inicial.
- La UI estrategica debe mostrar riesgo estimado antes de confirmar un airdrop.
- El resultado de perdidas y dano debe quedar registrado para replay, auditoria y persistencia de campana.

## Transportes y airdrops

- Existen diferentes tipos de transporte para airdrops.
- El tipo inicial sera transporte mixto.
- Cada tipo de transporte define capacidad por tipo de unidad.
- Capacidades iniciales del transporte mixto:
  - infanteria: 100,
  - vehiculos: 5,
  - aereos ligeros/drones: 10,
  - estructuras/despliegues tacticos: 3.
- Ejemplos de capacidad por transporte:
  - capacidad de infanteria: 100,
  - capacidad de vehiculos: 5.
- Las unidades tienen un tamano.
- Los tamanos iniciales de unidades pueden ser 1, 2, 3 o 4.
- Una unidad consume capacidad segun su tamano y su tipo.
- El consumo se aplica contra la capacidad correspondiente del transporte.
- Ejemplo: un mortero pequeno puede consumir 3 puntos de capacidad de infanteria.
- La perdida de un transporte destruye o elimina todo lo que transporta.
- El dano parcial por medidas antiaereas puede afectar algunas unidades transportadas sin destruir el transporte completo.
- La UI debe permitir ver capacidad usada, capacidad restante y riesgo por ruta antes de confirmar el airdrop.

## Unidades

- Categorias principales:
  - Unidades terrestres.
  - Unidades aereas.
  - Unidades maritimas.
  - Unidades anfibias.
  - Airdrops.
- No habra heroes ni comandantes como unidades especiales.
- Las unidades deben modelarse por capacidades y dominios de movimiento para permitir counters, terreno y reglas de pathfinding diferentes.
- Los airdrops deben tratarse como despliegues tacticos o entradas al campo de batalla, no necesariamente como unidades persistentes.
- Las unidades no se comandan de manera individual durante la batalla.
- El jugador comanda pequenos grupos de unidades.
- Los grupos pueden reorganizarse.
- Debe ser posible mover unidades entre grupos.
- La reorganizacion esta permitida dentro y fuera del combate.
- Reorganizar durante combate consume ordenes/slots.
- Un grupo puede dividirse y fusionarse libremente.
- Los grupos pueden mezclar unidades de cualquier tipo: terrestres, aereas, maritimas, anfibias, soporte o futuras categorias compatibles.
- El tamano minimo de grupo es 1 unidad.
- No hay maximo fijo de unidades por grupo.
- Aunque no exista maximo fijo, la UI y la simulacion deben manejar grupos grandes sin perder legibilidad ni rendimiento.
- La UI debe priorizar ordenes a grupos, composicion del grupo, estado agregado y reorganizacion clara entre grupos.

### Unidades iniciales del MVP

- Infanteria basica:
  - tamano: 1,
  - capacidad usada: infanteria.
- Mortero pequeno:
  - tamano: 3,
  - capacidad usada: infanteria.
- Vehiculo ligero:
  - tamano: 1,
  - capacidad usada: vehiculos.
- Tanque:
  - tamano: 3,
  - capacidad usada: vehiculos.
- Drone de reconocimiento:
  - tamano: 1,
  - capacidad usada: aereos ligeros/drones.
- Drone de ataque:
  - tamano: 2,
  - capacidad usada: aereos ligeros/drones.

## Facciones

- Las facciones no tienen diferencias visuales en las unidades.
- La diferenciacion visual principal entre facciones sera por simbolos en el mapa.
- El arbol tecnologico es comun para todas las facciones.
- Si existen diferencias de faccion, deben estar en identidad, posicion estrategica, control territorial, objetivos o contexto narrativo, no en apariencia de unidades.
- El numero de facciones por batalla debe ser variable.
- Las batallas deben soportar 2 o mas facciones.
- El MVP tendra 2 facciones.
- El sistema debe permitir que un nuevo atacante ingrese a media partida.
- La entrada tardia de una faccion debe modelarse como evento de batalla para que pueda guardarse, reproducirse en replay y sincronizarse en multiplayer futuro.

## Estructuras y despliegues

- Durante la batalla se permiten estructuras defensivas.
- Durante la batalla se permiten estructuras temporales.
- Durante la batalla se permiten despliegues tacticos, como sensores, minas, torretas, barreras u otros sistemas de apoyo.
- Los edificios permanentes se crean durante la fase estrategica, fuera de la batalla.
- Las estructuras de batalla deben tener duracion, limite, costo tactico o restriccion de uso para evitar que sustituyan a la capa estrategica.

## Vision y niebla de guerra

- Habra niebla de guerra tactica.
- Los enemigos pueden ocultarse por terreno.
- Los enemigos pueden ocultarse por habilidades.
- La deteccion debe considerar vision directa, sensores, drones, satelites, interferencia u otros sistemas tacticos.
- El terreno debe poder modificar visibilidad, cobertura y deteccion.
- La UI debe diferenciar entre:
  - terreno conocido,
  - terreno visible actualmente,
  - contactos detectados,
  - contactos perdidos o inciertos.

## IA enemiga

- La IA enemiga debe ser adaptativa.
- Debe poder cambiar su comportamiento segun el estado de la batalla.
- Comportamientos esperados:
  - defender zonas clave,
  - presionar posiciones vulnerables,
  - capturar o disputar objetivos,
  - flanquear si encuentra rutas favorables,
  - responder a la composicion de unidades del jugador,
  - retirarse o reagruparse si pierde ventaja.
- Para el MVP, la IA puede implementarse por capas:
  - reglas tacticas simples,
  - evaluacion de amenazas,
  - seleccion de objetivos,
  - ajustes basados en el desempeno del jugador.
- La IA no debe depender de informacion que estaria oculta por la niebla de guerra salvo que el escenario lo justifique explicitamente.

## Guardado, carga y replays

- El juego debe soportar autosave por slots.
- Cada autosave debe poder restaurar el estado de la batalla en un limite de slot consistente.
- El sistema de replay debe basarse en el registro de ordenes.
- Los replays deben almacenar:
  - semilla de aleatoriedad,
  - estado inicial de la batalla,
  - ordenes emitidas por slot,
  - eventos relevantes si son necesarios para auditoria/debug.
- El autosave y el replay deben aprovechar el mismo modelo de simulacion discreta.
- El registro de ordenes tambien servira como preparacion para multiplayer futuro.

## Multiplayer futuro

- El multiplayer futuro sera asincrono por slots.
- Todos los jugadores envian sus ordenes para el slot correspondiente.
- Cuando las ordenes requeridas estan disponibles, el servidor resuelve el slot.
- El resultado del slot se distribuye a los clientes para animacion continua y revision tactica.
- Tambien debe existir cooperativo contra IA.
- La campana debe ser persistente en single-player, multiplayer y cooperativo.
- El servidor debe ser autoritativo para resolver slots en multiplayer.
- El diseno debe tolerar jugadores desconectados, ordenes no enviadas y tiempos limite por slot.
- El modelo de replay/autosave debe servir como base para depurar resoluciones multiplayer.

## Estilo visual

- El estilo visual sera top-down pixel art.
- La direccion visual debe priorizar legibilidad tactica sobre detalle decorativo.
- Las unidades deben leerse claramente a distintos niveles de zoom.
- Los equipos/facciones deben distinguirse por color, silueta e iconografia.
- En movil, la UI y los marcadores tacticos deben seguir siendo legibles en pantallas pequenas.
- El pixel art debe funcionar con mapas grandes y 1000+ unidades sin saturar visualmente la pantalla.

## Monetizacion

- El modelo de monetizacion sera free-to-play con cosmeticos.
- Los cosmeticos pueden incluir skins de unidades, efectos visuales, temas de interfaz, emblemas, banners o variaciones esteticas de estructuras.
- La monetizacion no debe vender ventajas tacticas directas.
- En multiplayer, los cosmeticos deben ser compatibles con legibilidad competitiva y no ocultar informacion importante.

## Offline

- Todo el single-player debe funcionar offline.
- El jugador debe poder jugar batallas y campana local sin conexion.
- El progreso offline no debe modificar el estado del mundo estrategico online/persistente.
- Cuando haya conexion, se debe distinguir claramente entre:
  - progreso/campana local,
  - mundo estrategico persistente online,
  - cuenta, tienda, cosmeticos y eventos.
- El diseno debe evitar conflictos de sincronizacion donde una accion offline reclame territorio o beneficios del mundo online.

## MVP

- El MVP incluye campana.
- El MVP incluye unidades terrestres y aereas.
- El MVP no incluye unidades maritimas, anfibias ni airdrops.
- El MVP no incluye niebla de guerra.
- El MVP usa IA basica, no IA adaptativa completa.
- El MVP corre solo en navegador.
- Android, iOS, multiplayer, cooperativo, niebla tactica y categorias avanzadas de unidades quedan para fases posteriores.
- El MVP debe validar primero:
  - slots de 5 segundos,
  - limite dinamico de ordenes por tecnologia,
  - combate con animacion continua,
  - campana persistente local,
  - resolucion de zonas despues de batalla.
- La primera meta tecnica sera un vertical slice que conecte campana minima, batalla simple y resolucion de zona.

## Vertical slice

La planificacion tecnica detallada del vertical slice vive en `vertical.md`.

- El vertical slice debe incluir una pantalla de campana minima.
- Debe incluir una pantalla de organizacion de grupos.
- Desde la campana, el jugador debe poder entrar a una batalla.
- La batalla debe incluir 2 facciones.
- La batalla debe incluir unidades terrestres y aereas.
- Las unidades deben controlarse mediante grupos.
- El jugador debe poder reorganizar grupos antes de entrar a combate.
- La batalla debe funcionar con slots de 5 segundos.
- La batalla debe aplicar ordenes limitadas por tecnologia.
- Debe existir resolucion basica de dano de combate.
- Debe existir resolucion basica de dano de terreno.
- Debe existir resolucion de batalla y control de zona.
- Debe existir autosave por slot.
- Debe existir replay simple por registro de ordenes.
- El campo de batalla debe tener dos vistas distintas:
  - vista de nivel de tierra,
  - vista aerea para controlar activos aereos.
- La vista de nivel de tierra debe enfocarse en unidades terrestres, posicionamiento, combate cercano, terreno y estructuras tacticas.
- La vista aerea debe enfocarse en control de activos aereos, rutas, ataques, reconocimiento y apoyo.
- El sistema debe definir claramente que ordenes se emiten desde cada vista y como se sincronizan dentro del mismo slot.
- El jugador alterna entre vista de nivel de tierra y vista aerea mediante un boton toggle.
- Ambas vistas comparten el mismo estado de batalla, el mismo slot actual y el mismo limite de ordenes.
- Ordenes iniciales en vista de tierra:
  - mover grupos terrestres,
  - atacar,
  - desplegar defensas,
  - reorganizar grupos.
- Ordenes iniciales en vista aerea:
  - mover activos aereos,
  - ataque aereo,
  - reconocimiento,
  - intercepcion,
  - airdrop.

## Victoria y derrota

- Las condiciones de victoria y derrota pueden variar segun la mision.
- Para el MVP, la condicion principal de victoria sera controlar infraestructura clave durante cierto numero de slots.
- La infraestructura clave debe tener estado de control por faccion.
- El control debe medirse por slots completos para encajar con la simulacion discreta.
- El numero de slots necesarios para controlar una infraestructura es variable y depende del tipo de infraestructura.
- Para controlar infraestructura es necesario ingresar tropas de infanteria.
- Otras unidades pueden apoyar, atacar, defender o bloquear, pero la captura requiere infanteria.
- La UI debe mostrar progreso de control, faccion dominante y slots restantes para capturar o defender.
- Las reglas de derrota son variables por mision, por ejemplo perder todos los grupos disponibles, perder toda la infanteria, perder la infraestructura propia o agotar un limite de slots.

## Performance para 1000+ unidades

- Spatial hash o grid para queries espaciales.
- Flow fields para movimiento masivo.
- Evitar A* individual por unidad salvo casos puntuales.
- Mapas divididos en chunks/regiones para culling, streaming logico y actualizaciones parciales.
- Pathfinding jerarquico o por regiones para mapas grandes.
- Los sistemas no deben asumir un tamano fijo de mapa.
- Object pools para entidades visuales y efectos.
- IA actualizada por lotes.
- Colisiones y separacion local con vecindarios, no busquedas O(n^2).
- Web Workers para pathfinding, IA o calculos pesados si el prototipo lo exige.

## Roadmap inicial

1. Prototipo tecnico en navegador con 1000 unidades visibles.
2. Camara pan/zoom.
3. Seleccion rectangular de unidades.
4. Sistema de slots de tiempo.
5. Limite dinamico de ordenes por slot segun tecnologias.
6. Ordenes de movimiento.
7. Flow field basico para grupos grandes.
8. Separacion local entre unidades.
9. Animacion continua entre estados de simulacion.
10. Combate simple.
11. Resolucion de batalla y control de zonas.
12. Capa estrategica/economia entre batallas.
13. IA enemiga basica.
14. Guardado/carga por slots.
15. Replay por registro de ordenes.
16. Build Android con Capacitor.
17. Niebla de guerra tactica.
18. Preparacion multiplayer con command log/replay.
19. Servidor multiplayer real.

## Riesgos tecnicos

- Pathfinding por unidad puede saturar CPU.
- IA cada frame no escala a 1000+ unidades.
- Mezclar logica de juego con Phaser dificultaria multiplayer.
- Mobile tendra restricciones de CPU, bateria, memoria y controles tactiles.
- Multiplayer posterior puede requerir reescritura si no se modelan comandos y simulacion determinista desde el inicio.
- Con slots de 5 segundos, el juego debe comunicar claramente cuando una orden aplica al slot actual o al siguiente.
- El limite dinamico de ordenes debe ser visible para evitar frustracion del jugador.
- Un mapa de 1024x1024 tiles puede ser costoso si se carga o procesa completo en cada sistema.
- Grupos sin maximo fijo pueden crear problemas de pathfinding, formacion, UI y balance si no se aplican limites practicos por logistica, velocidad, cohesion o tecnologia.

## Puntos pendientes por cerrar

- Definicion de caminos en el mapa estrategico.

## Preguntas abiertas

1. Como se define si una region tiene caminos?

## Direccion creativa

El juego combina guerra moderna, postapocalipsis y sci-fi. La base visual y mecanica deberia sentirse militar/tactica, con tecnologia avanzada, facciones supervivientes, infraestructura dañada, recursos escasos y unidades o armas experimentales.
