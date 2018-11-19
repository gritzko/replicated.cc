# RON-RDT Composition

How to model high-level user data types in RON

- Use object UUID to embed one object into another.
- For any type looking like an ordered collection, it is recommended to use *RGA*.
- For any type looking like an unordered collection, it is recommended to use *OR-Set*.
- For product types (structures, objects), it is recommended to use *LWW-per-field*.