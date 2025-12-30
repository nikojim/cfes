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
<h1 id="apéndice">Apéndice</h1>
<h2 id="ejemplo-de-extracción-de-texto-a-partir-de-la-descripción-limpia">Ejemplo de extracción de texto a partir de la descripción limpia</h2>
<h1 id="extracción-de-features-estructurales-desde-descripcion_limpia">Extracción de Features Estructurales desde <code>descripcion_limpia</code></h1>
<h2 id="motivación-del-paso">1. Motivación del Paso</h2>
<p>Las descripciones de líneas de factura presentan una <strong>alta variabilidad léxica</strong>, incluso cuando refieren al <strong>mismo producto físico</strong>:</p>
<ul>
<li>“Agua mineral 500 ml”</li>
<li>“Agua sin gas 0,5L”</li>
<li>“Agua mineral x6 botellas 500cc”</li>
<li>“Agua bidón 6x0.5 litros”</li>
</ul>
<p>Si bien los <strong>embeddings</strong> capturan similitud semántica, <strong>no garantizan compatibilidad física</strong>. Por ejemplo, un embedding puede considerar similares:</p>
<ul>
<li>“Leche 1L”</li>
<li>“Leche 200ml”</li>
</ul>
<p>Desde el punto de vista del matching de productos, <strong>esto es incorrecto</strong>.</p>
<p>El objetivo de este paso es:</p>
<blockquote>
<p>Extraer señales <strong>estructurales y cuantitativas</strong> que permitan introducir <strong>reglas duras de compatibilidad</strong>, reduciendo falsos positivos.</p>
</blockquote>
<hr>
<h2 id="principio-de-diseño">2. Principio de Diseño</h2>
<p>Este paso se rige por tres principios:</p>
<ol>
<li>
<p><strong>Extracción conservadora</strong><br>
Si no se puede inferir con confianza, se devuelve <code>None</code>.</p>
</li>
<li>
<p><strong>Normalización a unidades base</strong><br>
Permite comparaciones numéricas directas.</p>
</li>
<li>
<p><strong>No bloquear el matching por ausencia de información</strong><br>
Si una feature no está presente, <strong>no invalida</strong> el par.</p>
</li>
</ol>
<hr>
<h2 id="input-del-proceso">3. Input del Proceso</h2>
<h3 id="columna-fuente">Columna fuente</h3>
<pre><code>descripcion_limpia : str
</code></pre>
<p>Características esperadas:</p>
<ul>
<li>Texto en minúsculas</li>
<li>Sin caracteres extraños</li>
<li>Sin HTML</li>
<li>Con abreviaturas comunes preservadas (<code>kg</code>, <code>ml</code>, <code>x6</code>, etc.)</li>
</ul>
<p>Este paso <strong>no limpia texto</strong>, solo <strong>interpreta</strong>.</p>
<hr>
<h2 id="features-extraídas">4. Features Extraídas</h2>

<table>
<thead>
<tr>
<th>Feature</th>
<th>Columna</th>
<th>Tipo</th>
<th>Ejemplo</th>
</tr>
</thead>
<tbody>
<tr>
<td>Unidad</td>
<td><code>unidad_norm</code></td>
<td>str / None</td>
<td><code>"g"</code>, <code>"ml"</code>, <code>"un"</code></td>
</tr>
<tr>
<td>Contenido</td>
<td><code>contenido_norm</code></td>
<td>float / None</td>
<td><code>500.0</code>, <code>1000.0</code></td>
</tr>
<tr>
<td>Pack</td>
<td><code>pack_norm</code></td>
<td>int</td>
<td><code>1</code>, <code>6</code>, <code>12</code></td>
</tr>
</tbody>
</table><hr>
<h2 id="extracción-de-pack-pack_norm">5. Extracción de Pack (<code>pack_norm</code>)</h2>
<p>Se detectan multiplicidades del tipo:</p>
<ul>
<li><code>x 6</code>, <code>por 6</code></li>
<li><code>pack 6</code></li>
<li><code>caja 12</code></li>
</ul>
<p>Si no se detecta pack → <code>pack_norm = 1</code>.</p>
<p>Nunca se devuelve <code>0</code> o <code>None</code>.</p>
<hr>
<h2 id="extracción-de-contenido-y-unidad">6. Extracción de Contenido y Unidad</h2>
<p>Unidades soportadas y normalizadas:</p>

<table>
<thead>
<tr>
<th>Texto detectado</th>
<th>Unidad normalizada</th>
</tr>
</thead>
<tbody>
<tr>
<td>kg, kilo</td>
<td>g</td>
</tr>
<tr>
<td>g, gr</td>
<td>g</td>
</tr>
<tr>
<td>l, lt</td>
<td>ml</td>
</tr>
<tr>
<td>ml, cc</td>
<td>ml</td>
</tr>
<tr>
<td>un, unidad</td>
<td>un</td>
</tr>
</tbody>
</table><p>Ejemplo:</p>

<table>
<thead>
<tr>
<th>Descripción</th>
<th>unidad_norm</th>
<th>contenido_norm</th>
<th>pack_norm</th>
</tr>
</thead>
<tbody>
<tr>
<td>Agua 500ml</td>
<td>ml</td>
<td>500</td>
<td>1</td>
</tr>
<tr>
<td>Agua x6 500cc</td>
<td>ml</td>
<td>500</td>
<td>6</td>
</tr>
<tr>
<td>Azúcar 1kg</td>
<td>g</td>
<td>1000</td>
<td>1</td>
</tr>
<tr>
<td>Servicio de flete</td>
<td>None</td>
<td>None</td>
<td>1</td>
</tr>
</tbody>
</table><hr>
<h2 id="uso-en-el-matching">7. Uso en el Matching</h2>
<p>Estas features <strong>no generan matching</strong>, solo lo restringen.</p>
<ul>
<li>Unidades incompatibles → <code>DIFFERENT</code></li>
<li>Contenido muy distinto → <code>DIFFERENT</code></li>
<li>Ausencia de datos → no bloquea</li>
</ul>
<hr>
<h2 id="filosofía">8. Filosofía</h2>
<p>Este paso no intenta “entender” el producto, sino <strong>evitar errores materiales obvios</strong>.</p>
<p>Es un control físico que complementa embeddings y protege la integridad del matching.</p>
<hr>
<h2 id="conclusión-1">9. Conclusión</h2>
<p>La extracción de features estructurales desde <code>descripcion_limpia</code> es un componente crítico para garantizar que el matching de productos refleje <strong>equivalencia real</strong> y no solo similitud textual.</p>

