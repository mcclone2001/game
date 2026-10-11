# Vertical slice tecnico

## Objetivo

Construir un vertical slice jugable en navegador que conecte campana minima, organizacion de grupos, batalla por slots y resolucion de control de zona.

El vertical slice no busca cubrir todo el juego. Debe validar las decisiones tecnicas centrales antes de ampliar contenido, mobile, multiplayer o sistemas avanzados.

## Alcance funcional

- Plataforma: navegador.
- Modo: single-player.
- Campana minima con regiones conectadas.
- Mapa estrategico del vertical slice: 3 regiones.
- Regiones iniciales: 1 del jugador, 1 neutral y 1 del enemigo.
- Conexion de regiones: linea jugador -> neutral -> enemigo.
- Capacidad maxima de poblacion del vertical slice: 3 por region.
- Recursos naturales maximos del vertical slice: 3 por cada tipo en cada region.
- JSON base de regiones del vertical slice: `vertical-regions.json`.
- Primera batalla: region neutral.
- Infraestructura clave de la region neutral: aeropuerto.
- Capacidad de infraestructura del vertical slice:
  - las 3 regiones permiten caminos hasta nivel 3,
  - la region intermedia permite nivel 1 de todos los tipos de infraestructura.
- `totalMaxLevel` de cada region del vertical slice: 20.
- Infraestructura inicial construida del vertical slice:
  - region del jugador: `road` nivel 1 y `military_base` nivel 1,
  - region enemiga: `road` nivel 1 y `military_base` nivel 1,
  - region neutral: nivel 1 en toda infraestructura permitida.
- Pantalla de organizacion de grupos.
- Una batalla iniciada desde una region de campana.
- Mapa de batalla: 128x128 tiles.
- Coordenadas: simulacion en tiles logicos y render independiente.
- 2 facciones.
- Unidades terrestres y aereas.
- Grupos mixtos completos.
- Tamano minimo de grupo: 1 unidad.
- Sin maximo fijo de grupo.
- Slots de combate de 5 segundos.
- Limite de ordenes por slot basado en tecnologia.
- Resolucion de combate por slot.
- Animacion continua entre resoluciones de slot.
- Dos vistas de batalla:
  - vista de nivel de tierra,
  - vista aerea.
- Cambio entre vistas mediante boton toggle.
- Control de infraestructura clave mediante infanteria.
- Infraestructura clave del vertical slice: aeropuerto.
- Victoria por controlar infraestructura clave durante un numero de slots definido por la infraestructura.
- Derrota simple inicial: el jugador pierde si se queda sin infanteria.
- Autosave por slot.
- Replay simple por registro de ordenes.
- Primera pantalla a implementar: mapa estrategico.

## Fuera de alcance del vertical slice

- Multiplayer.
- Cooperativo.
- Android/iOS.
- Niebla de guerra.
- IA adaptativa completa.
- Unidades maritimas.
- Unidades anfibias.
- Airdrops reales de campana.
- Monetizacion.
- Cuenta online.
- Mundo estrategico online/persistente.
- Editor completo de mapas.
- Tipos de unidades construibles por fabricas.
- Arbol tecnologico detallado.
- Formula y efectos de felicidad regional.
- Balance completo de infraestructura por region.
- Reglas completas para ataques especiales por eventos narrativos.

## Stack tecnico

- TypeScript.
- Vite.
- Phaser para render, input, camara, escenas y audio.
- Core de simulacion independiente de Phaser.
- Persistencia local en navegador para autosave/replay.

## Estructura propuesta

```text
packages/
  game-core/
    src/
      battle/
      campaign/
      commands/
      combat/
      groups/
      map/
      persistence/
      replay/
      simulation/
      units/
  game-client/
    src/
      scenes/
      ui/
      render/
      input/
      assets/
```

## Modelo de dominio inicial

### Campana

- `CampaignState`
- `Region`
- `RegionConnection`
- `FactionState`
- `TechnologyState`
- `StrategicResource`
- `MissionDefinition`

Cada `Region` debe soportar:

- capacidad maxima de poblacion,
- numero maximo de recursos naturales por tipo,
- felicidad regional pendiente de definir.

Tipos iniciales de recursos naturales:

- yacimientos petroleros,
- superficie agricola util,
- minerales preciosos,
- minerales metalicos,
- minerales radiactivos,
- rios y lagos.

Para el vertical slice, cada una de las 3 regiones tiene capacidad maxima de poblacion 3.
Para el vertical slice, cada una de las 3 regiones tiene maximo 3 en cada tipo de recurso natural. El JSON base completo de las 3 regiones vive en `vertical-regions.json`.

### Batalla

- `BattleState`
- `BattleMap`
- `BattleSlot`
- `FactionBattleState`
- `Group`
- `Unit`
- `Infrastructure`
- `TerrainHazard`
- `BattleEvent`

El mapa de batalla del vertical slice sera de 128x128 tiles.

La simulacion debe trabajar en tiles logicos. El render debe ser independiente para permitir ajustar escala visual, zoom, sprites y resolucion sin cambiar reglas de juego.

Datos minimos de cada tile logico:

- tipo de terreno,
- costo de movimiento,
- restriccion aerea si aplica,
- bloqueo de movimiento,
- bloqueo o modificacion de linea de vision,
- cobertura.

El hazard o dano de terreno se infiere del tipo de terreno. La infraestructura no vive como dato directo del tile: se modela como una entidad/unidad sin movimiento.

El detalle de terreno vive en `terreno.md`.
El detalle de infraestructura vive en `infrastructure.md`.

### Ordenes

- `MoveGroupOrder`
- `AttackOrder`
- `DeployDefenseOrder`
- `ReorganizeGroupOrder`
- `MoveAirAssetOrder`
- `AirAttackOrder`
- `ReconOrder`
- `InterceptOrder`
- `AirdropOrder`

## Unidades iniciales

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

## Resolucion de combate

- La simulacion avanza en slots de 5 segundos.
- Cada slot consume un lote de ordenes.
- La resolucion produce eventos.
- El cliente anima esos eventos de forma continua durante el slot.
- Tipos de dano:
  - dano de combate,
  - dano de terreno.
- El replay debe poder reconstruir la batalla con:
  - estado inicial,
  - semilla,
  - ordenes por slot,
  - eventos derivados si se decide persistirlos.

## Vistas de batalla

### Vista de tierra

Ordenes iniciales:

- Mover grupos terrestres.
- Atacar.
- Desplegar defensas.
- Reorganizar grupos.

Debe enfocarse en:

- posicionamiento terrestre,
- combate cercano,
- terreno,
- infraestructura,
- estructuras tacticas.

### Vista aerea

Ordenes iniciales:

- Mover activos aereos.
- Ataque aereo.
- Reconocimiento.
- Intercepcion.
- Airdrop.

Debe enfocarse en:

- rutas aereas,
- control de drones,
- ataques aereos,
- reconocimiento,
- apoyo tactico.

## Backlog tecnico inicial

1. Inicializar monorepo TypeScript/Vite.
2. Crear `game-core` sin dependencia de Phaser.
3. Crear `game-client` con Phaser.
4. Definir tipos base de campana, batalla, unidades, grupos y ordenes.
5. Implementar pantalla de mapa estrategico con 3 regiones conectadas.
6. Implementar seleccion de region.
7. Implementar datos basicos de control de region, infraestructura y beneficios.
8. Implementar accion para iniciar batalla desde una region.
9. Implementar pantalla de organizacion de grupos.
10. Implementar simulador de slots de 5 segundos en `game-core`.
11. Implementar registro de ordenes por slot.
12. Implementar grupos con division, fusion y reorganizacion.
13. Implementar limite de ordenes por tecnologia.
14. Implementar mapa de batalla simple.
15. Renderizar unidades/grupos en vista de tierra.
16. Renderizar activos aereos en vista aerea.
17. Implementar boton toggle entre vistas.
18. Implementar orden de movimiento de grupo.
19. Implementar orden de ataque basica.
20. Implementar dano de combate.
21. Implementar dano de terreno.
22. Implementar infraestructura capturable por infanteria.
23. Implementar victoria por control de infraestructura durante N slots.
24. Implementar derrota por quedarse sin infanteria.
25. Conectar campana -> organizacion -> batalla -> resultado.
26. Implementar autosave por slot.
27. Implementar replay simple.
28. Agregar pruebas unitarias del core de simulacion.
29. Agregar prueba manual/documentada del flujo completo.

## Criterios de aceptacion

- Se puede iniciar el juego en navegador.
- Se ve una campana minima con al menos 2 regiones conectadas.
- El mapa estrategico tiene 3 regiones: una del jugador, una neutral y una del enemigo.
- Las regiones estan conectadas en linea: jugador -> neutral -> enemigo.
- La primera batalla del vertical slice ocurre por la region neutral.
- Si el jugador gana, la region neutral pasa a control del jugador.
- Se puede seleccionar una region y preparar una batalla.
- Se puede reorganizar al menos un grupo antes de batalla.
- La batalla usa un mapa de 128x128 tiles.
- La simulacion usa tiles logicos desacoplados del render.
- Cada tile logico tiene terreno, costo de movimiento, restriccion aerea, bloqueo de movimiento, modificador de vision y cobertura.
- El hazard/dano de terreno se infiere desde el tipo de terreno.
- La infraestructura se modela como entidad/unidad sin movimiento.
- La lista y detalles de infraestructura se documentan en `infrastructure.md`.
- La infraestructura clave de la region neutral es un aeropuerto.
- El aeropuerto permite desplazar tropas rapidamente entre dos zonas.
- Desplazar tropas entre regiones sin infraestructura tarda 5 slots estrategicos.
- Si hay caminos en una zona del traslado, el costo se reduce 1 slot.
- Si hay caminos en ambas zonas del traslado, el costo se reduce 2 slots.
- Los caminos son infraestructura construible durante la fase estrategica.
- La construccion de caminos esta limitada por region y por conexion entre regiones.
- Dos regiones pueden ser adyacentes y aun asi no permitir construir camino entre ellas.
- Cada region define una lista de infraestructuras construibles y el maximo por tipo.
- Cada region tiene un maximo total de niveles de infraestructura.
- La suma de niveles de infraestructura de la region no puede superar ese maximo total.
- La capacidad de infraestructura de cada region se declara en JSON con maximo total y maximos por tipo.
- Los tipos de infraestructura y sus codigos JSON estan definidos en `infrastructure.md`.
- Ejemplo: una region puede permitir caminos nivel maximo 5 y aeropuerto nivel maximo 5, pero si su maximo total es 7 no puede tener ambos en nivel 5.
- Para el vertical slice, todas las regiones permiten `road` hasta nivel 3.
- Para el vertical slice, la region intermedia permite nivel 1 de todos los tipos de infraestructura.
- Para el vertical slice, cada region tiene `totalMaxLevel` 20.
- Para el vertical slice, la region del jugador inicia con `road` nivel 1 y `military_base` nivel 1.
- Para el vertical slice, la region enemiga inicia con `road` nivel 1 y `military_base` nivel 1.
- Para el vertical slice, la region neutral inicia con nivel 1 en toda infraestructura permitida.
- La infraestructura inicial de la region neutral se considera preexistente; puede superar `totalMaxLevel` porque `totalMaxLevel` limita construccion y mejoras futuras, no el estado inicial heredado.
- Las bases militares permiten entrenar infanteria.
- Si una region tiene fabricas, puede construir otros tipos de unidades.
- Si hay puente aereo, con aeropuerto en zona de origen y destino, el costo se reduce 4 slots por cada zona que se cruce.
- El costo minimo despues de reducciones siempre es 1 slot estrategico.
- Los tipos de terreno iniciales se documentan en `terreno.md`.
- La batalla tiene 2 facciones.
- La batalla tiene unidades terrestres y aereas.
- El jugador puede alternar entre vista de tierra y vista aerea con un boton.
- El jugador puede emitir ordenes por slot.
- El limite de ordenes por slot se respeta.
- El combate se resuelve por slots de 5 segundos.
- El cliente muestra animacion continua del resultado.
- Existe dano de combate.
- Existe dano de terreno.
- La infraestructura solo puede controlarse con infanteria.
- La infraestructura clave del vertical slice es un aeropuerto.
- La batalla termina al cumplir la condicion de control.
- La batalla se pierde si el jugador se queda sin infanteria.
- El resultado modifica la campana local.
- Existe autosave por slot.
- Existe replay simple de la batalla.

## Preguntas abiertas

1. Cual sera el siguiente dato base del vertical slice?
