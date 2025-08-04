# QDA-VR: Qualitative Data Analysis in Virtual Reality

A cutting-edge qualitative data analysis platform that enables researchers to analyze data in both traditional 2D interfaces and immersive VR/AR environments with real-time collaboration capabilities.

## 🌟 Features

### Core Analysis Tools

- **Node-based Interface**: Organize your research data using an intuitive node system
- **Rich Text Editor**: Powered by TipTap for comprehensive text analysis and annotation
- **Qualitative Coding**: Create, manage, and apply codes with color-coding and grouping
- **Multi-format Support**: Import and analyze PDFs, DOCX files, images, and plain text

### Immersive Experience

- **VR/AR Support**: Conduct analysis in virtual and augmented reality environments
- **3D Room Environment**: Immersive 3D workspace for spatial data organization
- **Cross-platform**: Seamlessly switch between 2D and VR/AR modes

### Collaboration

- **Real-time Multiplayer**: Collaborate with team members in real-time using Liveblocks
- **Live Cursors**: See where other researchers are working
- **Shared Workspaces**: Work together on the same analysis project simultaneously

### File Management

- **Drag & Drop**: Easy file import with drag-and-drop functionality
- **Export Options**: Export your analysis and findings
- **Multiple Formats**: Support for various document formats including Office files

## 🚀 Getting Started

### Prerequisites

- Node.js 18+
- npm, yarn, pnpm, or bun

### Installation

1. Clone the repository:

```bash
git clone <repository-url>
cd QDA-VR
```

2. Install dependencies:

```bash
npm install
# or
yarn install
# or
pnpm install
```

3. Run the development server:

```bash
npm run dev
# or
yarn dev
# or
pnpm dev
# or
bun dev
```

4. Open [http://localhost:3000](http://localhost:3000) to access the application

### VR Development

For VR functionality, you may need to run with HTTPS:

```bash
npm run dev:https
```

## 🛠️ Tech Stack

- **Framework**: Next.js 14 with React 18
- **Language**: TypeScript
- **3D/VR**: Three.js, React Three Fiber, React Three XR
- **Real-time Collaboration**: Liveblocks
- **Text Editor**: TipTap
- **Styling**: Tailwind CSS
- **UI Components**: Radix UI
- **File Processing**: Mammoth (DOCX), React-PDF, XLSX
- **State Management**: Jotai, Zustand-like patterns

## 📱 Usage

### Single User Mode

- Access the main interface at the root URL
- Create text and file nodes
- Apply qualitative codes to your data
- Switch to VR mode for immersive analysis

### Multiplayer Mode

- Create or join a room using `/[room-id]` URLs
- Collaborate in real-time with other researchers
- See live cursors and changes from team members

### VR/AR Analysis

- Toggle between 2D, 3D, VR, and AR modes
- Use the spatial environment to organize complex data relationships
- Experience your qualitative analysis in an immersive setting

## 🗂️ Project Structure

```
QDA-VR/
├── app/                    # Next.js app router
├── components/
│   ├── canvas/            # Main canvas and node system
│   │   ├── 3d/           # VR/AR components
│   │   ├── dialogs/      # File, code, and settings dialogs
│   │   ├── multiplayer/  # Real-time collaboration
│   │   ├── node/         # Text and file node components
│   │   └── text-editor/  # Rich text editing interface
│   └── ui/               # Reusable UI components
├── hooks/                # Custom React hooks
└── lib/                  # Utility functions
```

## 🤝 Contributing

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

## 📄 License

This project is licensed under the MIT License - see the LICENSE file for details.

## 🆘 Support

For support and questions:

- Open an issue on GitHub
- Check the documentation
- Review existing issues and discussions

## 🔮 Future Roadmap

- Enhanced VR interactions and gestures
- Advanced analytics and visualization
- Integration with popular qualitative analysis tools
- Mobile VR support
- Advanced export formats and reporting
