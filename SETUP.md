# PONCHO RECORDS — Guía de Configuración Supabase
## Tiempo estimado: 15 minutos

---

## PASO 1 — Crear proyecto en Supabase

1. Ve a https://supabase.com y crea una cuenta gratuita
2. Haz clic en **"New Project"**
3. Ponle nombre: `poncho-records`
4. Elige una contraseña segura para la base de datos
5. Región: `South America (São Paulo)` — la más cercana a México
6. Espera ~2 minutos a que se inicialice

---

## PASO 2 — Crear la tabla de lanzamientos

1. En tu proyecto, ve al menú izquierdo → **"SQL Editor"**
2. Haz clic en **"New Query"**
3. Pega y ejecuta el siguiente SQL:

```sql
-- Tabla principal de lanzamientos
CREATE TABLE releases (
  id            UUID DEFAULT gen_random_uuid() PRIMARY KEY,
  title         TEXT NOT NULL,
  genre         TEXT,
  year          INTEGER,
  emoji         TEXT DEFAULT '🎵',
  description   TEXT,
  accent_color  TEXT DEFAULT '#00f5ff',
  bg_gradient   TEXT,
  cover_url     TEXT,
  cover_path    TEXT,
  spotify_url   TEXT,
  created_at    TIMESTAMPTZ DEFAULT NOW(),
  updated_at    TIMESTAMPTZ DEFAULT NOW()
);

-- Permitir lectura pública (para la landing)
ALTER TABLE releases ENABLE ROW LEVEL SECURITY;

CREATE POLICY "Lectura pública"
  ON releases FOR SELECT
  USING (true);

CREATE POLICY "Solo admin puede escribir"
  ON releases FOR ALL
  USING (auth.role() = 'authenticated');
```

4. Haz clic en **"Run"** — debes ver "Success"

---

## PASO 3 — Crear el bucket de imágenes

1. Ve al menú izquierdo → **"Storage"**
2. Haz clic en **"New Bucket"**
3. Nombre: `covers`
4. Marca **"Public bucket"** ✓ (para que las portadas sean visibles)
5. Haz clic en **"Save"**

### Configurar permisos del bucket:

1. Haz clic en el bucket `covers`
2. Ve a **"Policies"**
3. Haz clic en **"New Policy"** → **"For full customization"**
4. Agrega estas dos políticas:

**Política 1 — Lectura pública:**
```
Policy name: Lectura pública
Allowed operation: SELECT
Target roles: (dejar vacío = todos)
USING expression: true
```

**Política 2 — Solo admin puede subir:**
```
Policy name: Solo admin sube
Allowed operation: INSERT, UPDATE, DELETE
Target roles: authenticated
USING expression: true
WITH CHECK expression: true
```

---

## PASO 4 — Crear usuario administrador

1. Ve al menú izquierdo → **"Authentication"**
2. Haz clic en **"Users"**
3. Haz clic en **"Invite User"** o **"Add User"**
4. Pon tu email y una contraseña segura
5. Guarda bien esas credenciales — las usarás para entrar al admin

---

## PASO 5 — Obtener tus credenciales

1. Ve al menú izquierdo → **"Project Settings"** (ícono de engrane)
2. Haz clic en **"API"**
3. Copia estos dos valores:
   - **Project URL** → la que empieza con `https://xxxxx.supabase.co`
   - **anon / public key** → la clave larga que empieza con `eyJ...`

---

## PASO 6 — Pegar las credenciales en los archivos

Abre **`index.html`** y **`admin.html`** con cualquier editor de texto (Notepad, VS Code, etc.)

En ambos archivos busca estas líneas al inicio del `<script>`:

```javascript
const SUPABASE_URL = 'TU_SUPABASE_URL';
const SUPABASE_ANON_KEY = 'TU_SUPABASE_ANON_KEY';
```

Reemplaza con tus valores reales:

```javascript
const SUPABASE_URL = 'https://abcdefghij.supabase.co';
const SUPABASE_ANON_KEY = 'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...';
```

---

## PASO 7 — Subir a Netlify

1. Ve a https://app.netlify.com
2. Arrastra la carpeta completa `poncho-records/` al área de deploy
3. ¡Listo! Netlify te da una URL automáticamente

### Tu carpeta debe verse así:
```
📂 poncho-records/
   ├── index.html     ← Landing pública
   └── admin.html     ← Panel admin (acceso: tudominio.netlify.app/admin.html)
```

---

## USO DIARIO DEL PANEL ADMIN

1. Entra a: `https://tu-sitio.netlify.app/admin.html`
2. Ingresa con tu email y contraseña de Supabase
3. Haz clic en **"+ Nuevo Lanzamiento"**
4. Sube la portada arrastrando o haciendo clic
5. Llena: título, año, género, descripción, link de Spotify
6. Haz clic en **"Guardar"**
7. ¡Aparece automáticamente en la landing! 🚀

---

## PREGUNTAS FRECUENTES

**¿Es gratis?**
Sí. Supabase free tier incluye:
- 500MB de base de datos
- 1GB de Storage para imágenes
- 50,000 peticiones/mes
Más que suficiente para un catálogo musical.

**¿Alguien más puede ver el panel admin?**
Solo quien tenga email + contraseña. La URL `admin.html` es accesible pero sin credenciales no puede hacer nada.

**¿Cómo cambio la contraseña de admin?**
En Supabase → Authentication → Users → selecciona tu usuario → "Send password reset".

**¿Las imágenes tienen algún límite de tamaño?**
El código limita a 5MB por imagen. Recomendamos portadas de 800×800px en JPG/WebP.
