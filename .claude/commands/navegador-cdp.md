---
description: Controla un Chrome ya abierto vía CDP (Playwright MCP) cuando WebSearch/WebFetch no bastan
argument-hint: [qué hacer en el navegador]
creado: 2026-09-19
revisado: 2026-09-19
descartado: chrome-devtools-mcp (Google) — también conecta por CDP a un Chrome existente (cumple el criterio eliminatorio), pero orientado a depurar (red, performance, consola) en vez de conducir el navegador; Puppeteer MCP — deprecado/archivado; navegadores cloud (BrowserBase y similares) — lanzan su propio navegador, no el existente. Ver [[entorno]] y [[decisiones]].
---

Paso 3 del escalado: navegador real por CDP contra un Chrome que ya está corriendo. Nunca lo lanzas tú, ni headless. El servidor MCP es `playwright-cdp`, definido en `.mcp.json` de este proyecto.

0. Antes de preguntar nada, mira con Bash `echo $CDP_ENDPOINT` y `curl -s -m 5 <endpoint>/json/version`. Si `CDP_ENDPOINT` está definida y responde, esa es la máquina elegida por el usuario al arrancar la sesión: dilo en una línea y usa `playwright-cdp` sin preguntar. Si está definida y no responde, avisa y pide que arranque el Chrome (plantilla del `.bat` abajo).

1. Si `CDP_ENDPOINT` no está definida, pregunta qué Chrome usar, una sola vez y antes de tocar ninguna herramienta:
   - a) El anfitrión por defecto, en `192.168.1.5:9222`. Ya está configurado en `.mcp.json` (fallback si `CDP_ENDPOINT` no está puesto). No requiere nada más: usa directamente las herramientas de `playwright-cdp`.
   - b) Otra máquina. Pide la IP.

2. Si eligen (b), ten en cuenta la limitación real: Playwright MCP fija el `--cdp-endpoint` al arrancar el proceso y no se puede recablear en caliente dentro de esta misma sesión. El flujo correcto es:
   - Dale el `.bat` completo (ver plantilla abajo) con `IP_DE_ESA_MAQUINA` sustituida por la IP indicada, y avisa de que hay que ejecutarlo como administrador en esa máquina.
   - Espera a que confirmen que el Chrome de esa máquina está arrancado y el `netsh portproxy` respondiendo.
   - Dile que para conectar contra esa IP hace falta relanzar esta sesión de Claude Code con la variable de entorno puesta, por ejemplo: `CDP_ENDPOINT=http://IP_DE_ESA_MAQUINA:9222 claude`. No se puede seguir en la sesión actual apuntando a otra máquina.

3. Nunca almacenes la IP ni la configuración de "otra máquina" en memoria ni en el wiki. El único host persistente es el anfitrión por defecto, ya en `.mcp.json` y en `areas/entorno.md`.

4. Recuerda: Windows 10 Pro solo admite una sesión interactiva. Si el usuario se conecta por RDP a esa máquina, el Chrome de la sesión de consola deja de renderizar.

5. Reporta siempre contra qué máquina se resolvió la tarea.

## Plantilla del .bat

```
@echo off
net session >nul 2>&1
if errorlevel 1 (
  echo ERROR: ejecuta este fichero como administrador.
  pause
  exit /b 1
)
taskkill /f /im chrome.exe >nul 2>&1
timeout /t 2 >nul
set CHROME="C:\Program Files\Google\Chrome\Application\chrome.exe"
if not exist %CHROME% set CHROME="C:\Program Files (x86)\Google\Chrome\Application\chrome.exe"
start "" %CHROME% --remote-debugging-port=9222 --user-data-dir="C:\chrome-agente" --disable-background-timer-throttling
timeout /t 4 >nul
netsh interface portproxy delete v4tov4 listenaddress=IP_DE_ESA_MAQUINA listenport=9222 >nul 2>&1
netsh interface portproxy add v4tov4 listenaddress=IP_DE_ESA_MAQUINA listenport=9222 connectaddress=127.0.0.1 connectport=9222
echo.
netstat -an | findstr 9222
pause
```
