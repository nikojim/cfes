---


---

<h1 id="diccionario-visual-de-columnas-del-pipeline-de-categorización"><strong>Diccionario Visual de Columnas del Pipeline de Categorización</strong></h1>
<p>Este documento presenta, de forma visual y clara, <strong>todas las columnas generadas durante el pipeline de clasificación de líneas de facturas</strong>, ordenadas según su origen:</p>
<ul>
<li>
<p>Clustering,</p>
</li>
<li>
<p>LLM,</p>
</li>
<li>
<p>Snorkel,</p>
</li>
<li>
<p>Reglas de negocio,</p>
</li>
<li>
<p>Ajuste por factura,</p>
</li>
<li>
<p>Corrección manual,</p>
</li>
<li>
<p>Categoría final y trazabilidad.</p>
</li>
</ul>
<hr>
<h1 id="🧱-1.-identificación-y-texto-base">🧱 <strong>1. Identificación y Texto Base</strong></h1>
<p>Columna</p>
<p>Descripción</p>
<p>Ejemplo</p>
<p><strong><code>descripcion_limpia</code></strong></p>
<p>Texto limpio sin tildes, signos ni mayúsculas. Base para embeddings, clustering y reglas.</p>
<p><code>"gasoil"</code>, <code>"lavandina"</code></p>
<p><strong><code>desc_norm</code></strong></p>
<p>Versión altamente normalizada (remueve ruido, tokens triviales, etc.) usada por reglas.</p>
<p><code>"lavandina"</code></p>
<p><strong><code>nombre_comercial</code></strong></p>
<p>Nombre comercial del emisor tal como viene en la factura.</p>
<p><code>"FERRETERIA MARTÍN"</code></p>
<p><strong><code>nomcom_norm</code></strong></p>
<p>Nombre comercial normalizado para reglas y Snorkel.</p>
<p><code>"ferreteria martin"</code></p>
<p><strong><code>emisor</code> / <code>emisor_norm</code></strong></p>
<p>Identificador del proveedor + versión normalizada.</p>
<p><code>"ANCAP"</code> → <code>"ancap"</code></p>
<p><strong><code>id_factura</code></strong></p>
<p>Identificador único de la factura.</p>
<p><code>"A-1234-214567890012"</code></p>
<hr>
<h1 id="🔷-2.-clustering--llm">🔷 <strong>2. Clustering + LLM</strong></h1>
<p>Columna</p>
<p>Capa</p>
<p>Descripción</p>
<p>Ejemplo</p>
<p><strong><code>cluster_id</code></strong></p>
<p>K-Means</p>
<p>Grupo semántico asignado por clustering.</p>
<p><code>12</code></p>
<p><strong><code>categoria_llm_cluster</code></strong></p>
<p>LLM (1ª pasada)</p>
<p>Categoría propuesta por el modelo analizando 20 ejemplos representativos del cluster.</p>
<p><code>"Herramientas"</code></p>
<p><strong><code>justificacion_llm_cluster</code></strong></p>
<p>LLM</p>
<p>Explicación textual del modelo sobre por qué asigna la categoría.</p>
<p><code>"Todas las descripciones refieren a insumos de ferretería"</code></p>
<p><strong><code>cat_actual_auditor</code></strong></p>
<p>LLM Auditor</p>
<p>Categoría evaluada por el auditor LLM. Puede coincidir o no con la primera pasada.</p>
<p><code>"Herramientas"</code></p>
<p><strong><code>cat_sugerida_auditor</code></strong></p>
<p>LLM Auditor</p>
<p>Categoría alternativa sugerida si detecta problemas.</p>
<p><code>"Productos de limpieza"</code></p>
<p><strong><code>coherente</code></strong></p>
<p>LLM Auditor</p>
<p>Si el cluster es consistente semánticamente.</p>
<p><code>True</code> / <code>False</code></p>
<p><strong><code>alertas</code></strong></p>
<p>LLM Auditor</p>
<p>Observaciones de inconsistencias en el cluster.</p>
<p><code>"posible mezcla de rubros"</code></p>
<p><strong><code>cat_final_cluster</code></strong></p>
<p>Post LLM</p>
<p>Categoría final seleccionada a nivel cluster (tras auditoría).</p>
<p><code>"Herramientas"</code></p>
<hr>
<h1 id="🧪-3.-supervisión-débil-snorkel">🧪 <strong>3. Supervisión Débil (Snorkel)</strong></h1>
<p>Estas columnas surgen de combinar las señales débiles:</p>
<ul>
<li>
<p>reglas JSON por descripción,</p>
</li>
<li>
<p>reglas por emisor,</p>
</li>
<li>
<p>reglas por nombre comercial (<code>reglas_nomcom_snorkel.json</code>),</p>
</li>
<li>
<p>pistas del prompt,</p>
</li>
<li>
<p>categoría de cluster (<code>cat_final_cluster</code>).</p>
</li>
</ul>
<p>Columna</p>
<p>Descripción</p>
<p>Ejemplo</p>
<p><strong><code>cat_snorkel</code></strong></p>
<p>Categoría generada por el LabelModel de Snorkel, después de consolidar todas las señales débiles.</p>
<p><code>"Combustible"</code></p>
<p><strong><code>_cat_before_reglas</code></strong></p>
<p>Snapshot de categoría base antes de reglas tradicionales.</p>
<p><code>"Herramientas"</code></p>
<hr>
<h1 id="⚙️-4.-reglas-de-negocio-p3">⚙️ <strong>4. Reglas de Negocio (P3)</strong></h1>
<p>Columna</p>
<p>Descripción</p>
<p>Ejemplo</p>
<p><strong><code>cat_reglas</code></strong></p>
<p>Categoría resultante de aplicar reglas de negocio (<code>reglas_recategorizacion.json</code>).</p>
<p><code>"Publicidad y marketing"</code></p>
<p>Aplicación incluye:</p>
<ul>
<li>
<p>reglas por emisor,</p>
</li>
<li>
<p>reglas por descripción,</p>
</li>
<li>
<p>consolidación de categorías.</p>
</li>
</ul>
<hr>
<h1 id="🧾-5.-categoría-por-factura-p4">🧾 <strong>5. Categoría por Factura (P4)</strong></h1>
<p>Columna</p>
<p>Descripción</p>
<p>Ejemplo</p>
<p><strong><code>n_lineas_factura</code></strong></p>
<p>Cantidad total de líneas que tiene la factura.</p>
<p><code>7</code></p>
<p><strong><code>categoria_factura</code></strong></p>
<p>Categoría dominante dentro de la factura.</p>
<p><code>"Herramientas"</code></p>
<p><strong><code>ratio_cat_dominante</code></strong></p>
<p>Proporción de líneas que pertenecen a la categoría dominante.</p>
<p><code>0.86</code></p>
<p><strong><code>es_monotematica</code></strong></p>
<p>Si la factura es monotemática según las reglas del JSON.</p>
<p><code>True</code></p>
<hr>
<h1 id="🔧-6.-ajuste-por-factura-p5">🔧 <strong>6. Ajuste por Factura (P5)</strong></h1>
<p>Columna</p>
<p>Descripción</p>
<p>Ejemplo</p>
<p><strong><code>categoria_ajustada_factura</code></strong></p>
<p>Categoría propuesta en caso de factura monotemática (contexto superior).</p>
<p><code>"Herramientas"</code></p>
<p><strong><code>ajuste_por_factura</code></strong></p>
<p>Indica si se aplicó la corrección por factura.</p>
<p><code>True</code></p>
<hr>
<h1 id="✍️-7.-corrección-manual-p6–p7">✍️ <strong>7. Corrección Manual (P6–P7)</strong></h1>
<p>Columna</p>
<p>Descripción</p>
<p>Ejemplo</p>
<p><strong><code>cat_manual_linea</code></strong></p>
<p>Categoría corregida manualmente por un humano en el Excel de auditoría.</p>
<p><code>"Productos de limpieza"</code></p>
<p>Cuando existe, sobrescribe cualquier otra capa anterior.</p>
<hr>
<h1 id="🏁-8.-categoría-final--trazabilidad-p8">🏁 <strong>8. Categoría Final + Trazabilidad (P8)</strong></h1>
<p>Columna</p>
<p>Descripción</p>
<p>Ejemplo</p>
<p><strong><code>cat_final_linea</code></strong></p>
<p>Categoría final definitiva usada para EDA, reporting y entrenamiento.</p>
<p><code>"Herramientas"</code></p>
<p><strong><code>fuente_categoria_final</code></strong></p>
<p>Indica de dónde proviene la categoría final.</p>
<p><code>"snorkel+reglas_cluster"</code></p>
<p>Valores posibles:</p>
<ul>
<li>
<p><code>"correccion_manual_excel"</code></p>
</li>
<li>
<p><code>"ajuste_por_factura"</code></p>
</li>
<li>
<p><code>"snorkel+reglas_cluster"</code></p>
</li>
<li>
<p><code>"reglas_cluster"</code></p>
</li>
<li>
<p><code>"llm_cluster"</code> (si no hay Snorkel)</p>
</li>
</ul>
<p>Esto permite entender y auditar cómo se construyó cada etiqueta.</p>
<hr>
<h1 id="🧮-9.-columna-financiera">🧮 <strong>9. Columna financiera</strong></h1>
<p>Columna</p>
<p>Descripción</p>
<p><strong><code>monto_uyu</code></strong></p>
<p>Importe en pesos uruguayos de cada línea de factura. Permite análisis monetarios por categoría.</p>
<hr>
<h1 id="🧠-mapa-visual-de-cómo-se-forman-las-categorías">🧠 <strong>Mapa visual de cómo se forman las categorías</strong></h1>
<p><code>descripcion_limpia ↓ embedding → cluster_id → categoria_llm_cluster → auditoría LLM → cat_final_cluster ↓ reglas Snorkel + nom_comercial ↓ cat_snorkel ↓ reglas de negocio JSON → cat_reglas ↓ |-------------- contexto de factura --------------| ↓ categoria_factura → ajuste → categoria_ajustada_factura ↓ corrección manual Excel (opcional) ↓ cat_final_linea (resultado definitivo) ↓ fuente_categoria_final (trazabilidad)</code>.io/).</p>

