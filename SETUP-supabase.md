# Configuración — Lista de venta con página pública

Tienes **dos archivos** que comparten la misma base de datos:

| Archivo | Para quién | Qué puede hacer |
|---|---|---|
| `lista-venta_2.html` | **Solo tú** (panel privado) | Marcar vendidos, subir fotos, ver totales/pendiente |
| `lista-venta-publica.html` | **Compradores** (link público) | Solo mirar. No pueden marcar vendido ni ven el dinero pendiente. Se actualiza en vivo. |

Cuando marcas algo como vendido en tu panel, la página pública se actualiza sola en ~1 segundo.

---

## Paso 1 — Crear el proyecto en Supabase (gratis, ~5 min)

1. Entra a **https://supabase.com** y crea una cuenta.
2. **New project** → ponle un nombre (ej. `lista-venta`) y una contraseña de base de datos (guárdala, no la necesitas después). Elige la región más cercana.
3. Espera ~2 min a que el proyecto se cree.

## Paso 2 — Crear la tabla

En Supabase, ve a **SQL Editor** (icono `</>` en la barra izquierda) → **New query**, pega esto y dale **Run**:

```sql
-- Tabla con el estado de cada artículo (vendido + fotos)
create table venta_estado (
  id integer primary key,
  sold boolean not null default false,
  photos jsonb not null default '[]'::jsonb,
  name text,
  description text,
  price numeric,
  cat text,
  updated_at timestamptz not null default now()
);

-- Seguridad: permitir leer y escribir con la clave pública (anon)
alter table venta_estado enable row level security;

create policy "lectura publica" on venta_estado
  for select using (true);

create policy "escritura publica" on venta_estado
  for insert with check (true);

create policy "actualizacion publica" on venta_estado
  for update using (true) with check (true);

-- Activar actualizaciones en tiempo real
alter publication supabase_realtime add table venta_estado;
```

### Si ya creaste la tabla antes

En **SQL Editor** corre también lo que te falte:

```sql
alter table venta_estado add column if not exists name text;
alter table venta_estado add column if not exists description text;
alter table venta_estado add column if not exists price numeric;
alter table venta_estado add column if not exists cat text;
```

Las columnas `price` y `cat` hacen falta para **agregar artículos nuevos** desde el panel privado.

## Paso 3 — Copiar tus llaves

En Supabase ve a **Project Settings** (engrane) → **API**. Copia dos cosas:

- **Project URL** → algo como `https://abcd1234.supabase.co`
- **anon public** key → una cadena larga que empieza con `eyJ...`

> La `anon key` es **pública y segura de compartir** (así funcionan las apps de Supabase). La seguridad la dan las políticas del Paso 2, no ocultar la llave.

## Paso 4 — Pegarlas en LOS DOS archivos

Abre cada archivo y busca cerca del inicio del `<script>` el bloque **CONFIG**:

```js
const SUPABASE_URL = 'https://TU-PROYECTO.supabase.co';
const SUPABASE_ANON_KEY = 'TU-ANON-KEY';
```

Reemplaza ambos valores con los tuyos. **Usa exactamente los mismos valores en los dos archivos** (`lista-venta_2.html` y `lista-venta-publica.html`).

## Paso 5 — Publicar la página pública

Sube `lista-venta-publica.html` a cualquier hosting estático gratis y comparte ese link:

- **Lo más fácil:** entra a **https://app.netlify.com/drop** y arrastra el archivo. Te da un link al instante.
- Otras opciones: GitHub Pages, Vercel, Cloudflare Pages.

Tu panel privado (`lista-venta_2.html`) puedes tenerlo también ahí (guárdate TÚ ese link y no lo compartas), o abrirlo desde tu propio hosting. Evita abrirlo con `file://` directo del disco, porque el navegador puede bloquear la conexión a Supabase.

---

## Notas

- **Las fotos** ahora se guardan en la nube (antes eran solo de tu dispositivo). Súbelas desde el panel privado y aparecerán en la página pública.
- **Seguridad (garage sale):** con esta configuración cualquiera con la `anon key` podría técnicamente escribir. Para una venta de garaje entre conocidos es más que suficiente. Si quieres blindarlo más adelante, se puede mover la escritura detrás de un login de Supabase — avísame.
- Si ves un aviso amarillo de "Falta conectar Supabase", es que las llaves del Paso 4 aún no están puestas.
