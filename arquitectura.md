---


---

<h1 id="readme-—-arquitectura-completa-del-sistema-de-categorización-de-líneas-de-facturas-cae-–-uruguay"><strong>README — Arquitectura Completa del Sistema de Categorización de Líneas de Facturas (CAE – Uruguay)</strong></h1>
<p>Este documento describe la arquitectura integral para la <strong>clasificación automática de descripciones de líneas de facturas</strong>, combinando:</p>
<ul>
<li>
<p>embeddings,</p>
</li>
<li>
<p>clustering semántico,</p>
</li>
<li>
<p>LLM multi-pasada,</p>
</li>
<li>
<p>auditoría automática,</p>
</li>
<li>
<p>reglas de negocio,</p>
</li>
<li>
<p>Snorkel (supervisión débil),</p>
</li>
<li>
<p>ajustes por factura,</p>
</li>
<li>
<p>correcciones humanas asistidas, y</p>
</li>
<li>
<p>generación de una categoría final robusta por línea.</p>
</li>
</ul>
<p>El objetivo es obtener una columna:</p>
<h1 id="➤-cat_final_linea">➤ <strong><code>cat_final_linea</code></strong></h1>
<p>que sirva para:</p>
<ul>
<li>
<p>análisis financiero y EDA,</p>
</li>
<li>
<p>reporting corporativo,</p>
</li>
<li>
<p>entrenamiento de modelos supervisados,</p>
</li>
<li>
<p>monitoreo temporal (mes a mes).</p>
</li>
</ul>
<hr>
<h1 id="📦-entradas-del-sistema">1. 📦 <strong>Entradas del sistema</strong></h1>
<h3 id="datos-de-origen-cfe-uruguay-–-xml">1.1. Datos de origen (CFE Uruguay – XML)</h3>
<ul>
<li>
<p><code>serie_cfe</code></p>
</li>
<li>
<p><code>nro_cfe</code></p>
</li>
<li>
<p><code>emisor</code></p>
</li>
<li>
<p><code>cliente</code></p>
</li>
<li>
<p>importes en UYU y USD</p>
</li>
<li>
<p>descripciones libres no estandarizadas<br>
(sin códigos de producto)</p>
</li>
</ul>
<h3 id="embeddings-precalculados">1.2. Embeddings precalculados</h3>
<p>Modelo:</p>
<p><code>sentence-transformers/paraphrase-multilingual-mpnet-base-v2</code></p>
<p>Columna:</p>
<ul>
<li><code>embedding</code> → vector de 768 dimensiones</li>
</ul>
<h3 id="reglas-de-negocio">1.3. Reglas de negocio</h3>
<p>Archivo:</p>
<p><code>reglas_recategorizacion.json</code></p>
<p>Incluye:</p>
<ul>
<li>
<p>reglas por emisor</p>
</li>
<li>
<p>reglas por palabras clave</p>
</li>
<li>
<p>consolidación de categorías</p>
</li>
<li>
<p>prioridades (orden de aplicación)</p>
</li>
</ul>
<h3 id="prompt-config-para-llm">1.4. Prompt config para LLM</h3>
<p>Archivo:</p>
<p><code>prompt_config_clusters.json</code></p>
<p>Incluye:</p>
<ul>
<li>
<p>categorías sugeridas</p>
</li>
<li>
<p>pistas por palabras clave</p>
</li>
<li>
<p>reglas semánticas</p>
</li>
<li>
<p>uso de nombre comercial (NomComercial)</p>
</li>
<li>
<p>ejemplos few-shot</p>
</li>
<li>
<p>instrucciones de auditoría</p>
</li>
</ul>
<h3 id="acceso-a-llm-openai">1.5. Acceso a LLM (OpenAI)</h3>
<p>Usado en:</p>
<ul>
<li>
<p>1ª pasada: clasificación por cluster</p>
</li>
<li>
<p>2ª pasada: auditoría por cluster</p>
</li>
</ul>
<hr>
<h1 id="🧠-vista-general-del-pipeline">2. 🧠 <strong>Vista general del pipeline</strong></h1>
<p>`flowchart TD</p>
<p>%% INPUT<br>
A0([XML CFE\n+ embeddings\n+ reglas JSON\n+ prompt config])</p>
<p>%% FASE 1<br>
A1[Parseo XML\nLimpieza descripciones\nNormalización importes]<br>
A0 --&gt; A1</p>
<p>%% FASE 2<br>
A2[Embeddings por línea\ny vectorizado]<br>
A1 --&gt; A2</p>
<p>%% FASE 3<br>
A3[K-Means (k óptimo)\ncluster_id por línea]<br>
A4[HDBSCAN por cluster\noutliers locales]<br>
A5[20 ejemplos más cercanos\npor cluster]<br>
A2 --&gt; A3 --&gt; A4 --&gt; A5</p>
<p>%% FASE 4<br>
A6[LLM 1ª Pasada:\ncategoría por cluster]<br>
A5 --&gt; A6</p>
<p>%% FASE 5<br>
A7[LLM Auditoría:\ncoherente / sugerida / alertas]<br>
A6 --&gt; A7</p>
<p>%% DASHBOARDS<br>
A8[Dashboards cluster\n(nube palabras,\ncoherencia, outliers)]<br>
A7 --&gt; A8</p>
<p>%% POSTPIPELINE<br>
subgraph POST [Post-pipeline]<br>
direction TB</p>
<p>P1[P1: Helpers + normalización\n(emisor_norm, desc_norm)]<br>
P2[P2: cat_final_cluster\n(auditor LLM)]<br>
S0[S0–S5: Snorkel\n(supervisión débil)]<br>
P3[P3: Reglas JSON\n(cat_reglas)]<br>
P4[P4: Categoría factura\n+ monotematismo]<br>
P5[P5: Ajuste por factura\n(cat_ajustada)]<br>
P6[P6: Excel auditoría\n+ descripciones_problematicas]<br>
P7[P7: Reimportar Excel\ncorrecciones humanas]<br>
P8[P8: fuente_categoria_final\n(trazabilidad)]</p>
<p>end</p>
<p>A8 --&gt; P1 --&gt; P2 --&gt; S0 --&gt; P3 --&gt; P4 --&gt; P5 --&gt; P6 --&gt; P7 --&gt; P8</p>
<p>%% OUTPUT<br>
Z([DF listo para modelo\ncat_final_linea])<br>
P8 --&gt; Z`</p>
<hr>
<h1 id="🟩-pipeline-principal-celdas-1–27">3. 🟩 <strong>Pipeline principal (celdas 1–27)</strong></h1>
<h3 id="contiene">Contiene:</h3>
<ul>
<li>
<p>parseo XML CFE,</p>
</li>
<li>
<p>normalización de importes,</p>
</li>
<li>
<p>limpieza de descripciones,</p>
</li>
<li>
<p>embeddings,</p>
</li>
<li>
<p>clustering K-Means + HDBSCAN,</p>
</li>
<li>
<p>selección de ejemplos por cluster,</p>
</li>
<li>
<p>LLM clasificación por cluster,</p>
</li>
<li>
<p>auditoría LLM,</p>
</li>
<li>
<p>dashboards para inspección.</p>
</li>
</ul>
<p><strong>Output de la etapa 0–27:</strong></p>
<ul>
<li>
<p><code>df</code> con:</p>
<ul>
<li>
<p>cluster_id</p>
</li>
<li>
<p>categoria_llm_cluster</p>
</li>
<li>
<p>datos limpios</p>
</li>
</ul>
</li>
<li>
<p><code>cluster_stats</code> con:</p>
<ul>
<li>
<p>cat_actual_auditor</p>
</li>
<li>
<p>cat_sugerida_auditor</p>
</li>
<li>
<p>coherencia</p>
</li>
<li>
<p>outliers</p>
</li>
<li>
<p>flags de problematicidad</p>
</li>
</ul>
</li>
</ul>
<hr>
<h1 id="🟧-post-pipeline-p1–p8">4. 🟧 <strong>Post-pipeline P1–P8</strong></h1>
<h2 id="✔-p1-—-helpers-y-normalización">✔ P1 — Helpers y normalización</h2>
<p>Crea:</p>
<ul>
<li>
<p><code>emisor_norm</code></p>
</li>
<li>
<p><code>desc_norm</code></p>
</li>
<li>
<p>detecta emisores multirrubro</p>
</li>
</ul>
<h2 id="✔-p2-—-categoría-final-por-cluster">✔ P2 — Categoría final por cluster</h2>
<p>Usa la auditoría LLM para decidir:</p>
<ul>
<li>
<p>mantener categoría,</p>
</li>
<li>
<p>o usar categoría sugerida.</p>
</li>
</ul>
<p>Genera:</p>
<ul>
<li><code>cat_final_cluster</code></li>
</ul>
<hr>
<h1 id="🟦-snorkel-s0–s5">5. 🟦 <strong>Snorkel (S0–S5)</strong></h1>
<h2 id="🎯-objetivo">🎯 Objetivo:</h2>
<p>Modelar la etapa de reglas débiles como un conjunto de <strong>Labeling Functions (LFs)</strong>.</p>
<h3 id="✔-s0">✔ S0</h3>
<p>Instalación e importación Snorkel.</p>
<h3 id="✔-s1">✔ S1</h3>
<p>Construcción del mapa:</p>
<ul>
<li>
<p><code>CATEGORIA2ID</code></p>
</li>
<li>
<p><code>ID2CATEGORIA</code></p>
</li>
</ul>
<h3 id="✔-s2">✔ S2</h3>
<p>Transformar reglas JSON en LFs:</p>
<ul>
<li>
<p>reglas por emisor,</p>
</li>
<li>
<p>reglas por descripción,</p>
</li>
<li>
<p>señal de cluster (<code>lf_cluster_direct</code>).</p>
</li>
</ul>
<h3 id="✔-s3">✔ S3</h3>
<p>Aplicación de LFs:</p>
<ul>
<li>
<p>Matriz L (n_muestras × n_lfs)</p>
</li>
<li>
<p>Entrenamiento:</p>
<p><code>LabelModel</code></p>
</li>
<li>
<p>Generación de:</p>
<ul>
<li>
<p><code>cat_snorkel</code></p>
</li>
<li>
<p>matriz de probas</p>
</li>
<li>
<p>análisis de LFs</p>
</li>
</ul>
</li>
</ul>
<h3 id="✔-s4">✔ S4</h3>
<p>Comparación:</p>
<ul>
<li>
<p>cluster vs snorkel</p>
</li>
<li>
<p>snorkel vs reglas</p>
</li>
<li>
<p>cluster vs reglas<br>
(tablas y porcentajes)</p>
</li>
</ul>
<h3 id="✔-s5">✔ S5</h3>
<p>Ejemplos concretos donde difieren<br>
(para inspección cualitativa).</p>
<hr>
<h1 id="🟥-p3–p7-—-consolidación-y-categoría-final">6. 🟥 <strong>P3–P7 — Consolidación y categoría final</strong></h1>
<h2 id="✔-p3-—-aplicar-reglas-json">✔ P3 — Aplicar reglas JSON</h2>
<p>Usa como base:</p>
<p><code>cat_snorkel (si existe) o cat_final_cluster</code></p>
<p>Genera:</p>
<ul>
<li><code>cat_reglas</code></li>
</ul>
<h2 id="✔-p4-—-categoría-por-factura">✔ P4 — Categoría por factura</h2>
<p>Genera:</p>
<ul>
<li>
<p><code>id_factura</code></p>
</li>
<li>
<p><code>categoria_factura</code></p>
</li>
<li>
<p><code>es_monotematica</code></p>
</li>
</ul>
<h2 id="✔-p5-—-ajuste-por-factura">✔ P5 — Ajuste por factura</h2>
<p>Genera:</p>
<ul>
<li>
<p><code>categoria_ajustada_factura</code></p>
</li>
<li>
<p><code>cat_final_linea</code> inicial</p>
</li>
<li>
<p><code>ajuste_por_factura</code></p>
</li>
</ul>
<h2 id="✔-p6-—-excel-auditoría">✔ P6 — Excel auditoría</h2>
<p>Genera archivo:</p>
<h3 id="auditoria_categorias_facturas.xlsx"><code>auditoria_categorias_facturas.xlsx</code></h3>
<p>Con hojas:</p>
<ul>
<li>
<p>resumen_global</p>
</li>
<li>
<p>transiciones</p>
</li>
<li>
<p>lineas_ajustadas</p>
</li>
<li>
<p>descripciones_problematicas ← editable</p>
</li>
</ul>
<h2 id="✔-p7-—-reimportar-excel">✔ P7 — Reimportar Excel</h2>
<p>Aplica cambios a:</p>
<ul>
<li><code>cat_final_linea</code><br>
en filas específicas (índices del Excel)</li>
</ul>
<hr>
<h1 id="🟪-p8-—-trazabilidad">7. 🟪 <strong>P8 — Trazabilidad</strong></h1>
<p>Agrega:</p>
<h3 id="fuente_categoria_final"><code>fuente_categoria_final</code></h3>
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
<p>Esto permite:</p>
<ul>
<li>
<p>auditoría,</p>
</li>
<li>
<p>confianza por línea,</p>
</li>
<li>
<p>filtrado de dataset para entrenamiento del modelo final.</p>
</li>
</ul>
<hr>
<h1 id="🎯-output-final">8. 🎯 <strong>Output final</strong></h1>
<p>DataFrame <code>df</code> listo para:</p>
<ul>
<li>
<p>modelos supervisados,</p>
</li>
<li>
<p>cuadros de mando,</p>
</li>
<li>
<p>reporting,</p>
</li>
<li>
<p>análisis financiero.</p>
</li>
</ul>
<p>Columnas clave:</p>
<p><code>cluster_id cat_final_cluster cat_snorkel cat_reglas categoria_factura categoria_ajustada_factura cat_final_linea fuente_categoria_final</code></p>

