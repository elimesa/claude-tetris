---
name: clima
description: Obtiene la información del clima local (o de cualquier ciudad) sin necesidad de API key. Usa cuando el usuario pida "el clima", "qué tiempo hace", "temperatura", "pronóstico", "va a llover", o invoque /clima. Consulta wttr.in vía curl.
---

# Clima

Consulta el clima usando [wttr.in](https://wttr.in), un servicio gratuito que responde vía `curl` **sin API key**.

## Cómo obtener el clima

1. **Ubicación**: si el usuario da una ciudad, úsala. Si no da ninguna, deja la ubicación vacía (`wttr.in` detecta la ubicación por la IP de salida) o pregunta si hay ambigüedad.

2. **Consulta rápida (una línea)** — ideal para "¿qué clima hace?":
   ```bash
   curl -s --max-time 10 "wttr.in/{CIUDAD}?format=3&lang=es"
   ```
   Ejemplo de salida: `Bogota: ☁️ +16°C`

3. **Resumen del día (actual + condiciones)**:
   ```bash
   curl -s --max-time 10 "wttr.in/{CIUDAD}?format=%l:+%c+%t+(sensacion+%f),+humedad+%h,+viento+%w&lang=es"
   ```

4. **Pronóstico completo (hoy + 2 días, panel ASCII)**:
   ```bash
   curl -s --max-time 12 "wttr.in/{CIUDAD}?lang=es&m"
   ```
   El flag `m` fuerza unidades métricas (°C, km/h).

## Notas

- **Ciudad con espacios**: reemplaza espacios por `+` o `%20` (ej. `wttr.in/San+Francisco`).
- **Sin ciudad**: `curl -s "wttr.in/?format=3"` usa la ubicación por IP.
- **Siempre** usa `--max-time` para no colgar la sesión si el servicio no responde.
- Responde al usuario en **español** con un resumen claro (temperatura, condición y, si es relevante, sensación térmica / lluvia). No pegues el panel ASCII completo salvo que pidan el pronóstico detallado.
- Si `curl` falla o no hay red, avísale al usuario en vez de inventar datos.

## Formatos útiles de `format=`

| Código | Significado          |
|--------|----------------------|
| `%l`   | ubicación            |
| `%c`   | condición (emoji)    |
| `%t`   | temperatura          |
| `%f`   | sensación térmica    |
| `%h`   | humedad              |
| `%w`   | viento               |
| `%p`   | precipitación        |
