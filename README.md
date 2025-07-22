# IFO Ontology – README

> *Italian Fashion Ontology (IFO)*  •  Version 1.0  •  July 2025
>
> **Ontology IRI**: <http://www.semanticweb.org/andrea/ontologies/2025/5/IFO>
>
> Maintainers: *Marica R.* and *Andrea L. R.*  •  Contact: <andrea.lr@example.org>

---

## 1  Project overview
The IFO ontology models the **fashion domain** with a focus on items (bags, shoes, accessories), the people who design and wear them (models, stylists, designers), and events or themes relevant to fashion shows. It is intended for coursework in the *Web Semantico* class (Università di Bologna – Cesena) but can serve as a lightweight vocabulary for research prototypes and knowledge‑graph demos.

---

## 2  Namespaces
```turtle
@prefix ifo:   <http://www.semanticweb.org/andrea/ontologies/2025/5/IFO#> .
@prefix foaf:  <http://xmlns.com/foaf/0.1/> .
@prefix skos:  <http://www.w3.org/2004/02/skos/core#> .
@prefix dct:   <http://purl.org/dc/terms/> .
@prefix xsd:   <http://www.w3.org/2001/XMLSchema#> .
```

The ontology **imports** FOAF and SKOS via `owl:imports` so that reasoners can pull axioms automatically.

---

## 3  Module statistics *(after refactor)*
- **Triples**: ≈ 740
- **Classes**: 24
- **Object properties**: 11
- **Datatype properties**: 6
- **Individuals (A‑Box)**: 18 sample instances (for demonstration)

---

## 4  Core classes (T‑Box)
| Class | Description |
|-------|-------------|
| `ifo:FashionItem` | Abstract root class for tangible products. |
| `ifo:Bag`, `ifo:Shoe`, `ifo:Accessory`, `ifo:Gown`, … | Main subclasses of `ifo:FashionItem`. |
| `ifo:Person` *(≡ foaf:Person)* | Top‑level human entity. |
| `ifo:Model`, `ifo:Stylist`, `ifo:Designer` | **Subclasses** of `foaf:Person`. |
| `ifo:FashionShow` | Event where items are presented. |
| `ifo:Theme`, `ifo:Material` | Controlled vocabulary anchors (optionally link to SKOS concepts). |

---

## 5  Key object properties
| Property | Domain ⇒ Range | Notes |
|----------|----------------|-------|
| `ifo:designedBy` | `ifo:FashionItem` ⇒ `ifo:Designer` | Functional. |
| `ifo:wornBy` | `ifo:FashionItem` ⇒ `ifo:Model` | Inverse of `ifo:wears`. |
| `ifo:wears` | `ifo:Person` ⇒ `ifo:FashionItem` | — |
| `ifo:featuredIn` | `ifo:FashionItem` ⇒ `ifo:FashionShow` | — |
| `ifo:hasTheme` | `ifo:FashionShow` ⇒ `ifo:Theme` | Range can be SKOS concept. |

---

## 6  Key datatype properties
| Property | Range | Example |
|----------|-------|---------|
| `ifo:itemName` | `xsd:string` | "Leather Mini‑Bag" |
| `ifo:priceEUR` | `xsd:decimal` | 1890.00 |
| `ifo:showDate` | `xsd:date` | 2025‑02‑21 |

---

## 7  Alignment to external vocabularies
- **FOAF**
  - `ifo:Model`, `ifo:Stylist`, `ifo:Designer` `rdfs:subClassOf foaf:Person`.
  - Re‑used FOAF’s datatype properties `foaf:firstName`, `foaf:lastName`, etc.
- **SKOS**
  - Themes and materials *may* be encoded as `skos:Concept` individuals inside `ifo:FashionConceptScheme`.

---

## 8  Modeling decisions
1. **Class vs Concept** – Fashion items remain OWL classes to allow extension by subclasses (e.g. `EveningBag`). Conceptual tagging (themes, materials) is delegated to SKOS.
2. **No duplicate URIs** – Lower‑case duplicates removed; all classes in UpperCamelCase.
3. **Open‑world assumption** – No disjointness axioms among subclasses; fashion items can overlap in reality.

---

## 9  Reasoning & validation steps
1. Load `ifo_ontology_corrected.ttl` in Protégé.
2. Ensure internet access for `owl:imports`.
3. Run **HermiT** → *Ontology is consistent*.
4. Verify inferred hierarchy: `ifo:Model` under `foaf:Person`, etc.

*(Screenshot of HermiT results can be included in the project report.)*

---

## 10  Example SPARQL queries
```sparql
# Q1 – List all fashion items designed by "Franca Rossi"
PREFIX ifo: <http://www.semanticweb.org/andrea/ontologies/2025/5/IFO#>
PREFIX foaf: <http://xmlns.com/foaf/0.1/>
SELECT ?item ?name WHERE {
  ?designer foaf:firstName "Franca" ;
            foaf:lastName  "Rossi" .
  ?item ifo:designedBy ?designer ;
        ifo:itemName ?name .
}
```

```sparql
# Q2 – Retrieve every person that wears at least one bag
PREFIX ifo: <http://www.semanticweb.org/andrea/ontologies/2025/5/IFO#>
SELECT DISTINCT ?person WHERE {
  ?person ifo:wears ?bag .
  ?bag    a ifo:Bag .
}
```

```sparql
# Q3 – Upcoming fashion shows with a "Sustainability" theme
PREFIX ifo: <http://www.semanticweb.org/andrea/ontologies/2025/5/IFO#>
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>
SELECT ?show ?date WHERE {
  ?show a ifo:FashionShow ;
        ifo:hasTheme ifo:Sustainability ;
        ifo:showDate ?date .
  FILTER(?date > xsd:date("2025-07-22"))
}
ORDER BY ?date
```

---

## 11  How to cite
If you use the IFO ontology in academic work, please cite:
> Marica R. & Andrea L. R. (2025). *IFO: Italian Fashion Ontology v1.0*. Unpublished class project, University of Bologna – Cesena.

---

## 12  License
© 2025 Marica R. & Andrea L. R. – Released under **CC BY 4.0**.

