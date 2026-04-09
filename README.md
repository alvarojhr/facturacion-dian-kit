# facturacion-dian-kit

Toolkit open source para integrar facturación electrónica DIAN en Colombia.

`facturacion-dian-kit` nace como un servicio HTTP self-hosted con un core reutilizable en Python. El repositorio está organizado desde el inicio para evolucionar sin romper su identidad hacia una arquitectura `core + server + SDKs`.

## Qué resuelve

Este proyecto busca reducir la fricción técnica de una integración DIAN al concentrar en un solo kit:

- construcción de XML UBL;
- cálculo de CUFE y CUDE;
- firma digital;
- envelopes SOAP y consumo de servicios DIAN;
- parsing de respuestas;
- una API HTTP pública lista para integrarse desde un ERP o cualquier otro sistema.

Hoy el primer entregable público no es un SDK. Es un servidor HTTP con un core reusable, pensado para despliegue propio.

## Qué incluye el repositorio

- `packages/core`: lógica de dominio DIAN, XML, firma, SOAP y parsing.
- `packages/server`: servidor FastAPI con el contrato HTTP público.
- `packages/sdk-python`: espacio reservado para un futuro SDK de Python.
- `packages/sdk-ts`: espacio reservado para un futuro SDK o cliente TypeScript.
- `.agents/skills/dian-integration`: skill y metadata de agente para adopción, troubleshooting y habilitación.

## Alcance actual

- envío de documentos electrónicos a DIAN;
- consulta de estado por `tracking_id`;
- generación de `AttachedDocument` en ZIP;
- consulta de adquiriente;
- consulta de rangos de numeración;
- empaquetado con Docker para self-hosting.

## A quién le sirve

- equipos que ya tienen un ERP, POS o backend propio y necesitan conectarse con DIAN;
- implementadores que prefieren correr el servicio en su propia infraestructura;
- equipos técnicos que quieren una base abierta antes de invertir en un SDK más formal.
- alternativa a MATIAS API

## Inicio rápido

1. Copia la plantilla de entorno.

```powershell
Copy-Item .env.example .env -Force
```

2. Instala los paquetes en modo editable.

```powershell
python -m pip install -e ./packages/core -e "./packages/server[dev]"
```

3. Ejecuta las validaciones locales.

```powershell
python scripts/validate_skill.py
python -m ruff check .
python -m mypy packages/core/src packages/server/src
python -m pytest
```

4. Inicia la API.

```powershell
uvicorn facturacion_dian_kit.server.app:app --host 0.0.0.0 --port 8000
```

## Docker

```powershell
docker build -t facturacion-dian-kit .
docker run --rm -p 8000:8000 --env-file .env facturacion-dian-kit
```

## API pública inicial

Endpoints disponibles:

- `POST /api/v1/documents/submissions`
- `GET /api/v1/documents/submissions/{tracking_id}`
- `POST /api/v1/attached-documents`
- `POST /api/v1/customers/lookup`
- `POST /api/v1/numbering-ranges/lookup`
- `GET /health`

Ejemplo de payload para envío:

```json
{
  "client_reference": "cliente-ref-001",
  "document": {
    "number": "FDK000001",
    "type": "FACTURA_ELECTRONICA",
    "issue_date": "2026-03-12",
    "issue_time": "14:30:00-05:00",
    "payment_method": "CASH"
  },
  "buyer": {
    "document_number": "800199436",
    "document_type": "NIT",
    "name": "Empresa Ejemplo S.A.S."
  },
  "resolution": {
    "number": "18764000001",
    "prefix": "FDK"
  },
  "totals": {
    "subtotal": 100000,
    "tax_total": 19000,
    "total": 119000
  },
  "line_items": [
    {
      "description": "Tornillo hexagonal 1/4 x 1 zinc",
      "quantity": 100,
      "unit_price": 500,
      "line_total": 50000,
      "tax_type": "IVA_19",
      "tax_amount": 9500
    }
  ],
  "submission_options": {
    "software_id": "software-demo",
    "software_pin": "pin-demo",
    "technical_key": "technical-key-demo",
    "test_set_id": "test-set-demo"
  }
}
```

La respuesta pública normaliza estos campos:

- `submission_id`
- `tracking_id`
- `document_key`
- `qr_url`
- `status`
- `messages`
- `dian_response`
- `artifacts` opcional
- `client_reference`

## Política operativa

- `422` cuando el payload es inválido;
- `503` cuando falta configuración local o el certificado no está disponible o es inválido;
- `502` cuando falla la comunicación con DIAN sin ser timeout;
- `504` cuando DIAN no responde a tiempo;
- `200` cuando DIAN procesa la solicitud y devuelve aceptación o rechazo funcional.

## Skill de integración

El repo incluye un skill de apoyo en:

- `.agents/skills/dian-integration`

Ese skill está pensado para ayudar a:

- mapear datos de un ERP al contrato público;
- revisar variables de entorno;
- explicar rechazos DIAN;
- guiar pruebas de habilitación;
- acompañar la adopción del kit sin depender de credenciales reales en el repositorio.

## Seguridad y uso responsable

- No subas certificados, llaves privadas, archivos `.env` ni credenciales reales de DIAN.
- No uses valores de ejemplo en producción.
- Mantén estable el contrato público dentro de `packages/server`.
- Evita reintroducir terminología o contratos acoplados a un ERP específico dentro de `packages/core`.

## Estado del proyecto

`facturacion-dian-kit` está en una etapa temprana orientada a hardening. La meta es estabilizar primero el servidor y el core, y luego extraer superficies de SDK sin comprometer el contrato público.

## Roadmap

- endurecer el contrato HTTP hacia `1.0.0`;
- consolidar el core reusable;
- publicar SDKs oficiales cuando la interfaz se estabilice;
- mantener el skill y la metadata de agente alineados con la documentación y la API.

## Contribuir

Revisa [CONTRIBUTING.md](CONTRIBUTING.md), [SECURITY.md](SECURITY.md) y [CODE_OF_CONDUCT.md](CODE_OF_CONDUCT.md) antes de abrir cambios o reportes.
