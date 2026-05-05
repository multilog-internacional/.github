# Metodología de desarrollo de scripts y automatizaciones

**Documento interno — Área de Datos**

Guía práctica para estandarizar el desarrollo diario de scripts de automatización y generación de reportes en el entorno DWH de Multilog.

| Campo | Detalle |
|---|---|
| Audiencia | Equipo de datos y automatización |
| Ámbito | Scripts Python, SQL, Power Automate, n8n, Task Scheduler |
| Entorno | Azure SQL (dev_bronze/silver/gold), servidor AWS EC2 Windows |
| Versión | 1.0 — 23/04/2026 |

---

## Filosofía: Data Engineering Lite

Las metodologías pesadas tipo Scrum formal o SAFe no se ajustan al trabajo de un equipo pequeño de datos, donde los requerimientos llegan fuera de ciclo y una tarea urgente puede interrumpir cualquier planeación quincenal. Pero la ausencia total de metodología lleva al caos técnico: scripts sin pruebas, credenciales hardcodeadas, cambios directos en producción, imposibilidad de auditar qué se hizo y cuándo.

La solución es un punto intermedio: **Kanban personal** para gestionar el trabajo sin ceremonias innecesarias, combinado con **prácticas de ingeniería de software** aplicadas con sentido común al mundo de datos. El proceso cabe en cinco fases que se repiten en ciclo, con una frontera clara entre el trabajo en desarrollo y el trabajo en producción.

### El principio que lo resume todo

Escribir código aburrido y predecible, seguir el mismo patrón en cada script, validar siempre contra una fuente de verdad, y jamás modificar código directamente en producción. La disciplina no es el enemigo de la velocidad: es lo que hace que la velocidad sea sostenible en el tiempo.

### Por qué estas cinco fases

Cada fase existe para resolver un problema específico que, si se omite, aparece tarde o temprano como incidente en producción:

| Fase | Problema que previene |
|---|---|
| 1. Planear | Empezar a codear sin saber exactamente qué se quiere hacer. |
| 2. Desarrollar | Scripts sin estructura común que nadie puede mantener. |
| 3. Validar | Código que corre sin errores pero produce números equivocados. |
| 4. Revisar | Cambios sin historial ni explicación que se pierden en el tiempo. |
| 5. Desplegar y operar | Fallas silenciosas en producción que nadie detecta a tiempo. |

---

## El ciclo de desarrollo

Una tarea avanza siempre en el mismo orden. Cada transición solo ocurre si se cumplen los criterios de la fase anterior. La retrospectiva semanal convierte lo aprendido en prod en mejoras al proceso.

```
[DEV]
  Planear  →  Desarrollar  →  Validar  →  Revisar y mergear
                                                   ↓
[PROD]                                    Desplegar y operar
                ↑                                  |
                └──────── Retro semanal ───────────┘
```

> **Nota sobre la frontera dev/prod.** El cambio entre la fase 4 y la 5 no es decorativo. Señala que se cruzó el límite entre trabajo experimental (donde romper algo es barato) y producción (donde un error puede afectar reportes, flujos o decisiones del negocio). El estándar de cuidado es distinto a cada lado de ese límite.

---

## 01 — Planear

> Trabajo en DEV

El 80% de los scripts que terminan siendo un problema no fallaron en la ejecución: fallaron porque nunca se definió bien qué tenían que hacer. La fase de planeación existe para que cuando te sientes a codear ya sepas exactamente qué estás resolviendo y por qué.

### Estructura del tablero Kanban

Cinco columnas, ni más ni menos. La herramienta concreta (GitHub Projects, Trello, Excel) importa menos que que el tablero sea el único lugar donde viven las tareas.

| Columna | Límite | Qué contiene |
|---|---|---|
| Backlog | Sin límite | Ideas, requerimientos y bugs sin refinar. |
| Próximo | 5–7 tarjetas | Tarjetas refinadas, listas para trabajarse. |
| En progreso | 2 tarjetas | Trabajo activo. WIP bajo a propósito. |
| En revisión | 2 tarjetas | Código esperando validación o PR. |
| Hecho | Sin límite | Mergeado y desplegado. Se archiva semanalmente. |

### Cómo priorizar

Cada tarjeta responde a dos preguntas: cuánto duele si no la hago esta semana (impacto) y cuánto me va a costar hacerla (esfuerzo en t-shirt sizes: XS a XL). Si estimas XL, la tarjeta todavía no está lista; hay que partirla. El orden de prioridad es: primero los XS/S de alto impacto (victorias rápidas), luego los M/L de alto impacto (el trabajo real), luego los XS/S de bajo impacto.

### Por qué el WIP de 2

El WIP bajo no es para trabajar menos, es para forzarte a terminar antes de empezar. Con cinco tareas en paralelo, cada una avanza al 20% y ninguna se entrega. Con dos, una termina esta semana y entrega valor. Además, un WIP bajo expone cuellos de botella: una tarjeta atorada cuatro días en revisión se vuelve visible en el tablero, en lugar de quedar oculta detrás de trabajo nuevo.

### Anatomía de una buena tarjeta (antes de pasar a Próximo)

- [ ] Título accionable (`"Agregar validación de duplicados"`, no `"Problema con ventas"`)
- [ ] Contexto en una frase: qué problema resuelve y para quién
- [ ] Criterio de aceptación explícito: cómo sabemos que está terminada
- [ ] Estimación t-shirt: XS, S, M o L (nunca XL sin partir)
- [ ] Ambiente objetivo: dev primero, prod después

---

## 02 — Desarrollar

> Trabajo en DEV

El objetivo no es escribir código brillante: es escribir código aburrido y predecible, que siga la misma estructura que todos los demás scripts del repo. En seis meses cualquiera, incluido tu yo futuro, debe poder leerlo sin contexto.

### Flujo de trabajo de Git

Trunk-based development: una sola rama `main` siempre desplegable, y ramas feature cortas que viven máximo unos días antes de mergearse.

```bash
git checkout main
git pull origin main
git checkout -b feature/validacion-duplicados-ventas
```

Convención de nombres: prefijo (`feature/`, `fix/`, `refactor/`, `docs/`) seguido de descripción corta en kebab-case. Minúsculas, sin acentos. **Regla de oro:** si la tarjeta del Kanban requiere dos ramas separadas, son dos tarjetas separadas.

### La plantilla como contrato

Para scripts nuevos, se copia `scripts/_template` y se adapta. La estructura es siempre la misma:

| Archivo | Responsabilidad |
|---|---|
| `README.md` | Propósito, insumos, salidas, frecuencia, cómo depurar |
| `config.py` | Carga de variables de entorno y validación dev/prod |
| `db.py` | Conexiones al DWH con context manager |
| `queries.py` | SQL parametrizado con `Template` (nunca concatenar) |
| `transforms.py` | Lógica de negocio pura, sin I/O (testeable) |
| `logger.py` | Logging a consola y archivo rotado por fecha |
| `main.py` | Orquestación con flags `--dry-run` y `--check` |
| `tests/` | Pruebas unitarias de `transforms.py` |

### El loop de desarrollo

Escribir una función pura, escribir su prueba unitaria antes de seguir, correr `pytest`, commitear. Repetir. Commits pequeños, uno por cada pieza lógica terminada. Si el mensaje de commit necesita la palabra "y", es que el commit es demasiado grande.

**Regla práctica:** conectar al mundo real (DB, APIs, archivos) solo cuando la lógica de `transforms.py` esté funcionando y probada con DataFrames inventados. Empezar por la conexión te condena a iteraciones de 10 segundos; empezar por la lógica pura te permite iterar en milisegundos.

### Un error frecuente que vale la pena nombrar

La tentación más fuerte es hacer el script "bonito" antes de que funcione: refactorizar clases, abstraer configuraciones, agregar opciones que nadie pidió. **Primero que funcione, luego que sea correcto, y solo al final que sea bonito.** La mayoría de scripts se quedan en "funciona y es correcto" y está perfectamente bien.

---

## 03 — Validar

> Trabajo en DEV

El código puede correr sin errores y seguir estando mal. Un script que tira un Excel con números equivocados es peor que uno que falla: el que falla te avisa, el que miente te hace tomar decisiones incorrectas. La validación es el gate que atrapa eso antes de que llegue al negocio.

### Cuatro niveles de validación

Se corren en orden, del más barato al más caro. Si uno falla, no tiene sentido pasar al siguiente hasta arreglarlo.

| Nivel | Qué valida | Cómo |
|---|---|---|
| Estilo y lint | Formato y convenciones | `black .` y `ruff check .` limpios. SQLFluff para SQL. |
| Pruebas unitarias | Funciones de `transforms.py` | `pytest` pasa en la raíz del repo completo. |
| Dry-run funcional | Integración end-to-end | `--dry-run` contra dev. Log coherente y conteos lógicos. |
| Contra fuente de verdad | Correctitud de los datos | Totales cuadran contra Tableau/AXIS/SAP con tolerancia documentada. |

### Validación contra fuente de verdad

Es la validación más importante y la que más se omite. La pregunta central: ¿los números de mi script cuadran con los números que la gente ve hoy? Antes de reemplazar un proceso existente, hay que demostrar equivalencia (o documentar que el nuevo arregla un bug real del viejo). Definir tolerancia (por ejemplo 1%) y documentar la comparación en el PR con números concretos.

### Checklist antes de abrir el pull request

- [ ] `black .` y `ruff check .` salen limpios
- [ ] `pytest` pasa en la raíz del repo completo
- [ ] SQL validado con SQLFluff si aplica
- [ ] `--check` confirma conectividad al DWH
- [ ] `--dry-run` corre sin errores, log coherente
- [ ] Conteos del log coinciden con queries manuales
- [ ] Validación contra fuente de verdad documentada con números
- [ ] El script es idempotente (corrido 2 veces, mismo resultado)
- [ ] README actualizado con cambios de comportamiento

---

## 04 — Revisar y mergear

> Trabajo en DEV

Aunque trabajes solo, el pull request importa. Cuando hay otro desarrollador, la revisión atrapa tus errores. Cuando estás solo, la estructura del PR es lo único que te atrapa a ti mismo. El principio de fondo: escribir y revisar son modos cognitivos incompatibles, y el PR es el mecanismo que te fuerza a cambiar de uno al otro.

### Dejar reposar el código

La regla más útil y la más difícil de adoptar: no abras el PR el mismo momento en que terminas de codear. Deja pasar unas horas, idealmente la noche entera. Con ojos nuevos vas a notar nombres confusos, lógica que parecía clara anoche y ahora no, comentarios obsoletos. Si no puedes esperar la noche, al menos 20–30 minutos de descanso antes de revisar.

### Anatomía de un buen PR

| Sección | Qué debe contener |
|---|---|
| Título | Accionable, con prefijo (`feat/fix/refactor`). Ejemplo: `"feat: detecta duplicados"`. |
| Qué cambia | Comportamiento antes vs. después, en prosa. No describir el código. |
| Por qué | Razón del cambio. Link a la tarjeta del Kanban. |
| Cómo probar | Pasos concretos que tú mismo ejecutaste al validar. |
| Riesgos | Qué podría salir mal, qué quedó fuera del alcance, qué decisiones tomaste. |

### Las cinco preguntas de auto-revisión

Revisando archivo por archivo en la interfaz de GitHub (no en el editor local, porque el editor oculta el diff):

1. **¿Hace lo que el PR dice que hace?** Cambios "de paso" que no están en la descripción son bandera roja: o se documentan o se sacan del PR.
2. **¿Los nombres comunican intención?** Variables como `df`, `data`, `temp`, y funciones como `process`, `handle`, `update` son opacas al releer.
3. **¿Qué pasa en el caso borde?** DataFrame vacío, columnas con nulos, fecha futura, todos los registros duplicados.
4. **¿Los logs bastan para diagnosticar a las 3 AM?** Si el error solo dice "falló", sin contexto ni traceback, es insuficiente.
5. **¿Hay código muerto, TODOs sin tarjeta, comentarios obsoletos?** O se limpia o se convierte en tarjeta del backlog.

### Antes de mergear: verificación final

```bash
git rebase main && pytest && python -m scripts.nombre.main --dry-run
```

Si alguno de esos tres falla, no se mergea. Se regresa a Desarrollar. No hay excepciones: `"es un arreglo menor, lo hago después del merge"` es exactamente el patrón que hace que `main` deje de ser desplegable.

Mergear con squash para mantener el historial de `main` limpio, y borrar la rama feature después del merge.

---

## 05 — Desplegar y operar

> Producción

Se cruza la frontera dev/prod. Hasta este momento, un error solo te afectaba a ti. A partir de aquí, un error puede reventar el reporte matutino del director, congelar facturación, o mandar un mensaje equivocado a un grupo de WhatsApp. El estándar de cuidado cambia y hay que interiorizarlo.

### Secuencia de despliegue

1. Conectarse por RDP al servidor DWH-Multilog.
2. `git pull origin main` en `C:\Scripts-Multilog`.
3. Si hay dependencias nuevas: activar venv y `pip install -r requirements.txt`.
4. Verificar que `.env` tenga `ENVIRONMENT=prod` (el incidente del flujo 0122 fue exactamente esto).
5. Correr `python -m scripts.nombre.main --check` para validar conectividad.
6. Correr `python -m scripts.nombre.main` completo, sin dry-run, con datos reales, monitoreando el log en vivo.
7. Programar en Task Scheduler o n8n. Zona horaria explícita `America/Mexico_City` (UTC-6).
8. Monitorear la primera ejecución programada en vivo.
9. Actualizar el README con la hora exacta, el nombre de la tarea y el usuario dueño.

### Monitoreo: los tres niveles

| Nivel | Cuándo | Para qué |
|---|---|---|
| Revisión matutina | Todos los días, primeros 10 min | Buscar `WARNING`/`ERROR` en logs de la noche. |
| Alertas activas | Solo scripts críticos | Correo, WhatsApp o Healthchecks.io cuando algo falla. |
| Revisión mensual | Primer lunes del mes | Tendencias, scripts lentos, rotación de logs, salud general. |

### Protocolo cuando algo falla

- **Estabilizar, no arreglar.** Si el reporte falló, la prioridad es dar salida (correr manualmente si hace falta), no debuggear.
- **Preservar la evidencia.** Copiar el log de la falla antes de modificar nada. Guardar archivos parciales y estado.
- **Investigar en dev, no en prod.** Replicar el caso localmente. Corregir, probar, abrir PR normal. La metodología aplica igual en caliente.
- **Rehabilitar y monitorear.** Una vez el fix esté en main y desplegado, observar la siguiente ejecución en vivo.
- **Postmortem corto.** `docs/postmortems/YYYY-MM-DD-nombre.md` con: qué pasó, impacto, causa raíz, acciones para evitar que se repita.

### La disciplina más importante de toda la metodología

> **Nunca modifiques código directamente en el servidor de producción. Jamás.** Ni un cambio de una línea, ni "solo para probar", ni "lo regreso en un minuto". El servidor de prod es un reflejo de `main`, no una fuente independiente. Todos los cambios pasan por: local → rama feature → validación → PR → merge → `git pull` en el servidor. Siempre. Sin esta disciplina, el resto de la metodología colapsa.

---

## Anti-patrones que vale la pena nombrar

Cada uno de estos patrones es la causa raíz de al menos un incidente real de producción. Reconocerlos por nombre hace más fácil evitarlos y corregir al compañero (o a uno mismo) cuando aparecen.

| Anti-patrón | Descripción |
|---|---|
| Código en el servidor | Modificar directamente en el servidor en lugar de pasar por git. Resulta en divergencia entre `main` y lo que realmente corre, imposible de auditar. |
| Credenciales hardcodeadas | Passwords, tokens o rutas específicas dentro del código. Deben vivir en `.env`, fuera del repo, cargadas por `config.py`. |
| Ambiente equivocado | Scripts apuntando a prod cuando deberían estar en dev o viceversa. Evitable con validación explícita de `ENVIRONMENT` al inicio de cada ejecución. |
| WIP ilimitado | Cinco tarjetas "en progreso" y ninguna terminada. Las prioridades cambian, el contexto se pierde, nadie entrega nada. |
| Tests de relleno | Pruebas que siempre pasan porque no prueban nada real. Peores que no tener pruebas, porque dan falsa sensación de seguridad. |
| Validación ajustada | Cambiar el `assert` para que la prueba pase en lugar de investigar por qué falló. La forma más común de silenciar bugs reales. |
| Print en producción | Usar `print()` en lugar de `logging`. Los prints se pierden cuando el script corre en background, dejándote sin evidencia cuando algo falla. |
| PR sin descripción | Pull request con título "cambios" y descripción vacía. Imposible de revisar, imposible de auditar en el futuro. |
| Arreglar y seguir | `"Es un arreglo menor, lo hago después del merge"`. El `main` deja de ser desplegable y la disciplina se rompe silenciosamente. |
| Logs sin conteos | Logs que solo dicen "proceso terminado" sin números. Impiden detectar tendencias (script cada vez más lento, conteos que bajan). |

---

## Las tres disciplinas no negociables

Si del documento completo hubiera que escoger solo tres reglas para cumplir religiosamente, serían estas. Todas las demás son consecuencia o aplicación de ellas.

### 1. Todos los cambios pasan por git, rama y PR

No hay cambio directo en el servidor. No hay commit directo a `main`. Sin esta disciplina, el código en GitHub y el que corre divergen, y la auditabilidad desaparece.

### 2. Separación estricta dev / prod

La variable `ENVIRONMENT` controla todo. Se valida explícitamente al inicio de cada script. Ningún script toca prod sin haber corrido exitosamente en dev primero.

### 3. Validar contra una fuente de verdad antes de publicar

No basta con que el código corra. Los números deben cuadrar, con tolerancia documentada, antes de que cualquier consumidor los use para decidir algo.

---

## Siguiente paso práctico

Este documento es una referencia, no un plan. Para volverlo operativo:

- Clonar el repositorio base con la plantilla `_template` y el ejemplo funcional.
- Copiar el tablero Kanban de 5 columnas en GitHub Projects (o la herramienta preferida).
- Migrar los scripts existentes de a uno, empezando por el más simple, para adaptarlos al estándar.
- Institucionalizar la retro de los viernes: 30 minutos, tres preguntas, tarjetas al backlog.
- Revisar este documento cada seis meses: la metodología también es código y merece mantenimiento.
