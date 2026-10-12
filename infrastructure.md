# Infraestructura

Este documento lista la infraestructura del juego y sus detalles de diseno. La infraestructura se comporta como una entidad o unidad sin movimiento, no como una propiedad directa del tile. Esto permite que tenga vida, faccion controladora, estado de captura, efectos, dano recibido, eventos y participacion en condiciones de victoria o derrota.

## Reglas generales

- La infraestructura puede existir en batalla y en el mapa estrategico.
- En batalla, la infraestructura se representa como entidad inmovil.
- En campana, la infraestructura puede otorgar beneficios estrategicos si la region esta controlada.
- Algunas infraestructuras son construibles durante la fase estrategica.
- La construccion estrategica esta limitada por region y por conexion entre regiones.
- Que dos regiones sean adyacentes no implica que todas las infraestructuras puedan construirse entre ellas.
- Puede haber dos regiones adyacentes entre las que no se pueda construir camino u otra conexion logistica.
- Las reglas de captura y combate interno aplican a cualquier infraestructura capturable.
- Los caminos (`road`) no son capturables.
- Cada region define una lista de infraestructuras construibles.
- Cada tipo de infraestructura construible tiene un nivel maximo permitido por region.
- Cada region tambien tiene un maximo total de niveles de infraestructura.
- La suma de todos los niveles de infraestructura de una region no puede superar el maximo total de la region.
- Esto significa que una region puede permitir, por ejemplo, nivel maximo 5 para caminos y nivel maximo 5 para aeropuerto, pero si el maximo total de infraestructura de la region es 7, no se puede maximizar todo a la vez.
- Para controlar infraestructura durante batalla es necesario ingresar tropas de infanteria.
- El numero de slots requeridos para controlar una infraestructura depende del tipo de infraestructura.
- La captura se mide por slots completos.
- La UI debe mostrar faccion controladora, progreso de captura y slots restantes.
- La infraestructura puede recibir dano de combate y dano de terreno.
- Algunas infraestructuras pueden colapsar, explotar, bloquear movimiento o modificar linea de vision.
- Los efectos de infraestructura deben quedar registrados por slot para autosave, replay y futuro multiplayer.

## Lista inicial

## Capacidad de infraestructura por region

Cada region debe declarar en JSON que infraestructura puede construir y hasta que nivel. Ademas, cada region debe declarar un limite total compartido por todas sus infraestructuras.

Ejemplo conceptual:

```json
{
  "regionId": "region-a",
  "infrastructureCapacity": {
    "totalMaxLevel": 7,
    "maxByType": {
      "road": 5,
      "airport": 5
    }
  }
}
```

En ese ejemplo, la region podria tener camino nivel 5 y aeropuerto nivel 2, o camino nivel 3 y aeropuerto nivel 4, pero no camino nivel 5 y aeropuerto nivel 5 al mismo tiempo porque la suma seria 10 y excederia el maximo total de 7.

Este modelo fuerza decisiones estrategicas por region y evita que todas las regiones puedan llegar al maximo en toda la infraestructura disponible.

Para el vertical slice, las 3 regiones permiten caminos (`road`) hasta nivel 3. La region intermedia permite nivel 1 de todos los tipos de infraestructura. Cada region del vertical slice tiene `totalMaxLevel` 20.

La region del jugador y la region enemiga inician con camino (`road`) nivel 1 y base militar (`military_base`) nivel 1. La region neutral inicia con nivel 1 en toda infraestructura permitida, excepto aeropuerto (`airport`) nivel 5.

Ejemplo parcial del vertical slice:

```json
[
  {
    "regionId": "player-region",
    "infrastructureCapacity": {
      "totalMaxLevel": 20,
      "maxByType": {
        "road": 3
      }
    },
    "builtInfrastructure": {
      "road": 1,
      "military_base": 1
    }
  },
  {
    "regionId": "neutral-region",
    "infrastructureCapacity": {
      "totalMaxLevel": 20,
      "maxByType": {
        "road": 3,
        "airport": 5,
        "port": 1,
        "shipyard": 1,
        "railway": 1,
        "factory": 1,
        "dam": 1,
        "power_grid": 1,
        "power_plant": 1,
        "refinery": 1,
        "oil_well": 1,
        "pipeline": 1,
        "mine": 1,
        "foundry": 1,
        "warehouse": 1,
        "workshop": 1,
        "nuclear_reactor": 1,
        "research_center": 1,
        "farm": 1,
        "hospital": 1,
        "education_center": 1,
        "military_base": 1,
        "anti_air_battery": 1,
        "artillery_battery": 1,
        "psyops": 1,
        "radar": 1,
        "electronic_warfare_station": 1,
        "missile_silo": 1,
        "space_command_center": 1,
        "space_launcher": 1,
        "satcom": 1,
        "intelligence_office": 1,
        "police": 1,
        "fire_rescue": 1
      }
    },
    "builtInfrastructure": {
      "road": 1,
      "airport": 5,
      "port": 1,
      "shipyard": 1,
      "railway": 1,
      "factory": 1,
      "dam": 1,
      "power_grid": 1,
      "power_plant": 1,
      "refinery": 1,
      "oil_well": 1,
      "pipeline": 1,
      "mine": 1,
      "foundry": 1,
      "warehouse": 1,
      "workshop": 1,
      "nuclear_reactor": 1,
      "research_center": 1,
      "farm": 1,
      "hospital": 1,
      "education_center": 1,
      "military_base": 1,
      "anti_air_battery": 1,
      "artillery_battery": 1,
      "psyops": 1,
      "radar": 1,
      "electronic_warfare_station": 1,
      "missile_silo": 1,
      "space_command_center": 1,
      "space_launcher": 1,
      "satcom": 1,
      "intelligence_office": 1,
      "police": 1,
      "fire_rescue": 1
    }
  },
  {
    "regionId": "enemy-region",
    "infrastructureCapacity": {
      "totalMaxLevel": 20,
      "maxByType": {
        "road": 3
      }
    },
    "builtInfrastructure": {
      "road": 1,
      "military_base": 1
    }
  }
]
```

## Tipos de infraestructura y codigos JSON

Los tipos de infraestructura deben declararse con codigos estables en `snake_case` para usarse en JSON, guardado, balance, UI y reglas de campana.

| Nombre | Codigo JSON |
| --- | --- |
| Aeropuerto | `airport` |
| Puerto | `port` |
| Astillero | `shipyard` |
| Caminos | `road` |
| Vias de ferrocarril | `railway` |
| Fabricas | `factory` |
| Presas | `dam` |
| Red electrica | `power_grid` |
| Planta de generacion de electricidad | `power_plant` |
| Refineria | `refinery` |
| Pozo petrolero | `oil_well` |
| Gasoductos y oleoductos | `pipeline` |
| Minas | `mine` |
| Fundiciones | `foundry` |
| Almacenes | `warehouse` |
| Talleres | `workshop` |
| Reactores nucleares | `nuclear_reactor` |
| Centros de investigacion | `research_center` |
| Granjas | `farm` |
| Clinicas y hospitales | `hospital` |
| Escuelas y universidades | `education_center` |
| Bases militares | `military_base` |
| Baterias antiaereas | `anti_air_battery` |
| Baterias de artilleria | `artillery_battery` |
| Psyops | `psyops` |
| Radares | `radar` |
| Estaciones de guerra electronica | `electronic_warfare_station` |
| Silos de misiles | `missile_silo` |
| Centros de comando espacial | `space_command_center` |
| Lanzaderas espaciales | `space_launcher` |
| SatCom | `satcom` |
| Oficinas de inteligencia | `intelligence_office` |
| Policia | `police` |
| Bomberos y rescatistas | `fire_rescue` |

### Torre de comunicaciones

Infraestructura tactica orientada a mando, informacion y control de zona. En campana puede justificar mejoras de comunicacion, aumento de ordenes por slot, mejor deteccion estrategica o desbloqueo de misiones.

Detalles iniciales:

- Captura: requiere infanteria dentro.
- Slots de captura sugeridos: 6.
- Beneficio estrategico sugerido: progreso tecnologico o aumento temporal/permanente de capacidad de ordenes.
- Efecto en batalla sugerido: mejora de vision tactica o informacion de objetivos.
- Riesgo de terreno: bajo.
- Colapso: no prioritario para MVP.

### Planta de energia

Infraestructura critica para sostener operaciones regionales. En campana puede otorgar energia, habilitar edificios permanentes, alimentar defensas o desbloquear tecnologias.

Detalles iniciales:

- Captura: requiere infanteria dentro.
- Slots de captura sugeridos: 8.
- Beneficio estrategico sugerido: recurso energetico o habilitacion de infraestructura avanzada.
- Efecto en batalla sugerido: puede activar o desactivar defensas locales.
- Riesgo de terreno: medio.
- Colapso/explosion: posible fuente de dano de terreno.

### Deposito de suministros

Infraestructura logistica para recursos, reparacion, municion o refuerzos. En campana puede aumentar recursos estrategicos, capacidad logistica o preparacion de grupos.

Detalles iniciales:

- Captura: requiere infanteria dentro.
- Slots de captura sugeridos: 5.
- Beneficio estrategico sugerido: recursos, reposicion o mejora logistica.
- Efecto en batalla sugerido: punto de reabastecimiento o objetivo vulnerable.
- Riesgo de terreno: medio/alto si contiene combustible o municion.
- Colapso/explosion: puede generar dano de terreno.

### Puente estrategico

Infraestructura de movilidad. Controlarlo puede abrir rutas terrestres, acelerar avance o permitir atacar regiones conectadas.

Detalles iniciales:

- Captura: requiere infanteria dentro o control del area inmediata.
- Slots de captura sugeridos: 4.
- Beneficio estrategico sugerido: ruta logistica, movimiento regional o acceso a nuevas misiones.
- Efecto en batalla sugerido: paso obligado, cuello de botella o punto defensivo.
- Riesgo de terreno: medio.
- Colapso: puede bloquear movimiento terrestre.

### Camino

Infraestructura estrategica construible durante la fase estrategica. Reduce el tiempo necesario para desplazar tropas entre regiones, pero no todas las regiones permiten construir caminos.

Detalles iniciales:

- Construccion: fase estrategica.
- Captura: no capturable.
- Disponibilidad: limitada por region y por conexion entre regiones.
- Restriccion clave: dos regiones pueden ser adyacentes y aun asi no permitir construccion de camino entre ellas.
- Beneficio estrategico: reduce el costo de movimiento estrategico.
- Regla de traslado estrategico:
  - si hay caminos en una de las dos zonas del traslado, reduce 1 slot,
  - si hay caminos en ambas zonas del traslado, reduce 2 slots,
  - el costo minimo despues de reducciones siempre es 1 slot.
- Efecto en batalla: no aplica directamente para el vertical slice salvo que una mision lo represente como carretera o ruta tactica.
- Persistencia: debe guardarse como infraestructura construida o mejora estrategica de region/conexion.

### Bunker

Infraestructura defensiva. Puede servir como objetivo de control, punto de resistencia o estructura que protege tropas.

Detalles iniciales:

- Captura: requiere infanteria dentro.
- Slots de captura sugeridos: 10.
- Beneficio estrategico sugerido: defensa regional o reduccion de riesgo ante ataques.
- Efecto en batalla sugerido: cobertura alta, posicion fortificada o punto de aparicion defensivo.
- Riesgo de terreno: bajo/medio.
- Colapso: posible, pero debe ser dificil.

### Base militar

Infraestructura militar de produccion y organizacion. Permite entrenar infanteria y sirve como punto base para preparar fuerzas terrestres.

Detalles iniciales:

- Construccion: fase estrategica.
- Beneficio estrategico: permite entrenar infanteria.
- Relacion con unidades: sin base militar, la region no puede entrenar infanteria localmente.
- Relacion con fabricas: si la region tambien tiene fabricas, puede construir otros tipos de unidades.
- Efecto en batalla sugerido: punto de despliegue, defensa o objetivo de alto valor.
- Riesgo de terreno: medio por municion, combustible o concentracion de tropas.
- Colapso/explosion: puede generar dano de terreno si la mision lo requiere.

### Fabrica

Infraestructura industrial para construccion de unidades no-infanteria.

Detalles iniciales:

- Construccion: fase estrategica.
- Beneficio estrategico: permite construir tipos de unidades distintos de infanteria.
- Tipos de unidades construibles: pendiente para despues.
- Relacion con base militar: la base militar entrena infanteria; la fabrica habilita otros tipos de unidades.
- Efecto en batalla sugerido: objetivo industrial, fuente de refuerzos o condicion de mision.
- Riesgo de terreno: medio/alto.
- Colapso/explosion: puede generar dano de terreno.

### Estacion de radar

Infraestructura de deteccion. En campana puede afectar rutas aereas, riesgo de airdrops, reconocimiento y eventos tacticos.

Detalles iniciales:

- Captura: requiere infanteria dentro.
- Slots de captura sugeridos: 6.
- Beneficio estrategico sugerido: deteccion regional, informacion de rutas o modificadores contra ataques aereos.
- Efecto en batalla sugerido: mejora de reconocimiento o deteccion de activos aereos.
- Riesgo de terreno: bajo.
- Colapso: no prioritario para MVP.

### Bateria antiaerea

Infraestructura defensiva contra rutas aereas y airdrops. No bloquea airdrops, pero aplica probabilidad de perdidas y probabilidad de dano si la ruta cruza su cobertura.

Detalles iniciales:

- Captura: requiere infanteria dentro.
- Slots de captura sugeridos: 7.
- Beneficio estrategico sugerido: cobertura antiaerea regional.
- Efecto estrategico: probabilidad de perdida de transportes y probabilidad de dano a unidades transportadas.
- Efecto en batalla sugerido: amenaza contra drones o activos aereos.
- Riesgo de terreno: medio si contiene municion.
- Colapso/explosion: puede generar dano de terreno.
- Sabotaje: no permitido en la regla inicial.

### Aeropuerto

Infraestructura logistica y aerea. Permite desplazar tropas rapidamente entre dos zonas y sirve como punto clave para movilidad estrategica, refuerzos y control operacional.

Detalles iniciales:

- Captura: requiere infanteria dentro.
- Nivel vertical slice: 5.
- Captura vertical slice: usa las reglas generales de captura de infraestructura.
- Slots de captura sugeridos: 8.
- Beneficio estrategico: permite desplazar tropas rapidamente entre dos zonas.
- Efecto estrategico sugerido: habilita rutas rapidas de traslado entre regiones conectadas o regiones compatibles.
- Regla de traslado estrategico:
  - desplazar tropas entre regiones sin infraestructura tarda 5 slots estrategicos,
  - si hay caminos en una de las dos zonas, reduce 1 slot,
  - si hay caminos en ambas zonas, reduce 2 slots,
  - si hay puente aereo, es decir aeropuerto en zona de origen y destino, reduce 4 slots por cada zona que se cruce.
  - el costo minimo despues de reducciones siempre es 1 slot.
- Efecto en batalla sugerido: punto de entrada/salida de refuerzos, activos aereos o control de pista.
- Riesgo de terreno: medio/alto por combustible, municion o aeronaves estacionadas.
- Colapso/explosion: puede generar dano de terreno.
- Relacion con vista aerea: debe ser visible y relevante para ordenes aereas.

## Infraestructura del vertical slice

La infraestructura clave de la region neutral sera un aeropuerto.

El aeropuerto permite desplazar tropas rapidamente entre dos zonas. Para el vertical slice, debe servir como objetivo principal de captura y como primer ejemplo de infraestructura que modifica la campana despues de una batalla.
