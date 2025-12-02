---
layout: post
title: ¿Cómo conectar Kakfa-UI con Event Hubs de Azure?
published: true
---

## "Introducción"

Si usamos Eventhubs de Azure como broker Kafka, y queremos usar Kafka-UI para monitorizar los topics, tenemos que tener en cuenta algunas cosas para que funcione.
Hablando siempre de autenticación utilizando SASL_SSL con mecanismo PLAIN. (Esto no funciona para Managed Identity).

En principio sabemos que podemos definir un connection string SAS para conectarnos a Event Hubs, pero Kafka-UI no soporta directamente este formato, por lo que tenemos que hacer algunos ajustes.

> KAfka en generar no soporta este formado, sino que hay que hacer un mapeo de los valores del connection string a las propiedades que Kafka espera, más específicamente en el usuario y password, además de protocolo de comunicación.

## Configuración de Client Kafka

Para conectar un cliente Kafka a Event Hubs, tenemos que mapear los valores del connection string a las propiedades de usuario y password.
Pero primero hay que configurar:

- Protocol: SASL_SSL
- Mechanism: PLAIN

Y luego en las propiedades de autenticación:
- Username: $ConnectionString
- Password: Endpoint=sb://<NAMESPACE>.servicebus.windows.net/;SharedAccessKeyName=<KEY_NAME>;SharedAccessKey=<KEY_VALUE>;EntityPath=<EVENT_HUB_NAME>

> El EntityPath es opcional, y solo si queremos conectarnos a un Event Hub específico. Si no se especifica, se puede acceder a todos los Event Hubs dentro del namespace.

Básicamente el username es el literal ```$ConnectionString``` (con el signo $) y el password es el connection string completo.

## Conectar Kafka-UI

En el caso de Kafka-UI tenemos un truco adicional, que sería agregar al user name un $ adicional al inicio, quedando así:
- Username: $$ConnectionString
- Password: Endpoint=sb://<NAMESPACE>.servicebus.windows.net/;SharedAccess

Esto en la propiedad **KAFKA_CLUSTERS_0_PROPERTIES_SASL_JAAS_CONFIG** (reemplazar el 0 por el índice del cluster que estemos configurando).

En el caso de un docker-compose.yml, quedaría algo así:

```yaml
version: '2'
services:
  kafka-ui:
    image: provectuslabs/kafka-ui:latest
    ports:
      - 9999:8080
    environment:
      - KAFKA_CLUSTERS_0_NAME=azure
      - KAFKA_CLUSTERS_0_BOOTSTRAPSERVERS=xxxx.servicebus.windows.net:9093
      - KAFKA_CLUSTERS_0_PROPERTIES_SECURITY_PROTOCOL=SASL_SSL
      - KAFKA_CLUSTERS_0_PROPERTIES_SASL_MECHANISM=PLAIN
      - KAFKA_CLUSTERS_0_PROPERTIES_SASL_JAAS_CONFIG=org.apache.kafka.common.security.plain.PlainLoginModule required username='$$ConnectionString' password='Endpoint=sb://<NAMESPACE>.servicebus.windows.net/;SharedAccessKeyName=<KEY_NAME>;SharedAccessKey=<KEY_VALUE>';
```
El connection string tiene que tener scope del namespace (lo que sería el broker de Kafka).

**Especial cuidado al doble $ en el username y al ; final**


Nos leemos.



