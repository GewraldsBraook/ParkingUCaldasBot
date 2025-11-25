# 🚗 Parking UCaldas

### Asistente Virtual para Reservas de Parqueaderos

**n8n + Twilio + Google Gemini + Docker en VPS Hostinger**

Parking UCaldas es un asistente virtual inteligente que permite realizar reservas de parqueaderos en la **Universidad de Caldas** mediante WhatsApp.
El sistema automatiza la interacción entre estudiantes y vigilantes, utilizando **IA conversacional, automatización y contenedores Docker**.

---

## ✨ ¿Qué hace Parking UCaldas?

👨‍🎓 **Estudiantes**

* Reservar puesto
* Cancelar reserva

👮 **Vigilantes**

* Validar placa

---

## 🚀 Tecnologías Principales

* **n8n** → Motor de automatización del flujo
* **Twilio** → Entrada/salida de mensajes (WhatsApp / SMS)
* **Google Gemini** → Modelo de IA responsable de la conversación
* **LangChain** → Framework del agente conversacional
* **Docker** → Contenedor para n8n en producción
* **Hostinger VPS** → Infraestructura del sistema

---

## 🔧 Funcionamiento General

El usuario envía un mensaje por WhatsApp/SMS. Twilio lo envía al webhook de **n8n**, donde un **AI Agent** con Gemini procesa el mensaje, sigue las reglas del flujo y responde nuevamente vía Twilio.

### 👋 Mensaje inicial (siempre):

```
¡Hola! 👋 Soy Parking Ucaldas, tu asistente virtual para los parqueaderos de la U. Estoy aquí para ayudarte a parquear fácil 🏍️🚗
¿Eres estudiante o vigilante? Dime qué necesitas hacer:
1️⃣ Reserva de puesto (Para estudiantes 📚)
2️⃣ Cancelar reserva (Para estudiantes 🙅)
3️⃣ Validar placa (Para vigilantes de turno 👮)
¡Escribe el número de la opción! 👇
```

---

## 🗺️ Sedes y Cupos

Cada sede tiene **100 puestos** con códigos únicos:

| Sede             | Rango       |
| ---------------- | ----------- |
| 🟦 Central       | C001 – C100 |
| 🟩 Derecho       | D001 – D100 |
| 🟧 Agropecuarias | G001 – G100 |
| 🟥 Medicina      | M001 – M100 |

---

## 📌 Reglas del Sistema

* Reservas **solo para hoy**
* 1 reserva activa por estudiante
* Reinicio automático: **00:00 (América/Bogotá)**
* Placas normalizadas
* Respuestas en **texto plano** (sin JSON ni listas técnicas)

---

# 🏗️ Arquitectura del Sistema

```
Usuario (WhatsApp/SMS)
        │
        ▼
      Twilio
        │ Webhook
        ▼
+-----------------------------+
|    VPS Hostinger (Docker)   |
|     └── n8n Workflow        |
|         ├ Twilio Trigger    |
|         ├ LangChain Agent   |
|         ├ Gemini Model      |
|         └ Twilio Response   |
+-----------------------------+
        │
        ▼
   Google Gemini
```

---

## 💬 Flujo Conversacional

```
Usuario → Twilio → n8n (AI Agent) → Gemini → n8n → Twilio → Usuario
```

Memoria por sesión → `From`
Contexto de hasta **30 mensajes**.

---

# ⚙️ Instalación y Despliegue

## 1️⃣ Requisitos

* VPS con Ubuntu 22.04
* Docker + Docker Compose
* n8n en contenedor
* Twilio configurado
* API Key de Gemini
* Archivo `ParkingUCaldas.json`

---

## 2️⃣ Importar el workflow

1. Abrir n8n → `https://tu-dominio.com`
2. **Workflows → Import**
3. Cargar `ParkingUCaldas.json`
4. Activar workflow
5. Configurar webhook en Twilio con la URL del nodo Trigger

Ejemplo:

```
https://tudominio.com/webhook/3de7047f-7f7f-40c7-86b6-9891b3a60e59
```

---

# 🧩 Componentes del Workflow

### 🟦 Twilio Trigger

Recibe Body, From y To.

### 🟧 LangChain AI Agent

* Prompt completo de Parking Ucaldas
* Lógica conversacional
* Manejo de memoria
* Reglas del flujo

### 🟩 Gemini Chat Model

Genera texto final.

### 🟨 Simple Memory

Memoria por número → persistencia por sesión.

### 🟥 Twilio SendResult

Envía la respuesta al usuario.

---

# 🛠️ Comandos de Infraestructura

Estado del contenedor:

```
docker ps
```

Ver logs:

```
docker logs -f n8n
```

Reiniciar servicio:

```
docker compose down
docker compose up -d
```

---

# 🌟 Funcionalidades del Bot

## 🧑‍🎓 1. Reservas

Solicita:

* Código
* Placa
* Horario
* Sede

Asigna automáticamente el **primer puesto disponible**.

---

## ❌ 2. Cancelar reserva

Solicita:

* Código
* Placa

Devuelve:

* Puesto liberado
* Sede
* Confirmación

---

## 👮 3. Validar placa

Recibe placa y responde con:

* Estado de la reserva
* Sede
* Código del estudiante
* Horario
* Puesto asignado

---

# 👨‍💻 Autor

**Gewralds Braook**
Software Developer (.NET) · IA Enthusiast
Infraestructura: Docker + n8n + Twilio + Gemini + Hostinger
