# 1. The Virtual Observatory (VO)

## The problem it solves

Before the VO, every astronomical archive was an island:

- SDSS had its own website, its own format, its own query interface.
- 2MASS had a different one.
- Chandra had another.
- The catalogue from a 1998 paper lived as a `.txt` file on somebody's personal page.

If you wanted to cross-match SDSS with 2MASS, you had to download both, understand two
different formats, write a conversion script, and match them yourself. Every single time.
For every pair of catalogues.

This was not just tedious — it was a **barrier**. Only people who could code well were able
to do multi-wavelength science.

## What the VO actually is

The **Virtual Observatory** is an international set of **standards** that lets astronomical
archives speak the same language.

It is **not a place**. It is **not a server**. It is **not a website you log into**.
It is a *protocol* — much like HTTP.

The goal: use every archive in the world **as if it were a single database**.

The standards are defined by the **IVOA** (International Virtual Observatory Alliance).
That is why VO identifiers begin with `ivo://` — just as web addresses begin with `http://`.

## The standards you will use in this class

| Standard | What it is | Where you'll meet it |
|---|---|---|
| **VOTable** | The standard table format. Carries the data **and its metadata** (units, descriptions, UCDs). | Every file you export from VizieR |
| **TAP** | *Table Access Protocol* — how you ask a server for tables. | The `VO → Table Access Protocol (TAP) Query` menu in TOPCAT |
| **ADQL** | *Astronomical Data Query Language* — SQL plus spherical geometry. | The query you write inside TAP |
| **Registry** | The phone book: who offers what, and at which address. | The list of services that appears when you search `SDSS` |
| **SAMP** | How programs on your own machine talk to each other. | Aladin sending a table straight into TOPCAT |
| **Cone Search** | "Give me everything inside this circle on the sky." | Positional queries in VizieR |
| **HiPS** | Tiled, multi-resolution all-sky images. | The sky maps in Aladin |

## Why this matters

The `VO` menu in TOPCAT is not just another menu. It is the door to **every public
astronomical dataset on the planet**, from a single program, without downloading anything
by hand.

> The Virtual Observatory is what lets you cross an optical catalogue from Strasbourg with
> an infrared catalogue from California and an X-ray catalogue from NASA — on your laptop,
> in two minutes, without writing a line of code.

For extragalactic work this is not a convenience. It is the infrastructure the field runs
on: nobody builds a multi-band SED or selects AGN by colour without it.

---

# 2. Reading the TAP service list

When you open `VO → Table Access Protocol (TAP) Query` and search for a keyword
(for example `SDSS`), TOPCAT queries the **Registry** and returns a list of services.

That list is the Registry's answer to the question:
*"Who in the world holds tables matching `SDSS`?"*

## Anatomy of a line

```
TAPVizieR (593/63752) - ivo://cds.vizier/tap
    ^        ^   ^              ^
    |        |   |              +-- Unique identifier (IVOID)
    |        |   +----------------- TOTAL tables held by the service
    |        +--------------------- Tables MATCHING your keyword
    +------------------------------ Service name
```

| Element | What it tells you |
|---|---|
| **Name** | Who provides the service |
| **First number** | How many of its tables match your search |
| **Second number** | The total size of the archive |
| **`ivo://...`** | The address. `ivo://` is the VO equivalent of `http://` |

**The numbers are a real decision criterion.** You do not pick a service by its name — you
pick it by whether it actually holds what you need.

## Who's who

These are the archives you will keep meeting throughout your career:

| Service | Who they are | Strong in |
|---|---|---|
| **TAPVizieR** | CDS, Strasbourg | **Everything.** ~63,000 tables — the universal aggregator of published catalogues |
| **Data Lab TAP** | NOIRLab (USA) | Deep optical surveys |
| **VSA / WSA / WFAU** | Univ. of Edinburgh | Infrared surveys (VISTA, UKIRT) |
| **HEASARC** | NASA | **High energies**: X-ray, gamma-ray — essential for AGN work |
| **GAVO** | Germany | Theoretical data and miscellaneous catalogues |
| **GAIA / ARI-Gaia / Gaia@AIP** | Various mirrors | Gaia only |
| **IRSA** | Caltech / IPAC | **Infrared**: WISE, Spitzer, Herschel |
| **PS1DR1 / PS1DR2** | STScI | Pan-STARRS |
| **ESO TAP_CAT** | ESO | ESO catalogues |

## Two things to notice

**1. Several services offer SDSS.** There is no single place where data "lives". This is the
entire point of the VO: the data are distributed, and the Registry is what helps you find
where.

**2. The counts differ enormously.** `TAPVizieR` matched 593 SDSS tables; `HEASARC` matched
32. For a query about SDSS, TAPVizieR is the obvious choice — not because it is famous, but
because the number says so.

## The habit to build

> Do not memorise where the data are. Learn to **ask the Registry**.
> Archives change, go down, get renamed. The Registry is the layer that protects you from that.

---

## Next step

Select **`TAPVizieR`** from the list and click **`Use Service`**.
You are now ready to write your first ADQL query.
