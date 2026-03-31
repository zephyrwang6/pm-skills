# Project Scaffolding Reference

When `create-next-app` is unavailable or too slow, manually scaffold the project using these templates.

## package.json

```json
{
  "name": "prototype",
  "version": "0.1.0",
  "private": true,
  "scripts": {
    "dev": "next dev",
    "build": "next build",
    "start": "next start",
    "lint": "next lint"
  },
  "dependencies": {
    "next": "^14.2.0",
    "react": "^18.3.0",
    "react-dom": "^18.3.0"
  },
  "devDependencies": {
    "@types/node": "^20",
    "@types/react": "^18",
    "@types/react-dom": "^18",
    "autoprefixer": "^10",
    "postcss": "^8",
    "tailwindcss": "^3.4",
    "typescript": "^5"
  }
}
```

## tailwind.config.ts template

```typescript
import type { Config } from "tailwindcss";

const config: Config = {
  content: [
    "./src/**/*.{js,ts,jsx,tsx,mdx}",
  ],
  theme: {
    extend: {
      colors: {
        // REPLACE: Map extracted colors here
        // primary: { DEFAULT: '#xxx', light: '#xxx', dark: '#xxx' },
        // secondary: '#xxx',
      },
      fontFamily: {
        // REPLACE: Map extracted fonts
        // heading: ['Inter', 'sans-serif'],
        // body: ['Inter', 'sans-serif'],
      },
      spacing: {
        // REPLACE: Add custom spacings if needed
      },
      borderRadius: {
        // REPLACE: Add extracted radii
      },
      boxShadow: {
        // REPLACE: Add extracted shadows
      },
    },
  },
  plugins: [],
};

export default config;
```

## postcss.config.mjs

```javascript
/** @type {import('postcss-load-config').Config} */
const config = {
  plugins: {
    tailwindcss: {},
    autoprefixer: {},
  },
};

export default config;
```

## tsconfig.json

```json
{
  "compilerOptions": {
    "lib": ["dom", "dom.iterable", "esnext"],
    "allowJs": true,
    "skipLibCheck": true,
    "strict": true,
    "noEmit": true,
    "esModuleInterop": true,
    "module": "esnext",
    "moduleResolution": "bundler",
    "resolveJsonModule": true,
    "isolatedModules": true,
    "jsx": "preserve",
    "incremental": true,
    "plugins": [{ "name": "next" }],
    "paths": { "@/*": ["./src/*"] }
  },
  "include": ["next-env.d.ts", "**/*.ts", "**/*.tsx", ".next/types/**/*.ts"],
  "exclude": ["node_modules"]
}
```

## next.config.js

```javascript
/** @type {import('next').NextConfig} */
const nextConfig = {
  images: {
    remotePatterns: [
      { protocol: 'https', hostname: '**' },
    ],
  },
};

module.exports = nextConfig;
```

## src/app/globals.css

```css
@tailwind base;
@tailwind components;
@tailwind utilities;
```

## src/app/layout.tsx template

```tsx
import type { Metadata } from "next";
// import { Inter } from "next/font/google"; // REPLACE with extracted font
import "./globals.css";

// const inter = Inter({ subsets: ["latin"] }); // REPLACE

export const metadata: Metadata = {
  title: "Prototype",
  description: "Web prototype",
};

export default function RootLayout({
  children,
}: Readonly<{
  children: React.ReactNode;
}>) {
  return (
    <html lang="en">
      <body>{children}</body>
    </html>
  );
}
```
