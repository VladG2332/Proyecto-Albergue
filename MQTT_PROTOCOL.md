# 📡 Protocolo MQTT - Proyecto Albergue

Especificación completa del protocolo MQTT, mensajes, temas y comunicación.

## 📋 Tabla de Contenidos

1. [Introducción a MQTT](#introducción-a-mqtt)
2. [Configuración del Broker](#configuración-del-broker)
3. [Temas (Topics)](#temas-topics)
4. [Mensajes](#mensajes)
5. [Quality of Service (QoS)](#quality-of-service-qos)
6. [Ciclo de Vida de Conexión](#ciclo-de-vida-de-conexión)
7. [Ejemplos de Comunicación](#ejemplos-de-comunicación)
8. [Debugging MQTT](#debugging-mqtt)

## 🔤 Introducción a MQTT

MQTT (Message Queuing Telemetry Transport) es un protocolo ligero de publicación/suscripción para IoT.

### Características

| Característica | Valor |
|---|---|
| **Protocolo** | TCP + WebSocket |
| **Modelo** | Pub/Sub (Publicador/Suscriptor) |
| **Versión** | 3.1.1 |
| **Tamaño de Mensaje Típico** | Bytes - Kilobytes |
| **Latencia** | Muy baja (<100ms) |
| **Confiabilidad** | QoS 0, 1, 2 |

### Ventajas para Este Proyecto

- ✅ **Baja latencia**: Ideal para tracking en tiempo real
- ✅ **Bajo ancho de banda**: Eficiente con GPS limitado
- ✅ **Escalable**: Múltiples clientes simultáneos
- ✅ **Confiable**: Garantía de entrega (QoS)
- ✅ **Pub/Sub desacoplado**: Clientes independientes

## ⚙️ Configuración del Broker

### Mosquitto Setup

#### 1. Instalación

```bash
# Windows (usando WSL)
sudo apt-get install mosquitto mosquitto-clients

# macOS
brew install mosquitto

# Docker
docker run -d --name mosquitto -p 1883:1883 -p 9001:9001 eclipse-mosquitto
```

#### 2. Configuración (mosquitto.conf)

```conf
# Puerto MQTT (TCP)
port 1883

# Puerto WebSocket
listener 9001
protocol websockets

# Permitir conexiones anónimas (desarrollo)
allow_anonymous true

# Archivo de contraseñas (producción)
# password_file /etc/mosquitto/passwd

# Persistencia
persistence true
persistence_location /var/lib/mosquitto/

# Log
log_dest file /var/log/mosquitto/mosquitto.log
log_dest stdout
log_type all
```

#### 3. Iniciar Broker

```bash
# Directamente
mosquitto -c /etc/mosquitto/mosquitto.conf

# Como servicio (Linux)
sudo systemctl start mosquitto
sudo systemctl status mosquitto

# Docker
docker run -d --name mosquitto \
  -p 1883:1883 \
  -p 9001:9001 \
  eclipse-mosquitto
```

#### 4. Verificar Conexión

```bash
# Suscribirse a tema
mosquitto_sub -h 192.168.1.137 -t "paquetes/posicion"

# Publicar mensaje
mosquitto_pub -h 192.168.1.137 -t "paquetes/posicion" \
  -m '{"latitude": 20.375, "longitude": -99.982}'
```

## 📌 Temas (Topics)

### Estructura de Temas

```
paquetes/
├── posicion
│   └── Ubicaciones en tiempo real del repartidor
└── ruta
    └── Ruta y destino calculados para el repartidor
```

### Tema: paquetes/posicion

**Tipo**: Flujo continuo
**Publicador**: App Repartidor
**Suscriptores**: Apps Albergue
**Frecuencia**: Cada 5 segundos

**Caso de Uso**:
- Repartidor comparte su ubicación actual
- Albergue recibe y procesa la ubicación
- Se calcula ETA y ruta basada en esta información

**Estructura de Árbol**:
```
paquetes/posicion
├── [App Repartidor 1] publica
├── [App Repartidor 2] publica
└── [App Albergue 1, 2, 3] suscriben
```

### Tema: paquetes/ruta

**Tipo**: Evento generado bajo demanda
**Publicador**: App Albergue
**Suscriptores**: App Repartidor
**Frecuencia**: Cuando se calcula (≈cada 5 segundos)

**Caso de Uso**:
- Albergue calcula ruta hacia su ubicación
- Envía ruta, destino y ETA al repartidor
- Repartidor visualiza en mapa

**Estructura de Árbol**:
```
paquetes/ruta
├── [App Albergue 1] publica
├── [App Albergue 2] publica
└── [App Repartidor 1, 2] suscriben
```

### Jerarquía de Temas Recomendada

Para escalamiento futuro:

```
proyecto-albergue/
├── eventos/
│   ├── gps/
│   │   └── posicion
│   └── entrega/
│       ├── inicio
│       ├── completada
│       └── cancelada
├── datos/
│   ├── rutas/
│   │   └── asignada
│   ├── albergues/
│   │   └── estado
│   └── repartidores/
│       └── estado
├── comandos/
│   ├── parar/viaje
│   └── recalcular/ruta
└── sistema/
    ├── heartbeat
    ├── status
    └── logs
```

## 📨 Mensajes

### Formato Base

Todos los mensajes son JSON válido:

```json
{
  "campo1": "valor1",
  "campo2": 12345,
  "campo3": {
    "subobjeto": "datos"
  }
}
```

### Mensaje: Posición del Repartidor

**Tema**: `paquetes/posicion`
**Dirección**: Repartidor → Albergue

#### Estructura

```json
{
  "latitude": 20.375264824639682,
  "longitude": -99.9821426911384,
  "timestamp": 1700500000000,
  "precision": 5.2,
  "altitude": 2250.5,
  "speed": 35.5,
  "heading": 125.3,
  "deviceId": "repartidor-abc123"
}
```

#### Campos Detallados

| Campo | Tipo | Descripción |
|-------|------|------------|
| `latitude` | number | Latitud (-90 a 90) |
| `longitude` | number | Longitud (-180 a 180) |
| `timestamp` | number | Milisegundos desde epoch |
| `precision` | number | Precisión en metros |
| `altitude` | number | Altitud en metros (opcional) |
| `speed` | number | Velocidad en m/s (opcional) |
| `heading` | number | Dirección en grados (opcional) |
| `deviceId` | string | Identificador del dispositivo |

#### Validación

```javascript
const esValida = (msg) => {
  return msg &&
    typeof msg.latitude === 'number' &&
    typeof msg.longitude === 'number' &&
    Math.abs(msg.latitude) <= 90 &&
    Math.abs(msg.longitude) <= 180 &&
    typeof msg.timestamp === 'number' &&
    msg.timestamp > 0;
};
```

#### Ejemplo Completo

```json
{
  "latitude": 20.37526,
  "longitude": -99.98214,
  "timestamp": 1700500000000,
  "precision": 5.2,
  "altitude": 2250.5,
  "speed": 35.5,
  "heading": 125.3,
  "deviceId": "repartidor-juan-001"
}
```

**Tamaño**: ~200 bytes

### Mensaje: Ruta y Destino

**Tema**: `paquetes/ruta`
**Dirección**: Albergue → Repartidor

#### Estructura

```json
{
  "ruta": [
    [20.375264, -99.982142],
    [20.376285, -99.983165],
    [20.377306, -99.984188],
    [20.389435, -99.996916]
  ],
  "destino": {
    "lat": 20.389435,
    "lng": -99.996916,
    "nombre": "Albergue Centro"
  },
  "eta": 15,
  "distancia": 15.5,
  "velocidadPromedio": 50,
  "timestamp": 1700500000000,
  "albergueId": 2,
  "rutaId": "ruta-20251121-001"
}
```

#### Campos Detallados

| Campo | Tipo | Descripción |
|-------|------|------------|
| `ruta` | array | Array de coords `[[lat,lng], ...]` |
| `destino.lat` | number | Latitud del destino |
| `destino.lng` | number | Longitud del destino |
| `destino.nombre` | string | Nombre del albergue |
| `eta` | number | Minutos estimados |
| `distancia` | number | Kilómetros |
| `velocidadPromedio` | number | km/h |
| `timestamp` | number | Cuando se calculó |
| `albergueId` | number | ID del albergue |
| `rutaId` | string | ID único de la ruta |

#### Validación

```javascript
const esValida = (msg) => {
  return msg &&
    Array.isArray(msg.ruta) &&
    msg.ruta.length > 1 &&
    msg.ruta.every(p => Array.isArray(p) && p.length === 2) &&
    msg.destino &&
    typeof msg.destino.lat === 'number' &&
    typeof msg.destino.lng === 'number' &&
    typeof msg.eta === 'number' &&
    msg.eta > 0;
};
```

#### Ejemplo Completo

```json
{
  "ruta": [
    [20.375264, -99.982142],
    [20.376265, -99.983143],
    [20.377266, -99.984144],
    [20.378267, -99.985145],
    [20.379268, -99.986146],
    [20.380269, -99.987147],
    [20.381270, -99.988148],
    [20.382271, -99.989149],
    [20.383272, -99.990150],
    [20.384273, -99.991151],
    [20.385274, -99.992152],
    [20.386275, -99.993153],
    [20.387276, -99.994154],
    [20.388277, -99.995155],
    [20.389435, -99.996916]
  ],
  "destino": {
    "lat": 20.389435,
    "lng": -99.996916,
    "nombre": "Albergue Centro"
  },
  "eta": 15,
  "distancia": 15.5,
  "velocidadPromedio": 50,
  "timestamp": 1700500000000,
  "albergueId": 2,
  "rutaId": "ruta-20251121-001"
}
```

**Tamaño**: ~1-2 KB (varía con ruta)

## 🎯 Quality of Service (QoS)

### Niveles QoS

#### QoS 0: At Most Once (Máximo una vez)

```javascript
client.send(message, TOPIC_POSICION, 0);
```

- **Garantía**: Ninguna
- **Caso de Uso**: Datos no críticos (GPS actualización frecuente)
- **Ventaja**: Menor overhead
- **Riesgo**: Mensaje puede perderse

#### QoS 1: At Least Once (Al menos una vez)

```javascript
client.send(message, TOPIC_POSICION, 1);
```

- **Garantía**: Mínimo una entrega
- **Caso de Uso**: Mensajes importantes (rutas)
- **Ventaja**: Confiabilidad + bajo overhead
- **Riesgo**: Posibles duplicados

#### QoS 2: Exactly Once (Exactamente una vez)

```javascript
client.send(message, TOPIC_POSICION, 2);
```

- **Garantía**: Exactamente una entrega
- **Caso de Uso**: Transacciones críticas
- **Ventaja**: Máxima confiabilidad
- **Riesgo**: Mayor overhead

### Recomendación para Este Proyecto

```
Tema                  QoS    Razón
─────────────────────────────────────
paquetes/posicion     0      Frecuente, tolerante
paquetes/ruta         1      Importante, no frecuente
```

## 🔌 Ciclo de Vida de Conexión

### Secuencia de Conexión

```
1. CREATE CLIENT
   new Client(host, port, path, clientId)
   
2. SET HANDLERS
   client.onMessageArrived = function(msg) { ... }
   client.onConnectionLost = function(err) { ... }
   
3. CONNECT
   client.connect({
     useSSL: false,
     onSuccess: () => { console.log('Conectado'); },
     onFailure: (err) => { console.error(err); }
   })
   
4. SUBSCRIBE
   client.subscribe(topic, {
     onSuccess: () => { console.log('Suscrito'); }
   })
   
5. PUBLISH (Si es necesario)
   let message = new Message(payload);
   message.destinationName = topic;
   client.send(message);
   
6. DISCONNECT
   client.disconnect();
```

### Implementación en App-Albergue

```javascript
useEffect(() => {
  if (!viajeActivo) return;
  
  // Paso 1: Crear cliente
  const client = new Client(
    BROKER_HOST,
    BROKER_PORT,
    BROKER_PATH,
    'albergue-' + Math.random()
  );
  
  // Paso 2: Configurar handlers
  client.onMessageArrived = (message) => {
    const data = JSON.parse(message.payloadString);
    if (message.destinationName === TOPIC_POSICION) {
      setRepartidorCoords(data);
      calcularETAYRuta(data, client);
    }
  };
  
  client.onConnectionLost = (responseObject) => {
    console.error('Conexión perdida:', responseObject);
    setMqttConnected(false);
  };
  
  // Paso 3: Conectar
  client.connect({
    useSSL: false,
    onSuccess: () => {
      setMqttClient(client);
      setMqttConnected(true);
      
      // Paso 4: Suscribirse
      client.subscribe(TOPIC_POSICION);
    },
    onFailure: (e) => {
      Alert.alert('Error MQTT', 'No se pudo conectar');
      setMqttConnected(false);
    }
  });
  
  // Paso 6: Cleanup
  return () => {
    if (client && client.isConnected()) {
      client.disconnect();
    }
  };
}, [viajeActivo]);
```

## 💬 Ejemplos de Comunicación

### Escenario 1: Repartidor Inicia Viaje

```
Tiempo    Evento
──────────────────────────────────────
T0        Repartidor abre app
T0+1      Ingresa nombre, presiona "Iniciar"
T0+2      Obtiene ubicación GPS
T0+3      Se conecta a MQTT
T0+5      Publica ubicación (Mensaje 1)
T0+10     Publica ubicación (Mensaje 2)
T0+15     Publica ubicación (Mensaje 3)
T0+16     Albergue publica ruta
T0+17     Repartidor recibe ruta
          (Visualiza en mapa)
T0+20     Publica ubicación (Mensaje 4)
T0+25     Publica ubicación (Mensaje 5)
T0+30     Publica ubicación (Mensaje 6)
T0+31     Albergue recalcula y publica ruta
          actualizada (ETA reducido)
...       (Continúa hasta llegar)
```

### Escenario 2: Múltiples Repartidores

```
Broker MQTT
├── Tema: paquetes/posicion
│   ├── Repartidor 1: Publica cada 5s
│   ├── Repartidor 2: Publica cada 5s
│   └── Repartidor 3: Publica cada 5s
│
└── Suscriptores:
    ├── Albergue 1: Recibe todas
    ├── Albergue 2: Recibe todas
    └── Albergue 3: Recibe todas
```

### Flujo Temporal Completo

```json
// T0: Repartidor publica ubicación
{
  "topic": "paquetes/posicion",
  "message": {
    "latitude": 20.375,
    "longitude": -99.982
  }
}

// T1: Albergue recibe
{
  "received": true,
  "topic": "paquetes/posicion"
}

// T2: Albergue procesa y publica ruta
{
  "topic": "paquetes/ruta",
  "message": {
    "ruta": [[20.375, -99.982], ...],
    "destino": { "lat": 20.389, "lng": -99.996 },
    "eta": 15
  }
}

// T3: Repartidor recibe ruta
{
  "received": true,
  "topic": "paquetes/ruta"
}
```

## 🔍 Debugging MQTT

### Cliente Mosquitto

#### Suscribirse a Todos los Temas

```bash
mosquitto_sub -h 192.168.1.137 -t "paquetes/#" -v
```

Salida:
```
paquetes/posicion {"latitude": 20.375, ...}
paquetes/ruta {"ruta": [...], "eta": 15}
paquetes/posicion {"latitude": 20.376, ...}
```

#### Publicar Mensaje de Prueba

```bash
mosquitto_pub -h 192.168.1.137 \
  -t "paquetes/posicion" \
  -m '{"latitude": 20.375, "longitude": -99.982}'
```

### Herramientas Gráficas

#### MQTT.fx

1. Descargar desde https://mqttfx.jensd.de/
2. Configurar:
   - Broker Address: `192.168.1.137`
   - Broker Port: `1883`
3. Conectar
4. Suscribirse a temas

#### MQTTLens (Chrome)

1. Instalar extensión Chrome
2. Conectar a broker
3. Publicar/Suscribirse a temas

### Logs de Mosquitto

```bash
# Ver logs en tiempo real
tail -f /var/log/mosquitto/mosquitto.log

# Ejemplo de salida
1700500000: New connection from 192.168.1.100 on port 1883.
1700500001: New client connected from 192.168.1.100, client id albergue-0.5, clean session 0.
1700500002: Received SUBSCRIBE from albergue-0.5
1700500002:   paquetes/posicion (QoS 0)
1700500005: Received PUBLISH from (unknown), topic "paquetes/posicion", retain 0, QoS 0, payload len 127
```

### Validación de Mensajes

```javascript
// Validar estructura de mensaje
const validarMensaje = (topic, payload) => {
  try {
    const data = JSON.parse(payload);
    
    if (topic === 'paquetes/posicion') {
      console.assert(typeof data.latitude === 'number');
      console.assert(typeof data.longitude === 'number');
      console.assert(Math.abs(data.latitude) <= 90);
      console.assert(Math.abs(data.longitude) <= 180);
    } else if (topic === 'paquetes/ruta') {
      console.assert(Array.isArray(data.ruta));
      console.assert(data.destino && data.destino.lat);
      console.assert(typeof data.eta === 'number');
    }
    
    return true;
  } catch (err) {
    console.error('Mensaje inválido:', err);
    return false;
  }
};
```

---

**Última actualización**: Noviembre 2025
**Versión**: 1.0.0
