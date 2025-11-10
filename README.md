# OutdoorPlus__UMCF_CrashingTerrain
This repository contains an OutdoorPlus-written urbanMicroclimateFoam case with terrain under the air internal mesh region. The simulation process fails at 'urbanMicroclimateFoam' with a sigSegv error.

<img width="1560" height="900" alt="terrain" src="https://github.com/user-attachments/assets/29ff1860-6dd1-49be-bb18-88a3bc5b96d5" />

The following commands were executed in a blueCFD terminal:

## Meshing:
```
blockMesh
surfaceFeatures

cp -r constant/polyMesh constant/polyMesh.bckp
cp system/snappyHexMeshDict.air system/snappyHexMeshDict
snappyHexMesh -overwrite

rm -rf constant/air/polyMesh
mv constant/polyMesh constant/air/
changeDictionary -region air

cp -r constant/polyMesh.bckp constant/polyMesh
cp system/snappyHexMeshDict.veg system/snappyHexMeshDict
snappyHexMesh -overwrite 2>&1 | tee -a 05_snappyHexMesh_veg.txt

rm -rf constant/vegetation/polyMesh
mv constant/polyMesh constant/vegetation/
createPatch -region vegetation -overwrite 2>&1 | tee -a 06_createPatch_air.txt
changeDictionary -region vegetation 2>&1 | tee -a 07_changeDictionary_air.txt

rm -rf constant/polyMesh.bckp
cp system/extrudeMeshDict.build system/extrudeMeshDict
extrudeMesh 2>&1 | tee -a 08_extrudeMesh_buildings.txt
mv constant/polyMesh constant/buildings
createPatch -region buildings  -overwrite 2>&1 | tee -a 09_createPatch_buildings.txt
changeDictionary -region buildings 2>&1 | tee -a 10_changeDictionary_buildings.txt
topoSet -region buildings 2>&1 | tee -a 11_topoSet_buildings.txt

extrudeMesh 2>&1 | tee -a -a 12_extrudeMesh_terrain_layer1.txt
cp system/terrain/extrudeMeshDict_terrain_layer2 system/extrudeMeshDict
extrudeMesh 2>&1 | tee -a 13_extrudeMesh_terrain_layer2.txt
mv constant/polyMesh/sets/addedCells.gz constant/polyMesh/sets/addedCells_layer2.gz
rm -rf constant/terrain/polyMesh
mv constant/polyMesh constant/terrain
createPatch -region terrain -overwrite 2>&1 | tee -a 14_createPatch_terrain.txt
changeDictionary -region terrain 2>&1 | tee -a 15_changeDictionary_terrain.txt

topoSet -region terrain 2>&1 | tee -a 16_topoSet_terrain.txt
topoSet -region air 2>&1 | tee -a 17_topoSet_air.txt
setFields -region air 2>&1 | tee -a 18_setFields_air.txt
```
## Running:
```
faceAgglomerate -region vegetation
calcLAI -region air
viewFactorsGen -region vegetation
solarRayTracingGen -region vegetation
urbanMicroclimateFoam
```
