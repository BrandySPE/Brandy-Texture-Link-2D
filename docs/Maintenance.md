# Brandy Texture Link 2D — Maintenance

[README](../README.md) · [Quick Start](QuickStart.md) · [User Guide](UserGuide.md) · [Support](Support.md) · [Change Log](ChangeLog.md) · [维护信息](维护信息.md)

## Testing and Support

### Supported Environment

- Windows x64
- Blender 4.2.0 - 5.2.0, including 5.2.0
- Adobe Photoshop desktop CC2017 - 2026, including 2026

### Test Machines

| | Windows 11 25H2 | Windows 10 22H2 |
|---|---|---|
| CPU | AMD Ryzen 9 7950X, 16 cores | AMD Ryzen 5 3600, 6 cores |
| GPU | NVIDIA RTX 4060 Ti 16GB | NVIDIA RTX 2060 6GB |
| Memory | 32GB * 2 | 16GB * 2 |
| Storage | 3000GB HDD | 500GB SSD |

### Tested Versions

The add-on was tested across a 7 * 5 version matrix on August 21, 2026:

Blender: 4.2.21 LTS, 4.3.2, 4.4.3, 4.5.10 LTS, 5.0.1, 5.1.2, 5.2.0 LTS

Photoshop: CC2017 18.1.6, 2020 21.2.1, 2022 23.5.0, 2025 26.10.0, 2026 27.7.0

## Test Coverage

The test process is as follows:

Primary development uses Blender 5.1.2 and Photoshop 2025 26.10.0. Once a build is stable, Blender 4.2.21 LTS and 5.2.0 LTS are tested to confirm that the core features and project-linking workflow remain stable.

The Blender and Photoshop version matrix is then retested on a second machine. The matrix focuses on code paths that are more likely to differ across Blender, Python, and Photoshop versions, including project protocols, communication between applications, state changes, file writeback, Undo, and project isolation.

All test assets are stored in local project directories and include:

- one simple PSD with 5 layers;
- one small 4-layer PSD with a small amount of complex layering;
- one complex PSD with 49 layers;
- one real Blender project containing 49 2D sprites, bones, and animation.

### 1. Environment and Communication Protocols

Before each test run, the actual Blender, Photoshop, and add-on versions are confirmed, along with the following checks:

- version consistency between the add-on Manifest, runtime code, and bundled Photoshop JSX Bridge;
- the Blender version range declared by the add-on;
- the Photoshop version and installation path actually connected to the add-on;
- core communication protocols including Job, Result, Association, Project, Layout, Import Plan, and Import Analysis;
- protocol capabilities and task-state fields required by both Blender and Photoshop;
- the add-on's result-reading and validation flow after Photoshop completes a task, including confirmation of the final task state;
- confirmation that every Blender version is loading the same release-candidate add-on contents.

### 2. JSON, Multi-Texture Assets, and the Core Round-Trip Workflow

- JSON and separate textures import correctly;
- the correct number of rectangular Mesh objects is generated from the textures;
- Blender Mesh, Image, project Part, and Photoshop Smart Object mappings remain complete;
- UID, texture file, size, position, corner orientation, and stacking relationships are correct;
- the Photoshop working document is created through the normal project workflow and remains the current working document;
- PhotoshopToSpine JSON exported from a committed project contains the same number of exported entries as project Mesh objects, and its `images` path resolves to the matching project textures;
- **Edit Single Texture** opens the correct project texture, which can then be edited and saved in Photoshop;
- after a manual Blender reload, the Image Buffer reflects the updated image content;
- **Apply Merge Layers** changes the matching project textures;
- **Split Merge Layers** and **Split as One Layer** change the matching project textures;
- after Apply or Split, new layers are moved to the correct Photoshop archive location;
- Undo restores the affected project textures precisely and returns archived layers using their original Layer IDs and order;
- the working PSD remains open and saves normally through repeated editing, Apply, Split, and Undo operations;
- after the complete round-trip workflow, the project mapping between Blender, the project folder, and Photoshop remains valid.

### 3. PhotoshopToSpine and Spine Region Attachment

Static JSON compatibility testing includes:

- standard import of PhotoshopToSpine-style JSON and separate textures;
- generation of the correct number and layout of rectangular sprites from the JSON;
- export of static PhotoshopToSpine JSON from a committed Blender project;
- verification of texture count and the `images` path in exported JSON;
- import of static Region Attachments from standard Spine JSON;
- creation of matching rectangular sprites from the Setup Pose;
- confirmation that the original JSON and texture assets remain unchanged during import.

### 4. Auto Reload

The current project texture is edited and saved in Photoshop, then the background Timer registered by the add-on detects the file change.

The test verifies that:

- the Auto Reload Timer is registered correctly;
- the Blender Image Buffer for the changed texture updates automatically;
- the reload completes without calling the manual reload Operator;
- other project textures that were not changed remain untouched;
- after Auto Reload finishes, the Blender, project-folder, and Photoshop mappings for all Parts remain valid.

### 5. Open Existing Project

- The Photoshop working document reopens correctly;
- the Blender project Collection, Mesh, Image, and project identity are restored correctly;
- all Part mappings are rebuilt;
- opening the same project again does not create duplicate Blender data;
- project files do not change simply because the project was reopened;
- the current working document and project state remain correct.

### 6. Multi-Project Switching

Two independent projects, A and B, are created and switched in both directions using B → A → B.

The test verifies that:

- both projects use independent Blender Collections and project assets;
- creating project B does not modify project A;
- the current project identity and Photoshop working document update correctly after each switch;
- files and Blender data belonging to the inactive project remain intact;
- both projects retain complete Part mappings after repeated switching;
- operations in one project do not contaminate the other.

### 7. Make Project Copy Independent

The core transaction used by **Make Project Copy Independent** is tested on a copied project folder with three deliberately constructed Smart Object link states:

- all Smart Objects already point to textures inside the candidate copy;
- Smart Objects contain a mix of links to the parent project and candidate project;
- Smart Objects incorrectly point to a third-party file outside both projects.

The test verifies that:

- the candidate copy receives a new independent project identity;
- the parent project's identity and files remain unchanged;
- the candidate project protocol is updated to the new project identity;
- Smart Objects that need migration are actually relinked;
- project protocols, working document, and texture paths all point to the candidate project directory;
- the candidate project rebuilds a complete Part mapping;
- project locks, transaction state, temporary backups, and recovery state are cleaned up correctly after the transaction finishes.

For the invalid third-party link case, the test verifies that:

- the Photoshop task ends with `FAILED` and returns `LINK_MISMATCH`;
- the error identifies the affected layer and actual linked path;
- the candidate Photoshop document and project protocol remain unchanged;
- the parent project remains unchanged;
- no project lock, transaction state, temporary backup, or recovery state is left behind.

### 8. PSD/PSB Document Import

- PSD and PSB files are saved in the correct underlying file formats;
- Photoshop can reopen the generated baseline test document;
- the source document remains unchanged during analysis and import;
- the add-on receives a complete layer-analysis result;
- Layer ID, hierarchy path, layer type, visibility, canvas size, and Bounds match the observed Photoshop state;
- hidden layers are ignored correctly according to the setting;
- the default **Simplify Complex Layers** mode converts complex layer groups into a deterministic import result;
- the built-in baseline produces 3 imported textures;
- the Import Plan, exported PNG files, project Parts, and Blender Mesh count agree with one another;
- exported PNG files contain actual visible pixels;
- output texture dimensions for regular layers match the Photoshop Bounds;
- Blender Mesh dimensions and layout match the composition in the Photoshop document;
- the project working document retains the correct canvas size and PSD/PSB format;
- the Photoshop layer-group structure required by the project is created correctly.

### 9. JPG, TGA, and Everyday Utility Tools

- JSON import, image linking, and dimensions for JPG and TGA;
- TGA Alpha-channel data;
- restoration of deleted materials and images from import records;
- synchronization of unconnected Shader parameters with matching names;
- safe merging of duplicate materials generated by the add-on and left unmodified;
- switching texture format when a same-named file in the target format already exists, with Blender Image and import metadata updated together.

### 10. UI and Public Interface Contract

- registration, titles, and descriptions of core public Operators;
- presence, default values, and enum contents of Scene Properties used by everyday workflows;
- registration state of the add-on's `VIEW_3D` Panel;
- core Chinese and English UI text and formatting templates;
- key public interfaces used by the UI remain intact after version updates.
