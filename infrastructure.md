# Infraestructura

Este documento lista la infraestructura del juego y sus detalles de diseno. La infraestructura se comporta como una entidad o unidad sin movimiento, no como una propiedad directa del tile. Esto permite que tenga vida, faccion controladora, estado de captura, efectos, dano recibido, eventos y participacion en condiciones de victoria o derrota.

## Reglas generales

- La infraestructura puede existir en batalla y en el mapa estrategico.
- En batalla, la infraestructura se representa como entidad inmovil.
- En campana, la infraestructura puede otorgar beneficios estrategicos si la region esta controlada.
- Para controlar infraestructura durante batalla es necesario ingresar tropas de infanteria.
- El numero de slots requeridos para controlar una infraestructura depende del tipo de infraestructura.
- La captura se mide por slots completos.
- La UI debe mostrar faccion controladora, progreso de captura y slots restantes.
- La infraestructura puede recibir dano de combate y dano de terreno.
- Algunas infraestructuras pueden colapsar, explotar, bloquear movimiento o modificar linea de vision.
- Los efectos de infraestructura deben quedar registrados por slot para autosave, replay y futuro multiplayer.

## Lista inicial

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

### Bunker

Infraestructura defensiva. Puede servir como objetivo de control, punto de resistencia o estructura que protege tropas.

Detalles iniciales:

- Captura: requiere infanteria dentro.
- Slots de captura sugeridos: 10.
- Beneficio estrategico sugerido: defensa regional o reduccion de riesgo ante ataques.
- Efecto en batalla sugerido: cobertura alta, posicion fortificada o punto de aparicion defensivo.
- Riesgo de terreno: bajo/medio.
- Colapso: posible, pero debe ser dificil.

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
