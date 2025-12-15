---


---

<p>Voy a suponer el esquema que definimos:</p>
<ul>
<li>
<p>Archivo: <code>auditoria_categorias_facturas.xlsx</code></p>
</li>
<li>
<p>Hojas principales:</p>
<ul>
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
<p><code>descripciones_problematicas</code> ✅ <strong>acá es donde se corrige</strong></p>
</li>
</ul>
</li>
</ul>
<hr>
<h2 id="¿qué-contiene-el-excel-y-para-qué-sirve-cada-hoja">1. ¿Qué contiene el Excel y para qué sirve cada hoja?</h2>
<h3 id="🧾-hoja-resumen_global">🧾 Hoja <code>resumen_global</code></h3>
<ul>
<li>
<p>Tablas agregadas, conteos por categoría, etc.</p>
</li>
<li>
<p>Uso: <strong>tener un panorama general</strong> de cómo está quedando el sistema.</p>
</li>
<li>
<p><strong>No se edita</strong>. Es solo informativa.</p>
</li>
</ul>
<h3 id="🔁-hoja-transiciones">🔁 Hoja <code>transiciones</code></h3>
<ul>
<li>
<p>Matrices tipo “de → a”:</p>
<ul>
<li>
<p><code>cat_final_cluster → cat_snorkel</code></p>
</li>
<li>
<p><code>cat_snorkel → cat_reglas</code></p>
</li>
<li>
<p><code>cat_reglas → categoria_ajustada_factura</code></p>
</li>
</ul>
</li>
<li>
<p>Uso: ver dónde más se cambian categorías.</p>
</li>
<li>
<p><strong>No se edita</strong>. Es diagnóstico.</p>
</li>
</ul>
<h3 id="📄-hoja-lineas_ajustadas">📄 Hoja <code>lineas_ajustadas</code></h3>
<ul>
<li>
<p>Listado de líneas donde se aplicó <code>ajuste_por_factura = True</code>.</p>
</li>
<li>
<p>Te muestra:</p>
<ul>
<li>
<p>categorías antes/después del ajuste,</p>
</li>
<li>
<p>detalle de factura,</p>
</li>
<li>
<p>por qué se ajustó.</p>
</li>
</ul>
</li>
<li>
<p><strong>No se edita</strong>, salvo que quieras marcar cosas a mano para análisis interno.<br>
La corrección <strong>formal</strong> se hace en la siguiente hoja.</p>
</li>
</ul>
<h3 id="🧩-hoja-descripciones_problematicas-✅">🧩 Hoja <code>descripciones_problematicas</code> ✅</h3>
<p>Esta es la <strong>hoja clave para corrección manual</strong>.<br>
Suele contener columnas de este estilo (ejemplo típico):</p>
<ul>
<li>
<p>Identificadores:</p>
<ul>
<li>
<p><code>index</code> (índice de la fila en el DF original)</p>
</li>
<li>
<p><code>id_factura</code></p>
</li>
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
<p><code>nombre_comercial</code></p>
</li>
</ul>
</li>
<li>
<p>Texto:</p>
<ul>
<li>
<p><code>descripcion_limpia</code> (o <code>desc_norm</code>)</p>
</li>
<li>
<p>a veces <code>alertas</code> del auditor LLM</p>
</li>
</ul>
</li>
<li>
<p>Categorías en distintas “capas”:</p>
<ul>
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
<p><code>cat_final_linea</code> (antes de corrección manual)</p>
</li>
</ul>
</li>
<li>
<p>Campos para corrección manual:</p>
<ul>
<li>
<p><code>cat_manual_linea</code> ⬅️ <strong>este lo completás vos</strong></p>
</li>
<li>
<p><code>comentario_manual</code> (opcional: explicación, notas)</p>
</li>
</ul>
</li>
</ul>
<hr>
<h2 id="paso-a-paso-cómo-corregir-el-excel">2. Paso a paso: cómo corregir el Excel</h2>
<h3 id="🥽-paso-1-abrir-y-ubicar-la-hoja">🥽 Paso 1: Abrir y ubicar la hoja</h3>
<ol>
<li>
<p>Abrí el archivo <code>auditoria_categorias_facturas.xlsx</code> en Excel o LibreOffice.</p>
</li>
<li>
<p>Ir a la hoja <strong><code>descripciones_problematicas</code></strong>.</p>
</li>
<li>
<p>Activar <strong>filtros</strong> en la fila de encabezados (en Excel: Datos → Filtro).</p>
</li>
</ol>
<hr>
<h3 id="🎯-paso-2-decidir-el-criterio-de-revisión">🎯 Paso 2: Decidir el criterio de revisión</h3>
<p>Hay varias formas de recorrerla, algunas recomendadas:</p>
<ul>
<li>
<p>Filtrar por:</p>
<ul>
<li>
<p><code>cat_final_linea</code> → categorías de más interés (ej. “Bebidas”),</p>
</li>
<li>
<p>o por <code>alertas</code> (si las incluimos),</p>
</li>
<li>
<p>o por casos donde:</p>
<ul>
<li>
<p><code>cat_reglas ≠ categoria_factura</code></p>
</li>
<li>
<p><code>cat_reglas ≠ cat_final_cluster</code></p>
</li>
<li>
<p>etc. (estas diferencias suelen estar ya pre-filtradas en esa hoja).</p>
</li>
</ul>
</li>
</ul>
</li>
</ul>
<p>En muchas implementaciones, la hoja <code>descripciones_problematicas</code> <strong>ya está filtrada</strong> para:</p>
<ul>
<li>
<p>casos donde hay inconsistencia,</p>
</li>
<li>
<p>o donde el sistema marcó problemas (clusters problemáticos, Snorkel dudando, etc.).</p>
</li>
</ul>
<hr>
<h3 id="👀-paso-3-inspección-caso-por-caso">👀 Paso 3: Inspección caso por caso</h3>
<p>Para <strong>cada fila</strong> problemática:</p>
<p>Mirá estas columnas en conjunto:</p>
<ol>
<li>
<p><strong><code>descripcion_limpia</code> / <code>desc_norm</code></strong><br>
→ Qué se compró realmente.</p>
</li>
<li>
<p><strong><code>nombre_comercial</code> / <code>emisor</code></strong><br>
→ Rubro del proveedor:</p>
<ul>
<li>
<p>ferretería,</p>
</li>
<li>
<p>supermercado,</p>
</li>
<li>
<p>banco,</p>
</li>
<li>
<p>veterinaria, etc.</p>
</li>
</ul>
</li>
<li>
<p><strong>Capas de categoría</strong>:</p>
<ul>
<li>
<p><code>cat_final_cluster</code><br>
→ qué dijo el LLM a nivel de cluster.</p>
</li>
<li>
<p><code>cat_snorkel</code><br>
→ consenso de reglas débiles.</p>
</li>
<li>
<p><code>cat_reglas</code><br>
→ después de aplicar reglas JSON.</p>
</li>
<li>
<p><code>categoria_factura</code><br>
→ contexto de la factura.</p>
</li>
<li>
<p><code>categoria_ajustada_factura</code><br>
→ lo que el ajuste por factura propuso.</p>
</li>
<li>
<p><code>cat_final_linea</code> (antes de tu corrección)<br>
→ lo que actualmente usaría el sistema si no tocás nada.</p>
</li>
</ul>
</li>
</ol>
<p>Tu tarea es responder:</p>
<blockquote>
<p>“¿Qué categoría debería tener <strong>esta línea</strong>, con el criterio del negocio?”</p>
</blockquote>
<hr>
<h3 id="✏️-paso-4-escribir-la-corrección">✏️ Paso 4: Escribir la corrección</h3>
<p>Si decidís que la categoría está <strong>bien</strong>, no hacés nada:<br>
→ Dejá <code>cat_manual_linea</code> <strong>vacío</strong>.</p>
<p>Si decidís cambiarla:</p>
<ol>
<li>
<p>En la columna <strong><code>cat_manual_linea</code></strong>, escribí la categoría correcta:</p>
<ul>
<li>
<p><strong>Debe ser exactamente</strong> uno de los nombres de categoría que usa el sistema:</p>
<ul>
<li>
<p><code>Alimentos</code>, <code>Bebidas</code>, <code>Productos de limpieza</code>,</p>
</li>
<li>
<p><code>Herramientas</code>, <code>Combustible</code>, <code>Gastos corporativos</code>, etc.</p>
</li>
</ul>
</li>
<li>
<p>Idealmente, las categorías posibles son las mismas que:</p>
<ul>
<li>
<p><code>categorias_sugeridas</code> del prompt,</p>
</li>
<li>
<p>las que ves en <code>cat_final_linea</code> en otras filas.</p>
</li>
</ul>
</li>
</ul>
</li>
<li>
<p>(Opcional) En <code>comentario_manual</code>, podés escribir algo como:</p>
<ul>
<li>
<p>“Es disolvente de pintura, debería ir en Herramientas / insumos de ferretería”</p>
</li>
<li>
<p>“Gasto bancario, pasó como Servicios financieros”</p>
</li>
<li>
<p>“Servicio de limpieza mensual, no Alimentos”</p>
</li>
</ul>
</li>
</ol>
<p>👉 Es <strong>muy importante</strong> que:</p>
<ul>
<li>
<p>no cambies el nombre de las columnas,</p>
</li>
<li>
<p>no borres filas intermedias,</p>
</li>
<li>
<p>no cambies el valor de <code>index</code> ni <code>id_factura</code>,<br>
porque el notebook usa esos campos para reubicar la fila original.</p>
</li>
</ul>
<hr>
<h3 id="🧠-ejemplo-concreto">🧠 Ejemplo concreto</h3>
<p>Supongamos una fila así:</p>
<p>index</p>
<p>descripcion_limpia</p>
<p>nombre_comercial</p>
<p>cat_reglas</p>
<p>categoria_factura</p>
<p>cat_final_linea</p>
<p>cat_manual_linea</p>
<p>comentario_manual</p>
<p>1234</p>
<p>aguarras</p>
<p>FERRETERIA RIVERA</p>
<p>Bebidas</p>
<p>Herramientas</p>
<p>Bebidas</p>
<p>Vos sabés que:</p>
<ul>
<li>
<p>“aguarrás” es un disolvente de pintura (insumo de ferretería),</p>
</li>
<li>
<p>el proveedor es una ferretería,</p>
</li>
<li>
<p>la factura puede estar llena de cosas de ferretería.</p>
</li>
</ul>
<p>Decisión:</p>
<ul>
<li>Categoría correcta: <code>Herramientas</code>.</li>
</ul>
<p>Entonces editás:</p>
<p>index</p>
<p>descripcion_limpia</p>
<p>nombre_comercial</p>
<p>cat_reglas</p>
<p>categoria_factura</p>
<p>cat_final_linea</p>
<p>cat_manual_linea</p>
<p>comentario_manual</p>
<p>1234</p>
<p>aguarras</p>
<p>FERRETERIA RIVERA</p>
<p>Bebidas</p>
<p>Herramientas</p>
<p>Bebidas</p>
<p>Herramientas</p>
<p>Disolvente de pintura, insumo de ferretería</p>
<p>Al guardar el Excel, esa corrección será aplicada por el notebook en P7.</p>
<hr>
<h3 id="💾-paso-5-guardar-el-archivo">💾 Paso 5: Guardar el archivo</h3>
<p>Recomendación:</p>
<ul>
<li>
<p>No sobrescribas el original sin versión.</p>
</li>
<li>
<p>Usá algo como:</p>
<ul>
<li><code>auditoria_categorias_facturas_2025-12.xlsx</code></li>
</ul>
</li>
</ul>
<p>Y usá siempre la ruta/nombre que espere el notebook (o parametrizalo).</p>
<hr>
<h2 id="¿qué-hace-el-notebook-después-con-estas-correcciones">3. ¿Qué hace el notebook después con estas correcciones?</h2>
<p>En la celda <strong>P7</strong>:</p>
<ol>
<li>
<p>Carga el Excel.</p>
</li>
<li>
<p>Toma sólo la hoja <code>descripciones_problematicas</code>.</p>
</li>
<li>
<p>Filtra las filas donde <code>cat_manual_linea</code> <strong>no es nula ni vacía</strong>.</p>
</li>
<li>
<p>Usa el <code>index</code> de esa fila para localizar la fila original en <code>df</code>.</p>
</li>
<li>
<p>Aplica:</p>
<p><code>df.loc[index, "cat_final_linea"] = cat_manual_linea</code></p>
</li>
<li>
<p>Marca esa fila como corrección manual (para P8):</p>
<p><code>fuente_categoria_final = "correccion_manual_excel"</code></p>
</li>
</ol>
<p>Así, en el DF final:</p>
<ul>
<li>
<p><code>cat_final_linea</code> ya incluye tus correcciones.</p>
</li>
<li>
<p>Podés filtrar fácilmente todas las líneas corregidas manualmente con:</p>
<p><code>df[df["fuente_categoria_final"] == "correccion_manual_excel"]</code></p>
</li>
</ul>
<hr>
<h2 id="recomendaciones-prácticas-para-que-no-se-rompa-nada">4. Recomendaciones prácticas para que no se rompa nada</h2>
<ol>
<li>
<p><strong>No toques los encabezados de las columnas.</strong><br>
Nada de renombrar <code>cat_manual_linea</code> a otra cosa.</p>
</li>
<li>
<p><strong>No borres filas</strong>, aunque no las corrijas.<br>
Es mejor que queden con <code>cat_manual_linea</code> vacío.</p>
</li>
<li>
<p><strong>No edites la columna <code>index</code></strong>.<br>
Es la llave que el notebook usa para mapear de vuelta.</p>
</li>
<li>
<p><strong>Usá exactamente los nombres de categorías que maneja el sistema</strong>:</p>
<ul>
<li>si escribís “herramientas” en minúscula y la categoría es <code>Herramientas</code>,<br>
→ mejor respetar la forma estándar (<code>Herramientas</code>) para evitar problemas downstream.</li>
</ul>
</li>
<li>
<p><strong>Documentá tus criterios en <code>comentario_manual</code></strong> cuando la decisión no es obvia.<br>
Eso ayuda para:</p>
<ul>
<li>
<p>re-entrenar reglas,</p>
</li>
<li>
<p>ajustar Snorkel,</p>
</li>
<li>
<p>explicar decisiones a terceros.</p>
</li>
</ul>
</li>
</ol>
<hr>
<h2 id="flujo-mental-recomendado-para-el-auditor-humano">5. Flujo mental recomendado para el auditor humano</h2>
<p>Para cada fila de <code>descripciones_problematicas</code>:</p>
<ol>
<li>
<p><strong>Leé la descripción y el nombre comercial</strong><br>
→ ¿qué rubro es? ¿qué parece?</p>
</li>
<li>
<p><strong>Mirar las capas de categoría</strong>:</p>
<ul>
<li>
<p>¿cluster + Snorkel + reglas + factura dicen cosas distintas?</p>
</li>
<li>
<p>¿cuál de esas capas te parece más razonable?</p>
</li>
</ul>
</li>
<li>
<p><strong>Preguntarte</strong>:</p>
<ul>
<li>
<p>¿Esta línea afecta algún análisis importante?</p>
<ul>
<li>(ej. insumos de campo vs gastos corporativos, etc.)</li>
</ul>
</li>
<li>
<p>¿La decisión es clara o dudosa?</p>
<ul>
<li>si es muy dudosa, puede valer dejarla como está o marcar un comentario en vez de corregir.</li>
</ul>
</li>
</ul>
</li>
<li>
<p><strong>Si la querés cambiar</strong> → escribí en <code>cat_manual_linea</code>.</p>
</li>
</ol>
<hr>
<p>Si querés, después puedo:</p>
<ul>
<li>
<p>proponerte un <strong>formato de “guía de auditoría”</strong> de 1 página para quien vaya a hacer estas correcciones (reglas prácticas tipo “si ves X, poné Y”),</p>
</li>
<li>
<p>o ayudarte a generar una hoja extra con <strong>estadísticas de cuántas líneas cambiaron por categoría</strong> luego de la corrección manual.<a href="https://stackedit.io/">https://stackedit.io/</a>).</p>
</li>
</ul>

