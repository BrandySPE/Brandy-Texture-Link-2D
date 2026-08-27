# Brandy Texture Link 2D — Change Log

[README](../README.md) · [User Guide](UserGuide.md) · [Support](SUPPORT.md) · [Maintenance](Maintenance.md) · [更新日志](更新日志.md)

## Brandy Texture Link 2D v1.1.0

**Brandy Texture Link 2D** was previously named **Brandy 2D Link**. Version **1.1.0** is the first public release under the new name.

This page uses the renamed 1.1.0 series as its baseline and summarizes the main workflow, stability, interface, and feature changes made from the internal **1.0.0** test version through the current 1.1.0 revision. Individual development revision numbers are not listed; related changes are grouped by their final behavior.

## Create Projects from Photoshop Documents

- Added a complete one-time import workflow for creating a project from a Photoshop document. The add-on can read PSD/PSB layers, layout, and canvas information, export usable content as separate transparent textures, and automatically create the Blender project collection, rectangular sprites, materials, and image data.
- The add-on now creates the working Photoshop document and complete project folder used for later round-trip editing. The source PSD/PSB is used only for the initial import and is not modified. After import, later editing and synchronization use the newly created project.
- Added complex-layer detection with three handling modes: **Simplify Complex Layers**, **Skip Complex Layers**, and **Preserve Usable Content**. Usable layer names, order, pixel dimensions, and canvas positions are kept according to the selected handling result.
- Added `[ignore]` to skip specific layers or layer groups, and `[asset]` to process a layer group as one asset.
- Added import settings including **Transparent Pixel Padding**, **Ignore Hidden Layers**, **Use Alpha**, **Premultiply Alpha**, **Import Scale**, and **Sprite Z Spacing**.
- Improved PSD and PSB creation. PSD is the default; enabling **Use PSB** or exceeding the PSD canvas limit switches the working document to PSB, with validation of the saved file and its actual format.
- Complete Brandy Texture Link 2D working documents are recognized as existing projects so they are not imported again as ordinary Photoshop documents.

## Project Creation, Opening, Switching, and Recovery

- Reworked the project entry flow. Based on **Texture Source**, **Project Folder**, and the current project state, the add-on automatically determines whether the action should be **Create Project** or **Open Existing Project**. Switching between already linked projects remains a separate **Switch Project** operation.
- **Open Existing Project** can restore the Blender Collection, Mesh, Image, materials, and project Part mapping from a validated project folder, including recorded layout, import scale, coordinate orientation, and sprite spacing.
- Opening the same linked project again does not create duplicate Blender data and does not rewrite project files simply because the project was reopened.
- Improved multi-project switching. A candidate project is validated for identity, project structure, and asset mapping before it replaces the current project. If the switch fails, the original project remains active.
- Improved detection of fully relocated projects, broken link records, and incomplete projects. An incomplete project is no longer identified from a single marker alone; persistent stage records and the actual project structure are also used to determine whether recovery can continue.
- Improved **Recover Incomplete Project** so a project whose Photoshop working document was already generated, but whose Blender-side link was not committed, can continue from that state.
- After creating, opening, recovering, or switching a project, the matching Photoshop working document is opened or brought forward so the active Blender project and Photoshop document stay aligned.

## Single-Texture Editing and Alpha Boundaries

- Restricted **Edit Single Texture** to texture assets recorded by the current project, reducing the risk of opening same-named files, shared data, or assets outside the project by mistake.
- **Edit Single Texture** now opens the project source texture for the selected Part directly, while keeping the normal Photoshop edit-and-save workflow.
- Projects record Alpha boundary information for project source textures so later multi-texture operations can distinguish existing pixels from areas that were originally transparent.
- After a source texture is edited and saved through **Edit Single Texture**, the add-on validates the new file state before a later operation and updates the matching Alpha boundary information. Erasing or adding pixels in a single texture can therefore become the new reference used by **Merge Into Transparency** and **Split Into Transparency**.
- Alpha boundary updates are tied to the identity of the project source texture and do not change simply because the Blender image is reloaded.

## Apply and Split Merge Layers

### Apply Merge Layers

- Improved **Apply Merge Layers**. The add-on reads visible layers in **BTL2 Merge Layers** whose names match project textures and writes each layer back to its matching source texture.
- Hidden layers are not included in the current Apply operation and remain in place.
- With **Merge Into Transparency** disabled, new content is clipped to the Alpha boundary recorded by the project and can be written only within the existing covered area. When enabled, new content can also be written into transparent areas inside the source texture canvas. Content outside the source texture canvas is still clipped.
- Successfully applied source layers are archived automatically in **BTL2 Merged Layers**. The matching Photoshop Smart Objects are updated and Blender reloads only the textures actually affected.

### Split Merge Layers

- Added a separate **Split Merge Layers** workflow for painted content that crosses overlapping project textures.
- The add-on decides which texture receives each pixel from the actual coverage of the project textures and their fixed project order. In overlapping areas, the frontmost texture receives the pixel first, and a texture behind it does not receive the same pixel again.
- Distribution is based on actual edited pixels and project texture coverage rather than only layer rectangle bounds. This avoids writing the same overlap back multiple times, which can otherwise increase Alpha values or stack soft edges.
- With **Split Into Transparency** disabled, distribution stays within the Alpha coverage recorded by the project. When enabled, remaining new content can also enter transparent areas of target textures.
- After Split completes, the source layers that were actually processed are also moved into **BTL2 Merged Layers**, and the working PSD/PSB and modified project source textures are saved and updated together.

### Split as One Layer

- Added **Split as One Layer**. When enabled, visible content in **BTL2 Merge Layers** is first treated as one combined result and then distributed automatically to project textures based on project texture coverage.
- If several usable layers are present in the merge group, their content is combined before distribution, so users do not need to create a same-named copy for every target texture first.
- Normal Split and **Split as One Layer** use the same project texture order, pixel-coverage logic, backups, writeback, and recovery flow.

### Multi-Texture Writeback

- **Apply Merge Layers** and **Split Merge Layers** now share the same writeback infrastructure, including source-format checks, trusted backups before writing, file-integrity validation, saving the Photoshop working document, Smart Object updates, rollback on failure, and undo records.
- After Photoshop reports success, Blender verifies the actual source-texture changes, project paths, and operation records before the new undo state is accepted and the writeback is considered complete.
- Multi-texture writeback to JPG/JPEG source textures now warns about additional lossy compression from resaving and creates the corresponding backups before the task starts.

## Undo Last Apply or Split

- Reworked the undo system. **Apply Merge Layers** and **Split Merge Layers** now share one “last operation” record. The interface shows **Undo Last Apply** or **Undo Last Split** according to the most recent valid operation.
- Undo restores only the project source textures actually modified by the most recent operation rather than rolling back the entire project unconditionally.
- Original layers archived to **BTL2 Merged Layers** by that operation are restored to **BTL2 Merge Layers** at the same time, using the recorded Photoshop Layer IDs and original order.
- Operations that make no effective change do not consume the current undo. For example, if there is nothing to Apply or Split, an existing valid undo remains available.
- A new valid Apply or Split replaces the previous undo record. After **Make Project Copy Independent**, the copy does not inherit unused undo history from the parent project.
- Added pre-undo state checks. If an affected project source texture was saved again after the operation, or Photoshop has unsaved changes that would be overwritten by Undo, the add-on checks and reports the condition before proceeding.
- Undo now has its own transaction stages and recovery records. After an interruption or abnormal exit, the saved before/after state can be used to continue recovery verification instead of treating a half-finished Undo as complete.
- When Undo finishes, affected Blender textures are reloaded and Photoshop documents that needed to be reopened during the task are restored.

## Photoshop Startup and Task Recovery

- Reworked Photoshop startup in **Windows COM** mode. Operations that require Photoshop now start the application configured under **Photoshop Path** automatically; users no longer need to open Photoshop separately first.
- Photoshop availability is no longer decided only by whether a `Photoshop.exe` process exists. The add-on verifies the actual automation connection and JSX capability before starting a task.
- When several Photoshop versions are installed, the add-on checks which installation is actually responding to automation requests so tasks are not sent to a version that does not match the add-on setting.
- Improved handling when Photoshop is already exiting. If an operation starts while the old Photoshop process is still shutting down, the add-on continues checking its process state, then starts the configured Photoshop version once the previous process has fully exited.
- Photoshop preparation now happens before the project lock, backup, and writeback transaction. If preparation fails or is cancelled, the add-on does not create a write task that never actually began.
- While waiting for Photoshop to become ready, `Esc` or the right mouse button can cancel the current wait. This only stops the follow-up operation; it does not force-close a Photoshop process that has already started.
- Long-running Photoshop tasks remain under add-on control after the high-frequency foreground checking stage ends. **Check Less Often** moves result monitoring to a lower frequency while Photoshop continues the original task.
- Only **Request Cancellation** sends a cancellation request to the current Photoshop task. Photoshop handles it at a safe checkpoint; the process is not forcibly terminated.
- Photoshop task state, project ownership, and Blender file session are stored persistently. After the add-on is reloaded, related tasks can resume recovery. Loading another Blender file isolates the original task so its result is not committed into the wrong file.
- **Continue Task Recovery** first checks the original Photoshop task and its complete result in read-only mode. It takes over the project lock and continues the commit only after safe ownership can be confirmed.
- **Manual Script** remains available as a fallback for environments where automatic COM execution cannot be used, generating the JSX task for manual execution.

## Make Project Copy Independent

- Improved **Make Project Copy Independent**. After a complete project directory is copied, the copy can receive a new project identity and Photoshop document identity so it no longer behaves as another instance of the parent project.
- The independence process updates the candidate project's project protocol, layout, and link records, and relinks Smart Objects in the Photoshop working document to the project source textures inside the copy.
- Strengthened Smart Object link-state validation. Links that can be confirmed as belonging to either the parent project or candidate copy are handled by the transaction rules; links that point to unrelated external files are detected and stop the commit.
- Project-copy independence uses a separate transaction flow with pre-write backups, state records, change detection, commit validation, failure rollback, and abnormal-exit recovery.
- The add-on distinguishes copies that have not changed, copies that have changed, and states that cannot be fully verified. This avoids unnecessary rollback while preserving a recovery path for uncertain transactions.
- After the independence process completes normally, Blender automatically switches to the new independent copy and opens or activates its matching Photoshop working document.
- Improved post-commit verification for copies. If the project was committed but final content verification or transaction cleanup did not finish, the add-on enters a dedicated recovery state instead of mixing it with an ordinary unconfirmed Photoshop task.
- Added **Retry Verification** and **Confirm Cleanup** for this state. The first reruns post-commit verification. The second only removes temporary backups and transaction markers owned by the already committed transaction; it does not roll the committed project back.

## Auto Reload and Large-Project Performance

- Reworked **Auto Reload** so monitoring state is shared per project collection instead of scanning and reloading the same project repeatedly across multiple scenes, views, or windows.
- File-change checks, project-structure checks, and dependency processing now run in smaller stages with scan budgets, stability waits, delayed batching, and controlled retries to reduce Blender UI stalls during continuous monitoring of large projects.
- Auto Reload only reloads project textures that actually changed and limits the work done in each scan and reload pass. Repeated failures use delayed retries and a blocked state instead of continuously retrying at full speed.
- **Reload Textures**, project switching, multi-texture writeback, and Undo now update monitoring state together. Re-enabling **Auto Reload** after it has been disabled also does not simply ignore file changes that happened while it was off.
- Improved feedback from **Reload Textures** while **Auto Reload** is disabled so manual reload still clearly reports success, partial completion, or a blocked result.
- Fixed cases where Auto Reload handlers or runtime state could be lost after loading another Blender file, reloading add-on modules, or working in multi-window setups.
- Optimized large-project creation by reducing repeated I/O and global scans during project texture copying, initial Alpha-boundary creation, Blender Object/Image lookup, and project recovery-state writing.
- When several Parts share the same project source texture, previously validated file results can be reused to reduce repeated copying and hash reads.
- Optimized temporary processing in **Split Merge Layers**. Pixel selection and writeback are completed target by target, with temporary selections, channels, and intermediate layers released as soon as possible to reduce Photoshop resource use on large multi-texture projects.

## Data Safety, Interface, and Localization

### Data Safety and Stability

- Strengthened project locks, task state, atomic writes, file fingerprints, SHA-256 validation, pre-write backups, rollback on failure, and abnormal-exit recovery throughout the workflow.
- Separate transaction stages and recovery information are now stored for incomplete projects, Apply, Split, Undo, and project-copy independence so each type of interruption can continue according to its actual state.
- Added a persistent ownership list for incomplete projects. Cleanup only removes files that can be confirmed as created or taken over by the current transaction, reducing the risk of deleting unknown files or valid Photoshop output.
- Strengthened project-directory boundary checks to prevent symbolic links, directory junctions, or external paths from bypassing project scope.
- Added size, nesting-depth, and data-complexity limits for external JSON, plus safety checks for PSD/PSB import counts, canvas size, and total exported pixel volume.
- Added and expanded Windows long-path preflight checks across project files, textures, Photoshop working documents, task records, backups, temporary files, and dynamic output paths.
- Improved cross-validation between project metadata, Layout, Association, Photoshop documents, project source textures, and Blender data. Writes do not continue when project identity, Part UID, layer order, texture path, or layout state is inconsistent.
- Improved recovery entry points including **Repair Project Links**, **Recover Interrupted Texture Update**, **Recover Incomplete Project**, **Recover Project-Copy Transaction**, and **Clear Stale Project Lock**, with detailed paths, causes, and results collected in **Open Detailed Report**.

### Interface and Localization

- Reorganized the Photoshop workflow panel so project setup, current project, task state, and editing controls for linked projects follow the actual workflow order.
- Added **Split Merge Layers**, **Split Into Transparency**, **Split as One Layer**, and the matching **Undo Last Split** state.
- Removed the old separate **Open Photoshop** action as Photoshop startup became part of normal task preparation. Operations that need Photoshop now start and prepare it automatically.
- Photoshop state, task state, current project, candidate project, Auto Reload state, and safety/recovery state are now shown separately to reduce repeated information and keep internal task details from getting in the way of normal use.
- Expanded Chinese and English UI coverage and standardized terminology around projects, complex-layer handling, Alpha, Premultiply Alpha, Apply, Split, Undo, task recovery, and project copies.
- **Auto** under **Interface Language** follows the add-on's current display language, while Chinese and English continue to show their own language names.
- Added missing bilingual text in lower-frequency task monitoring, abnormal recovery, Detailed Report, Undo Operator, and related paths, and standardized punctuation and formatting templates in English error messages.
- Simplified routine success messages and always-visible guidance. Detailed errors, file paths, and technical diagnostics are concentrated in **Open Detailed Report**, while direct actions remain visible when the user needs to respond.
- Removed unused legacy actions, compatibility branches, and Photoshop JSX helper code so the public interface stays aligned with the actual current task implementation.
