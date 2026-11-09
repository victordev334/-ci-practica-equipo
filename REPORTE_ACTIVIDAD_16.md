# Reporte técnico – Actividad 16
## Análisis y documentación de reportes generados por el pipeline CI/CD

## 1. Datos generales

| Campo | Valor |
|---|---|
| Proyecto | `-ci-practica-equipo` (repositorio `victordev334/-ci-practica-equipo`) |
| Lenguaje / framework de pruebas | Python 3.12 en GitHub Actions / pytest |
| Fecha de ejecución | 2026-11-09 |
| Pipeline ejecutado | `CI Pipeline` (`.github/workflows/ci.yml`), disparado con `push` |
| Job | `build` (ubuntu-latest) |

Pasos del pipeline: checkout → setup-python 3.12 → instalar dependencias → ejecutar pruebas → build (copia `calculadora.py` a `artefactos/`) → generar ZIP → subir artefacto.

## 2. Cambio realizado al workflow (pendiente de subir)

Se añadió, sin quitar ningún paso existente:
- Instalación de `pytest-cov`.
- `pytest -v --cov=calculadora --cov-report=xml --junitxml=resultados-pruebas.xml`, para medir cobertura y número de pruebas.
- Paso "Resumen de métricas" que escribe la tabla de cobertura en la pestaña **Summary** de la ejecución.
- Artefacto adicional `reportes-pruebas` (`resultados-pruebas.xml`, `coverage.xml`).
- `.gitignore`: se ignoran los archivos generados (`.coverage`, `coverage.xml`, `resultados-pruebas.xml`, `.pytest_cache`).

## 3. Métricas

### 3.1 Resultados REALES

**A) Última ejecución en GitHub Actions** (run `42719884193`, rama `Diego`, 2026-11-09, workflow original sin cobertura):

| Métrica | Valor |
|---|---|
| Build exitoso | Sí (`success`) |
| Tiempo total del job `build` | 7 s (23:22:12 → 23:22:19 UTC); la ejecución completa figura como 11 s |
| Artefactos | `build-python` (317 bytes) |

**B) Historial de ejecuciones del repositorio** (`gh run list`, 11 ejecuciones):

| Resultado | Cantidad |
|---|---|
| Exitosas | 9 |
| Fallidas | 2 |
| **Build Success Rate** | 9 / 11 = **81,8 %** |

**C) Ejecución local** (`pytest -v --cov=calculadora`, Python 3.14.2, mismo comando que el workflow):

| Métrica | Valor |
|---|---|
| Total de pruebas | 1 |
| Aprobadas | 1 |
| Fallidas | 0 |
| Cobertura | 100 % (2 de 2 sentencias en `calculadora.py`) |
| Tiempo de pytest | 0,04–0,05 s |

Prueba ejecutada: `test_calculadora.py::test_suma` (verifica `suma(2,3) == 5`).

### 3.2 Datos PENDIENTES (requieren subir el cambio y ejecutar el pipeline)

| Métrica | Dónde obtenerla | Valor |
|---|---|---|
| Total / aprobadas / fallidas en CI | Log del paso "Ejecutar pruebas" | _pendiente_ |
| Cobertura en CI | Pestaña Summary → "Métricas de pruebas" | _pendiente_ |
| Tiempo total de la nueva ejecución | Cabecera del run | _pendiente_ |
| Artefactos de la nueva ejecución | Sección Artifacts (`build-python`, `reportes-pruebas`) | _pendiente_ |
| Build Success Rate actualizado (con la nueva ejecución) | Pestaña Actions | _pendiente_ |

## 4. Bugs encontrados

Según el historial de git y de Actions:

| Bug | Evidencia | Corrección |
|---|---|---|
| `suma` restaba (`return a-b`) | Commit `551fb0f` "Fix suma function to perform addition"; el run `40417619921` falló | `return a+b` |
| Workflow inicial fallido | Run `40414967195` (falló en 0 s) con el commit `c26a793`; corregido en `719cd77` | Se corrigió el workflow |

Bugs abiertos en el código actual: ninguno detectado por las pruebas existentes (limitado a 1 prueba).
Nota: las causas de los dos fallos se infieren de los commits; no se revisó el log de cada uno.

## 5. Análisis de resultados

1. **¿El pipeline fue exitoso?** Sí. La última ejecución terminó en `success` con todos sus pasos en verde. En el historial, 9 de 11 ejecuciones fueron exitosas (81,8 %), y los 2 fallos son de los primeros commits y ya están corregidos.
2. **¿Qué pruebas fueron ejecutadas?** Una sola: `test_suma`, que comprueba que `suma(2,3)` devuelve 5.
3. **¿La cobertura es suficiente?** El 100 % es correcto en cifras, pero engaña: el código solo tiene una función y la prueba solo usa un caso. No hay pruebas con negativos, decimales, cero ni tipos inválidos. La cobertura de líneas no garantiza que se prueben casos límite.
4. **¿Qué mejora podría implementarse?**
   - Agregar más pruebas (`pytest.mark.parametrize` con negativos, cero y decimales) y más funciones de calculadora con sus pruebas.
   - Definir un umbral mínimo (`--cov-fail-under=80`) para que el pipeline falle si baja la cobertura.
   - Ejecutar también en `pull_request` y con varias versiones de Python (matriz).
   - Añadir un linter (`flake8` o `ruff`).
   - Añadir `requirements.txt` o fijar versiones de dependencias.

## 6. Capturas de pantalla a tomar en GitHub Actions

1. Pestaña **Actions** con la lista de ejecuciones del workflow `CI Pipeline` (muestra éxitos/fallos y el Build Success Rate).
2. Detalle de la ejecución nueva: estado `Success`, duración total y rama.
3. Job `build` con todos los pasos en verde (incluido "Resumen de métricas").
4. Log expandido del paso **Ejecutar pruebas** (`1 passed`, tabla de cobertura).
5. Pestaña **Summary** de la ejecución con la sección "Métricas de pruebas".
6. Sección **Artifacts** con `build-python` y `reportes-pruebas`.
7. Una ejecución fallida del historial (run `40417619921`), para el apartado de bugs.

## 7. Comparación entre equipos

Se llena solo con datos reales de cada equipo. Los valores de los demás equipos quedan vacíos hasta que los entreguen.

| Equipo | Build Success Rate | Total pruebas | Cobertura | Tiempo total | Bugs encontrados |
|---|---|---|---|---|---|
| A | _pendiente_ | _pendiente_ | _pendiente_ | _pendiente_ | _pendiente_ |
| B | _pendiente_ | _pendiente_ | _pendiente_ | _pendiente_ | _pendiente_ |
| C | _pendiente_ | _pendiente_ | _pendiente_ | _pendiente_ | _pendiente_ |
| D | _pendiente_ | _pendiente_ | _pendiente_ | _pendiente_ | _pendiente_ |

Este repositorio (`victordev334/-ci-practica-equipo`): 81,8 % / 1 prueba / 100 % (local) / 7 s el job / 1 bug de código (`suma`). Se debe asignar a la fila del equipo que corresponda.

## 8. Interpretación de incidentes

### Caso A: Build Success Rate = 100 %, cobertura = 20 %
- **Significa:** el pipeline nunca falla, pero solo se prueba una quinta parte del código.
- **Conclusión:** el verde es engañoso. Si las pruebas son pocas, el pipeline pasa aunque haya errores en el 80 % que no se prueba. Hay que aumentar las pruebas y poner un umbral mínimo de cobertura.

### Caso B: Cobertura = 95 %, bugs detectados = 15
- **Significa:** casi todo el código se ejecuta en las pruebas y esas pruebas encuentran muchos errores.
- **Conclusión:** las pruebas son efectivas, pero la calidad del código es baja. Hay que corregir los bugs y revisar el diseño o la revisión de código. Conviene ver si los 15 bugs están concentrados en un módulo.

### Caso C: Build Success Rate = 50 %
- **Significa:** la mitad de las ejecuciones fallan.
- **Conclusión:** el pipeline o el código son inestables. Pueden ser commits sin probar localmente, un workflow mal configurado, pruebas inestables o dependencias. Hay que revisar los logs de las ejecuciones fallidas, encontrar la causa más repetida y ejecutar las pruebas antes de hacer push. Es parecido a lo ocurrido en las primeras ejecuciones de este repositorio.

## 9. Resumen

- Real: pipeline exitoso, 9/11 ejecuciones exitosas (81,8 %), 1 prueba aprobada, 100 % de cobertura (local), 7 s de job, artefacto `build-python`.
- Pendiente: confirmar en Actions los valores de cobertura, pruebas y artefacto `reportes-pruebas` tras subir el cambio, y rellenar los datos de los otros equipos.
