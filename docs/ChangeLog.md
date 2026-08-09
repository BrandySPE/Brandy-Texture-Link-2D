# Brandy Texture Link 2D — Change Log

[Home](../README.md) · [User Guide](UserGuide.md) · [Support](Support.md) · [Maintenance](Maintenance.md) · [更新日志](更新日志.md)

## Brandy Texture Link 2D v1.1.0

**Brandy Texture Link 2D** was previously released as **Brandy 2D Link**. Version **1.1.0** is the first public release under the new name.

This change log starts from the renamed product line and summarizes the work completed between the internal test build **1.0.0** and the public **1.1.0** release.

### Create Projects from Photoshop Documents

- Added a complete one-time project-creation workflow from Photoshop documents. The add-on can read PSD/PSB layers and layout, export usable content as independent transparent textures, and automatically create the Blender project collection, rectangular sprites, materials, and image data.
- Automatically generates a Photoshop working document and complete project folder that can continue into the normal multi-texture roundtrip workflow. The source document is used only for the initial import and is not modified; later editing and synchronization use the newly created project.
- Complete Brandy Texture Link 2D working documents are recognized as existing projects so they are not imported again as ordinary Photoshop documents.
- Added **Transparent Pixel Padding**, **Ignore Hidden Layers**, **Use Alpha**, **Premultiply Alpha**, **Import Scale**, and **Sprite Z Spacing** import settings.
- Added complex-layer detection with **Simplify Complex Layers**, **Skip Complex Layers**, and **Preserve Usable Content** handling modes. The add-on preserves layer names, stacking order, pixel dimensions, and canvas positions where possible, and reports content that cannot be processed safely.
- Improved PSD and PSB creation. PSD is used by default; PSB is used when **Use PSB** is enabled or the PSD size limit is exceeded. The background save result and actual file format are validated afterward.

### Project Creation, Opening, and Switching

- Rebuilt the project entry flow. The add-on now determines whether to create a new project or open an existing one from the selected source, project path, and current project state. Project switching remains a separate action to reduce unnecessary mode choices and accidental operations.
- **Open Existing Project** can now restore Blender assets from a validated project folder, including sprite layout, Import Scale, horizontal-axis direction, Sprite Z Spacing, materials, and texture links.
- Improved detection and recovery for moved projects, abnormal link records, and incomplete projects so damaged or unfinished projects are not treated as writable projects.
- Added **Make Project Copy Independent**. After copying a project directory, the copied project can receive its own project and document identity, and Photoshop linked Smart Objects can be safely relinked to the copied textures instead of continuing to write to the parent project.
- Added a dedicated recovery transaction for project copies. The add-on distinguishes between copies that were not modified, copies that were already modified, and states that cannot be confirmed. This reduces unnecessary rollback while keeping the project lock and recovery entry point when the state is uncertain.
- Tightened the scope of **Edit Single Texture** and **Restore Imported Material** so they only operate on resources formally recorded by the current project, reducing the risk of modifying same-named files, shared meshes, or shared data-blocks.

### Photoshop Roundtrip Editing and Task Recovery

- Improved **Apply Multi-Texture Edits**, **Revert Last Applied Edit**, and interrupted write-back recovery. Required backups are created before writing, strict validation is performed afterward, and failed, interrupted, or inconsistent operations can be rolled back or recovered from preserved records.
- Long-running Photoshop tasks are no longer treated as unknown simply because they exceed the high-frequency check period. Blender can switch to lower-frequency checking while Photoshop continues the task.
- Photoshop tasks are now persistently associated with the Blender file session that submitted them. Tasks can continue to be recovered after an add-on reload, while switching Blender files isolates older tasks so their results are not committed into the wrong file.
- A Photoshop cancellation request is sent only when the user explicitly requests an interruption. Reloading the add-on or stopping high-frequency checking is not treated as task cancellation.
- Each Photoshop operation now validates its own runtime environment before execution, so users no longer need to run a separate Photoshop check step first.

### Auto Reload and Performance

- Rebuilt Auto Reload around shared monitoring state per project collection, preventing the same project from being scanned and reloaded repeatedly across multiple scenes or views.
- File checks, project-structure updates, and dependency handling are now processed in smaller stages with delayed batching and controlled retries, reducing unnecessary overhead and UI stalls while monitoring larger projects.
- Manual reload, project switching, multi-texture editing, and revert operations now update the monitoring baseline. Re-enabling Auto Reload also detects texture changes made while it was disabled.
- Fixed Auto Reload handlers and runtime state being lost when loading another Blender file, reloading modules, or working in multi-window setups.

### Data Safety and Stability

- Strengthened project locks, task state, atomic writes, file fingerprints, hash validation, pre-write backups, rollback, and abnormal-exit recovery throughout the workflow.
- Added persistent stage records and file-ownership lists for incomplete projects. Cleanup only removes content that can be confirmed as created or taken over by the current transaction, avoiding deletion of unknown files or valid Photoshop output.
- Tightened project-directory boundary checks to reject symbolic links, directory junctions, and external paths that bypass the project scope. External JSON input is also limited by file size, nesting depth, and data complexity.
- Added Windows long-path preflight checks covering project files, textures, task records, backups, temporary files, and dynamic Photoshop output paths, with consistent actionable error messages.
- Improved cross-validation between Photoshop documents, project metadata, layout data, texture records, and Blender data so the add-on does not continue writing when project identity, layer order, texture paths, or canvas layout are inconsistent.
- Improved Manual Script mode, project-link repair, task-record folders, and Detailed Reports so external-call failures or interrupted monitoring still leave useful diagnostic and recovery information.

### Interface and Feedback

- Reorganized the Photoshop workflow UI so project configuration, task state, and editing actions for the connected project are presented in one workflow panel and shown according to the actual project state.
- Separated Photoshop status, task status, current project, switch target, link status, and safety status to reduce repeated messages and keep low-level technical details out of the main workflow.
- Expanded Chinese and English interface coverage and standardized terminology for projects, layers, complex-content handling, Premultiply Alpha, Photoshop Run Mode, task checking, and related features.
- Reduced routine success messages and persistent explanations. Detailed errors, paths, and technical diagnostics are now concentrated in the Detailed Report while keeping the recovery actions and guidance needed for failed operations.
