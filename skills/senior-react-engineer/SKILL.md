---
name: senior-react-engineer
description: Use when writing React frontend code, custom hooks, state management, data integration strategies, component architecture, and full-stack integration patterns. Covers modern React 18/19, performance, security, a11y, testing, caching, and real-time communication.
version: 1.1.0
updated: 2026-07-13
loads: [security-first, senior-ui-ux-designer]
lazy_load_sections:
  core_always: ["Service Layer Pattern", "Component Architecture", "State Management", "Data Integration"]
  on_demand:
    testing: ["Unit Testing", "Integration Testing", "E2E Testing"]
    performance: ["Code Splitting", "Lazy Loading", "Memoization", "Bundle Optimization"]
    security: ["XSS Prevention", "JWT Handling", "CORS Configuration"]
    caching: ["React Query", "SWR", "Local Cache"]
    realtime: ["WebSocket", "Server-Sent Events", "Optimistic Updates"]
---

# Senior React Engineer

## 🔴 PERINGATAN KRITIKAL — WAJIB DIIKUTI TANPA PENGECUALIAN

**Skill ini adalah standar emas untuk semua kode React yang kamu tulis.** Setiap pelanggaran bisa menyebabkan aplikasi error di production, data tidak sinkron, performa drop, atau security celah yang tidak terdeteksi sampai di-eksploit.

🔴 **Konsekuensi jika dilanggar:**
- **Service layer dilanggar (fetch langsung di komponen)** → migrasi arsitektur berdarah-darah, tiap ganti backend harus ganti semua komponen 1 per 1
- **Memory leak (useEffect tanpa cleanup)** → app crash setelah navigasi cepat, user frustrasi
- **Data contract mismatch** → app error diam-diam di production, data undefined tanpa ada error jelas
- **Security loophole (JWT di localStorage, XSS)** → data pengguna bocor, account takeover

Model **WAJIB** mematuhi setiap aturan di bawah. **JANGAN skip, JANGAN tawar-menawar.**

---

## 📌 Tujuan

Skill ini memastikan setiap kode React yang ditulis memiliki **arsitektur robust, backend-agnostic, dan production-grade**. Fokus pada separation of concerns, reusability, preventif terhadap error umum, serta pattern modern yang bikin aplikasi scalable dan maintainable dalam jangka panjang.

---

# ⚡ Strategic Patterns (WAJIB)

---

### 1. Service Layer Pattern

**JANGAN PERNAH** menembak API atau mengakses data langsung dari komponen UI. Pisahkan semua logika data ke layer khusus.

❌ **Salah:**
```tsx
// page.tsx — aduk data fetching dengan UI
export default function DashboardPage() {
  const [data, setData] = useState(null);
  
  useEffect(() => {
    fetch('https://api.example.com/dashboard')
      .then(res => res.json())
      .then(setData);
  }, []);
  
  // ...render
}
```

✅ **Benar:**
```tsx
// services/dashboard.ts — service layer terpisah
import { apiClient } from '@/lib/apiClient';

export interface DashboardData {
  totalUsers: number;
  activeSessions: number;
  revenue: number;
}

export async function getDashboardData(): Promise<DashboardData> {
  const response = await apiClient.get('/dashboard');
  return response.data;
}
```

```tsx
// hooks/useDashboard.ts — custom hook untuk komponen
import { useQuery } from '@tanstack/react-query';
import { getDashboardData } from '@/services/dashboard';

export function useDashboard() {
  return useQuery({
    queryKey: ['dashboard'],
    queryFn: getDashboardData,
  });
}
```

```tsx
// page.tsx — UI murni, tanpa tahu dari mana data berasal
import { useDashboard } from '@/hooks/useDashboard';

export default function DashboardPage() {
  const { data, isLoading, error } = useDashboard();
  
  if (isLoading) return <DashboardSkeleton />;
  if (error) return <ErrorFallback error={error} />;
  
  return <DashboardContent data={data} />;
}
```

---

### 2. Asynchronous Dummy Data Simulation

Jika pakai dummy data di tahap awal, **WAJIB** bungkus dalam fungsi asynchronous (Promise) agar transisi ke real API hanya mengganti isi fungsi tanpa menyentuh komponen.

❌ **Salah:**
```tsx
// services/dashboard.ts — return langsung, tidak realistis
export function getDashboardData(): DashboardData {
  return {
    totalUsers: 1500,
    activeSessions: 342,
    revenue: 45000000,
  };
}
```

✅ **Benar:**
```tsx
// services/dashboard.ts — Promise wrapper, simulasi network delay
const dummyData: DashboardData = {
  totalUsers: 1500,
  activeSessions: 342,
  revenue: 45000000,
};

export async function getDashboardData(): Promise<DashboardData> {
  // Simulasi delay jaringan 300-800ms
  await new Promise(resolve => setTimeout(resolve, 300 + Math.random() * 500));
  return dummyData;
}
```

```diff
- // Nanti saat transisi ke real API, cukup ganti isi fungsi:
+ export async function getDashboardData(): Promise<DashboardData> {
+   const response = await apiClient.get('/dashboard');
+   return response.data;
+ }
- // Tidak ada perubahan di hook atau komponen!
```

---

### 3. Environment Variables Management

**SELALU** simpan Base URL, API keys, dan konfigurasi rahasia di environment variables. **JANGAN PERNAH hardcode.**

❌ **Salah:**
```tsx
const API_URL = 'https://api.example.com/v2';
const API_KEY = 'sk_live_abc123xyz';

fetch(`${API_URL}/users`, {
  headers: { Authorization: `Bearer ${API_KEY}` },
});
```

✅ **Benar:**
```env
# .env.local — BASE URL + keys
VITE_API_BASE_URL=https://api.example.com/v2
VITE_API_TIMEOUT=30000
```

```tsx
// config/env.ts — centralized env access
export const env = {
  apiBaseUrl: import.meta.env.VITE_API_BASE_URL,
  apiTimeout: Number(import.meta.env.VITE_API_TIMEOUT) || 10000,
  isDev: import.meta.env.DEV,
  isProd: import.meta.env.PROD,
} as const;

// Jangan export VITE_API_KEY jika ada — cukup panggil di server-side proxy
```

```tsx
// lib/apiClient.ts — pake env, bukan hardcode
import axios from 'axios';
import { env } from '@/config/env';

export const apiClient = axios.create({
  baseURL: env.apiBaseUrl,
  timeout: env.apiTimeout,
});
```

---

### 4. Custom Hooks & DRY Logic Reusability

Pisahkan logika bisnis dari UI. Logika yang bisa dipakai ulang **WAJIB** diekstrak ke Custom Hook.

❌ **Salah:**
```tsx
// Dua komponen dengan logic auth yang sama di-copy paste
function Sidebar() {
  const [user, setUser] = useState(null);
  
  useEffect(() => {
    const token = localStorage.getItem('token');
    if (token) {
      fetch('/api/me', { headers: { Authorization: `Bearer ${token}` } })
        .then(res => res.json())
        .then(setUser);
    }
  }, []);
  
  if (!user) return <LoginButton />;
  return <UserMenu user={user} />;
}
```

✅ **Benar:**
```tsx
// hooks/useAuth.ts — reusable auth logic
interface AuthState {
  user: User | null;
  isLoading: boolean;
  isAuthenticated: boolean;
}

export function useAuth(): AuthState {
  const { data: user, isLoading } = useQuery({
    queryKey: ['currentUser'],
    queryFn: () => apiClient.get('/me').then(r => r.data),
    retry: false,
    staleTime: 5 * 60 * 1000,
  });

  return {
    user: user ?? null,
    isLoading,
    isAuthenticated: !!user,
  };
}
```

```tsx
// Komponen tinggal pake hook, tanpa duplikasi logic
function Sidebar() {
  const { user, isLoading, isAuthenticated } = useAuth();
  
  if (isLoading) return <Skeleton />;
  if (!isAuthenticated) return <LoginButton />;
  return <UserMenu user={user} />;
}

function Navbar() {
  const { user, isAuthenticated } = useAuth();
  // Hook yang sama, beda komponen, zero duplication
}
```

---

### 5. Error Boundary & Suspense Wrapper Components

Buat komponen wrapper untuk Error Boundary dan Suspense agar penanganan error dan loading **konsisten di seluruh aplikasi**.

❌ **Salah:**
```tsx
// Tiap komponen handle loading/error sendiri-sendiri, format beda-beda
function UserProfile() {
  if (loading) return <p>Loading...</p>;
  if (error) return <p>Error: {error.message}</p>;
}
```

✅ **Benar:**
```tsx
// components/ErrorBoundary.tsx
import { Component, ReactNode } from 'react';

interface Props {
  fallback?: ReactNode;
  children: ReactNode;
}

interface State {
  hasError: boolean;
  error: Error | null;
}

export class ErrorBoundary extends Component<Props, State> {
  state: State = { hasError: false, error: null };

  static getDerivedStateFromError(error: Error): State {
    return { hasError: true, error };
  }

  componentDidCatch(error: Error, info: React.ErrorInfo) {
    console.error('[ErrorBoundary]', error, info.componentStack);
    // Kirim ke error tracking (Sentry, LogRocket, dll)
  }

  render() {
    if (this.state.hasError) {
      return this.props.fallback ?? <DefaultErrorFallback error={this.state.error} />;
    }
    return this.props.children;
  }
}
```

```tsx
// components/SuspenseWrapper.tsx
interface SuspenseWrapperProps {
  children: ReactNode;
  fallback?: ReactNode;
}

export function SuspenseWrapper({ children, fallback }: SuspenseWrapperProps) {
  return (
    <Suspense fallback={fallback ?? <PageSkeleton />}>
      {children}
    </Suspense>
  );
}
```

```tsx
// Pake di routing:
<ErrorBoundary fallback={<AppCrashScreen />}>
  <SuspenseWrapper>
    <Routes>
      <Route path="/dashboard" element={<LazyDashboard />} />
    </Routes>
  </SuspenseWrapper>
</ErrorBoundary>
```

---

### 6. Data Contract Validation

**WAJIB** pastikan struktur dummy data di tahap awal identik dengan response API asli. Gunakan TypeScript interface sebagai **single source of truth**.

❌ **Salah:**
```tsx
// Dummy data pake field name, tapi API asli pake fullName
const dummyUser = { id: 1, name: 'John' };
// API response: { id: 1, fullName: 'John', email: 'john@mail.com' }
// Pas integrasi: user.name undefined → crash
```

✅ **Benar:**
```tsx
// types/user.ts — single source of truth
export interface User {
  id: string;
  fullName: string;
  email: string;
  avatarUrl: string | null;
  createdAt: string;
}

// services/user.ts — dummy sesuai contract
const dummyUser: User = {
  id: 'uuid-1',
  fullName: 'John Doe',
  email: 'john@mail.com',
  avatarUrl: null,
  createdAt: '2024-01-01T00:00:00Z',
};

export async function fetchUser(id: string): Promise<User> {
  await delay(300);
  // Saat real API: return apiClient.get(`/users/${id}`).then(r => r.data);
  return dummyUser;
}
```

```tsx
// helpers/validateContract.ts — optional runtime validation
import { z } from 'zod';

const userSchema = z.object({
  id: z.string().uuid(),
  fullName: z.string().min(1),
  email: z.string().email(),
  avatarUrl: z.string().nullable(),
  createdAt: z.string().datetime(),
});

export function validateUser(data: unknown): User {
  return userSchema.parse(data);
}
```

---

### 7. Memory Leak Prevention (AbortController)

**WAJIB** sertakan cleanup function di `useEffect` untuk mencegah memory leak saat user berpindah halaman sebelum data selesai dimuat.

❌ **Salah:**
```tsx
useEffect(() => {
  fetch('/api/heavy-data')
    .then(res => res.json())
    .then(setData);
  // Kalau user pindah halaman, fetch tetap jalan → state di-update di komponen unmount
}, []);
```

✅ **Benar:**
```tsx
useEffect(() => {
  const controller = new AbortController();
  const { signal } = controller;

  fetch('/api/heavy-data', { signal })
    .then(res => res.json())
    .then(setData)
    .catch(err => {
      if (err.name !== 'AbortError') {
        console.error('Fetch failed:', err);
      }
    });

  return () => controller.abort(); // Cleanup saat unmount
}, []);
```

```tsx
// Custom hook reusable
export function useFetchWithCleanup<T>(url: string) {
  const [data, setData] = useState<T | null>(null);
  const [error, setError] = useState<Error | null>(null);

  useEffect(() => {
    const controller = new AbortController();
    
    fetch(url, { signal: controller.signal })
      .then(res => res.json())
      .then(setData)
      .catch(err => {
        if (err.name !== 'AbortError') setError(err);
      });

    return () => controller.abort();
  }, [url]);

  return { data, error };
}
```

---

### 8. Safe Rendering (Optional Chaining + Nullish Coalescing)

**SELALU** gunakan optional chaining (`?.`) dan nullish coalescing (`??`) saat me-render data dari API untuk mencegah aplikasi crash karena data lambat atau undefined.

❌ **Salah:**
```tsx
// Crash kalau user.address atau user.address.city undefined
<p>{user.address.city}</p>
<p>{user.profile.bio}</p>

// Conditional render yang verbose
{user && user.profile && user.profile.bio && <p>{user.profile.bio}</p>}
```

✅ **Benar:**
```tsx
// Safe — fallback ke empty string atau default value
<p>{user?.address?.city ?? 'Unknown'}</p>
<p>{user?.profile?.bio ?? 'No bio'}</p>

// Untuk array
<p>{user?.tags?.length ?? 0} tags</p>

// Untuk function calls
{user?.getDisplayName?.() ?? 'Anonymous'}

// Untuk nested data dengan default
const displayName = user?.profile?.displayName ?? user?.email ?? 'Guest';
```

---

# 🧠 Advanced Patterns (Senior-Level)

---

### 9. State Management Decision Tree

🔄 **INTERAKTIF** — Pilih mode:

```
[1] ⚡ Recommended (optimized default)
    → React Query untuk server state + Zustand untuk client state
    → Langsung dapet config + boilerplate

[2] ✏️ Custom (tanya detail)
```

**Jika pilih Custom:**

```
App scale:
[a] Small (1-3 pages, data sederhana)
[b] Medium (multi-page, data sharing antar komponen)
[c] Large (complex state, banyak global data)

Data fetching:
[a] Mostly server state (data dari API)
[b] Mostly client state (UI state, form, modal)
[c] Campuran

Special needs:
[ ] Real-time / WebSocket
[ ] Optimistic updates
[ ] Cache persistence (localStorage)
[ ] None
```

**Output rekomendasi:**

| Kombinasi | Rekomendasi |
|-----------|-------------|
| Small + server | React Query / SWR |
| Small + client | Context + useReducer |
| Medium + campuran | Zustand + React Query |
| Large + complex | Redux Toolkit + RTK Query |
| Real-time perlu | Zustand + subscription |

```tsx
// Recommended boilerplate: Zustand + React Query
// store/useAppStore.ts — client state
import { create } from 'zustand';

interface AppState {
  sidebarOpen: boolean;
  theme: 'light' | 'dark';
  toggleSidebar: () => void;
  setTheme: (theme: 'light' | 'dark') => void;
}

export const useAppStore = create<AppState>((set) => ({
  sidebarOpen: true,
  theme: 'light',
  toggleSidebar: () => set((s) => ({ sidebarOpen: !s.sidebarOpen })),
  setTheme: (theme) => set({ theme }),
}));
```

---

### 10. Component Composition (Compound & Slot Pattern)

Gunakan komposisi, bukan konfigurasi. Komponen yang fleksibel menggunakan **Compound Component** atau **Slot Pattern**.

❌ **Salah:**
```tsx
// Props explosion — semua opsi di parent
<Modal
  open={isOpen}
  onClose={handleClose}
  title="Confirm"
  body="Are you sure?"
  footer={<button onClick={handleClose}>OK</button>}
  showCloseButton
  closeOnOverlayClick
  size="md"
  // ... terus nambah props
/>
```

✅ **Benar (Compound Component):**
```tsx
// components/Modal.tsx — Compound dengan Context
import { createContext, useContext, ReactNode } from 'react';

interface ModalContextType {
  open: boolean;
  onClose: () => void;
}

const ModalContext = createContext<ModalContextType | null>(null);

function useModal() {
  const ctx = useContext(ModalContext);
  if (!ctx) throw new Error('Modal.* must be used within <Modal>');
  return ctx;
}

export function Modal({ children, open, onClose }: {
  children: ReactNode;
  open: boolean;
  onClose: () => void;
}) {
  if (!open) return null;
  return (
    <ModalContext.Provider value={{ open, onClose }}>
      <div className="overlay" onClick={onClose}>
        <div className="modal" onClick={(e) => e.stopPropagation()}>
          {children}
        </div>
      </div>
    </ModalContext.Provider>
  );
}

Modal.Header = function Header({ children }: { children: ReactNode }) {
  return <div className="modal-header">{children}</div>;
};

Modal.Body = function Body({ children }: { children: ReactNode }) {
  return <div className="modal-body">{children}</div>;
};

Modal.Footer = function Footer({ children }: { children: ReactNode }) {
  return <div className="modal-footer">{children}</div>;
};

Modal.CloseButton = function CloseButton() {
  const { onClose } = useModal();
  return <button onClick={onClose} aria-label="Close">&times;</button>;
};
```

```tsx
// Penggunaan — fleksibel, no props explosion
<Modal open={isOpen} onClose={handleClose}>
  <Modal.CloseButton />
  <Modal.Header>Confirm</Modal.Header>
  <Modal.Body>Are you sure?</Modal.Body>
  <Modal.Footer>
    <button onClick={handleClose}>Cancel</button>
    <button onClick={handleConfirm}>Confirm</button>
  </Modal.Footer>
</Modal>
```

---

### 11. TypeScript Strict Typing

**WAJIB** tipe semua hook, service, dan props dengan TypeScript strict. Gunakan generic types untuk reusable patterns.

❌ **Salah:**
```tsx
// any = lost all type safety
export function useFetch(url: string): any {
  const [data, setData] = useState<any>(null);
  // ...
}

// Props tanpa tipe eksplisit
function Card({ title, children, className }) {
  return <div className={className}>{title}{children}</div>;
}
```

✅ **Benar:**
```tsx
// Generic hook — reusable untuk tipe data apa pun
interface FetchState<T> {
  data: T | null;
  isLoading: boolean;
  error: Error | null;
}

export function useFetch<T>(url: string): FetchState<T> {
  const [state, setState] = useState<FetchState<T>>({
    data: null,
    isLoading: true,
    error: null,
  });

  useEffect(() => {
    const controller = new AbortController();
    
    fetch(url, { signal: controller.signal })
      .then(res => res.json() as Promise<T>)
      .then(data => setState({ data, isLoading: false, error: null }))
      .catch(error => {
        if (error.name !== 'AbortError') {
          setState({ data: null, isLoading: false, error });
        }
      });

    return () => controller.abort();
  }, [url]);

  return state;
}

// Generic digunakan — tipe otomatis ter-infer
const { data, isLoading } = useFetch<User[]>('/api/users');
// data otomatis bertipe User[] | null
```

```tsx
// Props dengan tipe eksplisit + default
interface CardProps {
  title: string;
  children: ReactNode;
  className?: string;
  variant?: 'elevated' | 'outlined' | 'ghost';
}

export function Card({
  title,
  children,
  className,
  variant = 'elevated',
}: CardProps) {
  return (
    <div className={cn(`card card--${variant}`, className)}>
      <h3 className="card__title">{title}</h3>
      {children}
    </div>
  );
}
```

---

### 12. Performance Boundaries (React.memo, useMemo, useCallback)

**HANYA** gunakan React.memo / useMemo / useCallback saat benar-benar dibutuhkan. **JANGAN** over-optimize.

❌ **Salah (over-optimization):**
```tsx
// useMemo untuk operasi sederhana — overhead > benefit
const fullName = useMemo(() => `${first} ${last}`, [first, last]);

// React.memo untuk komponen yang always re-render anyway
const Button = React.memo(({ label }: { label: string }) => {
  return <button>{label}</button>;
});
```

❌ **Salah (missing optimization):**
```tsx
// ListItem ter-render ulang tiap parent re-render, padahal gak perlu
function UserList({ users }: { users: User[] }) {
  return users.map(user => <UserCard user={user} onSelect={handleSelect} />);
  // Tanpa key yang stabil atau memo, semua UserCard re-render
}
```

✅ **Benar:**
```tsx
// Pakai React.memo kalo:
// 1. Komponen sering re-render dengan props yang sama
// 2. Render-nya berat (list panjang, chart, dll)
export const UserCard = React.memo(function UserCard({
  user,
  onSelect,
}: {
  user: User;
  onSelect: (id: string) => void;
}) {
  return (
    <div onClick={() => onSelect(user.id)}>
      <img src={user.avatarUrl} alt={user.fullName} />
      <p>{user.fullName}</p>
    </div>
  );
});

// useCallback untuk callback yang jadi dependency child yang di-memo
function UserList({ users }: { users: User[] }) {
  const handleSelect = useCallback((id: string) => {
    router.push(`/users/${id}`);
  }, []);

  return (
    <VirtualList>
      {users.map(user => (
        <UserCard key={user.id} user={user} onSelect={handleSelect} />
      ))}
    </VirtualList>
  );
}

// useMemo untuk computed value yang expensive (filter/sort ribuan item)
const sortedUsers = useMemo(() => {
  return [...users].sort((a, b) => a.fullName.localeCompare(b.fullName));
}, [users]);

// JANGAN pake untuk string concat / simple math — no benefit
```

---

### 13. Error Recovery & Resilience

Buat sistem yang bisa pulih dari error secara otomatis — bukan cuma nampilin error doang.

❌ **Salah:**
```tsx
// Cuma nampilin error, user harus refresh manual
function PaymentStatus() {
  const { data, error } = usePaymentStatus(orderId);
  
  if (error) return <p>Payment failed: {error.message}</p>;
  // ...
}
```

✅ **Benar:**
```tsx
// hooks/useRetry.ts — exponential backoff retry
export function useRetry<T>(
  fn: () => Promise<T>,
  options?: { maxRetries?: number; baseDelay?: number }
) {
  const maxRetries = options?.maxRetries ?? 3;
  const baseDelay = options?.baseDelay ?? 1000;

  return useCallback(async (): Promise<T> => {
    for (let attempt = 1; attempt <= maxRetries; attempt++) {
      try {
        return await fn();
      } catch (error) {
        if (attempt === maxRetries) throw error;
        const delay = baseDelay * Math.pow(2, attempt - 1) + Math.random() * 500;
        await new Promise(resolve => setTimeout(resolve, delay));
      }
    }
    throw new Error('Unreachable');
  }, [fn, maxRetries, baseDelay]);
}
```

```tsx
// Optimistic update pattern
function LikeButton({ postId }: { postId: string }) {
  const queryClient = useQueryClient();

  const { mutate } = useMutation({
    mutationFn: () => apiClient.post(`/posts/${postId}/like`),
    
    // Optimistic update — langsung ubah UI sebelum API response
    onMutate: async () => {
      await queryClient.cancelQueries({ queryKey: ['posts', postId] });
      const previous = queryClient.getQueryData(['posts', postId]);
      queryClient.setQueryData(['posts', postId], (old: Post) => ({
        ...old,
        likes: old.likes + 1,
        isLiked: true,
      }));
      return { previous };
    },
    
    // Rollback kalau gagal
    onError: (err, vars, context) => {
      queryClient.setQueryData(['posts', postId], context?.previous);
      toast.error('Failed to like post');
    },
    
    // Refetch untuk mastiin data akurat
    onSettled: () => {
      queryClient.invalidateQueries({ queryKey: ['posts', postId] });
    },
  });

  return <button onClick={() => mutate()}>Like</button>;
}
```

---

### 14. Testing Strategy

**WAJIB** test dengan pendekatan piramida: **unit (hooks/services) → integration (components) → e2e (flows)**.

❌ **Salah:**
```tsx
// Test yang cuma ngecek render doang, gak test logic
it('renders', () => {
  render(<UserProfile />);
  expect(screen.getByText('User')).toBeInTheDocument();
});
```

✅ **Benar:**
```tsx
// Unit test — service layer
import { getDashboardData } from '@/services/dashboard';

it('returns parsed dashboard data', async () => {
  const data = await getDashboardData();
  
  expect(data).toMatchObject({
    totalUsers: expect.any(Number),
    activeSessions: expect.any(Number),
    revenue: expect.any(Number),
  });
});
```

```tsx
// Integration test — hook + service (mock service)
import { renderHook, waitFor } from '@testing-library/react';
import { useDashboard } from '@/hooks/useDashboard';

jest.mock('@/services/dashboard');

it('returns dashboard data from service', async () => {
  (getDashboardData as jest.Mock).mockResolvedValue({
    totalUsers: 100,
    activeSessions: 20,
    revenue: 50000,
  });

  const { result } = renderHook(() => useDashboard());
  
  await waitFor(() => expect(result.current.isLoading).toBe(false));
  
  expect(result.current.data).toEqual({
    totalUsers: 100,
    activeSessions: 20,
    revenue: 50000,
  });
});
```

```tsx
// Integration test — komponen dengan error state
it('shows error fallback on API failure', async () => {
  (getDashboardData as jest.Mock).mockRejectedValue(
    new Error('Network Error')
  );

  render(<DashboardPage />);
  
  await waitFor(() => {
    expect(screen.getByText(/failed/i)).toBeInTheDocument();
  });
});
```

---

### 15. Project Structure Convention

Gunakan **feature-based** folder structure, bukan layer-based. Fitur yang sama berkumpul dalam satu folder.

❌ **Salah (layer-based, pisah-pisah):**
```
src/
  components/
    UserCard.tsx
    UserList.tsx
    PaymentForm.tsx
  hooks/
    useUsers.ts
    usePayment.ts
  services/
    users.ts
    payment.ts
  pages/
    UsersPage.tsx
    PaymentPage.tsx
```

✅ **Benar (feature-based):**
```
src/
  features/
    users/
      components/
        UserCard.tsx
        UserList.tsx
      hooks/
        useUsers.ts
      services/
        users.ts
      types/
        user.ts
      index.ts          # re-export public API
    payment/
      components/
        PaymentForm.tsx
      hooks/
        usePayment.ts
      services/
        payment.ts
      types/
        payment.ts
      index.ts

  components/           # Shared UI primitives only
    ui/
      Button.tsx
      Modal.tsx
      Input.tsx
    layout/
      Sidebar.tsx
      Navbar.tsx
  
  hooks/                # Shared hooks only
    useAuth.ts
    useMediaQuery.ts
  
  lib/                  # Utility libraries
    apiClient.ts
    utils.ts
  
  config/
    env.ts
    
  routes/
    index.tsx
```

---

# 🚀 Modern React Patterns

---

### 16. Modern React 18/19 Patterns

Gunakan Concurrent Features untuk UX yang lebih responsif. Pahami kapan pakai `useTransition`, `useDeferredValue`, dan Suspense untuk data fetching.

❌ **Salah:**
```tsx
// Semua state update blocking — UI freeze pas filter data besar
function SearchPage() {
  const [query, setQuery] = useState('');
  const filteredResults = bigData.filter(item => item.includes(query));
  
  return (
    <>
      <input onChange={(e) => setQuery(e.target.value)} />
      <ResultsList items={filteredResults} />
    </>
  );
}
```

✅ **Benar:**
```tsx
// useTransition — prioritaskan input, defer filter
function SearchPage() {
  const [query, setQuery] = useState('');
  const [isPending, startTransition] = useTransition();
  const [deferredQuery, setDeferredQuery] = useState('');

  const filteredResults = useMemo(
    () => bigData.filter(item => item.includes(deferredQuery)),
    [deferredQuery]
  );

  function handleChange(e: React.ChangeEvent<HTMLInputElement>) {
    const value = e.target.value;
    setQuery(value); // Langsung update input (urgent)
    startTransition(() => {
      setDeferredQuery(value); // Defer filter (non-urgent)
    });
  }

  return (
    <>
      <input value={query} onChange={handleChange} />
      {isPending && <Spinner />}
      <ResultsList items={filteredResults} />
    </>
  );
}
```

```tsx
// useDeferredValue — alternatif lebih simpel
function SearchPage() {
  const [query, setQuery] = useState('');
  const deferredQuery = useDeferredValue(query);
  const isPending = query !== deferredQuery;

  const filteredResults = useMemo(
    () => bigData.filter(item => item.includes(deferredQuery)),
    [deferredQuery]
  );

  return (
    <>
      <input value={query} onChange={(e) => setQuery(e.target.value)} />
      {isPending && <Spinner />}
      <ResultsList items={filteredResults} />
    </>
  );
}
```

```tsx
// Suspense untuk data fetching (dengan React Query)
function UserPage() {
  return (
    <Suspense fallback={<UserSkeleton />}>
      <UserProfile /> {/* Throw promise kalau data belum siap */}
    </Suspense>
  );
}

// Dengan React Query + suspense mode
function UserProfile() {
  const { data } = useQuery({
    queryKey: ['user'],
    queryFn: fetchUser,
    suspense: true, // Throw promise, biar Suspense yang handle
  });
  
  return <div>{data.fullName}</div>;
}
```

---

### 17. Caching & Data Freshness

🔄 **INTERAKTIF** — Pilih mode:

```
[1] ⚡ Recommended (optimized default)
    → TanStack Query + staleTime: 30s + stale-while-revalidate
    → Langsung dapet kode setup + hooks

[2] ✏️ Custom (tanya detail)
```

**Jika pilih Custom:**

```
Stale tolerance:
[a] Real-time (data harus selalu fresh)
[b] Near real-time (max 30 detik)
[c] Normal (beberapa menit)
[d] Static (jarang berubah)

Write strategy:
[a] Optimistic updates
[b] Refetch after mutate
[c] Polling (fetch berkala)

Offline support?
[ ] Yes
[ ] No
```

**Output rekomendasi:**

| Kondisi | Config |
|---------|--------|
| Real-time + optimistic | TanStack Query + `staleTime: 0` + `refetchInterval: 5000` + mutation callbacks |
| Normal + refetch | SWR + `dedupingInterval: 30000` |
| Static | TanStack Query + `staleTime: 5 * 60 * 1000` + `gcTime: 30 * 60 * 1000` |
| Near real-time + polling | `refetchInterval: 30000` + `refetchOnWindowFocus` |

```tsx
// Recommended boilerplate: TanStack Query
// main.tsx
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';

const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      staleTime: 30 * 1000,       // 30 detik sebelum dianggap stale
      gcTime: 5 * 60 * 1000,      // 5 menit cache di memory
      retry: 2,
      refetchOnWindowFocus: true,
      refetchOnReconnect: true,
    },
    mutations: {
      retry: 1,
    },
  },
});

root.render(
  <QueryClientProvider client={queryClient}>
    <App />
  </QueryClientProvider>
);
```

```tsx
// hooks/usePosts.ts — dengan cache invalidation pattern
export function usePosts() {
  return useQuery({
    queryKey: ['posts'],
    queryFn: () => apiClient.get('/posts').then(r => r.data),
  });
}

export function useCreatePost() {
  const queryClient = useQueryClient();
  
  return useMutation({
    mutationFn: (data: CreatePostDto) => apiClient.post('/posts', data),
    
    onSuccess: () => {
      // Invalidate list — refetch otomatis
      queryClient.invalidateQueries({ queryKey: ['posts'] });
      // Atau optimistis: langsung set cache tanpa refetch
      // queryClient.setQueryData(['posts'], (old) => [...old, newPost]);
    },
  });
}
```

---

# 📐 Essential Features

---

### 18. Form Handling Strategy

🔄 **INTERAKTIF** — Pilih mode:

```
[1] ⚡ Recommended (optimized default)
    → React Hook Form + zod resolver + controlled components
    → Langsung dapet kode form component + validation schema

[2] ✏️ Custom (tanya detail)
```

**Jika pilih Custom:**

```
Form size:
[a] Simple (1-5 fields, no validation logic)
[b] Medium (5-15 fields, basic validation)
[c] Complex (15+ fields, conditional fields, multi-step)

Validation:
[a] Basic required/email/pattern
[b] Schema validation (zod / yup)
[c] Custom async validation (cek unique, dll)

Multi-step / wizard?
[ ] Yes
[ ] No
```

**Output rekomendasi:**

| Kondisi | Rekomendasi |
|---------|-------------|
| Simple | Controlled biasa (useState) atau uncontrolled refs |
| Medium + schema | React Hook Form + zod resolver |
| Complex + multi-step | React Hook Form + zod + Context untuk persist per-step |
| Custom async | React Hook Form + `useWatch` untuk conditional |

```tsx
// Recommended boilerplate: React Hook Form + zod
import { useForm } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { z } from 'zod';

const userSchema = z.object({
  fullName: z.string().min(2, 'Name must be at least 2 characters'),
  email: z.string().email('Invalid email address'),
  age: z.coerce.number().min(18, 'Must be 18+'),
  role: z.enum(['admin', 'editor', 'viewer']),
});

type UserFormData = z.infer<typeof userSchema>;

export function UserForm({ onSubmit }: { onSubmit: (data: UserFormData) => Promise<void> }) {
  const {
    register,
    handleSubmit,
    formState: { errors, isSubmitting },
  } = useForm<UserFormData>({
    resolver: zodResolver(userSchema),
    defaultValues: {
      role: 'viewer',
    },
  });

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <Input
        label="Full Name"
        error={errors.fullName?.message}
        {...register('fullName')}
      />
      
      <Input
        label="Email"
        type="email"
        error={errors.email?.message}
        {...register('email')}
      />
      
      <Input
        label="Age"
        type="number"
        error={errors.age?.message}
        {...register('age')}
      />
      
      <Select label="Role" error={errors.role?.message} {...register('role')}>
        <option value="admin">Admin</option>
        <option value="editor">Editor</option>
        <option value="viewer">Viewer</option>
      </Select>

      <Button type="submit" disabled={isSubmitting}>
        {isSubmitting ? 'Saving...' : 'Save'}
      </Button>
    </form>
  );
}
```

---

### 19. Routing & Code Splitting

Gunakan lazy loading untuk route-level code splitting. Setiap route besar harus di-split biar bundle size tetap kecil.

❌ **Salah:**
```tsx
// Semua komponen di-load di awal — bundle size membengkak
import DashboardPage from '@/pages/DashboardPage';
import SettingsPage from '@/pages/SettingsPage';
import AdminPanel from '@/pages/AdminPanel'; // 500KB, jarang dikunjungi
```

✅ **Benar:**
```tsx
// routes/index.tsx — lazy load tiap route
const DashboardPage = lazy(() => import('@/features/dashboard/pages/DashboardPage'));
const SettingsPage = lazy(() => import('@/features/settings/pages/SettingsPage'));
const AdminPanel = lazy(() => import('@/features/admin/pages/AdminPanel'));

export function AppRoutes() {
  return (
    <Routes>
      <Route path="/" element={<Layout />}>
        <Route index element={
          <Suspense fallback={<PageSkeleton />}>
            <DashboardPage />
          </Suspense>
        } />
        <Route path="settings" element={
          <Suspense fallback={<PageSkeleton />}>
            <SettingsPage />
          </Suspense>
        } />
        <Route path="admin" element={
          <ProtectedRoute roles={['admin']}>
            <Suspense fallback={<PageSkeleton />}>
              <AdminPanel />
            </Suspense>
          </ProtectedRoute>
        } />
      </Route>
    </Routes>
  );
}
```

```tsx
// components/ProtectedRoute.tsx — route guard
function ProtectedRoute({
  children,
  roles,
}: {
  children: ReactNode;
  roles?: string[];
}) {
  const { user, isAuthenticated, isLoading } = useAuth();

  if (isLoading) return <PageSkeleton />;
  if (!isAuthenticated) return <Navigate to="/login" replace />;
  if (roles && !roles.includes(user?.role)) return <Navigate to="/" replace />;

  return <>{children}</>;
}
```

```tsx
// URL sebagai source of truth untuk state
// JANGAN simpan filter/sort/pagination di useState
// Simpan di URL search params:
function ProductListPage() {
  const [searchParams, setSearchParams] = useSearchParams();
  
  const page = Number(searchParams.get('page')) || 1;
  const sort = searchParams.get('sort') || 'newest';
  const category = searchParams.get('category') || 'all';

  function setPage(newPage: number) {
    setSearchParams(prev => {
      prev.set('page', String(newPage));
      return prev;
    });
  }
  // State persist di URL — bisa di-share, bisa di-bookmark
}
```

---

# 🔒 Security & Quality

---

### 20. Frontend Security

**Prinsip inti:** JWT di httpOnly cookie (bukan localStorage), sanitize sebelum dangerouslySetInnerHTML, proxy API keys via BFF.
**Detail lengkap:** Lihat skill `security-first` — XSS prevention, JWT best practices, CSP headers, input sanitization.

---

### 21. Design System & Theming Architecture

**Prinsip inti:** Theme Provider + CSS design tokens, dark mode dari awal, konsistensi styling seluruh aplikasi.
**Detail lengkap:** Lihat skill `senior-ui-ux-designer` — design tokens, CSS architecture, theming, dark/light mode, component specs.

---

### 22. DX & Code Quality Tooling

Setup ESLint, Prettier, dan pre-commit hooks untuk menjaga konsistensi kode otomatis.

❌ **Salah:**
```json
// Tanpa aturan spesifik React
{
  "extends": ["eslint:recommended"]
}
```

✅ **Benar:**
```json
// .eslintrc.cjs — WAJIB pake plugin react + hooks
{
  "extends": [
    "eslint:recommended",
    "plugin:react/recommended",
    "plugin:react-hooks/recommended",
    "plugin:@typescript-eslint/recommended"
  ],
  "rules": {
    "react/react-in-jsx-scope": "off",
    "react-hooks/exhaustive-deps": "warn",
    "react-hooks/rules-of-hooks": "error",
    "@typescript-eslint/no-unused-vars": ["warn", { "argsIgnorePattern": "^_" }],
    "no-console": ["warn", { "allow": ["warn", "error"] }]
  }
}
```

```json
// .prettierrc — formatting konsisten
{
  "semi": true,
  "singleQuote": true,
  "trailingComma": "all",
  "printWidth": 100,
  "tabWidth": 2
}
```

```json
// .vscode/settings.json — editor otomatis
{
  "editor.formatOnSave": true,
  "editor.defaultFormatter": "esbenp.prettier-vscode",
  "editor.codeActionsOnSave": {
    "source.fixAll.eslint": "explicit"
  }
}
```

```tsx
// tsconfig.json — strict mode WAJIB
{
  "compilerOptions": {
    "strict": true,
    "noUncheckedIndexedAccess": true,
    "noUnusedLocals": true,
    "baseUrl": ".",
    "paths": {
      "@/*": ["src/*"]
    }
  }
}
```

---

# 🎯 Cross-cutting Concerns

---

### 23. Accessibility (a11y)

**Prinsip inti:** Semantic HTML, ARIA attributes, keyboard navigation, focus trap untuk modal. WCAG 2.1 AA minimum.
**Detail lengkap:** Lihat skill `senior-ui-ux-designer` — a11y rules, ARIA patterns, focus management, color contrast, keyboard navigation.

---

### 24. Animation & Motion

Gunakan animasi dengan bijak — tingkatkan UX, jangan jadi gangguan. Respek `prefers-reduced-motion`.

❌ **Salah:**
```tsx
// Animasi yang gak bisa di-stop — bikin mual buat vestibular disorder
const Component = () => (
  <div style={{ animation: 'spin 1s infinite' }}>Loading...</div>
);
```

✅ **Benar:**
```tsx
// Respect prefers-reduced-motion
import { usePrefersReducedMotion } from '@/hooks/useMediaQuery';

// hooks/useMediaQuery.ts
export function usePrefersReducedMotion(): boolean {
  const matches = useMediaQuery('(prefers-reduced-motion: reduce)');
  return matches;
}

// framer-motion dengan reduced-motion respect
import { motion, type Variants } from 'framer-motion';

const fadeIn: Variants = {
  hidden: { opacity: 0, y: 20 },
  visible: { opacity: 1, y: 0 },
};

function AnimatedSection({ children }: { children: ReactNode }) {
  const prefersReducedMotion = usePrefersReducedMotion();

  if (prefersReducedMotion) {
    return <>{children}</>; // Skip animasi
  }

  return (
    <motion.div
      variants={fadeIn}
      initial="hidden"
      whileInView="visible"
      viewport={{ once: true }}
      transition={{ duration: 0.4 }}
    >
      {children}
    </motion.div>
  );
}
```

```tsx
// Layout animations — AnimatePresence
import { AnimatePresence, motion } from 'framer-motion';

function TodoList({ items }: { items: Todo[] }) {
  return (
    <AnimatePresence mode="popLayout">
      {items.map(item => (
        <motion.div
          key={item.id}
          layout
          initial={{ opacity: 0, x: -20 }}
          animate={{ opacity: 1, x: 0 }}
          exit={{ opacity: 0, x: 20 }}
          transition={{ type: 'spring', stiffness: 300, damping: 30 }}
        >
          {item.text}
        </motion.div>
      ))}
    </AnimatePresence>
  );
}
```

---

### 25. State Machines (Complex UI Flow)

Untuk flow kompleks (multi-step form, polling status, checkout wizard), gunakan Finite State Machine — bukan boolean explosion.

❌ **Salah:**
```tsx
// Boolean explosion — state gak terdefinisi dengan jelas
const [isLoading, setIsLoading] = useState(false);
const [isError, setIsError] = useState(false);
const [isSuccess, setIsSuccess] = useState(false);
const [isPolling, setIsPolling] = useState(false);
// Kombinasi ilegal: isLoading + isSuccess bisa true bersamaan!
```

✅ **Benar:**
```tsx
// FSM dengan useReducer — state transisi terdefinisi jelas
type PaymentState = 'idle' | 'processing' | 'polling' | 'success' | 'failed';
type PaymentEvent =
  | { type: 'SUBMIT' }
  | { type: 'POLL_START' }
  | { type: 'POLL_COMPLETE' }
  | { type: 'POLL_FAILED' }
  | { type: 'SUCCESS' }
  | { type: 'FAILED'; error: string }
  | { type: 'RESET' };

interface PaymentFSM {
  state: PaymentState;
  error: string | null;
}

function paymentReducer(state: PaymentFSM, event: PaymentEvent): PaymentFSM {
  switch (state.state) {
    case 'idle':
      if (event.type === 'SUBMIT') return { state: 'processing', error: null };
      break;
    case 'processing':
      if (event.type === 'POLL_START') return { state: 'polling', error: null };
      if (event.type === 'FAILED') return { state: 'failed', error: event.error };
      break;
    case 'polling':
      if (event.type === 'POLL_COMPLETE') return { state: 'success', error: null };
      if (event.type === 'POLL_FAILED') return { state: 'failed', error: 'Timeout' };
      break;
    case 'success':
    case 'failed':
      if (event.type === 'RESET') return { state: 'idle', error: null };
      break;
  }
  return state; // Invalid transition — ignore
}
```

```tsx
// hooks/usePayment.ts — hook dengan FSM
export function usePayment(orderId: string) {
  const [{ state, error }, dispatch] = useReducer(paymentReducer, {
    state: 'idle',
    error: null,
  });

  const submitPayment = useCallback(async () => {
    dispatch({ type: 'SUBMIT' });
    
    try {
      await apiClient.post(`/orders/${orderId}/pay`);
      dispatch({ type: 'POLL_START' });
      
      // Polling status
      const result = await pollUntilComplete(orderId);
      
      if (result.status === 'completed') {
        dispatch({ type: 'POLL_COMPLETE' });
      } else {
        dispatch({ type: 'POLL_FAILED' });
      }
    } catch (err) {
      dispatch({ type: 'FAILED', error: (err as Error).message });
    }
  }, [orderId]);

  return {
    state, // 'idle' | 'processing' | 'polling' | 'success' | 'failed'
    error,
    submitPayment,
    reset: () => dispatch({ type: 'RESET' }),
    isLoading: state === 'processing' || state === 'polling',
  };
}
```

```tsx
// Komponen — render berdasarkan state, bukan boolean
function PaymentButton({ orderId }: { orderId: string }) {
  const { state, error, submitPayment, reset, isLoading } = usePayment(orderId);

  switch (state) {
    case 'idle':
      return <Button onClick={submitPayment}>Pay Now</Button>;
    case 'processing':
      return <Button disabled><Spinner /> Processing...</Button>;
    case 'polling':
      return <Button disabled><Spinner /> Waiting for confirmation...</Button>;
    case 'success':
      return <SuccessMessage onReset={reset} />;
    case 'failed':
      return <ErrorMessage error={error} onRetry={submitPayment} onReset={reset} />;
  }
}
```

---

# 🔌 Real-time & Modern Infrastructure

---

### 26. Real-time Communication (WebSocket, SSE)

Untuk data real-time, gunakan WebSocket atau Server-Sent Events (SSE) dengan reconnection logic.

❌ **Salah:**
```tsx
// WebSocket tanpa cleanup atau reconnection
const ws = new WebSocket('wss://api.example.com/ws');
ws.onmessage = (event) => setData(JSON.parse(event.data));
// Gak ada cleanup — memory leak + koneksi nganggur
```

✅ **Benar:**
```tsx
// hooks/useWebSocket.ts — reusable WebSocket hook
interface UseWebSocketOptions {
  url: string;
  onMessage: (data: unknown) => void;
  reconnectInterval?: number;
  maxReconnectAttempts?: number;
}

export function useWebSocket({
  url,
  onMessage,
  reconnectInterval = 3000,
  maxReconnectAttempts = 10,
}: UseWebSocketOptions) {
  const wsRef = useRef<WebSocket | null>(null);
  const reconnectCountRef = useRef(0);
  const onMessageRef = useRef(onMessage);
  onMessageRef.current = onMessage;

  const connect = useCallback(() => {
    if (reconnectCountRef.current >= maxReconnectAttempts) return;

    const ws = new WebSocket(url);
    wsRef.current = ws;

    ws.onopen = () => {
      reconnectCountRef.current = 0; // Reset counter pas connect
    };

    ws.onmessage = (event) => {
      try {
        onMessageRef.current(JSON.parse(event.data));
      } catch {
        onMessageRef.current(event.data);
      }
    };

    ws.onclose = () => {
      if (reconnectCountRef.current < maxReconnectAttempts) {
        reconnectCountRef.current++;
        setTimeout(connect, reconnectInterval * reconnectCountRef.current);
      }
    };

    ws.onerror = () => {
      ws.close(); // Trigger onclose → reconnect
    };
  }, [url, reconnectInterval, maxReconnectAttempts]);

  useEffect(() => {
    connect();
    return () => {
      wsRef.current?.close();
      wsRef.current = null;
      reconnectCountRef.current = maxReconnectAttempts; // Stop reconnect on unmount
    };
  }, [connect, maxReconnectAttempts]);

  const send = useCallback((data: unknown) => {
    if (wsRef.current?.readyState === WebSocket.OPEN) {
      wsRef.current.send(JSON.stringify(data));
    }
  }, []);

  return { send, isConnected: wsRef.current?.readyState === WebSocket.OPEN };
}
```

```tsx
// Penggunaan
function LiveOrderBook() {
  const [orders, setOrders] = useState<Order[]>([]);

  const { send } = useWebSocket({
    url: `wss://api.example.com/orders/${orderId}/live`,
    onMessage: (data) => setOrders((data as LiveUpdate).orders),
  });

  return <OrderBookTable orders={orders} />;
}
```

```tsx
// Atau pake TanStack Query + refetchInterval untuk pseudo-real-time
function useLiveOrders(orderId: string) {
  return useQuery({
    queryKey: ['orders', orderId, 'live'],
    queryFn: () => apiClient.get(`/orders/${orderId}/status`),
    refetchInterval: 5000, // Poll tiap 5 detik
    refetchIntervalInBackground: true,
  });
}
```

---

### 27. i18n / Internationalization

Untuk aplikasi multi-bahasa, gunakan library i18n dengan locale switching yang mulus.

```tsx
// config/i18n.ts
import i18n from 'i18next';
import { initReactI18next } from 'react-i18next';
import LanguageDetector from 'i18next-browser-languagedetector';

import en from '@/locales/en.json';
import id from '@/locales/id.json';

i18n.use(LanguageDetector).use(initReactI18next).init({
  resources: { en: { translation: en }, id: { translation: id } },
  fallbackLng: 'en',
  interpolation: { escapeValue: false }, // React udah auto-escape
});

export default i18n;
```

```tsx
// components/LanguageSwitcher.tsx
import { useTranslation } from 'react-i18next';

export function LanguageSwitcher() {
  const { i18n } = useTranslation();

  return (
    <select
      value={i18n.language}
      onChange={(e) => i18n.changeLanguage(e.target.value)}
    >
      <option value="en">English</option>
      <option value="id">Bahasa Indonesia</option>
    </select>
  );
}
```

```tsx
// Penggunaan di komponen
import { useTranslation } from 'react-i18next';

function WelcomeMessage({ name }: { name: string }) {
  const { t } = useTranslation();

  return (
    <p>
      {t('welcome', { name })} {/* "Welcome, {{name}}" / "Selamat datang, {{name}}" */}
    </p>
  );
}
```

---

### 28. PWA / Offline-first

Untuk aplikasi yang perlu bekerja offline, gunakan service worker + IndexedDB untuk caching strategy.

```tsx
// vite.config.ts — generate service worker otomatis
import { VitePWA } from 'vite-plugin-pwa';

export default defineConfig({
  plugins: [
    VitePWA({
      registerType: 'autoUpdate',
      workbox: {
        globPatterns: ['**/*.{js,css,html,ico,png,svg,woff2}'],
        runtimeCaching: [
          {
            urlPattern: /^https:\/\/api\.example\.com\/.*/i,
            handler: 'StaleWhileRevalidate',
            options: {
              cacheName: 'api-cache',
              expiration: { maxEntries: 50, maxAgeSeconds: 60 * 60 * 24 },
            },
          },
        ],
      },
      manifest: {
        name: 'My App',
        short_name: 'App',
        theme_color: '#1a73e8',
        icons: [/* ... */],
      },
    }),
  ],
});
```

```tsx
// hooks/useOnlineStatus.ts
export function useOnlineStatus(): boolean {
  const [isOnline, setIsOnline] = useState(navigator.onLine);

  useEffect(() => {
    function handleOnline() { setIsOnline(true); }
    function handleOffline() { setIsOnline(false); }

    window.addEventListener('online', handleOnline);
    window.addEventListener('offline', handleOffline);
    return () => {
      window.removeEventListener('online', handleOnline);
      window.removeEventListener('offline', handleOffline);
    };
  }, []);

  return isOnline;
}
```

```tsx
// Offline banner
function OfflineBanner() {
  const isOnline = useOnlineStatus();

  if (isOnline) return null;

  return (
    <div role="alert" className="offline-banner">
      You're offline. Some features may be unavailable.
    </div>
  );
}
```

---

### 29. Micro-frontends (Module Federation)

Untuk aplikasi skala enterprise dengan tim terpisah, gunakan Module Federation untuk micro-frontends.

```tsx
// remote/webpack.config.js — expose komponen dari remote app
module.exports = {
  plugins: [
    new ModuleFederationPlugin({
      name: 'checkout',
      filename: 'remoteEntry.js',
      exposes: {
        './PaymentForm': './src/components/PaymentForm',
        './Cart': './src/components/Cart',
      },
      shared: {
        react: { singleton: true, requiredVersion: '^18.0.0' },
        'react-dom': { singleton: true },
      },
    }),
  ],
};
```

```tsx
// host/src/remotes/CheckoutRemote.tsx — lazy load remote component
const PaymentForm = lazy(() => import('checkout/PaymentForm'));
const Cart = lazy(() => import('checkout/Cart'));

export function CheckoutPage() {
  return (
    <ErrorBoundary fallback={<RemoteFallback name="Checkout" />}>
      <Suspense fallback={<RemoteSkeleton />}>
        <Cart />
        <PaymentForm />
      </Suspense>
    </ErrorBoundary>
  );
}
```

---

### 30. Storybook / Component Documentation

Dokumentasikan komponen UI dengan Storybook untuk memudahkan discovery dan testing visual.

```tsx
// src/components/ui/Button/Button.stories.tsx
import type { Meta, StoryObj } from '@storybook/react';
import { Button } from './Button';

const meta = {
  title: 'UI/Button',
  component: Button,
  argTypes: {
    variant: { control: 'select', options: ['primary', 'secondary', 'ghost'] },
    size: { control: 'select', options: ['sm', 'md', 'lg'] },
    disabled: { control: 'boolean' },
  },
} satisfies Meta<typeof Button>;

export default meta;

export const Primary: StoryObj<typeof meta> = {
  args: { children: 'Click Me', variant: 'primary' },
};

export const Disabled: StoryObj<typeof meta> = {
  args: { children: 'Disabled', disabled: true },
};

export const WithIcon: StoryObj<typeof meta> = {
  render: () => (
    <Button>
      <span aria-hidden>→</span> Next
    </Button>
  ),
};
```

---

### 31. Feature Flags

🔄 **INTERAKTIF** — Pilih mode:

```
[1] ⚡ Recommended (optimized default)
    → React Context + .env boolean flags untuk early stage
    → Bisa upgrade ke LaunchDarkly tanpa rewrite
    → Langsung dapet kode provider + hook + flag component

[2] ✏️ Custom (tanya detail)
```

**Jika pilih Custom:**

```
Scale:
[a] Small (1-5 flags, manual toggle)
[b] Medium (5-20 flags, gradual rollout)
[c] Large (20+ flags, A/B testing, multi-env)

Target audience:
[a] Internal dev toggle
[b] %-based rollout
[c] User segment / A/B testing

Need external dashboard?
[ ] Yes
[ ] No
```

**Output rekomendasi:**

| Kondisi | Rekomendasi |
|---------|-------------|
| Small + internal | Custom Context + .env boolean |
| Medium + % rollout | Flagsmith / custom service + localStorage cache |
| Large + A/B test | LaunchDarkly / GrowthBook + SDK |

```tsx
// Recommended boilerplate: Context-based flags, upgrade-ready

// types/flags.ts
export type FeatureFlag = 'newCheckout' | 'darkMode' | 'aiRecommendations';

export const defaultFlags: Record<FeatureFlag, boolean> = {
  newCheckout: import.meta.env.VITE_FLAG_NEW_CHECKOUT === 'true',
  darkMode: import.meta.env.VITE_FLAG_DARK_MODE === 'true',
  aiRecommendations: import.meta.env.VITE_FLAG_AI_RECS === 'true',
};
```

```tsx
// context/FeatureFlagContext.tsx
const FeatureFlagContext = createContext<Record<FeatureFlag, boolean>>(defaultFlags);

export function FeatureFlagProvider({ children }: { children: ReactNode }) {
  const [flags, setFlags] = useState(defaultFlags);

  // Nanti bisa upgrade: fetch flags dari server / LaunchDarkly
  // useEffect(() => {
  //   const ldClient = initialize('client-side-id');
  //   ldClient.on('ready', () => {
  //     setFlags({
  //       newCheckout: ldClient.variation('new-checkout', false),
  //       darkMode: ldClient.variation('dark-mode', false),
  //     });
  //   });
  // }, []);

  return (
    <FeatureFlagContext.Provider value={flags}>
      {children}
    </FeatureFlagContext.Provider>
  );
}

export function useFeatureFlag(flag: FeatureFlag): boolean {
  const flags = useContext(FeatureFlagContext);
  return flags[flag];
}
```

```tsx
// components/FeatureGate.tsx — conditional render based on flag
interface FeatureGateProps {
  flag: FeatureFlag;
  fallback?: ReactNode;
  children: ReactNode;
}

export function FeatureGate({ flag, fallback, children }: FeatureGateProps) {
  const isEnabled = useFeatureFlag(flag);

  if (!isEnabled) return fallback ?? null;
  return <>{children}</>;
}

// Penggunaan:
<FeatureGate flag="newCheckout" fallback={<OldCheckoutForm />}>
  <NewCheckoutForm />
</FeatureGate>
```

---

### 32. Monorepo (Turborepo / Nx)

Untuk multiple projects (web, admin, mobile, shared packages), gunakan monorepo dengan Turborepo.

```json
// turbo.json
{
  "pipeline": {
    "build": {
      "dependsOn": ["^build"],
      "outputs": ["dist/**", ".next/**"]
    },
    "lint": {},
    "test": {},
    "dev": {
      "cache": false,
      "persistent": true
    }
  }
}
```

```
packages/
  shared/
    types/          # Shared TypeScript interfaces
      user.ts
      product.ts
    utils/          # Shared utilities
      format.ts
      validation.ts
    ui/             # Shared UI components
      Button.tsx
      Modal.tsx
    package.json

apps/
  web/              # Next.js
    package.json
  admin/            # React + Vite
    package.json
  mobile/           # React Native
    package.json
```

```tsx
// packages/shared/types/user.ts — dikonsumsi oleh semua apps
export interface User {
  id: string;
  fullName: string;
  email: string;
  role: 'admin' | 'user';
}

// apps/web imports dari shared
import { User } from '@repo/types';
import { formatDate } from '@repo/utils';
import { Button } from '@repo/ui';
```

---

# 🚨 Self-Enforcement — WAJIB di Setiap Kode React

**Sebelum menulis komponen / hook / service / API call baru, WAJIB cek:**

1. **Service layer?** — JANGAN fetch langsung di komponen. Pisah ke `services/`.
2. **AbortController cleanup?** — WAJIB di setiap `useEffect` dengan async operation.
3. **Optional chaining?** — `?.` + `??` untuk SEMUA render data dari API.
4. **Data contract match?** — Dummy data HARUS identik dengan response API asli.
5. **Loading + error state?** — Setiap data fetching WAJIB punya fallback UI.
6. **State management scale?** — JANGAN pake Context untuk server state. JANGAN pake Redux untuk 3 komponen.
7. **Security?** — JANGAN hardcode secrets. JANGAN localStorage JWT. Sanitize HTML.
8. **a11y basics?** — Semantic HTML? Keyboard accessible? Labels?

**Kalau ragu dengan arsitektur atau approach → tanya user dulu. JANGAN asumsi.**

---

# ✅ Final Checklist

Sebelum menyelesaikan task React, pastikan:

```
□ Service layer dipisah dari komponen UI?
□ Dummy data pake Promise wrapper (bukan return langsung)?
□ Environment variables di .env, bukan hardcode?
□ Custom hooks untuk reusable logic?
□ Error Boundary + Suspense wrapper terpasang?
□ Data contract validated (dummy = API response)?
□ AbortController + cleanup di useEffect?
□ Optional chaining (?.) + nullish coalescing (??) untuk render?
□ State management sesuai scale?
□ Component composition (bukan props explosion)?
□ TypeScript strict + generic hooks?
□ Performance — memo/useMemo cuma kala perlu, bukan over-optimize?
□ Error recovery (retry / optimistic update)?
□ Testing — unit (services) + integration (hooks) + component?
□ Project structure feature-based?
□ Modern React — useTransition / useDeferredValue kala perlu?
□ Caching strategy sesuai kebutuhan data freshness?
□ Form handling — validation + error state?
□ Routing — lazy load + code splitting?
□ Frontend security — no JWT in localStorage, sanitize HTML?
□ Design system — design tokens + theme provider?
□ DX — ESLint react-hooks + Prettier + path aliases?
□ a11y — semantic HTML, keyboard nav, labels?
□ Animation — prefers-reduced-motion respected?
□ State machine untuk complex flow (gak pake boolean explosion)?
□ Real-time — proper cleanup + reconnection?
□ i18n — locale switching works?
□ Feature flags — Context-based, upgrade-ready?
□ Monorepo — shared packages properly referenced?
```
