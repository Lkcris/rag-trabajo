# rag-trabajo
# Asistente RAG — Solo Supabase

App web completa que corre 100% en la nube sin instalar nada.
Accedés desde cualquier navegador o celular con un link.

---

## Arquitectura

```
Tu navegador
 ├── Extrae texto del PDF/Word/Excel  (pdf.js, mammoth, sheetjs)
 ├── Calcula embeddings               (transformers.js — corre en el navegador)
 └── Llama a Supabase:
      ├── Edge Function /ingest  →  guarda fragmentos en pgvector
      ├── Edge Function /ask     →  busca fragmentos + llama a Claude
      ├── Auth                   →  login de usuarios
      └── Storage                →  archivos originales (opcional)
```

**No hay servidor intermedio** — todo corre en Supabase o en el navegador del usuario.

---

## Paso 1 — Crear cuenta en Supabase

1. Ir a https://supabase.com y crear cuenta gratuita
2. Crear un nuevo proyecto (elegí región más cercana, ej: São Paulo)
3. Guardar la contraseña del proyecto — la vas a necesitar

---

## Paso 2 — Configurar la base de datos

1. En el panel de Supabase, ir a **SQL Editor**
2. Copiar y pegar todo el contenido de `supabase/schema.sql`
3. Antes de ejecutar, reemplazar `TU_EMAIL@ejemplo.com` con tu email real
4. Hacer clic en **Run** — debería decir "Success"

---

## Paso 3 — Crear las Edge Functions

En la terminal de tu compu (o en cualquier máquina con internet):

```bash
# Instalar Supabase CLI
npm install -g supabase

# Iniciar sesión
supabase login

# Vincular con tu proyecto (el ID lo encontrás en Settings > General)
supabase link --project-ref TU_PROJECT_ID

# Configurar las variables de entorno secretas
supabase secrets set ANTHROPIC_API_KEY=tu-clave-de-claude
supabase secrets set ADMIN_EMAIL=tu@email.com

# Publicar las funciones
supabase functions deploy ask
supabase functions deploy ingest
```

Si no tenés terminal disponible, también podés crear las funciones
desde el panel web de Supabase en **Edge Functions > New Function**
y pegar el código de cada archivo.

---

## Paso 4 — Crear usuarios

1. En Supabase ir a **Authentication > Users**
2. Hacer clic en **Add User**
3. Crear tu usuario admin (con el email que pusiste en schema.sql)
4. Crear usuarios para tus compañeros — solo podrán consultar, no subir docs

---

## Paso 5 — Configurar y publicar la app web

1. Abrir `src/index.html` con cualquier editor de texto (Bloc de notas sirve)
2. Reemplazar estas tres líneas al inicio del script:
   ```
   const SUPABASE_URL  = 'https://TU_PROJECT.supabase.co';
   const SUPABASE_ANON = 'TU_ANON_KEY';
   const ADMIN_EMAIL   = 'TU_EMAIL@ejemplo.com';
   ```
   Los valores los encontrás en Supabase → **Settings > API**

3. Guardar el archivo

---

## Paso 6 — Publicar el HTML para que todos accedan

**Opción A — GitHub Pages (gratis, recomendado):**
1. Crear cuenta en github.com
2. Crear repositorio nuevo, subir solo `index.html`
3. Ir a Settings > Pages > Source: main branch
4. En minutos tenés una URL pública tipo `tuusuario.github.io/rag-trabajo`

**Opción B — Netlify (gratis, más simple):**
1. Ir a netlify.com
2. Arrastrar la carpeta `src/` a la página
3. Te da una URL instantánea tipo `random-name.netlify.app`
   (podés cambiarla por algo más legible en Settings)

**Opción B — Compartir el archivo directamente:**
Si no querés publicarlo en internet, podés mandar el `index.html`
por email o Teams a cada compañero y abrirlo con doble clic.
Funciona igual — se conecta a Supabase desde el navegador local.

---

## Uso diario

**Vos (admin):**
- Entrás con tu email y contraseña
- Arrastrás documentos PDF/Word/Excel y apretás "Indexar"
- El procesamiento corre en tu navegador (~30 seg para doc grande)
- Los datos quedan en Supabase para siempre

**Tus compañeros:**
- Entran con su usuario y contraseña
- Solo ven el chat — no pueden subir ni borrar nada
- Escriben su pregunta y reciben respuesta con fuentes

---

## Estructura del proyecto

```
rag_supabase/
├── src/
│   └── index.html              ← app web completa (un solo archivo)
└── supabase/
    ├── schema.sql              ← tablas, índices y políticas de seguridad
    └── functions/
        ├── ask/index.ts        ← busca fragmentos y llama a Claude
        └── ingest/index.ts     ← guarda fragmentos en pgvector
```

---

## Costos

| Servicio | Plan gratuito | Cuándo pagar |
|---|---|---|
| Supabase | 500 MB DB, 1 GB storage | +$25/mes si superás |
| GitHub Pages / Netlify | Gratis siempre | Nunca para este caso |
| Claude API | Pago por uso | ~$0.003 por pregunta |

Para un equipo de call center con uso normal, el costo real es solo
el de la API de Claude — aproximadamente **$1-3/mes** según cuánto consulten.
