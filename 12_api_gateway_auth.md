# API Gateway & Autenticación (Auth)

> Este documento detalla el diseño y las mejores prácticas del **API Gateway** y los mecanismos de **autenticación/ autorización** para la plataforma de pagos. Está pensado para arquitectos e ingenieros que implementarán la capa de entrada y seguridad.

---

## 1. Propósito del API Gateway
El **API Gateway** actúa como punto único de entrada para todas las integraciones externas (clientes, merchants, partners) y como proxy/autorizador para las APIs internas. Sus responsabilidades principales:

- Autenticación y autorización (OAuth2/JWT, API keys, mTLS)
- Validación y normalización de requests
- Rate limiting, throttling y protección contra abuso
- Enrutamiento a servicios backend (pay-ins, payouts, etc.)
- Terminación TLS y políticas de seguridad (WAF, CORS)
- Registro de request/response para auditoría (sin incluir datos sensibles)
- Transformaciones ligeras (versionado, compatibilidad)
- Gestión de cuotas y planes (si se exponen APIs a terceros)

---

## 2. Requisitos de seguridad y cumplimiento
- Forzar TLS 1.2+ (ideal 1.3).
- Soporte para mTLS (mutual TLS) en integraciones bancarias críticas.
- No almacenar datos sensibles (PAN, CVV) en logs. Enmascarar o tokenizar antes de loggear.
- Integración con HSM / KMS para manejo de claves y firma de tokens.
- Registro inmune a modificaciones (append-only) para fines de auditoría.
- Compatibilidad con políticas PCI-DSS y GDPR según jurisdicción.

---

## 3. Modelos de autenticación soportados

### 3.1 OAuth2 (recomendado para la mayoría de integraciones)
- **Grant types a soportar:**
  - Authorization Code (con PKCE para apps públicas)
  - Client Credentials (para servidores y servicios)
  - Refresh Token
- **Flujo típico (authorization code):**
  1. Cliente redirige al usuario al Authorization Server.
  2. Usuario autoriza y se emite código.
  3. Cliente intercambia código por `access_token` y `refresh_token`.
  4. `access_token` (JWT) se usa contra API Gateway en `Authorization: Bearer <token>`.
- **Scope y consent:** Scopes por recurso (`payins:write`, `payouts:read`).
- **Rotación de claves y revocación:** soporte para introspection / revocation endpoints.

### 3.2 JWT (JSON Web Tokens)
- `access_token` preferentemente JWT firmado (RS256) y con claims mínimos.
- **Ejemplo payload JWT:**
```json
{
  "iss": "https://auth.platform.local",
  "sub": "client_12345",
  "aud": "https://api.platform.local",
  "exp": 1711600000,
  "iat": 1711596400,
  "scope": "payins:write payouts:read",
  "client_id": "client_12345"
}
```
- Validar firma y claims en el Gateway. Soporte para jwks endpoint para key discovery.

### 3.3 API Keys
- Para integraciones legacy o simples: API keys con límites estrictos.
- Asociadas a cliente y scopes.
- No usadas para usuarios finales.
- Rotación periódica y posibilidad de revocación.

### 3.4 mTLS
- Recomendado para canales servidor-a-servidor (bancos, clearing houses).
- Validación de certificados cliente durante handshake TLS.

---

## 4. Políticas de autorización y scopes
- Implementar control por scope: cada endpoint define scopes mínimos.
- Roles y claims: `role:platform_admin`, `role:merchant`, `role:auditor`.
- Atributos contextuales: `ip_origin`, `device_id`, `geolocation` para reglas de acceso más finas.

---

## 5. Rate limiting, quotas y protección anti-abuso
- **Rate limiting por cliente (API Key / client_id)**: e.g., 100 requests/min
- **Burst window y leaky bucket** para suavizar picos.
- **Quotas mensuales** para modelos comerciales.
- **Circuit breaker**: bloquear integraciones que generan errores consecutivos.
- Integrar WAF (Web Application Firewall) y protección DDoS (Cloud provider / CDN).

---

## 6. Validación de payloads y seguridad de entrada
- Validar schemas (OpenAPI + JSON Schema) en el Gateway.
- Tamaños máximos de payload y límites en campos.
- Sanitizar inputs para evitar ataques de inyección.
- Rechazar requests que incluyan datos sensibles fuera de canales tokenizados (p.ej. PAN no tokenizado).

---

## 7. Webhooks y callbacks: seguridad y entregabilidad
- **Verificación de firma:** cada webhook debe incluir cabecera `X-Signature` (HMAC-SHA256) con secret compartido.
- **Retries y backoff:** política de reintentos (ej. 3 intentos: 1m, 5m, 30m) y uso de dead-letter queue si falla.
- **Idempotency keys:** para asegurar que reintentos no duplican operaciones.
- **Logging y observabilidad de entrega** (status, latencia, errores).

---

## 8. Ejemplos de endpoints (OpenAPI-style)

```yaml
openapi: 3.0.1
paths:
  /v1/payins:
    post:
      summary: Iniciar un pay-in
      security:
        - oauth2: ["payins:write"]
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/PayInRequest'
      responses:
        '201':
          description: Pay-in creado
        '400':
          description: Bad request
```

```yaml
components:
  securitySchemes:
    oauth2:
      type: oauth2
      flows:
        authorizationCode:
          authorizationUrl: https://auth.platform.local/authorize
          tokenUrl: https://auth.platform.local/token
  schemas:
    PayInRequest:
      type: object
      properties:
        amount:
          type: integer
          example: 10000 # cents
        currency:
          type: string
          example: USD
        payment_method:
          type: object
```

---

## 9. Ejemplo de flujo de autenticación (Client Credentials)
1. Servicio backend `client_id` solicita token:
   - POST `https://auth.platform.local/token` grant_type=client_credentials
2. Auth server valida credenciales y responde:
   ```json
   { "access_token": "ey...", "token_type": "bearer", "expires_in": 3600, "scope": "payouts:write" }
   ```
3. Servicio llama a API Gateway con `Authorization: Bearer ey...`.
4. Gateway valida token, scope y firma (si JWT), enruta a servicio de destino.

---

## 10. Manejo de sesiones y tokens
- `access_token` de corta duración (p.ej. 5–15 minutos) para reducir riesgo.
- `refresh_token` con políticas estrictas (rotación, revocación inmediata ante sospecha).
- Soporte para revocation endpoint y token introspection para validación centralizada.
- Recomendación: validar tokens en Gateway sin llamar siempre a Auth server (usar JWKS + verificación local), y usar introspection en casos de revocación sospechosa.

---

## 11. Idempotency y consistencia en entradas
- Requerir `Idempotency-Key` en endpoints mutativos (`POST /v1/payins`, `POST /v1/payouts`, etc.).
- Gateway debe persistir keys por un TTL (ej. 24–72h) y garantizar respuestas idempotentes.
- Proveer cabeceras de respuesta: `Idempotency-Status: processed|replayed`

---

## 12. Logging, audit y privacidad
- Registro estructurado (JSON) con correlación (`request_id`, `trace_id`).
- No loggear PAN/CVV ni PII en texto claro. Enmascarar o tokenizar.
- Logs de auditoría append-only (WORM) con retención configurable.
- Trazabilidad de quién ejecutó solicitudes (client_id, usuario, ip_origin).

---

## 13. Observabilidad y métricas
- Métricas a exponer (por cliente y global):
  - Requests por segundo
  - Latencia P95/P99
  - Tasa de errores 4xx/5xx
  - Retries y dead-letters
  - Throughput de webhooks
- Tracing distribuido (OpenTelemetry) para seguir la ruta de la transacción.
- Dashboards y alertas (errores aumentan >X%, latencia >Y ms).

---

## 14. Escalabilidad, caching y rendimiento
- Escalar el Gateway horizontalmente detrás de load balancers.
- Caching de JWT public keys (JWKS), respuestas idempotency y plantillas de validación.
- Evitar caching de respuestas que contengan datos sensibles.
- Offload pesado: validaciones de schema simples en Gateway; lógica compleja en microservicios.

---

## 15. Gestión de secretos y claves
- Usar KMS / HSM (AWS KMS, Google KMS, Vault, HSM) para almacenar claves privadas.
- Rotación regular de claves y certificados.
- Separación de roles (quién puede ver/usar claves vs. quién puede rotarlas).

---

## 16. Buenas prácticas operativas
- Deploys con canary o blue/green para evitar interrupciones.
- Feature flags para cambios de comportamiento en tiempo real.
- Pruebas de carga y caos engineering para resiliencia.
- Playbooks de incident response (pérdida de conectividad a procesador, fuga de claves, etc.).

---

## 17. Errores y códigos de respuesta recomendados
- `400` Bad Request — payload inválido
- `401` Unauthorized — credenciales inválidas / token expirado
- `403` Forbidden — scope insuficiente
- `404` Not found — recurso no existe
- `409` Conflict — idempotency conflict
- `429` Too Many Requests — rate limit
- `500` Internal Server Error — fallo interno
- `503` Service Unavailable — dependencia externa caída

---

## 18. Ejemplo práctico: Chequeo de seguridad de un Pay-in
1. Gateway valida: token, scope `payins:write`, schema, idempotency
2. Gateway encola evento `PAYIN_INITIATED` con `trace_id`
3. Servicio Pay-ins procesa solicitud, llama al procesador
4. Procesador responde → webhook → Gateway valida firma y publica `PAYIN_COMPLETED`
5. Ledger y Fees Engine consumen evento y actualizan balances

---

## 19. Checklist de implementación
- [ ] Soporte OAuth2 + JWT (RS256) con JWKS
- [ ] Validación de payloads (OpenAPI)
- [ ] Rate limiting + quotas por client_id
- [ ] Idempotency handling
- [ ] Webhook signature verification
- [ ] mTLS para integraciones críticas
- [ ] KMS/HSM para claves y secrets
- [ ] Observability (metrics, tracing, logs)
- [ ] WAF y DDoS protection
- [ ] Audit logs append-only

---

> Este documento puede servir como `12_api_gateway_auth.md`. Si querés, lo guardo como archivo listo para descargar y luego puedo generar archivos complementarios con ejemplos de OpenAPI completos, snippets de código (Node/Java/Python) para verificar tokens JWKS, o playbooks de incident response.
