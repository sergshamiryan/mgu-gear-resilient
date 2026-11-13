# Gear on Cluster (resilient) — MGU Store source (OTA)

**App id:** `gear_cluster2` · **Head unit:** iDrive 7 (`mguHmi: ["ID7"]`).

Same behaviour as the original gear-on-cluster app (draws `D1…D8` / `R` / `N` /
`P` into the cluster map area via the `cluster-render` socket, with the colour /
position / style settings), but built to **install even when the gear interface
is missing** from this car's catalog — which is what broke the original.

## Why it installs past the "nothing found" error

Every car-data signal is marked **`"_optional"`**. At install the unit runs
`optional_missing()`: it reads this car's catalog, and for each optional interface
the catalog doesn't have, it **removes that interface from the manifest** (writing
`kit/missing-interfaces.json`, which the app logs on start) instead of aborting.
So a missing `de.bmw.infrastructure.VehicleInformation` (the engaged-gear source)
no longer kills the install.

The cluster drawing itself uses the **`cluster-render` Unix socket, not SOME/IP**,
so it never went through the failing generator step in the first place — the only
problem was the declared car-data signals, now all optional.

## How the gear still shows if the interface is gone

It reads rpm + speed (also optional, but these are standard and almost certainly
present) and **learns the gear from the rpm/speed ratio** (`kit/car.js` GearGuess)
when the direct gear signal is absent — shown in a dimmer colour until learned.
So you get a gear reading either way, after a little driving.

## Install (GitHub OTA)

Commit this folder to a repo root (e.g. `sergshamiryan/mgu-gear-resilient`),
then in-car **Store → Add source → branch → install "Gear on Cluster (resilient)"**.
Open the tile, set it in **D**, drive. Tune colour/position/size in **Settings**.
It installs alongside the original (`gear_cluster`) — different id (`gear_cluster2`).

## Caveats

- Depends on this car's installer honouring `meta.json optional` (the cmg
  installer's `optional_missing`, and the Store's manifest-trimming per its
  CHANGELOG). If this car's Store build is old enough to ignore both and the gear
  interface is absent, it could still fail — in that case **Catalog Dump** tells
  us exactly what's present and we rebuild this with the right interface names.
- Needs `cluster-render` present (it is — installed with Waze-on-cluster).
- No `oap-sdk`/`node_modules` in this repo.
