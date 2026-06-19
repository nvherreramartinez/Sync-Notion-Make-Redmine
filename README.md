# Sync-Notion-Make-Redmine
Prototipo de integración automática entre formularios de Notion y Redmine para optimizar la carga y trazabilidad de tickets de un Call Center. Incluye el mapeo de datos, lógica condicional de payloads y análisis de bloqueos de red perimetral.

# Integración Sincronizada: Notion ↔ Redmine (Call Center)

Este proyecto detalla el diseño, la lógica de mapeo y la arquitectura de automatización para conectar los formularios de carga de un Call Center en **Notion** con el gestor de incidentes corporativo **Redmine**, optimizando la gestión de derivaciones de salud y reclamos ciudadanos.

> ⚠️ **Estado del Proyecto:** **Prueba de Concepto (PoC) / Implementación Parcial.**  
> La lógica de integración y el mapeo de datos entre plataformas fueron resueltos y validados exitosamente en un entorno local utilizando **Make.com**. Sin embargo, la implementación definitiva en producción se encuentra pausada debido a restricciones de seguridad perimetral (firewall e infraestructura on-premise del lado del servidor Redmine).

---

## 📋 Resumen del Proyecto

El objetivo principal era automatizar el flujo de trabajo de los agentes del Call Center. En lugar de duplicar cargas, el agente completa una fila en Notion (Gestión Hospitalaria o Reclamos) y el sistema se encarga de:
1. Detectar el nuevo registro.
2. Crear la petición (*Issue*) correspondiente en Redmine dentro del proyecto y *tracker* correctos con sus campos personalizados.
3. Devolver el **Nº de Issue** generado y la **Fecha de Envío** a la fila original de Notion para mantener la trazabilidad.

---

## 🛠️ Tecnologías y Herramientas
*   **Notion API:** Como base de datos operativa y frontend de carga para los agentes.
*   **Redmine API:** Sistema destino para la resolución de tickets de la red de gobierno.
*   **Make.com:** Plataforma de integración (iPaaS) utilizada para el prototipado y diseño de los payloads.
*   **n8n (Self-Hosted):** Identificado como el puente de infraestructura definitivo (*On-Premise Hub*).

---

## 🚀 Logros Alcanzados (Lo que sí funciona)

### 1. Modelado de Datos y Mapeo de Atributos
Se estructuraron los payloads JSON requeridos por la API de Redmine, diferenciando los flujos mediante lógica condicional:

*   **Flujo A: Gestión Hospitalaria** (Tracker ID: `57`)
    *   Mapeo dinámico de *Asunto*, *Descripción* y el campo select personalizado de *Hospital* (ID: `344`).
*   **Flujo B: Reclamos** (Tracker ID: `67`)
    *   Mapeo de *Observaciones de Carga*, *Cliente* (ID: `615`), *Motivo* (ID: `619`) y *Organización Resolutora* (ID: `620`).
    *   Inyección de valores fijos requeridos por diseño (e.g., *Medio de Contacto* como `"Call Center"` fijo, ID: `616`).

#### Ejemplo de Estructura de Payload Diseñado (Reclamos):
```json
{
  "issue": {
    "project_id": "atencion-ciudadanos",
    "tracker_id": 67,
    "subject": "{{1.properties.Asunto.title[].plain_text}}",
    "description": "{{1.properties.Observaciones.rich_text[].plain_text}}",
    "custom_fields": [
      { "id": 615, "value": "{{1.properties.Cliente.select.name}}" },
      { "id": 616, "value": "Call Center" },
      { "id": 619, "value": "{{1.properties.Motivo.select.name}}" },
      { "id": 620, "value": "{{1.properties.Organización Resolutora.select.name}}" }
    ]
  }
}
### 2. Conectividad Externa Validada
* Configuración exitosa del módulo de escucha activa (*Watch Database Items*) en Notion mediante Webhooks de Make.
* Control de duplicados mediante filtros de estado (procesar únicamente registros sin `Nº Issue Redmine` asignado).

---

## 🛑 Bloqueos y Desafíos Técnicos (Lo que no se pudo implementar)

### El Desafío del Perímetro de Red (Firewall & On-Premise)
El servidor de Redmine se encuentra alojado dentro de una **red gubernamental interna altamente custodiada**. A pesar de contar con la `X-Redmine-API-Key` y las credenciales correspondientes:

* **Restricción:** El firewall institucional bloquea las peticiones salientes y entrantes directas desde plataformas en la nube públicas como Make.com.
* **Limitación de Herramientas:** Para saltear este bloqueo, es mandatorio utilizar la instancia interna de **n8n** ya desplegada en los servidores locales (la cual cuenta con las reglas de red necesarias).
* **Gobernanza de TI (Restricción de Software):** Al tratarse de una instalación de **n8n Community Edition (Gratuita)**, la herramienta carece de un sistema de permisos granular por usuario. Otorgar acceso para este desarrollo implicaba exponer credenciales globales de otras integraciones críticas en producción, lo que generó un rechazo por parte del equipo de seguridad informática (IT).

---

## 3. 📈 Próximos Pasos & Hoja de Ruta
El proyecto se reanudará bajo la siguiente estrategia de mitigación de arquitectura:

1.  **Workflow delegativo en n8n:** Reformular la solicitud al equipo de IT para que implementen el pipeline diseñado directamente como un sub-proceso (*workflow* cerrado) dentro de la instancia existente, consumiendo los endpoints expuestos de Notion sin otorgar acceso directo a la consola de control.
