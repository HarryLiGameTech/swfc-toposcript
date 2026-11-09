# Shanghai World Financial Center — TopoScript Reference Project

A real-world **TopoScript reference project** exploring how to model a complex, mixed-use high-rise through reusable floor topology, vertical transportation, and declarative access constraints.

**Reference model, not an official building map.** 

This is an independent technical case study, not a verified navigation, evacuation, accessibility, or building-operations resource. It is not affiliated with or endorsed by the building owner or operator.

## Why SWFC?

The **Shanghai World Financial Center (SWFC)** in Shanghai, China, is a useful stress test for indoor spatial modeling. Rather than treating the building as a stack of identical floors, this project represents an interconnected system of:

- **Mixed-use zones:** office floors, sky lobbies, hotel areas, observation levels, and supporting spaces.
- **Vertical transportation:** express and local elevator groups, upper/lower double-deck connections, service and freight elevators, and stairs.
- **Access boundaries:** restrictions on paths and transport stations, including potentially direction-dependent access.
- **Reusable and exceptional floors:** recurring layouts modeled as templates alongside specialized maps for transfer levels and other nonstandard floors.

The objective is to demonstrate **TopoScript modeling patterns**, not to provide authoritative or comprehensive SWFC building data.

## At a glance

| Item | In this model snapshot |
| --- | ---: |
| Declared submap instances | **102** |
| Configured Transport declarations | **54** |
| Reusable office templates | **6** (`Zone1`–`Zone6`) |
| Explicit Sky Lobby maps | **4** (`SL28`, `SL29`, `SL52`, `SL53`) |

These are counts of **model declarations** in `configuration`, not independently verified counts of physical floors, elevator cars, or operating facilities. Some declarations are simplified or incomplete.

## Key TopoScript patterns

### 1. Compose a building from reusable floor maps

A standard floor can be defined once in `Zone3` and instantiated repeatedly by `configuration`:

```toposcript
building-includes {
    submap Floor35 using Zone3
    submap Floor36 using Zone3
    vehicle L3_lo
}
```

`Floor35` and `Floor36` are distinct logical map instances. The `using` clause reuses the topology of `Zone3`; it does **not** mean every physical floor is identical. Specialized areas such as Sky Lobbies use dedicated topo-maps.

### 2. Connect independently authored maps through Transport stations

A Transport references the **configured map instance** and a node inside it:

```toposcript
transport L3_lo is Elevator {
    let params: {maxSpeed: Float, acceleration: Float, cars: Int} = {
        maxSpeed = 3.5,
        acceleration = 1.0,
        cars = 4
    }
    station F35 at Floor35::L3_hall {
        location = 152.5,
        departureRate = 0.01
    } requires TenantsOnly
    station F28 at SL28::L3_hall {
        location = 121.0,
        departureRate = 0.94
    }
}
```

*Excerpt from `L3_lo`, shortened for illustration.* The floor team owns `L3_hall`; the transport team owns `L3_lo`; both must agree on the cross-file identifier `Floor35::L3_hall`.

### 3. Represent permissions declaratively

Root-level parameters and constraints can be shared by multiple floor maps and Transports:

```toposcript
root SWFC(haveStaffCard: Bool) {
    constraint TenantsOnly {
        require haveStaffCard
    }
}
```

*Reduced illustration of the full `root` declaration; not a drop-in replacement.* A map path or a Transport station can use `requires TenantsOnly`. Constraints are evaluated using the compilation context, so different parameter sets must be validated separately. The permission rules here illustrate modeling mechanics; they are **not verified current access instructions** for SWFC.

### 4. Model double-deck connectivity with an explicit workaround

The current SWFC example represents a physically linked double-deck system using separate logical Transports such as `S1_lo` and `S1_hi`:

- `S1_lo`: lower-deck connection from `LowerLobby` (1F) to `SL28` (28F).
- `S1_hi`: upper-deck connection from `UpperLobby` (2F) to `SL29` (29F).

This preserves **logical reachability** but does not fully model synchronized deck movement, shared hardware, real-world dispatching, or queue dynamics. See [Modeling notes](docs/modeling-notes.md#double-deck-elevators-a-deliberate-abstraction).

## Repository structure

The TopoScript sources retain the project's extensionless-file layout:

```text
.
├── configuration              # Building composition and instance list
├── root                       # Shared parameters and constraints
├── Zone1 ... Zone6            # Reusable office-floor topo-maps
├── SL28, SL29, SL52, SL53     # Dedicated Sky Lobby maps
├── LowerLobby, UpperLobby     # Entrance/lobby topo-maps
├── GuestRoomFloor, RefugeFloor, ...
├── L3_lo, L3_hi, S1_lo, S1_hi, ...   # Transport sources
├── docs/
│   └── modeling-notes.md      # Design decisions and collaboration guide
├── LICENSE
└── README.md
```

`configuration` is the project manifest: **source files not included by its `submap` or `vehicle` declarations do not automatically participate in building compilation**. The optional `root` source is loaded separately. A `topo-map` represents a logical local topology and need not correspond one-to-one with an entire physical floor.

## Explore the reference project

No running service is needed to inspect the model:

1. Read [`configuration`](configuration) to identify the building's logical maps, template instances, and included Transports.
2. Inspect [`Zone3`](Zone3), then find `submap Floor35 using Zone3` in the configuration.
3. Follow `Floor35::L3_hall` from [`L3_lo`](L3_lo) into the `Zone3` topo-map.
4. Compare [`S1_lo`](S1_lo) and [`S1_hi`](S1_hi) to examine the double-deck abstraction.
5. Inspect [`SL28`](SL28) and [`root`](root) for navigation actions and access-constraint examples.
6. Read [the modeling notes](docs/modeling-notes.md) for collaboration boundaries, assumptions, and known issues.

For language syntax and execution semantics, see the [TopoScript language reference](https://harryligametech.github.io/indoor-topo-navi/language) and the [TopoNavi engine](https://github.com/HarryLiGameTech/indoor-topo-navi).

### Import and runtime validation

The [TopoNavi project importer](https://github.com/HarryLiGameTech/indoor-topo-navi/blob/main/tools/import_toposcript_project.py) can read an extensionless TopoScript source directory. From a checkout of the **TopoNavi engine** (not this data-only repository), a local import preflight can be run with:

```sh
python3 tools/import_toposcript_project.py \
  --directory /path/to/toposcript-swfc \
  --building-id swfc \
  --name "Shanghai World Financial Center" \
  --dry-run
```

**A successful dry run checks source packaging, not full compilation or route correctness.** Importing into a running TopoNavi service requires the service configuration and credentials documented in the engine repository. The engine is under active development, and this model snapshot has **not been certified as a passing runtime fixture**.

## Model scope and known limitations

This repository is intentionally a **reference implementation**, not a complete or authoritative spatial dataset.

- **Template approximation:** the `RefugeFloor` source explicitly notes that actual refuge floors differ and its layout is simplified.
- **Access semantics:** constraints represent example assumptions and must not be taken as current operational permissions or security guidance.

See [Modeling notes — Validation and open questions](docs/modeling-notes.md#validation-and-open-questions) for details.

## Contributing

Issues and pull requests about **TopoScript modeling clarity, documentation, code consistency, and reproducible validation** are welcome. 

If you make a change, explain whether it is (1) a verified factual correction, (2) an explicit modeling approximation, or (3) a purely educational example. Keep map/transport identifier contracts consistent and check the building manifest after changes.

## License and attribution

The original materials in this repository that the rights holders are authorized to license are released under the **[Apache License, Version 2.0](LICENSE)**.

This license does not grant rights to third-party architectural drawings, photographs, trademarks, building names or logos beyond applicable law, or non-public building information. Such materials must have their own permissions or remain outside the repository. SWFC is used here as an identification of the subject of an independent technical study; no endorsement is implied.

**Publication prerequisite:** confirm the copyright holders' authorization for all contributed TopoScript files and document third-party sources before making this repository public.
