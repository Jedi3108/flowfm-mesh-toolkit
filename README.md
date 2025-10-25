# FlowFM Mesh Export & Analysis Toolkit

### Overview
This repository provides a collection of Python utilities (C++ has been made private) and Jupyter notebooks to analyze, extract, and convert Delft3D-FM / D-Flow FM (`FlowFM_map.nc`) NetCDF files into formats suitable for post-processing and visualization (e.g., Tecplot).

The notebook includes three main modules:
1. **2D Mesh Exporter** – Extracts and writes 2D triangular surface meshes to Tecplot.
2. **3D Combined Layer Exporter** – Builds full 3D layered meshes (sigma + Z) and writes them to a single-zone Tecplot file.
3. **NetCDF Metadata Extractor** – Scans and lists all dimensions, variables, attributes, and relationships in a `.nc` file for inspection.

---

## 1. 2D Mesh Exporter
**Goal:** Convert a 2D unstructured FlowFM mesh to Tecplot format for surface-level visualization.

**Key operations**
- Reads `mesh2d_node_x`, `mesh2d_node_y`, and `mesh2d_face_nodes`.
- Computes face connectivity (1-based for Tecplot).
- Writes surface mesh as `F=FEPOINT, ET=TRIANGLE`.

**Output example:**  
`Lake_Mesh_2D.tec`

---

## 2. 3D Combined Layer Exporter
**Goal:** Generate and visualize sigma + Z layer hybrid meshes for a FlowFM domain, with full time-dependent support.

**Core workflow**
1. Reads key NetCDF variables:
   - `mesh2d_node_x`, `mesh2d_node_y`, `mesh2d_node_z`
   - `mesh2d_face_nodes`
   - `mesh2d_s1` (surface elevation)
   - `mesh2d_interface_z`
   - `time`
2. Computes nodal surface elevations by averaging connected faces.
3. Builds sigma layers from the free surface down to a defined interface depth.
4. Appends Z layers extracted from `mesh2d_interface_z`.
5. Constructs full 3D brick connectivity (degenerate hexahedra from triangular faces).
6. Writes one Tecplot zone per time step, using:
   - `SOLUTIONTIME` = corresponding `time` value
   - `STRANDID = 1` (shared strand for time animation)

**Output example:**  
`Lake_Mesh_3D_SingleZone.tec`

**Example parameters**
```python
num_sigma_layers = 10
num_z_layers = 50
interface_depth = -5.03
layers_to_remove = 0
time_indices = [0, 1]
```

### Notes

- Assumes mesh2d_face_nodes is 1-based indexed.
- The file is designed for D-Flow FM / FlowFM map outputs.
- Sigma surfaces are clipped at bathymetry (max(z_layer, node_z)).
- Z-surfaces are extracted as num_z_layers + 1 levels starting at layers_to_remove.

### 3. NetCDF Variable Inspector

Goal: Automatically extract metadata for every variable in a FlowFM NetCDF file.

- Extracted details
- Dimensions, shapes, and data types
- Units, standard names, and attributes
- Min/max statistics (optional)
- Summary dictionary for quick reference

### Example output (abbreviated):
```python
Variable: mesh2d_s1
  Dimensions: (time, nMesh2d_face)
  Units: m
  Description: Water surface elevation
  Range: [-1.53, 0.42]
```

### Author & Notes

Developed by Akshat Shukla
Master’s Program, Universität Bayreuth
(Project: Numerical modeling and visualization of hydrodynamic meshes)

This toolkit simplifies the transition from FlowFM NetCDF data to visual 3D representations for diagnostic, presentation, and simulation validation purposes.
