# Database Schema & Data Models Matrix

> **AI Instruction:** Inspect the database definition layers (e.g., Prisma schema, Mongoose models, SQL initialization files). Map out table schemas and entity-relationship rules using precise Markdown tabular grids.

## 1. Entity Attributes Grid

### Table Name: [e.g., Users]

| Attribute Name | Storage Data Type | Key/Modifiers | Logical Field Description |
| :--- | :--- | :--- | :--- |
| `id` | `String` | Primary Key / UUID | Unique auto-generated entity identifier. |
| `email` | `String` | Unique / Indexed | User authentication and communication lookup key. |

---

## 2. Architectural Entity Relationships

Define relational links using exact symbolic directions (e.g., One-to-Many, Many-to-Many).
* **[Model A Name]** $\rightarrow$ **[Model B Name]**: [Describe relationship mechanics and cascade delete rules here]