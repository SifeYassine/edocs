# eDocs API

The **Laravel 11 REST API** for **eDocs**, a document management system. Users upload, categorize, and manage their documents (PDF, Office files, images, videos, and more), while administrators manage users, roles, and fine-grained permissions. Authentication uses **Laravel Sanctum**.

> Frontend: [edocs-frontend](https://github.com/SifeYassine/edocs-frontend) · Full-stack monorepo: [edocs-fullstack](https://github.com/SifeYassine/edocs-fullstack)

## Features

- **Sanctum token authentication**: register, login, logout
- **Automatic role bootstrap**: on the first sign-up, `Admin` and `User` roles are created and the first user becomes `Admin`
- **Permission-based access control** via a custom `PermissionMiddleware`:
  - `Admin` users bypass all checks
  - Other users need a specific permission (`Manage Roles`, `Manage Permissions`, `Manage Users`)
  - Permissions are assigned to and revoked from individual users
- **Documents**: upload (max 10 MB), rename, re-categorize, and delete
  - Allowed types: `pdf`, `docx`, `xlsx`, `pptx`, `txt`, `png`, `jpg`, `jpeg`, `mp4`
  - Stored on the `public` disk under `documents/`
- **Categories** per user (deleting a category keeps its documents, uncategorized)
- **Profile**: view and edit your own profile
- **Auto-generated API docs** via [laravel-request-docs](https://github.com/rakutentech/laravel-request-docs)

## Tech Stack

- PHP 8.2+, Laravel 11
- Laravel Sanctum
- MySQL (or SQLite)
- rakutentech/laravel-request-docs

## Data Model

| Table             | Key fields |
| ----------------- | ---------- |
| `users`           | `username`, `email`, `password`, `role_id` |
| `roles`           | `name`, `description` |
| `permissions`     | `label`, `description` |
| `permission_user` | `permission_id`, `user_id` |
| `categories`      | `name`, `user_id` |
| `documents`       | `title`, `format`, `path_url`, `category_id`, `user_id` |

## Getting Started

### Prerequisites

- PHP 8.2+ and [Composer](https://getcomposer.org/)
- [Node.js](https://nodejs.org/) and npm
- MySQL (or SQLite)

### Installation

```bash
git clone https://github.com/SifeYassine/edocs.git
cd edocs

composer install
npm install && npm run build

cp .env.example .env
php artisan key:generate
```

### Configure the database

```env
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=edocs
DB_USERNAME=root
DB_PASSWORD=
```

### Migrate and link storage

```bash
php artisan migrate
php artisan storage:link   # makes uploaded documents publicly accessible
```

### Run

```bash
php artisan serve
```

- API base URL: `http://127.0.0.1:8000/api`
- API docs: `http://127.0.0.1:8000/request-docs`

### Granting permissions

Register the first account (it becomes `Admin`). As Admin, create the permissions your team needs, using the exact labels `Manage Roles`, `Manage Permissions`, and `Manage Users`, and assign them to users with:

```
POST /api/users/{userId}/permissions/{permissionId}/assign
```

## API Overview

All routes are prefixed with `/api`. Except for `register` and `login`, send `Authorization: Bearer <token>`.

| Group | Endpoints | Required permission |
| ----- | --------- | ------------------- |
| Auth | `POST auth/register`, `POST auth/login`, `POST auth/logout` | none |
| Roles | `POST roles/create`, `GET roles/index`, `PUT roles/update/{id}`, `DELETE roles/delete/{id}` | `Manage Roles` |
| Permissions | `POST permissions/create`, `GET permissions/index`, `PUT permissions/update/{id}`, `DELETE permissions/delete/{id}` | `Manage Permissions` |
| User permissions | `POST users/{userId}/permissions/{permissionId}/assign`, `DELETE …/revoke` | `Manage Permissions` |
| | `GET users/{userId}/permissions/index` | authenticated |
| Users | `GET users/index`, `PUT users/update/{id}`, `DELETE users/delete/{id}` | `Manage Users` |
| Profile | `GET users/get_profile`, `PUT users/edit_profile` | authenticated |
| Categories | `POST categories/create`, `GET categories/index`, `PUT categories/update/{id}`, `DELETE categories/delete/{id}` | authenticated |
| Documents | `POST documents/create` (multipart: `path_url` file, optional `title`, `category_id`), `GET documents/index`, `PUT documents/update/{id}`, `DELETE documents/delete/{id}` | authenticated |

## Author

**Sifeddine Yassine** · [@SifeYassine](https://github.com/SifeYassine)
