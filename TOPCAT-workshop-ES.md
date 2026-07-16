# Bases de Datos Astronómicas y TOPCAT
## Taller práctico: encontrar la bimodalidad de galaxias en SDSS

> 🌐 [English](TOPCAT-workshop-EN.md) · **Español** · [Português](TOPCAT-workshop-PT.md)

> ⚠️ **La interfaz de TOPCAT está en inglés.** En este documento, los **nombres de menús,
> botones y columnas se mantienen en inglés** (`Views → Column Statistics`, `uPmag`, `nGood`)
> para que lo que leas coincida con lo que ves en pantalla.

---

**Qué vas a hacer:** partiendo de cero, vas a extraer una muestra de galaxias de un archivo
público, limpiarla, calcular sus magnitudes absolutas, y descubrir que las galaxias vienen en
**dos familias distintas** — una roja y muerta, otra azul y formando estrellas.

Después vas a hacer clic en un punto de tu gráfico y **ver la galaxia**.

Sin código. Noventa minutos.

---

## A dónde vas a llegar

**Este es el resultado final — lo que deberías tener en pantalla al terminar la sesión:**

![Diagrama color-magnitud con dos galaxias](images/cmd-two-galaxies.png)

A la izquierda, el **diagrama color-magnitud**, con contornos que revelan **dos picos
separados**.

A la derecha, dos galaxias sobre las que hiciste clic directamente en ese gráfico:

- **Arriba** — del **pico superior** (`u−r ≈ 2.7`): una **elíptica lisa y amarillenta**.
  Sin estructura.
- **Abajo** — del **pico inferior** (`u−r ≈ 1.7`): un **disco más azul** con estructura visible
  y un núcleo brillante.

**Vas a seleccionar estas galaxias usando nada más que un número.** Nunca le vas a decir a la
computadora nada sobre su forma — y aun así, los dos picos resultan ser **dos tipos de galaxia
que se ven distintos**.

Ese es el destino. Ahora vamos.

---

## Contenido

**[Parte 1 — El Observatorio Virtual](#parte-1--el-observatorio-virtual)**
**[Parte 2 — Explorar un servicio TAP](#parte-2--explorar-un-servicio-tap)**
**[Parte 3 — Tu primera consulta ADQL](#parte-3--tu-primera-consulta-adql)**
**[Parte 4 — Mirar los datos y encontrar la bimodalidad](#parte-4--mirar-los-datos-y-encontrar-la-bimodalidad)**
**[Parte 5 — Hacer clic en una galaxia](#parte-5--hacer-clic-en-una-galaxia-de-un-número-a-una-imagen)**
**[Parte 6 — Crossmatch: ¿el color predice la morfología?](#parte-6--crossmatch-el-color-realmente-predice-la-morfología)**
**[Apéndice — Las doce trampas](#apéndice--las-doce-trampas)**

---
---

# Parte 1 — El Observatorio Virtual

## 1.1 El problema que vino a resolver

Antes del VO, cada archivo astronómico era una isla:

- SDSS tenía su sitio web, su formato, su propia interfaz de búsqueda.
- 2MASS tenía otra.
- Chandra otra.
- El catálogo de un paper de 1998 vivía como un `.txt` en la página personal de alguien.

Para cruzar SDSS con 2MASS tenías que bajar los dos, entender dos formatos distintos, escribir un
script de conversión, y cruzarlos vos misma — cada vez, para cada par de catálogos.

No era solo tedioso; era una **barrera**. Solo quien sabía programar bien podía hacer ciencia
multi-longitud de onda.

## 1.2 Qué es realmente el VO

El **Observatorio Virtual** es un conjunto internacional de **estándares** que permite que los
archivos astronómicos hablen el mismo idioma.

**No es un lugar. No es un servidor. No es un sitio web al que entrás.**
Es un *protocolo* — como HTTP.

El objetivo: usar todos los archivos del mundo **como si fueran una sola base de datos**.

Los estándares los define la **IVOA** (*International Virtual Observatory Alliance*). Por eso los
identificadores empiezan con `ivo://`, así como las direcciones web empiezan con `http://`.

## 1.3 Los estándares que vas a usar hoy

| Estándar | Qué es | Dónde lo vas a ver |
|---|---|---|
| **VOTable** | El formato de tabla estándar. Trae los datos **y sus metadatos** (unidades, descripciones, UCDs). | Cada archivo que exportás o guardás |
| **TAP** | *Table Access Protocol* — cómo le pedís tablas a un servidor. | `VO → Table Access Protocol (TAP) Query` |
| **ADQL** | *Astronomical Data Query Language* — SQL más geometría esférica. | La consulta que escribís dentro de TAP |
| **Registry** | La guía telefónica: quién ofrece qué, y en qué dirección. | La lista de servicios que aparece al buscar |
| **SAMP** | Cómo se hablan entre sí los programas de tu máquina. | Aladin mandándole una tabla a TOPCAT |
| **Cone Search** | "Dame todo lo que hay dentro de este círculo del cielo." | Consultas posicionales en VizieR |
| **HiPS** | Imágenes del cielo teseladas y multi-resolución. | Los cutouts de galaxias de la Parte 5 |

## 1.4 Por qué esto importa

El menú `VO` de TOPCAT no es un menú más. Es la puerta a **todos los datos astronómicos públicos
del planeta**, desde un solo programa, sin bajar nada a mano.

> El Observatorio Virtual es lo que te permite cruzar un catálogo óptico de Estrasburgo con uno
> infrarrojo de California y uno de rayos X de la NASA — en tu laptop, en dos minutos, sin
> escribir una línea de código.

Para trabajo extragaláctico esto no es una comodidad. Es **la infraestructura sobre la que corre
el campo**: nadie construye una SED multibanda ni selecciona AGN por color sin el VO.

---
---

# Parte 2 — Explorar un servicio TAP

Abrí **`VO → Table Access Protocol (TAP) Query`**, buscá `SDSS`, y TOPCAT consulta el
**Registry**: *¿quién en el mundo tiene tablas que coincidan con SDSS?*

## 2.1 Cómo leer la lista de servicios

```
TAPVizieR (593/63752) - ivo://cds.vizier/tap
    ^        ^   ^              ^
    |        |   |              +-- Identificador único (IVOID)
    |        |   +----------------- Tablas TOTALES del servicio
    |        +--------------------- Tablas que COINCIDEN con tu búsqueda
    +------------------------------ Nombre del servicio
```

**Los números son un criterio de decisión real.** No elegís un servicio por su nombre — lo elegís
por si tiene lo que buscás. `TAPVizieR` matcheó 593 tablas de SDSS; `HEASARC` matcheó 32.

### Quién es quién

| Servicio | Quiénes son | Fuertes en |
|---|---|---|
| **TAPVizieR** | CDS, Estrasburgo | **Todo.** ~63.000 tablas — el agregador universal de catálogos publicados |
| **Data Lab TAP** | NOIRLab (EE.UU.) | Surveys ópticos profundos |
| **VSA / WSA / WFAU** | Univ. de Edimburgo | Surveys infrarrojos (VISTA, UKIRT) |
| **HEASARC** | NASA | **Altas energías**: rayos X, gamma — esencial para AGN |
| **IRSA** | Caltech / IPAC | **Infrarrojo**: WISE, Spitzer, Herschel |
| **PS1DR1 / PS1DR2** | STScI | Pan-STARRS |
| **ESO TAP_CAT** | ESO | Catálogos de ESO |
| **GAIA / ARI-Gaia / Gaia@AIP** | Varios espejos | Solo Gaia |

**Fijate que varios servicios ofrecen SDSS.** No hay un único lugar donde "vivan" los datos — y
ese es todo el punto del VO. Están distribuidos, y el Registry te ayuda a encontrar dónde.

> **No memorices dónde están los datos. Aprendé a preguntarle al Registry.**
> Los archivos cambian, se caen, se renombran. El Registry es la capa que te protege.

**Seleccioná `TAPVizieR` y hacé clic en `Use Service`** (o doble clic).

---

## 2.2 El explorador de esquemas

```
┌──────────────────────┬────────────────────────────────────┐
│  Find: [        ]    │  ○Service ○Schema ○Table ○Columns  │
│  ☑Name ☐Descrip      │                                    │
│                      │   (metadatos de lo que esté         │
│  ▼ TAPVizieR (63884) │    seleccionado a la izquierda)     │
│    ▸ I_astrometry    │                                    │
│    ▸ II_photometry   │                                    │
│    ▸ J_ApJ (10721)   │                                    │
│    ▸ large_tables    │                                    │
├──────────────────────┴────────────────────────────────────┤
│  ADQL Text:  (acá escribís tu consulta)                   │
│                        [ Run Query ]                      │
└───────────────────────────────────────────────────────────┘
```

### Cómo organiza VizieR sus 63.884 tablas

| Carpeta | Contenido |
|---|---|
| `I_astrometry` | Astrometría (Gaia, Hipparcos, ...) |
| `II_photometry` | Fotometría |
| `III_spectro` | Espectroscopía |
| `VI_misc`, `VII_nonstellar`, `VIII_radio`, `IX_HE` | Misceláneos, no estelares, radio, altas energías |
| `J_AA`, `J_ApJ`, `J_MNRAS`, ... | **Tablas de papers individuales**, agrupadas por revista |
| **`large_tables`** | **"extremely large catalogs"** — los catálogos a escala de survey |

`J_ApJ (10721)`: más de diez mil tablas publicadas solo en el *Astrophysical Journal*. **Ese es
el verdadero valor de VizieR** — si un paper publicó una tabla, probablemente está acá.

---

## 2.3 🔥 TRAMPA #1: `Name` vs `Descrip`

Escribí `sdss` en `Find:` con solo **`Name`** tildado:

```
TAPVizieR (32/63884)     <-- solo 32 resultados
```

Ahora tildá también **`Descrip`**:

```
TAPVizieR (642/63884)    <-- 642 resultados
```

**Veinte veces más resultados, por un solo checkbox.**

**Por qué:** con `Name` solo, TOPCAT busca en el *identificador* de la tabla (`V/154/sdss16`). La
mayoría de las tablas no llevan el nombre del survey en su identificador — lo llevan en su
**descripción**.

> **Cuando una búsqueda no devuelve nada, los datos casi seguro existen.**
> Estás buscando en el campo equivocado.

---

## 2.4 🔥 TRAMPA #2: dónde vive realmente SDSS

Incluso con 642 resultados, el catálogo principal de SDSS **no** está en `II_photometry`, ni en
ninguna carpeta llamada `V_algo`. Está en:

```
large_tables (8/148)      Descripción: "extremly large catalogs"   [el typo es de ellos]
    "II/294/sdss7"
    "II/306/sdss8"
    "V/139/sdss9"
    "V/147/sdss12"
    "V/154/sdss16"     <-- el que queremos
```

VizieR separa los catálogos a escala de survey (cientos de millones de filas) de los normales,
porque necesitan un tratamiento distinto del lado del servidor.

> **La organización de un archivo tiene una lógica interna — pero no necesariamente *tu*
> lógica.** Explorá; no asumas.

Están todas las versiones (DR7 → DR16). Usá la más reciente salvo que tengas una razón para no
hacerlo — y **decí en tu paper cuál usaste**.

---

## 2.5 Las pestañas de metadatos

Seleccioná `"V/154/sdss16"` con un clic, y recorré las pestañas de la derecha:

| Pestaña | Qué muestra |
|---|---|
| `Service` | Info del servicio TAP. La mirás una vez y no volvés. |
| `Schema` | Info de la **carpeta** (`large_tables`). El nivel de arriba. |
| `Table` | Info de **tu tabla**: nombre exacto (**copialo, no lo tipees**), descripción, **número de filas**. |
| **`Columns`** | **Cada columna: nombre, tipo, unidad, descripción, UCD. Esta es la pestaña que importa.** |
| `FKeys` | Claves foráneas — cómo esta tabla se conecta con otras. Avanzado. |

> ⚠️ **Mirá el número de filas en `Table`.** Si dice cientos de millones, tu cláusula `WHERE` más
> vale que sea selectiva, o vas a esperar una eternidad.

---

## 2.6 Leer `Columns` — el corazón del asunto

Acá se gana o se pierde el taller.

### Las columnas que necesitamos

| Columna | Unidad | Descripción | Uso |
|---|---|---|---|
| `RA_ICRS`, `DE_ICRS` | deg | Coordenadas (ICRS) | Posición |
| **`uPmag` … `zPmag`** | mag | Magnitudes **Petrosian** | Fotometría |
| `e_uPmag` … `e_zPmag` | mag | Sus errores | Calidad |
| **`zsp`** | — | **Redshift espectroscópico** (UCD: `src.redshift`) | Redshift |
| `e_zsp` | — | Error en `zsp` | Calidad |
| **`f_zsp`** | — | **Flag ZWarning** | ⚠️ Crítico |
| `zph` | — | Redshift fotométrico | Alternativa |
| `class` | — | `3=galaxy, 6=star` (**fotométrico**) | Filtro |
| **`spCl`** | — | `"GALAXY"`, `"QSO"`, `"STAR"` (**espectroscópico**) | Mejor filtro |
| `mode` | — | `1=primary` | Deduplicación |
| `clean` | — | `1=fotometría limpia` | Calidad |

---

### 🔥 TRAMPA #3: cinco columnas empiezan con `z` y significan cinco cosas distintas

```
zsp     -->  Redshift espectroscópico        <-- lo que querés
zph     -->  Redshift fotométrico            <-- otra cantidad (más ruidosa)
zmag    -->  Magnitud MODEL en banda z       <-- ¡una magnitud!
zpmag   -->  Magnitud PSF en banda z         <-- ¡otra magnitud!
zPmag   -->  Magnitud PETROSIAN en banda z   <-- ¡otra más!
```

**`zpmag` y `zPmag` se diferencian en UNA letra mayúscula.**

**La consulta no te va a avisar.** Va a correr, devolver números, y producir un gráfico. El
gráfico simplemente va a estar **mal**.

---

### 🔥 TRAMPA #4: las magnitudes PSF están mal para galaxias

Las primeras magnitudes de la lista son `upmag, gpmag, rpmag, ipmag, zpmag` — **magnitudes PSF**.
Es tentador usarlas.

**No lo hagas.** Una magnitud PSF ajusta la función de dispersión puntual de la imagen a la
fuente.

- Para una **estrella** → perfecto. La estrella *es* una PSF.
- Para una **galaxia extendida** → **ajusta la PSF dentro del núcleo y tira a la basura el disco y
  el halo.**

El flujo queda sistemáticamente subestimado, y el tamaño del error **depende del tamaño
angular** — o sea, del redshift y de la morfología. Estarías inyectando un sesgo correlacionado
con exactamente las variables que querés estudiar.

Para los colores es peor: las galaxias tienen gradientes de color (bulbos rojos, discos azules),
así que la PSF **sobrepesa el bulbo**. **Tu "secuencia roja" sería en parte un artefacto de
apertura.**

| Magnitud | Qué es | Usar para |
|---|---|---|
| **`?Pmag`** (Petrosian) | Apertura definida por el perfil de luz, independiente del redshift | **El estándar de SDSS para galaxias** |
| `?mag` (model) | Mejor ajuste de un perfil de Vaucouleurs o exponencial | También válida |
| `?pmag` (PSF) | ❌ | Solo estrellas |

#### ⚠️ No mezcles aperturas

Si medís `r` con apertura Petrosian y `(g−r)` con magnitudes model, **el punto que ploteás no
corresponde a ningún objeto físicamente consistente.**

**Elegí un sistema y usalo para todo.** Acá: **Petrosian**, magnitud *y* color.

---

### 🔥 TRAMPA #5: `class` no es `spCl`

| Columna | Cómo clasifica | Confiabilidad |
|---|---|---|
| `class` | **Fotométricamente** — por la forma en la imagen | Una *estimación* |
| **`spCl`** | **Espectroscópicamente** — por el espectro real | La **verdad** |

Si el objeto tiene espectro, **usá `spCl`**.

Y ojo: `spCl` distingue **`QSO`** de **`GALAXY`**. El `class=3` fotométrico te entregaría cuásares
junto con tus galaxias — y un cuásar es una fuente puntual con continuo de ley de potencias. Va a
caer en un lugar completamente distinto de tu CMD y va a contaminar todo.

---

### 🔥 TRAMPA #6: `f_zsp` — el flag que separa profesionales de principiantes

`f_zsp` es el flag **ZWarning** de SDSS.

```
f_zsp = 0    -->  el ajuste del redshift es confiable
f_zsp != 0   -->  el ajuste falló, el espectro era malo, algo salió mal
```

**Sin `f_zsp = 0`, tu muestra contiene redshifts basura.** Y una vez más, la consulta corre
perfectamente.

**Todos los surveys tienen flags como este. Encontralos. Usalos.**

---

### Dos columnas de calidad más que nunca habrías adivinado

| Columna | Por qué importa |
|---|---|
| **`mode = 1`** | SDSS fotografía regiones superpuestas. Sin esto, **la misma galaxia aparece varias veces** en tu muestra. |
| **`clean = 1`** | El flag compuesto de calidad fotométrica del propio SDSS. |

---

## 2.7 Una nota sobre los UCDs

Un **UCD** (*Unified Content Descriptor*) es una etiqueta estándar que describe qué *significa*
una columna, independientemente de cómo se *llame*.

`zsp` y `zph` llevan ambas `src.redshift`: nombres distintos, mismo significado físico. Dos
archivos pueden llamar a la misma cantidad `zsp`, `Z_SPEC` o `redshift` — pero todos la van a
etiquetar `src.redshift`.

**El software puede confiar en el UCD; no puede confiar en el nombre.** Así es como TOPCAT sabe
automáticamente cuáles son las coordenadas cuando hacés un crossmatch sin decirle nada.

---

## 2.8 Lo que hay que llevarse de la Parte 2

Cada una de las seis trampas de arriba **produce una consulta que corre sin error**.

Ninguna lanza una excepción. Ninguna imprime un warning. Cada una devuelve silenciosamente la
muestra equivocada — y vos la ploteás, y se ve perfectamente plausible.

> **La base de datos nunca te va a avisar que hiciste la pregunta equivocada.**
> Ese es tu trabajo.

---

## ✅ Checkpoint

Antes de escribir ADQL, tenés que poder decir:

- [ ] El **nombre exacto de la tabla**, copiado de la pestaña `Table`
- [ ] Las **columnas de coordenadas** — `RA_ICRS`, `DE_ICRS`
- [ ] La **columna de redshift** — `zsp`, y **por qué no** `zph`, `zmag`, `zpmag` ni `zPmag`
- [ ] Las **magnitudes** — `uPmag`…`zPmag`, y **por qué no** las PSF
- [ ] La **columna de clasificación** — `spCl`, y **por qué no** `class`
- [ ] Los **flags de calidad** — `f_zsp`, `mode`, `clean`, y de qué te protege cada uno

---
---

# Parte 3 — Tu primera consulta ADQL

## 3.1 La consulta

Pegá en la caja **`ADQL Text`** y hacé clic en **`Run Query`**:

```sql
SELECT TOP 30000
  RA_ICRS, DE_ICRS,
  "uPmag", "gPmag", "rPmag", "iPmag", "zPmag",
  "e_uPmag", "e_gPmag", "e_rPmag", "e_iPmag", "e_zPmag",
  zsp, e_zsp, spCl
FROM "V/154/sdss16"
WHERE RA_ICRS BETWEEN 320 AND 340
  AND DE_ICRS BETWEEN -1.25 AND 1.25
  AND spCl = 'GALAXY'
  AND zsp > 0.02 AND zsp < 0.2
  AND f_zsp = 0
  AND mode = 1
  AND clean = 1
```

**Resultado esperado: 7.539 filas, 15 columnas.**

> ✅ **Checkpoint.** Todos deberían obtener el mismo número.
> ¿Te dio 30.000? Falta un filtro. ¿Te dio 200? Algo se rompió.

---

## 3.2 ⏳ Si la consulta va lenta o falla — usá el archivo de respaldo

Esta consulta le pide a VizieR que escanee **cientos de millones de filas**. Con todos
consultando al mismo tiempo, el servidor puede ponerse lento o dar timeout.

**No te quedes esperando. Si tarda más de ~2 minutos, o falla:**

### 👉 Bajá `sdss_stripe82_galaxies.vot` de este repositorio

```
File  →  Load Table  →  seleccioná sdss_stripe82_galaxies.vot  →  OK
```

Es **exactamente el resultado de la consulta de arriba**. No perdés nada.

> **Esto no es hacer trampa.** Es lo que harías en la vida real. Apenas una consulta lenta tiene
> éxito, escribís el resultado a disco para no volver a depender nunca de un servidor remoto para
> los mismos datos.

**Si tu consulta sí funcionó:** guardala igual, ahora mismo —
`File → Save Table(s)` → formato `votable`.

---

## 3.3 Qué hace cada línea

| Línea | Por qué está ahí |
|---|---|
| `TOP 30000` | Un límite de filas **explícito**. A diferencia del formulario web de VizieR, ADQL nunca trunca en silencio. |
| `RA_ICRS BETWEEN 320 AND 340` | **Stripe 82** — una franja larga y angosta. Imposible de pedir con un cone search. |
| `DE_ICRS BETWEEN -1.25 AND 1.25` | Stripe 82 es angosta en declinación. *Eso* es lo que la hace Stripe 82. |
| `spCl = 'GALAXY'` | Galaxias — **no QSOs, no estrellas**. |
| `zsp > 0.02` | Descarta redshifts espurios y objetos muy cercanos. |
| `zsp < 0.2` | **El universo local.** |
| **`f_zsp = 0`** | **El ajuste del redshift es confiable.** |
| `mode = 1` | Solo detecciones primarias — sin duplicados. |
| `clean = 1` | El flag de "fotometría limpia" del propio SDSS. |

> **La consulta *es* la decisión científica.**
> Cada línea del `WHERE` sesga tu muestra, y tenés que poder defenderla.
> En un paper, esta consulta va en la sección de datos.

---

## 3.4 Lo que la consulta **no** filtra — a propósito

- ❌ Sin cortes de magnitud
- ❌ Sin cortes de color

**Deliberado.** Los datos que estás por cargar están **sucios**, y descubrir eso — y decidir qué
hacer al respecto — es el punto de la Parte 4.

Fijate en la ironía: la consulta *sí* contiene **`clean = 1`**, el flag de calidad fotométrica del
propio SDSS. Y aun así vas a encontrar magnitudes de 33. Guardá ese pensamiento.

---

## 3.5 🔥 TRAMPA #7: ADQL no distingue mayúsculas — y te va a frenar

Corré la consulta **sin** las comillas dobles y obtenés:

```
Incorrect ADQL query: 6 unresolved identifiers
  - Ambiguous column name "uPmag"! It may be (at least)
    "V/154/sdss16.upmag" or "V/154/sdss16.uPmag".
  ...
```

**El parser de ADQL no distingue mayúsculas de minúsculas.** Cuando escribís `uPmag`, el servidor
genuinamente no sabe si te referís a `upmag` (**PSF**) o a `uPmag` (**Petrosian**).

### La solución: encomillar el identificador

```sql
"uPmag"     -->  Petrosian. Sin ambigüedad.
```

### ⚠️ Dos tipos de comillas, dos trabajos distintos

| Comillas | Se usan para | Ejemplo |
|---|---|---|
| **Dobles** `"..."` | **Identificadores** (tablas, columnas) | `"uPmag"`, `"V/154/sdss16"` |
| **Simples** `'...'` | **Valores de texto** | `'GALAXY'` |

Si escribís `spCl = "GALAXY"` con dobles, ADQL va a buscar una *columna* llamada GALAXY y va a
fallar.

### La lección más profunda

**Ese mensaje de error te salvó.**

El parser solo se quejó porque *ambas* columnas existen. Si el catálogo hubiera tenido solo
`upmag`, tu consulta habría corrido perfectamente — y te habría entregado en silencio magnitudes
PSF de galaxias.

> **La ambigüedad fue lo que te protegió.** No cuentes con tener esa suerte dos veces.

---

## 3.6 Por qué la consulta es lenta: leer la columna `Indexed`

La demora no es un bug. Es información.

Volvé a **`Columns`** y mirá la columna **`Indexed`**:

| Columna | ¿Indexada? |
|---|---|
| `RA_ICRS`, `DE_ICRS`, `zsp`, `zph`, `objID` | ✅ |
| **`spCl`**, **`f_zsp`, `mode`, `clean`**, **`uPmag`…`zPmag`** | ❌ |

Una columna **indexada** se filtra al instante. Una **no indexada** obliga a escanear la tabla
entera.

Tu filtro más lento es `spCl = 'GALAXY'`: una comparación de **texto** sobre una columna **no
indexada**, contra cientos de millones de filas.

> **La consulta que escribís determina cuánto esperás.**
> En un survey grande, esa es la diferencia entre 10 segundos y 10 minutos.

**Para acelerarla,** achicá la región del cielo — tu palanca más barata, porque las coordenadas
**sí** están indexadas:

```sql
  AND RA_ICRS BETWEEN 330 AND 340    -- la mitad del área
```

---

## ✅ Checkpoint

- [ ] Tabla cargada — de tu consulta, o de `sdss_stripe82_galaxies.vot`
- [ ] **Filas = 7.539**, **Columnas = 15**
- [ ] La tabla está **guardada en tu disco**

---
---

# Parte 4 — Mirar los datos y encontrar la bimodalidad

**Todavía no plotees.**

> **Regla: nunca ploteés datos que no miraste.**
> Cinco minutos de inspección te ahorran una tarde debuggeando un gráfico que salió mal.

---

## 4.1 Mirá las filas

**`Views → Table Data`** (el ícono de grilla).

Verificá que la consulta hizo lo que creías: `RA_ICRS` en [320, 340], `DE_ICRS` en
[−1,25, 1,25], `zsp` en [0,02, 0,2], y `spCl` = `GALAXY` en todas.

---

## 4.2 Mirá las estadísticas — el paso que importa

**`Views → Column Statistics`** (el botón **`Σ`**).

```
Name       Mean      SD       Minimum    Maximum    nGood
RA_ICRS    330.22    5.86     320.00     339.99     7539
DE_ICRS      0.011   0.726     -1.249      1.249    7539
uPmag       20.41    1.30      15.683     33.147    7538   <-- !!
gPmag       18.97    1.10      14.161     29.146    7539   <-- !!
rPmag       18.20    1.14      13.437     24.041    7539
iPmag       17.82    1.19      13.037     27.632    7539   <-- !!
zPmag       17.63    1.31      12.782     26.657    7539   <-- !!
e_uPmag      0.381   0.894      0.017     23.884    7538   <-- !!!
e_gPmag      0.079   0.387      0.003     16.264    7539   <-- !!!
e_zPmag      0.189   0.822      0.005     44.086    7539   <-- !!!
zsp          0.1159  0.0429     0.0205     0.1999   7539
e_zsp        0.00047 0.0230     0.0        1.7997   7539   <-- !!!
spCl                           GALAXY     GALAXY    7539
```

---

## 4.3 Hay tres cosas mal acá

### 1. Hay un nulo

`uPmag` tiene **7.538** valores buenos, no 7.539. **Una galaxia no tiene magnitud `u`.**

Una galaxia no es nada. Pero el hábito lo es todo: esa galaxia va a **desaparecer en silencio** de
cualquier gráfico que use la banda `u`, y TOPCAT no te lo va a decir. En otro catálogo podrían ser
3.000.

### 2. Las magnitudes contienen basura

```
uPmag   Maximum = 33.147
iPmag   Maximum = 27.632
```

**SDSS tiene un límite de detección de ~22 mag.** Una galaxia de magnitud **33 no existe**.

Son objetos donde el ajuste Petrosiano **falló**: el algoritmo no convergió, devolvió un número
sin sentido, y el catálogo lo guardó igual.

El detalle revelador: la *misma* galaxia puede ser razonable en `r` (máx. 24) y absurda en `u`
(máx. 33). Eso no es física — es un ajuste roto.

### 3. Los errores son peores que las magnitudes

```
e_zPmag   Maximum = 44.086      un error de 44 magnitudes
e_zsp     Maximum =  1.7997     un error de 1,8 en el redshift
```

**Pará y pensá en `e_zsp`.** Tus redshifts van de 0,02 a 0,2. **El error más grande es 1,8** —
unas *doce veces* el mayor redshift de la muestra.

> Eso no es una medición. Es un ajuste que falló y reportó su fracaso honestamente.

---

## 🔥 TRAMPA #8: los flags de calidad son necesarios, no suficientes

Tu consulta contiene **dos** flags de calidad del propio SDSS:

```sql
AND f_zsp = 0     -- "el ajuste del redshift es confiable"
AND clean = 1     -- "la fotometría está limpia"
```

**Y aun así obtuviste magnitudes de 33 y errores de redshift de 1,8.**

> Los flags del propio survey dicen *este objeto está bien*. No lo está.
>
> **Los flags de calidad son necesarios pero no suficientes. Siempre mirá los datos vos misma.**

---

## 4.4 Por qué esto destruiría tu gráfico

Una galaxia con `uPmag = 33` y `rPmag = 17` da un color de **`u−r = 16`**. Una galaxia real tiene
`u−r` entre aproximadamente **0,5 y 3,5**.

**Ese único punto estira tu eje Y de 0–3,5 a 0–16.** Toda tu señal se aplasta en una franja
delgada y no ves nada.

**Probalo.** Ploteá con la basura adentro, mirá el desastre, y volvé.

---

## 4.5 Row subsets — limpiar sin borrar

Un **subset** **no borra filas**. Crea una *etiqueta* que marca qué filas cumplen una condición.
Es reversible, y podés tener varios a la vez y compararlos.

**`Views → Row Subsets`** → **`New Subset`**

| Campo | Valor |
|---|---|
| **Name** | `good` |
| **Expression** | `uPmag < 22 && gPmag < 22 && rPmag < 21 && iPmag < 22 && zPmag < 22 && (uPmag-rPmag) > 0.5 && (uPmag-rPmag) < 4` |

**Resultado: sobreviven 6.851 de 7.539. 688 galaxias — el 9,1% — eran inutilizables.** No es un
error de redondeo.

### ⚠️ Las expresiones de TOPCAT no son SQL

Usan una sintaxis tipo **Java**:

| Operador | Significado |
|---|---|
| `&&` | Y (AND) |
| `\|\|` | O (OR) |
| `!` | NO (NOT) |
| `==` | igual (¡**doble**!) |
| `!=` | distinto |

**Buena noticia:** a diferencia de ADQL, las expresiones de TOPCAT **sí** distinguen mayúsculas.
`uPmag` significa `uPmag` y nada más.

### Dos tipos de corte — y por qué necesitás los dos

**Los cortes de magnitud** (`< 22`) eliminan objetos más débiles de lo que SDSS puede detectar.

**Los cortes de color** (`0.5 < u−r < 4`) son los que realmente te salvan. Pensá en una galaxia
con `uPmag = 21.9` (basura, pero bajo el corte) y `rPmag = 17` (bien). **Pasa el filtro de
magnitud** — y te entrega `u−r = 4.9`.

> **Filtrar las magnitudes no alcanza. Tenés que filtrar la cantidad que efectivamente ploteás.**

### 🎓 Ejercicio

Construí un segundo subset usando los **errores**:

```
e_zsp < 0.001 && e_uPmag < 0.5 && e_rPmag < 0.2
```

¿Cuántas sobreviven? ¿Sigue apareciendo la bimodalidad? **¿Tu conclusión depende de qué estrategia
de limpieza elegiste?**

Si depende, eso es importante. Si no depende, *también* es importante — tu resultado es robusto.

---

## 4.6 Leer `Rows: 7539 (6851 apparent)`

```
Rows:        7.539 (6.851 apparent)
Row Subset:  good
```

- **`7.539`** — las filas que la tabla *tiene*. No se borró nada.
- **`6.851 apparent`** — las filas que TOPCAT te *va a mostrar*.

### ⚠️ La trampa clásica

**Todo lo que hagas ahora — gráficos, estadísticas, crossmatches — usa solo el subset activo.**

Dentro de media hora vas a obtener un número que no tiene sentido, y va a ser porque te olvidaste
de que había un subset activo.

> **Si un resultado te sorprende, lo primero que mirás es qué subset está activo.**

**Verificá:** volvé a `Column Statistics`, poné `Subset for calculations: good`. El máximo de
`uPmag` ahora debería estar bajo 22, no en 33. Y `nGood` debería ser **6.851 en todas las
columnas**.

---

## 4.7 Magnitud absoluta — hacer que el eje X sea físico

La magnitud aparente **no es una propiedad de la galaxia**. Depende de la distancia. Un eje en `r`
aparente mezcla "galaxias intrínsecamente brillantes lejos" con "galaxias débiles cerca". Eso es
geometría, no física.

Tenés `zsp`. Usalo.

$$M_r = m_r - \mathrm{DM}(z) - K_r(z,\ u{-}r)$$

### Primero: explorá lo que TOPCAT ya tiene

**`Help → Available Functions`** abre el **Function Browser**:

| Clase | Contiene |
|---|---|
| **`Distances`** | `luminosityDistance(z, H0, omegaM, omegaLambda)`, `lookbackTime`, `zToAge`. Basado en **Hogg (2000)**, astro-ph/9905116. |
| **`KCorrections`** | `kCorr(filter, redshift, colorType, colorValue)`. Basado en **Chilingarian, Melchior & Zolotukhin (2010)**, válido para **0 < z < 0,5**. |

> **Esta ventana es la pestaña `Columns`, pero para operaciones.** El mismo hábito: **explorá
> antes de asumir.** No escribas a mano una aproximación de algo que la herramienta ya hace bien.

Constantes que necesitás: **`KCF_r`** (el filtro r), **`KCC_ur`** (el color u−r de SDSS).

### Creá la columna

**`Views → Column Info`** → **`Columns → New Synthetic Column`** (o el botón **`f(x)`**).

> ⚠️ El menú `Columns` vive **dentro de la ventana `Table Columns`**, no en la barra de menú
> principal.

| Campo | Valor |
|---|---|
| **Name** | `Mr` |
| **Units** | `mag` |
| **Expression** | `rPmag - (5*log10(luminosityDistance(zsp, 70, 0.3, 0.7)) + 25) - kCorr(KCF_r, zsp, KCC_ur, uPmag - rPmag)` |

| Término | Significado |
|---|---|
| `luminosityDistance(zsp, 70, 0.3, 0.7)` | $d_L$ en **Mpc**; ΛCDM plana, $H_0=70$, $\Omega_m=0.3$, $\Omega_\Lambda=0.7$ |
| `5*log10(d_L) + 25` | el **módulo de distancia** (el `+25` convierte Mpc a la escala de 10 pc) |
| `kCorr(KCF_r, zsp, KCC_ur, uPmag-rPmag)` | la **corrección K** en `r`, usando el color `u−r` observado |

**Verificalo:** `Mr` debería ir de aproximadamente **−24 a −17**. Valores positivos significan
que algo salió mal.

### Por qué la corrección K importa más de lo que parece

La corrección K **depende del color de la galaxia**. Una roja y una azul al mismo redshift reciben
correcciones *distintas*.

**Si la omitís, introducís un error correlacionado con el eje mismo que querés medir.** Estarías
desplazando tus dos poblaciones en cantidades diferentes — distorsionando la separación que querés
estudiar.

---

## 4.8 Lo que *no* corregimos

| No aplicado | Tamaño | ¿Sesga el resultado? |
|---|---|---|
| **Extinción galáctica** | ~0,05–0,1 mag en `r`, a la alta latitud galáctica de Stripe 82 | **Aproximadamente un offset constante.** Desplaza el diagrama; no distorsiona su estructura interna. |
| **Función de selección** | Grande | **Sí.** SDSS es limitada en flujo, y las galaxias rojas son más débiles a igual masa — se pierden primero. |
| **Efectos de apertura** | Pequeño (Petrosian está diseñada para minimizarlos) | Levemente |

> **Omitir la extinción degrada tu precisión. Omitir la corrección K sesga tu resultado.**
> No son el mismo tipo de error.

---

## 4.9 El diagrama color-magnitud

**`Graphics → Plane Plot`**

| Campo | Valor |
|---|---|
| **X** | `Mr` |
| **Y** | `uPmag - rPmag` |
| **Axes** | tildá **`X Flip`** — $M_r$ es negativa; querés las brillantes a la izquierda |
| **Subset** | `good` |

Te sale una mancha roja sólida. **Ese es el problema.** Con ~6.800 puntos opacos dibujados uno
encima del otro, la región densa se satura y perdés toda la información de densidad.

### Agregá contornos

Panel de abajo → pestaña **`Form`** → botón verde **`+ Forms`** → **`Contour`**.

**Ahora sí lo ves: dos picos separados.**

- **Pico superior:** `u−r ≈ 2,7`, `Mr ≈ −21,3` → **LA SECUENCIA ROJA**
- **Pico inferior:** `u−r ≈ 1,7`, `Mr ≈ −20,3` → **LA NUBE AZUL**
- **Entre ellos:** contornos escasos → **EL GREEN VALLEY**

---

## 4.10 🚨 Los dos parámetros de los contornos — y el peligro

### `Level Count`

Cuántas curvas de nivel dibujar. Como un mapa topográfico: cada línea une puntos de igual
densidad.

- **Pocas (3–5):** estructura gruesa. **Los dos picos se destacan.**
- **Muchas (15–20):** detalle fino, pero el gráfico se satura y el ojo pierde la estructura
  global.

### `Smoothing` — el peligroso

El ancho del kernel de suavizado. Tus datos son puntos discretos; para dibujar contornos, TOPCAT
primero estima una densidad *continua* promediando cada punto con sus vecinos.

| Smoothing | Efecto |
|---|---|
| **Bajo (5–10)** | Contornos rugosos y ruidosos. **Aparecen picos falsos** por fluctuaciones estadísticas. |
| **Medio (~50)** | Equilibrado. La estructura real emerge; el ruido se suprime. |
| **Alto (200+)** | **Todo se funde en una sola mancha.** ⚠️ **La bimodalidad desaparece.** |

### Probalo. Ahora mismo.

1. Poné `Smoothing` en **200** → los dos picos **se fusionan en uno**. Concluirías que no hay
   bimodalidad.
2. Ponelo en **5** → aparecen grumos por todos lados. Concluirías que hay cinco poblaciones.

**Ninguna de las dos conclusiones es correcta.** Ambas son artefactos de un parámetro que elegiste
*vos*.

> **Si tu resultado depende críticamente de un parámetro de visualización, no es un resultado.**
> Verificalo con un método que no dependa de ese parámetro.

---

## 4.11 La verificación independiente: un histograma 1-D

**`Graphics → Histogram`** → **X = `uPmag - rPmag`**, subset `good`.

**Sin suavizado. Sin kernel. Sin parámetros libres.** Solo conteos en bins.

- **Pico 1** en `u−r ≈ 1,75` → **nube azul**
- **Un valle** en `u−r ≈ 2,3` → **green valley**
- **Pico 2** en `u−r ≈ 2,65` → **secuencia roja**

**Dos jorobas y un valle, directo de los conteos crudos.**

**Esta es la verificación.** La bimodalidad de los contornos *no* era un artefacto del suavizado —
está en los datos.

---

## 4.12 Lo que acabás de demostrar

> Las galaxias **no** se distribuyen continuamente en color. **Vienen en dos familias.**
>
> **Las azules** están formando estrellas activamente — las estrellas jóvenes y calientes emiten
> en el UV, así que `u` es brillante y `u−r` es chico.
>
> **Las rojas** dejaron de formar estrellas — solo quedan estrellas viejas, así que `u−r` es
> grande.
>
> **Y el valle entre ellas está poco poblado** — lo que significa que la transición de una a la
> otra es **rápida**. Las galaxias no se quedan mucho tiempo en el medio.

Un resultado científico real, obtenido en noventa minutos, sin escribir una sola línea de código.

---

## 4.13 ⚠️ Una cosa que NO podés concluir

El pico azul es **más alto** que el rojo.

**Esto *no* significa que haya menos galaxias rojas en el universo.** Significa que hay menos en
*tu muestra* — y tu muestra tiene un **sesgo de selección**.

SDSS es limitada en flujo. A masa estelar fija, las galaxias rojas son **más débiles** que las
azules, así que se pierden primero de un survey limitado en flujo.

> **Cualquier conteo relativo entre poblaciones está sesgado por la función de selección.**
> Hacerlo bien requiere calcular volúmenes máximos ($V_{max}$) y pesar cada galaxia.
> Eso es un paper, no una clase.

---

## ✅ Checkpoint

- [ ] Encontraste la fotometría basura **y** los errores absurdos **antes** de plotear
- [ ] Construiste el subset `good` — **sobreviven 6.851 de 7.539**
- [ ] Cortaste en **color**, no solo en magnitud
- [ ] `Mr` va de aproximadamente −24 a −17
- [ ] El gráfico de contornos muestra **dos picos**
- [ ] El histograma los **confirma independientemente**
- [ ] Podés nombrar **tres cosas que no corregiste**
- [ ] Podés explicar por qué `f_zsp = 0` y `clean = 1` **no alcanzaron**

---
---

# Parte 5 — Hacer clic en una galaxia: de un número a una imagen

Tenés dos picos en un histograma. Hasta ahora son pura estadística.

**Ahora miremos qué son esos números en realidad.**

---

## 5.1 Activation Actions — la función que nadie encuentra

TOPCAT puede **hacer algo cuando hacés clic en una fila** — en una tabla, o en un punto de un
gráfico.

**`Views → Activation Actions`** (o el botón amarillo **⚡**).

```
☑ Use Sky Coordinates in      <-- activa por defecto
☐ Send Sky Coordinates         (SAMP -> Aladin, DS9, ...)
☐ Display HiPS cutout          <-- lo que queremos
☐ Send HiPS cutout
☐ Delay
☐ Execute code
☐ Run system command
☐ Send row index
```

### ⚠️ La trampa de esta ventana

**El panel `Configuration` de la derecha muestra la configuración de la acción que está
SELECCIONADA — no de la que está TILDADA.**

Tenés que hacer **dos cosas distintas**:

1. **Tildar el checkbox** de `Display HiPS cutout` → esto la *habilita*
2. **Hacer clic en su nombre** → esto la *selecciona*, para poder configurarla

Si solo la tildás, te vas a quedar mirando sin entender por qué el panel derecho sigue mostrando
la configuración de otra cosa.

---

## 5.2 Configurar el cutout

| Campo | Valor | Notas |
|---|---|---|
| **RA Column** | `RA_ICRS` | debería estar completo |
| **Dec Column** | `DE_ICRS` | ídem |
| **HiPS Survey** | `SDSS9/color-alt` | clic en **`Select`** para explorar |
| **Field of View** | **`0.02`** grados | ver abajo |
| **Size in Pixels** | `300` | está bien así |

### ⚠️ Configurá bien el Field of View

Por defecto suele estar en **`0.2` grados = 12 arcmin**. **Eso es enorme.**

Una galaxia a z ≈ 0,1 tiene un tamaño angular de aproximadamente **10–20 arcsec**. Con 0,2° vas a
ver un campo amplio con decenas de galaxias, y la tuya va a ser un puntito en el centro.

**Usá `0.02` grados** (≈ 1,2 arcmin). La galaxia va a llenar el cuadro.

> El tamaño angular depende del redshift: una galaxia a z=0,05 se va a ver grande, una a z=0,19 va
> a ser diminuta. Un FoV fijo es un compromiso.

---

## 5.3 Ahora hacé clic en el gráfico

Volvé a tu **diagrama color-magnitud** y hacé clic directamente sobre un punto.

**Se abre una ventanita con la imagen SDSS de esa galaxia.**

El panel `Results` registra cada clic. Si hacés clic rápido vas a ver entradas `CANCELLED` — una
petición nueva reemplazó a una pendiente. Normal.

---

## 5.4 🔬 El experimento

### Hacé clic en una galaxia del pico ROJO — `u−r ≈ 2,7`, `Mr ≈ −21`

> **Una elíptica.** Amarillenta. Lisa. Un gradiente suave desde el centro hacia afuera.
> **Sin estructura alguna.**

### Hacé clic en una galaxia del pico AZUL — `u−r ≈ 1,7`, `Mr ≈ −20`

> **Estructura.** Un núcleo brillante rodeado de un disco más difuso y azulado. Asimetría.
> Extensión.

### Así se tiene que ver

![Diagrama color-magnitud con dos galaxias](images/cmd-two-galaxies.png)

**Cutout de arriba:** clic en el pico rojo — elíptica lisa.
**Cutout de abajo:** clic en el pico azul — disco con estructura.

**El mismo gráfico. Dos clics. Dos tipos de galaxia distintos.**

---

## 5.5 Lo que acabás de demostrar

> Seleccionaste estas galaxias por **un número** — el color `u−r`, calculado a partir de dos
> magnitudes de un catálogo.
>
> Nunca miraste una imagen. Nunca dijiste nada sobre la forma.
>
> **Y aun así, los dos picos del histograma corresponden a dos tipos de galaxia que
> *se ven distintos*.**
>
> El color mide la población estelar. La población estelar refleja la historia de formación
> estelar. Y la historia de formación estelar está ligada a la estructura.
>
> **Los picos no eran estadística. Eran física.**

---

## 5.6 ⚠️ Pero no lo sobrevendas

La galaxia azul que cliqueaste probablemente **no es una espiral de manual con brazos de gran
diseño**. Es más bien un disco difuso con bulbo.

- A z ≈ 0,1, el **seeing** de SDSS (~1,4″) borronea la estructura fina. Los brazos se pierden.
- El color mide **poblaciones estelares**, no morfología. La correlación es fuerte — pero **no es
  una ley**.
- **Existen espirales rojas** (apagadas, pero con disco). **Existen elípticas azules** (con
  formación estelar reciente).

> **El color predice la morfología. No la determina.**

---

## 5.7 Otras cosas que pueden hacer las Activation Actions

| Acción | Uso |
|---|---|
| **`Send Sky Coordinates`** | Vía **SAMP** — hace que Aladin o DS9 salten a ese objeto, en vivo, con catálogos superpuestos. |
| **`Send HiPS cutout`** | Manda la imagen a otra aplicación. |
| **`Execute code`** | Ejecuta una expresión arbitraria sobre la fila cliqueada. |
| **`Run system command`** | Lanza un programa externo con los valores de la fila como argumentos. |
| **`View URL`** | Abre el navegador en una URL construida a partir de la fila — por ejemplo, la página de SkyServer de ese objeto. |

**Así es como se inspeccionan los outliers.** Cada vez que un punto de un gráfico se vea raro,
**cliqueálo y mirá el objeto.** La mitad de las veces es un par blended, un artefacto, o un rastro
de satélite — y de los números solos nunca lo habrías sabido.

> **La herramienta de debugging más subutilizada en astronomía es mirar la imagen.**

---

## ✅ Checkpoint

- [ ] `Display HiPS cutout` está **tildada** *y* **seleccionada** (son cosas distintas)
- [ ] El Field of View es **0.02**, no 0.2
- [ ] Hacer clic en un punto del CMD abre una imagen
- [ ] Galaxia del pico rojo → **elíptica lisa**
- [ ] Galaxia del pico azul → **estructura**
- [ ] Podés explicar por qué la correlación es **fuerte pero no absoluta**

---
---

# Parte 6 — Crossmatch: ¿el color realmente predice la morfología?

En la Parte 5 cliqueaste dos galaxias y viste que la roja era lisa y la azul tenía estructura.
**Dos galaxias son una anécdota.**

Ahora probémoslo con **tres mil**.

**El detalle:** la morfología no está en SDSS. Vive en un catálogo completamente distinto,
producido por otra gente con otro método. Para responder la pregunta, **no te queda otra que hacer
un crossmatch.**

Este es el argumento entero del Observatorio Virtual, en un solo ejercicio.

---

## 6.1 El catálogo: Galaxy Zoo 1

**Galaxy Zoo** le pidió a cientos de miles de voluntarios que miraran imágenes de SDSS y las
clasificaran a ojo. Galaxy Zoo 1 (Lintott et al. 2011, MNRAS 410, 166) es el original.

En VizieR la tabla es:

```
J/MNRAS/410/166/galaxies
```

> **Para encontrarla vos misma:** en la ventana TAP, tildá **`Name` y `Descrip`**, buscá
> `galaxy zoo`, y expandí `J_MNRAS`. (Acordate de la Trampa #1 — con `Name` solo no encontrás
> nada.)

### Las columnas que importan

| Columna | Tipo | Descripción |
|---|---|---|
| `RAJ2000`, `DEJ2000` | DOUBLE (deg) | Coordenadas SDSS |
| **`fE`** | SMALLINT | **Flag `[0/1]` "Elliptical"** |
| **`fS`** | SMALLINT | **Flag `[0/1]` "Spiral"** |
| **`fU`** | SMALLINT | **Flag `[0/1]` Uncertain** |
| `objID` | CHAR(18) | Identificador de objeto de SDSS |
| `pDK`, `pDKm` | DOUBLE | Fracción de votos "Don't Know" |
| `Nt`, `N`, `Nt1`, `Nt2` | SMALLINT | Conteos de votos |

---

## 6.2 ⚠️ Qué significa realmente `fS` — leé esto antes de escribir conclusiones

**VizieR etiqueta `fS` como "Spiral". No lo tomes al pie de la letra.**

Galaxy Zoo 1 **no** le preguntó a los voluntarios *"¿esto es una espiral?"*. Les preguntó, más o
menos:

> *"¿Este objeto es **liso y redondeado**, o tiene **algo de estructura / características**?"*

Así que la **clase `fS` es realmente "no-elíptica, con estructura visible"**. Contiene:

- espirales genuinas
- **discos sin brazos claros**
- **lenticulares (S0)**
- **discos de canto (edge-on)**
- **irregulares**

**Eso no es lo mismo que "espiral" en el sentido morfológico estricto.**

> **Sé precisa en lo que afirmás.** Estás testeando si el color predice
> **"liso vs. estructurado"** — no si predice el tipo de Hubble.
>
> Sigue siendo un resultado real e interesante. Simplemente es un resultado *distinto*.

### Y los flags son un *corte*, no una medición

`fE` y `fS` son flags binarios derivados de **fracciones de voto** — un objeto se marca solo
cuando una gran mayoría de voluntarios estuvo de acuerdo (y después de corregir por sesgo de
clasificación).

**Las galaxias ambiguas quedan con `fU = 1`** y no se marcan como **ninguna de las dos**.

### 🎓 Ejercicio (hacelo antes de sacar conclusiones)

**¿Cuántas galaxias tienen `fU = 1`?**

Si ese número es grande, entonces los objetos que *sí* estás clasificando son solo los
**visualmente más obvios** — y **estás viendo una correlación más limpia de la que realmente
existe.**

Seleccionaste por no-ambigüedad. Eso es un sesgo, y es un sesgo *hacia tu propia conclusión*.

---

## 6.3 Cargar Galaxy Zoo

**`VO → Table Access Protocol (TAP) Query`** → `TAPVizieR` → `Use Service`

Pegá en la caja **`ADQL Text`**:

```sql
SELECT TOP 100000
  RAJ2000, DEJ2000,
  fE, fS, fU
FROM "J/MNRAS/410/166/galaxies"
WHERE RAJ2000 BETWEEN 320 AND 340
  AND DEJ2000 BETWEEN -1.25 AND 1.25
```

**`Run Query`**

| Línea | Por qué |
|---|---|
| `RAJ2000 BETWEEN 320 AND 340` | **La misma región de Stripe 82.** No bajes todo el cielo. |
| `DEJ2000 BETWEEN -1.25 AND 1.25` | ídem |
| `fE, fS, fU` | Los tres flags morfológicos |
| *(sin filtro sobre los flags)* | **Deliberado** — queremos contar las inciertas |

> **Esperá muchas más galaxias que tus 7.539.** Galaxy Zoo no requiere espectro; tu muestra de
> SDSS sí. **Esta asimetría importa — ver §6.6.**

---

## 6.4 ⚠️ Antes de cruzar: VERIFICÁ TU SUBSET

Andá a la ventana principal, seleccioná tu **tabla de SDSS**, y mirá **`Row Subset:`**.

Si dice **`good`**, TOPCAT va a cruzar solo esas **6.851** filas — no las 7.539.

**Eso es lo que queremos** — no tiene sentido buscar la morfología de galaxias cuya fotometría
está rota.

**Pero hacelo a propósito, no por accidente.** Es exactamente la trampa de §4.6: *si un resultado
te sorprende, lo primero que mirás es qué subset está activo.*

---

## 6.5 El crossmatch

**`Joins → Pair Match (2 tables)`**

| Campo | Valor |
|---|---|
| **Algorithm** | **`Sky`** |
| **Max Error** | **`2`** arcsec |
| **Table 1** | tu tabla de SDSS |
| → RA column | `RA_ICRS` |
| → Dec column | `DE_ICRS` |
| → Row Subset | **`good`** ⚠️ |
| **Table 2** | Galaxy Zoo |
| → RA column | `RAJ2000` |
| → Dec column | `DEJ2000` |
| → Row Subset | `All` |
| **Match Selection** | `Best match, symmetric` |
| **Join Type** | **`1 and 2`** ⚠️ **crítico — ver abajo** |

**`Go`**

---

## 🔥 TRAMPA #9: el Join Type te va a mentir en silencio

**Si te equivocás acá, TOPCAT te da una tabla que se ve perfecta y es inútil.**

| Join Type | Qué devuelve |
|---|---|
| **`1 and 2`** | **Solo las filas que matchearon.** ✅ Lo que querés. |
| `All from 1` | **Todas** las filas de la tabla 1, con las columnas de la tabla 2 **vacías** donde no hubo match |
| `All from 2` | El espejo |
| `1 or 2` | Todo de ambas |

### Cómo detectar el error

Si usás `All from 1`, tu resultado tiene **exactamente 6.851 filas** — igual que la entrada.

**Parece una tasa de match del 100%.** No lo es. Abrí la tabla y vas a ver que la mitad de las
columnas de Galaxy Zoo están **en blanco**.

> **Un crossmatch que devuelve el mismo número de filas que la tabla de entrada es una señal de
> alarma, no de éxito.**

**Con `1 and 2` deberías obtener unas 3.053 filas.**

---

## 6.6 🚨 Leé la tasa de match. Te está diciendo algo.

| | |
|---|---|
| Galaxias SDSS (`good`) | **6.851** |
| Con morfología en Galaxy Zoo | **3.053** |
| **Tasa de match** | **≈ 45%** |

**Más de la mitad de tus galaxias no tienen morfología.**

### Esto no es un fallo del crossmatch

La astrometría está bien (ver §6.7). El radio está bien. El método funcionó.

**Galaxy Zoo simplemente no clasificó todas las galaxias de SDSS.** Tiene su **propia función de
selección** — límites de magnitud, límites de tamaño angular, criterios de calidad de imagen.

> **Cruzar dos catálogos no te da su unión.**
> **Te da la INTERSECCIÓN de dos funciones de selección.**

### ⚠️ Y ahora la pregunta que nadie hace

**¿Esa pérdida del 55% es aleatoria?**

**Casi seguro que no.** Galaxy Zoo necesita *resolver* una galaxia para clasificarla. Así que
probablemente perdiste preferentemente:

- las **angularmente más chicas** → las **más lejanas**
- las **más débiles**

**Y eso está correlacionado con el color**, porque las galaxias rojas son más débiles a masa
estelar fija.

> **Tu muestra cruzada no es una submuestra aleatoria de tu muestra original.**
> **Es una submuestra sesgada — y el sesgo va en la dirección de tu resultado.**

### 🎓 Ejercicio

Ploteá un **histograma de `zsp`** para la tabla cruzada, y compará con el mismo histograma de la
muestra completa.

**¿El pico se corrió hacia redshift más bajo?** Si sí, confirmaste el sesgo.

---

## 6.7 El paso que nadie hace: el histograma de separaciones

**`Graphics → Histogram`** sobre la tabla cruzada → **X = `Separation`**

### Lo que deberías ver

```
0,0 – 0,1"  →  ~2.300 galaxias   (75%)
0,1 – 0,2"  →    ~720            (24%)
0,2 – 0,5"  →     ~30
> 0,5"      →  casi nada
```

**El 99% de los matches cae dentro de 0,2 arcsec.** Una caída brutal e inmediata.

**Los matches son reales.** Sin ambigüedad.

### Por qué es tan limpio acá

**Ambos catálogos usan la astrometría de SDSS.** No estás cruzando dos telescopios distintos — es
**el mismo objeto, medido una vez**. Las diferencias son de redondeo y de época de reducción.

### ⚠️ Pero mirá la cola

Hay unos poquitos matches en **1,8″, 2,7″, 3,2″**.

En una distribución donde el 99% cae bajo 0,2″, un objeto a 3″ **probablemente no es la misma
galaxia** — es un **vecino** que cayó dentro de tu radio de búsqueda.

> **Tu radio de búsqueda fue demasiado generoso.**
> Este histograma te dice que **0,5″ habría alcanzado** — y habría eliminado los falsos positivos
> sin perder ni un match real.

**Esta es una iteración normal:** elegís un radio generoso, mirás el histograma, **lo ajustás**.
Nadie acierta a la primera.

### El contraste que hay que recordar

| Forma del histograma | Qué significa |
|---|---|
| **Caída abrupta a separación chica** | Los matches son reales. Confiá. |
| **Plano hasta el radio de búsqueda** | **Estás casando ruido.** Los "matches" son vecinos al azar. |

**Siempre miralo. Cuesta diez segundos.**

---

## 6.8 El premio: superponer morfología en el CMD

Sobre tu **tabla cruzada**, creá dos subsets:

**`Views → Row Subsets → New Subset`**

| Name | Expression |
|---|---|
| `elliptical` | `fE == 1` |
| `spiral` | `fS == 1` |

*(acordate: `==` es doble, y ver §6.2 — `fS` realmente significa "estructurada", no "espiral")*

Ahora ploteá:

**`Graphics → Plane Plot`**

| Campo | Valor |
|---|---|
| **X** | `Mr` |
| **Y** | `uPmag - rPmag` |
| **Axes** | `X Flip` |
| **Layers** | tu muestra completa como **contornos**, más los dos subsets como **puntos de colores** |

---

## 6.9 Lo que ves

**Las galaxias `fE` caen en el pico ROJO.** `u−r ≈ 2,8–3,5`, y son las más luminosas
(`Mr ≈ −22 a −23`).

**Las galaxias `fS` caen en el pico AZUL.** `u−r ≈ 1,8–2,3`, y menos luminosas.

> **Vos dividiste las galaxias con un número.**
> **Un catálogo independiente las dividió con ojos humanos mirando fotos.**
> **Obtuvieron la misma división.**

Esa es la confirmación cuantitativa de lo que viste a ojo con dos cutouts en la Parte 5.

---

## 6.10 🚨 Ahora mirá las excepciones — ahí está la ciencia

**Hay puntos rojos abajo.** Galaxias `fE` en `u−r ≈ 2,2` — en plena zona azul.

**Hay puntos azules arriba.** Galaxias `fS` en `u−r ≈ 3,0` — en plena secuencia roja.

**Esas son DISCOS ROJOS y ELÍPTICAS AZULES.**

**No son errores.** Son objetos reales, y son **interesantes**:

| Objeto | Qué es probablemente |
|---|---|
| **Disco rojo** | Un disco **apagado** (quenched) — dejó de formar estrellas pero conservó su estructura. O una espiral de canto, enrojecida por su propio polvo. |
| **Elíptica azul** | Una elíptica con **formación estelar reciente** — quizás un merger reciente, o gas acretado. |

> **El grueso de la población es aburrido. Las excepciones son los papers.**

---

## 6.11 La conclusión real

> El color y la morfología **se correlacionan fuertemente, pero no son la misma cosa.**
>
> **El color** mide la **población estelar**: ¿hay estrellas jóvenes?
> **La morfología** mide la **estructura dinámica**: ¿hay disco?
>
> **Que se correlacionen es un hecho físico profundo** — significa que los procesos que apagan la
> formación estelar están ligados a los que destruyen los discos.
>
> **Y que *no* se correlacionen perfectamente es igual de profundo** — significa que se pueden
> desacoplar. Una galaxia puede apagarse sin perder su disco.

---

## ✅ Checkpoint

- [ ] Sabés que **`fS` significa "estructurada", no estrictamente "espiral"** (§6.2)
- [ ] Contaste las galaxias con **`fU = 1`** y sabés qué le hace ese sesgo a tu conclusión
- [ ] Usaste **`Join Type: 1 and 2`**, y podés explicar qué habría hecho `All from 1`
- [ ] Tasa de match ≈ **3.053 / 6.851 = 45%**, y podés explicar **por qué no es 100%**
- [ ] Ploteaste el **histograma de `Separation`** y **cae abruptamente**
- [ ] Podés decir qué radio de búsqueda **deberías** haber usado, y por qué
- [ ] Las elípticas caen en el pico rojo; las estructuradas en el azul
- [ ] **Encontraste las excepciones**, y podés decir qué podrían ser

---
---

# Apéndice — Las doce trampas

Cada una de estas **produce un resultado que se ve bien**. Ninguna lanza un error.

## Encontrar los datos

| # | Trampa | Qué te cuesta |
|---|---|---|
| **1** | Buscar en `Name` pero no en `Descrip` | Concluís que los datos no existen. Sí existen — 20× más. |
| **2** | Asumir dónde vive un catálogo | SDSS está en `large_tables`, no en `II_photometry` |

## Elegir las columnas

| # | Trampa | Qué te cuesta |
|---|---|---|
| **3** | `zsp` / `zph` / `zmag` / `zpmag` / `zPmag` | Cinco columnas, a una letra de distancia, cinco significados distintos |
| **4** | Magnitudes PSF para galaxias | Una "secuencia roja" que es en parte un artefacto de apertura |
| **5** | `class` en vez de `spCl` | Cuásares contaminando tu muestra de galaxias |
| **6** | Olvidar `f_zsp = 0` | Redshifts basura en tu muestra |
| **7** | Identificadores sin comillas en ADQL | Ambigüedad — **o peor, selección silenciosa de la columna equivocada** |

## Confiar en los datos

| # | Trampa | Qué te cuesta |
|---|---|---|
| **8** | Confiar solo en los flags de calidad | `clean = 1` **y** `f_zsp = 0`, y *aun así* magnitudes de 33 y errores de redshift de 1,8 |
| **12** | Confiar en la etiqueta de una columna | `fS` dice "Spiral". En realidad significa "no lisa". |

## Crossmatch

| # | Trampa | Qué te cuesta |
|---|---|---|
| **9** | `Join Type` equivocado | Una tabla con el número "correcto" de filas y la mitad de las columnas vacías — **parece un match del 100%** |
| **10** | No mirar nunca el histograma de separaciones | No podés distinguir un match real de un vecino al azar |
| **11** | Asumir que un crossmatch preserva tu muestra | **Obtenés la intersección de dos funciones de selección — y la pérdida no es aleatoria** |

---

## Lo único que hay que recordar

> **La base de datos nunca te va a avisar que hiciste la pregunta equivocada.**
>
> **El gráfico nunca te va a avisar que ploteaste la columna equivocada.**
>
> **El flag nunca te va a avisar que se le escapó algo.**
>
> **El crossmatch nunca te va a avisar que tiró a la basura la mitad de tu muestra — y que la
> mitad que tiró no era aleatoria.**
>
> **Ese es tu trabajo.**

---

## Lo que realmente aprendiste

No TOPCAT. TOPCAT son botones; eso se busca.

Lo que aprendiste es un **hábito de sospecha**, aplicado en un orden específico:

1. **Explorá el esquema antes de consultar.** Los nombres mienten; las descripciones y los UCDs
   no.
2. **Mirá las estadísticas antes de plotear.** Mínimos, máximos, nulos.
3. **Filtrá la cantidad que efectivamente ploteás** — no solo sus ingredientes.
4. **Verificá cada resultado con un método que no comparta los supuestos del primero.**
   Los contornos tienen un parámetro de suavizado; un histograma no.
5. **Cliqueá los outliers y mirá la imagen.** Es la herramienta de debugging más subutilizada en
   astronomía.
6. **Preguntá qué perdió tu muestra, y si la pérdida fue aleatoria.** Casi nunca lo es.
7. **Decí en voz alta qué no corregiste.**

Un pipeline que corre no es un pipeline que está bien.
