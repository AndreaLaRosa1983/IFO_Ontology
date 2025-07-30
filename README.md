# IFO – Italian Fashion Ontology  
**Version 1.1 – July 2025**  
Ontology IRI: <http://www.semanticweb.org/andrea/ontologies/2025/5/IFO>

Maintainers: *Marica R.* & *Andrea L. R.* · Contact: <andrea.lr@example.org>

---
## 1  Project overview
The **Italian Fashion Ontology (IFO)** models the contemporary fashion domain, covering garments and accessories, the professionals who create and wear them, and the events, seasons and collections in which they appear. A small mock dataset (`ifot:` namespace) is bundled for coursework and demo purposes.

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
## 3  Module statistics *(generated 2025‑07‑30)*
| Metric | Count |
|--------|-------|
| RDF triples | **952** |
| OWL / RDFS classes | **27** |
| Object properties | **44** |
| Datatype properties | **12** |
| Individuals (A‑Box) | **132** |

---
## 4  Core classes (T‑Box)
The table lists the most frequently used classes; additional concepts such as **Brand**, **City**, **Country**, **Event**, **FashionWeekShow**, **Item**, **Showroom** and **FashionTrend** are defined in the ontology but omitted here for brevity.

| Class | Description |
|-------|-------------|
| `ifo:Garment`, `ifo:Bag`, `ifo:Shoe`, `ifo:Accessory`, `ifo:Outfit` | Hierarchy for wearable items. |
| `ifo:Model`, `ifo:Stylist`, `ifo:Designer` | Human actors, aligned to **FOAF**. |
| `ifo:FashionShow`, `ifo:TrunkShow` | Events in which collections are presented. |
| `ifo:Pattern`, `ifo:Fabric`, `ifo:Collection`, `ifo:Location`, `ifo:FashionSeason` | Contextual concepts used as objects of properties. |
| `ifo:SustainabilityFeature`, `ifo:SizeSpecification`, `ifo:FitSpecification` | Support classes for detailed attributes. |

*(Garment subclasses are intentionally **non‑disjoint** to reflect real‑world overlap.)*

---
## 5  Key object properties *(aligned to `ifo_ontology_with_skos_ranges_updated.ttl`)*
| Property | Domain ⇒ Range | Inverse (if any) |
|----------|----------------|------------------|
| `ifo:hasDesigner` | `ifo:Collection` ⇒ `ifo:Designer` | — |
| `ifo:hasModel` | `ifo:FashionShow` ⇒ `ifo:Model` | — |
| `ifo:presentsCollection` | `ifo:FashionShow` ⇒ `ifo:Collection` | — |
| `ifo:hasPattern` | `ifo:Garment` ⇒ `ifo:Pattern` | — |
| `ifo:hasSeason` | `ifo:Collection` ⇒ `xsd:string` | — |
| `ifo:hasFit` | `ifo:Garment` ⇒ `ifo:FitSpecification` | — |
| `ifo:hasLocation` | `ifo:FashionShow` ⇒ `ifo:Location` | `ifo:isLocationOf` |

---
## 6  Key datatype properties
| Property | Range | Example |
|----------|-------|---------|
| `ifo:itemName` | `xsd:string` | "Leather Mini‑Bag" |
| `ifo:priceEUR` | `xsd:decimal` | 1890.00 |
| `ifo:showDate` | `xsd:date` | 2025‑02‑21 |

*(`ifo:hasFit` is **not** a datatype property; see § 5)*

---
## 7  Alignment to external vocabularies
* **FOAF** – `ifo:Model`, `ifo:Stylist`, `ifo:Designer` are subclasses of `foaf:Person`; FOAF name and image properties are reused.
* **SKOS** – `ifo:Pattern`, `ifo:CollectionSeason` and materials can be encoded as `skos:Concept` individuals to support multilingual labels.

---
## 8  Modeling decisions
1. **Namespaced inverses** – Binary relations follow the `hasX` / `isXOf` convention when an inverse is useful for querying.
2. **Open‑world** – No disjointness axioms among garment subclasses; the same item may belong to multiple categories.
3. **Season literals** – Values such as "Spring/Summer" or "Fall/Winter" remain plain literals for simplicity (align to SKOS in production).
4. **Mock data** – Synthetic individuals live in the separate `ifot:` namespace and can be purged easily.

---
## 9  Reasoning & validation steps
1. Load `ifo_ontology_with_mock.ttl` in **Protégé**.
2. Resolve `owl:imports` for FOAF and SKOS.
3. Run a DL reasoner (e.g. **HermiT**): ontology is **consistent**.
4. Inspect inferred hierarchy (e.g. `ifo:Model` ⊑ `foaf:Person`).

---
## 10  SPARQL examples *(mock dataset)*
Replace uppercase placeholders before execution.

### Q1 – Designers who contributed to a collection containing a garment with a given fit in Spring/Summer
```sparql
SELECT (COUNT(DISTINCT ?designer) AS ?numDesigners)
WHERE {
  ?col a ifo:Collection ;
       ifo:hasSeason "Spring/Summer" ;
       ifo:hasDesigner ?designer .
  ?g a ifo:Garment ;
     ifo:hasFit <FIT_URI> ;            # e.g. ifot:RegularFit
     ifo:includedInCollection ?col .   # assuming such linking property
}
```

### Q2 – Stylists at a trunk show for a season who presented ≥ 1 sustainable garment
```sparql
SELECT (COUNT(DISTINCT ?stylist) AS ?numStylists)
WHERE {
  ?show a ifo:TrunkShow ;
        ifo:hasSeason "VALUE_SEASON" ;
        ifo:hasStylist ?stylist .
  ?g a ifo:Garment ;
     ifo:hasSustainabilityFeature ?feature ;
     ifo:isShownIn ?show .            # link garment to event
}
```

### Q3 – Models who walked for > 2 designers in a season
```sparql
SELECT ?model (COUNT(DISTINCT ?designer) AS ?numDesigners)
WHERE {
  ?show a ifo:FashionShow ;
        ifo:hasSeason "VALUE_SEASON" ;
        ifo:hasModel ?model ;
        ifo:hasDesigner ?designer .
}
GROUP BY ?model
HAVING (COUNT(DISTINCT ?designer) > 2)
```

### Q4 – All garments included in a specific outfit
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
  ?event a ifo:FashionShow ;
         ifo:hasModel ?model .
  ?g a ifo:Garment ;
     ifo:hasPattern <ANIMAL_PRINT_URI> ;
     ifo:isShownIn ?event .
  ?model ifo:ethnicity "Black" .
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
> Marica R. & Andrea L. R. (2025). *IFO: Italian Fashion Ontology v1.1*. Class project, University of Bologna – Cesena.

---
## 13  License
© 2025 Marica R. & Andrea L. R. – Released under **CC BY 4.0**.
