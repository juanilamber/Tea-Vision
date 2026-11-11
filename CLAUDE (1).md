# App Tea · instrucciones del proyecto

## Qué es
App personal para cada niño o niña con diagnóstico del neurodesarrollo, usada por su familia.
Metáfora: álbum del recién nacido (cálido, hitos) + libreta de vacunas (ordenado, consultable).
Problema: la información está dispersa y se pierde cuando cambia un terapeuta, la escuela o un apoyo.
Prueba para cada función: ¿le ahorra algo a la familia o se lo suma?

Especificación detallada del inicio guiado y de la ficha: `docs/inicio-guiado-y-ficha.md`.
Ahí cada punto está marcado [Definido] (decidido por la responsable) o [Propuesta] (a validar).
Lo [Definido] no se cambia sin preguntar.
Decisiones tomadas después: `docs/decisiones.md` (manda sobre las [Propuesta] de la especificación).

## Alcance actual: V1
Recorrido único de punta a punta:
1. Un cuidador crea su cuenta y su grupo familiar.
2. Pantalla 0 de consentimiento (registro con fecha y versión del texto) + bienvenida.
3. Inicio guiado: las 7 preguntas definidas, una por pantalla, todas opcionales, autoguardado y retomar.
4. Pantalla final: texto de cierre, tarjeta "Así conocemos hoy a [nombre]", vista previa de la ficha y "¿Por dónde querés empezar?".
   Sin "pestañas activadas" en la V1 (llegan en V2), pero las reglas de activación ya se guardan.
5. Perfil del niño o niña, con los campos que la ficha necesita y el inicio no pide:
   adulto de referencia y teléfono (encabezado), "Cómo aprende" (consignas y forma de aprender),
   "Salud y seguridad" (medicación, alergias, conductas de riesgo).
   En la vista previa de la ficha, cada bloque vacío tiene "Completar", que abre esa pregunta en formato una por pantalla.
6. Ficha "Cómo conocerme" en PDF A4 de una página, editable antes de exportar.
   Primera persona por defecto ("Me llamo..."), con selector para tercera persona.
7. Hermanos: el grupo familiar puede tener varios niños; selector de niño visible.
8. Segundo cuidador: se invita por enlace al grupo familiar y tiene los mismos permisos que el primero.

FUERA de la V1 (no implementar aunque parezca fácil): pestañas de área completas, invitar profesionales,
enlaces de la ficha con vencimiento, versiones de la ficha por destino, notas de voz, IA que lee informes,
OCR, modo sin conexión, módulo administrativo, recordatorios, alerta de crisis.
La alerta de crisis y cualquier interpretación clínica NO se implementan sin revisión regulatoria (ANMAT).
En la V1 las notas de voz se reemplazan por texto libre; dejar el lugar previsto en la interfaz.

## Stack
- Next.js (App Router, TypeScript) como PWA, mobile-first.
- Supabase: Postgres, Auth, Storage. Row Level Security en TODAS las tablas.
- Vercel para previews.
- Sin otras dependencias sin preguntar antes.

## Modelo de datos
- households(id, name, created_at)  -> grupo familiar
- household_members(household_id, user_id, role: caregiver, invited_by, joined_at)
- household_invites(id, household_id, token_hash, expires_at, used_at, created_by)
- children(id, household_id, name, nickname, birth_date, photo_path, created_at)
- consents(id, user_id, household_id, text_version, accepted_at)
- profile_answers(id, child_id, block, question_key, value_json, free_text, created_at, created_by)
  -> append-only: nunca UPDATE ni DELETE; la vista actual es la última por question_key.
  -> el primer conjunto queda marcado como "Versión inicial".
- onboarding_progress(child_id, last_step, updated_at)
- activation_rules: configuración editable (JSON o tabla), NO en el código: qué respuesta activa qué pestaña o bloque.
- [V3] child_access(child_id, user_id, role: reader|contributor, sections[], expires_at) para profesionales.
- [V3] access_log, share_links.
RLS: un usuario ve un niño solo si es miembro del household del niño (o, en V3, si tiene child_access vigente).
Los dos cuidadores son pares: ninguno puede quitar al otro (decisión de la responsable, "no por ahora").

## Reglas de datos (no negociables)
- Solo datos ficticios. Nunca cargar datos reales de un niño hasta que la responsable lo autorice por escrito.
- El consentimiento se registra antes de la pregunta 3 (diagnóstico). Texto legal: placeholder hasta que lo entregue el abogado (Ley 25.326).
- Secretos solo en el servidor (.env.local, nunca en el cliente ni en el repo).
- Recolectar solo lo necesario. Todo dato lleva fecha.
- Ningún dato del niño se envía a servicios de terceros (IA, analytics) en la V1.
- La ficha por defecto NO incluye diagnóstico detallado, historial médico, informes, CUD ni cobertura.

## Experiencia de usuario
- Español rioplatense (vos), tono cálido y cotidiano. Sin términos clínicos en la interfaz de la familia.
- Usar los textos de bienvenida y pantalla final tal como están en la especificación.
- Una pregunta por pantalla, barra de progreso, botón "Saltar" siempre visible. Respuestas de un toque.
- Uso con una mano en el celular; botones grandes; letra ajustable; contraste WCAG AA; etiquetas para lector de pantalla.
- Prever la variante "tu hijo/a o familiar" para adolescentes y adultos (textos en un archivo, no en el código).

## Identidad visual (basada en el manual de marca de la responsable)
Nombre de la app: **App Tea**. No usar el nombre ni el logo de María José Rivero en ningún lugar de la app.
Personalidad: profesional, cálida, creativa, curiosa, segura.
Paleta de marca:
- Silver Lake Blue #6288AF · Blue Gray #7796BD · Olivine #94AB7C · Thistle #DCBFD5
- Persian Orange #DE864A · Pale Dogwood #F3D7CA · Linen #F2E7DF (fondo)
Ningún color de marca llega a 4.5:1 para texto. Usar estos tonos oscuros derivados para texto y botones:
- Texto principal #2B3F55 (8.9:1 sobre Linen) · Azul de acción #3E5F82 (5.5:1; texto blanco encima 6.6:1)
- Los colores de marca se usan para fondos, ilustraciones, chips y bordes, nunca para texto chico.
- Naranja y verde de marca no llevan texto blanco encima.
Tipografías de marca: Rosha Keylin (títulos), Mollen (subtítulos), Switzer (texto), Spanish Signature (destacados, solo decorativo, nunca para información).
Fuentes en uso hasta tener los archivos con licencia (variables CSS, para cambiarlas en un solo lugar):
- Títulos: Fraunces (Google Fonts, vía next/font) en lugar de Rosha Keylin.
- Subtítulos: Switzer seminegrita, mayúsculas, letter-spacing 0.08em, en lugar de Mollen.
- Texto: Switzer (Fontshare, autoalojada en /public/fonts).
- Destacados decorativos: Caveat (Google Fonts) en lugar de Spanish Signature; nunca para información.
Elementos gráficos: formas orgánicas (flores, hojas, arcos, asteriscos) sobre grilla suave; usarlos con moderación.

## Cómo trabajar
- Antes de construir: proponer plan y criterios de aceptación; preguntar lo indispensable.
- Implementar por etapas. Después de cada una: qué cambió, cómo se probó, qué queda pendiente.
- No agregar funciones fuera del alcance.

## Criterios de aceptación V1
- [ ] Una familia ficticia completa consentimiento + 7 preguntas en alrededor de 5 minutos.
- [ ] Si se cierra la app a mitad del inicio, al volver retoma en la misma pantalla.
- [ ] Al recargar o volver a iniciar sesión, el perfil sigue guardado.
- [ ] El consentimiento queda registrado con fecha y versión antes de la pregunta 3.
- [ ] La ficha "Cómo conocerme" se descarga en PDF A4 de una página, con la fecha de última actualización visible.
- [ ] La ficha cambia entre primera y tercera persona sin perder contenido.
- [ ] Desde un bloque vacío de la ficha, "Completar" lleva a la pregunta y al volver el bloque aparece lleno.
- [ ] Una familia con dos hermanos ve dos perfiles separados y cambia entre ellos en un toque.
- [ ] El segundo cuidador entra con su propia cuenta por enlace de invitación y ve y edita a los mismos niños.
- [ ] Un usuario de otra familia no puede ver ningún niño, ni cambiando la URL ni llamando a la API.
- [ ] Saltear todas las preguntas no rompe nada; la ficha muestra solo lo completado.
- [ ] Funciona en un celular de gama media con letra agrandada.

## Datos de prueba
- "Familia Prueba A": dos cuidadores, dos hermanos (un niño de 4 años que se comunica con pictogramas y una niña de 7 sin diagnóstico confirmado, "en proceso de evaluación").
- "Familia Prueba B": un cuidador, una niña de 9 años en escuela común con maestra integradora.
Nombres, teléfonos y datos de salud siempre inventados.
