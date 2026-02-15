# Journal Transcription App

A React frontend for transcribing and managing journal page entries.

## Features

- User authentication (login/register)
- Upload journal page images with drag-and-drop
- View all journal entries sorted by date
- Filter entries by date range
- View individual entries with lazy-loaded images
- Delete entries with confirmation

## Tech Stack

- **React 18** - UI framework
- **Vite** - Build tool and dev server
- **React Router** - Client-side routing
- **Tailwind CSS** - Styling
- **date-fns** - Date formatting

## Getting Started

### Prerequisites

- Node.js 18+ installed
- Backend API running on `http://localhost:3001` (or update proxy in `vite.config.js`)

### Installation

```bash
cd app
npm install
```

### Development

Start the development server:

```bash
npm run dev
```

The app will be available at `http://localhost:5173`

### Production Build

Build for production:

```bash
npm run build
```

Preview the production build:

```bash
npm run preview
```

The built files will be in the `dist/` directory.

## API Configuration

The app expects a backend API at `http://localhost:3001`. The Vite dev server proxies `/api/*` requests to this backend.

To change the backend URL, edit `vite.config.js`:

```js
server: {
  proxy: {
    '/api': {
      target: 'http://your-backend-url:port',
      changeOrigin: true,
    },
  },
}
```

## Project Structure

```
app/
├── src/
│   ├── components/       # Reusable components
│   │   ├── auth/        # Authentication components
│   │   ├── entries/     # Entry-related components
│   │   ├── layout/      # Layout components (Header, Navbar)
│   │   ├── pages/       # Page-specific components
│   │   └── ui/          # UI primitives (Button, Input, Modal)
│   ├── context/         # React contexts (AuthContext)
│   ├── hooks/           # Custom hooks (useAuth)
│   ├── pages/           # Page components
│   ├── services/        # API service layer
│   ├── App.jsx          # Main app component
│   └── main.jsx         # Entry point
├── index.html
└── package.json
```

## Available Routes

- `/login` - User login page
- `/register` - User registration page
- `/` - Dashboard with all entries
- `/upload` - Upload new journal page
- `/entry/:id` - View individual entry details
