# Logging System (Backend)

Aplicación Spring Boot que recibe logs por HTTP y los reparte con **RabbitMQ** según su nivel.

## ¿Qué hace?

1. Envías un log a `POST /log` con un nivel (`INFO`, `WARNING` o `ERROR`) y un mensaje.
2. El log se publica en el exchange `logs_direct_exchange` usando el **nivel como routing key**.
3. El exchange lo manda a las colas que correspondan:

| Cola                | Recibe                     | Qué muestra en consola |
|---------------------|----------------------------|------------------------|
| `all_logs_queue`    | `INFO`, `WARNING`, `ERROR` | `[MONITOR GENERAL]`    |
| `errors_only_queue` | solo `ERROR`               | `[ALERTA CRÍTICA]`     |

Por eso, un log `ERROR` aparece **dos veces**: en el monitor general y como alerta.

## Archivos principales

| Archivo                       | Para qué sirve                              |
|-------------------------------|---------------------------------------------|
| `LoggingSystemApplication.java` | Arranca la aplicación                     |
| `RabbitMQConfig.java`         | Crea el exchange, las colas y sus bindings  |
| `LogProducerController.java`  | Endpoint `POST /log` que envía el mensaje   |
| `LogConsumer.java`            | Escucha las dos colas e imprime los logs    |
| `application.properties`      | Puerto (8080) y conexión a RabbitMQ         |

## Requisitos

- Java 17 o superior
- Docker (para levantar RabbitMQ)

No necesitas instalar Maven: el proyecto trae `mvnw`.

## Cómo ejecutarlo

```bash
# 1. Levantar RabbitMQ (desde la carpeta guia-2.1.3)
docker compose up -d

# 2. Iniciar el backend (desde esta carpeta)
./mvnw spring-boot:run        # En Windows: mvnw.cmd spring-boot:run
```

- Backend: http://localhost:8080
- Panel de RabbitMQ: http://localhost:15672 (usuario `guest`, clave `guest`)

## Probarlo

```bash
curl -X POST http://localhost:8080/log \
  -H "Content-Type: application/json" \
  -d '{"level":"ERROR","message":"No se pudo conectar a la base de datos."}'
```

Revisa la consola del backend: verás el mensaje del monitor general y la alerta crítica.

También puedes usar el frontend en `../logging-frontend` (corre en http://localhost:5173).
