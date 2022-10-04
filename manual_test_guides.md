O3DE Manual Test Guides
--
The following test guides are recommended to be used for manual test verification of any changes made to their corresponding features, components and systems. Due to the complexity of the O3DE product and tools it has been determined that automated tests alone are not sufficient to fully mitigate the risk of escaped defects in submitted changes. The purpose of these manual test workflows are to serve as a supplement to the Automated Review test system and have been proven effective at uncovering defects not found through automated tests.

Each workflow test suite is owned and maintained by the SIG which owns their corresponding feature, component and system, the Testing SIG is therefore only responsible for maintaining this compiled list of test suites as reference documentation. Workflow suites tagged with Last Known Good (LKG) are recommended to be used as the bar for establishing a LKG tagged build, suites without the LKG tag can still be run but are not considered blocking for the LKG process.

SIG-Core
--
*Test Guidance Location:* TBD

SIG-Content
--
*Test Guidance Location:* https://github.com/o3de/sig-content/tree/main/testing-guidance

| Component | Test Suite | Location |
|---|---|---|
| Asset Pipeline | Asset Pipeline LKG | https://github.com/o3de/sig-content/blob/main/testing-guidance/workflow-tests/assetpipeline/assetpipeline-workflow-tests.md |
| Dynamic Vegetation | Dynamic Vegetation LKG | https://github.com/o3de/sig-content/blob/main/testing-guidance/workflow-tests/dynamic-vegetation/dynamic-vegetation-workflow-tests.md | 
| Editor | Editor LKG | https://github.com/o3de/sig-content/blob/main/testing-guidance/workflow-tests/editor/editor-workflow-tests.md |
| Landscape Canvas | Landscape Canvas LKG | https://github.com/o3de/sig-content/blob/main/testing-guidance/workflow-tests/landscape-canvas/landscape-canvas-workflow-tests.md |
| Prefabs | Prefabs LKG | https://github.com/o3de/sig-content/blob/main/testing-guidance/workflow-tests/prefabs/prefab-workflow-tests.md |
| Script Canvas | Script Canvas LKG | https://github.com/o3de/sig-content/blob/main/testing-guidance/workflow-tests/scripting/script-canvas-tests.md |
| Terrain | Terrain LKG | https://github.com/o3de/sig-content/blob/main/testing-guidance/workflow-tests/terrain/terrain-workflows.md |
| UI Editor | UI Editor LKG | https://github.com/o3de/sig-content/blob/main/testing-guidance/workflow-tests/ui-editor/ui-editor-workflow-tests.md |

SIG-Graphics-Audio
--
*Test Guidance Location:* TBD

SIG-Network
--
*Test Guidance Location:* TBD

SIG-Platform
--
*Test Guidance Location:* https://github.com/o3de/sig-platform/tree/main/testing-guidance
| Component | Test Suite | Location |
|---|---|---|
| Android | Android LKG | https://github.com/o3de/sig-platform/blob/main/testing-guidance/workflow-tests/android/android-lkg-workflows.md |

SIG-Simulation
--
*Test Guidance Location:* https://github.com/o3de/sig-simulation/tree/main/testing-guidance

| Component | Test Suite | Location |
|---|---|---|
| AI | AI LKG | https://github.com/o3de/sig-simulation/blob/main/testing-guidance/workflow-tests/ai/ai-lkg-workflows.md |
| Animation | Animation LKG | https://github.com/o3de/sig-simulation/blob/main/testing-guidance/workflow-tests/animation/animation-lkg-workflows.md |
| Animation | Root Motion Extraction workflows | https://github.com/o3de/sig-simulation/blob/main/testing-guidance/workflow-tests/animation/root-motion-extraction-workflow.md |
| Physics | Physics LKG | https://github.com/o3de/sig-simulation/blob/main/testing-guidance/workflow-tests/physics/physics-lkg-workflows.md |
| Physics | Blast Material Workflows | https://github.com/o3de/sig-simulation/blob/main/testing-guidance/workflow-tests/physics/physics-blast-materials-workflows.md |
| Physics | Ragdoll Authoring Workflows | https://github.com/o3de/sig-simulation/blob/main/testing-guidance/workflow-tests/physics/physics-ragdoll-workflows.md |
