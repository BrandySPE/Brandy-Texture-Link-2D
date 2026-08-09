# Brandy Texture Link 2D

[User Guide](docs/UserGuide.md) · [Support](docs/SUPPORT.md) · [Maintenance](docs/Maintenance.md) · [Change Log](docs/ChangeLog.md) · [中文主页](README_zh-CN.md)

**Brandy Texture Link 2D is a Blender add-on built for 2D game and animation workflows. It opens textures in Photoshop, reloads saved texture changes in Blender, and can create Blender assets from the layer layout of PSD documents.**

If you work with 2D characters or scenes and need to rebuild a Blender texture layout in Photoshop, import the layer layout of a PSD document into Blender, or move back and forth while making texture edits, the add-on reduces the amount of manual relinking and setup required.

For a quick start, follow the workflows below. For more detailed use, watch the tutorials or see the [User Guide](docs/UserGuide.md).

The add-on includes safeguards for file validation, project backups, project recovery, and related operations. It has also completed a 7 × 5 test matrix that includes Blender 5.2.0 LTS. See [Maintenance](docs/Maintenance.md) for test details and usage notes.

The interface is available in Chinese and English. You can change the default language with **Interface Language** under **Settings & Reports**. Restart the add-on or Blender afterward to refresh the panels.

If you have questions or need to report an issue, see [Support](docs/Support.md) or contact **brandyspe2026@gmail.com**. I will reply as soon as I can after receiving your email.

## Install and Create a Project

### Install the Add-on

1. In Blender, open **Edit** > **Preferences** > **Get Extensions** > the menu in the upper-right corner > **Install from Disk**, then select the complete Brandy Texture Link 2D.zip package. Do not extract the ZIP file first.

2. Make sure Brandy Texture Link 2D is enabled under **Preferences** > **Add-ons**.

3. Return to the 3D View, press `N` to open the sidebar, then open the **Brandy** tab.

### From Blender to Photoshop

1. Open the add-on panel. Under **Photoshop Path**, select `Photoshop.exe` from your Photoshop installation folder, then click **Open Photoshop**.

2. Keep **Texture Source** set to **Blender Collection**. Put all flat rectangular Mesh objects you want to edit (referred to below as `2D sprites`) into one Blender collection, then choose that collection under **Project Collection**.

3. Under **Project Folder**, choose an empty local folder and click **Create Project**. If the selected path already contains a project created by the add-on, **Create Project** automatically changes to **Open Existing Project**.

4. The add-on copies the textures used by the 2D sprites into the project folder and creates a new PSD document. If the canvas exceeds 30,000 pixels, it automatically creates a PSB instead. Photoshop reconstructs the sprite arrangement from Blender. The source textures used before project creation are not modified.

5. The PSD contains three reserved groups: **BTL2 Linked Content**, **BTL2 Merge Layers**, and **BTL2 Merged Layers**. Do not manually move, replace, or rename these groups. Keep the document structure in its default state.

## Texture Editing and Auto Reload

### Mode 1: Edit a Single Texture

When you only need to edit one texture, open its Smart Object directly from the PSD created by the add-on. You can also select a 2D sprite in Blender and click **Edit Single Texture** to open the corresponding texture file.

When **Auto Reload** is enabled (the button is highlighted), saving in Photoshop with `Ctrl+S` can trigger Blender to reload the updated texture. Auto Reload is not instantaneous; expect a delay of around 3 seconds, depending on project size and hardware.

You can also disable **Auto Reload** and use **Reload Textures** after editing and saving.

In `single-texture editing`, save the `Smart Object` or `individual texture file`, not only the main PSD document.

### Mode 2: Paint Across Multiple Textures

Suppose you want to paint one pattern across three textures named `Leg_Thigh`, `Leg_Shin`, and `Leg_Foot`.

Create a new layer in the main PSD and paint the artwork. Press `Ctrl+J` twice to create two copies, giving you three layers with identical content.

Rename the three layers to `Leg_Thigh`, `Leg_Shin`, and `Leg_Foot`, place all three directly inside **BTL2 Merge Layers**, then save the main PSD document.

Click **Apply Multi-Texture Edits** in Blender. The artwork is applied to the three matching textures. Pixels outside each target texture canvas are clipped.

If you do not want to keep the result, use the newly available **Revert Last Applied Edit** action to restore the previous state safely.

## Import a PSD Document

1. Make sure the add-on is installed as described above and Photoshop is open. In the add-on panel, change **Texture Source** to **Photoshop Document**, then choose the PSD document you want to import under **PSD Document**.

2. Under **Project Folder**, choose an empty local folder. Set options such as **Transparent Pixel Padding** and **Use Alpha**, then click **Create Project**.

3. During import, the add-on creates a 2D sprite for each successfully imported layer or asset unit and connects the resulting texture to its material.

For PSD files with complex structures, simplify the document when possible before importing. If the document contains many linked Smart Objects, layer styles, or similar content, the add-on provides three handling modes: **Simplify Complex Layers**, **Skip Complex Layers**, and **Preserve Usable Content**. Complex effects may still lose information during import.

PSB files are supported as a compatibility option, but very large or highly complex documents may not import completely.

4. After the import and project creation complete successfully, all generated 2D sprites are placed in a Blender collection. Click **Open Existing Project** to continue working with the project, then edit textures in Photoshop and use Auto Reload as usual.

## Demos and Tutorials

The current tutorials were made for an earlier version of the add-on. The core workflow is still applicable, but some button names have changed. New tutorials will replace them when ready. Until then, use the written [User Guide](docs/UserGuide.md) as the primary reference.

- [Chinese demo and tutorial](https://www.youtube.com/watch?v=-xTnPTlHHwc)

- [English demo and tutorial](https://www.youtube.com/watch?v=seKdFcPqHf4)

## Get Brandy Texture Link 2D

This GitHub repository is the product home page and does not include an installable add-on package. Use the complete ZIP package provided through a sales channel.

Superhive: https://superhivemarket.com/products/brandy-2d-link

itch.io: https://brandyspe.itch.io/brandy-2d-link

Free lightweight texture-editing add-on: [Brandy Texture Link Lite](https://github.com/BrandySPE/Brandy-Texture-Link-Lite)

## Other Information

**Brandy Texture Link 2D** was previously released as **Brandy 2D Link**. Version **1.1.0** is the first public release under the new name.

The rename accompanies a major overhaul of the core workflow, including `PSD import` and a simplified, rebuilt interface.

Users who previously received the older version have been provided with the new version.

## Support Information

### Supported Environment

- Windows x64
- Blender 4.2.0–5.2.0, including 5.2.0
- Adobe Photoshop desktop CC2017–2026, including 2026

### Supported Asset Types

- PNG, JPG, JPEG, and TGA textures.
- Flat characters or layered artwork in PSD documents.
- Rectangular image-plane meshes in Blender.
- 2D characters or scenes based on a static layout.

## License and Independent Product Notice

The add-on package is distributed under GPL-3.0-or-later and includes the corresponding source code.

Brandy Texture Link 2D is an independent product and is not affiliated with or endorsed by Adobe, the Blender Foundation, or their related organizations. See [NOTICE.md](NOTICE.md) for trademark information.
