# Brandy Texture Link 2D — User Guide

[README](../README.md) · [Support](SUPPORT.md) · [Maintenance](Maintenance.md) · [Change Log](ChangeLog.md) · [用户指南](用户指南.md)

For installation and a quick-start workflow, see the [Home page](../README.md). This guide expands on that material with more detailed settings, operating notes, and troubleshooting information.

## Recommended Practices

- Keep active project assets on a local drive whenever possible, with enough free space for the PSD, individual textures, and backups.
- Save the relevant documents before creating, switching, writing back, or recovering a project. For important work, keep your own backups as well.
- Brandy Texture Link 2D provides three handling modes for PSD documents with complex layer structures. Before importing, however, it is still recommended to back up the file and simplify or rasterize complex layers where practical. This can reduce import time and make the result more predictable.
- PSB files are supported as a compatibility option for import and export workflows. Very large or highly complex PSB documents may still fail, so simplify them first or convert them to PSD manually when possible.
- If an operation fails, check **Open Detailed Report** under **Settings & Reports** first. Follow the report before manually editing project JSON files or reconnecting Smart Objects.
- Do not manually clean the `jobs`, `backups`, or other internal project folders. Avoid having multiple Blender or Photoshop versions write to the same project at the same time.

## From Blender to Photoshop

### 1. Open Photoshop

Open the add-on panel. Under **Photoshop Path**, select `Photoshop.exe` from your Photoshop installation folder, then click **Open Photoshop**.

### 2. Choose a Blender Collection

Keep **Texture Source** set to **Blender Collection**. Put all flat rectangular Mesh objects you want to edit (referred to below as `2D sprites`) into one Blender collection, then select that collection under **Project Collection**.

When creating a project from a Blender collection, each valid 2D sprite should be a flat rectangular plane, have a unique name, have a valid active UV layer, and use one clearly identifiable primary texture.

If a texture is packed into the `.blend` file, unpack it before project creation and, when possible, keep it on a local drive.

A material may contain other nodes or supporting textures, but the add-on must be able to identify one primary texture for linking. If several textures are possible candidates and the primary texture cannot be determined, project creation stops and the reason is listed in the Detailed Report.

Blender's automatically generated numeric suffixes such as `.001` and `.002` are not treated as meaningful differences between texture item names. For example, `Arm` and `Arm.001` may be detected as duplicate names. Use clear, unique names before creating the project.

Object constraints, drivers, and animation are not transferred to Photoshop. The project records the base static layout at the time the project is created.

### 3. Create the Project

Under **Project Folder**, choose an empty local folder and click **Create Project**.

**Children**: When enabled, objects inside child collections of the selected collection are included in the project.

**Canvas Padding**: Adds transparent space outside the overall texture bounds in the Photoshop working document. This gives you more visible room for full-layout painting. It only expands the overall PSD canvas; it does not change the relative placement of the textures or increase the pixel dimensions of each individual texture.

**Use PSB**: When disabled, the add-on creates a PSD by default. If the canvas exceeds 30,000 pixels, it automatically switches to PSB.

### 4. Complete the Link

The add-on copies the textures used by the 2D sprites into the project folder and creates a new PSD document that reconstructs the Blender sprite arrangement in Photoshop. The source textures used before project creation are not modified.

The project collection, texture names, texture dimensions, and overall layout become part of the project's linking baseline. Changing them later may affect project recognition and linking. Make these changes before project creation, or create a new project afterward.

### 5. The Three Reserved Layer Groups

The PSD contains three reserved groups: **BTL2 Linked Content**, **BTL2 Merge Layers**, and **BTL2 Merged Layers**. Do not manually move, replace, or rename these groups, and do not create nested groups inside them. Keep the document structure in its default state.

**BTL2 Linked Content**: Contains the linked Smart Objects that correspond to the project textures. These Smart Objects reconstruct the Blender layout and provide the coordinate mapping used by the workflow.

**BTL2 Merge Layers**: Place new layers here when you want to use **Apply Multi-Texture Edits**. Keep the group opacity at 100%, with its blend mode set to `Normal` or `Pass Through`.

**BTL2 Merged Layers**: Layers processed by **Apply Multi-Texture Edits** are moved here automatically. This group is only an archive for working layers and does not replace project backups.

## Import a PSD Document

### 1. Choose a PSD Document

Make sure the add-on is installed as described on the Home page and Photoshop is open. In the add-on panel, change **Texture Source** to **Photoshop Document**, then choose the PSD document you want to import under **PSD Document**.

This workflow creates a project from the document's current layer state. It does not establish continuous synchronization with the original PSD.

### 2. Import Settings

Under **Project Folder**, choose an empty local folder. Set options such as **Transparent Pixel Padding** and **Use Alpha**, then click **Create Project**.

**Transparent Pixel Padding**: When cropping each layer, the add-on adds the specified number of transparent pixels around all four sides. This creates extra texture space for later edits or color bleed. The layer stays in the same position in the overall composition, but the resulting texture becomes larger and uses more storage.

**Ignore Hidden Layers**: When enabled, hidden layers and content inside hidden parent groups are not imported.

**Use Alpha**: When enabled, the texture's Alpha output is connected to material transparency and transparent display is enabled for the material. When disabled, the Alpha channel does not affect material display and the sprite is treated as opaque. This setting does not remove or alter the Alpha channel stored in the texture file.

**Premultiply Alpha**: When enabled, the texture is read as premultiplied Alpha. Standard PNG files and other straight-Alpha textures should normally leave this disabled. Enable it only when the texture itself already uses premultiplied Alpha; otherwise, transparent edges may show incorrect color or brightness.

**Import Scale**: Controls the overall conversion ratio from Photoshop pixel coordinates to Blender units. It does not scale, resample, or change the pixel dimensions of the texture files.

**Sprite Z Spacing**: Adds a fixed Z-axis offset between neighboring sprites to reduce flickering and sorting problems when planes would otherwise overlap exactly.

### 3. Import Logic

During PSD import, the add-on creates a 2D sprite for each successfully imported layer or asset unit and connects the resulting texture to its material.

For PSD files with complex structures, simplify the document when possible before importing. If the document contains many linked Smart Objects, layer styles, or similar content, the add-on provides **Simplify Complex Layers**, **Skip Complex Layers**, and **Preserve Usable Content**. Complex effects may still lose information during import.

**Simplify Complex Layers**: Imports standard layers directly. Complex layers or groups are flattened into single images where possible. Content that cannot be handled safely is skipped.

**Skip Complex Layers**: Imports standard layers directly while skipping complex layers and complex groups themselves. Standard child layers inside complex groups can still be imported individually when they can be handled independently.

**Preserve Usable Content**: Imports standard layers directly and first attempts the same simplification used by **Simplify Complex Layers**. If content still cannot be handled safely, the add-on reads the usable layer information and preserves its name and position with transparent placeholders where necessary.

For complex content, the import preserves the current visible result and layout rather than the full editability of text, masks, adjustment layers, layer styles, clipping relationships, or Smart Object internals.

You can also control import behavior with name prefixes:

`[ignore]`: Add this to the beginning of a layer or group name to skip that content.

`[asset]`: Add this to the beginning of a group name to process the group as one asset unit.

### 4. Complete the Import

After the import and project creation complete successfully, all generated 2D sprites are placed in a Blender collection. Click **Open Existing Project** to continue working with the project, then edit textures in Photoshop and use Auto Reload as usual.

Textures generated from PSD import are saved as PNG files.

## About the Project Folder

The project folder is the working asset directory used for the ongoing Blender–Photoshop link. It is not a temporary cache.

After the project is created successfully, Blender reconnects to the texture copies stored in the project folder. From that point onward, direct editing and multi-texture write-back modify those project copies. The external source textures used before project creation remain unchanged.

Do not move, rename, or replace individual PSD files, texture files, JSON files, or internal folders inside the project. If you need to move the project, move the entire project folder together.

If you copy a project and want to continue editing the copy as a separate project, select the copied project in the add-on and use **Make Project Copy Independent** when the option appears. Continue working only after the independence process completes.

`textures` folder: The independent texture files currently used and modified by the project.

`photoshop` folder: The working PSD document created by the add-on.

`layout` folder: Texture layout and linking information.

`jobs` folder: Task records used between Blender and Photoshop.

`backups` folder: Backups used for write-back, revert, and recovery operations.

`source` folder: Stores a copy of the source document when a project is created from a PSD document.

`brandy_texture_link_2d_project.json`: The project entry file.

## Single-Texture Editing and Full-Layout Painting

### Single-Texture Editing

When you only need to edit one texture, open its Smart Object directly from the PSD created by the add-on. You can also select a 2D sprite in Blender and click **Edit Single Texture** to open the corresponding texture file.

In `single-texture editing`, save the `Smart Object` or `individual texture file`, not only the main PSD document.

While editing, keep the project texture at its original file path and pixel dimensions. Changing either may cause reload operations to fail.

### Full-Layout Painting

1. Paint and rename the layers

Suppose you want to paint one pattern across three textures named `Leg_Thigh`, `Leg_Shin`, and `Leg_Foot`.

Create a new layer in the main PSD and paint the artwork. Press `Ctrl+J` twice to create two copies, giving you three layers with identical content.

Rename the three layers to `Leg_Thigh`, `Leg_Shin`, and `Leg_Foot`, place all three directly inside **BTL2 Merge Layers**, then save the main PSD document.

You can prepare more layers in the same way and process them in one operation.

2. Apply Multi-Texture Edits

Click **Apply Multi-Texture Edits**. The artwork is applied to the three name-matched project textures. Pixels outside each target texture canvas are clipped.

Layers whose names do not match a project texture are not applied.

Do not modify project files in Photoshop or another program while the apply task is running.

3. Revert Last Applied Edit

If you do not want to keep the result, use the newly available **Revert Last Applied Edit** action to restore the previous state safely.

**Revert Last Applied Edit** only applies to **Apply Multi-Texture Edits**. It is not a general history system and can only restore the PSD document, linking information, and project textures modified by the most recent successful apply operation.

After **Apply Multi-Texture Edits**, do not make further changes to the related PSD or textures if you intend to use the revert action. If those files have changed, the add-on will refuse the revert for safety.

## Additional Details

### Auto Reload and Manual Reload

When **Auto Reload** is enabled (the button is highlighted), saving in Photoshop with `Ctrl+S` can trigger Blender to reload the updated texture. Auto Reload is not instantaneous; expect a delay of around 3 seconds, depending on project size and hardware.

Auto Reload periodically checks the project texture files. Once a file has finished writing and its pixel dimensions are unchanged, Blender reloads the image.

You can disable **Auto Reload** and use **Reload Textures** manually after editing and saving.

If the pixel dimensions of a texture change, the add-on does not automatically resize the Blender sprite. Reloading stops or skips that texture instead. If you need to change texture dimensions, creating a new project is recommended.

### Project Entry Behavior

If the selected **Project Folder** already contains a project created by the add-on, **Create Project** automatically changes to **Open Existing Project**.

When a project is already open and you change **Project Folder**, **Open Existing Project** changes to **Create Project** if the new folder is empty. If the new path contains another Brandy Texture Link 2D project, a **Switch Project** action appears below the create/open control.

During a switch, the original project remains active until the candidate project passes validation. A failed switch does not commit an incomplete candidate project.

### Settings & Reports

If something goes wrong during normal use, **Open Detailed Report** contains failure reasons, execution details for some tasks, and recommended follow-up actions.

**Fast Check Duration (s)** can normally remain at its default value. After this period, Photoshop continues running and Blender continues checking for the result, but at a lower polling frequency to reduce overhead. This setting is not a task timeout and does not terminate Photoshop.

If automatic Photoshop control is unavailable, you can change **Photoshop Run Mode** from **Windows COM** to **Manual Script**. In **Manual Script** mode, the operation generates the corresponding script file, which you then run manually in Photoshop through `File` > `Scripts` > `Browse` by selecting `run_brandy_texture_link_2d_job.jsx` from the task folder. This mode is intended for temporary fallback or special environments rather than everyday use.

The interface is available in Chinese and English. Change the default language with **Interface Language**, then restart the add-on or Blender to refresh the panels.

### Safety and Recovery

The add-on also includes several context-sensitive actions for recovery and safety. They only appear when relevant. Follow the on-screen instructions when one of these actions is shown.

**Open Task Records Folder**: Opens the folder containing Manual Script files, task JSON files, results, and diagnostic data.

**View Analysis Summary**: Shows the count of standard and complex content detected in the PSD.

**View Full Analysis**: Shows the handling decision and reason for each layer.

**Recover Interrupted Texture Update**: Restores interrupted texture write-back from trusted backups.

**Recover Incomplete Project**: Continues a project whose Photoshop working document was created but whose Blender project binding was not fully committed.

**Recover Project-Copy Transaction**: Handles an interrupted **Make Project Copy Independent** transaction.

**Clear Stale Project Lock**: Releases an abnormal or stale project lock only after you have confirmed that all write activity has stopped.

**Confirm Task Ended**: Clears Blender's waiting state only. It does not terminate Photoshop.

## Additional Tools

### Switch Texture Format

This tool searches the same texture directory for an existing file with the same base name in PNG, TGA, JPG, or JPEG format, then switches the project to that file. It is intended for batch texture-format switching.

It does not convert image formats. Shared images, unsaved images, and packed images are not modified.

### Convenience Tools

**Copy Shader Settings to Selected Objects**: Copies shader input values from the last selected object to the other selected objects. Only unlinked inputs with matching names are copied.

**Merge Duplicate Materials**: Safely merges duplicate materials generated by the add-on when those materials have not been modified.

**Restore Imported Material**: Restores the imported material and texture link for selected objects based on the import record.

**Isolate Selection**: Temporarily isolates selected objects in the current 3D View. You can move or edit them normally while isolated. Click the action again to restore the previous visibility state.

### Static PhotoshopToSpine JSON

This is limited compatibility support and is not a replacement for the PhotoshopToSpine plug-in.

It can import static sprite images and JSON exported by PhotoshopToSpine. The export function does not export texture files; it only records the current Blender layout in a compatible JSON file.

Special thanks to EsotericSoftware, the author of PhotoshopToSpine. The plug-in saved me a great deal of time during my earlier 2D character work.

### Spine2D Region Attachment Import

This feature is provided as limited compatibility support and does not create a live connection to Spine 2D itself.

It only imports static Region Attachments from Spine JSON and creates rectangular sprites from the Setup Pose. Spine animation, Mesh Attachments, constraints, and animatable Blender rigs are not supported.
