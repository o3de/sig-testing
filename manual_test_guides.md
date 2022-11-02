O3DE Manual Test Guides
--
The following test guides are recommended to be used for manual test verification of any changes made to their corresponding features, components and systems to reduce the risk of defects being introduced. Due to the complexity of the O3DE product and tools it has been determined that automated tests alone are not sufficient to fully mitigate the risk of escaped defects in submitted changes. The purpose of these manual test workflows are to serve as a supplement to the Automated Review test system and have been proven effective at uncovering defects not found through automated tests. It is the Testing SIG's recommendation that each workflow test suite, including all LKG suites, are run during each Stabilization test period during the release process.

The Testing SIG does not own the test guides listed in this document and does not organize or schedule resources for their test execution. Each workflow test suite is owned, maintained and executed by the SIG which owns their corresponding feature, component and system. The Testing SIG's role includes consultation for content, format and review of test suites and is responsible for maintaining this compiled list of test suites as reference documentation. Workflow suites tagged with Last Known Good (LKG) are recommended to be used as the bar for establishing a LKG tagged build, suites without the LKG tag can still be run but are not considered blocking for the LKG process. 

SIG-Core
--
*Test Guidance Location:* TBD

SIG-Content
--
*Test Guidance Location:* https://github.com/o3de/sig-content/tree/main/testing-guidance

| Component | Test Suite | Location | Time Estimate |
|---|---|---|---|
| Asset Pipeline | Asset Pipeline LKG | https://github.com/o3de/sig-content/blob/main/testing-guidance/workflow-tests/assetpipeline/assetpipeline-workflow-tests.md | 5 Hours |
| Dynamic Vegetation | Dynamic Vegetation LKG | https://github.com/o3de/sig-content/blob/main/testing-guidance/workflow-tests/dynamic-vegetation/dynamic-vegetation-workflow-tests.md | 30 Minutes |
| Editor | Editor LKG | https://github.com/o3de/sig-content/blob/main/testing-guidance/workflow-tests/editor/editor-workflow-tests.md | 1 Hour |
| Landscape Canvas | Landscape Canvas LKG | https://github.com/o3de/sig-content/blob/main/testing-guidance/workflow-tests/landscape-canvas/landscape-canvas-workflow-tests.md | 40 Minutes |
| Prefabs | Prefabs LKG | https://github.com/o3de/sig-content/blob/main/testing-guidance/workflow-tests/prefabs/prefab-workflow-tests.md | 3 Hours 15 Minutes |
| Script Canvas | Script Canvas LKG | https://github.com/o3de/sig-content/blob/main/testing-guidance/workflow-tests/scripting/script-canvas-tests.md | 3 Hours 45 Minutes |
| Terrain | Terrain LKG | https://github.com/o3de/sig-content/blob/main/testing-guidance/workflow-tests/terrain/terrain-workflows.md | 1 Hour 40 Minutes |
| UI Editor | UI Editor LKG | https://github.com/o3de/sig-content/blob/main/testing-guidance/workflow-tests/ui-editor/ui-editor-workflow-tests.md | 50 Minutes |
| Project Manager | Remote Projects  LKG | https://github.com/o3de/sig-content/blob/main/testing-guidance/workflow-tests/remote-projects/remote-projects-workflow-tests.md |1 Hour |
| Project Manager | Remote Templates LKG | https://github.com/o3de/sig-content/blob/main/testing-guidance/workflow-tests/remote-templates/remote-templates-workflow-tests.md | 1 Hour |
| Project Manager | Create a Gem LKG | https://github.com/o3de/sig-content/blob/main/testing-guidance/workflow-tests/create-a-gem/create-a-gem-workflow-tests.md | 1 Hour |

SIG-Graphics-Audio
--
*Test Guidance Location:* https://github.com/o3de/sig-graphics-audio/tree/main/testing-guidance
| Component | Test Suite | Location | Time Estimate |
|---|---|---|---|
| Materials | Editor Mesh Materials Worfklows | https://github.com/o3de/sig-graphics-audio/blob/main/testing-guidance/workflow-tests/materials/editor_mesh_materials.md | 20 Minutes per platform |
| ASV | Tools Workflows LKG | https://github.com/o3de/sig-graphics-audio/blob/main/testing-guidance/workflow-tests/AtomSampleViewer/ASV_Debug_and_Profiling_Tools_Worflow_Test.md | 30 minutes |
| ASV | Feature Workflow LKG | https://github.com/o3de/sig-graphics-audio/blob/main/testing-guidance/workflow-tests/AtomSampleViewer/ASV_Feature_Worflow_Test.md | 10 Minutes |
| ASV | RHI Workflow LKG | https://github.com/o3de/sig-graphics-audio/blob/main/testing-guidance/workflow-tests/AtomSampleViewer/ASV_RHI_Worflow_Test.md | 5 Minutes per sample |
| ASV | RPI Workflow LKG | https://github.com/o3de/sig-graphics-audio/blob/main/testing-guidance/workflow-tests/AtomSampleViewer/ASV_RPI_Worflow_Test.md | 5 Minutes per sample |
| Decals and Cloth | Component Workflow LKG | https://github.com/o3de/sig-graphics-audio/blob/main/testing-guidance/workflow-tests/Decals_and_Cloth/Editor_Decals_and_Cloth_fog_workflow_test.md | 40 Minutes per platform |
| SMAA | SMAA Workflow LKG | https://github.com/o3de/sig-graphics-audio/blob/main/testing-guidance/workflow-tests/MSAA_and_Deferred_Fog/Editor_SMAA_and_Deferred_fog_workflow_test.md | 30 Minutes per platform |


SIG-Network
--
*Test Guidance Location:* https://github.com/o3de/sig-network/tree/main/testing-guidance/workflow-tests
| Component | Test Suite | Location | Time Estimate |
|---|---|---|---|
| Multiplayer | Basic Workflow Tests LKG | https://github.com/o3de/sig-network/blob/main/testing-guidance/workflow-tests/multiplayer/basic-multiplayer-workflow-tests.md | 2 Hours |
| Multiplayer | Multi-Machine Workflow Tests | https://github.com/o3de/sig-network/blob/main/testing-guidance/workflow-tests/multiplayer/multi-machine-multiplayer-workflow-tests.md | 2 Hours |
| Gems | Remote Tools Connection Gem Workflows | https://github.com/o3de/sig-network/blob/main/testing-guidance/workflow-tests/remote-tools-connection-gem/remote-tools-connection-gem-workflow-tests.md | 1 Hour |

SIG-Platform
--
*Test Guidance Location:* https://github.com/o3de/sig-platform/tree/main/testing-guidance
| Component | Test Suite | Location | Time Estimate |
|---|---|---|---|
| Android | Android LKG | https://github.com/o3de/sig-platform/blob/main/testing-guidance/workflow-tests/android/android-lkg-workflows.md | 1 Hour 30 Minutes |

SIG-Simulation
--
*Test Guidance Location:* https://github.com/o3de/sig-simulation/tree/main/testing-guidance

| Component | Test Suite | Location | Time Estimate |
|---|---|---|---|
| AI | AI LKG | https://github.com/o3de/sig-simulation/blob/main/testing-guidance/workflow-tests/ai/ai-lkg-workflows.md | 2 Hours per Platform |
| Animation | Animation LKG | https://github.com/o3de/sig-simulation/blob/main/testing-guidance/workflow-tests/animation/animation-lkg-workflows.md | 2 Hours |
| Animation | Root Motion Extraction workflows | https://github.com/o3de/sig-simulation/blob/main/testing-guidance/workflow-tests/animation/root-motion-extraction-workflow.md | 1 Hour per platform |
| Physics | Physics LKG | https://github.com/o3de/sig-simulation/blob/main/testing-guidance/workflow-tests/physics/physics-lkg-workflows.md | 5 Hours 45 minutes |
| Physics | Blast Material Workflows | https://github.com/o3de/sig-simulation/blob/main/testing-guidance/workflow-tests/physics/physics-blast-materials-workflows.md | 30 minutes |
| Physics | Ragdoll Authoring Workflows | https://github.com/o3de/sig-simulation/blob/main/testing-guidance/workflow-tests/physics/physics-ragdoll-workflows.md | 90 Minutes per platform |
