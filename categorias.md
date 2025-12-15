---


---

<h1 id="categoria_llm_cluster"><strong><code>categoria_llm_cluster</code></strong></h1>
<p>📌 <em>Primera pasada LLM (por cluster)</em><br>
💡 <strong>Qué es:</strong><br>
La categoría sugerida por el LLM usando:</p>
<ul>
<li>
<p>los 20 ejemplos representativos del cluster,</p>
</li>
<li>
<p>reglas del JSON,</p>
</li>
<li>
<p>pistas de palabras clave,</p>
</li>
<li>
<p>nombre comercial.</p>
</li>
</ul>
<p>💡 <strong>Quién la genera:</strong><br>
La primera llamada al LLM (clasificación inicial del cluster).</p>
<p>💡 <strong>Para qué sirve:</strong><br>
Es <strong>la primera versión</strong> de la categoría del cluster.<br>
No es definitiva.</p>
<p>💡 <strong>Advertencia:</strong><br>
Puede estar equivocada si:</p>
<ul>
<li>
<p>el cluster es ruidoso,</p>
</li>
<li>
<p>hay mezclas de rubros,</p>
</li>
<li>
<p>los ejemplos representativos no son perfectos.</p>
</li>
</ul>
<hr>
<h1 id="️⃣-cat_actual_auditor">2️⃣ <strong><code>cat_actual_auditor</code></strong></h1>
<p>📌 <em>Segunda pasada LLM (auditoría)</em><br>
💡 <strong>Qué es:</strong><br>
La categoría <strong>inicial</strong> del cluster, pero ahora vista desde el punto de vista del auditor.</p>
<p>💡 <strong>Quién la genera:</strong><br>
El LLM en la auditoría → simplemente copia la categoría que está auditando.</p>
<p>💡 <strong>Para qué sirve:</strong><br>
Para comparar con la categoría propuesta (ver ítem siguiente).</p>
<hr>
<h1 id="️⃣-cat_sugerida_auditor">3️⃣ <strong><code>cat_sugerida_auditor</code></strong></h1>
<p>📌 <em>Segunda pasada LLM (auditoría)</em><br>
💡 <strong>Qué es:</strong><br>
La categoría <strong>alternativa</strong> propuesta por el auditor si detecta que la categoría original es incorrecta o incoherente.</p>
<p>💡 <strong>Quién la genera:</strong><br>
El LLM auditor.</p>
<p>💡 <strong>Cuándo cambia:</strong><br>
Cuando:</p>
<ul>
<li>
<p>hay incoherencias,</p>
</li>
<li>
<p>hay dos tipos de productos mezclados,</p>
</li>
<li>
<p>la categoría real debería ser otra.</p>
</li>
</ul>
<p>💡 <strong>Para qué sirve:</strong><br>
Es la categoría que “el LLM recomendaría corregir”.</p>
<hr>
<h1 id="️⃣-cat_final_cluster">4️⃣ <strong><code>cat_final_cluster</code></strong></h1>
<p>📌 <em>Elección manual (Celda 28)</em><br>
💡 <strong>Qué es:</strong><br>
<strong>La categoría final del cluster</strong>, después de que vos decidís:</p>
<ul>
<li>
<p>aceptar la categoría sugerida por el auditor, o</p>
</li>
<li>
<p>mantener la categoría actual.</p>
</li>
</ul>
<p>💡 <strong>Quién la genera:</strong><br>
Vos (manual), a través del dashboard que marca qué clusters cambiar.</p>
<p>💡 <strong>Para qué sirve:</strong><br>
Es la categoría <strong>definitiva por cluster</strong>, usada luego para:</p>
<ul>
<li>
<p>calcular categoría de factura,</p>
</li>
<li>
<p>clasificar nuevas líneas,</p>
</li>
<li>
<p>ajuste por factura,</p>
</li>
<li>
<p>y terminar el dataset para entrenamiento.</p>
</li>
</ul>
<p>📌 Esta es la <strong>categoría oficial del cluster</strong>.</p>
<hr>
<h1 id="️⃣-categoria_factura">5️⃣ <strong><code>categoria_factura</code></strong></h1>
<p>📌 <em>Categorización por factura</em><br>
💡 <strong>Qué es:</strong><br>
La categoría dominante dentro de la factura según <strong>cat_final_cluster</strong>.</p>
<p>💡 <strong>Cómo se calcula:</strong><br>
Por voto de mayoría:</p>
<ul>
<li>
<p>tomás todas las líneas de la factura,</p>
</li>
<li>
<p>mirás qué categoría aparece más,</p>
</li>
<li>
<p>si la proporción supera un umbral (0.85) → factura monotemática.</p>
</li>
</ul>
<p>💡 <strong>Para qué sirve:</strong><br>
Es el contexto superior:<br>
→ “¿De qué es esta factura en su totalidad?”</p>
<p>Se usa para corregir líneas ambiguas.</p>
<hr>
<h1 id="️⃣-categoria_ajustada_factura">6️⃣ <strong><code>categoria_ajustada_factura</code></strong></h1>
<p>📌 <em>Ajuste automático de líneas por contexto de factura</em><br>
💡 <strong>Qué es:</strong><br>
La categoría final de cada <strong>línea</strong>, después de aplicar las reglas de ajuste por factura.</p>
<p>💡 <strong>Cómo funciona:</strong><br>
Si:</p>
<ul>
<li>
<p>la factura es monotemática,</p>
</li>
<li>
<p>el emisor no es multirrubro,</p>
</li>
<li>
<p>la categoría de la línea difiere de la de factura,</p>
</li>
</ul>
<p>→ <em>entonces</em> la línea se ajusta a <code>categoria_factura</code>.</p>
<p>💡 <strong>Para qué sirve:</strong><br>
Corrige errores típicos como:</p>
<ul>
<li>
<p>líneas sueltas mal clusterizadas,</p>
</li>
<li>
<p>confusiones como “Aguarrás → Bebidas”,</p>
</li>
<li>
<p>facturas 99% de herramientas con 1 ítem raro.</p>
</li>
</ul>
<hr>
<h1 id="️⃣-cat_linea_original">7️⃣ <strong><code>cat_linea_original</code></strong></h1>
<p>📌 <em>Copia de seguridad</em><br>
💡 <strong>Qué es:</strong><br>
Una copia de la categoría antes del ajuste por factura.</p>
<p>💡 <strong>Para qué sirve:</strong><br>
Para comparar:</p>
<p><code>cat_linea_original → categoria_ajustada_factura</code></p>
<p>y generar matriz de confusión / métricas.</p>
<hr>
<h1 id="️⃣-cat_final_linea">8️⃣ <strong><code>cat_final_linea</code></strong></h1>
<p>📌 <em>Edición manual línea a línea (Celda 29)</em><br>
💡 <strong>Qué es:</strong><br>
La categoría final definitiva de cada línea después de que vos mismo podés elegir:</p>
<ul>
<li>
<p>mantener la categoría del cluster,</p>
</li>
<li>
<p>aceptar la categoría de factura,</p>
</li>
<li>
<p>elegir una categoría manual de la lista (<code>categorias_sugeridas</code>).</p>
</li>
</ul>
<p>💡 <strong>Quién la genera:</strong><br>
Vos, con el dashboard de revisión por factura (Celda 29).</p>
<p>💡 <strong>Para qué sirve:</strong><br>
Es la categoría <strong>gold final</strong>:</p>
<ul>
<li>
<p>para entrenar el modelo supervisado,</p>
</li>
<li>
<p>para análisis,</p>
</li>
<li>
<p>para exportar a Excel,</p>
</li>
<li>
<p>para reporting.</p>
</li>
</ul>
<p>📌 <strong>Esta es la categoría final a nivel línea que debería usarse para todo análisis posterior.</strong></p>
<hr>
<h1 id="🧩-resumen-general-de-cómo-se-llega-a-la-categoría-final">🧩 <strong>Resumen general de cómo se llega a la categoría final</strong></h1>
<h3 id="por-cluster">Por cluster</h3>
<p><code>categoria_llm_cluster ↓ primera pasada cat_actual_auditor ↓ auditoría cat_sugerida_auditor ↓ decisión humana (Celda 28) cat_final_cluster ← categoría definitiva por cluster</code></p>
<h3 id="por-línea">Por línea</h3>
<p><code>cat_final_cluster (del cluster) ↓ contexto por factura categoria_factura ↓ ajuste automático por factura categoria_ajustada_factura ↓ revisión manual línea a línea (Celda 29) cat_final_linea ← categoría definitiva por línea</code>://stackedit.io/).</p>

