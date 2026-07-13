# Astronomical Databases and TOPCAT

### A hands-on workshop: finding galaxy bimodality in SDSS

---

## 🌐 Choose your language / Elige tu idioma / Escolha seu idioma

| | | |
|:---:|:---:|:---:|
| 🇬🇧 **[English](TOPCAT-workshop-EN.md)** | 🇪🇸 **[Español](TOPCAT-workshop-ES.md)** | 🇧🇷 **[Português](TOPCAT-workshop-PT.md)** |
| *main version* | *traducción* | *tradução* |

> **Note:** TOPCAT's interface is in English. In all three versions, **menu names, buttons and
> column names are kept in English** (`Views → Column Statistics`, `uPmag`, `nGood`) so that
> what you read matches what you see on screen.

---

## What you will do

Starting from nothing, you will pull a sample of galaxies from a public archive, clean it,
compute their absolute magnitudes, and discover that galaxies come in **two distinct
families** — one red and dead, one blue and star-forming.

You will then click on a point in your plot and **see the galaxy**.

Finally, you will cross-match with **Galaxy Zoo** and test, on three thousand galaxies,
whether colour really predicts morphology.

**No code. Ninety minutes.**

![Colour-magnitude diagram with two galaxy cutouts](images/cmd-two-galaxies.png)

---

## Contents of this repository

| File | What it is |
|---|---|
| **`TOPCAT-workshop-EN.md`** | The tutorial — **main version** |
| `TOPCAT-workshop-ES.md` | Spanish translation |
| `TOPCAT-workshop-PT.md` | Portuguese translation |
| **`sdss_stripe82_galaxies.vot`** | **The data.** Use this if the archive is slow or down. |
| `images/` | Figures |

---

## Before you start

### 1. Install TOPCAT

Download **`topcat-full.jar`** from
[the official page](https://www.star.bris.ac.uk/~mbt/topcat/).

> ⚠️ Get the **full** jar, not the "lite" one — you will need the extras.

You need Java. Check with:

```bash
java -version
```

Run it with:

```bash
java -jar topcat-full.jar
```

**Java is slow to start. Open TOPCAT at the beginning of the session and leave it open.**

### 2. Download the backup data

Get **`sdss_stripe82_galaxies.vot`** from this repository **now**, before the session.

You will query the archive live during the workshop — but archives get slow when twenty people
hit them at once. If your query stalls, load this file instead and carry on. **You lose
nothing.**

---

## What this workshop is really about

Not TOPCAT. TOPCAT is buttons; you can look those up.

It is about a **habit of suspicion**:

> **The database will never tell you that you asked the wrong question.**
>
> **The plot will never tell you that you plotted the wrong column.**
>
> **The flag will never tell you that it missed something.**
>
> **That is your job.**

The tutorial contains **twelve traps**. Every one of them produces output that looks
completely fine. None of them throws an error.

---

## Data credits

- **SDSS DR16** — `V/154/sdss16` via [VizieR](https://vizier.cds.unistra.fr/), CDS Strasbourg
- **Galaxy Zoo 1** — Lintott et al. 2011, MNRAS 410, 166 — `J/MNRAS/410/166`
- **K-corrections** — Chilingarian, Melchior & Zolotukhin 2010, MNRAS 405, 1409
- **Cosmological distances** — Hogg 2000, [astro-ph/9905116](https://arxiv.org/abs/astro-ph/9905116)
- **TOPCAT** — M. B. Taylor 2005, ASP Conf. Ser. 347, 29
