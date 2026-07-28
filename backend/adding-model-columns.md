# Adding Columns to a Production Model

> Think of a production database like a **moving train full of passengers**. You
> can add a new carriage while it's running — but you can't rip out seats people
> are sitting in, and any new seat has to work for passengers who are *already
> on board*.

This is a short, practical checklist for adding columns (or a new table) to a
model that's already live with real data.

All examples below use generic names (`users`, `orders`, `is_active`, …) so the
advice applies to any project.

---

## Remember it as CAMP 🏕️

- **C** — **C**opy the style (match how existing columns are written)
- **A** — **A**dditive only (add new, never remove what's in use)
- **M** — **M**ake it nullable (or add a `server_default`)
- **P** — **P**eek at the migration (autogenerate, then read it by hand)

> *"Set up CAMP before you touch production."*

If you remember nothing else, remember CAMP. The rest of this page is just
explaining why each letter matters.

---

## 1. Copy the house style 🏠

When you move into a new neighbourhood, you don't paint your house neon pink. You
match the street.

Look at the columns already in the file and copy exactly how they're written:
- Old style `Column(String, nullable=True)` vs new style `Mapped[str] = mapped_column(...)` — **pick whichever the file already uses.** Never mix both.
- Name your column like its siblings. If the table already has `created_by` and
  `updated_by`, then a new `approved_by` fits right in. Don't suddenly write
  `ApprovedByUserID`.

## 2. Only add. Never rearrange the furniture 🛋️

People are *using* this room right now. You can bring in a new chair. You cannot
take away the sofa someone's sitting on.

- **Add** new columns freely.
- **Never** alter or delete a column the app already depends on.
- If you're replacing an old column with a new one, add the new column *alongside*
  the old, migrate over time, and remove the old one **later** — not in the same change.

## 3. The trap everyone falls into: new columns need a default 🕳️

This is the big one. Read it twice.

Your table already has thousands of rows. When you add a new column, the database
asks: *"okay, but what do I put in this box for all the rows that already exist?"*

- If the column is **nullable** → the answer is "leave it empty (NULL)". Easy. No problem.
- If the column is **NOT nullable** → the database panics. It has no value for the
  old rows, and it's not allowed to leave them empty. **The migration crashes.**

The fix for a non-nullable column is a **`server_default`** — a value the database
itself stamps onto every existing row.

```python
# ❌ default=False alone → migration fails on existing rows
is_active = Column(Boolean, nullable=False, default=False)

# ✅ server_default fills the old rows too
is_active = Column(Boolean, nullable=False, server_default="false", default=False)
```

> **`default=`** is a *Python* thing — it only kicks in when your app inserts a new row.
> **`server_default=`** is a *database* thing — it's what backfills the rows that are
> already there. The migration needs the second one.

**Easiest path of all:** just make new columns **nullable**. Then you never touch
`server_default` and nothing can crash. Only reach for `server_default` when the
column genuinely must be non-nullable.

## 4. Foreign keys: point at the right door 🚪

A foreign key is a signpost to another table. Make sure it points at the real door:

```python
team_id = Column(Integer, ForeignKey("teams.id"), nullable=True, index=True)
```

- Get the target table name **exactly** right (`teams`, not `team`). A typo here
  breaks the migration.
- Keep it `nullable=True` unless you have a strong reason.
- Add `index=True` — it's like putting a bookmark in a book so lookups are fast.
  (Confirm with the team, since it shows up in the migration.)

## 5. Relationships: only if the house already has them 🔗

`relationship()` is optional sugar. If the existing models don't use it, don't be
the person who introduces a new pattern nobody else follows. If they do use it,
make sure both sides agree (`back_populates` matches on each end).

> Match the neighbours. Consistency beats cleverness.

## 6. The model is the blueprint; the migration is the builder 📐

Order of operations matters:

1. Write the model change first.
2. Autogenerate the migration (e.g. `alembic revision --autogenerate`).
3. **Read the generated migration with your own eyes.** Autogenerate is smart but
   lazy — it often *misses* `server_default` and index intent. Never trust it blindly.

> The model says *what the world should look like*. The migration is the set of
> instructions to get there. If the blueprint is wrong, the builder builds the wrong house.

## 7. Encrypted / sensitive fields are just strings here 🔐

A column that stores an encrypted value (say `secret_token_encrypted`) is just a
`String` in the model. The model doesn't do the locking — the **service layer**
encrypts before saving and decrypts after reading. Don't put crypto logic in the
model itself.

---

## Quick pre-flight checklist ✈️

Before you open the PR, tick these:

- [ ] Column style matches the existing file (no mixing `Column` and `Mapped`).
- [ ] Only added things — nothing existing was changed or removed.
- [ ] Every new non-nullable column has a `server_default` (or is nullable).
- [ ] Foreign keys point at the exact table name, with an index if needed.
- [ ] Relationships (if any) match on both sides.
- [ ] Migration was autogenerated **and read by hand**.
- [ ] No encryption/business logic snuck into the model.

---

## A worked mini-example

Say you have a live `users` table and you want to add an optional link to a `teams`
table, plus a flag for whether the user is active.

```python
# In the User model — additive, both safe:
team_id   = Column(Integer, ForeignKey("teams.id"), nullable=True, index=True)  # nullable → no server_default needed
is_active = Column(Boolean, nullable=False, server_default="true")              # non-nullable → server_default required
```

- `team_id` is nullable → existing users just get NULL. Safe.
- `is_active` is non-nullable → without `server_default`, the migration would crash
  on every existing user. With it, they all become `true`. Safe.

That's the whole idea in two lines.

---

*If it's additive and nullable, you're almost certainly safe. Everything else on
this page is about the moments when it isn't.*
