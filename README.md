# Pet_Society
## Pet Society Admin Dashboard

**Pet Society** is a full-stack, Instagram-like admin dashboard for managing an animal platform (adoption / sale / mating) built with **Django (backend)** and **React + Material UI (frontend)**.

Admins can manage users, categories, and posts with role-based permissions, modern UI, dark/light themes, and persistent JWT-based sessions.

---

## Main Features

### Backend (Django + DRF)
- **JWT authentication** for admin / superuser access (access + refresh tokens via Simple JWT).
- **User management**: list users, block/unblock, promote/demote admin (only superuser can modify admins).
- **Category management**: create, read, update, delete animal categories (e.g., Cats, Dogs, Birds).
- **Post management**: list posts, view details, delete inappropriate posts.
- **Role-based access control (RBAC)**:
  - Admin vs Superuser permissions enforced in views.
  - Superuser is the only role allowed to delete other admins.
- **RESTful API** using Django REST Framework with clear URL structure.
- **CORS configuration** for safe cross-origin frontend communication.

### Frontend (React + Material UI)
- **Modern, responsive UI** built with Material UI (MUI) and custom theming.
- **Admin login page** with full-screen layout and paw-print branding.
- **Protected routes** using React Router and a custom `AuthContext`.
- **Persistent sessions**:
  - Access + refresh tokens stored in `localStorage`.
  - Axios interceptors automatically refresh expired access tokens.
  - Session stays active until admin explicitly logs out.
- **Dark / light mode**:
  - Global theme toggle (saved in `localStorage`).
  - Custom gradients, shadows, and typography for both modes.
- **Dashboard**:
  - Key statistics: total users, posts, categories, blocked users.
  - Quick navigation cards for Users, Categories, Posts, and Overview.
- **Users page**:
  - Search by username, email, or name.
  - Block/unblock users.
  - Promote/demote admins (superuser only).
  - User avatars and role/status badges.
- **Categories page**:
  - Add / edit / delete categories.
  - Toggle active/inactive status.
  - Responsive table and dialogs.
- **Posts page**:
  - Search by title, user, category, location.
  - View details in a modal (user info, category, price, contact, image).
  - Delete posts.

---

## Project Structure

```text
Pet_Society/
├── backend/
│   └── pet_society/
│       ├── admins/          # Admin app (Category, admin APIs, dashboard stats)
│       ├── posts/           # Posts app (animal posts)
│       ├── users/           # Custom user model + admin authentication APIs
│       ├── db.sqlite3       # Development database
│       └── pet_society/     # Django project (settings, urls, wsgi/asgi)
└── frontend/
    └── pet_society/
        ├── public/          # Static assets (favicon, index.html base)
        ├── src/
        │   ├── components/  # Reusable components (e.g. LoadingScreen)
        │   ├── contexts/    # AuthContext, ThemeContext
        │   ├── pages/       # LoginPage, Dashboard, UsersPage, PostsPage, CategoriesPage
        │   ├── App.jsx      # Main route configuration
        │   ├── main.jsx     # React entry point
        │   └── index.css    # Global styles
        ├── package.json
        └── vite.config.js
```

---

## Getting Started

### Prerequisites

- **Backend**
  - Python 3.10+ (or compatible with Django 5.2.x)
  - `pip` and `venv`
- **Frontend**
  - Node.js 18+ (recommended)
  - npm

---

## Backend Setup (Django)

1. **Navigate to backend directory:**

   ```bash
   cd backend/pet_society
   ```

2. **Create and activate virtual environment:**

   ```bash
   python3 -m venv venv
   source venv/bin/activate  # Windows: venv\Scripts\activate
   ```

3. **Install dependencies:**

   ```bash
   pip install -r requirements.txt
   ```

4. **Apply database migrations:**

   ```bash
   python manage.py makemigrations
   python manage.py migrate
   ```

5. **Create superuser (for initial login):**

   ```bash
   python manage.py createsuperuser
   ```

6. **Run development server:**

   ```bash
   python manage.py runserver
   ```

Backend will be available at: `http://127.0.0.1:8000/`

---

## Frontend Setup (React + Vite)

1. **Navigate to frontend directory:**

   ```bash
   cd frontend/pet_society
   ```

2. **Install dependencies:**

   ```bash
   npm install
   ```

3. **Start development server:**

   ```bash
   npm run dev
   ```

Frontend will be available at: `http://localhost:5173/`

> Ensure the backend (`http://127.0.0.1:8000`) is running so the frontend can communicate with the APIs.

---

## Configuration & Environment

### Backend (Django)

Key settings are in `backend/pet_society/pet_society/settings.py`:

- **CORS**:
  - `CORS_ALLOWED_ORIGINS` should include your frontend origin (e.g. `http://localhost:5173`).
- **REST Framework**:
  - Default authentication class: `JWTAuthentication`.
- **Simple JWT**:
  - Access / refresh token lifetimes are configured here.
- **Custom User Model**:
  - `AUTH_USER_MODEL = "users.User"` (or equivalent).

For local development, environment variables can be added via `.env` (if you extend the project to use `django-environ` or similar).

### Frontend (React)

API base URL is set in `AuthContext`:

```javascript
axios.defaults.baseURL = 'http://localhost:8000/api';
```

If your backend URL changes (e.g. Docker, production), update this value or move it to an environment variable (e.g. `VITE_API_BASE_URL`).

---

## Authentication & Session Flow

### Login Flow

1. Admin navigates to `http://localhost:5173/login`.
2. Credentials are sent to:
   - `POST /api/auth/login/`
3. Backend returns:
   - `access` (JWT access token)
   - `refresh` (JWT refresh token)
   - `user` object (id, username, role flags)
4. Frontend:
   - Saves tokens to `localStorage` (`access_token`, `refresh_token`).
   - Sets `Authorization: Bearer <access>` header on Axios.
   - Fetches profile from `GET /api/auth/profile/`.
   - Redirects to `/` (dashboard).

### Session Persistence

- On app load, `AuthContext`:
  - Reads tokens from `localStorage`.
  - Tries `GET /api/auth/profile/` with current access token.
  - If 401, automatically calls `POST /api/auth/token/refresh/` with refresh token.
  - Updates access token and retries.
- If refresh fails:
  - User is logged out.
  - Tokens are cleared.

### Logout Flow

- Frontend calls:
  - `POST /api/auth/logout/` with `refresh` token (for blacklisting).
- Clears:
  - `access_token`
  - `refresh_token`
  - Auth state in context.

---

## API Endpoints (Summary)

### Authentication
- `POST /api/auth/login/` – Admin login (returns `access`, `refresh`, `user`).
- `GET /api/auth/profile/` – Get current admin profile.
- `POST /api/auth/logout/` – Logout and blacklist refresh token.
- `POST /api/auth/token/refresh/` – Refresh access token using refresh token.

### Dashboard
- `GET /api/admin/dashboard/stats/` – Aggregated dashboard statistics.

### Categories
- `GET /api/admin/categories/` – List categories.
- `POST /api/admin/categories/` – Create category.
- `GET /api/admin/categories/{id}/` – Category details.
- `PUT /api/admin/categories/{id}/` – Update category.
- `DELETE /api/admin/categories/{id}/` – Delete category.

### Users
- `GET /api/admin/users/` – List users (with roles, status, post counts).
- `GET /api/admin/users/{id}/` – User details.
- `PATCH /api/admin/users/{id}/update/` – Block/unblock, promote/demote.

### Posts
- `GET /api/admin/posts/` – List posts.
- `GET /api/admin/posts/{id}/` – Post details.
- `DELETE /api/admin/posts/{id}/delete/` – Delete post.

---

## Using the Admin Dashboard

### 1. Admin Login
- Go to `http://localhost:5173/login`.
- Enter the superuser credentials you created (`createsuperuser`).
- On success, you are redirected to the main dashboard.

### 2. Dashboard Overview
- View:
  - Total users
  - Total posts
  - Categories
  - Blocked users
- Use the **Quick Actions** cards to navigate to:
  - Users
  - Categories
  - Posts

### 3. User Management
- Search by username, email, first name, or last name.
- See:
  - Role (`User`, `Admin`, `Super Admin`).
  - Status (`Active` / `Blocked`).
  - Posts count.
- Actions:
  - Block / unblock users.
  - Promote to admin or demote from admin (superuser only).

### 4. Category Management
- Add categories like **Cats**, **Dogs**, **Birds**, etc.
- Edit name, description, and active status.
- Delete categories no longer needed.

### 5. Post Management
- Browse all posts with:
  - Title, user, category, type, price, location, status.
- Search posts by:
  - Title, user, category, or location.
- View detailed post info in a dialog:
  - Image, description, contact info, user details, timestamps.
- Delete inappropriate or invalid posts.

---

## Security & Roles

### Security Features

- **JWT authentication** for all protected endpoints.
- **Role-based access control**:
  - Only admin/superuser can access `/api/admin/*` endpoints.
- **Permissions enforced on backend** using DRF permissions.
- **Protected frontend routes**:
  - React Router + `AuthContext` guard all admin pages.

### Roles

#### Superuser
- Full access to everything:
  - Manage users, posts, categories.
  - Promote/demote admins.
  - Delete admins.

#### Admin
- Manage regular users (except other admins).
- Manage categories and posts.
- **Cannot** delete or demote other admins.

---

## Technologies

### Backend
- Django 5.2.4
- Django REST Framework 3.16.0
- Django CORS Headers 4.7.0
- Django REST Framework Simple JWT 5.3.1
- Pillow 11.3.0 (image handling)

### Frontend
- React 19.1.0
- React Router DOM 6.x
- Material UI (MUI) 5.x
- Axios
- Vite (build tool / dev server)

---

## Development Workflow

### Adding Backend Features
1. Create / update models in the relevant Django app (`admins`, `posts`, `users`).
2. Add or update serializers.
3. Implement views (class-based or function-based) with proper permissions.
4. Register URLs in the app’s `urls.py` and include them in the project `urls.py`.
5. Run:

   ```bash
   python manage.py makemigrations
   python manage.py migrate
   ```

### Adding Frontend Features
1. Create a new page component in `src/pages/` or a reusable component in `src/components/`.
2. Wire up routing in `App.jsx`.
3. Use `AuthContext` and `ThemeContext` where needed.
4. Use Axios (already configured) to call backend APIs.
5. Run `npm run dev` to test in the browser.

---

## Troubleshooting

### Common Issues

1. **CORS errors**  
   - Check `CORS_ALLOWED_ORIGINS` includes the frontend origin.

2. **401 Unauthorized / Token issues**  
   - Verify `access_token` and `refresh_token` exist in `localStorage`.
   - Confirm `POST /api/auth/token/refresh/` works with your refresh token.

3. **Database / migration errors**  
   - Run:
     ```bash
     python manage.py makemigrations
     python manage.py migrate
     ```

4. **Port conflicts**  
   - Backend default: `8000`.
   - Frontend default: `5173`.
   - Change one of them if ports are already in use.

### Logs

- **Backend**: Django server terminal.
- **Frontend**: Browser dev tools console (`F12` / `Ctrl+Shift+I`).

---

## Contributing

- **Code style**:
  - Follow existing Django/DRF structure on the backend.
  - Use functional components and hooks on the frontend.
- **Error handling**:
  - Validate input on both backend and frontend.
  - Return meaningful API error messages.
- **Permissions**:
  - Always consider role-based access for new admin actions.
- **Testing**:
  - Manually test flows (login, logout, CRUD, role checks) before opening PRs.

---

## License

This project is intended for **educational and portfolio purposes** and can be extended or adapted for commercial use with appropriate modifications.

