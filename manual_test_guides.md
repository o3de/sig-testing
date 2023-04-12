O3DE Manual Test Guides
--

The following test guides are recommended to be used for manual test verification of any changes made to their corresponding features, components and systems to reduce the risk of defects being introduced. Due to the complexity of the O3DE product and tools it has been determined that automated tests alone are not sufficient to fully mitigate the risk of escaped defects in submitted changes. The purpose of these manual test workflows are to serve as a supplement to the Automated Review test system and have been proven effective at uncovering defects not found through automated tests. It is the Testing SIG's recommendation that each workflow test suite, including all LKG suites, are run during each Stabilization test period during the release process.

The Testing SIG does not own the test guides listed in this document and does not organize or schedule resources for their test execution. Each workflow test suite is owned, maintained and executed by the SIG which owns their corresponding feature, component and system. The Testing SIG's role includes consultation for content, format and review of test suites and is responsible for maintaining this compiled list of test suites as reference documentation. Workflow suites tagged with Last Known Good (LKG) are recommended to be used as the bar for establishing a LKG tagged build, suites without the LKG tag can still be run but are not considered blocking for the LKG process. 

SIG-Core
--

*Test Guidance Location:* TBD

No Workflows for SIG-Core at this time.

---

SIG-Content
--

*Test Guidance Location:* https://github.com/o3de/sig-content/tree/main/testing-guidance

| Component                  | Test Suite                                       | Location                                                                                                                                               | Time Estimate                  |
|----------------------------|--------------------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------|--------------------------------|
| Asset Pipeline             | Asset Bundler Workflow Tests                     | https://github.com/o3de/sig-content/blob/main/testing-guidance/workflow-tests/assetpipeline/assetpipeline-assetbundling-workflow-tests.md              | 2 Hours                        |
| Asset Pipeline             | Asset Processor Workflow Tests                   | https://github.com/o3de/sig-content/blob/main/testing-guidance/workflow-tests/assetpipeline/assetpipeline-assetprocessor-workflow-tests.md             | 8 Hours 30 Minutes             |
| Asset Pipeline             | Scene Pipeline Workflow Tests                    | https://github.com/o3de/sig-content/blob/main/testing-guidance/workflow-tests/assetpipeline/assetpipeline-scenepipeline-workflow-tests.md              | 4 Hours 15 Minutes             |
| Asset Pipeline             | Asset Builder Framework Workflow Tests (**WIP**) | https://github.com/o3de/sig-content/blob/main/testing-guidance/workflow-tests/assetpipeline/assetpipeline-assetbuilderframework-workflow-tests.md      | TBD                            |
| Dynamic Vegetation         | Dynamic Vegetation LKG                           | https://github.com/o3de/sig-content/blob/main/testing-guidance/workflow-tests/dynamic-vegetation/dynamic-vegetation-workflow-tests.md                  | 30 Minutes                     |
| Editor                     | Editor LKG                                       | https://github.com/o3de/sig-content/blob/main/testing-guidance/workflow-tests/editor/editor-workflow-tests.md                                          | 1 Hour                         |
| Landscape Canvas           | Landscape Canvas LKG                             | https://github.com/o3de/sig-content/blob/main/testing-guidance/workflow-tests/landscape-canvas/landscape-canvas-workflow-tests.md                      | 40 Minutes                     |
| Prefabs                    | Prefabs LKG                                      | https://github.com/o3de/sig-content/blob/main/testing-guidance/workflow-tests/prefabs/prefab-workflow-tests.md                                         | 3 Hours 15 Minutes             |
| Scripting - Script Canvas  | Script Canvas Workflow Tests                     | https://github.com/o3de/sig-content/blob/main/testing-guidance/workflow-tests/scripting/script-canvas-tests.md                                         | 5 Hours 15 Minutes             |
| Scripting - Lua Scripting  | Lua Scripting Workflow Tests                     | https://github.com/o3de/sig-content/blob/main/testing-guidance/workflow-tests/scripting/lua-scripting-tests.md                                         | 1 Hours 15 Minutes             |
| Scripting - Python Scripts | Python Scripts Workflow Tests                    | https://github.com/o3de/sig-content/blob/main/testing-guidance/workflow-tests/scripting/script-python-tests.md                                         | 15 Minutes                     |
| Terrain                    | Terrain LKG                                      | https://github.com/o3de/sig-content/blob/main/testing-guidance/workflow-tests/terrain/terrain-workflows.md                                             | 1 Hour 40 Minutes              |
| Track View                 | Track View Workflow Tests                        | https://github.com/o3de/sig-content/blob/main/testing-guidance/workflow-tests/track-view/track-view-tests.md                                           | 2 Hours 15 Minutes             |
| UI Editor                  | UI Editor LKG                                    | https://github.com/o3de/sig-content/blob/main/testing-guidance/workflow-tests/ui-editor/ui-editor-workflow-tests.md                                    | 50 Minutes                     |
| O3DE Project Manager       | Engine Versioning Workflow Tests                 | https://github.com/o3de/sig-content/blob/main/testing-guidance/workflow-tests/o3de-project-manager/engine-versioning/engine-versioning.md              | 1 Hour per platform            |
| O3DE Project Manager       | Gem Workflow Tests                               | https://github.com/o3de/sig-content/blob/main/testing-guidance/workflow-tests/o3de-project-manager/gems/gem-workflow-tests.md                          | 1 Hour 15 Minutes per platform |
| O3DE Project Manager       | Project Export Workflow Tests                    | https://github.com/o3de/sig-content/blob/main/testing-guidance/workflow-tests/o3de-project-manager/project-export/project-export-workflow-tests.md     | TBD                            |
| O3DE Project Manager       | Remote Projects LKG                              | https://github.com/o3de/sig-content/blob/main/testing-guidance/workflow-tests/o3de-project-manager/remote-projects/remote-projects-workflow-tests.md   | 1 Hour                         |
| O3DE Project Manager       | Remote Templates LKG                             | https://github.com/o3de/sig-content/blob/main/testing-guidance/workflow-tests/o3de-project-manager/remote-templates/remote-templates-workflow-tests.md | 1 Hour                         |
___

SIG-Graphics-Audio
--

*Test Guidance Location:* https://github.com/o3de/sig-graphics-audio/tree/main/testing-guidance

| Component        | Test Suite                                                                 | Location                                                                                                                                                 | Time Estimate           |
|------------------|----------------------------------------------------------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------|-------------------------|
| Materials        | Editor Mesh Materials Workflows                                            | https://github.com/o3de/sig-graphics-audio/blob/main/testing-guidance/workflow-tests/materials/editor_mesh_materials.md                                  | 20 Minutes per platform |
| ASV              | Tools Workflows LKG                                                        | https://github.com/o3de/sig-graphics-audio/blob/main/testing-guidance/workflow-tests/AtomSampleViewer/ASV_Debug_and_Profiling_Tools_Worflow_Test.md      | 30 minutes              |
| ASV              | Feature Workflow LKG                                                       | https://github.com/o3de/sig-graphics-audio/blob/main/testing-guidance/workflow-tests/AtomSampleViewer/ASV_Feature_Worflow_Test.md                        | 10 Minutes              |
| ASV              | RHI Workflow LKG                                                           | https://github.com/o3de/sig-graphics-audio/blob/main/testing-guidance/workflow-tests/AtomSampleViewer/ASV_RHI_Worflow_Test.md                            | 5 Minutes per sample    |
| ASV              | RPI Workflow LKG                                                           | https://github.com/o3de/sig-graphics-audio/blob/main/testing-guidance/workflow-tests/AtomSampleViewer/ASV_RPI_Worflow_Test.md                            | 5 Minutes per sample    |
| Decals and Cloth | Component Workflow LKG                                                     | https://github.com/o3de/sig-graphics-audio/blob/main/testing-guidance/workflow-tests/Decals_and_Cloth/Editor_Decals_and_Cloth_fog_workflow_test.md       | 40 Minutes per platform |
| Lighting & Skies | Editor component Eye Adaptation Workflow Tests                             | https://github.com/o3de/sig-graphics-audio/blob/main/testing-guidance/workflow-tests/Lighting_and_Skies/editor_eye_adaptation.md                         | 15 Minutes per platform |
| Lighting & Skies | Editor component Punctual/Area Lights Workflow Tests                       | https://github.com/o3de/sig-graphics-audio/blob/main/testing-guidance/workflow-tests/Lighting_and_Skies/editor_punctual_area_lights.md                   | 15 Minutes per platform |
| Lighting & Skies | Editor component Reflection Probe Workflow Tests                           | https://github.com/o3de/sig-graphics-audio/blob/main/testing-guidance/workflow-tests/Lighting_and_Skies/editor_reflection_probe.md                       | 15 Minutes per platform |
| Lighting & Skies | Editor component Shadows (Spotlight / Cascaded Shadow Maps) Workflow Tests | https://github.com/o3de/sig-graphics-audio/blob/main/testing-guidance/workflow-tests/Lighting_and_Skies/editor_shadows.md                                | 15 Minutes per platform |
| Lighting & Skies | Editor component Skies Workflow Tests                                      | https://github.com/o3de/sig-graphics-audio/blob/main/testing-guidance/workflow-tests/Lighting_and_Skies/editor_skies.md                                  | 15 Minutes per platform |
| Lighting & Skies | Editor component Sky Atmosphere/Stars Workflow Tests                       | https://github.com/o3de/sig-graphics-audio/blob/main/testing-guidance/workflow-tests/Lighting_and_Skies/editor_sky_atmosphere_stars.md                   | 15 Minutes per platform |
| MSAA             | MSAA Workflow LKG                                                          | https://github.com/o3de/sig-graphics-audio/blob/main/testing-guidance/workflow-tests/MSAA_and_Deferred_Fog/Editor_SMAA_and_Deferred_fog_workflow_test.md | 30 Minutes per platform |
| PostFX           | Editor component PostFX Layer and Modifiers Workflow Tests                 | https://github.com/o3de/sig-graphics-audio/blob/main/testing-guidance/workflow-tests/PostFX/PostFx-Layer.md                                              | 45 Minutes per platform |
___

SIG-Network
--

*Test Guidance Location:* https://github.com/o3de/sig-network/tree/main/testing-guidance/workflow-tests

| Component     | Test Suite                            | Location                                                                                                                                                | Time Estimate                   |
|---------------|---------------------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------|---------------------------------|
| Multiplayer   | Multiplayer Testing Workflow Tests    | https://github.com/o3de/sig-network/blob/main/testing-guidance/workflow-tests/multiplayer/multiplayer-workflow-tests.md                                 | 6 Hours 30 Minutes per platform |
| Multiplayer   | Multi-Machine Workflow Tests          | https://github.com/o3de/sig-network/blob/main/testing-guidance/workflow-tests/multiplayer/multi-machine-multiplayer-workflow-tests.md                   | 2 Hours                         |
| Gems          | Remote Tools Connection Gem Workflows | https://github.com/o3de/sig-network/blob/main/testing-guidance/workflow-tests/remote-tools-connection-gem/remote-tools-connection-gem-workflow-tests.md | 1 Hour                          |
___

SIG-Platform
--

*Test Guidance Location:* https://github.com/o3de/sig-platform/tree/main/testing-guidance

| Component | Test Suite    | Location                                                                                                        | Time Estimate     |
|-----------|---------------|-----------------------------------------------------------------------------------------------------------------|-------------------|
| Android   | Android LKG   | https://github.com/o3de/sig-platform/blob/main/testing-guidance/workflow-tests/android/android-lkg-workflows.md | 1 Hour 30 Minutes |
---

SIG-Simulation
--

*Test Guidance Location:* https://github.com/o3de/sig-simulation/tree/main/testing-guidance

| Component | Test Suite                       | Location                                                                                                                      | Time Estimate                   |
|-----------|----------------------------------|-------------------------------------------------------------------------------------------------------------------------------|---------------------------------|
| AI        | AI LKG                           | https://github.com/o3de/sig-simulation/blob/main/testing-guidance/workflow-tests/ai/ai-lkg-workflows.md                       | 2 Hours 45 Minutes per platform |
| Animation | Animation LKG                    | https://github.com/o3de/sig-simulation/blob/main/testing-guidance/workflow-tests/animation/animation-lkg-workflows.md         | 2 Hours per platform            |
| Animation | Animation Editor Workflow Tests  | https://github.com/o3de/sig-simulation/blob/main/testing-guidance/workflow-tests/animation/animation-editor-workflows.md      | 6 Hours per platform            |
| Animation | Deforming Objects Workflow Tests | https://github.com/o3de/sig-simulation/blob/main/testing-guidance/workflow-tests/animation/deforming-objects-workflows.md     | 1 Hour per platform             |
| Animation | FootIK Workflow Tests            | https://github.com/o3de/sig-simulation/blob/main/testing-guidance/workflow-tests/animation/footik-workflow.md                 | 30 Minutes per platform         |
| Animation | Root Motion Extraction workflows | https://github.com/o3de/sig-simulation/blob/main/testing-guidance/workflow-tests/animation/root-motion-extraction-workflow.md | 1 Hour per platform             |
| Physics   | Physics LKG                      | https://github.com/o3de/sig-simulation/blob/main/testing-guidance/workflow-tests/physics/physics-lkg-workflows.md             | 7 Hours 45 Minutes per platform |
| Physics   | Blast Material Workflows         | https://github.com/o3de/sig-simulation/blob/main/testing-guidance/workflow-tests/physics/physics-blast-materials-workflows.md | 30 Minutes per platform         |
| Physics   | Ragdoll Authoring Workflows      | https://github.com/o3de/sig-simulation/blob/main/testing-guidance/workflow-tests/physics/physics-ragdoll-workflows.md         | 1 Hour 30 Minutes per platform  |
| Physics   | Shapes LKG                       | https://github.com/o3de/sig-simulation/blob/main/testing-guidance/workflow-tests/shapes/shapes-lkg-workflows.md               | 1 Hour per platform             |
---
