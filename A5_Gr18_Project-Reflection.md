## A5: Project Reflection

## Your learning experience for the concept you focused on.

When we started the course we used BIM as a Modeller. Now we are closer to Self Learner since we have been able to analyze the problem and teach ourselves to create a code that can solve it. This is only for one use case. 

![Ingen tilgængelig beskrivelse.](https://scontent-arn2-1.xx.fbcdn.net/v/t1.15752-9/369387379_752418903382285_7930300458431768532_n.png?_nc_cat=111&ccb=1-7&_nc_sid=8cd0a2&_nc_ohc=uCb5H0TrnwQAX-Wp2OG&_nc_ht=scontent-arn2-1.xx&oh=03_AdSXQV7GTK-Rnq8J-VugeSIhn7PlsbEcFzXaCbCE-Rw-XA&oe=658C0B08)

## Your process of developing the tutorial

We are able to define problems that can be solved with OpenBIM tools, but we have gotten very little experience in answering them, and we have only done it for one case. 

The free choice in use case is great for how the course is at the moment, as there are very different levels of experience with coding. For our part, we could possibly have learned more programming with a given use case, as we would have had more time to focus on the programming itself.

The structure of the course and the very quick introduction to new tools and software were quite confusing for us. The structure on Learn could have been more like the traditional structure, otherwise, the structure and format could have been introduced more thoroughly.

## Your received feedback on the tutorial

##### Feedback from Group 40

- The output is very long: Maybe create a summary instead or print what is problematic. Then the entire list of walls can be an option.

- The scope of the project is a little unclear with the difference between the ifc2x3 and 4 file if it is a general issue or just in this case. 

After the feedback, we changed a small thing in the script so you don't have to create a new link to the data model.

## Your future for Advanced use of OpenBIM

During the course, our knowledge about the possibilities of OpenBIM has been broadened, and the possible use cases are many.

Due to our lack of knowledge and experience with programming, the ideas we wished to execute have been limited. Furthermore, problems with the IFC-file have also limited us and challenged us during the course. 

Despite these challenges, we feel an improvement in our programming skills and our ability to present the intended use of our script.

Both members of this group feel that they are likely to use OpenBIM tools in the future because of the possibilities and efficiency of the tool. With more experience in programming, the tool will hopefully become easier to use as well. 

## Wrap up

From A1 to A5 we definitely have a better idea of how to work with open BIm Tools. The course has for our part been limited by our skills in Python, and that made it more challenging to think more creatively with the use case. 

## Final code after feedback

```python
# Necessary libraries are imported
import ifcopenshell
import ifcopenshell.api
from ifcopenshell.util import element

# Print the version of ifcopenshell library
print(ifcopenshell.version + "\n")

# Define paths to the 2 IFC files used
ifc2x3 = r'data\LLYN-ARK_2x3.ifc'
ifc4 = r'data\LLYN-ARK_4.ifc'

#Insert the path to our IFC2x3-file
model2x3 = ifcopenshell.open(ifc2x3)

#Insert the path to our IFC4-file
model4 = ifcopenshell.open(ifc4)

# Before creating a redesign of the IFC 4 file. The difference between the two files will be investigated.

# Function to count walls in an IFC files to see if some elements was not transferred correctly.
def count_walls(ifc_file):
    model = ifcopenshell.open(ifc_file)
    walls = model.by_type('IfcWall')

    return len(walls)

count2x3 = count_walls(ifc2x3)
count4 = count_walls(ifc4)
#Count walls in both IFC files to find the discrepancies


# Print the results
print(f"Number of walls in IFC2x3 file: {count2x3}")
print(f"Number of walls in IFC4 file: {count4}")


# Main function
def main():
     # Load walls from both IFC files
    walls2x3 = load_walls(model2x3)
    walls4 = load_walls(model4)

    # Match layers from IFC4 to IFC2x3 file
    new_walls4 = match_layers(walls2x3, walls4)

    # Define the path for the new IFC file
    new_file_path = r'outputs\modified_model4.ifc'

     # Print information about new walls in IFC4
    print("New walls in ifc4:")
    for key in new_walls4:
        print(f'\n{key} :')
        for layer in new_walls4[key]["layers"]:
            print(f'{" " : <4}{layer} : {new_walls4[key]["layers"][layer][0]: <35} | {str(new_walls4[key]["layers"][layer][1])+"mm" : <8}')

    # Update the IFC4 model with new walls and write to a new file
    update_model_and_write(model4, new_walls4, new_file_path)
    print("\nSuccessfully wrote walls to new file!")

# Match layers fra ifc 4 to ifc 2x3 file
def match_layers(source, target):
    for wall4 in target:
        for wall2x3 in source:
            # Skip if layers information is not available for the source wall
            if source[wall2x3]["layers"] is None:
                continue
            # If the walls in IFC4 and IFC2x3 match, iterate through layers
            if wall4 == wall2x3:
                for layer4 in target[wall4]["layers"]:
                    for layer2x3 in source[wall2x3]["layers"]:
                        # If layers match, update the IFC4 layer with IFC2x3 layer information
                        if(target[wall4]["layers"][layer4][0] == source[wall2x3]["layers"][layer2x3][0]):
                            target[wall4]["layers"][layer4] = source[wall2x3]["layers"][layer2x3]
    return target


#Load walls from ifc file
# Returns dictionary over wall_names with layers and product
def load_walls(model):
    walls = model.by_type('IfcWall')
    wall_dict = {}
    for wall in walls:
        wall_name = wall.get_info()["Name"]

        layers = {}
        material, thickness = get_ifc_materials(wall)
        # Check if material information is available for the wall
        if(len(material) == 0):
            layers = None
        else:
              # Iterate through material layers and add to the dictionary
            for i, layer in enumerate(zip(material, thickness)):
                layers["layer"+str(i)] = layer
        # Add wall information to the dictionary
        wall_dict[wall_name] = {"layers": layers, "product": wall}
    return wall_dict


# Load materials from ifc4 file
# Returns a list of material names and corresponding thicknesses for an ifc_product
def get_ifc_materials(ifc_product):
    material_name = []
    material_thickness = []

    if ifc_product:
        ifc_material = ifcopenshell.util.element.get_material(ifc_product)
        # Extract material information based on the type of IFC material
        if ifc_material:
            if ifc_material.is_a('IfcMaterial'):
                material_name.append(ifc_material.Name)
                material_thickness.append(0.0) 

            if ifc_material.is_a('IfcMaterialList'):
                # Iterate through materials in the material list
                for materials in ifc_material.Materials:
                    material_name.append(materials.Name)
                    material_thickness.append(0.0)

            if ifc_material.is_a('IfcMaterialConstituentSet'):
                # Iterate through material constituents in the set
                for material_constituents in ifc_material.MaterialConstituents:
                    material_name.append(material_constituents.Material.Name)
                    material_thickness.append(0.0)

            if ifc_material.is_a('IfcMaterialLayerSetUsage'):
                # Check if 'MaterialLayers' is available
                if hasattr(ifc_material.ForLayerSet, 'MaterialLayers'):
                    for material_layer in ifc_material.ForLayerSet.MaterialLayers:
                        material_name.append(material_layer.Material.Name)
                        material_thickness.append(material_layer.LayerThickness)
                    
            if ifc_material.is_a('IfcMaterialProfileSetUsage'):
                # Iterate through material profiles in the set
                for material_profile in ifc_material.ForProfileSet.MaterialProfiles:
                    material_name.append(material_profile.Material.Name)
                    material_thickness.append(0.0)

    return material_name, material_thickness

# Update new IFC file with walls
def update_model_and_write(model, walls, target_path):
    for wall_name in walls:
         # For each wall create a material set
        material_set = ifcopenshell.api.run("material.add_material_set", model, name=wall_name, set_type="IfcMaterialLayerSet")
        product = walls[wall_name]["product"]
        for layer_key in walls[wall_name]["layers"]:
            layer_name = walls[wall_name]["layers"][layer_key][0]
            #Add layer thickness
            layer_thickness = walls[wall_name]["layers"][layer_key][1]
            # Create a layer material
            layer_material = ifcopenshell.api.run("material.add_material", model, name=layer_name, category=layer_name)
            layer_to_add = ifcopenshell.api.run("material.add_layer", model, layer_set=material_set, material=layer_material)
            ifcopenshell.api.run("material.edit_layer", model, layer=layer_to_add, attributes={"LayerThickness": layer_thickness})
        

        # Update new IFC file with walls
        ifcopenshell.api.run("material.assign_material", model, product=product, material=material_set)
         # ... (code to update the IFC model with new walls)


    # Write the updated model to a new file
    model4.write(target_path)

# Run the main function when the script is executed
if __name__ == "__main__":
    main()

```


