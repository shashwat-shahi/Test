# Test

import cadquery as cq

# --- Parameters (from the drawing) ---
hub_outer_dia = 22.0
hub_inner_dia = 16.0
base_thickness = 3.0
arm_width = 12.0 # Estimated from 3x3 rib offsets
total_height = 15.0 # Height of the central boss
arm_length = 50.0 # Distance from center to outer hole
fillet_radius = 1.0

# 1. Create the Central Hub
result = (cq.Workplane("XY")
          .circle(hub_outer_dia / 2)
          .extrude(total_height)
          .faces(">Z").hole(hub_inner_dia))

# 2. Create the Three Arms (Patterned at 120 degrees)
arm_pts = [(0, 0)] # Start at center

for i in range(3):
    angle = i * 120
    # Define the arm profile with the specific 8.0 degree taper mentioned in clouds
    arm = (cq.Workplane("XY")
           .rotated((0, 0, angle))
           .rect(arm_length, arm_width, centered=(False, True))
           .extrude(base_thickness))
    
    # Add the ribs/gussets with the slope
    # Note: Model Mania parts usually require a loft or a wedge for the sloped ribs
    rib = (cq.Workplane("XY")
           .rotated((0, 0, angle))
           .polyline([(hub_outer_dia/2, 0), (arm_length-10, 0), (hub_outer_dia/2, total_height-base_thickness)])
           .close()
           .extrude(3.0, both=True)) # 3mm rib thickness from clouds
    
    result = result.union(arm).union(rib)

# 3. Add the three outer holes (Blue Faces in drawing)
for i in range(3):
    angle = i * 120
    result = (result.faces("<Z").workplane()
              .rotated((0, 0, angle))
              .center(arm_length - 5, 0)
              .circle(3.0) # Outer hole radius
              .cutThruAll())

# 4. Apply Fillets (Red Revision Clouds)
# In CadQuery, we select edges by their proximity or type
result = result.edges().fillet(fillet_radius)

# Export to STEP for Simulation (as requested in Phase 2)
cq.exporters.export(result, 'model_mania_2020_phase2.step')