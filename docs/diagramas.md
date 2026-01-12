# Diagramas técnicos de IASystem

Este archivo contiene los esquemas de arquitectura, flujo de decisión y auditoría con blockchain.

## Visualizaciones renderizadas (SVG)

![Banner](./docs/images/banner.svg)

### Flowchart — Validación de API

![Flowchart](./docs/images/diagram_flow.svg)

```mermaid
flowchart TD
  Client["Cliente"] -->|HTTP request| API["API Gateway / Endpoint"]
  API --> Auth["Autenticación (API Key / JWT)"]
  Auth -->|ok| Rate["Limitador de tasa"]
  Auth -->|fallo| Reject["401 Unauthorized"]
  Rate -->|ok| Schema["Validación de esquema (JSON Schema)"]
  Rate -->|excedido| Throttle["429 Too Many Requests"]
  Schema -->|válido| Biz["Validaciones de negocio"]
  Schema -->|inválido| BadRequest["400 Bad Request"]
  Biz -->|ok| DB["Almacenamiento / Servicio"]
  Biz -->|rechazado| BizReject["422 Unprocessable Entity"]
  DB -->|éxito| Resp["200 OK / Respuesta"]
  DB -->|error| Err["500 Internal Server Error"]

  Resp --> Client
  BadRequest --> Client
  Reject --> Client
  Throttle --> Client
  BizReject --> Client
  Err --> Client
```

### Secuencia — Validación contra proveedor externo

![Sequence](./docs/images/diagram_sequence.svg)

```mermaid
sequenceDiagram
    participant User
    participant IASystem
    participant Validator
    participant ExternalAPI
    participant AuditLog

    User->>IASystem: GET /fx/latest?base=USD&target=INR
    IASystem->>ExternalAPI: GET https://api.fixer.io/latest?base=USD
    ExternalAPI-->>IASystem: {success:true, rates:{INR:83.5}, ...}
    IASystem->>Validator: Validar status=200, JSON schema, keys: success, rates, base
    Validator-->>IASystem: OK / Error (ej: "Missing 'rates'")
    alt OK
        IASystem->>AuditLog: Log {hash_request, timestamp, status: "success"}
        IASystem-->>User: {rate: 83.5, audited: true}
    else Error
        IASystem-->>User: {error: "Invalid response from provider"}
    end
```

### Mapa mental — Principios

![Mindmap](./docs/images/diagram_mindmap.svg)

```mermaid
mindmap
  root((IASystem))
    Privacidad
      No datos personales
      Secrets inyectados
      Zero PII in logs
    Auditoría
      Hash verificable
      No invasiva
      Transparente
    Protección
      Validaciones estrictas
      Fail-safe errors
      No vigilancia
```
