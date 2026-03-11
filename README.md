# Search & Replace Block Visualizer

A web-based developer tool that elegantly parses and visualizes AI-generated `<<<<<<< SEARCH` ... `>>>>>>> REPLACE` modification blocks using the powerful [Monaco Editor](https://microsoft.github.io/monaco-editor/).

## 🌟 Features

- **Visual Diff Editor:** Seamlessly compares the original code with the AI's proposed replacement using a side-by-side Monaco Diff Editor.
- **Robust Parsing:** Accurately extracts file paths and modification blocks from Markdown-formatted AI responses.
- **Responsive Layout:** Gracefully handles window resizing and ensures accurate line wrapping (word wrap) even in complex CSS layouts.
- **Zero Build Step:** The core functionality is contained within `diff.html`, ready to run in any modern web browser without complex build pipelines.

## 🚀 Getting Started

1. Clone the repository:
   ```bash
   git clone https://github.com/one-pyy/Search-Replace-Block.git
   ```
2. Open `diff.html` directly in your web browser.
3. Paste the original code and the AI's Search & Replace block into the tool to instantly visualize the changes!

## 💡 How it works

AI coding assistants (like Claude, GPT-4, or Gemini) often suggest code modifications using a standard Search & Replace format:

```javascript
<<<<<<< SEARCH
function oldCode() {
  console.log("old");
}
=======
function newCode() {
  console.log("new");
}
>>>>>>> REPLACE
```

This tool takes that raw text output and renders it into a professional, IDE-like diff view, making it incredibly easy to review exactly what the AI intends to change before applying it to your actual codebase.

## 🛠️ Built With

- HTML / CSS / JavaScript
- [Monaco Editor](https://microsoft.github.io/monaco-editor/) - The code editor that powers VS Code.

## 📄 License

This project is open-source and available for any personal or commercial use.
