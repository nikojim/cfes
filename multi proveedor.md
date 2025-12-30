---


---

<h1 id="product-matching-y-detección-de-multi-proveedor">Product Matching y Detección de Multi-Proveedor</h1>
<h2 id="propósito-del-módulo">1. Propósito del Módulo</h2>
<p>Este módulo implementa un <strong>pipeline reproducible y auditable</strong> para identificar <strong>productos equivalentes comprados a múltiples proveedores</strong>, a partir de <strong>líneas de facturas electrónicas sin código de producto</strong>.</p>
<p>El enfoque combina:</p>
<ul>
<li>
<p>Clasificación semántica previa</p>
</li>
<li>
<p>Embeddings de texto</p>
</li>
<li>
<p>Matching intra-categoría</p>
</li>
<li>
<p>Reglas débiles (Snorkel)</p>
</li>
<li>
<p>Agrupación por componentes conexas</p>
</li>
</ul>
<p>El resultado principal es la generación de un <strong><code>product_candidate_id</code></strong>, que agrupa líneas equivalentes y permite detectar <strong>multi-proveedor</strong>, dispersión de precios y oportunidades de ahorro.</p>
<hr>
<h2 id="supuestos-de-diseño">2. Supuestos de Diseño</h2>
<ul>
<li>
<p>El matching <strong>solo ocurre dentro de una misma categoría final</strong> (<code>cat_reglas</code>).</p>
</li>
<li>
<p>No se intenta resolver matching global entre categorías.</p>
</li>
<li>
<p>Se prioriza <strong>precisión sobre recall</strong> (pipeline conservador).</p>
</li>
<li>
<p>Los proveedores se identifican por <code>rut_emisor</code>.</p>
</li>
</ul>
<hr>
<h2 id="entradas-esperadas">3. Entradas Esperadas</h2>
<h3 id="dataframe-de-líneas-df_lines">DataFrame de líneas (<code>df_lines</code>)</h3>
<p>Columnas mínimas requeridas:</p>
<p>Columna</p>
<p>Descripción</p>
<p><code>line_id</code></p>
<p>Identificador único de la línea</p>
<p><code>descripcion_limpia</code></p>
<p>Texto normalizado del ítem</p>
<p><code>embedding_vec</code></p>
<p>Embedding vectorial (list / np.array)</p>
<p><code>rut_emisor</code></p>
<p>Proveedor</p>
<p><code>importe_total</code></p>
<p>Monto de la línea</p>
<p><code>cat_reglas</code></p>
<p>Categoría final</p>
<hr>
<h2 id="flujo-del-proceso">4. Flujo del Proceso</h2>
<h3 id="enriquecimiento-estructural">4.1 Enriquecimiento estructural</h3>
<p>Desde <code>descripcion_limpia</code> se extraen:</p>
<ul>
<li>
<p>Unidad normalizada (<code>unidad_norm</code>)</p>
</li>
<li>
<p>Contenido (<code>contenido_norm</code>)</p>
</li>
<li>
<p>Pack (<code>pack_norm</code>)</p>
</li>
</ul>
<p>Esto permite reglas duras de compatibilidad.</p>
<hr>
<h3 id="generación-de-pares-candidatos">4.2 Generación de pares candidatos</h3>
<p>Para cada categoría:</p>
<ul>
<li>
<p>Se construye una matriz de embeddings</p>
</li>
<li>
<p>Se calculan similitudes cosine</p>
</li>
<li>
<p>Para cada línea se toman sus <code>top_k</code> vecinos</p>
</li>
<li>
<p>Se filtran pares con similitud mínima</p>
</li>
</ul>
<p>Resultado: dataset de <strong>pares potenciales</strong>.</p>
<hr>
<h3 id="exclusión-de-pares-triviales">4.3 Exclusión de pares triviales</h3>
<p>Por defecto:</p>
<ul>
<li>Se eliminan pares del <strong>mismo proveedor</strong></li>
</ul>
<p>Motivo: el objetivo es detectar <strong>proveedores alternativos</strong>, no duplicados internos.</p>
<hr>
<h3 id="scoring-same--different-snorkel">4.4 Scoring SAME / DIFFERENT (Snorkel)</h3>
<p>Cada par se evalúa mediante <strong>Labeling Functions</strong>:</p>
<p>Ejemplos:</p>
<ul>
<li>
<p>Alta similitud + compatibilidad → SAME</p>
</li>
<li>
<p>Unidades incompatibles → DIFFERENT</p>
</li>
<li>
<p>Contenido muy distinto → DIFFERENT</p>
</li>
<li>
<p>Similaridad baja → DIFFERENT</p>
</li>
</ul>
<p>El <code>LabelModel</code> produce:</p>
<p><code>p_same ∈ [0,1]</code></p>
<hr>
<h3 id="construcción-de-productos-candidatos">4.5 Construcción de productos candidatos</h3>
<p>Para cada categoría:</p>
<ul>
<li>
<p>Se toman pares con <code>p_same ≥ threshold</code></p>
</li>
<li>
<p>Cada par es una arista</p>
</li>
<li>
<p>Cada línea es un nodo</p>
</li>
<li>
<p>Se calculan <strong>componentes conexas</strong></p>
</li>
</ul>
<p>Cada componente se asigna a un:</p>
<p><code>product_candidate_id</code></p>
<hr>
<h3 id="detección-de-multi-proveedor">4.6 Detección de multi-proveedor</h3>
<p>Para cada producto candidato se calcula:</p>
<ul>
<li>
<p>Número de proveedores distintos</p>
</li>
<li>
<p>Lista de proveedores</p>
</li>
<li>
<p>Precio mínimo / mediano / máximo</p>
</li>
<li>
<p>Dispersión de precios</p>
</li>
<li>
<p>Ahorro potencial aproximado</p>
</li>
</ul>
<p>Criterio:</p>
<p><code>n_proveedores ≥ 2 → producto multi-proveedor</code></p>
<hr>
<h2 id="diagrama-del-pipeline-mermaid">5. Diagrama del Pipeline (Mermaid)</h2>
<p>`flowchart TD<br>
A[XML Facturas] --&gt; B[Parseo y Normalización]<br>
B --&gt; C[descripcion_limpia]<br>
C --&gt; D[Embeddings]<br>
C --&gt; E[Clasificación por categoría\ncat_reglas]</p>
<pre><code>E --&gt; F[Split por categoría]
D --&gt; F

F --&gt; G[Top-K vecinos por embedding]
G --&gt; H[Filtrado por similitud mínima]

H --&gt; I[Pares candidatos]
I --&gt; J[Excluir mismo proveedor]

J --&gt; K[Labeling Functions\nSnorkel]
K --&gt; L[LabelModel]
L --&gt; M[p_same por par]

M --&gt; N[Filtrar pares p_same ≥ threshold]
N --&gt; O[Grafo por categoría]
O --&gt; P[Componentes conexas]

P --&gt; Q[product_candidate_id]
Q --&gt; R[KPIs por producto]
R --&gt; S[Productos multi-proveedor]` 
</code></pre>
<hr>
<h2 id="salidas-del-módulo">6. Salidas del Módulo</h2>
<h3 id="dataframes-generados">DataFrames generados</h3>
<p>DataFrame</p>
<p>Contenido</p>
<p><code>df_enriched</code></p>
<p>Líneas con <code>product_candidate_id</code></p>
<p><code>df_pairs_scored</code></p>
<p>Pares evaluados con <code>p_same</code></p>
<p><code>df_products</code></p>
<p>Productos candidatos agregados</p>
<p><code>df_flagged</code></p>
<p>Productos multi-proveedor</p>
<hr>
<h2 id="interpretación-de-resultados">7. Interpretación de Resultados</h2>
<ul>
<li>
<p>Categorías con <code>n_productos ≈ n_lineas</code><br>
→ No hay matching fuerte (servicios, ítems únicos).</p>
</li>
<li>
<p>Categorías con bajo <code>% multi-proveedor</code><br>
→ Mercado concentrado o matching conservador.</p>
</li>
<li>
<p>Categorías con alta dispersión de precios<br>
→ Oportunidades claras de optimización.</p>
</li>
</ul>
<hr>
<h2 id="diagnósticos-recomendados">8. Diagnósticos Recomendados</h2>
<p>Antes de ajustar el modelo:</p>
<ul>
<li>
<p>Distribución de <code>p_same</code> por categoría</p>
</li>
<li>
<p>% de pares que superan el umbral</p>
</li>
<li>
<p>% de abstención de LFs</p>
</li>
<li>
<p>Comparación <code>sim_emb</code> vs <code>p_same</code></p>
</li>
</ul>
<p>Estos diagnósticos explican <strong>por qué</strong> una categoría genera (o no) multi-proveedor.</p>
<hr>
<h2 id="extensiones-futuras">9. Extensiones Futuras</h2>
<ul>
<li>
<p>Umbral <code>p_same</code> dinámico por categoría</p>
</li>
<li>
<p>LFs específicas por rubro</p>
</li>
<li>
<p>Incorporar marca / modelo / SKU textual</p>
</li>
<li>
<p>Análisis temporal (cambio de proveedor)</p>
</li>
<li>
<p>Visualización tipo red por categoría</p>
</li>
</ul>
<hr>
<h2 id="conclusión">10. Conclusión</h2>
<p>Este módulo transforma un problema típicamente <strong>no estructurado</strong> (líneas de facturas sin código) en una <strong>estructura analítica sólida</strong>, habilitando:</p>
<ul>
<li>
<p>Consolidación de compras</p>
</li>
<li>
<p>Evaluación de proveedores</p>
</li>
<li>
<p>Análisis de eficiencia</p>
</li>
<li>
<p>Soporte a decisiones estratégicas</p>
</li>
</ul>

