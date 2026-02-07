```
   ______                __                  __   ______      _     __
  / ____/________  ____  / /____  ____  ____/ /  / ____/_  __(_)___/ /__  _____
 / /_  / ___/ __ \/ __ \/ __/ _ \/ __ \/ __  /  / / __/ / / / / __  / _ \/ ___/
/ __/ / /  / /_/ / / / / /_/  __/ / / / /_/ /  / /_/ / /_/ / / /_/ /  __(__  )
\__/  /_/   \____/_/ /_/\__/\___/_/ /_/\__,_/   \____/\__,_/_/\__,_/\___/____/

		   ╔═══════════════════════════════════════════════════════╗
		   ║     Frontend Best Practices & Essential Configs       ║
		   ╚═══════════════════════════════════════════════════════╝
```

# Frontend Configs & Guides

> Front-end development best practices, setup guides, and essential configurations for a streamlined workflow.

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Contributions Welcome](https://img.shields.io/badge/contributions-welcome-brightgreen.svg?style=flat)](https://github.com/cardacci/frontend-configs-and-guides/issues)

---

## Table of Contents

- [Overview](#overview)
- [Project Structure](#project-structure)
- [Content](#content)
  - [AI Integration](#ai-integration)
  - [Code Standards](#code-standards)
  - [Languages](#languages)
  - [Visual Studio Code](#visual-studio-code)
- [Contributing](#contributing)
- [License](#license)
- [Author](#author)

---

## Overview

This repository serves as a **comprehensive reference** for frontend developers, providing:

- **Language Guides** - In-depth best practices for JavaScript and TypeScript
- **Framework Standards** - Component structure guidelines for React (JSX & TSX)
- **IDE Configurations** - Visual Studio Code tips and productivity enhancements
- **Clean Code Principles** - Patterns and practices for maintainable code

Whether you're a beginner looking to learn best practices or an experienced developer seeking a quick reference, this repository has something for you.

---

## Project Structure

```
frontend-configs-and-guides/
│
├── 📂 AI/                            # AI agent integration guides
│   └── 📂 Claude/                    # Claude-specific setup
│       └── CLAUDE.md                 # How to use .claude folder with standards
│
├── 📂 Code standards/               # Framework-specific standards
│   └── 📂 React/                    # React component guidelines
│       ├── JSX_components.md        # JavaScript React components
│       └── TSX_components.md        # TypeScript React components
│
├── 📂 Languages/                    # Programming language guides
│   ├── 📂 JavaScript/               # JavaScript best practices (7 guides)
│   │   ├── 01 - Variables and constants.md
│   │   ├── 02 - Functions.md
│   │   ├── 03 - Arrays and objects.md
│   │   ├── 04 - Async programming.md
│   │   ├── 05 - Error handling.md
│   │   ├── 06 - Clean code principles.md
│   │   └── 07 - ES6+ features.md
│   │
│   └── 📂 TypeScript/               # TypeScript fundamentals (9 guides)
│       ├── 01 - Configuration file.md
│       ├── 02 - Primitives and built-in types.md
│       ├── 03 - Custom types with interfaces.md
│       ├── 04 - Functions & Generics.md
│       ├── 05 - keyof operator.md
│       ├── 06 - Indexed access types.md
│       ├── 07 - Record utility type.md
│       ├── 08 - Decorators.md
│       └── 09 - Modules.md
│
├── 📂 Visual Studio Code/           # IDE configuration guides
│   ├── File nesting.md              # Organize project files visually
│   ├── Key bindings.md              # Custom keyboard shortcuts
│   └── Workspace indentation.md     # Consistent code formatting
│
├── 📂 assets/                       # Static resources
│   └── ...
│
├── LICENSE                          # MIT License
└── README.md                        # This file
```

---

## Content

### AI Integration

Guides for integrating coding standards with AI agents so they generate code that
follows your project conventions automatically:

- **[Claude — `CLAUDE.md` Setup](AI/Claude/CLAUDE.md)** — How to configure a `.claude`
  directory with a `CLAUDE.md` index file so that Claude reads and applies your React
  component standards (JSX/TSX) when generating code.

---

### Code Standards

#### React Component Guidelines

Comprehensive structure guides for React components:

- **`JSX_components.md`** - JavaScript functional components with:
  - Recommended section ordering (imports, constants, state, refs, etc.)
  - Alphabetical ordering rules
  - Naming conventions (`handle*`, `is*`, `on*`, `render*`, `get*`)
  - PropTypes patterns
  - Real-world examples (basic and complex components)

- **`TSX_components.md`** - TypeScript functional components with:
  - Type definitions and interfaces
  - Props typing patterns
  - Same structural guidelines as JSX with TypeScript specifics

---

### Languages

#### JavaScript (7 Guides)

A progressive learning path covering essential JavaScript concepts:

| #   | Topic                       | Description                                            |
| --- | --------------------------- | ------------------------------------------------------ |
| 01  | **Variables and Constants** | `const` vs `let`, naming conventions, UPPER_SNAKE_CASE |
| 02  | **Functions**               | Arrow functions, parameters, pure functions            |
| 03  | **Arrays and Objects**      | Destructuring, spread operator, array methods          |
| 04  | **Async Programming**       | Promises, async/await, error handling patterns         |
| 05  | **Error Handling**          | try/catch, custom errors, graceful degradation         |
| 06  | **Clean Code Principles**   | Readability, DRY, single responsibility                |
| 07  | **ES6+ Features**           | Modern JavaScript syntax and features                  |

#### TypeScript (9 Guides)

From configuration to advanced types:

| #   | Topic                             | Description                     |
| --- | --------------------------------- | ------------------------------- |
| 01  | **Configuration File**            | tsconfig.json setup and options |
| 02  | **Primitives and Built-in Types** | Basic TypeScript types          |
| 03  | **Custom Types with Interfaces**  | Defining object shapes          |
| 04  | **Functions & Generics**          | Type-safe functions             |
| 05  | **keyof Operator**                | Type-level key extraction       |
| 06  | **Indexed Access Types**          | Dynamic type lookup             |
| 07  | **Record Utility Type**           | Object type mapping             |
| 08  | **Decorators**                    | Class and method decoration     |
| 09  | **Modules**                       | Import/export patterns          |

---

### Visual Studio Code

Productivity enhancements for VS Code:

| Guide                     | Description                                                     |
| ------------------------- | --------------------------------------------------------------- |
| **File Nesting**          | Collapse config files under `package.json` for cleaner explorer |
| **Key Bindings**          | Custom keyboard shortcuts for faster development                |
| **Workspace Indentation** | Per-project formatting consistency                              |

Each guide includes step-by-step instructions with screenshots.

---

## Contributing

Contributions are welcome! Here's how you can help:

1. **Fork** the repository
2. **Create** a feature branch (`git checkout -b feature/new-guide`)
3. **Commit** your changes (`git commit -m 'Add new TypeScript guide'`)
4. **Push** to the branch (`git push origin feature/new-guide`)
5. **Open** a Pull Request

### Contribution Guidelines

- Follow the existing file naming conventions
- Include practical code examples with ✅ Good and ❌ Bad patterns
- Add screenshots when documenting visual configurations
- Keep guides focused and concise

---

## License

This project is licensed under the **MIT License** - see the [LICENSE](LICENSE) file for details.

---

## Author

- GitHub: [@cardacci](https://github.com/cardacci)
- LinkedIn: [Gabriel Cardacci](https://www.linkedin.com/in/cardacci/)

---

```
  +---------------------------------------------------------+
  |                                                         |
  |     ⭐ Star this repo if you find it useful! ⭐        |
  |                                                         |
  +---------------------------------------------------------+
```
