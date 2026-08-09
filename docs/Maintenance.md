# Brandy Texture Link 2D — Maintenance

[README](../README.md) · [User Guide](UserGuide.md) · [Support](SUPPORT.md) · [Change Log](ChangeLog.md) · [维护信息](维护信息.md)

## Testing and Support

The current test process is as follows:

Development is carried out primarily with Blender 5.1.2 and Photoshop 2025 26.10.0. Once a build is stable, the core workflow and linking path are manually checked again with Blender 4.2.21 LTS and Blender 5.2.0 LTS.

After that, the full test matrix is repeated across the installed Blender and Photoshop versions listed below. Detailed coverage is described in the following sections.

The test assets come from real project files and include at least one simple PSD with 5 layers, one complex PSD with 49 layers, and one Blender file with 49 2D sprites. All test assets are stored in project folders on a local drive.

### Supported Environment

- Windows x64
- Blender 4.2.0–5.2.0, including 5.2.0
- Adobe Photoshop desktop CC2017–2026

### Test Machine

| Component | Configuration |
|---|---|
| Operating system | Windows 10 22H2 19045 |
| CPU | AMD Ryzen 5 3600, 6 cores |
| GPU | NVIDIA RTX 2060 6GB |
| Memory | 16GB × 2 |
| Storage | 500GB SSD |
| Texture formats | PNG / TGA / JPG / JPEG |
| Interface languages | Simplified Chinese / English |

### Tested Versions

The following 7 × 5 matrix was completed on August 5, 2026:

| Software | Tested versions |
|---|---|
| Blender | 4.2.21 LTS / 4.3.2 / 4.4.3 / 4.5.10 LTS / 5.0.1 / 5.1.2 / 5.2.0 LTS |
| Photoshop | CC2017 18.1.6 / 2020 21.2.1 / 2022 23.5.0 / 2025 26.10.0 / 2026 27.7.0 |

### Test Coverage

1. Environment and protocol identity validation

- Add-on Manifest ID and runtime module identity;
- version consistency across the Manifest, add-on runtime, and bundled Photoshop JSX Bridge;
- add-on installation fingerprint;
- declared Blender support range;
- Photoshop communication backend and the actual Photoshop application identity in use;
- Photoshop COM Observer application version and installation path;
- protocol contracts required for Job, Result, Association, Project, Layout, Import Plan, and related records;
- final Photoshop task state confirmed through the add-on's standard result-reading and validation flow.

2. JSON / multi-texture asset import and core roundtrip workflow

- JSON and individual texture import;
- correct number of rectangular Mesh objects generated from textures;
- Photoshop project creation;
- project-part count validation;
- complete mapping between Blender Meshes, Images, project Parts, and Photoshop Smart Objects;
- UID, file path, dimensions, position, corner orientation, and ordering consistency;
- PhotoshopToSpine JSON export from a committed project;
- exported texture count and `images` path validity;
- selected-texture editing entry point;
- actual texture-pixel edits in Photoshop;
- visible Blender Image Buffer change after manual reload;
- Blender Image Buffer restoration after restoring the source file;
- actual source-texture byte changes after **Apply Multi-Texture Edits**;
- correct archiving of Photoshop test layers;
- complete SHA-256 restoration of source textures and the Photoshop working document after revert;
- original input assets remain read-only throughout the roundtrip.

3. Limited Spine Region Attachment import

Standard Spine Region JSON and sprite assets are used to verify the limited compatibility mode, including:

- the normal Spine Region Attachment JSON import entry point;
- two textures correctly generating two rectangular sprites;
- source assets remaining unchanged.

4. Auto Reload

After Photoshop modifies a project texture, the following are verified without invoking the manual reload Operator:

- the Auto Reload timer is registered correctly;
- the Blender Image Buffer for the modified texture updates automatically;
- unmodified project textures remain unchanged;
- all project Part mappings across Blender, the project folder, and Photoshop remain valid.

5. Open Existing Project

Existing Brandy Texture Link 2D projects are reopened through the public add-on entry point and checked for:

- the Photoshop working document actually opening;
- restoration of the Blender project Collection, Meshes, Images, and project binding data;
- restoration of all Part mappings;
- reopening the same project does not generate duplicate Blender resources;
- no modification of project-folder files during opening;
- no leftover document-launch helper process after completion.

6. Multi-project switching

Two independent projects, A and B, are created and verified for:

- independent Blender Collections;
- creating project B does not modify project A;
- after switching B → A, the current committed project is A;
- inactive project B remains intact and its files are not modified;
- after switching A → B, the current committed project is B;
- inactive project A remains intact;
- complete Blender–project-folder–Photoshop mappings remain valid for both projects throughout the process.

7. Make Project Copy Independent / Fork

After a project folder is copied, the add-on's project-copy transaction is tested under three real Smart Object link states:

- all Smart Objects already point to the copied project's textures;
- Smart Objects contain a mix of parent-project and copied-project texture links;
- a Smart Object incorrectly points to a third-party file outside the project folder.

Successful cases verify:

- the copied project receives a new project identity;
- the parent-project identity remains unchanged;
- parent-project files remain byte-for-byte unchanged;
- the copied project's protocol data and project identity are actually updated;
- Photoshop Smart Objects are relinked when required;
- project protocol data, working document, and texture paths all point into the copied project directory;
- Blender remains bound to the parent project during the transaction;
- project locks, transaction state, temporary backups, and pending-recovery state are fully cleaned after completion.

The incorrect-link case must be rejected deterministically:

- the Photoshop task must end in `FAILED`;
- the error code must be `LINK_MISMATCH`;
- the error message must identify the relevant layer and the parent-project, copied-project, and actual link paths;
- the copied Photoshop document and project protocol data must not be modified;
- no project lock, transaction marker, temporary backup, or pending-recovery state may remain;
- the current Blender project must not be switched incorrectly.

8. PSD / PSB document import

For every Blender version, deterministic PSD and PSB baselines generated and frozen by QA in the current Photoshop version are used to verify:

- actual PSD and PSB generation by Photoshop;
- `8BPS` file-header validation;
- PSD format version = 1;
- PSB format version = 2;
- Photoshop can reopen the generated baseline documents;
- the original documents and test working copies remain byte-for-byte unchanged;
- the project working document retains the actual PSD/PSB format of the source document;
- the add-on's layer analysis result is not empty;
- layer ID, path, layer type, visibility, and canvas dimensions match an independent Photoshop Observer result;
- Import Plan, exported PNG files, project Parts, and Blender Mesh counts are consistent;
- exported texture pixel dimensions match Photoshop layer bounds;
- Blender Mesh dimensions and placement match the document layer positions;
- the project working-document canvas matches the source document;
- reserved-group count and structure are valid;
- the built-in complex-document baseline produces the expected result under the default **Simplify Complex Layers** mode.

9. Everyday utility tools

The following tools are checked for actual results rather than only verifying that their Operators can be invoked:

- restoring deleted materials and Images from import records;
- copying matching unlinked shader parameters;
- safely merging unmodified duplicate materials generated by the add-on;
- switching texture format when a target-format file already exists and updating both the Blender Image and import metadata.

10. UI and public-interface contract

- registration, labels, and descriptions of 17 core public Operators;
- existence, default values, and enum contracts of 13 Scene Properties used by everyday workflows;
- registration of 5 `VIEW_3D / UI` add-on panels;
- core Chinese and English UI strings and formatting-template contracts.

## Safety Design

1. **Open Detailed Report** provides failure reasons, execution details for some tasks, and recommended follow-up actions for normal troubleshooting.

2. In **Manual Script** mode, an operation generates the corresponding script file, which can then be run manually from Photoshop through `File` > `Scripts`. This provides a fallback path for environments where automatic control is unavailable.

3. The add-on includes several context-sensitive actions for reconnecting, recovering, or cleaning up interrupted operations. See `Additional Details` in the [User Guide](UserGuide.md).

4. The add-on itself performs no network communication and does not depend on cloud services. Project files, task protocols, backups, and diagnostic information are handled locally. Photoshop, the operating system, cloud-sync software, and other host-environment components may have their own network behavior, which is outside the add-on's control.

5. During operations, the add-on performs checks or handling at different stages, including:

- validating the identity of the Photoshop launch target;
- cross-checking project data so that missing files, mismatched identities, hash mismatches, incomplete projects, and similar conditions do not silently continue;
- validating project paths and directory boundaries, including long or invalid paths;
- checking illegal characters, control characters, and related path hazards when filenames are generated from layer, object, or asset names;
- validating job IDs, request/result protocol versions, project-lock versions, capabilities, and related JSON data, and requiring a `result.ready` marker after Photoshop finishes writing a result;
- recording the Blender instance ID, process PID, and process start time when available in project locks to reduce the risk of unknown project writes;
- writing JSON files, task states, and marker files in multiple stages instead of progressively overwriting the final target file, reducing invalid leftovers after interruptions;
- creating verifiable backups before destructive operations;
- recording incomplete-state markers and stage logs during project creation and PSD import, with a project becoming complete only after both Photoshop output and Blender commit succeed;
- during PSD import, treating the external source document as read-only input. The add-on first creates a validated archive copy inside the project, then generates an independent working document from that copy rather than writing directly to the original PSD or PSB;
- isolating writes across projects;
- sending a cancellation request from Blender when the user requests an interruption, with the JSX side responding at safe checkpoints. The add-on does not forcibly terminate the Photoshop process;
- saving diagnostics locally and reducing exposure of known local paths in the Detailed Report.

## Usage Notes

1. Texture roundtripping is primarily designed around an 8-bit RGB and sRGB display workflow. It is not intended as a lossless roundtrip solution for CMYK, 16-bit or 32-bit images, or strict color-managed master artwork.

2. JPG and JPEG are supported, but they do not support transparency and repeated saving introduces lossy compression. PNG or TGA is recommended for sprites, characters, and billboards.

3. A valid `2D sprite` should meet the following conditions:

- all vertices lie on one 2D plane, form at least one valid face area, and belong to one connected region;
- the outer boundary has one closed loop and forms an axis-aligned rectangle in the project plane; the outer edges may not be diagonal or curved;
- an active UV layer contains one continuous rectangular UV layout without local stretching or outer-boundary UV seams;
- internal subdivisions, triangulation, and additional collinear vertices along rectangular edges are allowed, but internal holes, multiple disconnected mesh islands, or an edge shared by three or more faces are not;
- constraints, drivers, animation, and NLA data may exist, but they are not included as Photoshop roundtrip data.

4. Active projects, backups, and recovery folders should preferably be stored on a local drive. Cloud-sync or network folders may work, but synchronization conflicts, latency, and differences in non-local filesystem behavior can reduce the reliability of locking and recovery behavior.

5. Auto Reload should be understood as `background checks in small batches, followed by reload after file stability is confirmed`, not zero-latency synchronization.

The polling behavior is approximately:

- fastest active interval: about 0.6 seconds;
- idle interval: about 1.5 seconds;
- medium and large projects: relaxed to about 2.5 or 4 seconds;
- per-scan time budget: about 8 ms;
- after a file change: about 0.35 seconds to confirm stability;
- repeated topology changes may trigger backoff or a paused state;
- validation is performed before an image is reloaded.

6. Selected per-operation limits and requirements are listed below:

| Item | Code limit or requirement | Behavior at the boundary |
|---|---:|---|
| PSD/PSB import units | Up to 2048 items | Includes texture items in the final Import Plan |
| PSD/PSB document width/height | Up to 300,000 px per side | Add-on safety limit |
| Single exported texture width/height | Up to 300,000 px per side | Practical limits are usually lower |
| Normalized canvas width/height | Up to 300,000 px per side | Also checked when creating directly from Blender |
| Total exported PSD texture pixels | Up to 250,000,000 px | Combined total for all output textures |
| Transparent pixel padding | 0–8192 px | Enforced by both UI and JSX |
| Automatic PSB selection | Either canvas dimension exceeds 30,000 px | Direct project creation automatically uses PSB |
| General import JSON | Maximum 64 MiB | Larger files are blocked |
| Cancellation confirmation wait | 60 seconds | Photoshop is not forcibly terminated |
| Result-reading retries | Up to 5 attempts | Also uses an approximately 5-second retry window |
| Auto Reload files per scan | Up to 32 files | Per-scan time budget is about 8 ms |
| Auto Reload images per reload pass | Up to 2 images | Helps avoid blocking the Blender UI |
| Auto Reload failed attempts | Up to 5 attempts | Delays are approximately 1, 2, 5, and 10 seconds |
