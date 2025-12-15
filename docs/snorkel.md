---


---

<h1 id="snorkel-en-el-pipeline-de-categorización">Snorkel en el pipeline de categorización</h1>
<h3 id="supervisión-débil-para-consolidar-señales-heterogéneas-de-clasificación"><em>Supervisión débil para consolidar señales heterogéneas de clasificación</em></h3>
<hr>
<h2 id="problema-que-snorkel-resuelve-en-este-pipeline">1. Problema que Snorkel resuelve en este pipeline</h2>
<p>En este pipeline <strong>no existe una única fuente confiable de verdad</strong> para categorizar líneas de factura. En su lugar, tenemos múltiples <strong>señales imperfectas</strong>:</p>
<ul>
<li>
<p>Categoría inferida por cluster (<code>cat_final_cluster</code>), proveniente de LLM</p>
</li>
<li>
<p>Reglas heurísticas por descripción (<code>reglas_recategorizacion.json</code>)</p>
</li>
<li>
<p>Reglas por proveedor (<code>emisor</code>, <code>nombre_comercial</code>)</p>
</li>
<li>
<p>Pistas semánticas (keywords, bigramas, n-gramas)</p>
</li>
<li>
<p>Contexto local del negocio</p>
</li>
<li>
<p>Contexto de la factura (monotematicidad)</p>
</li>
</ul>
<p>Cada una de estas señales:</p>
<ul>
<li>
<p><strong>puede fallar en casos específicos</strong>,</p>
</li>
<li>
<p><strong>tiene distinto nivel de confiabilidad</strong>,</p>
</li>
<li>
<p><strong>entra en conflicto con otras señales</strong>.</p>
</li>
</ul>
<p>Snorkel se utiliza para <strong>combinar todas estas señales de manera estadística</strong>, evitando:</p>
<ul>
<li>
<p>reglas rígidas hardcodeadas,</p>
</li>
<li>
<p>decisiones arbitrarias,</p>
</li>
<li>
<p>dependencia excesiva del LLM.</p>
</li>
</ul>
<hr>
<h2 id="¿qué-es-snorkel-conceptualmente">2. ¿Qué es Snorkel conceptualmente?</h2>
<p>Snorkel implementa un enfoque de <strong>supervisión débil (weak supervision)</strong>:</p>
<ul>
<li>
<p>En lugar de entrenar con etiquetas manuales,</p>
</li>
<li>
<p>se definen <strong>funciones de etiquetado (Labeling Functions, LFs)</strong>,</p>
</li>
<li>
<p>cada LF puede:</p>
<ul>
<li>
<p>asignar una categoría,</p>
</li>
<li>
<p>o abstenerse (<code>ABSTAIN</code>).</p>
</li>
</ul>
</li>
</ul>
<p>Snorkel aprende:</p>
<ul>
<li>
<p>qué tan confiable es cada LF,</p>
</li>
<li>
<p>cómo resolver conflictos entre ellas,</p>
</li>
<li>
<p>cómo producir una <strong>etiqueta latente óptima</strong>.</p>
</li>
</ul>
<hr>
<h2 id="qué-son-las-labeling-functions-lfs-en-este-proyecto">3. Qué son las Labeling Functions (LFs) en este proyecto</h2>
<p>En este pipeline, una <strong>LF representa una heurística explícita</strong> del negocio o del dominio.</p>
<p>Ejemplos reales de LFs:</p>
<p>Tipo de LF</p>
<p>Ejemplo</p>
<p>Qué expresa</p>
<p>Por descripción</p>
<p><code>"gasoil" → Combustible</code></p>
<p>Heurística semántica</p>
<p>Por proveedor</p>
<p><code>"ANCAP" → Combustible</code></p>
<p>Conocimiento del rubro</p>
<p>Por cluster</p>
<p><code>cluster 12 → Herramientas</code></p>
<p>Consenso semántico</p>
<p>Por palabras clave</p>
<p><code>"lavandina" → Limpieza</code></p>
<p>Regla léxica</p>
<p>Por contexto</p>
<p><code>"comisión bancaria" → Servicios financieros</code></p>
<p>Regla contextual</p>
<p>Formalmente, cada LF es una función:</p>
<p><code>LF_i: X → {categoria_id, ABSTAIN}</code></p>
<p>Donde:</p>
<ul>
<li>
<p><code>X</code> es una fila del DataFrame (una línea de factura),</p>
</li>
<li>
<p>la LF <strong>NO siempre decide</strong> (muchas veces se abstiene).</p>
</li>
</ul>
<hr>
<h2 id="origen-de-las-lfs-en-tu-pipeline">4. Origen de las LFs en tu pipeline</h2>
<p>Las LFs <strong>NO están hardcodeadas</strong>, sino que se generan desde archivos externos:</p>
<h3 id="📁-reglas_recategorizacion.json">📁 <code>reglas_recategorizacion.json</code></h3>
<ul>
<li>
<p>reglas por descripción,</p>
</li>
<li>
<p>reglas por emisor,</p>
</li>
<li>
<p>consolidación de categorías.</p>
</li>
</ul>
<h3 id="📁-reglas_nomcom_snorkel.json">📁 <code>reglas_nomcom_snorkel.json</code></h3>
<ul>
<li>
<p>reglas basadas en <code>nombre_comercial</code>,</p>
</li>
<li>
<p>heurísticas del rubro del proveedor.</p>
</li>
</ul>
<h3 id="📁-señales-derivadas-del-pipeline">📁 Señales derivadas del pipeline</h3>
<ul>
<li>
<p><code>cat_final_cluster</code>,</p>
</li>
<li>
<p>reglas de contexto local,</p>
</li>
<li>
<p>pistas semánticas.</p>
</li>
</ul>
<p>Esto permite:</p>
<ul>
<li>
<p>versionar reglas,</p>
</li>
<li>
<p>auditar cambios,</p>
</li>
<li>
<p>ajustar heurísticas sin tocar código.</p>
</li>
</ul>
<hr>
<h2 id="cómo-funciona-snorkel-paso-a-paso-en-este-pipeline">5. Cómo funciona Snorkel paso a paso en este pipeline</h2>
<h3 id="paso-1-—-definición-de-espacio-de-etiquetas">Paso 1 — Definición de espacio de etiquetas</h3>
<p>Todas las categorías se mapean a IDs:</p>
<p><code>CATEGORIA2ID = { "Alimentos": 0, "Bebidas": 1, "Herramientas": 2, ... }</code></p>
<p>Snorkel trabaja <strong>siempre en el espacio de IDs</strong>, no de strings.</p>
<hr>
<h3 id="paso-2-—-aplicación-de-lfs">Paso 2 — Aplicación de LFs</h3>
<p>Se aplica cada LF sobre cada fila:</p>
<p><code>fila i: LF_1 → Herramientas LF_2 → ABSTAIN LF_3 → Combustible LF_4 → Herramientas</code></p>
<p>Esto genera una <strong>matriz L</strong> de tamaño:</p>
<p><code>(n_filas × n_LFs)</code></p>
<p>Ejemplo conceptual:</p>
<p>Fila</p>
<p>LF1</p>
<p>LF2</p>
<p>LF3</p>
<p>LF4</p>
<p>1</p>
<p>Herr</p>
<p>ABST</p>
<p>Herr</p>
<p>ABST</p>
<p>2</p>
<p>ABST</p>
<p>Limp</p>
<p>ABST</p>
<p>Limp</p>
<p>3</p>
<p>Comb</p>
<p>Comb</p>
<p>ABST</p>
<p>ABST</p>
<hr>
<h3 id="paso-3-—-entrenamiento-del-labelmodel">Paso 3 — Entrenamiento del LabelModel</h3>
<p>El <code>LabelModel</code> estima:</p>
<ul>
<li>
<p>la <strong>precisión de cada LF</strong>,</p>
</li>
<li>
<p>la <strong>correlación entre LFs</strong>,</p>
</li>
<li>
<p>cómo resolver conflictos.</p>
</li>
</ul>
<p>Conceptualmente aprende:</p>
<blockquote>
<p>“Cuando LF1 y LF4 coinciden, confío más que cuando solo decide LF3”.</p>
</blockquote>
<p>Esto se hace <strong>sin etiquetas reales</strong>, usando solo estadísticas internas.</p>
<hr>
<h3 id="paso-4-—-inferencia-de-etiqueta-latente">Paso 4 — Inferencia de etiqueta latente</h3>
<p>Para cada fila, el modelo infiere:</p>
<p><code>cat_snorkel = argmax P(y | L)</code></p>
<p>Es decir:</p>
<ul>
<li>
<p>la categoría más probable,</p>
</li>
<li>
<p>considerando todas las señales disponibles,</p>
</li>
<li>
<p>ponderadas por su confiabilidad aprendida.</p>
</li>
</ul>
<hr>
<h2 id="qué-aporta-snorkel-frente-a-reglas-tradicionales">6. Qué aporta Snorkel frente a reglas tradicionales</h2>
<h3 id="❌-reglas-tradicionales">❌ Reglas tradicionales</h3>
<ul>
<li>
<p>rígidas,</p>
</li>
<li>
<p>orden-dependientes,</p>
</li>
<li>
<p>difíciles de mantener,</p>
</li>
<li>
<p>explotan en conflictos.</p>
</li>
</ul>
<h3 id="✅-snorkel">✅ Snorkel</h3>
<ul>
<li>
<p>tolera contradicciones,</p>
</li>
<li>
<p>aprende qué regla falla más,</p>
</li>
<li>
<p>permite agregar reglas sin romper las existentes,</p>
</li>
<li>
<p>escala bien con complejidad creciente.</p>
</li>
</ul>
<p>Ejemplo real:</p>
<blockquote>
<p>Regla A dice “Bebidas”<br>
Regla B dice “Limpieza”<br>
Regla C dice “Herramientas”</p>
</blockquote>
<p>Snorkel no elige arbitrariamente:</p>
<ul>
<li>
<p>aprende cuál regla históricamente es más fiable,</p>
</li>
<li>
<p>produce una decisión estable.</p>
</li>
</ul>
<hr>
<h2 id="relación-entre-snorkel-y-llm">7. Relación entre Snorkel y LLM</h2>
<p>Snorkel <strong>NO reemplaza al LLM</strong>. Cumple un rol distinto:</p>
<p>LLM</p>
<p>Snorkel</p>
<p>Comprensión semántica profunda</p>
<p>Consolidación estadística</p>
<p>Costoso</p>
<p>Barato</p>
<p>Difícil de auditar</p>
<p>Totalmente auditable</p>
<p>Bueno para clusters</p>
<p>Excelente para líneas individuales</p>
<p>En este pipeline:</p>
<ul>
<li>
<p>el LLM <strong>propone estructura semántica</strong> (clusters),</p>
</li>
<li>
<p>Snorkel <strong>consolida decisiones línea por línea</strong>.</p>
</li>
</ul>
<hr>
<h2 id="salida-de-snorkel-en-el-pipeline">8. Salida de Snorkel en el pipeline</h2>
<p>La salida principal es:</p>
<p><code>cat_snorkel</code></p>
<p>Que luego pasa por:</p>
<ol>
<li>
<p>Reglas JSON (<code>cat_reglas</code>),</p>
</li>
<li>
<p>Ajuste por factura,</p>
</li>
<li>
<p>Corrección manual,</p>
</li>
<li>
<p>Categoría final (<code>cat_final_linea</code>).</p>
</li>
</ol>
<p>Snorkel es <strong>una capa intermedia</strong>, no el veredicto final.</p>
<hr>
<h2 id="cuándo-snorkel-es-especialmente-valioso">9. Cuándo Snorkel es especialmente valioso</h2>
<ul>
<li>
<p>Ambigüedad léxica (<code>agua</code> vs <code>aguarrás</code>),</p>
</li>
<li>
<p>Proveedores multirrubro,</p>
</li>
<li>
<p>Descripciones cortas o genéricas,</p>
</li>
<li>
<p>Conflictos entre reglas,</p>
</li>
<li>
<p>Reducción de errores sistemáticos del LLM.</p>
</li>
</ul>
<hr>
<h2 id="principio-clave-de-diseño">10. Principio clave de diseño</h2>
<blockquote>
<p><strong>Snorkel no decide “qué es verdad”, sino<br>
cómo combinar múltiples verdades parciales.</strong></p>
</blockquote>
<p>Esto lo hace ideal para dominios como facturación, donde:</p>
<ul>
<li>
<p>el contexto importa,</p>
</li>
<li>
<p>las reglas cambian,</p>
</li>
<li>
<p>y no existe ground truth perfecta.</p>
</li>
</ul>
<hr>
<h2 id="resumen-operativo">11. Resumen operativo</h2>
<ul>
<li>
<p>Las LFs representan conocimiento experto externalizado.</p>
</li>
<li>
<p>Snorkel aprende a confiar (o no) en cada una.</p>
</li>
<li>
<p>Produce una etiqueta robusta y explicable.</p>
</li>
<li>
<p>Permite evolucionar el sistema sin reescribir lógica.</p>
</li>
</ul>
<h3 id="ejemplo-práctico-cómo-snorkel-consolida-reglas-en-conflicto">Ejemplo práctico: cómo Snorkel consolida reglas en conflicto</h3>
<p>En este pipeline, Snorkel se utiliza para <strong>combinar múltiples señales de categorización imperfectas</strong> y producir una etiqueta robusta por línea de factura.</p>
<h3 id="caso-de-entrada">Caso de entrada</h3>
<p>Supongamos las siguientes líneas de factura:</p>
<p>id   descripcion_limpia  nombre_comercial categoria_llm_cluster</p>
<p>1  aguarras mineral  FERRETERIA SAN JOSE Bebidas</p>
<p>2 gasoil premium ANCAP DEL CENTRO Combustible<br>
3 lavandina jane SUPERMERCADO X Alimentos<br>
4 resma a4 75gr PAPELERIA CENTRAL Suministros de oficina</p>
<p>Observamos errores típicos:</p>
<ul>
<li>
<p><strong>Ambigüedad léxica</strong>: “aguarrás” contiene “agua”.</p>
</li>
<li>
<p><strong>Sesgo por emisor</strong>: supermercado → alimentos.</p>
</li>
<li>
<p><strong>Clasificación por cluster no siempre confiable</strong>.</p>
</li>
</ul>
<hr>
<h3 id="definición-del-espacio-de-etiquetas">Definición del espacio de etiquetas</h3>
<p>Snorkel opera sobre IDs numéricos:</p>
<p>`CATEGORIA2ID = { “Alimentos”: 0, “Bebidas”: 1, “Productos de limpieza”: 2, “Suministros de oficina”: 3, “Combustible”: 4, “Otros”: 5 }</p>
<p>ABSTAIN = -1`</p>
<hr>
<h3 id="labeling-functions-lfs">Labeling Functions (LFs)</h3>
<p>Cada LF representa una <strong>heurística explícita del dominio</strong>.</p>
<h4 id="lf-por-palabras-clave-limpieza">LF por palabras clave (limpieza)</h4>
<p><code>@labeling_function() def lf_limpieza(row): if "lavandina" in row.descripcion_limpia or "aguarras" in row.descripcion_limpia: return CATEGORIA2ID["Productos de limpieza"] return ABSTAIN</code></p>
<h4 id="lf-por-proveedor-ancap">LF por proveedor ANCAP</h4>
<p><code>@labeling_function() def lf_emisor_ancap(row): if "ANCAP" in row.nombre_comercial: return CATEGORIA2ID["Combustible"] return ABSTAIN</code></p>
<h4 id="lf-por-categoría-del-cluster-llm">LF por categoría del cluster (LLM)</h4>
<p><code>@labeling_function() def lf_cluster_llm(row): if row.categoria_llm_cluster in CATEGORIA2ID: return CATEGORIA2ID[row.categoria_llm_cluster] return ABSTAIN</code></p>
<hr>
<h3 id="matriz-de-etiquetas-generada">Matriz de etiquetas generada</h3>
<p>id lf_limpieza lf_emisor lf_cluster<br>
1 Limpieza ABSTAIN Bebidas  2<br>
ABSTAIN</p>
<p>Combustible</p>
<p>Combustible</p>
<p>3</p>
<p>Limpieza</p>
<p>ABSTAIN</p>
<p>Alimentos</p>
<p>4</p>
<p>ABSTAIN</p>
<p>ABSTAIN</p>
<p>Suministros</p>
<p>Esta matriz contiene:</p>
<ul>
<li>
<p>consensos,</p>
</li>
<li>
<p>conflictos,</p>
</li>
<li>
<p>abstenciones.</p>
</li>
</ul>
<hr>
<h3 id="entrenamiento-del-labelmodel">Entrenamiento del LabelModel</h3>
<p><code>label_model = LabelModel(cardinality=len(CATEGORIA2ID)) label_model.fit(L_train, n_epochs=500)</code></p>
<p>El modelo aprende:</p>
<ul>
<li>
<p>qué LFs son más confiables,</p>
</li>
<li>
<p>cuáles fallan en ciertos contextos,</p>
</li>
<li>
<p>cómo resolver conflictos de forma estadística.</p>
</li>
</ul>
<hr>
<h3 id="inferencia-final-de-snorkel">Inferencia final de Snorkel</h3>
<p>id</p>
<p>categoria_snorkel</p>
<p>1</p>
<p>Productos de limpieza</p>
<p>2</p>
<p>Combustible</p>
<p>3</p>
<p>Productos de limpieza</p>
<p>4</p>
<p>Suministros de oficina</p>
<p>Snorkel <strong>corrige errores sistemáticos del cluster y del LLM</strong>, sin reglas rígidas ni prioridades manuales.</p>
<hr>
<h3 id="rol-de-snorkel-en-el-pipeline">Rol de Snorkel en el pipeline</h3>
<p>Snorkel actúa como una <strong>capa de consolidación</strong>, no como decisión final:</p>
<p><code>LLM (cluster) → Snorkel → Reglas JSON → Ajuste por factura → Corrección manual → cat_final_linea</code></p>
<p>Su valor principal es:</p>
<ul>
<li>
<p>reducir errores,</p>
</li>
<li>
<p>estabilizar decisiones,</p>
</li>
<li>
<p>disminuir carga de corrección humana.</p>
</li>
</ul>

