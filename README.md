<div align="center">

<img src="assets/preview-hero.jpg" alt="VintHook" width="100%" />

# VintHook

**Plataforma full-stack + extensión de navegador para detectar chollos en Vinted en tiempo real.**
*Full-stack platform + browser extension that spots underpriced Vinted listings in real time.*

<p>
  <img src="https://img.shields.io/badge/Python-3.11-3776AB?style=for-the-badge&logo=python&logoColor=white&labelColor=13121f" alt="Python" />
  <img src="https://img.shields.io/badge/Flask-3.0-cdbdff?style=for-the-badge&logo=flask&logoColor=white&labelColor=13121f" alt="Flask" />
  <img src="https://img.shields.io/badge/PostgreSQL-JSONB-4169E1?style=for-the-badge&logo=postgresql&logoColor=white&labelColor=13121f" alt="PostgreSQL" />
  <img src="https://img.shields.io/badge/Chrome-Manifest%20V3-ffa9fc?style=for-the-badge&logo=googlechrome&logoColor=white&labelColor=13121f" alt="Chrome MV3" />
  <br/>
  <img src="https://img.shields.io/badge/Status-En%20producci%C3%B3n-2ea043?style=for-the-badge&labelColor=13121f" alt="Status" />
  <img src="https://img.shields.io/github/license/mirandahidalgopablo-lab/Proyecto-VintHook?style=for-the-badge&color=f0b0ff&labelColor=13121f" alt="License" />
  <img src="https://img.shields.io/github/last-commit/mirandahidalgopablo-lab/Proyecto-VintHook?style=for-the-badge&color=cdbdff&labelColor=13121f" alt="Last commit" />
</p>

<p>
  <a href="https://vinthook.com"><b>🌐 vinthook.com</b></a> ·
  <a href="https://mirandahidalgopablo-lab.github.io/Proyecto-VintHook/"><b>✨ Landing del proyecto</b></a> ·
  <a href="FichaTecnicaVintHook.html"><b>📄 Ficha técnica</b></a> ·
  <a href="DireccionVisualVintHook.html"><b>🎨 Dirección visual</b></a>
</p>

</div>

> En Vinted el chollo es de quien lo ve primero. **VintHook escanea cada anuncio nuevo de los 26 mercados de Vinted en menos de 300 ms**, lo compara con la mediana de precios de anuncios parecidos (filtro IQR) y te avisa al instante por Telegram o Discord, mientras un AutoPilot sigue trabajando 24/7 en el servidor.

---

## 🎬 Preview

https://github.com/user-attachments/assets/2ee3b712-361c-46db-94d0-4cc10daea48e

<table>
  <tr>
    <td width="50%"><img src="assets/preview-hero.jpg" alt="Landing" /></td>
    <td width="50%"><img src="assets/preview-alerts.jpg" alt="Alertas en vivo" /></td>
  </tr>
  <tr>
    <td align="center"><sub>Landing · vinthook.com</sub></td>
    <td align="center"><sub>Feed de alertas con descuento sobre la mediana</sub></td>
  </tr>
</table>

---

## ✨ Características

| | Feature | Detalle |
|---|---|---|
| ⚡ | **Tracking automatizado** | Búsqueda concurrente (`ThreadPool` + `aiohttp` / `curl_cffi`) en los 26 dominios de Vinted, con rate-limiting, reintentos y rotación de proxies. |
| 🎯 | **Filtros personalizados** | Marca, talla, estado, precio máximo y país. Matching difuso de títulos con `thefuzz`. |
| 📉 | **Detección de chollos real** | Precio de referencia por mediana con filtrado de outliers por **IQR**: solo avisa de lo que está por debajo de mercado. |
| 🔔 | **Alertas instantáneas** | Notificaciones con foto, precio y enlace directo vía **Telegram Bot API** y **Discord webhooks**. |
| 🤖 | **AutoPilot 24/7** | Daemon en servidor con presets guardados (JSONB): sigue buscando con el ordenador apagado. |
| 🧩 | **Extensión de navegador** | *VintHook Bridge* (Chrome, Manifest V3) conecta tu sesión de Vinted para publicar, editar y comprar desde la plataforma. |
| 🧠 | **IA integrada** | Google **Gemini** con salida JSON estructurada (búsqueda por imagen, análisis de anuncios). |
| 💳 | **SaaS completo** | Google OAuth 2.0, verificación por email, planes Free / Pro / Max con **Stripe** (checkout + webhooks firmados) y web multilingüe ES/EN. |

---

## 🏗️ Arquitectura

```mermaid
flowchart LR
    subgraph Client["🖥️ Cliente"]
        UI["Dashboard web<br/>Jinja2 + JS vanilla"]
        EXT["Extensión Chrome MV3<br/>VintHook Bridge"]
    end

    subgraph Server["☁️ Oracle Cloud · Docker"]
        API["Backend Flask<br/>Gunicorn gthread"]
        SCAN["Motor de escaneo<br/>ThreadPool + aiohttp / curl_cffi"]
        AP["AutoPilot daemon<br/>24/7"]
        PW["Playwright headless<br/>plan Max"]
    end

    DB[("PostgreSQL<br/>usuarios · configuracion JSONB<br/>autopilot_presets")]
    VINTED["Vinted<br/>26 mercados"]
    TG["Telegram Bot API"]
    DC["Discord Webhooks"]
    STRIPE["Stripe"]
    GEM["Google Gemini"]

    UI <-->|REST JSON| API
    EXT <-->|sesión Vinted · REST| API
    EXT <-->|content scripts| VINTED
    API <--> DB
    API --> SCAN
    AP --> SCAN
    SCAN <-->|catálogo| VINTED
    PW <--> VINTED
    SCAN -->|chollo detectado| TG
    SCAN -->|chollo detectado| DC
    STRIPE -->|webhook firmado| API
    API <--> GEM

    classDef core fill:#1f1e2b,stroke:#cdbdff,color:#e4e0f3;
    classDef ext fill:#13121f,stroke:#ffa9fc,color:#e4e0f3;
    class API,SCAN,AP,PW,UI,EXT core;
    class DB,VINTED,TG,DC,STRIPE,GEM ext;
    style Client fill:#13121f,stroke:#f0b0ff,color:#e4e0f3
    style Server fill:#13121f,stroke:#cdbdff,color:#e4e0f3
```

### Flujo de una alerta

```mermaid
sequenceDiagram
    autonumber
    participant V as Vinted
    participant S as Motor de escaneo
    participant DB as PostgreSQL
    participant N as Telegram / Discord
    S->>V: Consulta catálogo (filtros del usuario)
    V-->>S: Anuncios nuevos
    S->>DB: Histórico de precios comparables
    S->>S: Mediana + filtro IQR
    alt Precio por debajo de la referencia
        S->>N: Alerta con foto, precio y enlace
    else Precio normal
        S-->>S: Descartar
    end
```

---

## 🧰 Tech Stack

| Capa | Tecnologías |
|---|---|
| **Backend** | Python 3.11 · Flask 3 · Gunicorn (gthread) · Jinja2 · Whitenoise |
| **Base de datos** | PostgreSQL (psycopg2, pool de conexiones, JSONB) |
| **Scraping / concurrencia** | aiohttp · curl_cffi (impersonación de navegador) · ThreadPool · rotación de proxies |
| **Automatización** | Playwright + stealth · Xvfb · AutoPilot daemon |
| **Frontend** | HTML5 · CSS3 · JavaScript vanilla · diseño *Lumina Noir* (dark-first, glassmorphism) |
| **Extensión** | Chrome Extension Manifest V3 (service worker, content scripts, popup) |
| **Integraciones** | Telegram Bot API · Discord Webhooks · Stripe · Google OAuth 2.0 · Google Gemini |
| **DevOps** | Docker · GitHub Actions (deploy por SSH) · Oracle Cloud (Ubuntu ARM) |
| **Seguridad** | Cifrado de datos sensibles (`cryptography`/Fernet) · CSP · rate limiting · webhooks firmados |



<div align="center">

Hecho por **[Pablo Miranda Hidalgo](https://github.com/mirandahidalgopablo-lab)** · [LinkedIn](https://www.linkedin.com/in/pablo-miranda-hidalgo) · [vinthook.com](https://vinthook.com)

<sub>Distribuido bajo licencia MIT. VintHook no está afiliado a Vinted.</sub>

</div>
