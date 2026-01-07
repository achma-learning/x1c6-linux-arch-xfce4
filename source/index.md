# Arch Linux Configuration and Documentation for Lenovo ThinkPad X1 Carbon 6th Gen

This document serves as the main overview for the documentation related to setting up, optimizing, and customizing an Arch Linux system on a Lenovo ThinkPad X1 Carbon 6th Gen (20KG).

## Directory Overview

This collection of documents aims to provide a comprehensive reference for maintaining a stable, secure, and performant Arch Linux installation tailored for the X1C6 hardware, with a focus on specific customizations.

## Key Documentation Files:

*   **`Home (This file):`** Provides an overview and index to the other documentation within this project.
*   **`Arch Linux Installation:`** A detailed, phase-by-phase guide for the initial core system setup.
*   **`X1 Carbon 6th Gen Optimization:`** Covers essential package installation, service configuration, TLP power management, and hardware-specific fixes.
*   **`XFCE4 & macOS-like Customization:`** A guide to transform the XFCE4 desktop environment into a macOS-like appearance.
*   **`System Information:`** Provides an overview of the system's hardware and software configuration.
*   **`Old Setup Guide (Legacy):`** The original `arch_x1c6_setup.md` guide, kept for historical reference. Refer to the updated installation and optimization guides for current information.
*   **`2026-01-04_09-22.png`:** A screenshot showcasing the customized desktop environment, providing a visual reference for the intended aesthetic.

## Usage:

These documents are intended to be used as a personal knowledge base and actionable guides for:

*   **New Installations:** Following the [Arch Linux Installation Guide](arch_installation.md) for a fresh Arch Linux deployment on the specified hardware.
*   **System Optimization:** Referencing the [X1 Carbon 6th Gen Optimization Guide](x1c6_optimization.md) for fine-tuning power management, performance, and hardware-specific issues.
*   **Desktop Customization:** Utilizing the [XFCE4 & macOS-like Customization Guide](xfce4_macos_customization.md) to achieve a specific aesthetic for the desktop environment.
*   **System Information:** Consulting the [System Information page](system_info.md) for an overview of the system's hardware and software configuration.
*   **Troubleshooting:** Using the documented configurations and fixes as a reference point for diagnosing issues.

## Interactive Wiki-style Documentation (MkDocs)

A wiki-style website has been generated using `mkdocs` to provide a more interactive and browsable format for the core documentation. The source for this website (Markdown files and configuration) is located in the `source/` directory within the project root.

### Setup and Usage:

1.  **Install MkDocs:** If not already installed, `mkdocs` can be installed via `pacman`:
    ```bash
    sudo pacman -S mkdocs
    ```
2.  **Navigate to the documentation project root:**
    ```bash
    cd x1c6-linux-arch-xfce4/
    ```
3.  **Preview the documentation locally:**
    To start a local development server and view the documentation in your web browser:
    ```bash
    mkdocs serve
    ```
    Open your web browser and go to `http://127.0.0.1:8000` (or the address indicated in the terminal output). The site will automatically refresh as you make changes to the source markdown files in the `source/` directory. Press `Ctrl+C` to stop the server.

4.  **Build the static site for deployment:**
    To generate a static HTML website that can be hosted on any web server (e.g., GitHub Pages), use:
    ```bash
    mkdocs build
    ```
    This will create (or update) the `docs/` directory containing all the static files for deployment.

## Important Note on Local Changes and Synchronization

All modifications and file operations performed by the agent are applied directly to the local project folder. To reflect these changes on your remote GitHub repository and subsequently on your GitHub Pages website, you *must* manually commit and push these changes using GitHub Desktop (or your preferred Git client). Remember to follow the specific instructions provided for publishing the MkDocs site, especially after major updates to content or configuration.

---

**Note:** The detailed guides (originally `installation_and_setup_legacy.md`) contain commands and instructions that should be executed with care. Always understand the implications of a command before running it. Refer to the specific documents for details.
