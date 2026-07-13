# 2. Exploring a TAP service: finding the table *and the columns* you need

You have selected **TAPVizieR** and clicked `Use Service`. You are now looking at the
service's **schema browser** — and you cannot write a single line of ADQL until you know
what lives inside.

> **The habit this section teaches:**
> *Never guess a table name. Never guess a column name. Look them up.*
>
> A query written from memory of some archive's website will fail — and worse, it might
> **run perfectly and give you the wrong answer**.

---

## 2.1 The layout

```
┌──────────────────────┬────────────────────────────────────┐
│  Find: [        ]    │  ○Service ○Schema ○Table ○Columns  │
│  ☑Name ☐Descrip      │                                    │
│                      │   (metadata about whatever is      │
│  ▼ TAPVizieR (63884) │    selected on the left)           │
│    ▸ I_astrometry    │                                    │
│    ▸ II_photometry   │                                    │
│    ▸ J_ApJ (10721)   │                                    │
│    ▸ large_tables    │                                    │
├──────────────────────┴────────────────────────────────────┤
│  ADQL Text:  (you write your query here)                  │
│                        [ Run Query ]                      │
└───────────────────────────────────────────────────────────┘
```

- **Left:** the tree of *schemas* (folders) and *tables*.
- **Right:** metadata about whatever you select on the left.
- **Bottom:** where you type ADQL.

---

## 2.2 How VizieR organises its 63,884 tables

| Folder | Contents |
|---|---|
| `I_astrometry` | Astrometry (Gaia, Hipparcos, ...) |
| `II_photometry` | Photometry |
| `III_spectro` | Spectroscopy |
| `VI_misc`, `VII_nonstellar`, `VIII_radio`, `IX_HE` | Miscellaneous, non-stellar, radio, high-energy |
| `J_AA`, `J_ApJ`, `J_MNRAS`, ... | **Tables from individual papers**, grouped by journal |
| **`large_tables`** | **"extremely large catalogs"** — the survey-scale catalogues |

`J_ApJ (10721)`: over ten thousand tables published in the *Astrophysical Journal* alone.
**This is VizieR's real value** — if a paper published a table, it is probably here.

---

## 2.3 TRAP #1: `Name` vs `Descrip`

Type `sdss` into `Find:` with only **`Name`** ticked:

```
TAPVizieR (32/63884)     <-- only 32 matches
```

Now also tick **`Descrip`**:

```
TAPVizieR (642/63884)    <-- 642 matches
```

**Twenty times more results, from one checkbox.**

**Why:** with `Name` only, TOPCAT searches the *table identifier* (`V/154/sdss16`). Most
tables do not carry the survey name in their identifier — they carry it in their
**description**.

> **Lesson:** When a search returns nothing, the data almost certainly exist.
> You are searching the wrong field.

---

## 2.4 TRAP #2: where SDSS actually lives

Even with 642 matches, the main SDSS catalogue is **not** in `II_photometry`, and **not**
in any folder called `V_something`. It is in:

```
large_tables (8/148)      Description: "extremly large catalogs"   [typo is theirs]
    "II/294/sdss7"
    "II/306/sdss8"
    "V/139/sdss9"
    "V/147/sdss12"
    "V/154/sdss16"     <-- the one we want
```

VizieR separates survey-scale catalogues (hundreds of millions of rows) from ordinary ones,
because they need different handling server-side.

> **Lesson:** An archive's organisation has an internal logic — but not necessarily *your*
> logic. Explore; do not assume.

Every data release is available (DR7 → DR16). Use the most recent unless you have a reason
not to — and **state in your paper which one you used**.

---

## 2.5 The metadata tabs

Select `"V/154/sdss16"` with a single click, then walk the tabs on the right:

| Tab | What it shows |
|---|---|
| `Service` | Info about the TAP service. Look once, never again. |
| `Schema` | Info about the **folder** (`large_tables`). The level above your table. |
| `Table` | Info about **your table**: exact name (**copy it, never retype**), description, **row count**. |
| **`Columns`** | **Every column, with name, type, unit, description, and UCD. This is the tab that matters.** |
| `FKeys` | Foreign keys — how this table links to others. Advanced. |

> ⚠️ **Check the row count in `Table`.** If it says hundreds of millions, your `WHERE`
> clause had better be selective, or you will wait forever.

---

# 2.6 Reading the `Columns` tab — the heart of the matter

This is where the class is won or lost. Below is what `V/154/sdss16` actually contains.

## The columns we need

| Column | Unit | Description | Use |
|---|---|---|---|
| `RA_ICRS`, `DE_ICRS` | deg | Coordinates (ICRS) | Position |
| **`uPmag` … `zPmag`** | mag | **Petrosian** magnitudes | Photometry |
| `e_uPmag` … `e_zPmag` | mag | Their errors | Quality |
| **`zsp`** | — | **Spectroscopic redshift** (UCD: `src.redshift`) | Redshift |
| `e_zsp` | — | Error on `zsp` | Quality |
| **`f_zsp`** | — | **ZWarning flag** | ⚠️ Critical |
| `zph` | — | Photometric redshift | Alternative |
| `class` | — | `3=galaxy, 6=star` (**photometric**) | Filter |
| **`spCl`** | — | `"GALAXY"`, `"QSO"`, `"STAR"` (**spectroscopic**) | Better filter |
| `mode` | — | `1=primary` | Deduplication |
| `clean` | — | `1=clean photometry` | Quality |
| `SpObjID` | — | Pointer to spectrum, or `0` | Has a spectrum? |

---

## 🔥 TRAP #3: five columns start with `z` and mean five different things

```
zsp     -->  Spectroscopic redshift          <-- what you want
zph     -->  Photometric redshift            <-- a different (noisier) quantity
zmag    -->  MODEL magnitude in z band       <-- a magnitude!
zpmag   -->  PSF magnitude in z band         <-- a different magnitude!
zPmag   -->  PETROSIAN magnitude in z band   <-- yet another magnitude!
```

**`zpmag` and `zPmag` differ by a single capital letter.** One is PSF, the other Petrosian.

**The query will not warn you.** It will run, return numbers, and produce a plot. The plot
will simply be *wrong*.

> This is the single strongest argument for reading the schema. It is not pedantry —
> it is the difference between science and garbage.

---

## 🔥 TRAP #4: PSF magnitudes are wrong for galaxies

The first magnitudes in the list are `upmag, gpmag, rpmag, ipmag, zpmag` — **PSF
magnitudes**. It is tempting to just use them.

**Do not.** A PSF magnitude fits the image's point-spread function to the source.

- For a **star** → perfect. The star *is* a PSF.
- For an **extended galaxy** → **it fits the PSF inside the nucleus and throws away the disc
  and the halo.**

The flux is systematically underestimated, and the size of the error **depends on angular
size** — i.e. on redshift and morphology. You would be injecting a bias correlated with
exactly the variables you want to study.

For colours it is worse: galaxies have colour gradients (red bulges, blue discs), so the PSF
over-weights the bulge. **Your "red sequence" would be partly an aperture artefact.**

### The galaxy magnitudes in SDSS

| Magnitude | What it is | Use for |
|---|---|---|
| **`?Pmag`** (Petrosian) | Aperture set by the light profile, redshift-independent | **The SDSS standard for galaxies** |
| `?mag` (model) | Best fit of a de Vaucouleurs or exponential profile | Also valid |
| `?pmag` (PSF) | ❌ | Stars only |

### ⚠️ Do not mix apertures

If you measure `r` with a Petrosian aperture and `(g−r)` with model magnitudes, **the point
you plot on the CMD does not correspond to any physically consistent object.**

**Pick one system and use it for everything.** In this tutorial: **Petrosian** (`uPmag`
through `zPmag`), magnitude *and* colour.

---

## 🔥 TRAP #5: `class` is not `spCl`

| Column | How it classifies | Reliability |
|---|---|---|
| `class` | **Photometrically** — from the shape in the image | An *estimate* |
| **`spCl`** | **Spectroscopically** — from the actual spectrum | The **truth** |

If the object has a spectrum, **use `spCl`**.

And note: `spCl` distinguishes **`QSO`** from **`GALAXY`**. Photometric `class=3` would
happily hand you quasars along with your galaxies — and a quasar is a point source with a
power-law continuum. It will sit somewhere completely different on your colour–magnitude
diagram and contaminate everything.

---

## 🔥 TRAP #6: `f_zsp` — the flag that separates professionals from beginners

`f_zsp` is SDSS's **ZWarning** flag.

```
f_zsp = 0    -->  the redshift fit is reliable
f_zsp != 0   -->  the fit failed, the spectrum was bad, something went wrong
```

**Without `f_zsp = 0`, your sample contains garbage redshifts.** And, once again, the query
runs perfectly. It does not complain. It simply gives you the wrong answer.

Every survey has flags like this. **Find them. Use them.**

---

## 2.7 Two more quality columns you would never have guessed

Neither appears in any tutorial. You only find them by reading the schema.

| Column | Why it matters |
|---|---|
| **`mode = 1`** | SDSS images overlapping regions. Without this, **the same galaxy appears several times** in your sample. `1 = primary` keeps one detection per object. |
| **`clean = 1`** | SDSS's own composite photometry-quality flag. |

---

## 2.8 A note on UCDs

A **UCD** (Unified Content Descriptor) is a standard label describing what a column *means*,
independent of what it is *called*.

Look at the UCD column: `zsp` and `zph` both carry `src.redshift`. Different names, same
physical meaning. Two archives may call the same quantity `zsp`, `Z_SPEC`, or `redshift` —
but all will tag it `src.redshift`.

**Software can rely on the UCD; it cannot rely on the name.** This is how TOPCAT
automatically knows which columns are coordinates when you run a cross-match without being
told.

---

## 2.9 Checkpoint

Before writing any ADQL, you should be able to state:

- [ ] The **exact table name**, copied from the `Table` tab
- [ ] The **coordinate columns** — `RA_ICRS`, `DE_ICRS`
- [ ] The **redshift column** — `zsp`, and **why not** `zph`, `zmag`, `zpmag`, or `zPmag`
- [ ] The **magnitudes** — `uPmag`…`zPmag`, and **why not** the PSF ones
- [ ] The **classification column** — `spCl`, and **why not** `class`
- [ ] The **quality flags** — `f_zsp`, `mode`, `clean`, and what each protects you from

**If you cannot fill in every line above, do not write the query yet.** Go back to
`Columns`.

---

## The thing to take away

Every one of the six traps above **produces a query that runs without error**.

None raises an exception. None prints a warning. Each one quietly returns the wrong sample —
and you plot it, and it looks perfectly plausible.

> **The database will never tell you that you asked the wrong question.**
> That is your job.

---

## Next

With the real column names — and the real quality flags — in hand, we write our first ADQL
query and pull a clean sample of galaxies from SDSS Stripe 82.
