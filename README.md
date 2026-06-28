Plataforma de análisis de mercado y automatización para Vinted.** Detecta anuncios infravalorados en tiempo real y automatiza el reselling.

Producto:https://vinthook.com
Demo: <img width="1463" height="840" alt="vinthook-demo" src="https://github.com/user-attachments/assets/6950c31f-fcf1-45df-9752-c8540cd81c22" />

Qué es

VintHook escanea cada anuncio nuevo en menos de 300 ms y lo compara con la distribución histórica de precios para su marca, talla y condición. Lo que cae por debajo de la referencia IQR llega al instante a Telegram/Discord, con foto, precio y enlace directo de compra. El módulo AutoPilot opera 24/7 en servidor, sin depender del equipo del usuario, y en el plan Max la propia plataforma cierra las compras cualificadas de forma automática.

Qué hace

- Detección sub-300 ms. Indexa cada publicación nueva en menos de 300 milisegundos; la ventana entre publicación y compra es la variable clave en la reventa.
- Escaneo de 26 mercados en paralelo, con rotación de proxies integrada.
- Filtro estadístico IQR sobre histórico depurado para identificar precios atípicos (chollos reales).
- Alertas sub-segundo por Telegram y Discord.
- AutoPilot 24/7 como daemon en servidor, sin navegador ni sesión activa.
- AutoCompra y AutoLike (plan Max) con integración nativa en la cuenta de Vinted.

## Stack técnico

- Backend: Python, Flask, PostgreSQL
- Scraping / concurrencia: aiohttp, impersonación de navegador
- Automatización: Playwright (modo stealth, captura de sesión)
- IA: Google Gemini (salida estructurada JSON)
- Pagos: Stripe
- Email: Resend (transaccional) + MailerLite (marketing)
- Despliegue: Oracle Cloud (Linux, 24/7)

Retos técnicos resueltos

Captura de sesión frente a protección anti-bot. Vinted está protegido por sistemas tipo Datadome/Cloudflare. Resolví la captura y renovación de tokens de sesión mediante automatización con Playwright en modo stealth, manteniendo sesiones válidas de forma sostenida sin intervención manual.

Detección de outliers por IQR. En lugar de umbrales fijos, el motor de detección construye una distribución de precios por marca/talla/condición y aplica el rango intercuartílico para marcar como chollo solo lo que cae estadísticamente por debajo de lo esperado, reduciendo falsos positivos.

Servicio 24/7 resiliente. El AutoPilot corre como daemon en infraestructura propia (Oracle Cloud), diseñado para sobrevivir a reinicios y reanudar el escaneo automáticamente, garantizando cobertura continua sin depender del equipo del usuario.
