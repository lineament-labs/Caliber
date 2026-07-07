# Stratum-Lite

## Domain Model
keeping the domain model incredibly lean, instead of heavy roles and groups with dynamic matrices, we are going to focus on specific permissions and allow assigning them to individual users

enterprise RBAC is reserved for **Stratum**

```typescript
// libs/auth/domain/src/lib/models/user.model.ts
export type CaliberPermission = 'invoice:create' | 'invoice:delete' | 'rates:configure' | 'reporting:view';

export interface CaliberUser {
  id: string; // Supabase Auth UUID
  email: string;
  companyName: string;
  permissions: CaliberPermission[];
  createdAt: number;
}
```
## Infrastructure

Dexie table will hold 
id: 'current_session' (A single static string key to enforce a local singleton row)

user_id: string (UUID)

email: string

permissions: string[]

jwt_token: string (For hydrating the Supabase client headers when network recovers)

expires_at: number

## Injection Tokens and Guards

```ts
// libs/auth/application/src/lib/auth/auth-repository.interface.ts
export interface AuthRepositoryInterface {
  getCurrentUser(): Observable<CaliberUser | null>;
  loginWithPasscodeOrProvider(credentials: any): Promise<CaliberUser>;
  hasPermission(permission: CaliberPermission): boolean;
}
```
our Angular route guards will check the Signal Store state, which is fed directly from the local Dexie auth_session table.

    When Online: The Supabase client verifies the JWT.

    When Offline: The Guard reads the locally stored permissions snapshot inside Dexie. If your brother opens the app on his phone, the app reads the cached session, sees rates:configure, and boots the UI instantly without ever hitting a timeout error trying to reach Supabase.
