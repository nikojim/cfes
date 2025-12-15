---


---

<h1 id="readme-—-sistema-de-categorización-inteligente-de-líneas-de-facturas-cfe-uruguay"><strong>README — Sistema de Categorización Inteligente de Líneas de Facturas (CFE Uruguay)</strong></h1>
<p><em>Pipeline unificado con Clustering + LLM + Supervisión Débil (Snorkel) + Reglas JSON + Ajuste por Factura</em></p>
<hr>
<h2 id="🧭-resumen-general-del-flujo">🧭 <strong>Resumen General del Flujo</strong></h2>
<p>Este proyecto implementa un pipeline completo para <strong>clasificar descripciones de líneas de facturas</strong> provenientes del sistema CFE (DGI Uruguay), donde no existen códigos de producto normalizados. El objetivo es generar una columna final:</p>
<p><code>cat_final_linea</code></p>
<p>que sea coherente, estable, auditable y útil para:</p>
<ul>
<li>
<p>análisis financiero y EDA,</p>
</li>
<li>
<p>reporting corporativo,</p>
</li>
<li>
<p>políticas de control de gasto,</p>
</li>
<li>
<p>entrenamiento de modelos supervisados futuros,</p>
</li>
<li>
<p>monitoreo mes a mes.</p>
</li>
</ul>
<p>El pipeline combina:</p>
<ol>
<li>
<p><strong>Embeddings semánticos</strong></p>
</li>
<li>
<p><strong>Clustering (K-Means + HDBSCAN)</strong></p>
</li>
<li>
<p><strong>LLM multi-pasada por cluster (clasificación + auditoría)</strong></p>
</li>
<li>
<p><strong>Snorkel: supervisión débil fusionando múltiples reglas</strong></p>
</li>
<li>
<p><strong>Reglas JSON tradicionales del negocio</strong></p>
</li>
<li>
<p><strong>Categoría por factura + monotematismo</strong></p>
</li>
<li>
<p><strong>Correcciones humanas en Excel</strong></p>
</li>
<li>
<p><strong>Trazabilidad total mediante <code>fuente_categoria_final</code></strong></p>
</li>
</ol>
<hr>
<h1 id="🏗️-arquitectura-del-pipeline">🏗️ Arquitectura del Pipeline</h1>
<h2 id="️⃣-parsing-y-limpieza-inicial-celdas-1–20">1️⃣ Parsing y limpieza inicial (Celdas 1–20)</h2>
<p><strong>Objetivo:</strong><br>
Transformar los XML CFE en un dataset limpio y vectorizable.</p>
<h3 id="pasos-clave">Pasos clave:</h3>
<ul>
<li>
<p>Parseo XML por factura.</p>
</li>
<li>
<p>Limpieza de descripciones:</p>
<ul>
<li>lowercase, sin tildes, sin caracteres raros, normalización.</li>
</ul>
</li>
<li>
<p>Conversión de importes a pesos uruguayos.</p>
</li>
<li>
<p>Eliminación de facturas anuladas y remitos.</p>
</li>
<li>
<p>Preprocesamiento de texto para embeddings.</p>
</li>
</ul>
<p><strong>Producto:</strong><br>
DataFrame con:</p>
<p><code>descripcion_limpia nombre_comercial serie_cfe nro_cfe emisor monto_uyu embedding</code></p>
<p>Embeddings generados con:</p>
<p><code>sentence-transformers/paraphrase-multilingual-mpnet-base-v2</code></p>
<hr>
<h2 id="️⃣-clustering-semántico-k-means--hdbscan-celdas-21–24">2️⃣ Clustering semántico (K-Means + HDBSCAN) (Celdas 21–24)</h2>
<p><strong>Objetivo:</strong><br>
Agrupar líneas similares para derivar categorías desde un contexto grupal.</p>
<h3 id="tareas">Tareas:</h3>
<ul>
<li>
<p>Determinación automática de <strong>k óptimo</strong> (método del codo).</p>
</li>
<li>
<p>Entrenamiento de K-Means.</p>
</li>
<li>
<p>Detección de outliers dentro de cada cluster usando <strong>HDBSCAN</strong>.</p>
</li>
<li>
<p>Selección de las <strong>20 descripciones más cercanas al centroide</strong> (representativas).</p>
</li>
</ul>
<p><strong>Produce:</strong></p>
<ul>
<li>
<p><code>cluster_id</code> por línea.</p>
</li>
<li>
<p><code>ejemplos_por_cluster</code>.</p>
</li>
<li>
<p>Mediciones de cohesión para auditoría.</p>
</li>
</ul>
<p><strong>Motivación:</strong><br>
El LLM no recibe 25.000 descripciones, sino <strong>30 clusters × 20 ejemplos</strong>, reduciendo costos y ruido.</p>
<hr>
<h2 id="️⃣-llm-—-primera-pasada-de-clasificación-por-cluster-celda-25">3️⃣ LLM — Primera pasada de clasificación por cluster (Celda 25)</h2>
<p>El LLM recibe:</p>
<ul>
<li>
<p>cluster_id</p>
</li>
<li>
<p>las 20 descripciones representativas</p>
</li>
<li>
<p>pistas y reglas del archivo <code>prompt_config_clusters.json</code></p>
</li>
</ul>
<p>Genera:</p>
<p><code>categoria_llm_cluster justificacion_llm_cluster</code></p>
<hr>
<h2 id="️⃣-llm-—-auditoría-del-cluster-celda-26">4️⃣ LLM — Auditoría del cluster (Celda 26)</h2>
<p>Segunda pasada donde el LLM actúa como <strong>auditor</strong>:</p>
<p>Evalúa:</p>
<ul>
<li>
<p>coherencia del cluster,</p>
</li>
<li>
<p>contradicciones internas,</p>
</li>
<li>
<p>palabras clave conflictivas,</p>
</li>
<li>
<p>categorías alternativas sugeridas.</p>
</li>
</ul>
<p>Produce:</p>
<p><code>cat_actual_auditor cat_sugerida_auditor coherente alertas</code></p>
<hr>
<h2 id="️⃣-eda-estática-de-clusters-conflictivos-reemplazo-de-celda-27">5️⃣ EDA estática de clusters conflictivos (Reemplazo de celda 27)</h2>
<p>Resumen simple de:</p>
<ul>
<li>
<p>clusters donde <code>cat_actual_auditor ≠ cat_sugerida_auditor</code>,</p>
</li>
<li>
<p>clusters con baja coherencia,</p>
</li>
<li>
<p>clusters con outliers.</p>
</li>
</ul>
<hr>
<h1 id="🟧-post-pipeline-p1–p8">🟧 <strong>POST-PIPELINE (P1–P8)</strong></h1>
<p>Una vez finalizada la parte de LLM, comienza el procesamiento profundo final.</p>
<hr>
<h2 id="🔹-p1-—-normalización--helpers">🔹 <strong>P1 — Normalización + helpers</strong></h2>
<p>Agrega:</p>
<ul>
<li>
<p><code>desc_norm</code></p>
</li>
<li>
<p><code>emisor_norm</code></p>
</li>
<li>
<p>detección de emisores multirrubro</p>
</li>
<li>
<p>normalización de <code>nombre_comercial</code></p>
</li>
</ul>
<hr>
<h2 id="🔹-p2-—-definición-de-categoría-final-por-cluster-cat_final_cluster">🔹 <strong>P2 — Definición de categoría final por cluster (<code>cat_final_cluster</code>)</strong></h2>
<p>Regla:</p>
<ul>
<li>Si el auditor propone una categoría diferente → el analista decide si adoptarla.<br>
(En automatización se mantiene la sugerida).</li>
</ul>
<hr>
<h1 id="🧠-supervisión-débil-con-snorkel-s0–s5">🧠 <strong>SUPERVISIÓN DÉBIL CON SNORKEL (S0–S5)</strong></h1>
<p>Esta es la parte más robusta del pipeline: toma <em>todas</em> las señales débiles y las combina para producir:</p>
<p><code>cat_snorkel</code></p>
<p>una etiqueta “consenso” estadísticamente optimizada.</p>
<hr>
<h2 id="🔹-s0-—-importación-snorkel">🔹 <strong>S0 — Importación Snorkel</strong></h2>
<h2 id="🔹-s1-—-construcción-de-categoria2id">🔹 <strong>S1 — Construcción de CATEGORIA2ID</strong></h2>
<p>Mapa numérico requerido por Snorkel.</p>
<hr>
<h2 id="🔹-s1.5-—-validación-json-cruzada">🔹 <strong>S1.5 — Validación JSON cruzada</strong></h2>
<p>Verifica:</p>
<ul>
<li>
<p>que todas las categorías usadas en reglas estén en:</p>
<ul>
<li>
<p><code>CATEGORIA2ID</code></p>
</li>
<li>
<p><code>categorias_sugeridas</code> del prompt</p>
</li>
<li>
<p><code>reglas_recategorizacion.json</code></p>
</li>
</ul>
</li>
</ul>
<p>Sugiere:</p>
<ul>
<li>
<p>categorías que faltan,</p>
</li>
<li>
<p>snippets JSON para copiar al prompt,</p>
</li>
<li>
<p>posibles reglas base nuevas.</p>
</li>
</ul>
<hr>
<h2 id="🔹-s2-—-labeling-functions-lfs-desde-reglas-de-negocio">🔹 <strong>S2 — Labeling Functions (LFs) desde reglas de negocio</strong></h2>
<p>Derivadas de <code>reglas_recategorizacion.json</code>:</p>
<h3 id="tipo-1-—-reglas-por-emisor">Tipo 1 — Reglas por emisor</h3>
<p><code>{ "col": "emisor_norm", "any_of": ["ferreteria"], "target": "Herramientas" }</code></p>
<p>LF equivalente:<br>
“si la palabra aparece en el emisor → votar Herramientas”.</p>
<h3 id="tipo-2-—-reglas-por-descripción">Tipo 2 — Reglas por descripción</h3>
<p><code>{ "col": "desc_norm", "any_of": ["lavandina","hipoclorito"], "target": "Productos de limpieza" }</code></p>
<h3 id="tipo-3-—-señal-de-cluster">Tipo 3 — Señal de cluster</h3>
<p>LF extra basada en la salida del LLM auditor:</p>
<p><code>lf_cluster_direct → vota cat_final_cluster</code></p>
<hr>
<h2 id="🔹-s2b-—-lfs-desde-pistas-de-palabras-clave-del-prompt_config">🔹 <strong>S2b — LFs desde pistas de palabras clave del prompt_config</strong></h2>
<p>Toma <code>pistas_palabras_clave</code>:</p>
<p><code>"Bebidas": ["salus", "botellon", "refresco"]</code></p>
<p>Cada categoría agrega una LF extra.<br>
Ejemplo LF:</p>
<p>“si desc_norm contiene salus → votar Bebidas”.</p>
<hr>
<h2 id="🔹-s2c-—-lfs-desde-nombre_comercial-rubro-empresarial">🔹 <strong>S2c — LFs desde nombre_comercial (rubro empresarial)</strong></h2>
<p>Toma reglas desde:</p>
<p><code>reglas_nomcom_snorkel.json</code></p>
<p>Ejemplo:</p>
<p><code>{ "lf_name": "nomcom_ferreteria", "keywords_upper": ["FERRETERIA","PINTURERIA"], "target_categoria": "Herramientas" }</code></p>
<p>estas LFs reducen errores clásicos como:</p>
<ul>
<li>“aguarrás” clasificado como Bebidas porque contiene “agua”.</li>
</ul>
<hr>
<h2 id="🔹-s3-—-entrenamiento-del-labelmodel-snorkel">🔹 <strong>S3 — Entrenamiento del LabelModel (Snorkel)</strong></h2>
<p>Toma todos los votos de todas las LFs y entrena un modelo estadístico que:</p>
<ul>
<li>
<p>estima confiabilidad de cada LF,</p>
</li>
<li>
<p>maneja contradicciones,</p>
</li>
<li>
<p>produce una etiqueta limpia:</p>
</li>
</ul>
<p><code>cat_snorkel</code></p>
<hr>
<h2 id="🔹-s4-—-comparaciones-estructuradas">🔹 <strong>S4 — Comparaciones estructuradas</strong></h2>
<p>Tablas:</p>
<ul>
<li>
<p><code>cat_final_cluster → cat_snorkel</code></p>
</li>
<li>
<p><code>cat_snorkel → cat_reglas</code></p>
</li>
<li>
<p><code>cluster → reglas</code></p>
</li>
</ul>
<p>Porcentajes de match / mismatch.</p>
<hr>
<h2 id="🔹-s5-—-ejemplos-cualitativos">🔹 <strong>S5 — Ejemplos cualitativos</strong></h2>
<p>Listas concretas de:</p>
<ul>
<li>
<p>casos donde Snorkel corrige a cluster,</p>
</li>
<li>
<p>casos donde reglas corrigen a Snorkel,</p>
</li>
<li>
<p>casos donde reglas se alejan de cluster+snorkel.</p>
</li>
</ul>
<p>Excelente para auditoría humana y mejora de reglas.</p>
<hr>
<h1 id="🟥-p3-—-aplicación-de-reglas-json-capa-final-de-negocio">🟥 <strong>P3 — Aplicación de reglas JSON (capa final de negocio)</strong></h1>
<p>Usa como entrada:</p>
<ul>
<li>
<p><code>cat_snorkel</code> si existe,</p>
</li>
<li>
<p>caso contrario: <code>cat_final_cluster</code>.</p>
</li>
</ul>
<p>Formato ejemplo:</p>
<p><code>{ "col": "desc_norm", "any_of": ["cartel","publicidad"], "target": "Publicidad y marketing" }</code></p>
<p>Las reglas:</p>
<ol>
<li>
<p><strong>reglas_por_emisor</strong></p>
</li>
<li>
<p><strong>reglas por descripción</strong></p>
</li>
<li>
<p><strong>consolidación de categorías</strong></p>
</li>
<li>
<p><strong>prioridad</strong> para resolver conflictos</p>
</li>
</ol>
<p>La salida es:</p>
<p><code>cat_reglas</code></p>
<hr>
<h1 id="🟦-p4-—-categoría-por-factura-contexto-superior">🟦 <strong>P4 — Categoría por factura (contexto superior)</strong></h1>
<p>Crea ID factura:</p>
<p><code>id_factura = serie_cfe + "-" + nro_cfe + "-" + emisor</code></p>
<p>Luego calcula:</p>
<ul>
<li>
<p><code>categoria_factura</code> = mayoría</p>
</li>
<li>
<p><code>n_lineas_factura</code></p>
</li>
<li>
<p><code>ratio_cat_dominante</code></p>
</li>
<li>
<p><code>es_monotematica</code> (según JSON):</p>
</li>
</ul>
<p><code>"ajuste_por_factura": { "habilitado": true, "min_lineas_factura": 3, "umbral_ratio_monotematica": 0.85 }</code></p>
<hr>
<h1 id="🟩-p5-—-ajuste-por-factura">🟩 <strong>P5 — Ajuste por factura</strong></h1>
<p>Si:</p>
<ul>
<li>
<p>factura es monotemática</p>
</li>
<li>
<p>emisor NO es multirrubro</p>
</li>
<li>
<p>categoría mayoritaria supera umbral</p>
</li>
</ul>
<p>Entonces:</p>
<p><code>categoria_ajustada_factura = categoria_factura cat_final_linea = categoria_factura</code></p>
<p>Si no:</p>
<p><code>cat_final_linea = cat_reglas</code></p>
<hr>
<h1 id="🟨-p6-—-exportación-a-excel-para-auditoría-humana">🟨 <strong>P6 — Exportación a Excel para auditoría humana</strong></h1>
<p>Genera:</p>
<h3 id="auditoria_categorias_facturas.xlsx-con"><code>auditoria_categorias_facturas.xlsx</code> con:</h3>
<ol>
<li>
<p><code>resumen_global</code></p>
</li>
<li>
<p><code>transiciones</code></p>
</li>
<li>
<p><code>lineas_ajustadas</code></p>
</li>
<li>
<p><code>descripciones_problematicas</code><br>
Donde el usuario puede escribir <code>cat_manual_linea</code>.</p>
</li>
</ol>
<hr>
<h1 id="🟩-p7-—-reimportación-del-excel">🟩 <strong>P7 — Reimportación del Excel</strong></h1>
<p>Actualiza:</p>
<p><code>cat_final_linea</code></p>
<p>para todas las correcciones manuales válidas del archivo.</p>
<hr>
<h1 id="🟪-p8-—-trazabilidad">🟪 <strong>P8 — Trazabilidad</strong></h1>
<p>Agrega:</p>
<p><code>fuente_categoria_final</code></p>
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
</ul>
<p>Esto permite evaluar la <strong>calidad y origen</strong> de cada etiqueta.</p>
<hr>
<h1 id="🏁-salida-final">🏁 <strong>Salida Final</strong></h1>
<p>DataFrame con:</p>
<ul>
<li>
<p><code>cluster_id</code></p>
</li>
<li>
<p><code>cat_final_cluster</code></p>
</li>
<li>
<p><code>cat_snorkel</code></p>
</li>
<li>
<p><code>cat_reglas</code></p>
</li>
<li>
<p><code>categoria_factura</code></p>
</li>
<li>
<p><code>categoria_ajustada_factura</code></p>
</li>
<li>
<p><code>cat_manual_linea</code></p>
</li>
<li>
<p><code>cat_final_linea</code></p>
</li>
<li>
<p><code>fuente_categoria_final</code></p>
</li>
</ul>
<hr>
<h1 id="📂-dónde-viven-las-reglas-y-qué-hace-cada-una">📂 <strong>Dónde viven las reglas y qué hace cada una</strong></h1>
<h2 id="reglas_recategorizacion.json">1. <code>reglas_recategorizacion.json</code></h2>
<p>Define reglas de negocio <em>tradicionales</em>:</p>
<h3 id="✔-reglas_por_emisor">✔ <code>reglas_por_emisor</code></h3>
<p>Etiqueta basada en rubro del emisor.<br>
Ejemplo:<br>
Si el emisor contiene “ferretería” → Herramientas.</p>
<h3 id="✔-reglas-por-descripción">✔ <code>reglas</code> (por descripción)</h3>
<p>Palabras clave → categoría.</p>
<h3 id="✔-consolidacion_categorias">✔ <code>consolidacion_categorias</code></h3>
<p>Une categorías pequeñas en grandes.</p>
<hr>
<h2 id="prompt_config_clusters.json">2. <code>prompt_config_clusters.json</code></h2>
<p>Usado exclusivamente por el LLM. Contiene:</p>
<h3 id="✔-categorias_sugeridas">✔ <code>categorias_sugeridas</code></h3>
<p>Conjunto global de categorías válidas.</p>
<h3 id="✔-pistas_palabras_clave">✔ <code>pistas_palabras_clave</code></h3>
<p>Guía semántica por categoría.</p>
<h3 id="✔-ejemplos_few_shot">✔ <code>ejemplos_few_shot</code></h3>
<p>Ejemplos de aprendizaje en el prompt.</p>
<h3 id="✔-uso_nom_comercial">✔ <code>uso_nom_comercial</code></h3>
<p>Instrucciones para rubros empresariales.</p>
<h3 id="✔-reglas_semanticas">✔ <code>reglas_semanticas</code></h3>
<p>Reglas de sentido común para el LLM:</p>
<ul>
<li>
<p>Dominancia</p>
</li>
<li>
<p>Coherencia</p>
</li>
<li>
<p>Ignorar genéricos</p>
</li>
<li>
<p>Contexto local</p>
</li>
<li>
<p>MIXTO</p>
</li>
</ul>
<h3 id="✔-instrucciones_segunda_pasada_llm">✔ <code>instrucciones_segunda_pasada_llm</code></h3>
<p>Guía para la auditoría.</p>
<hr>
<h2 id="reglas_nomcom_snorkel.json">3. <code>reglas_nomcom_snorkel.json</code></h2>
<p>Aporta <strong>supervisión débil basada en rubro del nombre comercial</strong>.</p>
<p>Ejemplo:</p>
<p><code>{ "lf_name": "nomcom_supermercado", "keywords_upper": ["SUPERMERC", "DISCO", "DEVOTO"], "target_categoria": "Alimentos" }</code></p>
<p>Estas reglas corrigen errores clásicos como:</p>
<ul>
<li>
<p>“agua Salus” → <strong>NO</strong> debe ser “Herramientas”</p>
</li>
<li>
<p>“Aguarrás” → <strong>Sí</strong> debe ser insumo de ferretería</p>
</li>
</ul>

