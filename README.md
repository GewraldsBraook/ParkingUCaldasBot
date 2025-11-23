# Parking Ucaldas – Asistente virtual de parqueaderos

### n8n + Twilio + Google Gemini + Docker en VPS Hostinger

Este repositorio contiene el flujo de automatización de **Parking Ucaldas**, el asistente virtual de los parqueaderos de la **Universidad de Caldas**, implementado en **n8n**, ejecutado dentro de un **contenedor Docker** en un **VPS de Hostinger**, integrado con **Twilio** y potenciado por **Google Gemini**.

Parking Ucaldas permite:

* Realizar **reservas de puestos** (Estudiantes 📚)
* **Cancelar reservas** (Estudiantes 🙅)
* **Validar placas** (Vigilantes 👮)

La interacción se da vía **WhatsApp/SMS** usando Twilio, y la lógica de conversación está controlada por un **AI Agent** con **Gemini**, usando memoria por sesión.

---

# 1. Funcionalidad General

El asistente guía a estudiantes y vigilantes según la opción seleccionada:

1️⃣ **Reserva de puesto**
2️⃣ **Cancelar reserva**
3️⃣ **Validar placa**

El mensaje inicial SIEMPRE es:

```
¡Hola! 👋 Soy Parking Ucaldas, tu asistente virtual para los parqueaderos de la U. Estoy aquí para ayudarte a parquear fácil 🏍️🚗
¿Eres estudiante o vigilante? Dime qué necesitas hacer:
1️⃣ Reserva de puesto (Para estudiantes 📚)
2️⃣ Cancelar reserva (Para estudiantes 🙅)
3️⃣ Validar placa (Para vigilantes de turno 👮)
¡Escribe el número de la opción! 👇
```

### Sedes y cupos

Cada sede tiene **100 puestos**, con prefijo:

* Central → C001 – C100
* Derecho → D001 – D100
* Agropecuarias → G001 – G100
* Medicina → M001 – M100

### Reglas principales

* Reservas **solo para el día actual**
* Cada estudiante solo puede tener **una reserva activa por día**
* Reinicio diario **00:00 América/Bogotá**
* Placas normalizadas a **mayúsculas + sin espacios**
* Respuestas **solo en texto plano**, sin JSON, sin listas, sin llaves, sin corchetes

---

# 2. Arquitectura del Sistema

Parking Ucaldas está desplegado sobre:

### ✔️ VPS Hostinger

* Ubuntu 22.04 recomendado
* Docker + Docker Compose
* n8n corriendo como servicio
* Webhook público HTTPS para Twilio

### ✔️ Docker

El contenedor ejecuta n8n con persistencia en:

```
~/.n8n
```

### ✔️ Twilio

* Número de WhatsApp o SMS
* Webhook → n8n

### ✔️ Google Gemini

* PaLM API key
* Modelo conectado a través del nodo LangChain oficial

---

# 3. Diagrama de Arquitectura

```
┌─────────────────┐      WhatsApp/SMS        ┌─────────────────────┐
│     Usuario      │ ──────────────────────▶ │        Twilio        │
└─────────────────┘                          └──────────┬──────────┘
                                                       │ Webhook
                                                       ▼
                                        ┌────────────────────────────────┐
                                        │   VPS Hostinger (Ubuntu)       │
                                        │  Docker + Docker Compose       │
                                        └───────────┬────────────────────┘
                                                    │
                                                    ▼
                                  ┌────────────────────────────────────┐
                                  │             n8n Workflow            │
                                  │  ─ Twilio Trigger                   │
                                  │  ─ LangChain AI Agent               │
                                  │  ─ Gemini Chat Model                │
                                  │  ─ Simple Memory (por sesión)       │
                                  │  ─ Twilio SendResult                │
                                  └───────────┬────────────────────────┘
                                              │
                                              ▼
                                ┌───────────────────────────────────────┐
                                │        Google Gemini (PaLM API)       │
                                └───────────────────────────────────────┘
```

---

# 4. Diagrama del Flujo Conversacional

```
Usuario
   │
   ▼
Twilio Trigger (n8n)
   │ Body, From, To
   ▼
AI Agent (Prompt Parking Ucaldas)
   │ ├─ Usa memoria por número
   │ └─ Pide datos paso a paso
   ▼
Gemini (Genera respuesta)
   │
   ▼
SendResult (Twilio)
   │
   ▼
Usuario recibe respuesta
```

---

# 5. Requisitos Previos

Antes de importar el flujo necesitas:

* VPS Hostinger con Docker instalado
* n8n corriendo en un contenedor
* Número de Twilio habilitado
* API Key de Google Gemini
* Archivo `ParkingUCaldas.json` (incluido en este repo)

---

# 6. Importar el flujo en n8n

1. Abre n8n en tu VPS → `https://tu-dominio.com`
2. Ve a **Workflows → Import**
3. Selecciona el archivo `ParkingUCaldas.json`
4. Activa el workflow
5. En Twilio configura el webhook de mensajes entrantes con la URL del nodo **Twilio Trigger**

Ejemplo:

```
https://tudominio.com/webhook/3de7047f-7f7f-40c7-86b6-9891b3a60e59
```

---

# 7. Configuración de Credenciales

### 7.1 Twilio

Debe existir una credencial con nombre:

```
Twilio account
```

Contiene:

* Account SID
* Auth Token

### 7.2 Google Gemini (PaLM)

Credencial usada por:

```
Gemini Chat Model
```

Contiene:

* API Key de PaLM / Gemini

---

# 8. Detalle de Nodos del Workflow

### ✔️ Twilio Trigger

Recibe mensajes entrantes → pasa Body, From y To al agente.

### ✔️ AI Agent (LangChain)

* Prompt completo de Parking Ucaldas
* Lógica conversacional
* Maneja reserva, cancelación y validación
* Se conecta a:

  * Gemini (modelo)
  * Simple Memory (contexto)

### ✔️ Gemini Chat Model

Genera las respuestas del asistente.

### ✔️ Simple Memory

Memoria por número (`sessionKey = From`)
Contexto de 30 intervenciones.

### ✔️ SendResult

Envía la respuesta al usuario usando Twilio.

---

# 9. Flujo Funcional

### 9.1 Reserva (Estudiantes)

El bot solicita:

1. Código
2. Placa
3. Horario
4. Sede

Luego asigna:

* Primer puesto libre
* En sede solicitada
* Para la franja indicada

Y responde con:

* Sede
* Puesto asignado
* Cupos restantes
* Mensaje amigable con emojis

---

### 9.2 Cancelar reserva

Solicita:

* Código
* Placa

Y devuelve:

* Puesto liberado
* Sede
* Mensaje de confirmación

---

### 9.3 Validar placa

Solicita:

* Placa del vehículo

Y responde con:

* Si la reserva está activa
* Sede
* Horario
* Código de estudiante
* Puesto asignado
* Estado del cupo

---

# 10. Infraestructura (Hostinger + Docker)

### Status del contenedor

```
docker ps
```

### Logs del bot

```
docker logs -f n8n
```

### Reinicio rápido

```
docker compose down
docker compose up -d
```

---

# 11. Autor / Créditos

Proyecto creado e implementado por:

**Gewralds Braook**
Software Developer(.NET) · IA Enthusiast
Infraestructura propia → Docker + n8n + Hostinger VPS
---
