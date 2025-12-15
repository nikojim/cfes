---


---

<h1 id="readme-técnico-—-pipeline-de-categorización-de-líneas-de-facturas-cae-uruguay">README técnico — Pipeline de categorización de líneas de facturas (CAE Uruguay)</h1>
<p>Este documento describe el pipeline de categorización a nivel técnico, orientado a equipos de datos. Se explica:</p>
<ul>
<li>cómo se puebla el DataFrame principal,</li>
<li>qué hace cada etapa algorítmica (K-Means, HDBSCAN, LLM, Snorkel, reglas JSON, ajuste por factura),</li>
<li>y qué significa cada columna relacionada con la categorización.</li>
</ul>
<hr>
<h2 id="vista-general-del-pipeline">1. Vista general del pipeline</h2>
<p>El sistema toma como entrada:</p>
<ul>
<li>XML CFE (DGI Uruguay),</li>
<li>un DataFrame con embeddings precalculados por línea (<code>embedding</code>),</li>
<li>tres archivos de configuración/reglas:
<ul>
<li><code>prompt_config_clusters.json</code></li>
<li><code>reglas_recategorizacion.json</code></li>
<li><code>reglas_nomcom_snorkel.json</code></li>
</ul>
</li>
</ul>
<p>El output principal es una columna final:</p>
<ul>
<li><code>cat_final_linea</code> — categoría final y auditada para cada línea de factura.</li>
</ul>
<p>Además, se generan varias columnas intermedias para trazabilidad y análisis.</p>
<hr>
<h2 id="modelo-de-datos-y-columnas-base">2. Modelo de datos y columnas base</h2>
<p>El DataFrame principal <code>df</code> se va poblando en distintas etapas:</p>
<h3 id="columnas-de-identificación-y-texto">2.1. Columnas de identificación y texto</h3>
<ul>
<li>
<p><code>serie_cfe</code><br>
Serie del comprobante fiscal electrónico.</p>
</li>
<li>
<p><code>nro_cfe</code><br>
Número correlativo del CFE.</p>
</li>
<li>
<p><code>emisor</code><br>
Identificador del emisor (puede ser RUT, nombre abreviado, etc.).</p>
</li>
<li>
<p><code>nombre_comercial</code> / <code>NomComercial</code><br>
Nombre comercial del emisor (texto libre).<br>
Se normaliza a <code>nomcom_norm</code> para reglas de Snorkel.</p>
</li>
<li>
<p><code>descripcion_limpia</code><br>
Descripción de la línea, normalizada:</p>
<ul>
<li>minúsculas,</li>
<li>sin tildes,</li>
<li>sin caracteres especiales.</li>
</ul>
</li>
<li>
<p><code>monto_uyu</code><br>
Importe en pesos uruguayos, luego de convertir desde USD si aplica.</p>
</li>
</ul>
<h3 id="embeddings">2.2. Embeddings</h3>
<ul>
<li>
<p><code>embedding</code><br>
Lista de floats (vector) precalculada con<br>
<code>sentence-transformers/paraphrase-multilingual-mpnet-base-v2</code>.</p>
</li>
<li>
<p><code>embedding_vec</code><br>
Versión de <code>embedding</code> convertida a <code>np.array</code> (para cálculo numérico).</p>
</li>
</ul>
<hr>
<h2 id="clustering-k-means--hdbscan">3. Clustering: K-Means + HDBSCAN</h2>
<h3 id="k-means-—-agrupación-primaria">3.1. K-Means — agrupación primaria</h3>
<p><strong>Objetivo:</strong> agrupar descripciones de líneas en <em>k</em> clusters de similar significado.</p>
<p>El algoritmo K-Means:</p>
<ol>
<li>Inicializa <em>k</em> centroides en el espacio de embeddings.</li>
<li>Iterativamente:
<ul>
<li>asigna cada punto al centroide más cercano (distancia Euclídea),</li>
<li>recalcula cada centroide como la media de los puntos asignados.</li>
</ul>
</li>
<li>Minimiza la <strong>inercia</strong> (suma de distancias cuadradas dentro del cluster).</li>
</ol>
<p>Se elige <em>k</em> usando el <strong>método del codo</strong>:</p>
<ul>
<li>se calcula inercia para varios valores de <em>k</em> (por ejemplo 10, 20, 30, 40…),</li>
<li>se grafica <em>k</em> vs inercia,</li>
<li>se elige el <em>k</em> donde la curva deja de decrecer fuertemente (el “codo”).</li>
</ul>
<p><strong>Columnas generadas:</strong></p>
<ul>
<li>
<p><code>cluster_id</code><br>
Id del cluster asignado a cada línea por K-Means.</p>
</li>
<li>
<p><code>cluster_centers_</code> (interno en el objeto KMeans)<br>
Usado para calcular distancias a centroide.</p>
</li>
</ul>
<h3 id="hdbscan-—-detección-de-outliers-dentro-de-cada-cluster">3.2. HDBSCAN — detección de outliers dentro de cada cluster</h3>
<p><strong>Objetivo:</strong> identificar puntos “ruidosos” o atípicos dentro de los clusters.</p>
<p>HDBSCAN (Hierarchical DBSCAN):</p>
<ul>
<li>es un algoritmo de clustering basado en densidad,</li>
<li>no requiere fijar <em>k</em>,</li>
<li>detecta puntos que están en regiones de baja densidad → outliers.</li>
</ul>
<p>En este pipeline se usa <strong>por cluster</strong>:</p>
<ol>
<li>Para cada <code>cluster_id</code>, se toma el subconjunto de embeddings.</li>
<li>Se corre HDBSCAN en ese subconjunto.</li>
<li>Los puntos marcados como ruido (label = -1) se flaggean como outliers.</li>
</ol>
<p><strong>Columnas/flags:</strong></p>
<ul>
<li><code>es_outlier_hdbscan</code> (opcional)<br>
Booleano que indica si la línea es un outlier dentro de su cluster.</li>
</ul>
<p>Estos outliers <strong>no se usan</strong> como ejemplos representativos para el LLM.</p>
<h3 id="selección-de-ejemplos-representativos">3.3. Selección de ejemplos representativos</h3>
<p>Para cada cluster:</p>
<ol>
<li>Se calcula la distancia al centroide K-Means.</li>
<li>Se seleccionan las ~20 descripciones más cercanas (no outliers).</li>
<li>Se construye una lista de ejemplos textuales + contexto para el LLM.</li>
</ol>
<p>No crea nuevas columnas, pero alimenta la fase de prompts.</p>
<hr>
<h2 id="llm-por-cluster-clasificación--auditoría">4. LLM por cluster: clasificación + auditoría</h2>
<h3 id="primera-pasada-—-clasificación-por-cluster">4.1. Primera pasada — Clasificación por cluster</h3>
<p>El LLM recibe, por cada cluster:</p>
<ul>
<li><code>cluster_id</code></li>
<li>~20 descripciones representativas (<code>descripcion_limpia</code>)</li>
<li>nombres comerciales relevantes</li>
<li><code>prompt_config_clusters.json</code>:
<ul>
<li><code>categorias_sugeridas</code></li>
<li><code>pistas_palabras_clave</code></li>
<li><code>reglas_semanticas</code></li>
<li><code>uso_nom_comercial</code></li>
<li><code>ejemplos_few_shot</code></li>
</ul>
</li>
</ul>
<p>El LLM devuelve, en formato JSON:</p>
<ul>
<li><code>cluster_id</code></li>
<li><code>categoria</code></li>
<li><code>justificacion</code></li>
</ul>
<p><strong>Columnas generadas:</strong></p>
<ul>
<li>
<p><code>categoria_llm_cluster</code><br>
Categoría propuesta por el LLM a nivel de cluster.</p>
</li>
<li>
<p><code>justificacion_llm_cluster</code><br>
Texto explicativo de por qué se eligió esa categoría.</p>
</li>
</ul>
<p>Estas se mergean a <code>df</code> usando <code>cluster_id</code>.</p>
<hr>
<h3 id="segunda-pasada-—-auditoría-llm">4.2. Segunda pasada — Auditoría LLM</h3>
<p>El LLM actúa como auditor del cluster:</p>
<ul>
<li>revisa la categoría actual,</li>
<li>analiza coherencia entre descripciones,</li>
<li>busca subgrupos con otro perfil,</li>
<li>aplica <code>reglas_semanticas</code>.</li>
</ul>
<p>Devuelve:</p>
<ul>
<li><code>cluster_id</code></li>
<li><code>cat_actual_auditor</code> (categoría que está evaluando)</li>
<li><code>cat_sugerida_auditor</code> (alguna alternativa si ve problemas)</li>
<li><code>coherente</code> (True/False)</li>
<li><code>alertas</code> (lista de strings con observaciones)</li>
</ul>
<p>Esto se almacena en una tabla <code>cluster_stats</code> que resume:</p>
<ul>
<li><code>cluster_id</code></li>
<li><code>cat_actual_auditor</code></li>
<li><code>cat_sugerida_auditor</code></li>
<li><code>coherente</code></li>
<li><code>alertas</code></li>
<li><code>n_lineas</code> (cantidad de líneas en el cluster)</li>
<li><code>pct_outliers</code> (si se calcula)</li>
</ul>
<hr>
<h2 id="post-pipeline-columnas-de-categorización-p1–p8--snorkel">5. Post-pipeline: columnas de categorización (P1–P8 + Snorkel)</h2>
<h3 id="p1-—-normalización">5.1. P1 — Normalización</h3>
<ul>
<li>
<p><code>desc_norm</code><br>
= versión de <code>descripcion_limpia</code> normalizada para reglas (regex, keywords).</p>
</li>
<li>
<p><code>emisor_norm</code><br>
= versión normalizada del nombre del emisor.</p>
</li>
<li>
<p><code>nomcom_norm</code><br>
= versión normalizada de <code>nombre_comercial</code>.</p>
</li>
</ul>
<hr>
<h3 id="p2-—-cat_final_cluster">5.2. P2 — <code>cat_final_cluster</code></h3>
<p>Se decide la categoría final del cluster usando la auditoría:</p>
<ul>
<li>Si <code>coherente == True</code> → <code>cat_final_cluster = cat_actual_auditor</code>.</li>
<li>Si <code>coherente == False</code> y hay <code>cat_sugerida_auditor</code> → se puede optar por usar la sugerida.</li>
<li>Si no hay información suficiente → se conserva <code>categoria_llm_cluster</code>.</li>
</ul>
<p><strong>Columna:</strong></p>
<ul>
<li><code>cat_final_cluster</code><br>
Categoría final por cluster, antes de pasar por Snorkel y reglas.</li>
</ul>
<hr>
<h2 id="snorkel-supervisión-débil-s0–s5">6. Snorkel: supervisión débil (S0–S5)</h2>
<p>Snorkel permite combinar <strong>múltiples reglas débiles</strong> (LFs) en una etiqueta única estadísticamente optimizada.</p>
<h3 id="espacio-de-etiquetas-s1">6.1. Espacio de etiquetas (S1)</h3>
<p>Se construyen:</p>
<ul>
<li>
<p><code>CATEGORIA2ID</code><br>
diccionario <code>{nombre_categoria → id entero}</code></p>
</li>
<li>
<p><code>ID2CATEGORIA</code><br>
diccionario inverso.</p>
</li>
</ul>
<hr>
<h3 id="labeling-functions-s2-s2b-s2c">6.2. Labeling Functions (S2, S2b, S2c)</h3>
<p>Se definen varias LFs independientes:</p>
<h4 id="a-lfs-desde-reglas_recategorizacion.json">a) LFs desde <code>reglas_recategorizacion.json</code></h4>
<ul>
<li>
<p><strong>Por emisor (<code>reglas_por_emisor</code>)</strong><br>
Miran <code>emisor_norm</code> y devuelven una categoría si se encuentra alguna palabra de <code>any_of</code>, opcionalmente condicionada a la categoría actual.</p>
</li>
<li>
<p><strong>Por descripción (<code>reglas</code>)</strong><br>
Miran <code>desc_norm</code> y votan por <code>target</code> según <code>any_of</code>, <code>whole_word</code>, etc.</p>
</li>
</ul>
<h4 id="b-lfs-desde-prompt_config_clusters.json-pistas_palabras_clave-s2b">b) LFs desde <code>prompt_config_clusters.json</code> (<code>pistas_palabras_clave</code>) (S2b)</h4>
<ul>
<li>Por cada categoría en <code>pistas_palabras_clave</code>, se crea una LF que mira <code>desc_norm</code> y vota por esa categoría si encuentra cualquiera de sus keywords.</li>
</ul>
<h4 id="c-lfs-desde-reglas_nomcom_snorkel.json-s2c">c) LFs desde <code>reglas_nomcom_snorkel.json</code> (S2c)</h4>
<ul>
<li>
<p>Cada entrada define:</p>
<ul>
<li><code>lf_name</code></li>
<li><code>keywords_upper</code> (substrings en <code>nomcom_norm</code>)</li>
<li><code>target_categoria</code></li>
</ul>
<p>La LF vota <code>target_categoria</code> si alguno de los keywords aparece en <code>nomcom_norm</code>.</p>
</li>
</ul>
<h4 id="d-lf-directa-de-cluster">d) LF directa de cluster</h4>
<ul>
<li><code>lf_cluster_direct</code><br>
Usa <code>cat_final_cluster</code> como voto directo si esa categoría está en <code>CATEGORIA2ID</code>.</li>
</ul>
<hr>
<h3 id="entrenamiento-del-labelmodel-s3">6.3. Entrenamiento del LabelModel (S3)</h3>
<p>Se aplica Snorkel:</p>
<ol>
<li>Se construye una matriz <code>L</code> (n_muestras × n_LFs) con los votos de cada LF:
<ul>
<li>valor ∈ <code>{0, 1, ..., NUM_CLASSES-1}</code> o <code>ABSTAIN</code> (sin voto).</li>
</ul>
</li>
<li>Se entrena un <code>LabelModel</code>:
<ul>
<li>estima la precisión / correlación entre LFs,</li>
<li>construye una distribución de probabilidad sobre clases por ejemplo.</li>
</ul>
</li>
<li>Se obtiene:
<ul>
<li><code>y_snorkel = label_model.predict(L)</code></li>
<li><code>y_probs = label_model.predict_proba(L)</code></li>
</ul>
</li>
</ol>
<p>Se mapean los labels de vuelta a texto:</p>
<ul>
<li><code>cat_snorkel</code><br>
categoría resultante de la combinación de todas las LFs.</li>
</ul>
<hr>
<h3 id="análisis-de-snorkel-s4-s5">6.4. Análisis de Snorkel (S4, S5)</h3>
<ul>
<li>Tablas de transición:
<ul>
<li><code>cat_final_cluster → cat_snorkel</code></li>
<li><code>cat_snorkel → cat_reglas</code> (una vez calculada)</li>
</ul>
</li>
<li>Ejemplos concretos donde:
<ul>
<li>Snorkel corrige a cluster,</li>
<li>reglas corrigen a Snorkel,</li>
<li>reglas se separan de cluster+Snorkel.</li>
</ul>
</li>
</ul>
<hr>
<h2 id="reglas-json-finales-p3-→-cat_reglas">7. Reglas JSON finales (P3) → <code>cat_reglas</code></h2>
<p>En P3 se aplican las reglas JSON clásicas sobre una columna base:</p>
<ul>
<li>si existe <code>cat_snorkel</code>, se usa como base (<code>CURRENT_CAT_COL = cat_snorkel</code>);</li>
<li>si no, <code>CURRENT_CAT_COL = cat_final_cluster</code>.</li>
</ul>
<p>Se genera:</p>
<ul>
<li>
<p><code>_cat_before_reglas</code><br>
copia de la categoría antes de reglas (para trazabilidad).</p>
</li>
<li>
<p><code>cat_reglas</code><br>
categoría después de aplicar:</p>
<ul>
<li><code>reglas_por_emisor</code>,</li>
<li><code>reglas</code> por descripción,</li>
<li><code>consolidacion_categorias</code>.</li>
</ul>
</li>
</ul>
<hr>
<h2 id="ajuste-por-factura-p4–p5">8. Ajuste por factura (P4–P5)</h2>
<h3 id="p4-—-agregación-por-factura">8.1. P4 — Agregación por factura</h3>
<p>Se arma un ID:</p>
<ul>
<li><code>id_factura = f"{serie_cfe}-{nro_cfe}-{emisor}"</code></li>
</ul>
<p>Se agrupa por <code>id_factura</code> a partir de <code>cat_reglas</code>:</p>
<ul>
<li><code>n_lineas_factura</code></li>
<li>conteo de categorías dentro de la factura,</li>
<li><code>categoria_factura</code> = categoría mayoritaria (modo),</li>
<li><code>ratio_cat_dominante</code> = frecuencia de la categoría mayoritaria / total</li>
<li><code>es_monotematica</code><br>
según <code>min_lineas_factura</code> y <code>umbral_ratio_monotematica</code> del JSON de configuración.</li>
</ul>
<p><strong>Columnas agregadas:</strong></p>
<ul>
<li><code>id_factura</code></li>
<li><code>categoria_factura</code></li>
<li><code>n_lineas_factura</code></li>
<li><code>ratio_cat_dominante</code></li>
<li><code>es_monotematica</code></li>
</ul>
<hr>
<h3 id="p5-—-ajuste-por-factura-y-categoría-final-inicial">8.2. P5 — Ajuste por factura y categoría final inicial</h3>
<p>Se aplica el <strong>ajuste contextual</strong>:</p>
<ul>
<li>Si la factura es claramente monotemática, el emisor no es multirrubro, y la categoría de línea es ambigua o distinta de <code>categoria_factura</code>,<br>
entonces se ajusta la línea a <code>categoria_factura</code>.</li>
</ul>
<p><strong>Columnas:</strong></p>
<ul>
<li>
<p><code>categoria_ajustada_factura</code><br>
categoría propuesta a partir del contexto de factura.</p>
</li>
<li>
<p><code>ajuste_por_factura</code><br>
booleano que indica si se aplicó ajuste o no.</p>
</li>
<li>
<p><code>cat_final_linea</code> (versión inicial)<br>
si hubo ajuste → <code>categoria_ajustada_factura</code>,<br>
si no → <code>cat_reglas</code>.</p>
</li>
</ul>
<hr>
<h2 id="auditoría-manual-en-excel-p6–p7">9. Auditoría manual en Excel (P6–P7)</h2>
<h3 id="p6-—-exportación">9.1. P6 — Exportación</h3>
<p>Se genera un Excel <code>auditoria_categorias_facturas.xlsx</code> con:</p>
<ul>
<li>Hoja <code>resumen_global</code><br>
métricas agregadas,</li>
<li>Hoja <code>transiciones</code><br>
matrices entre categorías,</li>
<li>Hoja <code>lineas_ajustadas</code><br>
casos donde hubo <code>ajuste_por_factura</code>,</li>
<li>Hoja <code>descripciones_problematicas</code><br>
con columnas:
<ul>
<li>identificadores,</li>
<li>descripción,</li>
<li>categorías en sus distintas versiones,</li>
<li>columna vacía <code>cat_manual_linea</code> para correcciones humanas,</li>
<li><code>comentario_manual</code>.</li>
</ul>
</li>
</ul>
<hr>
<h3 id="p7-—-reimportación">9.2. P7 — Reimportación</h3>
<p>El Excel se edita externamente y se vuelve a cargar. Se procesan las filas donde:</p>
<ul>
<li><code>cat_manual_linea</code> no está vacía.</li>
</ul>
<p>Para esas filas:</p>
<ul>
<li>se actualiza <code>df["cat_final_linea"] = cat_manual_linea</code>.</li>
</ul>
<p><strong>Columna adicional:</strong></p>
<ul>
<li><code>cat_manual_linea</code> (sólo en las filas corregidas).</li>
</ul>
<hr>
<h2 id="trazabilidad-de-la-categoría-final-p8">10. Trazabilidad de la categoría final (P8)</h2>
<p>Se genera una columna:</p>
<ul>
<li><code>fuente_categoria_final</code></li>
</ul>
<p>Reglas típicas:</p>
<ol>
<li>
<p>Si el índice de la línea aparece en la tabla de correcciones manuales →<br>
<code>"correccion_manual_excel"</code>.</p>
</li>
<li>
<p>Si <code>ajuste_por_factura == True</code> →<br>
<code>"ajuste_por_factura"</code>.</p>
</li>
<li>
<p>Si existe <code>cat_snorkel</code> →<br>
<code>"snorkel+reglas_cluster"</code>.</p>
</li>
<li>
<p>Si no →<br>
<code>"reglas_cluster"</code>.</p>
</li>
</ol>
<p>Esto permite:</p>
<ul>
<li>medir la confianza por tipo de origen,</li>
<li>filtrar ejemplos “de alta calidad” para entrenamiento supervisado,</li>
<li>hacer análisis posteriores por fuente.</li>
</ul>
<hr>
<h2 id="resumen-de-columnas-de-categorización">11. Resumen de columnas de categorización</h2>
<p>Listado rápido de todas las columnas relacionadas con la categorización, en orden lógico:</p>
<ol>
<li>
<p><code>categoria_llm_cluster</code><br>
→ 1ª pasada LLM por cluster.</p>
</li>
<li>
<p><code>justificacion_llm_cluster</code><br>
→ explicación del LLM.</p>
</li>
<li>
<p><code>cat_actual_auditor</code><br>
→ categoría que el auditor LLM evalúa (generalmente igual a <code>categoria_llm_cluster</code>).</p>
</li>
<li>
<p><code>cat_sugerida_auditor</code><br>
→ categoría alternativa sugerida por el auditor si detecta problemas.</p>
</li>
<li>
<p><code>coherente</code><br>
→ booleano: el cluster es coherente según el auditor.</p>
</li>
<li>
<p><code>alertas</code><br>
→ texto/lista indicando problemas detectados en el cluster.</p>
</li>
<li>
<p><code>cat_final_cluster</code><br>
→ categoría final escogida a nivel de cluster (tras auditoría).</p>
</li>
<li>
<p><code>cat_snorkel</code><br>
→ categoría generada por Snorkel a partir de todas las LFs (reglas débiles + cluster + rubro + palabras clave).</p>
</li>
<li>
<p><code>_cat_before_reglas</code><br>
→ copia de la categoría base antes de aplicar reglas JSON (para trazabilidad).</p>
</li>
<li>
<p><code>cat_reglas</code><br>
→ categoría después de aplicar <code>reglas_recategorizacion.json</code> (por emisor, por descripción, consolidación).</p>
</li>
<li>
<p><code>id_factura</code><br>
→ identificador único de factura.</p>
</li>
<li>
<p><code>categoria_factura</code><br>
→ categoría dominante dentro de la factura.</p>
</li>
<li>
<p><code>n_lineas_factura</code><br>
→ cantidad de líneas por factura.</p>
</li>
<li>
<p><code>ratio_cat_dominante</code><br>
→ proporción de la categoría dominante dentro de la factura.</p>
</li>
<li>
<p><code>es_monotematica</code><br>
→ booleano según criterios de configuración.</p>
</li>
<li>
<p><code>categoria_ajustada_factura</code><br>
→ categoría propuesta desde el contexto de factura para corregir líneas.</p>
</li>
<li>
<p><code>ajuste_por_factura</code><br>
→ booleano indicando si la línea fue modificada por regla de factura.</p>
</li>
<li>
<p><code>cat_manual_linea</code><br>
→ categoría corregida manualmente vía Excel (sólo en filas auditadas).</p>
</li>
<li>
<p><code>cat_final_linea</code><br>
→ categoría final definitiva luego de:</p>
<ul>
<li>clustering + LLM,</li>
<li>Snorkel,</li>
<li>reglas JSON,</li>
<li>ajuste por factura,</li>
<li>correcciones manuales.</li>
</ul>
</li>
<li>
<p><code>fuente_categoria_final</code><br>
→ origen de la categoría final:</p>
<ul>
<li><code>"correccion_manual_excel"</code></li>
<li><code>"ajuste_por_factura"</code></li>
<li><code>"snorkel+reglas_cluster"</code></li>
<li><code>"reglas_cluster"</code></li>
</ul>
</li>
</ol>
<hr>
<p>Este README debería dejar al equipo de datos con una visión clara de:</p>
<ul>
<li>qué hace cada etapa,</li>
<li>cómo se puebla cada columna,</li>
<li>dónde meter mano (reglas, Snorkel, prompts),</li>
<li>y cómo usar <code>cat_final_linea</code> con confianza.</li>
</ul>

