# GEMINI.md: Arch Linux Configuration and Documentation for Lenovo ThinkPad X1 Carbon 6th Gen

This directory serves as a centralized repository for documentation and configuration guides related to setting up, optimizing, and customizing an Arch Linux system on a Lenovo ThinkPad X1 Carbon 6th Gen (20KG). The files within detail various aspects, from core system installation and power management to desktop environment theming and productivity enhancements.

## Directory Overview

This collection of documents aims to provide a comprehensive reference for maintaining a stable, secure, and performant Arch Linux installation tailored for the X1C6 hardware, with a focus on specific customizations.

## Key Files:

*   **`GEMINI.md` (This file):** Provides an overview and index to the other documentation within this directory.
*   **`arch_x1c6_setup.md`:** A detailed, phase-by-phase guide for the initial core system setup and optimization of Arch Linux on the X1 Carbon 6th Gen. This covers BIOS settings, essential package installation, service configuration, TLP power management, and hardware-specific fixes.
*   **`macos.txt`:** A step-by-step guide outlining the commands and procedures to transform the XFCE4 desktop environment into a macOS-like appearance, including themes, icons, cursors, global menu, Plank dock, and various utilities.
*   **`packages.txt`:** A comprehensive list of all installed packages on the system. This acts as a record of the software environment and can be useful for replication or auditing.
*   **`wiki.txt`:** An alternative or supplementary guide offering additional optimization and productivity tweaks for the X1 Carbon 6th Gen on Arch Linux. It covers similar ground to `arch_x1c6_setup.md` but may offer different or expanded perspectives on certain configurations.
*   **`2026-01-04_09-22.png`:** Likely a screenshot showcasing the customized desktop environment, providing a visual reference for the intended aesthetic.

## Usage:

These documents are intended to be used as a personal knowledge base and actionable guides for:

*   **New Installations:** Following the `arch_x1c6_setup.md` for a fresh Arch Linux deployment on the specified hardware.
*   **System Optimization:** Referencing `arch_x1c6_setup.md` and `wiki.txt` for fine-tuning power management, performance, and hardware-specific issues.
*   **Desktop Customization:** Utilizing `macos.txt` to achieve a specific aesthetic for the XFCE4 desktop environment.
*   **Software Reference:** Consulting `packages.txt` to review the installed software ecosystem.
*   **Troubleshooting:** Using the documented configurations and fixes as a reference point for diagnosing issues.

## Interactive Wiki-style Documentation (MkDocs)

A wiki-style website has been generated using `mkdocs` to provide a more interactive and browsable format for the core documentation. The source for this website is located in the `arch_docs/` directory.

### Setup and Usage:

1.  **Install MkDocs:** If not already installed, `mkdocs` can be installed via `pacman`:
    ```bash
    sudo pacman -S mkdocs
    ```
2.  **Navigate to the documentation project:**
    ```bash
    cd arch_docs
    ```
3.  **Preview the documentation locally:**
    To start a local development server and view the documentation in your web browser:
    ```bash
    mkdocs serve
    ```
    Open your web browser and go to `http://127.0.0.1:8000` (or the address indicated in the terminal output). The site will automatically refresh as you make changes to the source markdown files. Press `Ctrl+C` to stop the server.

4.  **Build the static site for deployment:**
    To generate a static HTML website that can be hosted on any web server (e.g., GitHub Pages), use:
    ```bash
    mkdocs build
    ```
    This will create a `site/` directory containing all the static files.

## Important Note on Local Changes and Synchronization

All modifications and file operations performed by the agent are applied directly to the local project folder. To reflect these changes on your remote GitHub repository and subsequently on your GitHub Pages website, you *must* manually commit and push these changes using GitHub Desktop (or your preferred Git client). Remember to follow the specific instructions provided for publishing the MkDocs site, especially after major updates to content or configuration.

---

**Note:** The detailed guides (originally `gemini.md` and `wiki.txt`) contain commands and instructions that should be executed with care. Always understand the implications of a command before running it.
