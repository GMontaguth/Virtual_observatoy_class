# Astronomical Databases and TOPCAT
## A hands-on workshop: finding galaxy bimodality in SDSS

**What you will do:** starting from nothing, you will pull a sample of galaxies from a
public archive, clean it, compute their absolute magnitudes, and discover that galaxies come
in **two distinct families** — one red and dead, one blue and star-forming.

You will then click on a point in your plot and **see the galaxy**.

No code. Ninety minutes.

---

## Where you are going

**This is the end result. This is what you should have on your screen by the end of the
session:**

![Colour-magnitude diagram with two galaxy cutouts](images/cmd-two-galaxies.png)

On the left: the **colour–magnitude diagram**, with contours revealing **two separate peaks**.

On the right, two galaxies you clicked on directly in that plot:

- **Top** — from the **upper peak** (`u−r ≈ 2.7`): a **smooth, yellowish elliptical**.
  No structure.
- **Bottom** — from the **lower peak** (`u−r ≈ 1.7`): a **bluer disc** with visible
  structure and a bright nucleus.

**You will select these galaxies using nothing but a number.** You will never tell the computer
anything about their shape.

And yet the two peaks turn out to be **two kinds of galaxy that look different**.

That is the destination. Now let's get there.

---

## Contents

**[Part 1 — The Virtual Observatory](#part-1--the-virtual-observatory)**
**[Part 2 — Exploring a TAP service](#part-2--exploring-a-tap-service)**
**[Part 3 — Your first ADQL query](#part-3--your-first-adql-query)**
**[Part 4 — Looking at the data, and finding the bimodality](#part-4--looking-at-the-data--and-finding-the-bimodality)**
**[Part 5 — Clicking on a galaxy](#part-5--clicking-on-a-galaxy--from-a-number-to-an-image)**
**[Appendix — The eight traps](#appendix--the-eight-traps)**

---
---

# Part 1 — The Virtual Observatory

## 1.1 The problem it solves

Before the VO, every astronomical archive was an island:

- SDSS had its own website, its own format, its own query interface.
- 2MASS had a different one.
- Chandra had another.
- The catalogue from a 1998 paper lived as a `.txt` file on somebody's personal page.

To cross-match SDSS with 2MASS you had to download both, understand two different formats,
write a conversion script, and match them yourself. Every single time. For every pair of
catalogues.

This was not merely tedious — it was a **barrier**. Only people who could code well could do
multi-wavelength science.

## 1.2 What the VO actually is

The **Virtual Observatory** is an international set of **standards** that lets astronomical
archives speak the same language.

It is **not a place**. It is **not a server**. It is **not a website you log into**.
It is a *protocol* — much like HTTP.

The goal: use every archive in the world **as if it were a single database**.

The standards are defined by the **IVOA** (International Virtual Observatory Alliance). That
is why VO identifiers begin with `ivo://` — just as web addresses begin with `http://`.

## 1.3 The standards you will use today

| Standard | What it is | Where you'll meet it |
|---|---|---|
| **VOTable** | The standard table format. Carries the data **and its metadata** (units, descriptions, UCDs). | Every file you export or save |
| **TAP** | *Table Access Protocol* — how you ask a server for tables. | `VO → Table Access Protocol (TAP) Query` |
| **ADQL** | *Astronomical Data Query Language* — SQL plus spherical geometry. | The query you write inside TAP |
| **Registry** | The phone book: who offers what, and at which address. | The service list that appears when you search |
| **SAMP** | How programs on your own machine talk to each other. | Aladin sending a table straight into TOPCAT |
| **Cone Search** | "Give me everything inside this circle on the sky." | Positional queries in VizieR |
| **HiPS** | Tiled, multi-resolution all-sky images. | The galaxy cutouts in Part 5 |

## 1.4 Why this matters

The `VO` menu in TOPCAT is not just another menu. It is the door to **every public
astronomical dataset on the planet**, from a single program, without downloading anything by
hand.

> The Virtual Observatory is what lets you cross an optical catalogue from Strasbourg with an
> infrared catalogue from California and an X-ray catalogue from NASA — on your laptop, in two
> minutes, without writing a line of code.

For extragalactic work this is not a convenience. It is the infrastructure the field runs on:
nobody builds a multi-band SED or selects AGN by colour without it.

---
---

# Part 2 — Exploring a TAP service

Open **`VO → Table Access Protocol (TAP) Query`**, search for `SDSS`, and TOPCAT queries the
**Registry**: *"who in the world holds tables matching SDSS?"*

## 2.1 Reading the service list

```
TAPVizieR (593/63752) - ivo://cds.vizier/tap
    ^        ^   ^              ^
    |        |   |              +-- Unique identifier (IVOID)
    |        |   +----------------- TOTAL tables held by the service
    |        +--------------------- Tables MATCHING your keyword
    +------------------------------ Service name
```

**The numbers are a real decision criterion.** You do not pick a service by its name — you
pick it by whether it actually holds what you need. `TAPVizieR` matched 593 SDSS tables;
`HEASARC` matched 32.

### Who's who

| Service | Who they are | Strong in |
|---|---|---|
| **TAPVizieR** | CDS, Strasbourg | **Everything.** ~63,000 tables — the universal aggregator of published catalogues |
| **Data Lab TAP** | NOIRLab (USA) | Deep optical surveys |
| **VSA / WSA / WFAU** | Univ. of Edinburgh | Infrared surveys (VISTA, UKIRT) |
| **HEASARC** | NASA | **High energies**: X-ray, gamma-ray — essential for AGN work |
| **IRSA** | Caltech / IPAC | **Infrared**: WISE, Spitzer, Herschel |
| **PS1DR1 / PS1DR2** | STScI | Pan-STARRS |
| **ESO TAP_CAT** | ESO | ESO catalogues |
| **GAIA / ARI-Gaia / Gaia@AIP** | Various mirrors | Gaia only |

**Notice: several services offer SDSS.** There is no single place where data "live". That is
the whole point of the VO — the data are distributed, and the Registry helps you find where.

> **Do not memorise where the data are. Learn to ask the Registry.**
> Archives change, go down, get renamed. The Registry is the layer that protects you.

**Select `TAPVizieR` and click `Use Service`** (or just double-click it).

---

## 2.2 The schema browser

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

### How VizieR organises its 63,884 tables

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

## 2.3 🔥 TRAP #1: `Name` vs `Descrip`

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

> **When a search returns nothing, the data almost certainly exist.**
> You are searching the wrong field.

---

## 2.4 🔥 TRAP #2: where SDSS actually lives

Even with 642 matches, the main SDSS catalogue is **not** in `II_photometry`, and **not** in
any folder called `V_something`. It is in:

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

> **An archive's organisation has an internal logic — but not necessarily *your* logic.**
> Explore; do not assume.

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
| **`Columns`** | **Every column: name, type, unit, description, UCD. This is the tab that matters.** |
| `FKeys` | Foreign keys — how this table links to others. Advanced. |

> ⚠️ **Check the row count in `Table`.** If it says hundreds of millions, your `WHERE` clause
> had better be selective, or you will wait forever.

---

## 2.6 Reading `Columns` — the heart of the matter

This is where the workshop is won or lost.

### The columns we need

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

---

### 🔥 TRAP #3: five columns start with `z` and mean five different things

```
zsp     -->  Spectroscopic redshift          <-- what you want
zph     -->  Photometric redshift            <-- a different (noisier) quantity
zmag    -->  MODEL magnitude in z band       <-- a magnitude!
zpmag   -->  PSF magnitude in z band         <-- a different magnitude!
zPmag   -->  PETROSIAN magnitude in z band   <-- yet another magnitude!
```

**`zpmag` and `zPmag` differ by a single capital letter.**

**The query will not warn you.** It will run, return numbers, and produce a plot. The plot
will simply be *wrong*.

---

### 🔥 TRAP #4: PSF magnitudes are wrong for galaxies

The first magnitudes in the list are `upmag, gpmag, rpmag, ipmag, zpmag` — **PSF magnitudes**.
It is tempting to just use them.

**Do not.** A PSF magnitude fits the image's point-spread function to the source.

- For a **star** → perfect. The star *is* a PSF.
- For an **extended galaxy** → **it fits the PSF inside the nucleus and throws away the disc
  and the halo.**

The flux is systematically underestimated, and the size of the error **depends on angular
size** — i.e. on redshift and morphology. You would be injecting a bias correlated with
exactly the variables you want to study.

For colours it is worse: galaxies have colour gradients (red bulges, blue discs), so the PSF
over-weights the bulge. **Your "red sequence" would be partly an aperture artefact.**

| Magnitude | What it is | Use for |
|---|---|---|
| **`?Pmag`** (Petrosian) | Aperture set by the light profile, redshift-independent | **The SDSS standard for galaxies** |
| `?mag` (model) | Best fit of a de Vaucouleurs or exponential profile | Also valid |
| `?pmag` (PSF) | ❌ | Stars only |

#### ⚠️ Do not mix apertures

If you measure `r` with a Petrosian aperture and `(g−r)` with model magnitudes, **the point
you plot does not correspond to any physically consistent object.**

**Pick one system and use it for everything.** Here: **Petrosian**, magnitude *and* colour.

---

### 🔥 TRAP #5: `class` is not `spCl`

| Column | How it classifies | Reliability |
|---|---|---|
| `class` | **Photometrically** — from the shape in the image | An *estimate* |
| **`spCl`** | **Spectroscopically** — from the actual spectrum | The **truth** |

If the object has a spectrum, **use `spCl`**.

And note: `spCl` distinguishes **`QSO`** from **`GALAXY`**. Photometric `class=3` would hand
you quasars along with your galaxies — and a quasar is a point source with a power-law
continuum. It will sit somewhere completely different on your CMD and contaminate everything.

---

### 🔥 TRAP #6: `f_zsp` — the flag that separates professionals from beginners

`f_zsp` is SDSS's **ZWarning** flag.

```
f_zsp = 0    -->  the redshift fit is reliable
f_zsp != 0   -->  the fit failed, the spectrum was bad, something went wrong
```

**Without `f_zsp = 0`, your sample contains garbage redshifts.** And once again, the query
runs perfectly.

**Every survey has flags like this. Find them. Use them.**

---

### Two more quality columns you would never have guessed

| Column | Why it matters |
|---|---|
| **`mode = 1`** | SDSS images overlapping regions. Without this, **the same galaxy appears several times** in your sample. |
| **`clean = 1`** | SDSS's own composite photometry-quality flag. |

---

## 2.7 A note on UCDs

A **UCD** (Unified Content Descriptor) is a standard label describing what a column *means*,
independent of what it is *called*.

`zsp` and `zph` both carry `src.redshift`. Different names, same physical meaning. Two archives
may call the same quantity `zsp`, `Z_SPEC`, or `redshift` — but all will tag it `src.redshift`.

**Software can rely on the UCD; it cannot rely on the name.** This is how TOPCAT automatically
knows which columns are coordinates when you run a cross-match without being told.

---

## 2.8 The thing to take away from Part 2

Every one of the six traps above **produces a query that runs without error**.

None raises an exception. None prints a warning. Each one quietly returns the wrong sample —
and you plot it, and it looks perfectly plausible.

> **The database will never tell you that you asked the wrong question.**
> That is your job.

---

## ✅ Checkpoint

Before writing any ADQL, you should be able to state:

- [ ] The **exact table name**, copied from the `Table` tab
- [ ] The **coordinate columns** — `RA_ICRS`, `DE_ICRS`
- [ ] The **redshift column** — `zsp`, and **why not** `zph`, `zmag`, `zpmag`, or `zPmag`
- [ ] The **magnitudes** — `uPmag`…`zPmag`, and **why not** the PSF ones
- [ ] The **classification column** — `spCl`, and **why not** `class`
- [ ] The **quality flags** — `f_zsp`, `mode`, `clean`, and what each protects you from

---
---

# Part 3 — Your first ADQL query

## 3.1 The query

Paste into the **`ADQL Text`** box and click **`Run Query`**:

```sql
SELECT TOP 30000
  RA_ICRS, DE_ICRS,
  "uPmag", "gPmag", "rPmag", "iPmag", "zPmag",
  e_uPmag, e_gPmag, e_rPmag, e_iPmag, e_zPmag,
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

**Expected result: 7,539 rows, 15 columns.**

> ✅ **Checkpoint.** Everyone should get the same number.
> Got 30,000? A filter is missing. Got 200? Something broke.

---

## 3.2 ⏳ If the query is slow or fails — use the backup file

This query asks VizieR to scan **hundreds of millions of rows**. With everyone querying at
once, the server may be slow or time out.

**Do not sit and wait. If it takes more than ~2 minutes, or fails:**

### 👉 Download `sdss_stripe82_galaxies.vot` from this repository

```
File  →  Load Table  →  select sdss_stripe82_galaxies.vot  →  OK
```

It is **exactly the result of the query above**. You lose nothing.

> **This is not cheating.** It is what you would do in real life. The moment a slow query
> succeeds, you write the result to disk so you never depend on a remote server twice for the
> same data.

**If your query did succeed:** save it anyway, right now —
`File → Save Table(s)` → format `votable`.

---

## 3.3 What every line does

| Line | Why it is there |
|---|---|
| `TOP 30000` | An **explicit** row limit. Unlike VizieR's web form, ADQL never truncates silently. |
| `RA_ICRS BETWEEN 320 AND 340` | **Stripe 82** — a long, narrow strip. Impossible to request with a cone search. |
| `DE_ICRS BETWEEN -1.25 AND 1.25` | Stripe 82 is narrow in declination. *This* is what makes it Stripe 82. |
| `spCl = 'GALAXY'` | Galaxies — **not QSOs, not stars**. |
| `zsp > 0.02` | Removes spurious redshifts and very nearby objects. |
| `zsp < 0.2` | **The genuinely local universe.** See §3.4. |
| **`f_zsp = 0`** | **The redshift fit is reliable.** |
| `mode = 1` | Primary detections only — no duplicates. |
| `clean = 1` | SDSS's own "clean photometry" flag. |

> **The query *is* the scientific decision.**
> Every line of the `WHERE` clause biases your sample, and you must be able to defend it.
> In a paper, this query belongs in the data section.

---

## 3.4 Why `z < 0.2` and not `z < 0.3`

| Reason | Effect |
|---|---|
| **The K-correction shrinks.** ~0.2 mag in `r` at z=0.2, versus ~0.4 at z=0.3 | Less systematic residual |
| **Incompleteness drops sharply.** SDSS's spectroscopic sample is flux-limited | At z=0.3 you only see intrinsically very bright galaxies — and the truncation is **colour-dependent**, since red galaxies are fainter at fixed mass |
| **Cosmology stops mattering.** At z<0.2, Ωm = 0.3 vs 0.25 is negligible | You need not worry about having picked the "right" cosmology |

**What you give up:** some statistics and a little dynamic range in $M_r$. **A cheap price** —
bimodality shows up perfectly well with 7,000 galaxies.

---

## 3.5 What the query does **not** filter — on purpose

- ❌ No magnitude cuts
- ❌ No colour cuts

**Deliberate.** The data you are about to load are **dirty**, and finding that out — and
deciding what to do about it — is the point of Part 4.

Notice the irony: the query *does* contain **`clean = 1`**, SDSS's own photometry-quality
flag. You will still find magnitudes of 33. Hold that thought.

---

## 3.6 🔥 TRAP #7: ADQL is case-insensitive — and it will stop you

Run the query **without** the double quotes and you get:

```
Incorrect ADQL query: 6 unresolved identifiers
  - Ambiguous column name "uPmag"! It may be (at least)
    "V/154/sdss16.upmag" or "V/154/sdss16.uPmag".
  ...
```

**The ADQL parser does not distinguish upper from lower case.** When you write `uPmag`, the
server genuinely cannot tell whether you mean `upmag` (**PSF**) or `uPmag` (**Petrosian**).

### The fix: quote the identifier

```sql
"uPmag"     -->  Petrosian. Unambiguous.
```

### ⚠️ Two kinds of quote, two different jobs

| Quote | Used for | Example |
|---|---|---|
| **Double** `"..."` | **Identifiers** (tables, columns) | `"uPmag"`, `"V/154/sdss16"` |
| **Single** `'...'` | **String values** | `'GALAXY'` |

Write `spCl = "GALAXY"` and ADQL will look for a *column* named GALAXY, and fail.

### The deeper lesson

**That error message saved you.**

The parser only complained because *both* columns exist. Had the catalogue contained only
`upmag`, your query would have run perfectly — and silently handed you PSF magnitudes for
galaxies.

> **The ambiguity is what protected you.** Do not count on being that lucky twice.

---

## 3.7 Why the query is slow: reading the `Indexed` column

The delay is not a bug. It is information.

Go back to **`Columns`** and look at the **`Indexed`** column:

| Column | Indexed? |
|---|---|
| `RA_ICRS`, `DE_ICRS`, `zsp`, `zph`, `objID` | ✅ |
| **`spCl`**, **`f_zsp`, `mode`, `clean`**, **`uPmag`…`zPmag`** | ❌ |

An **indexed** column filters instantly. A **non-indexed** column forces a scan of the whole
table.

Your slowest filter is `spCl = 'GALAXY'`: a **string** comparison on an **unindexed** column,
against hundreds of millions of rows.

> **The query you write determines how long you wait.**
> On a large survey this is the difference between 10 seconds and 10 minutes.

**To speed it up,** shrink the sky region — your cheapest lever, because the coordinates *are*
indexed:

```sql
  AND RA_ICRS BETWEEN 330 AND 340    -- half the sky area
```

---

## ✅ Checkpoint

- [ ] Table loaded — from your query, or from `sdss_stripe82_galaxies.vot`
- [ ] **Rows = 7,539**, **Columns = 15**
- [ ] The table is **saved on your disk**

---
---

# Part 4 — Looking at the data, and finding the bimodality

**Do not plot yet.**

> **Rule: never plot data you have not looked at.**
> Five minutes of inspection saves an afternoon of debugging a plot that came out wrong.

---

## 4.1 Look at the rows

**`Views → Table Data`** (the grid icon).

Check the query did what you thought: `RA_ICRS` in [320, 340], `DE_ICRS` in [−1.25, 1.25],
`zsp` in [0.02, 0.2], `spCl` = `GALAXY` everywhere.

---

## 4.2 Look at the statistics — the step that matters

**`Views → Column Statistics`** (the **`Σ`** button).

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

## 4.3 Three things are wrong here

### 1. There is a null

`uPmag` has **7,538** good values, not 7,539. **One galaxy has no `u` magnitude.**

One galaxy is nothing. But the habit is everything: it will **silently vanish** from any plot
using the `u` band, and TOPCAT will not tell you. In another catalogue it could be 3,000.

### 2. The magnitudes contain garbage

```
uPmag   Maximum = 33.147
iPmag   Maximum = 27.632
```

**SDSS has a detection limit of ~22 mag.** A galaxy at magnitude **33 does not exist**.

These are objects where the Petrosian fit **failed**: the algorithm did not converge, returned
nonsense, and the catalogue stored it anyway.

The giveaway: the *same* galaxy can be reasonable in `r` (max 24) and absurd in `u` (max 33).
That is not physics — that is a broken fit.

### 3. The errors are worse than the magnitudes

```
e_zPmag   Maximum = 44.086      an error of 44 magnitudes
e_zsp     Maximum =  1.7997     an error of 1.8 in the redshift
```

**Stop and think about `e_zsp`.** Your redshifts run from 0.02 to 0.2. **The largest error is
1.8** — roughly *twelve times* the largest redshift in the sample.

> That is not a measurement. That is a fit that failed and reported its failure honestly.

---

## 🔥 TRAP #8: quality flags are necessary, not sufficient

Your query contains **two** of SDSS's own quality flags:

```sql
AND f_zsp = 0     -- "the redshift fit is reliable"
AND clean = 1     -- "the photometry is clean"
```

**And you still got magnitudes of 33 and redshift errors of 1.8.**

> The survey's own flags say *this object is fine*. It is not.
>
> **Quality flags are necessary but not sufficient. Always look at the data yourself.**

---

## 4.4 Why this would destroy your plot

A galaxy with `uPmag = 33` and `rPmag = 17` gives a colour of **`u−r = 16`**. A real galaxy
has `u−r` between roughly **0.5 and 3.5**.

**That single point stretches your y-axis from 0–3.5 to 0–16.** Your entire signal gets
squashed into a thin band and you see nothing.

**Try it.** Plot it with the garbage in, look at the mess, then come back.

---

## 4.5 Row subsets — cleaning without deleting

A **subset** **does not delete rows**. It creates a *label* marking which rows satisfy a
condition. Reversible, and you can keep several at once and compare them.

**`Views → Row Subsets`** → **`New Subset`**

| Field | Value |
|---|---|
| **Name** | `good` |
| **Expression** | `uPmag < 22 && gPmag < 22 && rPmag < 21 && iPmag < 22 && zPmag < 22 && (uPmag-rPmag) > 0.5 && (uPmag-rPmag) < 4` |

**Result: 6,851 of 7,539 survive. 688 galaxies — 9.1% — were unusable.** Not a rounding error.

### ⚠️ TOPCAT expressions are not SQL

They use a **Java-like** syntax:

| Operator | Meaning |
|---|---|
| `&&` | AND |
| `\|\|` | OR |
| `!` | NOT |
| `==` | equals (**double**) |
| `!=` | not equal |

**Good news:** unlike ADQL, TOPCAT expressions **are** case-sensitive. `uPmag` means `uPmag`
and nothing else.

### Two kinds of cut — and why you need both

**Magnitude cuts** (`< 22`) remove objects fainter than SDSS can detect.

**Colour cuts** (`0.5 < u−r < 4`) are the ones that actually save you. A galaxy with
`uPmag = 21.9` (garbage, but under the cut) and `rPmag = 17` (fine) **passes the magnitude
filter** — and hands you `u−r = 4.9`.

> **Filtering the magnitudes is not enough. You have to filter the quantity you actually
> plot.**

### 🎓 Exercise

Build a second subset using the **errors** instead:

```
e_zsp < 0.001 && e_uPmag < 0.5 && e_rPmag < 0.2
```

How many survive? Does the bimodality still appear? **Does your conclusion depend on which
cleaning strategy you chose?**

If it does, that is important. If it does not, that is *also* important — your result is
robust.

---

## 4.6 Reading `Rows: 7539 (6851 apparent)`

```
Rows:        7.539 (6.851 apparent)
Row Subset:  good
```

- **`7,539`** — rows the table *has*. Nothing was deleted.
- **`6,851 apparent`** — rows TOPCAT will *show you*.

### ⚠️ The classic trap

**Everything you now do — plots, statistics, cross-matches — uses only the active subset.**

Half an hour from now you will get a number that makes no sense, and it will be because you
forgot a subset was active.

> **If a result surprises you, the first thing you check is which subset is active.**

**Verify:** go back to `Column Statistics`, set `Subset for calculations: good`. `uPmag` max
should now be under 22, not 33. And `nGood` should be **6,851 in every column**.

---

## 4.7 Absolute magnitude — making the x-axis physical

Apparent magnitude is **not a property of the galaxy**. It depends on distance. An axis in
apparent `r` mixes "intrinsically bright galaxies far away" with "faint galaxies nearby". That
is geometry, not physics.

You have `zsp`. Use it.

$$M_r = m_r - \mathrm{DM}(z) - K_r(z,\ u{-}r)$$

### First: explore what TOPCAT already has

**`Help → Available Functions`** opens the **Function Browser**:

| Class | Contains |
|---|---|
| **`Distances`** | `luminosityDistance(z, H0, omegaM, omegaLambda)`, `lookbackTime`, `zToAge`. Based on **Hogg (2000)**, astro-ph/9905116. |
| **`KCorrections`** | `kCorr(filter, redshift, colorType, colorValue)`. Based on **Chilingarian, Melchior & Zolotukhin (2010)**, valid for **0 < z < 0.5**. |

> **This window is the `Columns` tab, but for operations.** Same habit: **explore before you
> assume.** Do not hand-code an approximation of something the tool already does properly.

Constants you need: **`KCF_r`** (the r filter), **`KCC_ur`** (the SDSS u−r colour).

### Create the column

**`Views → Column Info`** → **`Columns → New Synthetic Column`** (or the **`f(x)`** button).

> ⚠️ The `Columns` menu lives **inside the `Table Columns` window**, not in the main menu bar.

| Field | Value |
|---|---|
| **Name** | `Mr` |
| **Units** | `mag` |
| **Expression** | `rPmag - (5*log10(luminosityDistance(zsp, 70, 0.3, 0.7)) + 25) - kCorr(KCF_r, zsp, KCC_ur, uPmag - rPmag)` |

| Term | Meaning |
|---|---|
| `luminosityDistance(zsp, 70, 0.3, 0.7)` | $d_L$ in **Mpc**; flat ΛCDM, $H_0=70$, $\Omega_m=0.3$, $\Omega_\Lambda=0.7$ |
| `5*log10(d_L) + 25` | the **distance modulus** (`+25` converts Mpc to the 10 pc scale) |
| `kCorr(KCF_r, zsp, KCC_ur, uPmag-rPmag)` | the **K-correction** in `r`, using the observed `u−r` colour |

**Check it:** `Mr` should run from roughly **−24 to −17**. Positive values mean something went
wrong.

### Why the K-correction matters more than you think

The K-correction **depends on the galaxy's colour**. A red and a blue galaxy at the same
redshift get *different* corrections.

**Omit it, and you introduce an error correlated with the very axis you are trying to
measure.** You would shift your two populations by different amounts — distorting the
separation you want to study.

---

## 4.8 What we did *not* correct for

| Not applied | Size | Does it bias the result? |
|---|---|---|
| **Galactic extinction** | ~0.05–0.1 mag in `r`, at Stripe 82's high galactic latitude | **Roughly a constant offset.** Shifts the diagram; does not distort its internal structure. |
| **Selection function** | Large | **Yes.** SDSS is flux-limited, and red galaxies are fainter at fixed mass — they drop out first. |
| **Aperture effects** | Small (Petrosian is designed to minimise them) | Mildly |

> **Omitting extinction degrades your precision. Omitting the K-correction biases your
> result.** These are not the same kind of error.

---

## 4.9 The colour–magnitude diagram

**`Graphics → Plane Plot`**

| Field | Value |
|---|---|
| **X** | `Mr` |
| **Y** | `uPmag - rPmag` |
| **Axes** | tick **`X Flip`** — $M_r$ is negative; bright galaxies on the left |
| **Subset** | `good` |

You get a solid red blob. **That is the problem.** With ~6,800 opaque points drawn on top of
one another, the dense region saturates and you lose all density information.

### Add contours

Bottom panel → tab **`Form`** → green **`+ Forms`** → **`Contour`**.

**Now you see it: two separate peaks.**

- **Upper peak:** `u−r ≈ 2.7`, `Mr ≈ −21.3` → **THE RED SEQUENCE**
- **Lower peak:** `u−r ≈ 1.7`, `Mr ≈ −20.3` → **THE BLUE CLOUD**
- **Between them:** sparse contours → **THE GREEN VALLEY**

---

## 4.10 🚨 The two contour parameters — and the danger

### `Level Count`

How many contour lines. Like a topographic map: each line joins points of equal density.

- **Few (3–5):** coarse structure. **The two peaks stand out.**
- **Many (15–20):** fine detail, but the plot saturates and the eye loses the global structure.

### `Smoothing` — the dangerous one

The width of the smoothing kernel. Your data are discrete points; to draw contours TOPCAT first
estimates a *continuous* density by averaging each point with its neighbours.

| Smoothing | Effect |
|---|---|
| **Low (5–10)** | Ragged, noisy contours. **False peaks appear** from statistical fluctuations. |
| **Medium (~50)** | Balanced. Real structure emerges; noise is suppressed. |
| **High (200+)** | **Everything merges into one blob.** ⚠️ **The bimodality disappears.** |

### Try it. Right now.

1. Set `Smoothing` to **200** → the two peaks **fuse into one**. You would conclude there is no
   bimodality.
2. Set it to **5** → lumps appear everywhere. You would conclude there are five populations.

**Neither conclusion is correct.** Both are artefacts of a parameter *you* chose.

> **If your result depends critically on a visualisation parameter, it is not a result.**
> Verify it with a method that does not depend on that parameter.

---

## 4.11 The independent check: a 1-D histogram

**`Graphics → Histogram`** → **X = `uPmag - rPmag`**, subset `good`.

**No smoothing. No kernel. No free parameters.** Just counts in bins.

- **Peak 1** at `u−r ≈ 1.75` → **blue cloud**
- **A dip** at `u−r ≈ 2.3` → **green valley**
- **Peak 2** at `u−r ≈ 2.65` → **red sequence**

**Two humps and a valley, straight out of the raw counts.**

**This is the verification.** The bimodality in the contour plot was *not* an artefact of the
smoothing — it is in the data.

---

## 4.12 What you have just shown

> Galaxies are **not** distributed continuously in colour. **They come in two families.**
>
> **Blue galaxies** are actively forming stars — young hot stars emit in the UV, so `u` is
> bright and `u−r` is small.
>
> **Red galaxies** have stopped forming stars — only old stars remain, so `u−r` is large.
>
> **And the valley between them is sparsely populated** — meaning the transition from one to
> the other is **fast**. Galaxies do not linger in the middle.

A real scientific result, obtained in ninety minutes, without writing a single line of code.

---

## 4.13 ⚠️ One thing you may NOT conclude

The blue peak is **higher** than the red peak.

**This does *not* mean there are fewer red galaxies in the universe.** It means there are fewer
in *your sample* — and your sample has a **selection bias**.

SDSS is flux-limited. At fixed stellar mass, red galaxies are **fainter** than blue ones, so
they drop out of a flux-limited survey first.

> **Any relative count between populations is biased by the selection function.**
> Doing this properly requires computing maximum volumes ($V_{max}$) and weighting each galaxy.
> That is a paper, not a class.

---

## ✅ Checkpoint

- [ ] You found the garbage photometry **and** the absurd errors **before** plotting
- [ ] `good` subset built — **6,851 of 7,539 survive**
- [ ] You cut on **colour**, not just magnitude
- [ ] `Mr` runs from roughly −24 to −17
- [ ] Contour plot shows **two peaks**
- [ ] Histogram **independently confirms** them
- [ ] You can name **three things you did not correct for**
- [ ] You can explain why `f_zsp = 0` and `clean = 1` **were not enough**

---
---

# Part 5 — Clicking on a galaxy: from a number to an image

You have two peaks in a histogram. So far they are pure statistics.

**Now let's look at what those numbers actually are.**

---

## 5.1 Activation Actions — the feature nobody finds

TOPCAT can **do something when you click a row** — in a table, or on a point in a plot.

**`Views → Activation Actions`** (or the yellow **⚡** button).

```
☑ Use Sky Coordinates in      <-- on by default
☐ Send Sky Coordinates         (SAMP -> Aladin, DS9, ...)
☐ Display HiPS cutout          <-- what we want
☐ Send HiPS cutout
☐ Delay
☐ Execute code
☐ Run system command
☐ Send row index
```

### ⚠️ The trap in this window

**The `Configuration` panel on the right shows the settings of the action that is
SELECTED — not the one that is TICKED.**

You must do **two separate things**:

1. **Tick the checkbox** of `Display HiPS cutout` → this *enables* it
2. **Click on its name** → this *selects* it, so you can configure it

If you only tick it, you will sit there wondering why the right-hand panel still shows someone
else's settings.

---

## 5.2 Configuring the cutout

| Field | Value | Notes |
|---|---|---|
| **RA Column** | `RA_ICRS` | should be filled in already |
| **Dec Column** | `DE_ICRS` | ditto |
| **HiPS Survey** | `SDSS9/color-alt` | click **`Select`** to browse |
| **Field of View** | **`0.02`** degrees | see below |
| **Size in Pixels** | `300` | fine as is |

### ⚠️ Get the Field of View right

The default is often **`0.2` degrees = 12 arcmin**. **That is enormous.**

A galaxy at z ≈ 0.1 has an angular size of roughly **10–20 arcsec**. At 0.2° you get a wide
field with dozens of galaxies in it, and yours is a dot in the middle.

**Use `0.02` degrees** (≈ 1.2 arcmin). The galaxy will fill the frame.

> Angular size depends on redshift: a galaxy at z=0.05 looks large, one at z=0.19 is tiny. A
> single fixed FoV is a compromise.

---

## 5.3 Now click on the plot

Go back to your **colour–magnitude diagram** and click directly on a data point.

**A small window opens with the SDSS image of that galaxy.**

The `Results` panel logs each click. Fast clicking produces `CANCELLED` entries — a new request
superseded a pending one. Normal.

---

## 5.4 🔬 The experiment

### Click a galaxy in the RED peak — `u−r ≈ 2.7`, `Mr ≈ −21`

> **An elliptical.** Yellowish. Smooth. A gentle gradient from the centre outward.
> **No structure at all.**

### Click a galaxy in the BLUE peak — `u−r ≈ 1.7`, `Mr ≈ −20`

> **Structure.** A bright nucleus surrounded by a more diffuse, bluer disc. Asymmetry. Extent.

### This is what it should look like

![Colour-magnitude diagram with two galaxy cutouts](images/cmd-two-galaxies.png)

**Top cutout:** clicked in the red peak — smooth elliptical.
**Bottom cutout:** clicked in the blue peak — structured disc.

**Same plot. Two clicks. Two different kinds of galaxy.**

---

## 5.5 What you have just demonstrated

> You selected these galaxies by **a number** — the colour `u−r`, computed from two magnitudes
> in a catalogue.
>
> You never looked at an image. You never said anything about shape.
>
> **And yet the two peaks of the histogram correspond to two kinds of galaxy that
> *look different*.**
>
> Colour measures the stellar population. The stellar population reflects the star-formation
> history. And the star-formation history is tied to the structure.
>
> **The peaks were not statistics. They were physics.**

---

## 5.6 ⚠️ But do not oversell it

The blue galaxy you clicked is probably **not a textbook grand-design spiral**. It is more
likely a fuzzy disc with a bulge.

- At z ≈ 0.1, SDSS's **seeing** (~1.4″) smears out fine structure. Spiral arms are lost.
- Colour measures **stellar populations**, not morphology. The correlation is strong — but it is
  **not a law**.
- **Red spirals exist** (quenched, but still discs). **Blue ellipticals exist** (recent star
  formation).

> **Colour predicts morphology. It does not determine it.**

---

## 5.7 Other things Activation Actions can do

| Action | Use |
|---|---|
| **`Send Sky Coordinates`** | Via **SAMP** — makes Aladin or DS9 jump to that object, live, with catalogues overlaid. |
| **`Send HiPS cutout`** | Pushes the image to another application. |
| **`Execute code`** | Run an arbitrary expression on the clicked row. |
| **`Run system command`** | Launch an external program with the row's values as arguments. |
| **`View URL`** | Open a browser at a URL built from the row — e.g. the SDSS SkyServer page. |

**This is how you inspect outliers.** Whenever a point on a plot looks wrong, **click it and
look at the object.** Half the time it is a blended pair, an artefact, or a satellite trail —
and you would never have known from the numbers alone.

> **The most under-used debugging tool in astronomy is looking at the image.**

---

## ✅ Checkpoint

- [ ] `Display HiPS cutout` is both **ticked** and **selected** (different things)
- [ ] Field of View is **0.02**, not 0.2
- [ ] Clicking a point on the CMD opens an image
- [ ] Red-peak galaxy → **smooth elliptical**
- [ ] Blue-peak galaxy → **structure**
- [ ] You can explain why the correlation is **strong but not absolute**

---
---

# Appendix — The eight traps

Every one of these **produces output that looks fine**. None throws an error.

| # | Trap | What it costs you |
|---|---|---|
| **1** | Searching `Name` but not `Descrip` | You conclude the data do not exist. They do. |
| **2** | Assuming where a catalogue lives | SDSS is in `large_tables`, not `II_photometry` |
| **3** | `zsp` / `zph` / `zmag` / `zpmag` / `zPmag` | Five columns, one letter apart, five meanings |
| **4** | PSF magnitudes for galaxies | A "red sequence" that is partly an aperture artefact |
| **5** | `class` instead of `spCl` | Quasars contaminating your galaxy sample |
| **6** | Forgetting `f_zsp = 0` | Garbage redshifts in your sample |
| **7** | Unquoted identifiers in ADQL | Ambiguity — **or worse, silent selection of the wrong column** |
| **8** | Trusting quality flags alone | `clean = 1` and `f_zsp = 0`, and *still* magnitudes of 33 |

## The single thing to remember

> **The database will never tell you that you asked the wrong question.**
> **The plot will never tell you that you plotted the wrong column.**
> **The flag will never tell you that it missed something.**
>
> **That is your job.**
