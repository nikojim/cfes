---


---

<h2 id="🎯-objetivo-de-la-auditoría">🎯 <strong>Objetivo de la auditoría</strong></h2>
<p>El archivo <code>auditoria_categorias_facturas.xlsx</code> reúne únicamente las líneas de factura que el sistema considera <strong>ambiguas, inconsistentes o críticas</strong>.<br>
La tarea del auditor es revisar esas líneas y, cuando corresponda, <strong>corregir la categoría final sugerida por el sistema</strong>.</p>
<p>La corrección se hace <strong>solo</strong> en la hoja:</p>
<p><code>descripciones_problematicas</code></p>
<p>Y en una sola columna editable:</p>
<p><code>cat_manual_linea</code></p>
<hr>
<h1 id="🧭-1.-¿qué-mirar-primero">🧭 1. ¿Qué mirar primero?</h1>
<p>Cada fila representa una línea de factura considerada problemática. Para decidir correctamente la categoría, se deben evaluar tres fuentes de verdad:</p>
<h3 id="🔍-a.-la-descripción-del-productoservicio">🔍 A. La descripción del producto/servicio</h3>
<p>Columna clave:</p>
<ul>
<li><code>descripcion_limpia</code></li>
</ul>
<p>Preguntas que ayudan:</p>
<ul>
<li>
<p>¿Esto es un producto físico o un servicio?</p>
</li>
<li>
<p>¿Pertenece al rubro alimentos, limpieza, ferretería, repuestos, informática, etc.?</p>
</li>
</ul>
<hr>
<h3 id="🔍-b.-el-nombre-comercial-del-proveedor">🔍 B. El nombre comercial del proveedor</h3>
<p>Columnas clave:</p>
<ul>
<li>
<p><code>nombre_comercial</code></p>
</li>
<li>
<p><code>emisor</code></p>
</li>
</ul>
<p>Preguntas útiles:</p>
<ul>
<li>
<p>¿El proveedor es una <strong>ferretería</strong>? Entonces probablemente no vende alimentos.</p>
</li>
<li>
<p>¿El proveedor es un <strong>restaurante</strong>? Entonces lo más probable es Alimentos/Gastos corporativos.</p>
</li>
<li>
<p>¿Es un <strong>banco</strong>, <strong>Abitab</strong>, <strong>RedPagos</strong>? Probablemente es Servicios financieros.</p>
</li>
<li>
<p>¿Es una <strong>veterinaria</strong>, <strong>farmacia</strong> o <strong>clínica</strong>? Puede indicar insumos médicos.</p>
</li>
</ul>
<hr>
<h3 id="🔍-c.-la-categoría-de-la-factura-completa">🔍 C. La categoría de la factura completa</h3>
<p>Columnas clave:</p>
<ul>
<li>
<p><code>categoria_factura</code> (categoría dominante en la factura)</p>
</li>
<li>
<p><code>n_lineas_factura</code></p>
</li>
<li>
<p><code>es_monotematica</code></p>
</li>
</ul>
<p>Preguntas:</p>
<ul>
<li>
<p>¿Toda la factura es del mismo rubro?</p>
</li>
<li>
<p>¿Esta línea es la única que difiere del resto?<br>
→ Si sí, probablemente está mal categorizada.</p>
</li>
</ul>
<p>Ejemplo clásico:<br>
Una factura de FERRETERÍA donde todo es “tornillos”, “lijas”, “adhesivos”, menos una línea “aguarrás” que el sistema clasificó como Bebidas → <strong>claramente debe ser Herramientas/Materiales</strong>.</p>
<hr>
<h1 id="🧠-2.-decisión-¿corregir-o-no-corregir">🧠 2. Decisión: ¿Corregir o no corregir?</h1>
<p><strong>Solo corregí cuando tengas alta claridad.</strong></p>
<p>Casos claros para corregir:</p>
<ul>
<li>
<p>Errores semánticos obvios:</p>
<ul>
<li>
<p>“aguarrás” → NO Bebidas</p>
</li>
<li>
<p>“paracetamol” → NO Suministros de oficina</p>
</li>
<li>
<p>“resma A4” → NO Medicamentos</p>
</li>
<li>
<p>“gasoil” → NO Herramientas</p>
</li>
</ul>
</li>
<li>
<p>Cuando el rubro del proveedor contradice la categoría:</p>
<ul>
<li>
<p>Ferretería → Herramientas/Materiales, NO Alimentos/Bebidas.</p>
</li>
<li>
<p>Bar/Restaurante → Alimentos/Gastos corporativos.</p>
</li>
<li>
<p>Farmacia → Medicamentos y suministros médicos.</p>
</li>
<li>
<p>Estación de servicio (ANCAP) → Combustible.</p>
</li>
<li>
<p>Papelería → Suministros de oficina.</p>
</li>
</ul>
</li>
<li>
<p>Cuando la factura es <strong>monotemática</strong> y la línea aislada está mal.</p>
</li>
</ul>
<p>Casos donde <strong>NO</strong> conviene corregir:</p>
<ul>
<li>
<p>La línea es ambigua pero razonable dentro del rubro.</p>
</li>
<li>
<p>No sabés qué es el producto (ej. códigos internos de empresas).</p>
</li>
<li>
<p>La diferencia no impacta análisis superiores.</p>
</li>
<li>
<p>Es una categoría residual (“Otros”) que no amerita reclasificación.</p>
</li>
</ul>
<hr>
<h1 id="✏️-3.-cómo-editar-el-excel-correctamente">✏️ 3. Cómo editar el Excel correctamente</h1>
<h3 id="✔-paso-1-—-activar-filtros">✔ Paso 1 — Activar filtros</h3>
<p>En Excel:<br>
<strong>Datos → Filtro</strong></p>
<p>Filtrá por:</p>
<ul>
<li>
<p>categoría asignada,</p>
</li>
<li>
<p>proveedor,</p>
</li>
<li>
<p>productos específicos.</p>
</li>
</ul>
<hr>
<h3 id="✔-paso-2-—-para-corregir-una-línea">✔ Paso 2 — Para corregir una línea:</h3>
<p>Modificá <strong>solo una columna</strong>:</p>
<p><code>cat_manual_linea</code></p>
<p>Escribí <strong>exactamente</strong> uno de los nombres de categoría válidos del sistema, por ejemplo:</p>
<ul>
<li>
<p><code>Alimentos</code></p>
</li>
<li>
<p><code>Bebidas</code></p>
</li>
<li>
<p><code>Productos de limpieza</code></p>
</li>
<li>
<p><code>Suministros de oficina</code></p>
</li>
<li>
<p><code>Herramientas</code></p>
</li>
<li>
<p><code>Materiales de construcción</code></p>
</li>
<li>
<p><code>Combustible</code></p>
</li>
<li>
<p><code>Servicios profesionales</code></p>
</li>
<li>
<p><code>Gastos corporativos</code></p>
</li>
<li>
<p><code>Repuestos mecánicos</code></p>
</li>
<li>
<p><code>Servicio de mantenimiento</code></p>
</li>
<li>
<p><code>Medicamentos y suministros médicos</code></p>
</li>
<li>
<p><code>Publicidad y marketing</code></p>
</li>
<li>
<p><code>Otros</code></p>
</li>
</ul>
<blockquote>
<p>⚠ No uses nombres inventados ni variantes ortográficas<br>
(ej. “limpieza”, “herramienta”, “alimento”, etc. → inválidos)</p>
</blockquote>
<hr>
<h3 id="✔-paso-3-—-opcional-agregar-comentarios">✔ Paso 3 — (Opcional) Agregar comentarios</h3>
<p>En la columna:</p>
<p><code>comentario_manual</code></p>
<p>Es útil dejar notas como:</p>
<ul>
<li>
<p>“Artículo químico, es insumo de ferretería”</p>
</li>
<li>
<p>“Es gasto bancario, debe ir en servicios financieros”</p>
</li>
<li>
<p>“Producto de panadería”</p>
</li>
<li>
<p>“Artículo de jardinería, no alimentos”</p>
</li>
</ul>
<hr>
<h3 id="✔-paso-4-—-nunca-modificar">✔ Paso 4 — Nunca modificar:</h3>
<ul>
<li>
<p>el orden de las filas,</p>
</li>
<li>
<p>los encabezados,</p>
</li>
<li>
<p>la columna <code>index</code> (clave para mapear al DF),</p>
</li>
<li>
<p>la columna <code>id_factura</code>,</p>
</li>
<li>
<p>ninguna categoría intermedia (<code>cat_reglas</code>, <code>cat_snorkel</code>, etc.).</p>
</li>
</ul>
<blockquote>
<p>❌ No borres filas.<br>
❌ No agregues columnas.<br>
❌ No filtres y guardes sin querer sólo una parte (hacer “Guardar como” si vas a exportar una subset).</p>
</blockquote>
<hr>
<h1 id="🔄-4.-cómo-impactan-las-correcciones-en-el-sistema">🔄 4. Cómo impactan las correcciones en el sistema</h1>
<p>Cuando vuelvas al notebook y ejecutes <strong>P7</strong> (Reimportación):</p>
<ul>
<li>
<p>el sistema leerá la hoja,</p>
</li>
<li>
<p>filtrará filas donde <code>cat_manual_linea</code> <strong>no está vacía</strong>,</p>
</li>
<li>
<p>actualizará la categoría final:</p>
</li>
</ul>
<p><code>cat_final_linea = cat_manual_linea</code></p>
<ul>
<li>y marcará:</li>
</ul>
<p><code>fuente_categoria_final = "correccion_manual_excel"</code></p>
<p>Esto te permite monitorear fácilmente:</p>
<p><code>df[df.fuente_categoria_final == "correccion_manual_excel"]</code></p>
<hr>
<h1 id="❤️-5.-buenas-prácticas-del-auditor">❤️ 5. Buenas prácticas del auditor</h1>
<ol>
<li>
<p>✔ <strong>Usar sentido común + rubro del proveedor.</strong></p>
</li>
<li>
<p>✔ Priorizar correcciones de impacto (altos montos, emisores críticos).</p>
</li>
<li>
<p>✔ No sobrecorregir: si duda razonable → dejar como está.</p>
</li>
<li>
<p>✔ Documentar criterios en <code>comentario_manual</code>.</p>
</li>
<li>
<p>✔ Hacer revisión preliminar por categoría:</p>
<ul>
<li>
<p>mirar primero todas las de Bebidas,</p>
</li>
<li>
<p>luego Limpieza,</p>
</li>
<li>
<p>luego Combustible,</p>
</li>
<li>
<p>luego Herramientas, etc.</p>
</li>
</ul>
</li>
<li>
<p>✔ Si aparecen muchos errores similares:</p>
<ul>
<li>avisar al equipo de datos → se puede crear una nueva regla JSON o Snorkel para automatizarlo.</li>
</ul>
</li>
</ol>
<hr>
<h1 id="🏁-6.-ejemplos-típicos-de-corrección">🏁 6. Ejemplos típicos de corrección</h1>
<h3 id="🔧-caso-1-—-aguarrás">🔧 Caso 1 — Aguarrás</h3>
<p>descripcion</p>
<p>nombre_comercial</p>
<p>categoría actual</p>
<p>debería ser</p>
<p>aguarras</p>
<p>FERRETERÍA XYZ</p>
<p>Bebidas</p>
<p>Herramientas / Materiales</p>
<h3 id="🧴-caso-2-—-lavandina-en-almacén">🧴 Caso 2 — Lavandina en almacén</h3>
<p>descripcion</p>
<p>proveedor</p>
<p>cat actual</p>
<p>debería ser</p>
<p>lavandina</p>
<p>ALMACÉN</p>
<p>Alimentos</p>
<p>Productos de limpieza</p>
<h3 id="💳-caso-3-—-comisión-bancaria">💳 Caso 3 — Comisión bancaria</h3>
<p>descripcion</p>
<p>proveedor</p>
<p>cat actual</p>
<p>debería ser</p>
<p>comision bancaria</p>
<p>ABITAB</p>
<p>Otros</p>
<p>Servicios financieros</p>
<h3 id="🖨-caso-4-—-tóner-y-papelería">🖨 Caso 4 — Tóner y papelería</h3>
<p>descripcion</p>
<p>proveedor</p>
<p>cat actual</p>
<p>debería ser</p>
<p>toner hp</p>
<p>PAPELERÍA XYZ</p>
<p>Otros</p>
<p>Suministros de oficina</p>
<hr>
<h1 id="🧩-7.-resultado-final-esperado">🧩 7. Resultado final esperado</h1>
<p>Tras aplicar el Excel y ejecutar P7 + P8 en el notebook:</p>
<p>Cada línea tendrá una categoría final confiable y auditada:</p>
<p><code>cat_final_linea fuente_categoria_final</code></p>
<p>Las correcciones manuales prevalecen sobre:</p>
<ul>
<li>
<p>LLM,</p>
</li>
<li>
<p>Snorkel,</p>
</li>
<li>
<p>reglas JSON,</p>
</li>
<li>
<p>ajuste por factura,</p>
</li>
</ul>
<p>y quedan registradas con trazabilidad.</p>

