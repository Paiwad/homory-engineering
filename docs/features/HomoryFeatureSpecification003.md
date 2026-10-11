# HOM-F003 — Location & Container Management

> **Product:** Homory — Memory for your home  
> **Feature ID:** HOM-F003  
> **Version:** 1.0  
> **Status:** Ready for Development  
> **Priority:** P0  
> **Dependencies:** HOM-F001 Item Management; HOM-F002 attachments remain unaffected  
> **Platform:** Native Android (Kotlin, Jetpack Compose, Room)  
> **Architecture:** MVVM + Clean Architecture, offline-first

## 1. Overview

### Problem
Users can register household items, but cannot identify where they are physically stored.

### Goal
Allow users to organize storage locations and containers hierarchically, assign items to exactly one storage destination, navigate the hierarchy, and move items without losing inventory records.

### Example

```text
My Home
├── Living Room
│   └── TV Cabinet [Location]
│       └── HDMI Cable [Item]
├── Kitchen
│   └── Coffee Corner [Location]
│       └── Coffee Machine [Item]
└── Storage Room
    └── Cabinet A [Location]
        └── Box A12 [Container]
            ├── USB Hub [Item]
            └── Power Bank [Item]
```

## 2. Scope

### In scope (v1)
- Create, list, view, rename, and delete locations.
- Nest locations under other locations (unlimited logical depth; UI must handle deep trees safely).
- Create, list, view, rename, and delete containers within a location or another container.
- Assign or move an existing item to a location **or** a container; unassigned items are supported.
- Show an item's resolved location breadcrumb.
- Browse and filter items by a selected location/container, including descendants.
- Protect items during location/container deletion.
- Persist everything locally with Room and work fully offline.

### Out of scope
- Multiple homes/profiles, shared ownership, cloud sync, QR generation/scanning (HOM-F006), GPS/maps, location photos, batch item moves, drag-and-drop reordering, and automated location detection.

## 3. Terminology and decisions

| Term | Meaning |
|---|---|
| Location | A physical area or storage place, e.g. Kitchen, Cabinet A. Can contain child locations and containers. |
| Container | A movable or named storage unit, e.g. Box A12. Can contain child containers and items. |
| Destination | Exactly one of `locationId`, `containerId`, or neither for each item. |
| Descendant | A node anywhere below another node in its respective tree. |
| Breadcrumb | Resolved path, e.g. `Storage Room > Cabinet A > Box A12`. |

**Decisions for v1**
1. Locations form a tree via `parentLocationId`; root locations have `null` parent.
2. Containers form a tree via `parentContainerId`; root containers have `locationId` identifying their owning location. Nested containers inherit that location; their `locationId` must equal the parent's location ID.
3. A container cannot directly own a location. Locations and containers are distinct trees linked by the root container's location.
4. An item has **at most one** destination. When assigned to a container, `locationId` on the item is `null`; the physical location is resolved through its container.
5. Names are trimmed, 1–100 characters, case-insensitive uniqueness **among siblings of the same type** (enforced in the domain layer; DB constraints where practical).
6. Delete policy: **block deletion of any location/container with descendants, containers, or directly assigned items**. Users must move/unassign items and delete child nodes first. No cascade delete of items.
7. Unassigned items remain visible in Item List.
8. No default home record is created automatically. Users create their own top-level locations.

## 4. User stories

| ID | Story |
|---|---|
| US-001 | As a user, I can create rooms/locations to organize my home. |
| US-002 | I can create sublocations such as cabinets under rooms. |
| US-003 | I can create storage boxes within a location or another box. |
| US-004 | I can assign an item to a location or container. |
| US-005 | I can see where an item is stored from Item Detail. |
| US-006 | I can move an item to a new destination. |
| US-007 | I can browse items within a location or container. |
| US-008 | I cannot accidentally delete inventory when deleting storage nodes. |

## 5. Functional requirements and acceptance criteria

### FR-001 — Location CRUD
- Create a root location or a child location under an existing location.
- Edit the location name without changing its ID or descendants.
- Display child locations, root containers, and items directly assigned to the selected location.
- Delete only when the location has **no** child locations, containers, or directly assigned items.

**Acceptance criteria**
- **AC-001:** A valid location can be created and is visible immediately.
- **AC-002:** A child location displays under the correct parent.
- **AC-003:** Blank or whitespace-only names are rejected.
- **AC-004:** Names longer than 100 characters are rejected.
- **AC-005:** Duplicate sibling names (case-insensitive, trimmed) are rejected.
- **AC-006:** Renaming preserves descendants and existing item assignments.
- **AC-007:** Deleting a non-empty location is blocked with an explanation.
- **AC-008:** Deleting an empty location requires confirmation and removes only that location.

### FR-002 — Container CRUD
- Create a container directly under a location, or nested under another container.
- Edit container name.
- Display child containers and items directly assigned to a container.
- Delete only when it has no child containers or assigned items.

**Acceptance criteria**
- **AC-009:** Container can be created within an existing location.
- **AC-010:** Nested container can be created under another container.
- **AC-011:** Nested container's owning location matches its parent container.
- **AC-012:** Duplicate sibling container names are rejected.
- **AC-013:** Non-empty container deletion is blocked.
- **AC-014:** Empty container can be deleted after confirmation.

### FR-003 — Assign and move item
- From Add/Edit Item and Item Detail, users can choose a location, a container, or `Unassigned`.
- The system stores one destination and never both IDs simultaneously.
- Moving an item must not change its item ID, attachments, `createdAt`, or other business fields; only destination and `updatedAt` may change.

**Acceptance criteria**
- **AC-015:** Item can be assigned to a location.
- **AC-016:** Item can be assigned to a container.
- **AC-017:** Moving from a location to a container clears `locationId`.
- **AC-018:** Moving from a container to a location clears `containerId`.
- **AC-019:** Item can be unassigned without being deleted.
- **AC-020:** Invalid/nonexistent destination is rejected and old assignment remains unchanged.
- **AC-021:** Moving an item preserves its photos/receipts and other metadata.

### FR-004 — Browse and breadcrumbs
- Show a hierarchy screen listing locations/containers and item counts.
- Item Detail shows a breadcrumb for the assigned destination.
- A location filter includes items assigned to that location, child locations, and containers beneath those locations.
- A container filter includes items in that container and descendant containers.
- Provide a direct-items-only option if needed later; v1 counts/filters default to including descendants.

**Acceptance criteria**
- **AC-022:** Item Detail displays the correct location path.
- **AC-023:** Unassigned items display `Not assigned`.
- **AC-024:** Filtering by location includes descendant items without duplicates.
- **AC-025:** Filtering by container includes nested-container items.
- **AC-026:** Breadcrumb updates after an item move or location/container rename.
- **AC-027:** Empty states are shown when no locations/containers/items exist.

### FR-005 — Hierarchy integrity
- Disallow setting a location/container as its own parent or a descendant as its parent.
- Validate parent existence and ownership.
- All hierarchy modifications must be atomic where they update multiple records.

**Acceptance criteria**
- **AC-028:** Self-parenting is rejected.
- **AC-029:** Circular parent chains are rejected.
- **AC-030:** Concurrent changes cannot create dangling item references or inconsistent container ownership.
- **AC-031:** All data remains available after app restart and in airplane mode.

## 6. UI specification

### 6.1 Locations screen
```text
← Locations                        [+]
-------------------------------------
My Home
  > Living Room                  (4)
  > Kitchen                      (8)
  > Storage Room                (12)
-------------------------------------
[+ Add Location]
```

### 6.2 Location detail
```text
← Storage Room                     [⋮]
Path: My Home > Storage Room

Sub-locations
  > Cabinet A

Containers
  > Box A12

Items stored here
  (Items assigned directly here)

[+ Sub-location]  [+ Container]
```

### 6.3 Item destination picker
```text
Select storage location
( ) Unassigned
( ) Kitchen
    ( ) Coffee Corner
( ) Storage Room
    ( ) Cabinet A
        ( ) Box A12 [Container]

[Cancel]                     [Save]
```

### UI states
`Loading`, `Content`, `Empty`, `Saving`, `ValidationError`, `Error`, `DeleteBlocked`, `NotFound`.

Navigation must avoid excessively deep recursive Compose rendering; use a flattened tree/list or drill-down screens for deep hierarchies.

## 7. Domain models

**Module:** `:core:model`

```kotlin
data class Location(
    val id: String,
    val name: String,
    val parentLocationId: String?,
    val createdAt: Long,
    val updatedAt: Long,
)

data class Container(
    val id: String,
    val name: String,
    val locationId: String,
    val parentContainerId: String?,
    val createdAt: Long,
    val updatedAt: Long,
)

// Extend existing Item model:
// val locationId: String? = null
// val containerId: String? = null
```

The item destination invariant is:

```kotlin
require(locationId == null || containerId == null)
```

For a production-grade API, prefer a sealed `ItemDestination` value type in the domain layer so callers cannot accidentally supply both IDs.

## 8. Room schema and migration

**Module:** `:core:database`

```kotlin
@Entity(
    tableName = "locations",
    foreignKeys = [
        ForeignKey(
            entity = LocationEntity::class,
            parentColumns = ["id"],
            childColumns = ["parentLocationId"],
            onDelete = ForeignKey.RESTRICT,
        )
    ],
    indices = [Index("parentLocationId")],
)
data class LocationEntity(
    @PrimaryKey val id: String,
    val name: String,
    val parentLocationId: String?,
    val createdAt: Long,
    val updatedAt: Long,
)

@Entity(
    tableName = "containers",
    foreignKeys = [
        ForeignKey(
            entity = LocationEntity::class,
            parentColumns = ["id"],
            childColumns = ["locationId"],
            onDelete = ForeignKey.RESTRICT,
        ),
        ForeignKey(
            entity = ContainerEntity::class,
            parentColumns = ["id"],
            childColumns = ["parentContainerId"],
            onDelete = ForeignKey.RESTRICT,
        ),
    ],
    indices = [Index("locationId"), Index("parentContainerId")],
)
data class ContainerEntity(
    @PrimaryKey val id: String,
    val name: String,
    val locationId: String,
    val parentContainerId: String?,
    val createdAt: Long,
    val updatedAt: Long,
)
```

Extend the existing `items` table with nullable `locationId` and `containerId`, plus indexes and **RESTRICT/NO ACTION** foreign keys to prevent accidental cascades. Enforce the one-destination invariant with a database `CHECK (locationId IS NULL OR containerId IS NULL)` constraint when the chosen migration strategy permits it; otherwise enforce in repository/use cases and cover with tests.

### Migration checklist
1. Increment Room database version from the **actual current version** (do not assume a version number).
2. Create `locations` and `containers` tables, indexes, and foreign keys.
3. Add nullable destination columns to `items`; existing items become unassigned.
4. If SQLite table reconstruction is needed to add foreign keys/check constraints, copy existing rows and retain all F001/F002 data and attachment relationships.
5. Register entities/DAOs with `AppDatabase`, export Room schemas, and add `MigrationTestHelper` tests.
6. **Never use destructive migration** for existing user data.

### Integrity rules
- Foreign keys guard missing parents and deletion of referenced records.
- Domain/service validation guards cycles and sibling-name collisions.
- Use Room `withTransaction` for multi-step mutations and delete precondition checks.
- Avoid cascading deletion from locations/containers to items or attachments.
- If adding support for reparenting in a future release, update descendant container ownership transactionally and reject cycles.

## 9. Repositories and use cases

**Modules:** `:domain:inventory`, `:data:inventory`

Suggested repository interfaces:

```kotlin
interface LocationRepository {
    fun observeRoots(): Flow<List<Location>>
    fun observeChildren(parentId: String): Flow<List<Location>>
    fun observeLocation(id: String): Flow<Location?>
    suspend fun create(name: String, parentId: String?): Result<String>
    suspend fun rename(id: String, name: String): Result<Unit>
    suspend fun deleteEmpty(id: String): Result<Unit>
}

interface ContainerRepository {
    fun observeByLocation(locationId: String): Flow<List<Container>>
    fun observeChildren(parentContainerId: String): Flow<List<Container>>
    suspend fun create(
        name: String,
        locationId: String,
        parentContainerId: String?,
    ): Result<String>
    suspend fun rename(id: String, name: String): Result<Unit>
    suspend fun deleteEmpty(id: String): Result<Unit>
}
```

Extend `ItemRepository` with a single atomic `setDestination(itemId, destination)` operation. Use cases:

- `CreateLocationUseCase`, `RenameLocationUseCase`, `DeleteLocationUseCase`
- `CreateContainerUseCase`, `RenameContainerUseCase`, `DeleteContainerUseCase`
- `AssignItemDestinationUseCase`, `ObserveItemLocationPathUseCase`
- `ObserveItemsInLocationUseCase`, `ObserveItemsInContainerUseCase`

Use injected ID and clock providers for deterministic tests. Represent expected failures (invalid name, duplicate sibling, nonempty delete, not found) with typed domain errors rather than parsing database exception strings.

## 10. Module responsibilities

```text
:app
 ├── :feature:item
 ├── :feature:location       <-- NEW Android Library
 └── :data:inventory

:feature:location -> :domain:inventory, :core:model, :core:designsystem
:feature:item     -> :domain:inventory, :core:model, :core:designsystem
:domain:inventory -> :core:model
:data:inventory   -> :domain:inventory, :core:model, :core:database
```

Keep Room entities and Android `Context` out of pure Kotlin domain modules. No new `:domain:location` or `:data:location` modules are required for v1.

## 11. Error handling

| Condition | Expected behavior |
|---|---|
| Invalid/blank name | Inline validation; Save disabled or rejected |
| Duplicate sibling name | Show conflict; retain input |
| Missing parent/destination | Show not found; preserve existing data |
| Attempt to delete nonempty node | Block deletion; explain what must be moved/removed |
| Attempt to create hierarchy cycle | Reject without modifying data |
| Database write failure | Show retryable error; no false success |
| Concurrent item move and node deletion | Transaction/FK constraint prevents dangling reference |
| Very deep hierarchy | UI remains navigable without stack overflow |

## 12. Testing plan

### Unit tests
- [ ] Name validation: blank, trim, max length, sibling duplicates.
- [ ] Create location and child location.
- [ ] Create root and nested container; validate owning location.
- [ ] Assign item to location, container, or unassigned.
- [ ] Moving item clears the previous destination field.
- [ ] Cycle detection for location/container hierarchies.
- [ ] Delete policy blocks nonempty nodes.
- [ ] Breadcrumb updates when ancestors are renamed.
- [ ] Descendant filtering returns correct items without duplicates.

### Room / integration tests
- [ ] FK prevents invalid parent/destination IDs.
- [ ] FK prevents deletion of referenced location/container.
- [ ] Migration preserves existing items and attachments.
- [ ] Existing items migrate with null destination IDs.
- [ ] Transaction rollback on failed mutation.
- [ ] Repeated create/delete does not leave orphan records.
- [ ] Deep hierarchy queries terminate and return correct results.

### Compose UI / manual regression
- [ ] Location list, create, edit, delete confirmation.
- [ ] Container list, create, edit, delete confirmation.
- [ ] Item destination picker, move, unassign.
- [ ] Breadcrumb display, filtering, empty states.
- [ ] Airplane mode, app restart, process recreation, rotation.
- [ ] F001 item CRUD/search still works.
- [ ] F002 photos and receipts remain linked after item moves.

## 13. Non-functional requirements

- **Offline:** No network required for any F003 action.
- **Performance:** Support at least 1,000 items and a realistic nested hierarchy without main-thread database access.
- **Consistency:** Never lose or delete item records as a side effect of location/container management.
- **Observability:** Surface typed error states without exposing private household information in logs.
- **Accessibility:** Touch targets, screen reader labels, and readable breadcrumbs.
- **Privacy:** Location and inventory metadata stay on device.

## 14. Delivery plan

### Phase 1 — Locations foundation
- [ ] Domain models and repository contracts.
- [ ] Room tables, DAO, migration and migration tests.
- [ ] Location list/create/edit/delete screens.

### Phase 2 — Item destination
- [ ] Extend Item persistence and domain model.
- [ ] Add destination picker to item flow.
- [ ] Display breadcrumb in Item Detail.
- [ ] Add item move/unassign behavior and tests.

### Phase 3 — Containers
- [ ] Container CRUD and nesting.
- [ ] Add container destinations to picker.
- [ ] Browse/filter items by container and location descendants.

### Phase 4 — Hardening
- [ ] Enforce delete policy and hierarchy integrity.
- [ ] Handle concurrency and failures.
- [ ] Regression test F001/F002, offline and persistence.

## 15. Definition of Done

- [ ] All AC-001 through AC-031 pass.
- [ ] Locations and nested locations can be managed offline.
- [ ] Containers and nested containers can be managed offline.
- [ ] Items can be assigned, moved, unassigned, and browsed by destination.
- [ ] Breadcrumbs reflect the current hierarchy.
- [ ] Deletion never cascades to inventory items or attachments.
- [ ] Hierarchy cycles and invalid references are prevented.
- [ ] Room migration preserves F001/F002 user data.
- [ ] Unit, database, and critical UI tests pass.
- [ ] README roadmap and feature documentation are updated.

## 16. Explicit follow-up decisions (not blockers)

- Whether to support reparenting/moving existing locations and containers in a later version (excluded from F003 v1).
- Whether to provide a combined search across locations, containers, and items.
- Whether to allow item counts to be cached for very large datasets.
- Whether to introduce QR labels in HOM-F006.

---

**Next feature:** HOM-F004 — Warranty & Purchase Information.
