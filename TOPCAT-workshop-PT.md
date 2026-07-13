# Bancos de Dados Astronômicos e TOPCAT
## Oficina prática: encontrando a bimodalidade de galáxias no SDSS

> 🌐 [English](TOPCAT-workshop-EN.md) · [Español](TOPCAT-workshop-ES.md) · **Português**

> ⚠️ **A interface do TOPCAT está em inglês.** Neste documento, os **nomes de menus, botões e
> colunas são mantidos em inglês** (`Views → Column Statistics`, `uPmag`, `nGood`) para que o
> que você lê corresponda ao que você vê na tela.

---

**O que você vai fazer:** partindo do zero, você vai extrair uma amostra de galáxias de um
arquivo público, limpá-la, calcular suas magnitudes absolutas, e descobrir que as galáxias vêm em
**duas famílias distintas** — uma vermelha e morta, outra azul e formando estrelas.

Depois você vai clicar em um ponto do seu gráfico e **ver a galáxia**.

Por fim, você vai fazer um cruzamento com o **Galaxy Zoo** e testar, com três mil galáxias, se a
cor realmente prevê a morfologia.

Sem código. Noventa minutos.

---

## Onde você vai chegar

**Este é o resultado final. É isto que você deve ter na tela ao terminar:**

![Diagrama cor-magnitude com duas galáxias](images/cmd-two-galaxies.png)

À esquerda: o **diagrama cor-magnitude**, com contornos revelando **dois picos separados**.

À direita, duas galáxias nas quais você clicou diretamente naquele gráfico:

- **Em cima** — do **pico superior** (`u−r ≈ 2,7`): uma **elíptica lisa e amarelada**.
  Sem estrutura.
- **Embaixo** — do **pico inferior** (`u−r ≈ 1,7`): um **disco mais azul** com estrutura visível
  e um núcleo brilhante.

**Você vai selecionar essas galáxias usando nada além de um número.** Você nunca vai dizer ao
computador nada sobre a forma delas.

E ainda assim, os dois picos acabam sendo **dois tipos de galáxia que parecem diferentes**.

Esse é o destino. Vamos lá.

---

## Conteúdo

**[Parte 1 — O Observatório Virtual](#parte-1--o-observatório-virtual)**
**[Parte 2 — Explorando um serviço TAP](#parte-2--explorando-um-serviço-tap)**
**[Parte 3 — Sua primeira consulta ADQL](#parte-3--sua-primeira-consulta-adql)**
**[Parte 4 — Olhando os dados e encontrando a bimodalidade](#parte-4--olhando-os-dados-e-encontrando-a-bimodalidade)**
**[Parte 5 — Clicando em uma galáxia](#parte-5--clicando-em-uma-galáxia-de-um-número-a-uma-imagem)**
**[Parte 6 — Cruzamento: a cor realmente prevê a morfologia?](#parte-6--cruzamento-a-cor-realmente-prevê-a-morfologia)**
**[Apêndice — As doze armadilhas](#apêndice--as-doze-armadilhas)**

---
---

# Parte 1 — O Observatório Virtual

## 1.1 O problema que ele veio resolver

Antes do VO, cada arquivo astronômico era uma ilha:

- O SDSS tinha seu site, seu formato, sua forma de buscar.
- O 2MASS tinha outra.
- O Chandra, outra.
- O catálogo de um artigo de 1998 vivia como um `.txt` na página pessoal de alguém.

Para cruzar SDSS com 2MASS, você tinha que baixar os dois, entender dois formatos diferentes,
escrever um script de conversão, e cruzá-los você mesma. Toda vez. Para cada par de catálogos.

Não era só tedioso — era uma **barreira**. Só quem sabia programar bem conseguia fazer ciência
multi-comprimento de onda.

## 1.2 O que o VO realmente é

O **Observatório Virtual** é um conjunto internacional de **padrões** que permite que os arquivos
astronômicos falem a mesma língua.

**Não é um lugar. Não é um servidor. Não é um site em que você entra.**
É um *protocolo* — como o HTTP.

O objetivo: usar todos os arquivos do mundo **como se fossem um único banco de dados**.

Os padrões são definidos pela **IVOA** (*International Virtual Observatory Alliance*). É por isso
que os identificadores começam com `ivo://`, assim como endereços web começam com `http://`.

## 1.3 Os padrões que você vai usar hoje

| Padrão | O que é | Onde você vai encontrar |
|---|---|---|
| **VOTable** | O formato de tabela padrão. Carrega os dados **e seus metadados** (unidades, descrições, UCDs). | Cada arquivo que você exporta ou salva |
| **TAP** | *Table Access Protocol* — como você pede tabelas a um servidor. | `VO → Table Access Protocol (TAP) Query` |
| **ADQL** | *Astronomical Data Query Language* — SQL mais geometria esférica. | A consulta que você escreve dentro do TAP |
| **Registry** | A lista telefônica: quem oferece o quê, e em qual endereço. | A lista de serviços que aparece ao buscar |
| **SAMP** | Como os programas da sua máquina conversam entre si. | O Aladin mandando uma tabela para o TOPCAT |
| **Cone Search** | "Me dê tudo que existe dentro deste círculo no céu." | Consultas posicionais no VizieR |
| **HiPS** | Imagens do céu em mosaico e multi-resolução. | Os recortes de galáxias da Parte 5 |

## 1.4 Por que isso importa

O menu `VO` do TOPCAT não é só mais um menu. É a porta para **todos os dados astronômicos
públicos do planeta**, a partir de um único programa, sem baixar nada na mão.

> O Observatório Virtual é o que permite você cruzar um catálogo óptico de Estrasburgo com um
> infravermelho da Califórnia e um de raios X da NASA — no seu notebook, em dois minutos, sem
> escrever uma linha de código.

Para trabalho extragaláctico isso não é uma conveniência. É **a infraestrutura sobre a qual a
área funciona**: ninguém constrói uma SED multibanda nem seleciona AGN por cor sem o VO.

---
---

# Parte 2 — Explorando um serviço TAP

Abra **`VO → Table Access Protocol (TAP) Query`**, busque `SDSS`, e o TOPCAT consulta o
**Registry**: *"quem no mundo tem tabelas que combinam com SDSS?"*

## 2.1 Como ler a lista de serviços

```
TAPVizieR (593/63752) - ivo://cds.vizier/tap
    ^        ^   ^              ^
    |        |   |              +-- Identificador único (IVOID)
    |        |   +----------------- Tabelas TOTAIS do serviço
    |        +--------------------- Tabelas que COMBINAM com sua busca
    +------------------------------ Nome do serviço
```

**Os números são um critério de decisão real.** Você não escolhe um serviço pelo nome — escolhe
pelo fato de ele ter ou não o que você procura. O `TAPVizieR` deu 593 tabelas de SDSS; o
`HEASARC` deu 32.

### Quem é quem

| Serviço | Quem são | Fortes em |
|---|---|---|
| **TAPVizieR** | CDS, Estrasburgo | **Tudo.** ~63.000 tabelas — o agregador universal de catálogos publicados |
| **Data Lab TAP** | NOIRLab (EUA) | Levantamentos ópticos profundos |
| **VSA / WSA / WFAU** | Univ. de Edimburgo | Levantamentos infravermelhos (VISTA, UKIRT) |
| **HEASARC** | NASA | **Altas energias**: raios X, gama — essencial para AGN |
| **IRSA** | Caltech / IPAC | **Infravermelho**: WISE, Spitzer, Herschel |
| **PS1DR1 / PS1DR2** | STScI | Pan-STARRS |
| **ESO TAP_CAT** | ESO | Catálogos do ESO |

**Repare: vários serviços oferecem SDSS.** Não existe um único lugar onde os dados "moram". Esse é
o ponto do VO — eles são distribuídos, e o Registry ajuda você a encontrar onde.

> **Não decore onde os dados estão. Aprenda a perguntar ao Registry.**
> Arquivos mudam, caem, são renomeados. O Registry é a camada que protege você.

**Selecione `TAPVizieR` e clique em `Use Service`** (ou dê um duplo clique).

---

## 2.2 O navegador de esquemas

```
┌──────────────────────┬────────────────────────────────────┐
│  Find: [        ]    │  ○Service ○Schema ○Table ○Columns  │
│  ☑Name ☐Descrip      │                                    │
│                      │   (metadados do que estiver         │
│  ▼ TAPVizieR (63884) │    selecionado à esquerda)          │
│    ▸ I_astrometry    │                                    │
│    ▸ II_photometry   │                                    │
│    ▸ J_ApJ (10721)   │                                    │
│    ▸ large_tables    │                                    │
├──────────────────────┴────────────────────────────────────┤
│  ADQL Text:  (aqui você escreve sua consulta)             │
│                        [ Run Query ]                      │
└───────────────────────────────────────────────────────────┘
```

### Como o VizieR organiza suas 63.884 tabelas

| Pasta | Conteúdo |
|---|---|
| `I_astrometry` | Astrometria (Gaia, Hipparcos, ...) |
| `II_photometry` | Fotometria |
| `III_spectro` | Espectroscopia |
| `VI_misc`, `VII_nonstellar`, `VIII_radio`, `IX_HE` | Diversos, não estelares, rádio, altas energias |
| `J_AA`, `J_ApJ`, `J_MNRAS`, ... | **Tabelas de artigos individuais**, agrupadas por revista |
| **`large_tables`** | **"extremely large catalogs"** — os catálogos em escala de levantamento |

`J_ApJ (10721)`: mais de dez mil tabelas publicadas só no *Astrophysical Journal*.
**Esse é o verdadeiro valor do VizieR** — se um artigo publicou uma tabela, ela provavelmente
está aqui.

---

## 2.3 🔥 ARMADILHA #1: `Name` vs `Descrip`

Digite `sdss` em `Find:` com só **`Name`** marcado:

```
TAPVizieR (32/63884)     <-- só 32 resultados
```

Agora marque também **`Descrip`**:

```
TAPVizieR (642/63884)    <-- 642 resultados
```

**Vinte vezes mais resultados, por causa de um único checkbox.**

**Por quê:** com `Name` apenas, o TOPCAT busca no *identificador* da tabela (`V/154/sdss16`). A
maioria das tabelas não carrega o nome do levantamento no identificador — carrega na
**descrição**.

> **Quando uma busca não retorna nada, os dados quase certamente existem.**
> Você está buscando no campo errado.

---

## 2.4 🔥 ARMADILHA #2: onde o SDSS realmente mora

Mesmo com 642 resultados, o catálogo principal do SDSS **não** está em `II_photometry`, nem em
nenhuma pasta chamada `V_alguma_coisa`. Está em:

```
large_tables (8/148)      Descrição: "extremly large catalogs"   [o erro de digitação é deles]
    "II/294/sdss7"
    "II/306/sdss8"
    "V/139/sdss9"
    "V/147/sdss12"
    "V/154/sdss16"     <-- o que queremos
```

O VizieR separa os catálogos em escala de levantamento (centenas de milhões de linhas) dos
comuns, porque precisam de tratamento diferente do lado do servidor.

> **A organização de um arquivo tem uma lógica interna — mas não necessariamente a *sua*
> lógica.** Explore; não presuma.

Todas as versões estão disponíveis (DR7 → DR16). Use a mais recente, a menos que tenha um motivo
para não usar — e **diga no seu artigo qual você usou**.

---

## 2.5 As abas de metadados

Selecione `"V/154/sdss16"` com um clique, e percorra as abas da direita:

| Aba | O que mostra |
|---|---|
| `Service` | Info do serviço TAP. Olha uma vez e nunca mais. |
| `Schema` | Info da **pasta** (`large_tables`). O nível acima da sua tabela. |
| `Table` | Info da **sua tabela**: nome exato (**copie, não digite**), descrição, **número de linhas**. |
| **`Columns`** | **Cada coluna: nome, tipo, unidade, descrição, UCD. Esta é a aba que importa.** |
| `FKeys` | Chaves estrangeiras — como esta tabela se conecta a outras. Avançado. |

> ⚠️ **Verifique o número de linhas em `Table`.** Se disser centenas de milhões, é melhor que sua
> cláusula `WHERE` seja seletiva, ou você vai esperar uma eternidade.

---

## 2.6 Lendo `Columns` — o coração da coisa

É aqui que a oficina é ganha ou perdida.

### As colunas de que precisamos

| Coluna | Unidade | Descrição | Uso |
|---|---|---|---|
| `RA_ICRS`, `DE_ICRS` | deg | Coordenadas (ICRS) | Posição |
| **`uPmag` … `zPmag`** | mag | Magnitudes **Petrosian** | Fotometria |
| `e_uPmag` … `e_zPmag` | mag | Seus erros | Qualidade |
| **`zsp`** | — | **Redshift espectroscópico** (UCD: `src.redshift`) | Redshift |
| `e_zsp` | — | Erro em `zsp` | Qualidade |
| **`f_zsp`** | — | **Flag ZWarning** | ⚠️ Crítico |
| `zph` | — | Redshift fotométrico | Alternativa |
| `class` | — | `3=galaxy, 6=star` (**fotométrico**) | Filtro |
| **`spCl`** | — | `"GALAXY"`, `"QSO"`, `"STAR"` (**espectroscópico**) | Filtro melhor |
| `mode` | — | `1=primary` | Deduplicação |
| `clean` | — | `1=fotometria limpa` | Qualidade |

---

### 🔥 ARMADILHA #3: cinco colunas começam com `z` e significam cinco coisas diferentes

```
zsp     -->  Redshift espectroscópico        <-- o que você quer
zph     -->  Redshift fotométrico            <-- outra grandeza (mais ruidosa)
zmag    -->  Magnitude MODEL na banda z      <-- uma magnitude!
zpmag   -->  Magnitude PSF na banda z        <-- outra magnitude!
zPmag   -->  Magnitude PETROSIAN na banda z  <-- e mais outra!
```

**`zpmag` e `zPmag` diferem por UMA letra maiúscula.**

**A consulta não vai avisar você.** Ela vai rodar, retornar números, e produzir um gráfico. O
gráfico simplesmente vai estar **errado**.

---

### 🔥 ARMADILHA #4: magnitudes PSF estão erradas para galáxias

As primeiras magnitudes da lista são `upmag, gpmag, rpmag, ipmag, zpmag` — **magnitudes PSF**.
É tentador simplesmente usá-las.

**Não use.** Uma magnitude PSF ajusta a função de espalhamento pontual da imagem à fonte.

- Para uma **estrela** → perfeito. A estrela *é* uma PSF.
- Para uma **galáxia extensa** → **ajusta a PSF dentro do núcleo e joga fora o disco e o halo.**

O fluxo é sistematicamente subestimado, e o tamanho do erro **depende do tamanho angular** — ou
seja, do redshift e da morfologia. Você estaria injetando um viés correlacionado exatamente com
as variáveis que quer estudar.

Para cores é pior: galáxias têm gradientes de cor (bojos vermelhos, discos azuis), então a PSF
**superpondera o bojo**. **Sua "sequência vermelha" seria em parte um artefato de abertura.**

| Magnitude | O que é | Usar para |
|---|---|---|
| **`?Pmag`** (Petrosian) | Abertura definida pelo perfil de luz, independente do redshift | **O padrão do SDSS para galáxias** |
| `?mag` (model) | Melhor ajuste de um perfil de Vaucouleurs ou exponencial | Também válida |
| `?pmag` (PSF) | ❌ | Só estrelas |

#### ⚠️ Não misture aberturas

Se você medir `r` com abertura Petrosian e `(g−r)` com magnitudes model, **o ponto que você plota
não corresponde a nenhum objeto fisicamente consistente.**

**Escolha um sistema e use em tudo.** Aqui: **Petrosian**, magnitude *e* cor.

---

### 🔥 ARMADILHA #5: `class` não é `spCl`

| Coluna | Como classifica | Confiabilidade |
|---|---|---|
| `class` | **Fotometricamente** — pela forma na imagem | Uma *estimativa* |
| **`spCl`** | **Espectroscopicamente** — pelo espectro real | A **verdade** |

Se o objeto tem espectro, **use `spCl`**.

E atenção: `spCl` distingue **`QSO`** de **`GALAXY`**. O `class=3` fotométrico entregaria quasares
junto com suas galáxias — e um quasar é uma fonte pontual com contínuo em lei de potência. Ele vai
cair em um lugar completamente diferente do seu CMD e contaminar tudo.

---

### 🔥 ARMADILHA #6: `f_zsp` — o flag que separa profissionais de iniciantes

`f_zsp` é o flag **ZWarning** do SDSS.

```
f_zsp = 0    -->  o ajuste do redshift é confiável
f_zsp != 0   -->  o ajuste falhou, o espectro era ruim, algo deu errado
```

**Sem `f_zsp = 0`, sua amostra contém redshifts lixo.** E, mais uma vez, a consulta roda
perfeitamente.

**Todos os levantamentos têm flags assim. Encontre-os. Use-os.**

---

### Mais duas colunas de qualidade que você nunca teria adivinhado

| Coluna | Por que importa |
|---|---|
| **`mode = 1`** | O SDSS fotografa regiões sobrepostas. Sem isso, **a mesma galáxia aparece várias vezes** na sua amostra. |
| **`clean = 1`** | O flag composto de qualidade fotométrica do próprio SDSS. |

---

## 2.7 Uma nota sobre UCDs

Um **UCD** (*Unified Content Descriptor*) é um rótulo padrão que descreve o que uma coluna
*significa*, independentemente de como ela se *chama*.

`zsp` e `zph` carregam ambas `src.redshift`. Nomes diferentes, mesmo significado físico. Dois
arquivos podem chamar a mesma grandeza de `zsp`, `Z_SPEC` ou `redshift` — mas todos vão rotulá-la
`src.redshift`.

**O software pode confiar no UCD; não pode confiar no nome.** É assim que o TOPCAT sabe
automaticamente quais são as coordenadas quando você faz um cruzamento sem dizer nada.

---

## 2.8 O que levar da Parte 2

Cada uma das seis armadilhas acima **produz uma consulta que roda sem erro**.

Nenhuma lança uma exceção. Nenhuma imprime um aviso. Cada uma retorna silenciosamente a amostra
errada — e você plota, e parece perfeitamente plausível.

> **O banco de dados nunca vai avisar você de que fez a pergunta errada.**
> Esse é o seu trabalho.

---

## ✅ Checkpoint

Antes de escrever qualquer ADQL, você deve ser capaz de dizer:

- [ ] O **nome exato da tabela**, copiado da aba `Table`
- [ ] As **colunas de coordenadas** — `RA_ICRS`, `DE_ICRS`
- [ ] A **coluna de redshift** — `zsp`, e **por que não** `zph`, `zmag`, `zpmag` ou `zPmag`
- [ ] As **magnitudes** — `uPmag`…`zPmag`, e **por que não** as PSF
- [ ] A **coluna de classificação** — `spCl`, e **por que não** `class`
- [ ] Os **flags de qualidade** — `f_zsp`, `mode`, `clean`, e do que cada um protege você

---
---

# Parte 3 — Sua primeira consulta ADQL

## 3.1 A consulta

Cole na caixa **`ADQL Text`** e clique em **`Run Query`**:

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

**Resultado esperado: 7.539 linhas, 15 colunas.**

> ✅ **Checkpoint.** Todo mundo deve obter o mesmo número.
> Deu 30.000? Falta um filtro. Deu 200? Algo quebrou.

---

## 3.2 ⏳ Se a consulta estiver lenta ou falhar — use o arquivo de backup

Esta consulta pede ao VizieR que varra **centenas de milhões de linhas**. Com todo mundo
consultando ao mesmo tempo, o servidor pode ficar lento ou dar timeout.

**Não fique esperando. Se demorar mais de ~2 minutos, ou falhar:**

### 👉 Baixe `sdss_stripe82_galaxies.vot` deste repositório

```
File  →  Load Table  →  selecione sdss_stripe82_galaxies.vot  →  OK
```

É **exatamente o resultado da consulta acima**. Você não perde nada.

> **Isto não é trapaça.** É o que você faria na vida real. Assim que uma consulta lenta dá certo,
> você escreve o resultado em disco para nunca mais depender de um servidor remoto para os mesmos
> dados.

**Se sua consulta funcionou:** salve mesmo assim, agora —
`File → Save Table(s)` → formato `votable`.

---

## 3.3 O que cada linha faz

| Linha | Por que está aí |
|---|---|
| `TOP 30000` | Um limite de linhas **explícito**. Diferente do formulário web do VizieR, o ADQL nunca trunca em silêncio. |
| `RA_ICRS BETWEEN 320 AND 340` | **Stripe 82** — uma faixa longa e estreita. Impossível de pedir com um cone search. |
| `DE_ICRS BETWEEN -1.25 AND 1.25` | A Stripe 82 é estreita em declinação. *Isso* é o que a torna Stripe 82. |
| `spCl = 'GALAXY'` | Galáxias — **não QSOs, não estrelas**. |
| `zsp > 0.02` | Remove redshifts espúrios e objetos muito próximos. |
| `zsp < 0.2` | **O universo genuinamente local.** Ver §3.4. |
| **`f_zsp = 0`** | **O ajuste do redshift é confiável.** |
| `mode = 1` | Só detecções primárias — sem duplicatas. |
| `clean = 1` | O flag de "fotometria limpa" do próprio SDSS. |

> **A consulta *é* a decisão científica.**
> Cada linha do `WHERE` enviesa sua amostra, e você tem que ser capaz de defendê-la.
> Em um artigo, esta consulta vai na seção de dados.

---

## 3.4 Por que `z < 0.2` e não `z < 0.3`

| Motivo | Efeito |
|---|---|
| **A correção K encolhe.** ~0,2 mag em `r` a z=0,2, contra ~0,4 a z=0,3 | Menos resíduo sistemático |
| **A incompletude cai drasticamente.** A amostra espectroscópica do SDSS é limitada em fluxo | A z=0,3 você só vê galáxias intrinsecamente muito brilhantes — e o truncamento **depende da cor**, já que as vermelhas são mais fracas a massa fixa |
| **A cosmologia deixa de importar.** A z<0,2, Ωm = 0,3 vs 0,25 é desprezível | Você não precisa se preocupar em ter escolhido a cosmologia "certa" |

**O que você perde:** um pouco de estatística e um pouco de faixa dinâmica em $M_r$. **Um preço
barato** — a bimodalidade aparece perfeitamente bem com 7.000 galáxias.

---

## 3.5 O que a consulta **não** filtra — de propósito

- ❌ Sem cortes de magnitude
- ❌ Sem cortes de cor

**Deliberado.** Os dados que você está prestes a carregar estão **sujos**, e descobrir isso — e
decidir o que fazer a respeito — é o objetivo da Parte 4.

Repare na ironia: a consulta *contém* **`clean = 1`**, o flag de qualidade fotométrica do próprio
SDSS. E você ainda vai encontrar magnitudes de 33. Guarde esse pensamento.

---

## 3.6 🔥 ARMADILHA #7: o ADQL não distingue maiúsculas — e vai te parar

Rode a consulta **sem** as aspas duplas e você recebe:

```
Incorrect ADQL query: 6 unresolved identifiers
  - Ambiguous column name "uPmag"! It may be (at least)
    "V/154/sdss16.upmag" or "V/154/sdss16.uPmag".
  ...
```

**O parser do ADQL não distingue maiúsculas de minúsculas.** Quando você escreve `uPmag`, o
servidor genuinamente não sabe se você quer dizer `upmag` (**PSF**) ou `uPmag` (**Petrosian**).

### A solução: colocar o identificador entre aspas

```sql
"uPmag"     -->  Petrosian. Sem ambiguidade.
```

### ⚠️ Dois tipos de aspas, dois trabalhos diferentes

| Aspas | Usadas para | Exemplo |
|---|---|---|
| **Duplas** `"..."` | **Identificadores** (tabelas, colunas) | `"uPmag"`, `"V/154/sdss16"` |
| **Simples** `'...'` | **Valores de texto** | `'GALAXY'` |

Escreva `spCl = "GALAXY"` com aspas duplas e o ADQL vai procurar uma *coluna* chamada GALAXY, e
falhar.

### A lição mais profunda

**Aquela mensagem de erro salvou você.**

O parser só reclamou porque *ambas* as colunas existem. Se o catálogo tivesse apenas `upmag`, sua
consulta teria rodado perfeitamente — e entregado silenciosamente magnitudes PSF de galáxias.

> **A ambiguidade foi o que protegeu você.** Não conte com essa sorte duas vezes.

---

## 3.7 Por que a consulta é lenta: lendo a coluna `Indexed`

A demora não é um bug. É informação.

Volte para **`Columns`** e olhe a coluna **`Indexed`**:

| Coluna | Indexada? |
|---|---|
| `RA_ICRS`, `DE_ICRS`, `zsp`, `zph`, `objID` | ✅ |
| **`spCl`**, **`f_zsp`, `mode`, `clean`**, **`uPmag`…`zPmag`** | ❌ |

Uma coluna **indexada** é filtrada instantaneamente. Uma **não indexada** obriga a varrer a tabela
inteira.

Seu filtro mais lento é `spCl = 'GALAXY'`: uma comparação de **texto** em uma coluna **não
indexada**, contra centenas de milhões de linhas.

> **A consulta que você escreve determina quanto você espera.**
> Em um levantamento grande, essa é a diferença entre 10 segundos e 10 minutos.

**Para acelerar,** reduza a região do céu — sua alavanca mais barata, porque as coordenadas **são**
indexadas:

```sql
  AND RA_ICRS BETWEEN 330 AND 340    -- metade da área
```

---

## ✅ Checkpoint

- [ ] Tabela carregada — da sua consulta, ou de `sdss_stripe82_galaxies.vot`
- [ ] **Linhas = 7.539**, **Colunas = 15**
- [ ] A tabela está **salva no seu disco**

---
---

# Parte 4 — Olhando os dados e encontrando a bimodalidade

**Ainda não plote.**

> **Regra: nunca plote dados que você não olhou.**
> Cinco minutos de inspeção economizam uma tarde depurando um gráfico que saiu errado.

---

## 4.1 Olhe as linhas

**`Views → Table Data`** (o ícone de grade).

Verifique se a consulta fez o que você achava: `RA_ICRS` em [320, 340], `DE_ICRS` em
[−1,25, 1,25], `zsp` em [0,02, 0,2], `spCl` = `GALAXY` em todas.

---

## 4.2 Olhe as estatísticas — o passo que importa

**`Views → Column Statistics`** (o botão **`Σ`**).

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

## 4.3 Três coisas estão erradas aqui

### 1. Existe um nulo

`uPmag` tem **7.538** valores bons, não 7.539. **Uma galáxia não tem magnitude `u`.**

Uma galáxia não é nada. Mas o hábito é tudo: ela vai **sumir silenciosamente** de qualquer gráfico
que use a banda `u`, e o TOPCAT não vai avisar. Em outro catálogo poderiam ser 3.000.

### 2. As magnitudes contêm lixo

```
uPmag   Maximum = 33.147
iPmag   Maximum = 27.632
```

**O SDSS tem um limite de detecção de ~22 mag.** Uma galáxia de magnitude **33 não existe**.

São objetos em que o ajuste Petrosiano **falhou**: o algoritmo não convergiu, retornou um número
sem sentido, e o catálogo guardou mesmo assim.

O detalhe revelador: a *mesma* galáxia pode ser razoável em `r` (máx. 24) e absurda em `u`
(máx. 33). Isso não é física — é um ajuste quebrado.

### 3. Os erros são piores que as magnitudes

```
e_zPmag   Maximum = 44.086      um erro de 44 magnitudes
e_zsp     Maximum =  1.7997     um erro de 1,8 no redshift
```

**Pare e pense em `e_zsp`.** Seus redshifts vão de 0,02 a 0,2. **O maior erro é 1,8** — cerca de
*doze vezes* o maior redshift da amostra.

> Isso não é uma medição. É um ajuste que falhou e reportou o próprio fracasso honestamente.

---

## 🔥 ARMADILHA #8: flags de qualidade são necessários, não suficientes

Sua consulta contém **dois** flags de qualidade do próprio SDSS:

```sql
AND f_zsp = 0     -- "o ajuste do redshift é confiável"
AND clean = 1     -- "a fotometria está limpa"
```

**E você ainda obteve magnitudes de 33 e erros de redshift de 1,8.**

> Os próprios flags do levantamento dizem *este objeto está bem*. Não está.
>
> **Flags de qualidade são necessários mas não suficientes. Sempre olhe os dados você mesma.**

---

## 4.4 Por que isso destruiria seu gráfico

Uma galáxia com `uPmag = 33` e `rPmag = 17` dá uma cor de **`u−r = 16`**. Uma galáxia real tem
`u−r` entre aproximadamente **0,5 e 3,5**.

**Esse único ponto estica seu eixo Y de 0–3,5 para 0–16.** Todo o seu sinal é esmagado em uma
faixa fina e você não vê nada.

**Teste.** Plote com o lixo dentro, olhe a bagunça, e volte.

---

## 4.5 Row subsets — limpar sem apagar

Um **subset** **não apaga linhas**. Ele cria um *rótulo* marcando quais linhas satisfazem uma
condição. Reversível, e você pode manter vários ao mesmo tempo e compará-los.

**`Views → Row Subsets`** → **`New Subset`**

| Campo | Valor |
|---|---|
| **Name** | `good` |
| **Expression** | `uPmag < 22 && gPmag < 22 && rPmag < 21 && iPmag < 22 && zPmag < 22 && (uPmag-rPmag) > 0.5 && (uPmag-rPmag) < 4` |

**Resultado: sobrevivem 6.851 de 7.539. 688 galáxias — 9,1% — eram inutilizáveis.** Não é erro de
arredondamento.

### ⚠️ As expressões do TOPCAT não são SQL

Usam uma sintaxe tipo **Java**:

| Operador | Significado |
|---|---|
| `&&` | E (AND) |
| `\|\|` | OU (OR) |
| `!` | NÃO (NOT) |
| `==` | igual (**duplo**!) |
| `!=` | diferente |

**Boa notícia:** diferente do ADQL, as expressões do TOPCAT **distinguem** maiúsculas. `uPmag`
significa `uPmag` e nada mais.

### Dois tipos de corte — e por que você precisa dos dois

**Cortes de magnitude** (`< 22`) removem objetos mais fracos do que o SDSS consegue detectar.

**Cortes de cor** (`0.5 < u−r < 4`) são os que realmente salvam você. Pense numa galáxia com
`uPmag = 21.9` (lixo, mas abaixo do corte) e `rPmag = 17` (bom). Ela **passa no filtro de
magnitude** — e entrega `u−r = 4.9`.

> **Filtrar as magnitudes não basta. Você tem que filtrar a grandeza que efetivamente plota.**

### 🎓 Exercício

Construa um segundo subset usando os **erros**:

```
e_zsp < 0.001 && e_uPmag < 0.5 && e_rPmag < 0.2
```

Quantas sobrevivem? A bimodalidade ainda aparece? **Sua conclusão depende de qual estratégia de
limpeza você escolheu?**

Se depende, isso é importante. Se não depende, *também* é importante — seu resultado é robusto.

---

## 4.6 Lendo `Rows: 7539 (6851 apparent)`

```
Rows:        7.539 (6.851 apparent)
Row Subset:  good
```

- **`7.539`** — as linhas que a tabela *tem*. Nada foi apagado.
- **`6.851 apparent`** — as linhas que o TOPCAT *vai mostrar* a você.

### ⚠️ A armadilha clássica

**Tudo o que você fizer agora — gráficos, estatísticas, cruzamentos — usa apenas o subset ativo.**

Daqui a meia hora você vai obter um número sem sentido, e vai ser porque esqueceu que havia um
subset ativo.

> **Se um resultado te surpreende, a primeira coisa que você verifica é qual subset está ativo.**

**Verifique:** volte a `Column Statistics`, coloque `Subset for calculations: good`. O máximo de
`uPmag` agora deve estar abaixo de 22, não em 33. E `nGood` deve ser **6.851 em todas as
colunas**.

---

## 4.7 Magnitude absoluta — deixando o eixo X físico

A magnitude aparente **não é uma propriedade da galáxia**. Depende da distância. Um eixo em `r`
aparente mistura "galáxias intrinsecamente brilhantes longe" com "galáxias fracas perto". Isso é
geometria, não física.

Você tem `zsp`. Use.

$$M_r = m_r - \mathrm{DM}(z) - K_r(z,\ u{-}r)$$

### Primeiro: explore o que o TOPCAT já tem

**`Help → Available Functions`** abre o **Function Browser**:

| Classe | Contém |
|---|---|
| **`Distances`** | `luminosityDistance(z, H0, omegaM, omegaLambda)`, `lookbackTime`, `zToAge`. Baseado em **Hogg (2000)**, astro-ph/9905116. |
| **`KCorrections`** | `kCorr(filter, redshift, colorType, colorValue)`. Baseado em **Chilingarian, Melchior & Zolotukhin (2010)**, válido para **0 < z < 0,5**. |

> **Esta janela é a aba `Columns`, mas para operações.** O mesmo hábito: **explore antes de
> presumir.** Não escreva à mão uma aproximação de algo que a ferramenta já faz direito.

Constantes de que você precisa: **`KCF_r`** (o filtro r), **`KCC_ur`** (a cor u−r do SDSS).

### Crie a coluna

**`Views → Column Info`** → **`Columns → New Synthetic Column`** (ou o botão **`f(x)`**).

> ⚠️ O menu `Columns` fica **dentro da janela `Table Columns`**, não na barra de menu principal.

| Campo | Valor |
|---|---|
| **Name** | `Mr` |
| **Units** | `mag` |
| **Expression** | `rPmag - (5*log10(luminosityDistance(zsp, 70, 0.3, 0.7)) + 25) - kCorr(KCF_r, zsp, KCC_ur, uPmag - rPmag)` |

| Termo | Significado |
|---|---|
| `luminosityDistance(zsp, 70, 0.3, 0.7)` | $d_L$ em **Mpc**; ΛCDM plana, $H_0=70$, $\Omega_m=0.3$, $\Omega_\Lambda=0.7$ |
| `5*log10(d_L) + 25` | o **módulo de distância** (o `+25` converte Mpc para a escala de 10 pc) |
| `kCorr(KCF_r, zsp, KCC_ur, uPmag-rPmag)` | a **correção K** em `r`, usando a cor `u−r` observada |

**Verifique:** `Mr` deve ir de aproximadamente **−24 a −17**. Valores positivos significam que
algo deu errado.

### Por que a correção K importa mais do que parece

A correção K **depende da cor da galáxia**. Uma vermelha e uma azul no mesmo redshift recebem
correções *diferentes*.

**Se você a omite, introduz um erro correlacionado com o próprio eixo que quer medir.** Você
estaria deslocando suas duas populações em quantidades diferentes — distorcendo a separação que
quer estudar.

---

## 4.8 O que *não* corrigimos

| Não aplicado | Tamanho | Enviesa o resultado? |
|---|---|---|
| **Extinção galáctica** | ~0,05–0,1 mag em `r`, na alta latitude galáctica da Stripe 82 | **Aproximadamente um deslocamento constante.** Move o diagrama; não distorce sua estrutura interna. |
| **Função de seleção** | Grande | **Sim.** O SDSS é limitado em fluxo, e galáxias vermelhas são mais fracas a massa fixa — saem primeiro. |
| **Efeitos de abertura** | Pequeno (a Petrosian foi projetada para minimizá-los) | Levemente |

> **Omitir a extinção degrada sua precisão. Omitir a correção K enviesa seu resultado.**
> Não são o mesmo tipo de erro.

---

## 4.9 O diagrama cor-magnitude

**`Graphics → Plane Plot`**

| Campo | Valor |
|---|---|
| **X** | `Mr` |
| **Y** | `uPmag - rPmag` |
| **Axes** | marque **`X Flip`** — $M_r$ é negativa; você quer as brilhantes à esquerda |
| **Subset** | `good` |

Você obtém uma mancha vermelha sólida. **Esse é o problema.** Com ~6.800 pontos opacos desenhados
um sobre o outro, a região densa satura e você perde toda a informação de densidade.

### Adicione contornos

Painel de baixo → aba **`Form`** → botão verde **`+ Forms`** → **`Contour`**.

**Agora você vê: dois picos separados.**

- **Pico superior:** `u−r ≈ 2,7`, `Mr ≈ −21,3` → **A SEQUÊNCIA VERMELHA**
- **Pico inferior:** `u−r ≈ 1,7`, `Mr ≈ −20,3` → **A NUVEM AZUL**
- **Entre eles:** contornos esparsos → **O GREEN VALLEY**

---

## 4.10 🚨 Os dois parâmetros dos contornos — e o perigo

### `Level Count`

Quantas curvas de nível desenhar. Como um mapa topográfico: cada linha une pontos de igual
densidade.

- **Poucas (3–5):** estrutura grosseira. **Os dois picos se destacam.**
- **Muitas (15–20):** detalhe fino, mas o gráfico satura e o olho perde a estrutura global.

### `Smoothing` — o perigoso

A largura do núcleo de suavização. Seus dados são pontos discretos; para desenhar contornos, o
TOPCAT primeiro estima uma densidade *contínua* fazendo a média de cada ponto com seus vizinhos.

| Smoothing | Efeito |
|---|---|
| **Baixo (5–10)** | Contornos rugosos e ruidosos. **Aparecem picos falsos** por flutuações estatísticas. |
| **Médio (~50)** | Equilibrado. A estrutura real emerge; o ruído é suprimido. |
| **Alto (200+)** | **Tudo se funde em uma única mancha.** ⚠️ **A bimodalidade desaparece.** |

### Teste. Agora mesmo.

1. Coloque `Smoothing` em **200** → os dois picos **se fundem em um**. Você concluiria que não há
   bimodalidade.
2. Coloque em **5** → aparecem grumos por toda parte. Você concluiria que há cinco populações.

**Nenhuma das duas conclusões está correta.** Ambas são artefatos de um parâmetro que *você*
escolheu.

> **Se seu resultado depende criticamente de um parâmetro de visualização, não é um resultado.**
> Verifique com um método que não dependa desse parâmetro.

---

## 4.11 A verificação independente: um histograma 1-D

**`Graphics → Histogram`** → **X = `uPmag - rPmag`**, subset `good`.

**Sem suavização. Sem núcleo. Sem parâmetros livres.** Só contagens em bins.

- **Pico 1** em `u−r ≈ 1,75` → **nuvem azul**
- **Um vale** em `u−r ≈ 2,3` → **green valley**
- **Pico 2** em `u−r ≈ 2,65` → **sequência vermelha**

**Duas corcovas e um vale, direto das contagens brutas.**

**Esta é a verificação.** A bimodalidade dos contornos *não* era um artefato da suavização —
está nos dados.

---

## 4.12 O que você acabou de mostrar

> As galáxias **não** se distribuem continuamente em cor. **Elas vêm em duas famílias.**
>
> **As azuis** estão formando estrelas ativamente — estrelas jovens e quentes emitem no UV, então
> `u` é brilhante e `u−r` é pequeno.
>
> **As vermelhas** pararam de formar estrelas — só restam estrelas velhas, então `u−r` é grande.
>
> **E o vale entre elas é pouco povoado** — o que significa que a transição de uma para a outra é
> **rápida**. As galáxias não ficam muito tempo no meio.

Um resultado científico real, obtido em noventa minutos, sem escrever uma única linha de código.

---

## 4.13 ⚠️ Uma coisa que você NÃO pode concluir

O pico azul é **mais alto** que o vermelho.

**Isso *não* significa que há menos galáxias vermelhas no universo.** Significa que há menos na
*sua amostra* — e sua amostra tem um **viés de seleção**.

O SDSS é limitado em fluxo. A massa estelar fixa, as galáxias vermelhas são **mais fracas** que as
azuis, então saem primeiro de um levantamento limitado em fluxo.

> **Qualquer contagem relativa entre populações é enviesada pela função de seleção.**
> Fazer isso direito exige calcular volumes máximos ($V_{max}$) e ponderar cada galáxia.
> Isso é um artigo, não uma aula.

---

## ✅ Checkpoint

- [ ] Você encontrou a fotometria lixo **e** os erros absurdos **antes** de plotar
- [ ] Construiu o subset `good` — **sobrevivem 6.851 de 7.539**
- [ ] Cortou em **cor**, não só em magnitude
- [ ] `Mr` vai de aproximadamente −24 a −17
- [ ] O gráfico de contornos mostra **dois picos**
- [ ] O histograma os **confirma independentemente**
- [ ] Você sabe nomear **três coisas que não corrigiu**
- [ ] Você sabe explicar por que `f_zsp = 0` e `clean = 1` **não bastaram**

---
---

# Parte 5 — Clicando em uma galáxia: de um número a uma imagem

Você tem dois picos em um histograma. Até agora são pura estatística.

**Agora vamos ver o que esses números realmente são.**

---

## 5.1 Activation Actions — o recurso que ninguém encontra

O TOPCAT pode **fazer alguma coisa quando você clica em uma linha** — numa tabela, ou num ponto de
um gráfico.

**`Views → Activation Actions`** (ou o botão amarelo **⚡**).

```
☑ Use Sky Coordinates in      <-- ativa por padrão
☐ Send Sky Coordinates         (SAMP -> Aladin, DS9, ...)
☐ Display HiPS cutout          <-- o que queremos
☐ Send HiPS cutout
☐ Delay
☐ Execute code
☐ Run system command
☐ Send row index
```

### ⚠️ A armadilha desta janela

**O painel `Configuration` da direita mostra as configurações da ação que está SELECIONADA — não
da que está MARCADA.**

Você tem que fazer **duas coisas separadas**:

1. **Marcar o checkbox** de `Display HiPS cutout` → isso a *habilita*
2. **Clicar no nome dela** → isso a *seleciona*, para que você possa configurá-la

Se você só marcar, vai ficar olhando sem entender por que o painel da direita ainda mostra as
configurações de outra coisa.

---

## 5.2 Configurando o recorte

| Campo | Valor | Notas |
|---|---|---|
| **RA Column** | `RA_ICRS` | já deve estar preenchido |
| **Dec Column** | `DE_ICRS` | idem |
| **HiPS Survey** | `SDSS9/color-alt` | clique em **`Select`** para explorar |
| **Field of View** | **`0.02`** graus | ver abaixo |
| **Size in Pixels** | `300` | está bom assim |

### ⚠️ Acerte o Field of View

O padrão costuma ser **`0.2` graus = 12 arcmin**. **Isso é enorme.**

Uma galáxia a z ≈ 0,1 tem tamanho angular de aproximadamente **10–20 arcsec**. Com 0,2° você
recebe um campo largo com dezenas de galáxias, e a sua é um pontinho no meio.

**Use `0.02` graus** (≈ 1,2 arcmin). A galáxia vai preencher o quadro.

> O tamanho angular depende do redshift: uma galáxia a z=0,05 vai parecer grande, uma a z=0,19 vai
> ser minúscula. Um FoV fixo é um compromisso.

---

## 5.3 Agora clique no gráfico

Volte ao seu **diagrama cor-magnitude** e clique diretamente sobre um ponto.

**Abre uma janelinha com a imagem SDSS daquela galáxia.**

O painel `Results` registra cada clique. Se você clicar rápido, vai ver entradas `CANCELLED` — uma
requisição nova substituiu uma pendente. Normal.

---

## 5.4 🔬 O experimento

### Clique numa galáxia do pico VERMELHO — `u−r ≈ 2,7`, `Mr ≈ −21`

> **Uma elíptica.** Amarelada. Lisa. Um gradiente suave do centro para fora.
> **Sem estrutura nenhuma.**

### Clique numa galáxia do pico AZUL — `u−r ≈ 1,7`, `Mr ≈ −20`

> **Estrutura.** Um núcleo brilhante cercado por um disco mais difuso e azulado. Assimetria.
> Extensão.

### É assim que deve ficar

![Diagrama cor-magnitude com duas galáxias](images/cmd-two-galaxies.png)

**Recorte de cima:** clique no pico vermelho — elíptica lisa.
**Recorte de baixo:** clique no pico azul — disco com estrutura.

**O mesmo gráfico. Dois cliques. Dois tipos diferentes de galáxia.**

---

## 5.5 O que você acabou de demonstrar

> Você selecionou essas galáxias por **um número** — a cor `u−r`, calculada a partir de duas
> magnitudes de um catálogo.
>
> Você nunca olhou uma imagem. Nunca disse nada sobre a forma.
>
> **E ainda assim, os dois picos do histograma correspondem a dois tipos de galáxia que
> *parecem diferentes*.**
>
> A cor mede a população estelar. A população estelar reflete a história de formação estelar. E a
> história de formação estelar está ligada à estrutura.
>
> **Os picos não eram estatística. Eram física.**

---

## 5.6 ⚠️ Mas não exagere

A galáxia azul em que você clicou provavelmente **não é uma espiral de livro-texto com braços de
grande desenho**. É mais provável que seja um disco difuso com bojo.

- A z ≈ 0,1, o **seeing** do SDSS (~1,4″) borra a estrutura fina. Os braços se perdem.
- A cor mede **populações estelares**, não morfologia. A correlação é forte — mas **não é uma
  lei**.
- **Existem espirais vermelhas** (apagadas, mas com disco). **Existem elípticas azuis** (com
  formação estelar recente).

> **A cor prevê a morfologia. Não a determina.**

---

## 5.7 Outras coisas que as Activation Actions podem fazer

| Ação | Uso |
|---|---|
| **`Send Sky Coordinates`** | Via **SAMP** — faz o Aladin ou o DS9 pular para aquele objeto, ao vivo, com catálogos sobrepostos. |
| **`Send HiPS cutout`** | Envia a imagem para outro aplicativo. |
| **`Execute code`** | Executa uma expressão arbitrária sobre a linha clicada. |
| **`Run system command`** | Lança um programa externo com os valores da linha como argumentos. |
| **`View URL`** | Abre o navegador numa URL construída a partir da linha — por exemplo, a página do SkyServer daquele objeto. |

**É assim que se inspecionam outliers.** Sempre que um ponto do gráfico parecer errado, **clique
nele e olhe o objeto.** Metade das vezes é um par sobreposto, um artefato, ou um rastro de
satélite — e só pelos números você nunca saberia.

> **A ferramenta de depuração mais subutilizada da astronomia é olhar a imagem.**

---

## ✅ Checkpoint

- [ ] `Display HiPS cutout` está **marcada** *e* **selecionada** (são coisas diferentes)
- [ ] O Field of View é **0.02**, não 0.2
- [ ] Clicar num ponto do CMD abre uma imagem
- [ ] Galáxia do pico vermelho → **elíptica lisa**
- [ ] Galáxia do pico azul → **estrutura**
- [ ] Você sabe explicar por que a correlação é **forte mas não absoluta**

---
---

# Parte 6 — Cruzamento: a cor realmente prevê a morfologia?

Na Parte 5 você clicou em duas galáxias e viu que a vermelha era lisa e a azul tinha estrutura.
**Duas galáxias são uma anedota.**

Agora vamos testar com **três mil**.

**O detalhe:** a morfologia não está no SDSS. Ela vive num catálogo completamente diferente,
produzido por outras pessoas com outro método. Para responder à pergunta **você não tem escolha
senão fazer um cruzamento.**

Este é o argumento inteiro do Observatório Virtual, em um único exercício.

---

## 6.1 O catálogo: Galaxy Zoo 1

O **Galaxy Zoo** pediu a centenas de milhares de voluntários que olhassem imagens do SDSS e as
classificassem a olho. O Galaxy Zoo 1 (Lintott et al. 2011, MNRAS 410, 166) é o original.

No VizieR a tabela é:

```
J/MNRAS/410/166/galaxies
```

> **Para encontrá-la você mesma:** na janela TAP, marque **`Name` e `Descrip`**, busque
> `galaxy zoo`, e expanda `J_MNRAS`. (Lembre da Armadilha #1 — só com `Name` você não acha nada.)

### As colunas que importam

| Coluna | Tipo | Descrição |
|---|---|---|
| `RAJ2000`, `DEJ2000` | DOUBLE (deg) | Coordenadas SDSS |
| **`fE`** | SMALLINT | **Flag `[0/1]` "Elliptical"** |
| **`fS`** | SMALLINT | **Flag `[0/1]` "Spiral"** |
| **`fU`** | SMALLINT | **Flag `[0/1]` Uncertain** |
| `objID` | CHAR(18) | Identificador de objeto do SDSS |
| `pDK`, `pDKm` | DOUBLE | Fração de votos "Don't Know" |
| `Nt`, `N`, `Nt1`, `Nt2` | SMALLINT | Contagens de votos |

---

## 6.2 ⚠️ O que `fS` realmente significa — leia isto antes de escrever conclusões

**O VizieR rotula `fS` como "Spiral". Não leve isso ao pé da letra.**

O Galaxy Zoo 1 **não** perguntou aos voluntários *"isto é uma espiral?"*. Ele perguntou, mais ou
menos:

> *"Este objeto é **liso e arredondado**, ou tem **alguma estrutura / características**?"*

Então a **classe `fS` é na verdade "não-elíptica, com estrutura visível"**. Ela contém:

- espirais genuínas
- **discos sem braços claros**
- **lenticulares (S0)**
- **discos de perfil (edge-on)**
- **irregulares**

**Isso não é a mesma coisa que "espiral" no sentido morfológico estrito.**

> **Seja precisa no que você afirma.** Você está testando se a cor prevê **"liso vs.
> estruturado"** — não se ela prevê tipo de Hubble.
>
> Continua sendo um resultado real e interessante. É simplesmente um resultado *diferente*.

### E os flags são um *corte*, não uma medição

`fE` e `fS` são flags binários derivados de **frações de voto** — um objeto só é marcado quando
uma grande maioria dos voluntários concordou (e depois de corrigir por viés de classificação).

**Galáxias ambíguas ficam com `fU = 1`** e não são marcadas como **nenhuma das duas**.

### 🎓 Exercício (faça antes de tirar conclusões)

**Quantas galáxias têm `fU = 1`?**

Se esse número for grande, então os objetos que você *está* classificando são só os
**visualmente mais óbvios** — e **você está vendo uma correlação mais limpa do que a que realmente
existe.**

Você selecionou por não-ambiguidade. Isso é um viés, e é um viés *na direção da sua própria
conclusão*.

---

## 6.3 Carregando o Galaxy Zoo

**`VO → Table Access Protocol (TAP) Query`** → `TAPVizieR` → `Use Service`

Cole na caixa **`ADQL Text`**:

```sql
SELECT TOP 100000
  RAJ2000, DEJ2000,
  fE, fS, fU
FROM "J/MNRAS/410/166/galaxies"
WHERE RAJ2000 BETWEEN 320 AND 340
  AND DEJ2000 BETWEEN -1.25 AND 1.25
```

**`Run Query`**

| Linha | Por quê |
|---|---|
| `RAJ2000 BETWEEN 320 AND 340` | **A mesma região da Stripe 82.** Não baixe o céu inteiro. |
| `DEJ2000 BETWEEN -1.25 AND 1.25` | idem |
| `fE, fS, fU` | Os três flags morfológicos |
| *(sem filtro nos flags)* | **Deliberado** — queremos contar as incertas |

> **Espere muito mais galáxias do que suas 7.539.** O Galaxy Zoo não exige espectro; sua amostra
> SDSS exige. **Essa assimetria importa — ver §6.6.**

---

## 6.4 ⚠️ Antes de cruzar: VERIFIQUE SEU SUBSET

Vá à janela principal, selecione sua **tabela SDSS**, e olhe **`Row Subset:`**.

Se disser **`good`**, o TOPCAT vai cruzar apenas aquelas **6.851** linhas — não as 7.539.

**É isso que queremos** — não faz sentido procurar a morfologia de galáxias cuja fotometria está
quebrada.

**Mas faça de propósito, não por acidente.** É exatamente a armadilha de §4.6: *se um resultado te
surpreende, a primeira coisa que você verifica é qual subset está ativo.*

---

## 6.5 O cruzamento

**`Joins → Pair Match (2 tables)`**

| Campo | Valor |
|---|---|
| **Algorithm** | **`Sky`** |
| **Max Error** | **`2`** arcsec |
| **Table 1** | sua tabela SDSS |
| → RA column | `RA_ICRS` |
| → Dec column | `DE_ICRS` |
| → Row Subset | **`good`** ⚠️ |
| **Table 2** | Galaxy Zoo |
| → RA column | `RAJ2000` |
| → Dec column | `DEJ2000` |
| → Row Subset | `All` |
| **Match Selection** | `Best match, symmetric` |
| **Join Type** | **`1 and 2`** ⚠️ **crítico — ver abaixo** |

**`Go`**

---

## 🔥 ARMADILHA #9: o Join Type vai mentir para você em silêncio

**Erre isso e o TOPCAT te dá uma tabela que parece perfeita e é inútil.**

| Join Type | O que retorna |
|---|---|
| **`1 and 2`** | **Só as linhas que casaram.** ✅ O que você quer. |
| `All from 1` | **Todas** as linhas da tabela 1, com as colunas da tabela 2 **vazias** onde não houve casamento |
| `All from 2` | O espelho |
| `1 or 2` | Tudo de ambas |

### Como detectar o erro

Se você usar `All from 1`, sua saída tem **exatamente 6.851 linhas** — igual à entrada.

**Parece uma taxa de casamento de 100%.** Não é. Abra a tabela e você vai ver que metade das
colunas do Galaxy Zoo está **em branco**.

> **Um cruzamento que retorna o mesmo número de linhas da tabela de entrada é um sinal de alarme,
> não de sucesso.**

**Com `1 and 2` você deve obter cerca de 3.053 linhas.**

---

## 6.6 🚨 Leia a taxa de casamento. Ela está te dizendo algo.

| | |
|---|---|
| Galáxias SDSS (`good`) | **6.851** |
| Com morfologia no Galaxy Zoo | **3.053** |
| **Taxa de casamento** | **≈ 45%** |

**Mais da metade das suas galáxias não tem morfologia.**

### Isso não é uma falha do cruzamento

A astrometria está bem (ver §6.7). O raio está bem. O método funcionou.

**O Galaxy Zoo simplesmente não classificou todas as galáxias do SDSS.** Ele tem sua **própria
função de seleção** — limites de magnitude, limites de tamanho angular, critérios de qualidade de
imagem.

> **Cruzar dois catálogos não dá a união deles.**
> **Dá a INTERSEÇÃO de duas funções de seleção.**

### ⚠️ E agora a pergunta que ninguém faz

**Essa perda de 55% é aleatória?**

**Quase certamente não.** O Galaxy Zoo precisa *resolver* uma galáxia para classificá-la. Então
você provavelmente perdeu preferencialmente:

- as **angularmente menores** → as **mais distantes**
- as **mais fracas**

**E isso está correlacionado com a cor**, porque galáxias vermelhas são mais fracas a massa
estelar fixa.

> **Sua amostra cruzada não é uma subamostra aleatória da sua amostra original.**
> **É uma subamostra enviesada — e o viés vai na direção do seu resultado.**

### 🎓 Exercício

Plote um **histograma de `zsp`** para a tabela cruzada, e compare com o mesmo histograma da
amostra completa.

**O pico se deslocou para redshift mais baixo?** Se sim, você confirmou o viés.

---

## 6.7 O passo que ninguém faz: o histograma de separações

**`Graphics → Histogram`** na tabela cruzada → **X = `Separation`**

### O que você deve ver

```
0,0 – 0,1"  →  ~2.300 galáxias   (75%)
0,1 – 0,2"  →    ~720            (24%)
0,2 – 0,5"  →     ~30
> 0,5"      →  quase nada
```

**99% dos casamentos caem dentro de 0,2 arcsec.** Uma queda brutal e imediata.

**Os casamentos são reais.** Sem ambiguidade.

### Por que é tão limpo aqui

**Ambos os catálogos usam a astrometria do SDSS.** Você não está cruzando dois telescópios
diferentes — é **o mesmo objeto, medido uma vez**. As diferenças são de arredondamento e de época
de redução.

### ⚠️ Mas olhe a cauda

Há alguns poucos casamentos em **1,8″, 2,7″, 3,2″**.

Numa distribuição em que 99% cai abaixo de 0,2″, um objeto a 3″ **provavelmente não é a mesma
galáxia** — é um **vizinho** que caiu dentro do seu raio de busca.

> **Seu raio de busca foi generoso demais.**
> Este histograma diz que **0,5″ teria bastado** — e teria eliminado os falsos positivos sem
> perder um único casamento real.

**Esta é uma iteração normal:** escolhe um raio generoso, olha o histograma, **aperta**.
Ninguém acerta de primeira.

### O contraste que vale lembrar

| Forma do histograma | O que significa |
|---|---|
| **Queda abrupta em separação pequena** | Os casamentos são reais. Confie. |
| **Plano até o raio de busca** | **Você está casando ruído.** Os "casamentos" são vizinhos aleatórios. |

**Sempre olhe. Custa dez segundos.**

---

## 6.8 A recompensa: sobrepor morfologia no CMD

Na sua **tabela cruzada**, crie dois subsets:

**`Views → Row Subsets → New Subset`**

| Name | Expression |
|---|---|
| `elliptical` | `fE == 1` |
| `spiral` | `fS == 1` |

*(lembre: `==` é duplo, e veja §6.2 — `fS` realmente significa "estruturada", não "espiral")*

Agora plote:

**`Graphics → Plane Plot`**

| Campo | Valor |
|---|---|
| **X** | `Mr` |
| **Y** | `uPmag - rPmag` |
| **Axes** | `X Flip` |
| **Layers** | sua amostra completa como **contornos**, mais os dois subsets como **pontos coloridos** |

---

## 6.9 O que você vê

**As galáxias `fE` caem no pico VERMELHO.** `u−r ≈ 2,8–3,5`, e são as mais luminosas
(`Mr ≈ −22 a −23`).

**As galáxias `fS` caem no pico AZUL.** `u−r ≈ 1,8–2,3`, e menos luminosas.

> **Você dividiu as galáxias com um número.**
> **Um catálogo independente as dividiu com olhos humanos olhando fotos.**
> **Vocês chegaram à mesma divisão.**

Essa é a confirmação quantitativa do que você viu a olho com dois recortes na Parte 5.

---

## 6.10 🚨 Agora olhe as exceções — é aí que está a ciência

**Há pontos vermelhos embaixo.** Galáxias `fE` em `u−r ≈ 2,2` — em plena zona azul.

**Há pontos azuis em cima.** Galáxias `fS` em `u−r ≈ 3,0` — em plena sequência vermelha.

**Essas são DISCOS VERMELHOS e ELÍPTICAS AZUIS.**

**Não são erros.** São objetos reais, e são **interessantes**:

| Objeto | O que provavelmente é |
|---|---|
| **Disco vermelho** | Um disco **apagado** (quenched) — parou de formar estrelas mas manteve a estrutura. Ou uma espiral de perfil, avermelhada pela própria poeira. |
| **Elíptica azul** | Uma elíptica com **formação estelar recente** — talvez uma fusão recente, ou gás acretado. |

> **O grosso da população é chato. As exceções são os artigos.**

---

## 6.11 A conclusão real

> A cor e a morfologia **se correlacionam fortemente, mas não são a mesma coisa.**
>
> **A cor** mede a **população estelar**: há estrelas jovens?
> **A morfologia** mede a **estrutura dinâmica**: há disco?
>
> **Que elas se correlacionem é um fato físico profundo** — significa que os processos que
> desligam a formação estelar estão ligados aos que destroem discos.
>
> **E que *não* se correlacionem perfeitamente é igualmente profundo** — significa que os dois
> podem ser desacoplados. Uma galáxia pode apagar sem perder seu disco.

---

## ✅ Checkpoint

- [ ] Você sabe que **`fS` significa "estruturada", não estritamente "espiral"** (§6.2)
- [ ] Contou as galáxias com **`fU = 1`** e sabe o que esse viés faz com você
- [ ] Usou **`Join Type: 1 and 2`**, e sabe explicar o que `All from 1` teria feito
- [ ] Taxa de casamento ≈ **3.053 / 6.851 = 45%**, e você sabe explicar **por que não é 100%**
- [ ] Plotou o **histograma de `Separation`** e ele **cai abruptamente**
- [ ] Você sabe dizer que raio de busca **deveria** ter usado, e por quê
- [ ] Elípticas caem no pico vermelho; estruturadas no azul
- [ ] **Você encontrou as exceções**, e sabe dizer o que podem ser

---
---

# Apêndice — As doze armadilhas

Cada uma delas **produz um resultado que parece bom**. Nenhuma lança erro.

## Encontrar os dados

| # | Armadilha | O que custa a você |
|---|---|---|
| **1** | Buscar em `Name` mas não em `Descrip` | Você conclui que os dados não existem. Existem — 20× mais. |
| **2** | Presumir onde um catálogo mora | O SDSS está em `large_tables`, não em `II_photometry` |

## Escolher as colunas

| # | Armadilha | O que custa a você |
|---|---|---|
| **3** | `zsp` / `zph` / `zmag` / `zpmag` / `zPmag` | Cinco colunas, a uma letra de distância, cinco significados diferentes |
| **4** | Magnitudes PSF para galáxias | Uma "sequência vermelha" que é em parte um artefato de abertura |
| **5** | `class` em vez de `spCl` | Quasares contaminando sua amostra de galáxias |
| **6** | Esquecer `f_zsp = 0` | Redshifts lixo na sua amostra |
| **7** | Identificadores sem aspas no ADQL | Ambiguidade — **ou pior, seleção silenciosa da coluna errada** |

## Confiar nos dados

| # | Armadilha | O que custa a você |
|---|---|---|
| **8** | Confiar só nos flags de qualidade | `clean = 1` **e** `f_zsp = 0`, e *ainda assim* magnitudes de 33 e erros de redshift de 1,8 |
| **12** | Confiar no rótulo de uma coluna | `fS` diz "Spiral". Na verdade significa "não lisa". |

## Cruzamento

| # | Armadilha | O que custa a você |
|---|---|---|
| **9** | `Join Type` errado | Uma tabela com o número "certo" de linhas e metade das colunas vazias — **parece um casamento de 100%** |
| **10** | Nunca olhar o histograma de separações | Você não consegue distinguir um casamento real de um vizinho aleatório |
| **11** | Presumir que um cruzamento preserva sua amostra | **Você obtém a interseção de duas funções de seleção — e a perda não é aleatória** |

---

## A única coisa a lembrar

> **O banco de dados nunca vai avisar você de que fez a pergunta errada.**
>
> **O gráfico nunca vai avisar você de que plotou a coluna errada.**
>
> **O flag nunca vai avisar você de que deixou passar algo.**
>
> **O cruzamento nunca vai avisar você de que jogou fora metade da sua amostra — e que a metade
> jogada fora não era aleatória.**
>
> **Esse é o seu trabalho.**

---

## O que você realmente aprendeu

Não TOPCAT. TOPCAT são botões; isso se procura.

O que você aprendeu foi um **hábito de desconfiança**, aplicado numa ordem específica:

1. **Explore o esquema antes de consultar.** Nomes mentem; descrições e UCDs não.
2. **Olhe as estatísticas antes de plotar.** Mínimos, máximos, nulos.
3. **Filtre a grandeza que você efetivamente plota** — não só seus ingredientes.
4. **Verifique cada resultado com um método que não compartilhe as premissas do primeiro.**
   Contornos têm um parâmetro de suavização; um histograma não.
5. **Clique nos outliers e olhe a imagem.** É a ferramenta de depuração mais subutilizada da
   astronomia.
6. **Pergunte o que sua amostra perdeu, e se a perda foi aleatória.** Quase nunca foi.
7. **Diga em voz alta o que você não corrigiu.**

Um pipeline que roda não é um pipeline que está certo.
