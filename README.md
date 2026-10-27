# Simulation outputs: drying of three Scots pine cubes (CFD and multi-Fickian moisture model)

Output data of the paper

> R. D. Dsouza, E. Hajian, T. Harjupatana, J. A. J. Huber, L. Hansson, S. Fortino.
> *A CFD-informed multi-Fickian model of heat and moisture transport in wood validated by X-ray computed tomography.*
> Results in Engineering (under review, 2026).

The repository holds the simulation results for the three 50 mm Scots pine cubes S1, S2 and S3 that were dried together
in a kiln duct and monitored by in situ X-ray CT:

- **CFD**: steady RANS k-epsilon airflow around the three cubes in the 400 mm duct, at four air temperatures, and the
  heat and mass transfer coefficient of every cube face.
- **Moisture model**: the coupled heat and moisture transport inside each cube (bound water, water vapour, free water
  and temperature), from 0 to 25 h of drying.

All files open in ParaView (`.vtu`, `.pvd`) or in any spreadsheet or script (`.csv`). The measured CT data and the
solver code are not part of this repository; they are available from the corresponding author.

**Time window.** The cube results end at 25 h, the last CT scan used in the paper (the times of the later scans are
uncertain). The drying histories therefore stop at 25 h (the last row is linearly interpolated at exactly 25 h), and
the last field snapshot is at 24.48 h.

## Contents

```
cfd/
  h_per_face.csv               transfer coefficients per cube, face and air temperature (the values used by the moisture model)
  T295/ T320/ T345/ T375/      one steady RANS solution per air temperature [K]
    duct_volume.vtu            full 3-D field in the duct (tetrahedral mesh, nodal values)
    cube_faces.vtu             surface mesh of the cube faces with the wall quantities (local h, y+)
moisture/
  S1/ S2/ S3/
    S1_snapshots.pvd           ParaView time series of the snapshots below (open this one file)
    snapshots_vtu/             3-D fields at 0, 4.08, 8.16, 12.24, 16.32, 20.40 and 24.48 h
    snapshots_csv/             the same fields as tables: one row per mesh node with x, y, z and all fields
    snapshot_times.csv         exact time of each snapshot
    drying_history.csv         averages, central-slice MC, surface values and kiln conditions vs. time (0-25 h)
    profiles.csv               MC and temperature along the three centrelines at 0, 5, 10, 15, 20 and 25 h
LICENSE
```

## The three cubes

Each cube is 50 x 50 x 50 mm and rests on a support plate with its bottom face. The cubes stand in one row across the
duct with 30 mm gaps, S1, S2, S3 from x- to x+, and are dried under the same kiln schedule (air at 4 m/s).

| cube | exposed faces | sealed faces | basic density (kg/m3) | initial MC (%) |
|---|---|---|---|---|
| S1 | longitudinal (end grain): 1, 2 | 3, 4, 5, 6 | 465 | 112.6 |
| S2 | transverse: 3, 4, 6 | 1, 2, 5 | 428 | 89.7 |
| S3 | all except the bottom: 1, 2, 3, 4, 6 | 5 | 433 | 131.2 |

The initial state is uniform: MC from the first CT scan and 22 °C. MC is on the dry basis, (c_b + c_w) / rho_d x 100,
with rho_d the basic density (oven-dry mass over green volume).

### Faces and coordinates

The face numbering is the same in the CFD and moisture files.

| face | normal | position in the duct | wood direction |
|---|---|---|---|
| 1 | x- | side | L |
| 2 | x+ | side | L |
| 3 | y- | windward | R |
| 4 | y+ | leeward | R |
| 5 | z- | bottom, on the plate | T |
| 6 | z+ | top | T |

- **Moisture model**: each cube is its own box from (0, 0, 0) to (0.05, 0.05, 0.05) m, with x = longitudinal (L),
  y = radial (R), z = tangential (T).
- **CFD**: one frame for the duct. The duct axis is y, the air flows in +y from the inlet at y = 0 to the outlet at
  y = 0.45 m, and the duct is a circle of radius 0.2 m in the x-z plane. The cubes occupy y = 0.10-0.15 m and
  z = -0.1511 to -0.1011 m, with x = -0.105 to -0.055 (S1), -0.025 to 0.025 (S2) and 0.055 to 0.105 m (S3).
  A cube's moisture coordinates are its CFD coordinates minus its lower corner, so the two frames have parallel axes.

## CFD files

Steady incompressible RANS, standard k-epsilon with wall functions, FEniCSx (DOLFINx 0.9), P1-P1 elements, 3 mm mesh
near the cubes. Inlet 4 m/s with 5 % turbulence intensity, zero pressure at the outlet, no-slip on the duct wall, plate
and cubes. The run is repeated at T_gas = 295, 320, 345 and 375 K (air properties at that temperature).

To obtain h, the wood surfaces are held at T_gas - 2 K and the heat flux into them is computed. The temperature field
`T_air` is therefore this probe field, not the kiln temperature history.

**Fields in `duct_volume.vtu` and `cube_faces.vtu`** (nodal values)

| field | unit | meaning |
|---|---|---|
| velocity | m/s | velocity vector |
| speed | m/s | velocity magnitude |
| pressure | m2/s2 | kinematic pressure p/rho relative to the outlet |
| k_turb | m2/s2 | turbulent kinetic energy |
| eps_turb | m2/s3 | dissipation rate |
| nu_t | m2/s | turbulent viscosity |
| T_air | K | air temperature of the h probe (see above) |
| h_wf_local | W/(m2 K) | local heat transfer coefficient from the thermal wall function (the method used in the paper) |
| h_local | W/(m2 K) | local heat transfer coefficient from the resolved first-cell gradient (comparison method) |
| yplus_local | - | y+ of the first node at the walls |

The wall quantities are zero away from the walls. `cube_faces.vtu` also carries the cell data `cube` (1 = S1, 2 = S2,
3 = S3), `face` (1-6, table above) and `tag` = 100 x cube + face. Face 5 of each cube lies on the plate and is not a
wall of the airflow, so it does not appear.

**`h_per_face.csv`**: one row per cube, air temperature and face.

| column | meaning |
|---|---|
| h_W_m2K | face-averaged heat transfer coefficient from the wall function, W/(m2 K) |
| k_cv_m_s | mass transfer coefficient from h by the Chilton-Colburn analogy, m/s |
| in_moisture_model | how the face enters the moisture model: exposed, sealed, or (face 5) sealed for moisture with heat exchange by plate contact at h = 100 W/(m2 K) |

The moisture model interpolates h and k_cv linearly in the kiln air temperature between the four CFD temperatures.

## Moisture-model files

Multi-Fickian model with four coupled fields: bound water c_b, water vapour c_v, free water c_w (kg per m3 of wood) and
temperature. The phases exchange water through rate-limited sorption and evaporation terms. FEniCSx (DOLFINx 0.9), P1
tetrahedra, 5 mm mesh. The exposed faces exchange heat and moisture with the kiln air through the coefficients of
`cfd/h_per_face.csv`.

**Fields in the snapshots** (`snapshots_vtu/*.vtu`, `snapshots_csv/*.csv`; nodal values)

| field | unit | meaning |
|---|---|---|
| MC_percent | % | moisture content, dry basis |
| c_b | kg/m3 | bound water |
| c_v | kg/m3 | water vapour |
| c_w | kg/m3 | free (liquid) water |
| T_C | °C | temperature |

The CSV tables have one row per mesh node (`node`, `x_m`, `y_m`, `z_m`, then the fields). The node order and
coordinates are the same as in the VTU file, which also holds the tetrahedra.

**`drying_history.csv`**: one row per time step (time_s in s, time_hr in h).

| column | unit | meaning |
|---|---|---|
| MC_pct | % | volume-averaged MC |
| MC_slice_x_pct, MC_slice_y_pct, MC_slice_z_pct | % | mean MC in a central slab normal to x, y or z (thickness 12 % of the edge). **MC_slice_y_pct is the quantity compared with the CT in the paper** (the CT plane is normal to the duct axis) |
| M_bound_pct, M_free_pct | % | bound and free water parts of MC_pct |
| cv_avg_gm3 | g/m3 | mean water-vapour concentration |
| T_wood_K | K | mean wood temperature |
| T_gas_K, RH | K, - | kiln air temperature and relative humidity (the kiln schedule as applied) |
| M_eq_pct | % | equilibrium MC of the kiln air |
| M_fsp_pct | % | fibre saturation point at the mean wood temperature |
| S_w | - | mean free-water saturation of the lumen |
| f_lum_gas | - | mean gas fraction of the lumen |
| h_xm, h_xp, h_ym, h_yp, h_zm, h_zp | W/(m2 K) | h applied on faces 1-6 at this time (0 on sealed faces; the plate-contact h = 100 of face 5 is not listed here) |
| h_avg | W/(m2 K) | mean of the six columns above |
| M_surf_1 ... M_surf_6 | % | mean MC on face 1-6 |
| T_surf_1 ... T_surf_6 | K | mean temperature on face 1-6 |

**`profiles.csv`**: MC_pct (%) and T_K (K) along the three lines through the cube centre (`direction` L_x, R_y or
T_z; `position_mm` from the x-, y- or z- face).

The paper's agreement scores follow from `MC_slice_y_pct` against the CT points up to 25 h: RMSE 2.3, 2.5 and
2.9 % MC and R2 0.99, 0.92 and 0.99 for S1, S2 and S3.

## Opening the files

- **ParaView**: open `moisture/S2/S2_snapshots.pvd` for the time series of a cube, or any `.vtu` file directly. For the
  airflow, open `cfd/T320/duct_volume.vtu` (Slice, Stream Tracer) and `cfd/T320/cube_faces.vtu` (colour by
  `h_wf_local`, threshold on `cube` or `face`).
- **Python**: `meshio.read("cfd/T320/duct_volume.vtu")` gives the points, cells and point data; the CSV files read with
  `pandas.read_csv`.

## Licence and citation

The files are released under the MIT licence (see `LICENSE`). Please cite the paper above when you use them.

Contact: Royson D. Dsouza, VTT Technical Research Centre of Finland, royson.dsouza@vtt.fi

Funding: Research Council of Finland (formerly Academy of Finland), project WaterInWood, decision 349194.
