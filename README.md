# Daily Tasks

Daily Tasks is a React app with an Express API and MongoDB persistence. Users can keep private personal tasks or collaborate in shared workspaces. The API checks authentication, ownership, and workspace roles for every protected operation.

## Local development

Requirements: Node.js `^20.19.0` or `>=22.12.0` and a MongoDB database (local MongoDB or MongoDB Atlas).

1. Set `MONGODB_URI` and a random `JWT_SECRET` of at least 32 characters in `.env` at the repository root or in `server/.env`. The API loads `server/.env` first, then uses the root `.env`, then `server/atlas-credentials.env` for any still-missing values; environment variables supplied by the host take precedence. For MongoDB Atlas, use the connection string from your Atlas cluster and URL-encode any special characters in the database password. Keep credential files private.
2. Install packages with `npm install`.
3. Start the API with `npm start` (or `npm run dev:server` for automatic restarts).
4. In a second terminal, start the Vite app with `npm run dev`.
5. Open the URL printed by Vite. The dev server proxies `/api` requests to `http://localhost:3000`.

To run the API and frontend on different but same-site origins (for example, separate subdomains), configure `CLIENT_ORIGIN` and `VITE_API_URL` for those origins. Use HTTPS in production so the session cookie is secure. Cross-site deployments need an additional CSRF-token strategy before changing the session cookie's `SameSite` policy.

## Environment variables

| Variable | Required | Purpose |
| --- | --- | --- |
| `MONGODB_URI` | Yes | MongoDB connection string |
| `JWT_SECRET` | Yes | Secret used to sign seven-day session tokens; at least 32 characters |
| `PORT` | No | API port (defaults to `3000`) |
| `CLIENT_ORIGIN` | No | Allowed browser origin (defaults to `http://localhost:5173`) |
| `VITE_API_URL` | No | API origin or base URL for the browser (defaults to `/api`; `/api` is added when omitted) |
| `NODE_ENV` | No | Set to `production` to enable secure-only session cookies |

## MongoDB collections

Mongoose manages four collections. IDs below are `ObjectId` references; password hashes are never included in API responses.

```js
User {
  email: String,          // normalized, unique
  passwordHash: String,
  displayName: String,
  createdAt: Date,
  updatedAt: Date
}

Workspace {
  name: String,
  createdBy: ObjectId,    // User
  archivedAt: Date | null,
  createdAt: Date,
  updatedAt: Date
}

Membership {
  workspaceId: ObjectId,  // Workspace
  userId: ObjectId,       // User
  role: "owner" | "admin" | "member",
  joinedAt: Date
}
// Unique compound index: workspaceId + userId.

Task {
  title: String,
  description: String,
  status: "open" | "in_progress" | "completed",
  dueAt: Date | null,
  completedAt: Date | null,
  personalOwnerId: ObjectId | null, // User for a personal task
  workspaceId: ObjectId | null,     // Workspace for a shared task
  createdBy: ObjectId,              // User
  updatedBy: ObjectId,              // User
  archivedAt: Date | null,
  createdAt: Date,
  updatedAt: Date
}
// Exactly one scope reference (personalOwnerId or workspaceId) is required.
```

Completed tasks remain in the database. Deleting a task sets `archivedAt`; active and archived lists are queried separately. Workspaces are archived rather than erased. Membership and task access is always checked on the server.

## Workspace permissions

| Action | Owner | Admin | Member |
| --- | --- | --- | --- |
| Read and create workspace tasks | Yes | Yes | Yes |
| Update and complete workspace tasks | Yes | Yes | Yes |
| Archive/restore workspace tasks | Any task | Any task | Tasks they created |
| Rename workspace | Yes | Yes | No |
| Add/remove regular members | Yes | Yes | No |
| Add or promote admins | Yes | No | No |
| Archive workspace | Yes | No | No |

The workspace owner cannot be removed or demoted through the member-management endpoints. Ownership transfer is not yet exposed.

## API

- `POST /api/auth/register`, `POST /api/auth/login`, `POST /api/auth/logout`, `GET /api/auth/me`
- `GET/POST /api/tasks`, `GET/PATCH/DELETE /api/tasks/:taskId`
- `GET/POST /api/workspaces`, `GET/PATCH/DELETE /api/workspaces/:workspaceId`
- `GET/POST /api/workspaces/:workspaceId/members`, `PATCH/DELETE /api/workspaces/:workspaceId/members/:membershipId`
- `GET/POST /api/workspaces/:workspaceId/tasks`

`DELETE` on tasks archives them; `PATCH` with `{ "restore": true }` restores a task from history. Task list endpoints accept `?archived=true` to return archived records.

## Checks

```sh
npm run lint
npm run test:server
npm run build
```
"# daily_tasks" 
