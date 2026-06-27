<img width="1920" height="1080" alt="Screenshot_2026-06-28_00_44_22" src="https://github.com/user-attachments/assets/3d0667ae-ee9f-469d-9cf3-98af42a35745" />


⚡ HACKING TEAM · COMUNIDAD DE HACKERS — UI Local para Ollama

─────────────────────────────────────────────────────────────────
1. CONTEXTO Y MOTIVACIÓN
─────────────────────────────────────────────────────────────────

En operaciones de seguridad (red team, pentest, análisis forense,
research de malware), el flujo de trabajo implica manejar datos
que NO deben salir del perímetro controlado del analista:

  • Credenciales, hashes, tokens de sesión
  • Código fuente propietario del cliente bajo NDA
  • Indicadores de compromiso (IOCs) de incidentes activos
  • Muestras de malware y dumps de memoria
  • Información personal identificable (PII) de víctimas o targets

Cualquier consulta que un analista haga a una API de LLM comercial
(OpenAI, Anthropic, Google) envía ese contexto a servidores de
terceros, donde puede ser:
  • Almacenado para reentrenamiento (dependiendo de los TOS)
  • Expuesto en caso de breach del proveedor
  • Sujeto a solicitudes legales de jurisdicciones extranjeras
  • Logged en sistemas que el analista no controla

La respuesta técnica obvia es ejecutar LLMs localmente. Ollama es
la opción madura para hacerlo en 2026: corre modelos GGUF
quantizados (Q4_K_M, Q5, Q6) en CPU y GPU, expone una API REST
compatible con OpenAI, y soporta toda la familia de modelos de
Meta (Llama 3.2/3.3), Alibaba (Qwen 2.5/3), Mistral, DeepSeek,
Phi-3, Gemma, etc.

Lo que faltaba era una interfaz web usable. Las opciones existentes
(Open WebUI, LibreChat, AnythingLLM) son pesadas (Docker, cientos
de MB, configs de base de datos) y no siempre encajan en un
entorno de pentest donde quieres desplegar rápido y moverte. Esta
UI resuelve eso en ~10 KB de Python.

─────────────────────────────────────────────────────────────────
2. STACK TÉCNICO
─────────────────────────────────────────────────────────────────

  Lenguaje backend:  Python 3.11+
  Framework web:     Flask 3.x
  Cliente HTTP:      urllib nativo (sin requests/httpx)
  Frontend:          HTML + CSS + vanilla JavaScript
  API backend:       Ollama (http://localhost:11434)
  Streaming:         NDJSON sobre HTTP/1.1 (newline-delimited JSON)
  Bind:              0.0.0.0:8080 (LAN-accesible)

Sin React, sin Vue, sin Tailwind, sin build step, sin webpack. El
HTML completo + CSS + JS van inline en un único string template
de Flask. Editable en 2 minutos con cualquier editor.

Dependencias (mínimas):
  $ pipx install flask

─────────────────────────────────────────────────────────────────
3. CARACTERÍSTICAS
─────────────────────────────────────────────────────────────────

  • Descubrimiento automático de modelos locales vía GET /api/tags
  • Listado en sidebar con nombre, tamaño y cuantización
  • Streaming token-a-token de las respuestas (SSE simulado con
    NDJSON, latencia < 50 ms en LAN)
  • Historial de conversación por sesión (en memoria del cliente)
  • Indicador de estado de Ollama (verde = OK, rojo = caído)
  • Banner identificativo "HACKING TEAM · COMUNIDAD DE HACKERS"
  • Tema cyberpunk: gradiente animado fucsia→morado→cian, efectos
    neón con box-shadow, glassmorphism con backdrop-filter
  • Animaciones CSS nativas (gradientShift, pulse)
  • Sin telemetría, sin cookies de tracking, sin analytics
  • Sin login (uso local — añadir auth si se expone a internet)

─────────────────────────────────────────────────────────────────
4. INSTALACIÓN
─────────────────────────────────────────────────────────────────

# Prerrequisitos: Ollama ya corriendo
$ ollama --version        # >= 0.3
$ ollama pull llama3.2    # o el modelo que prefieras

# Instalar Flask en un venv aislado
$ pipx install flask

# Crear el directorio y descargar app.py
$ mkdir -p ~/.local/share/ollama-ui
$ curl -L https://[repo]/app.py -o ~/.local/share/ollama-ui/app.py

# Lanzador en ~/go/bin/
$ cat > ~/go/bin/ollama-ui << 'EOF'
#!/usr/bin/env bash
# (lanzador con start/stop/status/restart/logs)
EOF
$ chmod +x ~/go/bin/ollama-ui

# Arrancar
$ ollama-ui start
✅ UI arrancada
    URL:  http://localhost:8080 ve a tu navegador y pones esto 

─────────────────────────────────────────────────────────────────
5. SEGURIDAD Y HARDENING
─────────────────────────────────────────────────────────────────

Por diseño la UI escucha en 0.0.0.0:8080 sin autenticación. Esto
es aceptable para uso en host local (127.0.0.1) o LAN privada,
pero NO para exposición directa a internet.

Opciones de endurecimiento según el caso de uso:

  • Bind a 127.0.0.1 únicamente: cambiar host="0.0.0.0" a
    host="127.0.0.1" en app.py
  • Reverse proxy con auth básica (nginx + htpasswd)
  • Tailscale/ZeroTier para acceso remoto autenticado punto a punto
  • Cloudflare Tunnel con Access policies
  • mTLS si lo sirves entre varios analistas en una LAN

Consideraciones de aislamiento:
  • Ollama corre con permisos del usuario que lo arrancó
  • La UI hereda esos mismos permisos
  • Los modelos GGUF se cachean en ~/.ollama/models
  • No hay cifrado en tránsito entre la UI y Ollama porque ambos
    están en la misma máquina

─────────────────────────────────────────────────────────────────
6. LIMITACIONES CONOCIDAS Y ROADMAP
─────────────────────────────────────────────────────────────────

  Limitaciones actuales:
  • Sin persistencia de conversaciones (viven en memoria de pestaña)
  • Sin system prompt editable desde UI
  • Sin selector de temperatura, top_p, max_tokens
  • Sin upload de imágenes (los modelos vision-capable no se usan)
  • Sin export de conversación a Markdown/PDF
  • Sin métricas (tokens/segundo, contexto usado)

  Roadmap plausible:
  • Persistencia SQLite de conversaciones
  • Panel de settings (temperatura, system prompt)
  • Code syntax highlighting (highlight.js inline)
  • Export a Markdown con timestamp
  • Autenticación opcional por API key
  • Multi-usuario con sesiones aisladas

─────────────────────────────────────────────────────────────────
7. LICENCIA Y CONTACTO
─────────────────────────────────────────────────────────────────

  Licencia: MIT 
  Repo:     [enlace]
  Autor:    By AnonSec777

⚡ HACKING TEAM · COMUNIDAD DE HACKERS
─────────────────────────────────────────────────────────────────

#Ollama #LLM #LocalAI #LLMLocal #PrivacyFirst #AIsecurity
#Cybersecurity #RedTeam #Pentesting #OSINT #BugBounty
#OpenSource #Flask #Python #SelfHosted #DataPrivacy
#HackingTeam #ComunidadDeHackers


