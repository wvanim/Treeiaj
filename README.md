# **Treeiaj**, system topology:

Treeiaj : 'j' = Json format

## Présentation

It is an empty container, similar to the original idea of the PC:
a neutral box filled with electronic cards.

Treeia AI acts as a carrier for typed, XML-like structures.

To understand the limits of Treeia, imagine a diagram that represents a tree:
not to describe its content, but only to define its shape.# Treeiaj
Document format designed for AI prompts: a tree of typed components

### **Rules**

Based on JSON, this prompt enforces a **deterministic structure**: typed components, canonical ordering, and fixed layout for direct lookup.

#### **Key Benefits of Treeiaj**
- **Flexibility**: Treeiaj acts as a universal container, capable of transporting a wide variety of typed structures without being tied to a specific content format.
- **Determinism**: The typed and ordered nature of Treeiaj ensures predictable behavior, making it reliable for AI processing and data exchange.
- **Extensibility**: New types or components can be introduced in a controlled way, allowing the system to evolve while preserving backward compatibility.

### Main components

| Module                | Description                                               |
| --------------------- | --------------------------------------------------------- |
| Library of structures | Contains structure declarations                           |
| Script                | Describes the typed content of the tree                   |
| References (IDs)      | Links nodes together by identifier, forming a typed graph |

---

## Sample : The Time/Space prompt is composed of two layers:

- **treeia**: a vehicle (illustrated by the blue car), a generic container capable of carrying different contents.
<img width="300" height="300" alt="vehicule_bleu_300px" src="https://github.com/user-attachments/assets/fe005434-3257-4a70-9129-6e3f6da64a29" />

-
- **tispi**: the internal geometry (illustrated as the driver in the second image), representing the Time/Space structure itself (Piece/Face, Timeline, Tracks, Keys).
<img width="300" height="300" alt="passager_bleu_300px" src="https://github.com/user-attachments/assets/ec6977bc-1b54-4c19-ad7b-0d3fb1ae481b" />

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











