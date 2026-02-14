# **Treeiaj**, system topology:

Treeiaj : 'ia' = prompt / 'j' = Json format

Document format designed for AI prompts: a tree of typed components

## The Problem

With free-form JSON, AI must:
- Guess the structure and role of each element
- Keep the entire context in memory simultaneously
- Interpret relationships between components

For complex prompts, this creates ambiguity and errors.

## The Treeiaj Solution

A deterministic format where **order and role are explicit**:

1. **Types**: structure definitions (the vocabulary)
2. **Script**: concrete instances (the data)  
3. **References**: links between elements (the graph)

This separation allows AI to process each section with the right "mindset":
- Read types = learn the vocabulary
- Read script = validate and build objects
- Read references = assemble relationships

**Result**: deterministic parsing, focused attention, fewer errors.

---

## Concept

Treeiaj acts as a universal container, similar to the original PC idea:
a neutral box that receives typed components.

- **treeiaj**: a vehicle (illustrated by the blue car), a generic container capable of carrying different contents.
<img width="300" height="300" alt="vehicule_bleu_300px" src="https://github.com/user-attachments/assets/fe005434-3257-4a70-9129-6e3f6da64a29" />

-
- **tispi**: the internal geometry (illustrated as the driver in the second image), representing the Time/Space structure itself (Piece/Face, Timeline, Tracks, Keys).
<img width="300" height="300" alt="passager_bleu_300px" src="https://github.com/user-attachments/assets/ec6977bc-1b54-4c19-ad7b-0d3fb1ae481b" />


### Key Benefits

- **Determinism**: AI knows where to look for information
- **Flexibility**: transports different structures without being tied to a specific format
- **Extensibility**: new types possible while maintaining compatibility


### Main components

| Module                | Description                                               |
| --------------------- | --------------------------------------------------------- |
| Library of structures | Contains structure declarations                           |
| Script                | Describes the typed content of the tree                   |
| References (IDs)      | Links nodes together by identifier, forming a typed graph |

---

### Sample prompt

* Type definitions (schematic)

```json
{
  "types": {
    "Image": {
      "url": "string"
    },
    "Coord": {
      "x": "number",
      "y": "number"
    },
    "Piece": {
      "coord": "Coord",
      "images": ["Image"]
    }
  }
}
```

---

* Content values (Script)

```json
{
  "Piece": {
    "coord": {
      "x": 100,
      "y": 200
    },
    "images": [
      {
        "url": "mypicture"
      }
    ]
  }
}
```











