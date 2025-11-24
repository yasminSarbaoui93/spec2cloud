# Task: Frontend UI Scaffolding (React + TypeScript)

**Task ID**: 003  
**Feature**: Frontend Foundation  
**Priority**: Critical (Foundational)  
**Estimated Complexity**: High  
**Dependencies**: 001-task-infrastructure-scaffolding

---

## Task Description

Create the foundational React/TypeScript frontend structure with Azure Static Web Apps, Azure Fluent UI components, state management, routing, and API integration. This provides the UI framework that all user-facing features will build upon.

---

## Technical Requirements

### **Project Structure**

```
frontend/
├── public/
│   ├── index.html
│   └── favicon.ico
├── src/
│   ├── App.tsx                    # Main app component
│   ├── main.tsx                   # Entry point
│   ├── components/
│   │   ├── common/
│   │   │   ├── Button.tsx
│   │   │   ├── Input.tsx
│   │   │   ├── Table.tsx
│   │   │   ├── Modal.tsx
│   │   │   ├── FileUpload.tsx
│   │   │   ├── LoadingSpinner.tsx
│   │   │   └── ErrorBoundary.tsx
│   │   ├── layout/
│   │   │   ├── Header.tsx
│   │   │   ├── Sidebar.tsx
│   │   │   └── MainLayout.tsx
│   │   └── ...                   # Feature-specific components (added in feature tasks)
│   ├── pages/
│   │   ├── Dashboard.tsx
│   │   ├── NotFound.tsx
│   │   └── ...                   # Feature pages (added in feature tasks)
│   ├── services/
│   │   ├── api.ts                # Base API client
│   │   ├── propertyService.ts
│   │   ├── schemaService.ts
│   │   └── authService.ts
│   ├── hooks/
│   │   ├── useApi.ts             # React Query wrapper
│   │   ├── useAuth.ts
│   │   └── useFileUpload.ts
│   ├── types/
│   │   ├── property.ts
│   │   ├── schema.ts
│   │   ├── mapping.ts
│   │   └── api.ts
│   ├── utils/
│   │   ├── validators.ts
│   │   ├── formatters.ts
│   │   └── constants.ts
│   ├── styles/
│   │   └── global.css
│   └── routes.tsx                # React Router configuration
├── tests/
│   ├── setup.ts
│   ├── components/
│   │   └── Button.test.tsx
│   └── ...
├── package.json
├── tsconfig.json
├── vite.config.ts
├── tailwind.config.js
├── .eslintrc.json
├── staticwebapp.config.json      # Azure Static Web Apps configuration
└── README.md
```

---

## Core Components

### **1. Base API Client**

**File**: `src/services/api.ts`

```typescript
import axios, { AxiosInstance, AxiosRequestConfig, AxiosResponse } from 'axios';

const API_BASE_URL = import.meta.env.VITE_API_BASE_URL || 'http://localhost:7071/api';

class ApiClient {
  private client: AxiosInstance;

  constructor() {
    this.client = axios.create({
      baseURL: API_BASE_URL,
      timeout: 30000,
      headers: {
        'Content-Type': 'application/json',
      },
    });

    // Request interceptor for authentication
    this.client.interceptors.request.use(
      (config) => {
        const token = localStorage.getItem('auth_token');
        if (token) {
          config.headers.Authorization = `Bearer ${token}`;
        }
        return config;
      },
      (error) => Promise.reject(error)
    );

    // Response interceptor for error handling
    this.client.interceptors.response.use(
      (response) => response,
      (error) => {
        if (error.response?.status === 401) {
          // Handle unauthorized
          window.location.href = '/login';
        }
        return Promise.reject(error);
      }
    );
  }

  async get<T>(url: string, config?: AxiosRequestConfig): Promise<T> {
    const response: AxiosResponse<T> = await this.client.get(url, config);
    return response.data;
  }

  async post<T>(url: string, data?: any, config?: AxiosRequestConfig): Promise<T> {
    const response: AxiosResponse<T> = await this.client.post(url, data, config);
    return response.data;
  }

  async put<T>(url: string, data?: any, config?: AxiosRequestConfig): Promise<T> {
    const response: AxiosResponse<T> = await this.client.put(url, data, config);
    return response.data;
  }

  async delete<T>(url: string, config?: AxiosRequestConfig): Promise<T> {
    const response: AxiosResponse<T> = await this.client.delete(url, config);
    return response.data;
  }
}

export const apiClient = new ApiClient();
```

### **2. React Query Hook**

**File**: `src/hooks/useApi.ts`

```typescript
import { useQuery, useMutation, UseQueryOptions, UseMutationOptions } from '@tanstack/react-query';
import { apiClient } from '../services/api';

export function useApiQuery<T>(
  key: string[],
  url: string,
  options?: Omit<UseQueryOptions<T>, 'queryKey' | 'queryFn'>
) {
  return useQuery<T>({
    queryKey: key,
    queryFn: () => apiClient.get<T>(url),
    ...options,
  });
}

export function useApiMutation<TData, TVariables>(
  method: 'post' | 'put' | 'delete',
  url: string,
  options?: UseMutationOptions<TData, Error, TVariables>
) {
  return useMutation<TData, Error, TVariables>({
    mutationFn: (variables: TVariables) => {
      switch (method) {
        case 'post':
          return apiClient.post<TData>(url, variables);
        case 'put':
          return apiClient.put<TData>(url, variables);
        case 'delete':
          return apiClient.delete<TData>(url);
        default:
          throw new Error(`Unsupported method: ${method}`);
      }
    },
    ...options,
  });
}
```

### **3. Main Layout Component**

**File**: `src/components/layout/MainLayout.tsx`

```typescript
import React from 'react';
import { Outlet } from 'react-router-dom';
import { FluentProvider, webLightTheme, makeStyles } from '@fluentui/react-components';
import Header from './Header';
import Sidebar from './Sidebar';

const useStyles = makeStyles({
  root: {
    display: 'flex',
    height: '100vh',
    flexDirection: 'column',
  },
  body: {
    display: 'flex',
    flex: 1,
    overflow: 'hidden',
  },
  sidebar: {
    width: '240px',
    borderRight: '1px solid #e0e0e0',
    overflowY: 'auto',
  },
  content: {
    flex: 1,
    overflowY: 'auto',
    padding: '24px',
  },
});

export const MainLayout: React.FC = () => {
  const styles = useStyles();

  return (
    <FluentProvider theme={webLightTheme}>
      <div className={styles.root}>
        <Header />
        <div className={styles.body}>
          <aside className={styles.sidebar}>
            <Sidebar />
          </aside>
          <main className={styles.content}>
            <Outlet />
          </main>
        </div>
      </div>
    </FluentProvider>
  );
};
```

### **4. File Upload Component**

**File**: `src/components/common/FileUpload.tsx`

```typescript
import React, { useCallback, useState } from 'react';
import { useDropzone } from 'react-dropzone';
import { Button, ProgressBar, Text } from '@fluentui/react-components';
import { DocumentArrowUp24Regular, Dismiss24Regular } from '@fluentui/react-icons';

interface FileUploadProps {
  onFilesSelected: (files: File[]) => void;
  acceptedFileTypes?: string[];
  maxFiles?: number;
  maxSizeMB?: number;
}

export const FileUpload: React.FC<FileUploadProps> = ({
  onFilesSelected,
  acceptedFileTypes = ['.txt', '.csv', '.xlsx', '.pdf'],
  maxFiles = 20,
  maxSizeMB = 50,
}) => {
  const [uploadedFiles, setUploadedFiles] = useState<File[]>([]);

  const onDrop = useCallback((acceptedFiles: File[]) => {
    setUploadedFiles((prev) => [...prev, ...acceptedFiles]);
    onFilesSelected([...uploadedFiles, ...acceptedFiles]);
  }, [uploadedFiles, onFilesSelected]);

  const { getRootProps, getInputProps, isDragActive } = useDropzone({
    onDrop,
    accept: acceptedFileTypes.reduce((acc, ext) => ({ ...acc, [ext]: [] }), {}),
    maxFiles,
    maxSize: maxSizeMB * 1024 * 1024,
  });

  const removeFile = (index: number) => {
    const newFiles = uploadedFiles.filter((_, i) => i !== index);
    setUploadedFiles(newFiles);
    onFilesSelected(newFiles);
  };

  return (
    <div>
      <div
        {...getRootProps()}
        style={{
          border: '2px dashed #ccc',
          borderRadius: '8px',
          padding: '32px',
          textAlign: 'center',
          cursor: 'pointer',
          backgroundColor: isDragActive ? '#f0f0f0' : '#fff',
        }}
      >
        <input {...getInputProps()} />
        <DocumentArrowUp24Regular style={{ fontSize: '48px', color: '#0078d4' }} />
        <Text as="p" size={400}>
          {isDragActive
            ? 'Drop files here...'
            : `Drag files here or click to browse (${acceptedFileTypes.join(', ')})`}
        </Text>
        <Text as="p" size={300} style={{ color: '#666' }}>
          Max {maxFiles} files, {maxSizeMB}MB each
        </Text>
      </div>

      {uploadedFiles.length > 0 && (
        <div style={{ marginTop: '16px' }}>
          <Text as="h4" size={500}>Uploaded Files ({uploadedFiles.length})</Text>
          {uploadedFiles.map((file, index) => (
            <div key={index} style={{ display: 'flex', alignItems: 'center', marginTop: '8px' }}>
              <Text style={{ flex: 1 }}>{file.name} ({(file.size / 1024).toFixed(2)} KB)</Text>
              <Button
                icon={<Dismiss24Regular />}
                appearance="subtle"
                onClick={() => removeFile(index)}
              />
            </div>
          ))}
        </div>
      )}
    </div>
  );
};
```

### **5. Routes Configuration**

**File**: `src/routes.tsx`

```typescript
import React from 'react';
import { createBrowserRouter } from 'react-router-dom';
import { MainLayout } from './components/layout/MainLayout';
import Dashboard from './pages/Dashboard';
import NotFound from './pages/NotFound';
// Feature pages will be imported here (added in feature tasks)

export const router = createBrowserRouter([
  {
    path: '/',
    element: <MainLayout />,
    children: [
      {
        index: true,
        element: <Dashboard />,
      },
      // Feature routes will be added here
      {
        path: '*',
        element: <NotFound />,
      },
    ],
  },
]);
```

---

## Configuration Files

### **File**: `package.json`

```json
{
  "name": "spec2cloud-frontend",
  "version": "1.0.0",
  "type": "module",
  "scripts": {
    "dev": "vite",
    "build": "tsc && vite build",
    "preview": "vite preview",
    "lint": "eslint . --ext ts,tsx --report-unused-disable-directives --max-warnings 0",
    "test": "vitest",
    "test:ui": "vitest --ui",
    "test:coverage": "vitest run --coverage"
  },
  "dependencies": {
    "react": "^18.2.0",
    "react-dom": "^18.2.0",
    "react-router-dom": "^6.21.0",
    "@fluentui/react-components": "^9.45.0",
    "@fluentui/react-icons": "^2.0.230",
    "@tanstack/react-query": "^5.17.0",
    "axios": "^1.6.5",
    "react-dropzone": "^14.2.3",
    "zustand": "^4.4.7"
  },
  "devDependencies": {
    "@types/react": "^18.2.48",
    "@types/react-dom": "^18.2.18",
    "@typescript-eslint/eslint-plugin": "^6.18.1",
    "@typescript-eslint/parser": "^6.18.1",
    "@vitejs/plugin-react": "^4.2.1",
    "eslint": "^8.56.0",
    "eslint-plugin-react-hooks": "^4.6.0",
    "eslint-plugin-react-refresh": "^0.4.5",
    "typescript": "^5.3.3",
    "vite": "^5.0.11",
    "vitest": "^1.1.3",
    "@testing-library/react": "^14.1.2",
    "@testing-library/jest-dom": "^6.2.0",
    "tailwindcss": "^3.4.0",
    "autoprefixer": "^10.4.16",
    "postcss": "^8.4.33"
  }
}
```

### **File**: `vite.config.ts`

```typescript
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';

export default defineConfig({
  plugins: [react()],
  server: {
    port: 3000,
    proxy: {
      '/api': {
        target: 'http://localhost:7071',
        changeOrigin: true,
      },
    },
  },
  build: {
    outDir: 'dist',
    sourcemap: true,
  },
});
```

### **File**: `staticwebapp.config.json`

```json
{
  "routes": [
    {
      "route": "/api/*",
      "allowedRoles": ["authenticated"]
    }
  ],
  "navigationFallback": {
    "rewrite": "/index.html",
    "exclude": ["/api/*", "/*.{css,scss,js,png,gif,ico,jpg,svg}"]
  },
  "responseOverrides": {
    "404": {
      "rewrite": "/index.html",
      "statusCode": 200
    }
  },
  "globalHeaders": {
    "content-security-policy": "default-src 'self'; script-src 'self' 'unsafe-inline'; style-src 'self' 'unsafe-inline';"
  }
}
```

---

## Acceptance Criteria

### **Project Structure**
- [ ] All directories and files created as specified
- [ ] `package.json` includes all dependencies with correct versions
- [ ] TypeScript configuration (`tsconfig.json`) enables strict mode
- [ ] Vite configuration includes API proxy for local development

### **Core Components**
- [ ] `ApiClient` handles authentication, error handling, and retries
- [ ] `useApi` hooks integrate with React Query
- [ ] `MainLayout` renders header, sidebar, and content area
- [ ] `FileUpload` component supports drag-and-drop and file validation
- [ ] Routes configured with React Router

### **Styling**
- [ ] Azure Fluent UI components render correctly
- [ ] Tailwind CSS utility classes work
- [ ] Responsive design works on desktop and tablet
- [ ] Color scheme consistent with Azure design system

### **Testing**
- [ ] Vitest configured and runs tests successfully
- [ ] Sample tests for `Button` and `FileUpload` components pass
- [ ] Test coverage ≥85% for shared components

### **Deployment**
- [ ] Azure Static Web Apps configuration valid
- [ ] Build process completes without errors
- [ ] App deploys to Azure Static Web Apps successfully
- [ ] API routes proxied correctly

---

## Testing Requirements

### **Unit Tests**
- [ ] Test `ApiClient` methods (get, post, put, delete)
- [ ] Test API hooks (`useApiQuery`, `useApiMutation`)
- [ ] Test `FileUpload` component (file selection, validation, removal)
- [ ] Test routing configuration

### **Integration Tests**
- [ ] Test API client with mock backend
- [ ] Test file upload with mock Blob Storage
- [ ] Test authentication flow

### **E2E Tests** (Playwright)
- [ ] Navigate between pages
- [ ] Upload files via drag-and-drop
- [ ] Submit forms and verify API calls

---

## Success Metrics

- Frontend build completes in <2 minutes
- All core components render without errors
- API client successfully communicates with backend
- App loads in <2 seconds on good network connection
- 100% of Fluent UI components accessible (WCAG AA)

---

**Document Status**: Draft v1.0  
**Last Updated**: November 24, 2025  
**Owner**: Developer (spec2cloud workflow)  
**Dependencies**: 001-task-infrastructure-scaffolding
