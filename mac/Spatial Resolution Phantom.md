##########################################################
# Spatial Resolution Phantom - 6 Point Sources
# Paper 7: Section 2.3 - Spatial Resolution
# NEMA NU-2 2018 - Point Source Method
##########################################################
# Six spherical point sources at different positions:
#   Axial positions: z=0 (center) and z=1/8 AFOV (24.25 cm)
#   Radial positions: r=1, 10, 20 cm
# Total: 6 point sources (2 axial × 3 radial)
##########################################################

# ============================================================
# POINT SOURCE 1: Radial=1cm, Axial=0cm (Center of FOV)
# ============================================================

/gate/world/daughters/name point_source_1
/gate/world/daughters/insert sphere

# Material: Water (carrier for F-18)
/gate/point_source_1/setMaterial Water

# Geometry: Small sphere (0.5 mm diameter)
# Diameter: 0.5 mm → Radius: 0.25 mm = 0.025 cm
/gate/point_source_1/geometry/setRmax 0.025 cm

# Position: 1 cm radial offset (X-direction), center axial (Z=0)
# X=1.0 cm, Y=0, Z=0
/gate/point_source_1/placement/setTranslation 1.0 0.0 0.0 cm

# Visualization
/gate/point_source_1/vis/setColor red
/gate/point_source_1/vis/setVisible 1


# ============================================================
# POINT SOURCE 2: Radial=10cm, Axial=0cm (Center of FOV)
# ============================================================

/gate/world/daughters/name point_source_2
/gate/world/daughters/insert sphere

/gate/point_source_2/setMaterial Water
/gate/point_source_2/geometry/setRmax 0.025 cm

# Position: 10 cm radial offset, center axial
# X=10.0 cm, Y=0, Z=0
/gate/point_source_2/placement/setTranslation 10.0 0.0 0.0 cm

/gate/point_source_2/vis/setColor red
/gate/point_source_2/vis/setVisible 1


# ============================================================
# POINT SOURCE 3: Radial=20cm, Axial=0cm (Center of FOV)
# ============================================================

/gate/world/daughters/name point_source_3
/gate/world/daughters/insert sphere

/gate/point_source_3/setMaterial Water
/gate/point_source_3/geometry/setRmax 0.025 cm

# Position: 20 cm radial offset, center axial
# X=20.0 cm, Y=0, Z=0
/gate/point_source_3/placement/setTranslation 20.0 0.0 0.0 cm

/gate/point_source_3/vis/setColor red
/gate/point_source_3/vis/setVisible 1


# ============================================================
# POINT SOURCE 4: Radial=1cm, Axial=1/8 AFOV (24.25 cm)
# ============================================================

/gate/world/daughters/name point_source_4
/gate/world/daughters/insert sphere

/gate/point_source_4/setMaterial Water
/gate/point_source_4/geometry/setRmax 0.025 cm

# Position: 1 cm radial, 1/8 AFOV axial
# AFOV = 194 cm → 1/8 AFOV = 24.25 cm
# X=1.0 cm, Y=0, Z=24.25 cm
/gate/point_source_4/placement/setTranslation 1.0 0.0 24.25 cm

/gate/point_source_4/vis/setColor yellow
/gate/point_source_4/vis/setVisible 1


# ============================================================
# POINT SOURCE 5: Radial=10cm, Axial=1/8 AFOV (24.25 cm)
# ============================================================

/gate/world/daughters/name point_source_5
/gate/world/daughters/insert sphere

/gate/point_source_5/setMaterial Water
/gate/point_source_5/geometry/setRmax 0.025 cm

# Position: 10 cm radial, 1/8 AFOV axial
# X=10.0 cm, Y=0, Z=24.25 cm
/gate/point_source_5/placement/setTranslation 10.0 0.0 24.25 cm

/gate/point_source_5/vis/setColor yellow
/gate/point_source_5/vis/setVisible 1


# ============================================================
# POINT SOURCE 6: Radial=20cm, Axial=1/8 AFOV (24.25 cm)
# ============================================================

/gate/world/daughters/name point_source_6
/gate/world/daughters/insert sphere

/gate/point_source_6/setMaterial Water
/gate/point_source_6/geometry/setRmax 0.025 cm

# Position: 20 cm radial, 1/8 AFOV axial
# X=20.0 cm, Y=0, Z=24.25 cm
/gate/point_source_6/placement/setTranslation 20.0 0.0 24.25 cm

/gate/point_source_6/vis/setColor yellow
/gate/point_source_6/vis/setVisible 1


##########################################################
# PHANTOM SUMMARY
##########################################################
# Point Source Specifications:
#   - Diameter: 0.5 mm (radius 0.025 cm)
#   - Material: Water + F-18
#   - Volume per source: (4/3)π × (0.025)³ = 6.545×10⁻⁵ cm³
#
# Position Matrix:
#   ┌─────────┬──────────┬──────────┬──────────┐
#   │ Source  │ Radial   │ Axial    │ Position │
#   ├─────────┼──────────┼──────────┼──────────┤
#   │ 1 (red) │  1 cm    │   0 cm   │ (1,0,0)  │
#   │ 2 (red) │ 10 cm    │   0 cm   │ (10,0,0) │
#   │ 3 (red) │ 20 cm    │   0 cm   │ (20,0,0) │
#   │ 4 (yel) │  1 cm    │ 24.25 cm │ (1,0,24) │
#   │ 5 (yel) │ 10 cm    │ 24.25 cm │ (10,0,24)│
#   │ 6 (yel) │ 20 cm    │ 24.25 cm │ (20,0,24)│
#   └─────────┴──────────┴──────────┴──────────┘
#
# Per Paper (Section 2.3.1):
#   - Activity: 300 kBq per source
#   - Acquisition: 60 seconds
#   - Expected events: ~70 million total
#   - Unit difference: Restricted to 1 (24 cm AFOV)
##########################################################
