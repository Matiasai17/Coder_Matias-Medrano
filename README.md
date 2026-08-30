# Ecosistema de Automatización IA Autónomo

### Gestión inteligente de leads · Vector Consultores

**Matías Medrano** · Módulo 8 — Proyecto Integrador · Curso de IA Automation

---

## Qué es esto

Un sistema que recibe consultas comerciales, las clasifica con inteligencia artificial, redacta una respuesta y la envía al cliente — con una persona aprobando antes de cada envío.

El caso es una consultora ficticia de auditoría y optimización de procesos, con un volumen de referencia de 1.000 consultas mensuales. El sistema resuelve el proceso de punta a punta: desde que llega el mensaje hasta que sale la respuesta, pasando por la clasificación, el ruteo por segmento y el registro de todo lo que ocurrió.

**El principio de diseño:** autónomo en el procesamiento, supervisado en la interacción con el exterior. Todo lo que pasa puertas adentro ocurre sin intervención. Nada sale hacia un cliente sin que alguien lo autorice.

---

## Enlaces

| Recurso | Enlace |
|---|---|
| **Panel de control operativo** (público) | https://matiasai17.github.io/Coder_Matias-Medrano/ |
| **Base de datos** (modo lectura) | https://airtable.com/appmVz5S3bfSSMfgU/shrOlTWeS1sUlkFAx |
| **Video demostración** (3 min) | https://www.loom.com/share/b96f19a2b693477d8ce0d01e8f62aa51 |

---

## Entregables por criterio

| # | Criterio | Peso | Documento |
|---|---|---|---|
| 1 | Mapa de Arquitectura del Sistema | 20% | [Diagrama-Arquitectura.pdf](01-arquitectura/Diagrama-Arquitectura.pdf) |
| 2 | Manual Operativo de Estructuras de Datos | 20% | [Manual-Operativo-Datos.pdf](02-manual-datos/Manual-Operativo-Datos.pdf) |
| 3 | Estrategia de Optimización de Costos | 20% | [Matriz-Optimizacion-Costos.pdf](03-costos/Matriz-Optimizacion-Costos.pdf) |
| 4 | Malla de Seguridad, Privacidad y Resiliencia | 20% | [Malla-Seguridad-Resiliencia.pdf](04-seguridad-resiliencia/Malla-Seguridad-Resiliencia.pdf) |
| 5 | Dashboard de Control Ejecutivo | 20% | [Panel público](https://matiasai17.github.io/Coder_Matias-Medrano/) |

---

## Stack técnico

| Categoría | Herramienta | Rol |
|---|---|---|
| Orquestador | **n8n** | Flujo principal, sub-flujo de errores y proceso mensual por lotes |
| Base de datos | **Airtable** (`VECTOR_OPS`) | Memoria relacional: 7 tablas vinculadas |
| Procesamiento IA | **GPT-4o-mini** + **Claude** | Clasificación económica y análisis de lectura densa |
| Canales de salida | **Gmail** · **Slack** · WhatsApp API *(diseñado, no implementado)* | Respuesta al cliente, aprobación interna |

> WhatsApp está completamente especificado — plantilla, esquema JSON y punto HITL de resguardo — en el criterio 2 y en el diagrama de arquitectura. No se implementó en esta demo porque el enunciado exige un solo canal de salida y Gmail + Slack ya lo cubren; agregar WhatsApp real requiere alta de número en Meta Business, un trámite ajeno al alcance del proyecto.

---

## Cómo funciona

```
Consulta entrante
  → normalización y validación
  → registro en Airtable
  → clasificación con GPT-4o-mini (segmento, urgencia, score)
  → ruteo según segmento
        VIP        → análisis denso con Claude + borrador de propuesta
        Estándar   → respuesta breve
        Descartado → archivado con motivo
  → SOLICITUD DE APROBACIÓN EN SLACK  ← el sistema se detiene acá
  → decisión humana
        Aprobado  → envío por Gmail en el hilo original
        Rechazado → registro del motivo, sin contacto externo
  → auditoría de tokens, costos y tiempos
  → panel de control actualizado
```

El diagrama completo, con los nombres de todos los nodos y las rutas de error, está en el PDF del criterio 1.

---

## Decisiones que vale la pena señalar

**Dos modelos, no uno.** Clasificar mil mensajes cortos y redactar una propuesta a partir de un pliego son problemas distintos. El primero va a GPT-4o-mini; el segundo a Claude. Sumado a Prompt Caching y a la API de lotes para el informe mensual, el ahorro medido es del **60,4%** contra la línea de base. El cálculo completo está en el criterio 3.

**Ningún dato hardcodeado.** Umbrales, modelos, canales y tiempos de espera viven en la tabla `CFG_Config`; los prompts en `PRM_Prompts`, versionados. Cambiar de proveedor de IA es editar dos celdas.

**El error se registra siempre, se alerta solo si importa.** Los fallos se clasifican en cinco tipos y quedan en `LOG_Errores` con el payload anonimizado. La alerta se reserva para lo que requiere acción inmediata, porque un canal que notifica todo se ignora en una semana.

**El punto de control humano está en el último paso reversible.** El borrador ya existe y puede evaluarse, pero todavía no salió nada. Un paso antes el revisor vería datos crudos; un paso después estaría leyendo lo que el cliente ya recibió.

---

## Estructura del repositorio

```
├── index.html                      Panel de control (GitHub Pages)
├── 01-arquitectura/                Diagrama en PDF vectorial + fuente SVG
├── 02-manual-datos/
│   ├── Manual-Operativo-Datos.pdf
│   ├── esquema-relacional.png      Diagrama entidad-relación
│   ├── evidencia-omni-ai.png       Prompt a Omni AI antes de ejecutar
│   └── schemas/                    5 esquemas JSON de transferencia
├── 03-costos/                      Matriz de decisión y cálculo del ahorro
├── 04-seguridad-resiliencia/       Minimización, error handlers y HITL
├── flujo/                          Exportación .json de n8n (sanitizada)
└── evidencias/                     Capturas de las pruebas
```

---

## Nota sobre seguridad

Los archivos `.json` del flujo fueron sanitizados antes de subirse: no contienen claves de API, tokens ni URLs de webhook privadas. Las credenciales viven en el gestor de n8n y nunca salen de ahí.

El Base ID de Airtable aparece en la documentación de forma deliberada: identifica la base pero no da acceso. Lo que da acceso es la clave de API, y esa no está en ningún archivo de este repositorio.
