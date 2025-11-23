# Parking Ucaldas – Asistente virtual de parqueaderos

### n8n + Twilio + Google Gemini + VPS Hostinger (Docker)

Este repositorio contiene el flujo de automatización del asistente **Parking Ucaldas**, un chatbot que administra reservas de parqueaderos para estudiantes y vigilantes de la **Universidad de Caldas**, operando sobre:

* **n8n** corriendo en un **contenedor Docker**
* Desplegado en un **VPS de Hostinger**
* Integrado con **Twilio** para WhatsApp/SMS
* Impulsado por **Google Gemini** para inteligencia conversacional

---

# 🚀 1. Infraestructura General

Parking Ucaldas se ejecuta en un entorno de producción estable compuesto por:

### **✔️ VPS Hostinger**

* Sistema operativo: Ubuntu 22.04 (recomendado)
* n8n desplegado usando **Docker + Docker Compose**
* Puerto de n8n expuesto (generalmente 5678)
* Certificado SSL opcional con Cloudflare o Let’s Encrypt

### **✔️ Docker**

El sistema se ejecuta aislado en un contenedor:

Ejemplo típico del `docker-compose.yml` usado:

```
version: '3.3'
services:
  n8n:
    image: n8nio/n8n:latest
    restart: always
    ports:
      - 5678:5678
    volumes:
      - ~/.n8n:/home/node/.n8n
    environment:
      - N8N_BASIC_AUTH_ACTIVE=true
      - N8N_BASIC_AUTH_USER=admin
      - N8N_BASIC_AUTH_PASSWORD=superpassword
      - WEBHOOK_TUNNEL_URL=https://tudominio.com
```

### **✔️ Twilio Webhooks**

Twilio redirige los mensajes entrantes hacia el webhook HTTPS del VPS.

### **✔️ n8n → Gemini Integration**

El flujo usa el nodo oficial de LangChain para comunicarse con **Google Gemini (PaLM API)**.

---

# 🧩 2. Arquitectura del Sistema (Diagrama)

```
┌─────────────────┐      WhatsApp/SMS       ┌─────────────────────┐
│     Usuario      │ ─────────────────────▶ │       Twilio        │
└─────────────────┘                         └─────────┬───────────┘
                                                      │ Webhook
                                                      ▼
                                           ┌──────────────────────┐
                                           │   VPS Hostinger       │
                                           │ (Docker + n8n stack)  │
                                           └─────────┬────────────┘
                                                     │
                                                     ▼
                                          ┌────────────────────────┐
                                          │  n8n Workflow:          │
                                          │  Parking Ucaldas Bot    │
                                          ├────────────────────────┤
                                          │ Twilio Trigger          │
                                          │ LangChain Agent         │
                                          │ Gemini Chat Model       │
                                          │ Session Memory          │
                                          │ Twilio Sender           │
                                          └─────────┬──────────────┘
                                                    │
                                                    ▼
                                          ┌─────────────────────────┐
                                          │ Google Gemini (PaLM API)│
                                          └─────────────────────────┘
```

---

# 🔄 3. Diagrama del Flujo Conversacional

```
               ┌──────────────────┐
               │   Usuario         │
               └───────┬──────────┘
                       ▼
           ┌──────────────────────────┐
           │ Twilio Trigger (n8n)     │
           └───────┬──────────────────┘
                   ▼
      ┌──────────────────────────────┐
      │ LangChain AI Agent           │
      │ (con el mega prompt)         │
      └─────────┬────────────────────┘
                ▼
   ┌──────────────────────────────┐
   │ 1️⃣ Reserva                  │
   | 2️⃣ Cancelar                 │
   | 3️⃣ Validar placa            │
   └─────────┬────────────────────┘
             ▼
  ┌─────────────────────────────────┐
  │ Gemini: Genera la respuesta     │
  └─────────┬──────────────────────┘
            ▼
   ┌──────────────────────────────┐
   │  SendResult (Twilio Out)     │
   └─────────┬────────────────────┘
             ▼
       ┌─────────────┐
       │   Usuario    │
       └─────────────┘
```

---

# 📦 4. Estructura del Repositorio

```
/
├── ParkingUCaldas.json   → Flow completo exportado de n8n
├── README.md             → Este documento
└── docs/
    ├── arquitectura.png  → Diagrama sugerido
    ├── flujo.png
```

*Los diagramas puedes generarlos desde acá mismo si quieres que te los exporte como PNG.*

---

# ⚙️ 5. Configuración Detallada

### **n8n en Docker dentro de Hostinger**

Ventajas:

* Reinicio automático
* Aislamiento
* Fácil actualización de versiones
* Persistencia garantizada con bind volumes

### **Webhooks**

Asegura que:

* El VPS tenga dominio o subdominio apuntado
* El puerto 443 esté abierto
* Twilio pueda llegar al webhook

---

# 📡 6. Configuración de Twilio

Tu número de Twilio debe tener configurado:

* **Webhook de mensajes entrantes** (WhatsApp o SMS)
  → apuntando al webhook del nodo **Twilio Trigger**.

Ejemplo:

```
https://tudominio.com/webhook/3de7047f-7f7f-40c7-86b6-9891b3a60e59
```

---

# 🤖 7. Lógica del Bot (Reglas Clave)

Ya incluidas previamente en detalle:

* Mensaje inicial fijo
* Flujo guiado paso a paso
* Sedes con 100 cupos cada una
* Prefijos: C, D, G, M
* No memorias cross-day
* Reinicio diario automático
* Sin JSON/llaves/corchetes en salida

> OJO: El flujo actual **no tiene base de datos**, toda la lógica de disponibilidad la maneja Gemini según el prompt.
> Si quieres persistencia real, te la puedo montar con Supabase o MySQL.

---

# 📈 8. Monitoreo y Logs

### **n8n UI**

* Panel "Executions"
* Filtrar por error
* Ver entradas/salidas de cada nodo

### **Docker logs**

```
docker logs -f n8n
```

### **Hostinger VPS**

Opciones adicionales:

* Monitor de procesos (htop)
* Firewall UFW
* Fail2ban opcional

---

# 🚀 9. Despliegue / Actualización Rápida

Actualizar n8n:

```
docker pull n8nio/n8n:latest
docker compose down
docker compose up -d
```

El flujo funciona igual porque todo vive en:

```
~/.n8n
```

---

# 🙋 Autor / Créditos

Proyecto implementado y desplegado por:

**Gewralds Braook**
Software Developer (.NET) – Apasionado por el desarrollo
Basado en infraestructura propia con Docker + VPS Hostinger
---
