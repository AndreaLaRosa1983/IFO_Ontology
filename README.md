# IFO – Italian Fashion Ontology  
**Version 1.1 – July 2025**  
Ontology IRI: <http://www.semanticweb.org/andrea/ontologies/2025/5/IFO>

Maintainers: *Marica R.* & *Andrea L. R.*  ·  Contact: <andrea.lr@example.org>

---

## 1  Project overview
The **Italian Fashion Ontology (IFO)** models the contemporary fashion domain, with classes for garments and accessories, the people who create and wear them, and the shows, seasons and collections in which they appear. It ships with a small mock dataset (`ifot:` namespace) for coursework and demo purposes.

---

## 2  Namespaces
```turtle
@prefix ifo:   <http://www.semanticweb.org/andrea/ontologies/2025/5/IFO/> .
@prefix ifot:  <http://www.semanticweb.org/andrea/ontologies/2025/5/IFO/test/> .
@prefix foaf:  <http://xmlns.com/foaf/0.1/> .
@prefix skos:  <http://www.w3.org/2004/02/skos/core#> .
@prefix dct:   <http://purl.org/dc/terms/> .
@prefix xsd:   <http://www.w3.org/2001/XMLSchema#> .
```

---

## 3  Module statistics *(current)*
| Metric | Count |
|--------|-------|
| RDF triples | **802** |
| OWL / RDFS classes | **27** |
| Object properties | **41** |
| Datatype properties | **12** |
| Individuals (A‑Box) | **108** |

---

## 4  Core classes (T‑Box)
| Class | Description |
|-------|-------------|
| `ifo:Garment` | Root class for wearable items (bags, shoes, gowns, …). |
| `ifo:Bag`, `ifo:Shoe`, `ifo:Accessory`, `ifo:Outfit` | Main garment subclasses. |
| `ifo:Model`, `ifo:Stylist`, `ifo:Designer` | People subclasses imported from **FOAF**. |
| `ifo:FashionShow`, `ifo:TrunkShow` | Events where collections are presented. |
| `ifo:Pattern`, `ifo:Collection`, `ifo:Location` | Contextual concepts used as objects of properties. |

*(All garment subclasses are mutually non‑disjoint to reflect real‑world overlap.)*

---

## 5  Key object properties
| Property | Domain ⇒ Range | Inverse |
|----------|----------------|---------|
| `ifo:hasDesigner` | `ifo:Garment` ⇒ `ifo:Designer` | `ifo:isDesignerOf` |
| `ifo:hasModel` | `ifo:Garment` ⇒ `ifo:Model` | `ifo:isModelOf` |
| `ifo:hasPattern` | `ifo:Garment` ⇒ `ifo:Pattern` | — |
| `ifo:hasSeason` | `ifo:Collection`/`ifo:Garment` ⇒ `xsd:string` | — |
| `ifo:hasFit` | `ifo:Garment` ⇒ `xsd:string` | — |
| `ifo:hasLocation` | `ifo:FashionShow` ⇒ `ifo:Location` | `ifo:isLocationOf` |
| `ifo:hasCollection` | `ifo:FashionShow` ⇒ `ifo:Collection` | `ifo:isCollectionOf` |

---

## 6  Key datatype properties
| Property | Range | Example |
|----------|-------|---------|
| `ifo:itemName` | `xsd:string` | "Leather Mini‑Bag" |
| `ifo:priceEUR` | `xsd:decimal` | 1890.00 |
| `ifo:showDate` | `xsd:date` | 2025‑02‑21 |

---

## 7  Alignment to external vocabularies
* **FOAF** – `ifo:Model`, `ifo:Stylist`, `ifo:Designer` are subclasses of `foaf:Person`; FOAF name and image properties are reused.
* **SKOS** – `ifo:Pattern`, `ifo:CollectionSeason` and materials can be encoded as `skos:Concept` individuals to support multilingual labels.

---

## 8  Modeling decisions
1. **Namespaced inverses** – Most binary relations follow the `hasX` / `isXOf` convention to aid query readability.
2. **Open‑world** – No disjointness axioms among garment subclasses; the same item may belong to multiple categories.
3. **Season strings** – Values like "Spring/Summer" or "Fall/Winter" are kept as plain literals for simplicity; align to a SKOS concept scheme in production.
4. **Mock data** – Synthetic individuals live in the separate `ifot:` namespace so they can be purged easily.

---

## 9  Reasoning & validation steps
1. Load `ifo_ontology_with_mock.ttl` in **Protégé**.
2. Resolve `owl:imports` for FOAF and SKOS.
3. Run a DL reasoner (e.g. **HermiT**): ontology is **consistent**.
4. Inspect inferred hierarchy (e.g. `ifo:Model` ⊑ `foaf:Person`).

---

## 10  SPARQL examples *(mock dataset)*
> The queries below target the `ifot:` dataset shipped with the ontology. Replace uppercase placeholders when running on real data.

```sparql
PREFIX ifo:  <http://www.semanticweb.org/andrea/ontologies/2025/5/IFO/>
PREFIX ifot: <http://www.semanticweb.org/andrea/ontologies/2025/5/IFO/test/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
```

### Q1 – Designers who created a garment with a given fit in Spring/Summer
```sparql
SELECT (COUNT(DISTINCT ?designer) AS ?numDesigners)
WHERE {
  ?g a ifo:Garment ;
     ifo:hasFit "VALUE_FIT" ;
     ifo:hasSeason "Spring/Summer" ;
     ifo:hasDesigner ?designer .
}
```

### Q2 – Stylists at a trunk show for a season who presented ≥1 sustainable garment
```sparql
SELECT (COUNT(DISTINCT ?stylist) AS ?numStylists)
WHERE {
  ?show a ifo:TrunkShow ;
        ifo:hasSeason "VALUE_SEASON" ;
        ifo:hasStylist ?stylist .
  ?g a ifo:Garment ;
     ifo:hasDesigner ?stylist ;
     ifo:hasSustainabilityFeature ?feature .
}
```

### Q3 – Models who worked with >2 stylists in a season
```sparql
SELECT ?model (COUNT(DISTINCT ?stylist) AS ?numStylists)
WHERE {
  ?event a ifo:FashionShow ;
         ifo:hasSeason "VALUE_SEASON" ;
         ifo:hasModel ?model ;
         ifo:hasStylist ?stylist .
}
GROUP BY ?model
HAVING (COUNT(DISTINCT ?stylist) > 2)
```

### Q4 – All garments linked to a specific outfit
```sparql
SELECT ?garment
WHERE {
  <OUTFIT_URI>  ifo:includesGarment ?garment .
}
```

### Q5 – Occurrences of animal‑print garments worn by Black models
```sparql
SELECT (COUNT(*) AS ?occurrences)
WHERE {
  ?g a ifo:Garment ;
     ifo:hasPattern "animal print" ;
     ifo:hasModel ?model .
  ?model a ifo:Model ;
         foaf:gender "Female" ;   # if gender stored
         ifo:ethnicity "Black" .   # custom literal or SKOS concept
}
```

### Q6 – Stylists who used sandals during Autumn/Winter
```sparql
SELECT (COUNT(DISTINCT ?stylist) AS ?numStylists)
WHERE {
  ?shoe a ifo:Shoe ;
        ifo:hasType "Sandal" ;
        ifo:hasSeason "Autumn/Winter" ;
        ifo:hasStylist ?stylist .
}
```

---

## 11  Removing mock data
```sparql
DELETE WHERE {
  ?s ?p ?o
  FILTER(STRSTARTS(STR(?s), "http://www.semanticweb.org/andrea/ontologies/2025/5/IFO/test/"))
}
```

---

## 12  How to cite
> Marica R. & Andrea L. R. (2025). *IFO: Italian Fashion Ontology v1.1*. Class project, University of Bologna – Cesena.

---

## 13  License
© 2025 Marica R. & Andrea L. R. – Released under **CC BY 4.0**.
