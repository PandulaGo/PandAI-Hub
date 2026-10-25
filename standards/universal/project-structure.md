# Project Structure

## Standard Folder Layout

All projects follow this consistent structure:

```
project-root/
├── Backend/                    # Backend application code
├── Frontend/                   # Frontend application code
├── Tests/                      # Test projects/suites
│   ├── Unit/
│   ├── Integration/
│   └── E2E/
├── Scripts/                    # Utility scripts (build, deploy, etc.)
├── Configurations/             # Environment configs
│   ├── .env
│   ├── appsettings.backend.json
│   └── appsettings.frontend.json
└── Docs/                       # Project documentation
    ├── architecture-and-api-reference.md
    ├── context-ledger.md
    └── sdlc/
        └── <project-slug>/
            ├── analysis.md
            ├── dev-notes.md
            ├── test-report.md
            └── state.json
```

## Rules

- **Empty folders**: Add `.gitkeep` to preserve empty folders in git
- **Backend/**: Contains server-side application (API, services, database logic)
- **Frontend/**: Contains client-side application (UI, components, state management)
- **Tests/**: Mirrors the structure of Backend/ and Frontend/ for test organization
- **Scripts/**: Build, deployment, and utility scripts
- **Configurations/**: Environment-specific configs (never commit secrets)
- **Docs/**: Always keep documentation up-to-date with implementation

## Technology-Specific Subfolders

Each technology stack has its own conventions within Backend/ and Frontend/:

### Backend (e.g., .NET)
```
Backend/
├── Controllers/
├── Services/
├── Repositories/
├── Models/
├── DTOs/
└── Program.cs
```

### Backend (e.g., Node.js/Express)
```
Backend/src/
├── controllers/
├── services/
├── repositories/
├── models/
├── routes/
└── main.ts
```

### Frontend (e.g., Angular)
```
Frontend/src/
├── app/
│   ├── components/
│   ├── services/
│   ├── models/
│   └── app.module.ts
└── main.ts
```

### Frontend (e.g., React)
```
Frontend/src/
├── components/
├── hooks/
├── services/
├── types/
└── App.tsx
```

## Configuration Management

- Use `.env` files for environment variables
- Never commit `.env` files with secrets
- Commit `.env.example` with required variable names
- Use stack-specific config files (appsettings.json for .NET, etc.)
