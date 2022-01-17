# objectmsh
Object oriented Gmsh modeling.

## Project description

This tool provides some utilities for the [gmsh python API](https://pypi.org/project/gmsh/) that are especially helpful when you're working on complex geometries using the Open CASCADE kernel.

## Installation

The latest release can be installed using pip:

```
pip install objectgmsh
```

## Basic principle

objectgmsh bases on a *Model* class implementing an interface to the main Gmsh functionality, and a *Shape* class used to access information about the geometry, e.g., the IDs of the interface between two bodies. Furthermore, several classes for simplified control of mesh sizes are provided.

objectgmsh is just a wrapper for the gmsh python API, and all gmsh commands are still working "as usual". However, in complex geometries, it highly reduces the modelling effort.

## Example

```Python
import gmsh
from objectgmsh import Model, Shape, MeshControlConstant, MeshControlExponential, cut


# dimensions
cruc_r = 0.06  # crucible radius
cruc_h = 0.03  # crucible height
cruc_hi = 0.015  # crucible height inside
melt_r = 0.025  # melt radius
melt_h = 0.01  # melt height
crys_r = 0.005  # crystal radius
crys_h = 0.1  # crystal height


occ = gmsh.model.occ
model = Model()

# main bodies
crystal = Shape(model, 2, "crystal", [occ.addRectangle(0, 0, 0, crys_r, crys_h)])
melt = Shape(model, 2, "melt", [occ.addRectangle(0, -melt_h, 0, melt_r, melt_h)])

crucible = occ.addRectangle(0, -melt_h - (cruc_h - cruc_hi), 0, cruc_r, cruc_h)
crucible_hole = occ.addRectangle(0, -melt_h, 0, melt_r, cruc_hi)
cut([(2, crucible)], [(2, crucible_hole)])
crucible = Shape(model, 2, "crucible", [crucible])

# create connection between the shapes
crystal.set_interface(melt)
melt.set_interface(crucible)

# detect boundaries
bnd_crystal_out = Shape(
    model, 1, "bnd_crystal_out", [crystal.top_boundary, crystal.right_boundary]
)
bnd_melt = Shape(
    model, 1, "bnd_melt_surf", melt.get_boundaries_in_box([crys_r, melt_r], [0, 0])
)
surfs = [
    crucible.get_boundaries_in_box(
        [melt_r, melt_r], [0, cruc_hi - melt_h], one_only=True
    ),
    crucible.top_boundary,
    crucible.right_boundary,
]
bnd_crucible_outside = Shape(model, 1, "bnd_crucible_outside", surfs)
bnd_crucible_bottom = Shape(model, 1, "bnd_crucible_bottom", [crucible.bottom_boundary])

if_crystal_melt = Shape(model, 1, "if_crystal_melt", crystal.get_interface(melt))

# add physical groups
model.make_physical()

# set mesh constraints
model.deactivate_characteristic_length()
MeshControlConstant(model, 0.005, [crucible, melt])
MeshControlConstant(model, 0.0025, [crystal])
MeshControlExponential(
    model, if_crystal_melt, 0.001, exp=1.7, shapes=[crystal, melt, crucible]
)

# create mesh, show, export
model.generate_mesh()
model.show()
model.write_msh("crystal-growth-2D.msh")
```

This is still under construction. Additional examples, e.g., with more complex geometries or 3D modelling are planned.

## Documentation

Documentation is still under construction.

## Support

In case of questions just open an issue or contact Arved Enders-Seidlitz.

## Acknowledgements

[This project](https://www.researchgate.net/project/NEMOCRYS-Next-Generation-Multiphysical-Models-for-Crystal-Growth-Processes) has received funding from the European Research Council (ERC) under the European Union's Horizon 2020 research and innovation programme (grant agreement No 851768).

<img src="https://raw.githubusercontent.com/nemocrys/objectgmsh/master/EU-ERC.png">

## Contribution

Any help to improve this package is very welcome!
