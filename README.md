# Open Study Model Framework Introduction

The Open Study Model Framework (OSMF) is a domain-agnostic declarative modeling framework that decouples human knowledge from the methodology and software used to study it.

Traditional learning software tightly couples three concerns: the content being studied, the relationships and progress tracking around that content, and the study methodology itself. OSMF breaks these into distinct, interoperable entities maintainable separate from any application:

- **Data**: the information being studied
- **Relationships**: connections between entities — user progress, ordering, collections
- **Pedagogy**: the study methodology — how content is presented and practiced

OSMF defines valid data, relationships, and study logic requirements — but remains agnostic to how an application resolves queries, manages user sessions, or renders the interface. OSMF models structure, not behavior: it is a declarative modeling standard that provides the contracts; an application provides execution.

The core philosophy of the framework is Structural Interoperability.

Models enforce structural interfaces rather than domain-specific classes. This means:

- Any pedagogy (study mechanism) can operate on any data that meets its functional requirements (e.g., a "Listening Drill" works on any document with an audio file)
- User progress is a transferable relationship rather than a database lock-in

This architecture generalizes the data models of legacy systems like Anki and eliminates the rigidity of existing platforms. Users can import existing decks, hot-swap study methods, and retain their review history.

---

## OSMF Definitions

### Framework Primitives

All OSMF entities are JSON files. Models are JSON schemas that define structural contracts; Documents are JSON objects that can be validated against those schemas.

OSMF follows JSON Schema conventions: Models must declare a `$id` for unique identification; Documents must also declare a `$id` for unique identification and may declare a `$schema` to specify adherence to a Model (whether `$schema` is required is up to the Model definer). This enables multiple validation approaches: hard-typed by `$id` (referencing a specific Document), hard-typed by `$schema` (referencing Documents of a specific Model), or duck-typed by attributes (matching any Document with the required structure).

Models use the `.schema.json` file extension.

OSMF provides two primitives:

- **Data Model / Data Document**: for modeling content
- **Relational Model / Relational Document**: for modeling relationships and pedagogy

---

## Data

### Concept

Data represents the information being studied — vocabulary, facts, concepts, media, or any structured knowledge. OSMF treats data as pure content, separate from how it's organized, related to users, or studied.

### Data Model

Defines a JSON schema for a specific class of knowledge.

**Example Model:** A "Japanese Word Model":
```json
{
  "$schema": "https://raw.githubusercontent.com/AndreiGrumazescu/OpenStudyModelFramework/main/data-model.schema.json",
  "$id": "japanese-word.schema.json",
  "name": "Japanese Word Model",
  "schema": {
    "type": "object",
    "properties": {
      "kanji": { "type": "string", "description": "The word written in kanji" },
      "kana": { "type": "string", "description": "The word in hiragana/katakana" },
      "type": { "type": "string", "enum": ["noun", "verb", "adjective"] }
    },
    "required": ["kanji", "kana"]
  }
}
```

### Data Document

The information to be studied, represented as a set of attributes that implements a Data Model.

**Example Document:** A Japanese vocabulary word:
```json
{
  "$schema": "japanese-word.schema.json",
  "$id": "neko",
  "kanji": "猫",
  "kana": "ねこ",
  "type": "noun"
}
```

A Data Document can use either validation approach:
- **Hard-typed**: Explicitly declare adherence to a specific Model ID for strict validation
- **Duck-typed**: Implicitly belong to any Model whose structural requirements it satisfies

---

## Relationships

### Concept

Relationships represent connections between entities — how data relates to users, to other data, or to external resources. This includes user progress tracking, ordered collections, groupings, and any metadata that describes connections rather than content itself.

### Relational Model

Defines a JSON schema for modeling relationships between Documents or resources external to the framework.

A Relational Model specifies:

- **Connectors**: Named participants in the relationship. "Each connector defines a criteria — a JSON Schema fragment (with at least one property) describing required attributes for matching entities." As a JSON schema fragment, criteria must specify at least one constraint (a `required` array alone is sufficient); every stored relationship requires some structural contract with its connected documents. How an application resolves matching entities is outside the framework's scope.
- **Directionality** (optional): Relational Documents may specify `from` and `to` arrays naming which connectors represent the source and target of the relationship. The Model's connector definitions implicitly constrain valid values; the Document chooses actual directionality. Omit for undirected relationships.
- **Data**: Schema for literal values stored on the relationship itself.
- **Many** (optional): Schema for validating an array of relationship instances within a single document (for collections, ordered lists, etc.).

The model permits connectors to map documents in 1-1, 1-many, or many-many relationships, depending on the criteria:

- **ID criteria** → 1-1 relationship (direct; static)
- **Attribute criteria** (e.g., `type: "noun"`) → 1-many or many-many, representing relationships to documents matching those attributes

**Example Model:** An "SRS User Progress Model":
```json
{
  "$schema": "https://raw.githubusercontent.com/AndreiGrumazescu/OpenStudyModelFramework/main/relational-model.schema.json",
  "$id": "srs-user-progress.schema.json",
  "connectors": {
    "user": {
      "criteria": { "required": ["User_ID"] },
      "description": "The user studying the content"
    },
    "content": {
      "criteria": { "required": ["Content_ID"] },
      "description": "The content being studied"
    }
  },
  "data": {
    "SRS_Interval": { "type": "integer", "description": "Days until next review" },
    "easeFactor": { "type": "number", "description": "Multiplier for interval calculation" },
    "repetitions": { "type": "integer", "description": "Number of successful reviews" }
  }
}
```

### Relational Document

A concrete relationship that implements a Relational Model. It provides values for connectors (identifying specific entities) and data fields.

**Example Document:** A document implementing "SRS User Progress Model":
```json
{
  "$schema": "srs-user-progress.schema.json",
  "$id": "user-101-neko-progress",
  "connectors": {
    "user": { "User_ID": "101" },
    "content": { "Content_ID": "neko" }
  },
  "from": ["user"],
  "to": ["content"],
  "data": {
    "SRS_Interval": 7,
    "easeFactor": 2.5,
    "repetitions": 3
  }
}
```

### Collections

A single Relational Document can represent multiple relationships using the `many` pattern. Each item in the array has the same structure as a standalone relationship (connectors, data) but inherits from the parent document: top-level attributes (connectors, data, directionality) serve as defaults, and item-level attributes override their corresponding top-level values when present.

To support the `many` pattern, a Relational Model defines the `many` property with validation schemas for array items. The OSMF schema provides helper properties (`itemConnectors`, `itemData`) that let Models specify validation for each item's connectors and data without fully overriding the standard structure. If a Model needs a different many structure, it can set `manyOverride: true` to define its own schema.

**Example Model:** An "Ordered List Model":
```json
{
  "$schema": "https://raw.githubusercontent.com/AndreiGrumazescu/OpenStudyModelFramework/main/relational-model.schema.json",
  "$id": "ordered-list.schema.json",
  "connectors": {
    "item": {
      "criteria": { "required": ["$id"] },
      "description": "An item in the ordered list"
    }
  },
  "data": {
    "position": { "type": "integer", "description": "Position in the list (0-indexed)" }
  },
  "many": {
    "type": "array",
    "items": { "$ref": "https://raw.githubusercontent.com/AndreiGrumazescu/OpenStudyModelFramework/main/relational-model.schema.json#/$defs/manyItem" },
    "itemConnectors": {
      "$dynamicAnchor": "itemConnectors",
      "properties": { "item": { "required": ["$id"] } },
      "required": ["item"]
    },
    "itemData": {
      "$dynamicAnchor": "itemData",
      "properties": { "position": { "type": "integer" } }
    }
  }
}
```

**Example Document:** An ordered list of vocabulary words:
```json
{
  "$schema": "ordered-list.schema.json",
  "$id": "jlpt-n5-word-order",
  "many": [
    { "connectors": { "item": { "$id": "neko" } }, "data": { "position": 0 } },
    { "connectors": { "item": { "$id": "inu" } }, "data": { "position": 1 } },
    { "connectors": { "item": { "$id": "sakana" } }, "data": { "position": 2 } }
  ]
}
```

---

## Pedagogy

### Concept

Pedagogy represents how content is studied — the study methodology that transforms data into learning experiences. A flashcard drill, a listening exercise, a fill-in-the-blank quiz — these are pedagogies.

Traditional frameworks implement pedagogy as hardcoded application logic. OSMF models pedagogy declaratively, making study methods portable and interchangeable.

### Implementation: Pedagogy as Relational

Here's the key insight: **a pedagogy is a relationship that projects fields from connected documents rather than storing its own data.**

Consider a flashcard drill. It needs:
- A stimulus (what to show the user)
- A response (the correct answer)
- SRS scheduling state

These aren't stored on the pedagogy itself — they come from connected Data Documents and Relational Documents.

OSMF implements this using the same Relational Model primitive, with one addition: **projections**. Unlike stored relationships, pedagogy Models define only the projection slots they need — not connectors. This separation is intentional: the Model specifies *what* data is needed (abstractly), while the Document specifies *where* to get it (concretely). The Document provides both the connectors (data sources) and the mappings from projections to connector fields. This enables the same pedagogy Model to work with entirely different content types. Consequently, Relational Models that define projections are not required to define connectors, as the Document provides these."

### Projections

While stored relationships use `data` for literal values, pedagogies use `projections` for field references. A projection specifies:
- Which connector to pull from
- Which field to extract

The Relational Model defines the projection slots (what data is needed); the Relational Document defines connectors and provides the mappings (where to get it).

To ensure consistency, the OSMF schema provides a standard `projectionMapping` definition that Models reference for each slot. This enforces that all Documents provide mappings in the expected `{ connector, field }` format. If a Model needs a different projection structure, it can set `refOverride: true` to define its own schema.

**Example Model:** A "Binary SRS Flashcard Model":
```json
{
  "$schema": "https://raw.githubusercontent.com/AndreiGrumazescu/OpenStudyModelFramework/main/relational-model.schema.json",
  "$id": "binary-srs-flashcard.schema.json",
  "projections": {
    "Stimulus": {
      "$ref": "https://raw.githubusercontent.com/AndreiGrumazescu/OpenStudyModelFramework/main/relational-model.schema.json#/$defs/projectionMapping",
      "description": "The prompt shown to the user"
    },
    "Response": {
      "$ref": "https://raw.githubusercontent.com/AndreiGrumazescu/OpenStudyModelFramework/main/relational-model.schema.json#/$defs/projectionMapping",
      "description": "The expected correct answer"
    },
    "SRS_Interval": {
      "$ref": "https://raw.githubusercontent.com/AndreiGrumazescu/OpenStudyModelFramework/main/relational-model.schema.json#/$defs/projectionMapping",
      "description": "Days until next review"
    }
  },
  "required": ["Stimulus", "Response", "SRS_Interval"]
}
```

**Example Document:** A "Japanese SRS Drill" implementing that model:
```json
{
  "$schema": "binary-srs-flashcard.schema.json",
  "$id": "japanese-kanji-to-kana-drill",
  "connectors": {
    "content": { "$id": "japanese-word.schema.json" },
    "userState": { "$id": "srs-user-progress.schema.json" }
  },
  "projections": {
    "Stimulus": { "connector": "content", "field": "kanji" },
    "Response": { "connector": "content", "field": "kana" },
    "SRS_Interval": { "connector": "userState", "field": "SRS_Interval" }
  }
}
```

### Why This Works

Modeling pedagogy as a relational projection enables powerful interoperability:

- **Same model, different content**: The "Binary SRS Flashcard" model works for Japanese vocabulary, medical terminology, or historical dates — just create different documents that map different fields.
- **Same content, different pedagogy**: Japanese words can be studied with flashcards, listening drills, or typing exercises — each pedagogy projects the fields it needs.
- **Composable interfaces**: A pedagogy can pull from multiple sources (content from a Data Document, scheduling from a Relational Document) without coupling to any specific implementation.

---