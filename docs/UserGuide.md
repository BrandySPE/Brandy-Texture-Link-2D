# Brandy Texture Link 2D — User Guide

[README](../README.md) · [Quick Start](QuickStart.md) · [Support](Support.md) · [Maintenance](Maintenance.md) · [Change Log](ChangeLog.md) · [用户指南](用户指南.md)

This guide expands on the [Quick Start](QuickStart.md) with more detailed settings, usage notes, workflows, and troubleshooting.

## Recommended Practices

- Keep the project folder and related project assets on a local drive whenever possible. Other storage types may introduce extra delay or instability.
- For compatibility, the add-on also supports PSB in import and export workflows. Very large or highly complex PSB files may still fail to process, so simplify them first or convert them to PSD manually when practical.
- During certain waits, you can press `ESC` to cancel the wait and prevent the follow-up operation from continuing. Work that has already started may not stop immediately. For example, if you press `ESC` while the add-on is opening Photoshop, Photoshop may still finish launching, but the following operation will not continue.
- If an operation fails, use **Open Detailed Report** under **Settings & Reports** and follow the report before editing project JSON files or manually reconnecting Smart Objects.
- Do not manually edit or delete files inside the project folder while the add-on is running a task. Unless you clearly understand the project structure, avoid manual changes that could break project recognition or make the project unusable.
- When moving or switching project folders, use **Make Project Copy Independent** and **Switch Project** where appropriate. See **How the Project Link Works** below for details.

## Installation

In Blender, open `Edit > Preferences > Get Extensions`, open the menu in the upper-right corner, and choose **Install from Disk**. Select the complete Brandy Texture Link 2D.zip package. Do not extract the ZIP first.

Make sure Brandy Texture Link 2D is enabled under `Preferences > Add-ons`.

Return to the 3D View and press `N` to open the sidebar. Open the **Brandy** tab, then set **Photoshop Path** to `Photoshop.exe` in your Photoshop installation folder.

## Create a Project from a Blender Collection

If several Photoshop versions are installed, the add-on only uses the version configured under **Photoshop Path**. If a version conflict interrupts a task, close the other Photoshop versions and try again, or follow the message shown by the add-on.

Avoid having multiple Blender and Photoshop versions write to the same project at the same time.

### 1. Confirm the Texture Source

Under **Project Folder**, choose an empty local folder and keep **Texture Source** set to **Blender Collection**.

### 2. Choose a Blender Collection

Put all flat rectangular mesh objects you want to edit (referred to below as 2D sprites) into one Blender collection. Select that collection under **Project Collection**, then click **Create Project**.

At minimum, a valid 2D sprite should be a flat rectangular mesh with a unique name, a valid active UV layer, and one clearly identifiable main texture.

If a texture is packed into the `.blend` file, unpack it before creating the project and, when possible, keep it on a local drive.

A 2D sprite may have constraints, bones, animation, or similar Blender data, but only its static base composition at the time of project creation is sent to Photoshop.

A material may contain other nodes or supporting textures, but it must have one identifiable main texture. If several textures are possible candidates and the add-on cannot determine which one is primary, project creation stops and the reason is listed in the Detailed Report.

Blender's automatic numeric suffixes such as `.001` and `.002` are not treated as meaningful texture-name differences. For example, `Arm` and `Arm.001` may be recognized as duplicate names, so use clearly unique names before creating the project.

Option notes:

- **Children**: Includes objects from child collections under the selected collection.
- **Canvas Padding**: Adds transparent space around the overall texture layout so you have more visible room for full-layout painting in Photoshop. It only expands the working PSD canvas. It does not change the relative positions of the textures or increase the size of each individual texture.
- **Use PSB**: When disabled, the add-on creates a PSD by default. If either canvas dimension exceeds 30,000 pixels, the add-on switches to PSB automatically.

### 3. Create the Project

The add-on copies the source textures used by the 2D sprites into the project folder and creates a new PSD document that rebuilds the texture arrangement from Blender in Photoshop. The source textures you used before creating the project are not modified.

At this stage, the add-on records each texture's name, size, layout, and Alpha data. These records form the basis of the project link and are also used by later Apply and Split operations.

The add-on has some protection against temporary changes. For example, if you temporarily change a texture name, size, or layout in the PSD, an operation may be refused. Once the affected information is restored correctly, the operation can usually continue.

Changing the layer order in the PSD does not affect task execution, but it also does not change the texture stacking order recorded by the project.

You can rename or move a 2D sprite in Blender, but those changes are not synchronized back to the PSD document.

Changing or deleting the following can seriously damage the project:

- Blender UIDs;
- manually edited project structure, files, or links;
- the three layer groups **BTL2 Linked Content**, **BTL2 Merge Layers**, and **BTL2 Merged Layers**. These groups may be renamed temporarily, but if they are deleted, recreating three groups with the same names will not restore normal operation.

## Create a Project from a PSD Document

### 1. Confirm the Texture Source

Under **Project Folder**, choose an empty local folder, then change **Texture Source** to **Photoshop document**.

### 2. Choose a PSD Document

Under **PSD Document**, choose the PSD you want to import. Set options such as **Transparent Pixel Padding**, **Use Alpha**, and **Sprite Z Spacing**, then click **Create Project**.

For PSD files with complex structures, simplify the document when practical. If the file contains many linked Smart Objects, layer styles, or similar content, the add-on provides **Simplify Complex Layers**, **Skip Complex Layers**, and **Preserve Usable Content**. Complex effects may still look different after import.

Option notes:

- **Simplify Complex Layers**: Imports regular layers directly and tries to flatten complex layers or layer groups into single images. Content that cannot be handled safely is skipped.
- **Skip Complex Layers**: Imports regular layers directly and skips complex layers and complex layer groups. Regular child layers inside a complex group can still be imported if they can be handled independently.
- **Preserve Usable Content**: Imports regular layers directly. Complex layers or groups are first simplified as a whole. If a complex group cannot be safely simplified, the add-on expands it and continues importing usable child layers. Content that still cannot be handled is represented by a transparent placeholder that preserves its position and name.

You can also control the import with name prefixes:

- `[ignore]`: Add this to the start of a layer or layer-group name to skip that content.
- `[asset]`: Add this to the start of a layer-group name to process the whole group as one asset.

For complex content, the import preserves the current visible result and layout. It does not fully preserve the editability of text, masks, adjustment layers, layer styles, clipping relationships, or the internal structure of Smart Objects.

PSB can also be imported for compatibility, but very large or highly complex documents may not import completely.

Option notes:

- **Transparent Pixel Padding**: When each layer is cropped, this value adds transparent pixels around all four sides. It gives the exported texture extra canvas space for later edits or color bleed. It does not change the layer's position in the full composition, but it does increase the actual texture dimensions and storage use.
- **Ignore Hidden Layers**: Hidden layers and content inside hidden parent groups are not imported.
- **Use Alpha**: Connects the texture's Alpha output to material opacity and enables transparent display for the material. When disabled, the Alpha channel is not used for material display and the sprite is treated as opaque. This option does not delete or modify the Alpha channel in the texture file itself.
- **Premultiply Alpha**: Reads textures as premultiplied Alpha. Standard PNG files and other straight-Alpha textures should normally leave this disabled. Enable it only when the texture itself already uses premultiplied Alpha; otherwise, transparent edges may show color or brightness artifacts.
- **Import Scale**: Controls the overall conversion from Photoshop pixel coordinates to Blender units. It does not resize, resample, or change the pixel dimensions of the textures themselves.
- **Sprite Z Spacing**: Adds a fixed Z-axis gap between neighboring sprites to reduce flicker and ordering problems when planes would otherwise overlap exactly.

### 3. Create the Project

For each successfully imported layer or asset, the add-on creates a 2D sprite in Blender and uses the layer content as its texture. The front-to-back order of the sprites is built from the layer order in the PSD document.

Treat this as a one-time import. Later work is no longer linked to the source PSD, and the source PSD is not modified. After the import, the add-on follows the same project-linking workflow used when creating a project from a Blender collection and finishes by creating and linking a new PSD document.

Project textures created from a PSD document are saved as PNG files.

## How the Project Link Works

### 1. Project Folder

Whichever source you start from, Photoshop launches after the **Project Folder** is created.

The project folder is stored at the location selected under **Project Folder**. It is the working asset directory used to maintain the link between Blender and Photoshop.

If **Project Folder** points to a project previously created by the add-on, **Create Project** automatically changes to **Open Existing Project**.

If another project is already open and you select a different existing project folder, **Switch Project** appears below **Open Existing Project**. Use it to switch between linked projects.

During a switch, the original project remains active until the candidate project passes validation. If validation fails, the current project stays active and the incomplete candidate project is not committed.

If you make a complete copy of the current project folder at a new location, select that copy under **Project Folder**. The add-on shows **Make Project Copy Independent**. Running it registers the copy as a new independent project.

A complete project folder normally contains these directories:

- `textures`: the individual textures currently used and edited by the project.
- `photoshop`: the working PSD created by the add-on.
- `layout`: texture layout and link information.
- `jobs`: task records exchanged between Blender and Photoshop.
- `backups`: backups used for writeback, undo, and recovery.
- `source`: a copy of the source document when the project was created from a PSD document.

### 2. The Three Layer Groups

The working PSD contains three layer groups: **BTL2 Linked Content**, **BTL2 Merge Layers**, and **BTL2 Merged Layers**.

Do not delete these groups:

- **BTL2 Linked Content**: Contains the Smart Objects whose names match the 2D sprites. Their texture links, layout, and Alpha data are recorded when the project is created.
- **BTL2 Merge Layers**: Place new layers here for **Apply Merge Layers** and **Split Merge Layers**. Keep the group opacity at 100% and its blend mode set to `Normal` or `Pass Through`.
- **BTL2 Merged Layers**: Layers processed by **Apply Merge Layers** or **Split Merge Layers** are moved here automatically for Undo. This group is only an archive for working layers and does not replace project backups.

## Single-Texture Editing

After the project is created, the main editing controls appear below.

Select a 2D sprite in Blender and click **Edit Single Texture** to open its matching texture file for editing.

When **Auto Reload** is enabled (the button is highlighted), saving in Photoshop with `Ctrl+S` automatically triggers a texture reload in Blender. Auto Reload is not real-time synchronization, so there may be a delay of a few seconds after saving, depending on project size and hardware.

For single-texture editing, edit and save either the individual texture file or the individual Smart Object in the working PSD. Saving only the working PSD does not trigger Auto Reload.

When you edit and save a texture through **Edit Single Texture**, the add-on updates its stored Alpha data to match the texture's edited Alpha. This keeps the Alpha reference accurate after you erase existing pixels and affects how **Merge Into Transparency** and **Split Into Transparency** work later.

You can also disable **Auto Reload** and use **Reload Textures** manually.

**Edit Single Texture** can change the texture content and Alpha, but do not change the texture file's pixel dimensions. Using Crop, Canvas Size, or Image Size in a way that changes the project texture's dimensions will cause project size validation to fail.

You can also double-click a Smart Object in the working PSD, edit it, and save it to trigger Auto Reload. However, this editing method does not update that stored Alpha data.

Choose the editing method based on whether you need the stored Alpha data to be updated.

## Multi-Texture Editing

This workflow centers on **Apply Merge Layers** and **Split Merge Layers** and covers several common repainting needs.

This section is easier to follow together with the [YouTube demo](https://www.youtube.com/watch?v=-xTnPTlHHwc).

### 1. Apply Merge Layers

For example, suppose you want to paint on three textures named Leg_Thigh, Leg_Shin, and Leg_Foot.

Create three new layers in the working PSD and paint on them. Rename the layers to Leg_Thigh, Leg_Shin, and Leg_Foot, place all three inside **BTL2 Merge Layers**, then save the working PSD.

Click **Apply Merge Layers**. The add-on applies each new layer to the matching Leg_Thigh, Leg_Shin, or Leg_Foot texture by name, then reloads the changed textures in Blender.

The same workflow can be used with more than three layers.

Hidden layers stay where they are and are not included in the Apply operation.

This is the simplest and most predictable option in the multi-texture editing workflow.

With **Merge Into Transparency** disabled, new pixels that fall into transparent areas are clipped using the currently stored Alpha data, so the existing pixel edge does not expand.

With **Merge Into Transparency** enabled, new pixels may be written into transparent areas. Only content outside the original texture canvas is clipped.

### 2. Split Merge Layers

The setup is similar to **Apply Merge Layers**, and **Split Into Transparency** affects transparent areas in the same general way as **Merge Into Transparency**.

The difference is that **Split Merge Layers** automatically distributes overlapping painted pixels according to the texture stacking order recorded by the project. Once the frontmost texture receives a new pixel, a texture hidden behind it does not receive that same pixel again.

For example, suppose you want to paint one pattern across Leg_Thigh, Leg_Shin, and Leg_Foot, but the joint areas overlap and you do not want the new pixels to be written into those hidden areas.

Paint the full pattern on one new layer, then press `Ctrl+J` twice to create two copies. Rename the three layers to Leg_Thigh, Leg_Shin, and Leg_Foot, place them together inside **BTL2 Merge Layers**, and save the working PSD.

After you click **Split Merge Layers**, the part of the pattern around the thigh and knee is assigned to the frontmost Leg_Thigh texture. The Leg_Shin texture behind it does not receive the knee pixels again and only receives the visible part of the pattern on the shin.

This keeps the pattern continuous across Leg_Thigh, Leg_Shin, and Leg_Foot without stacking the same pixels in overlapping areas. It is especially useful around semi-transparent edges.

### 3. Split as One Layer

This option only affects **Split Merge Layers** and automates the distribution step.

When enabled, you only need to place one new layer inside **BTL2 Merge Layers**. The add-on detects all new pixels and distributes them to every covered texture using the same logic as **Split Merge Layers**.

If several layers are present, the add-on combines all new layers in **BTL2 Merge Layers** first, then performs the split.

This option is convenient but takes longer, so it is best used on smaller areas.

### 4. Undo Last Apply / Undo Last Split

After a valid **Apply Merge Layers** or **Split Merge Layers** operation, the corresponding **Undo Last Apply** or **Undo Last Split** action appears below the controls.

Undo restores the textures changed by the previous operation and moves the applied or split layers back into **BTL2 Merge Layers**.

Only the most recent operation can be undone, and Apply and Split share the same undo slot. For example, if you run **Apply Merge Layers** and then **Split Merge Layers**, the earlier Apply can no longer be undone.

An operation that makes no effective change, such as applying or splitting empty layers, does not replace the current undo state.

After **Make Project Copy Independent**, any unused undo history is cleared.

## Other Design Details

### Settings & Reports

If you run into a problem during normal use, **Open Detailed Report** can show why a task failed, execution details for some operations, and recommended steps after an error.

If automatic control of Photoshop is not working, you can change **Photoshop Run Mode** from **Windows COM** to **Manual Script**. In **Manual Script** mode, the operation generates a script file. In Photoshop, use `File > Scripts > Browse` and run `run_brandy_texture_link_2d_job.jsx` from the task folder. This mode is intended as a temporary fallback or for special cases, not as the normal workflow.

The interface is available in Chinese and English. Change the default language with **Interface Language** under **Settings & Reports**, then reload the add-on or restart Blender.

If you have questions or want to report an issue, see [Support](Support.md) or contact `brandyspe2026@gmail.com`. I will reply as soon as I can after receiving your email.

### Safety and Recovery

The add-on includes several context-sensitive safety and recovery controls. They only appear when relevant.

Use them according to the message shown by the add-on:

- **Open Task Records Folder**: Opens the scripts, JSON files, results, and diagnostic records for the current manual or abnormal task.
- **View Analysis Summary**: Shows layer-category counts and representative handling results from the PSD/PSB analysis.
- **View Full Analysis**: Shows the handling decision and reason for each layer.
- **Recover Interrupted Texture Update**: Restores textures from trusted backups after a multi-texture writeback was interrupted.
- **Recover Incomplete Project**: Completes a project whose working document was created but whose Blender-side link was not finished.
- **Repair Project Links**: Backs up and repairs project link records when project link IDs do not match, so the project can be opened or switched again.
- **Recover Project-Copy Transaction**: Recovers or completes an interrupted **Make Project Copy Independent** operation.
- **Clear Stale Project Lock**: Use only after confirming that write operations have stopped. It releases a stale project lock and clears related leftover task state. If an unfinished project-copy transaction exists, the add-on requires **Recover Project-Copy Transaction** first and does not allow the lock to be cleared separately.
- **Confirm Task Ended**: After confirming that the Photoshop task has ended, releases the project lock, keeps recoverable files, and clears the task state. It does not terminate Photoshop.
- **Request Cancellation**: Appears while a task is running. It writes `cancel.request`; Photoshop stops at a safe checkpoint. It does not force-close the Photoshop process.
- **Continue Task Recovery**: Appears when Blender has lost its normal monitoring connection to the original Photoshop task. It first checks the original task in read-only mode, then takes over the lock and commits the result only after a complete result has been confirmed.
- **Check Less Often**: Changes result monitoring to a lower-frequency background check. It does not stop Photoshop.

## Additional Tools

### Switch Texture Format

This tool looks in the same texture directory for an existing PNG, TGA, JPG, or JPEG file with the same base name and switches the texture to that file. It can be used to switch several textures at once.

It does not convert image formats. Shared images, unsaved images, and packed images are not modified.

### Convenience Tools

Option notes:

- **Copy Shader Settings to Selected Objects**: Copies shader settings from the last selected source object (the highlighted active object) to the other selected objects. Only unconnected inputs with matching names are copied.
- **Merge Duplicate Materials**: Safely merges duplicate materials generated by the add-on, provided they have not been modified.
- **Restore Imported Material**: Restores imported material and texture links from the import record.
- **Isolate Selection**: Temporarily shows only the selected objects in the current 3D View. You can move or edit them normally while isolated. Click the button again to restore the previous visibility state.

### Static PhotoshopToSpine JSON

This feature provides limited compatibility and is not a replacement for the PhotoshopToSpine plug-in.

It can import separate static textures and JSON exported by PhotoshopToSpine. The export function does not export textures; it only writes positional information from the current Blender composition to a compatible JSON file.

Many thanks to EsotericSoftware for PhotoshopToSpine. The plug-in saved me a great deal of time during my earlier 2D character work.

### Spine2D Region Attachment Import

This feature provides limited compatibility and does not create a link with Spine 2D itself.

It can only import static Region Attachments from Spine JSON and create rectangular sprites from the Setup Pose. Spine animation, Mesh Attachments, constraint systems, and animatable Blender rigs are not supported by this importer.

## Additional Limits

1. Texture round-trip editing is primarily designed for an 8-bit RGB and sRGB display workflow. It is not intended as a lossless round-trip workflow for CMYK, 16-bit or 32-bit images, or strictly color-managed master artwork.

2. JPG and JPEG are supported formats, but they do not support transparency and repeated saving introduces lossy compression. PNG or TGA is preferred for sprites, characters, and billboards.

3. A valid 2D sprite should meet the following conditions:

- all vertices lie on one 2D plane, form at least one valid face region, and belong to one connected region;
- the outer boundary has one closed loop and forms an axis-aligned rectangle in the project plane; outer edges may not be diagonal or curved;
- it has a valid active UV layer with one continuous rectangular UV layout, without local stretching or UV seams along the outer boundary;
- internal subdivisions, triangulation, and extra collinear vertices along rectangular edges are allowed, but internal holes, multiple disconnected mesh islands, or an edge shared by three or more faces are not;
- constraints, drivers, animation, and NLA data may exist, but they are not included as Photoshop round-trip data.

4. Active projects, backups, and recovery folders should preferably be stored on a local drive. Cloud-sync or network folders may work, but synchronization conflicts, latency, and non-local filesystem behavior can make project locking and recovery less reliable.

5. Auto Reload uses small background checks and reloads only after file stability is confirmed; it is not zero-latency synchronization.

Its polling behavior is approximately:

- fastest active interval: about 0.6 seconds;
- idle interval: about 1.5 seconds;
- medium and large projects: relaxed to about 2.5 or 4 seconds;
- per-scan time budget: about 8 ms;
- after a file change: about 0.35 seconds to confirm stability;
- repeated topology changes may trigger backoff or a paused state;
- validation is performed before an image is reloaded.

6. Selected operation limits and requirements are listed below:

| Item | Code limit or requirement | Behavior at the boundary |
|---|---|---|
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
