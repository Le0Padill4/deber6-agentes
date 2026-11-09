# Equipo de revisión

La tarea es revisar `usuarios.py` antes de abrir un pull request.

| Agente | Misión | Herramientas autorizadas | Entrada | Salida | Límites |
|---|---|---|---|---|---|
| Orquestador | Repartir la revisión y reunir los resultados | Solo delegar | Ruta `usuarios.py` | Tabla con hallazgos y casos de prueba | No modifica archivos |
| Seguridad | Buscar credenciales y entradas sin validar | Leer y buscar | Ruta `usuarios.py` | Línea, problema y severidad | Solo ese archivo; no escribe |
| Pruebas | Proponer tres pruebas faltantes | Leer | Ruta `usuarios.py` | Nombre del caso y qué verifica | Solo propone; no escribe pruebas |

## Flujo

```text
                 ┌─ Seguridad ───┐
usuarios.py → Orquestador        ├→ Tabla consolidada → Persona revisa y decide → Ejecución
                 └─ Pruebas ─────┘                    ↑ checkpoint humano
```

Los especialistas pueden trabajar en paralelo. Antes de aprobar, la persona compara cada línea con el código y revisa que las pruebas tengan sentido. Rechazaría líneas inventadas, resultados sin respaldo o una propuesta fuera del alcance.

Si un especialista no responde, entrega una lista vacía sin explicación, inventa una línea o ignora un límite, el orquestador debe pedir una corrección o reintentar y avisar a la persona si el problema continúa.

## Hallazgos consolidados

| Línea | Hallazgo | Severidad |
|---:|---|---|
| 10 | Clave de ejemplo escrita en el código. | Media |
| 11 | Contraseña de administrador escrita en el código. | Alta |
| 18 y 21 | El nombre se concatena en SQL sin validación ni parámetros. | Alta |
| 25–27 | Precio y porcentaje se usan sin validar sus valores. | Media |
| 30–35 | Nombre, edad y correo se guardan sin comprobar su validez. | Media |

## Tres pruebas propuestas

| Caso | Qué verifica |
|---|---|
| Nombre con comilla o texto de inyección | Que la búsqueda trate el nombre como dato y no cambie la consulta SQL. |
| Descuento fuera de 0 a 100 | Que se rechacen porcentajes negativos o mayores que 100. |
| Crear usuario con edad inválida y correo vacío | Que no se guarden datos inválidos. |

Son propuestas de comportamiento esperado; el archivo actual todavía no implementa esas validaciones.

## A2A

*Simplificación didáctica; no es JSON conforme a una versión específica de A2A.*

```json
{
  "nombre": "Especialista en pruebas",
  "descripcion": "Propone pruebas para un archivo de código.",
  "skill": "proponer-pruebas",
  "interfaz": "https://agente.ejemplo/a2a",
  "capacidades": {"streaming": true},
  "seguridad": {"autenticacion": "credencial del equipo autorizada", "transporte": "HTTPS"}
}
```

La tarjeta podría obtenerse en `https://agente.ejemplo/.well-known/agent-card.json` o desde un registro. El cliente verifica su origen, la skill, la interfaz y la autenticación requerida antes de enviar información. La tarjeta anuncia capacidades; conocerla no demuestra que el agente sea confiable.

```text
Obtener Agent Card → verificar skill, interfaz y seguridad → enviar mensaje
→ respuesta directa o Task → consultar o recibir estado → obtener resultado (Artifact)
```

En el camino simplificado una tarea puede pasar por `submitted → working → completed`. Si llega a `input-required`, el cliente pide a la persona los datos necesarios antes de continuar. Si aparece `failed`, muestra el error y permite decidir si se reintenta.

Frente al subagente local, el remoto necesita una interfaz de red, descubrimiento, permisos explícitos y un límite de confianza entre equipos. A2A permite delegar una revisión autónoma a otro agente. MCP, en cambio, permite acceder a herramientas o recursos, como un archivo o datos. La elección depende de la arquitectura y de la interfaz disponible.
