# Copilot Instructions

This document provides guidance for AI assistants to effectively contribute to the `azure-devops-mermaid-viewer` codebase.

## Architecture

This project is an Azure DevOps extension that renders Mermaid diagrams within the Azure Repos UI.

- **Extension Manifest**: `vss-extension.json` is the manifest file. It defines the extension's contributions, such as the custom preview tab for files.
- **Entry Point**: The extension's client-side script starts in `src/index.ts`. This file initializes the Azure DevOps SDK and registers the `mermaid_viewer` object, which is responsible for rendering the content.
- **Core Logic**: The main rendering logic resides in `src/viewer.ts`. The `MermaidViewer` class handles the transformation of text into a visual diagram.
- **Rendering Path**:
  1.  The `renderContent` method in `MermaidViewer` is the core of the rendering logic.
  2.  It detects whether the content is a plain Mermaid file (`.mmd`) or a Markdown file (`.md`) containing Mermaid syntax.
  3.  For Markdown, it uses the `commonmark` library to parse the document and find `mermaid` code blocks. It specifically handles a custom `:::mermaid` block syntax by converting it to the standard ` ```mermaid` before processing.
  4.  For both cases, the `mermaid` library is used to render the diagram definition into an SVG, which is then injected into the DOM.

## Developer Workflow

The development process involves building the extension, packaging it, and deploying it to the Azure DevOps Marketplace for testing.

### Initial Setup

1.  Install dependencies:
    ```bash
    npm install
    ```
2.  Install the TFX CLI globally, which is used for packaging the extension:
    ```bash
    npm install -g tfx-cli
    ```

### Development and Debugging

1.  **Build and Package for Dev**: Create a development version of the extension using the `dev.json` configuration. This typically includes settings for local development, like pointing to a local dev server.
    ```bash
    npx tfx-cli extension create --rev-version --overrides-file configs/dev.json
    ```
2.  **Deploy to Marketplace**: Upload the generated `.vsix` file to your personal publisher on the [Visual Studio Marketplace](https://marketplace.visualstudio.com/manage/).
3.  **Run Local Server**: Start the webpack dev server to serve the extension's content locally. This enables live reloading.
    ```bash
    npx webpack-dev-server --mode development
    ```
4.  **Debug**: With the extension installed in an Azure DevOps organization and the local server running, you can debug the extension's behavior directly within Azure DevOps.

### Production Build

1.  **Build for Production**:
    ```bash
    npm run build
    ```
2.  **Package for Production**: Create the release version of the extension using the `release.json` configuration.
    ```bash
    npx tfx-cli extension create --rev-version --env mode=production --overrides-file configs/release.json
    ```

## Key Conventions

- **Mermaid in Markdown**: To embed a Mermaid diagram within a Markdown file (`.md`), use a fenced code block with the `:::mermaid` syntax. This is a project-specific convention to avoid conflicts with standard Markdown. The `viewer.ts` file contains the logic to convert this to a standard ` ```mermaid` block before rendering.

  Example from `src/test/markdown-test.md`:

  ```markdown
  :::mermaid
  graph TD;
  A-->B;
  :::
  ```

- **File Handling**: The extension contributes a viewer tab for files with `.mmd` and `.md` extensions in Azure Repos (both in the "Files" hub and "Pull Request" hub). This is configured in `vss-extension.json`.
