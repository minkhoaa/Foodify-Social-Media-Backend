# Foodify — Social Media Backend

![.NET](https://img.shields.io/badge/.NET%208-512BD4?style=for-the-badge&logo=dotnet&logoColor=white)
![C#](https://img.shields.io/badge/C%23-239120?style=for-the-badge&logo=csharp&logoColor=white)
![PostgreSQL](https://img.shields.io/badge/PostgreSQL-4169E1?style=for-the-badge&logo=postgresql&logoColor=white)
![Redis](https://img.shields.io/badge/Redis-DC382D?style=for-the-badge&logo=redis&logoColor=white)
![Cloudinary](https://img.shields.io/badge/Cloudinary-3448C5?style=for-the-badge&logo=cloudinary&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-2496ED?style=for-the-badge&logo=docker&logoColor=white)
![Azure](https://img.shields.io/badge/Azure-0078D4?style=for-the-badge&logo=microsoftazure&logoColor=white)

A RESTful backend API for a food-focused social media platform. Users can share recipes, follow other cooks, like and comment on posts, receive notifications, and manage a personal ingredient library — all secured with JWT authentication and backed by a PostgreSQL database.

---

## Table of Contents

- [Features](#features)
- [Tech Stack](#tech-stack)
- [Architecture](#architecture)
- [Database Schema](#database-schema)
- [API Endpoints](#api-endpoints)
- [Getting Started](#getting-started)
- [Environment Variables](#environment-variables)
- [Docker](#docker)
- [CI/CD](#cicd)

---

## Features

- **Authentication & Authorization** — JWT Bearer token authentication, role-based access control via ASP.NET Core Identity, OTP-based email verification for sign-up and password reset
- **Recipe Management** — Create, read, update, and delete food recipes with ingredients, calorie tracking, and cover images
- **Social Interactions** — Like, comment, share, save, and report recipes; delete your own comments
- **User Profiles** — Follow/unfollow other users, view public profile information, update personal details
- **Notification System** — Real-time notifications for social interactions; mark notifications as seen
- **Ingredient Library** — Global ingredient catalogue with admin management controls
- **Image Uploads** — Cloud-hosted image storage via Cloudinary with multipart form-data support
- **Email Service** — Transactional email delivery using FluentEmail over SMTP (OTP codes, password resets)
- **Caching** — Redis distributed cache for session and performance-sensitive data
- **Admin Controls** — Admin-only endpoints for deleting comments and removing ingredients

---

## Tech Stack

| Layer | Technology |
|---|---|
| Framework | ASP.NET Core 8 (Minimal Hosting Model) |
| Language | C# 12 (.NET 8) |
| ORM | Entity Framework Core 8 (Npgsql provider) |
| Database | PostgreSQL |
| Cache | Redis (StackExchange.Redis) |
| Identity | ASP.NET Core Identity |
| Auth | JWT Bearer (Microsoft.AspNetCore.Authentication.JwtBearer) |
| Image Storage | Cloudinary (CloudinaryDotNet SDK) |
| Email | FluentEmail + SMTP |
| Object Mapping | AutoMapper 12 |
| API Documentation | Swagger / Swashbuckle |
| Containerization | Docker (multi-stage build, .NET 8 base image) |
| Deployment | Azure App Service |
| CI/CD | GitHub Actions |

---

## Architecture

The project follows a layered **Repository / Service pattern** with dependency injection throughout:

```
Controllers
    └── Repository Interfaces (IAccountRepository, IRecipeRepository, ...)
            └── Service Implementations (AccountManager, RecipeService, ...)
                    └── EF Core DbContext (FoodifyContext)
                            └── PostgreSQL
```

- **Controllers** handle HTTP routing, request validation, and response shaping. They depend only on repository interfaces.
- **Repository Interfaces** (`/Repository`) define the contracts for each domain area.
- **Service Implementations** (`/Service`) contain the business logic and interact directly with `FoodifyContext` and external services (Cloudinary, Redis, Email).
- **Data Layer** (`/Data`) holds EF Core entity classes, the `FoodifyContext`, and AutoMapper profile (`DbMapper`).
- **Model Layer** (`/Model`) contains DTOs and request/response models used for data transfer between controller and service layers.

---

## Database Schema

Key entities persisted via Entity Framework Core migrations:

| Entity | Description |
|---|---|
| `TaiKhoan` | Identity user account (extends `IdentityUser<int>`) |
| `VaiTro` | Identity role |
| `NguoiDung` | User profile information |
| `CongThuc` | Recipe post (title, description, calories, image, view/like/comment/share counters) |
| `NguyenLieu` | Ingredient master data |
| `CTCongThuc` | Recipe-ingredient junction (ingredient detail per recipe) |
| `CTDaThich` | Like relationships (user ↔ recipe) |
| `CTDaLuu` | Saved/bookmarked recipes (user ↔ recipe) |
| `CtDaShare` | Share relationships (user ↔ recipe) |
| `Comment` | Comments on recipes |
| `TheoDoi` | Follow relationships (follower ↔ followee) |
| `ThongBao` | Notification records |
| `CtToCaos` | Report records (user ↔ recipe) |
| `OtpEntry` | OTP codes for email verification and password reset |
| `DanhGia` | Rating records |

---

## API Endpoints

All routes are prefixed with `/api`. The full interactive documentation is available at `/swagger` when the application is running.

### Account — `/api/Account`

| Method | Route | Auth | Description |
|---|---|---|---|
| POST | `/signup` | Public | Initiate sign-up — sends OTP to email |
| POST | `/signup/otp` | Public | Confirm OTP and create account |
| POST | `/login` | Public | Sign in, returns JWT token pair |
| POST | `/login/auth` | Public | Validate / refresh token |
| POST | `/update` | Public | Update user profile information |
| POST | `/forgotPass` | Public | Send OTP for password reset |
| POST | `/forgotPass/otp` | Public | Confirm OTP and reset password |
| POST | `/getuserinfo` | Bearer | Retrieve current user info from token |

### Recipes — `/api/CongThuc`

| Method | Route | Auth | Description |
|---|---|---|---|
| POST | `/getAll` | Bearer | List all recipes (paginated / filtered) |
| GET | `/{id}` | Public | Get recipe by ID |
| POST | `/addcongthuc` | Bearer | Create a new recipe |
| PUT | `/updatecongthuc` | Bearer | Update an existing recipe |
| DELETE | `/deletecongthuc` | Bearer | Delete own recipe |
| POST | `/LikePost` | Bearer | Toggle like on a recipe |
| POST | `/CommentPost` | Bearer | Add a comment to a recipe |
| POST | `/deleteComment` | Bearer | Delete own comment |
| DELETE | `/deleteCommentForAdmin` | Admin | Admin delete any comment |
| POST | `/ShareCongthuc` | Bearer | Share a recipe |
| POST | `/getDetailedPost` | Bearer | Get full recipe detail with comments |
| POST | `/getallPostandSharedPost` | Bearer | Get own posts and shared posts feed |
| POST | `/getOneUserAndSharedPost` | Bearer | Get a specific user's posts and shares |
| POST | `/findPosts` | Bearer | Search recipes by keyword |
| POST | `/getComment` | Bearer | Get comments for a recipe |
| POST | `/reportCongthuc` | Bearer | Report a recipe |

### Users — `/api/NguoiDung`

| Method | Route | Auth | Description |
|---|---|---|---|
| POST | `/getAll` | Bearer | List all users |
| POST | `/getoneuserinfo` | Bearer | Get one user's public profile |
| POST | `/followUser` | Bearer | Follow or unfollow a user |
| POST | `/getAllThongBao` | Bearer | Get all notifications for current user |
| POST | `/seenOnePost` | Bearer | Mark a notification as seen |

### Ingredients — `/api/NguyenLieu`

| Method | Route | Auth | Description |
|---|---|---|---|
| POST | `/getallnguyenlieu` | Bearer | List all ingredients |
| POST | `/addnguyenlieu` | Bearer | Add a new ingredient |
| POST | `/deleteNguyenLieuForAdmin` | Admin | Admin delete an ingredient |

### Images — `/api/Image`

| Method | Route | Auth | Description |
|---|---|---|---|
| POST | `/Upload` | Public | Upload an image to Cloudinary, returns `public_id` and `url` |

---

## Getting Started

### Prerequisites

- [.NET 8 SDK](https://dotnet.microsoft.com/download/dotnet/8)
- [PostgreSQL](https://www.postgresql.org/download/) (v14 or higher recommended)
- [Redis](https://redis.io/download/) (v7 or higher recommended)
- A [Cloudinary](https://cloudinary.com/) account
- An SMTP mail account (Gmail, SendGrid, etc.)

### Local Setup

1. **Clone the repository**

   ```bash
   git clone https://github.com/minkhoaa/Foodify-Social-Media-Backend.git
   cd Foodify-Social-Media-Backend
   ```

2. **Create a `.env` file** in the project root (see [Environment Variables](#environment-variables) below) or populate `appsettings.json` directly.

3. **Apply database migrations**

   ```bash
   cd Foodify_DoAn
   dotnet ef database update
   ```

4. **Run the application**

   ```bash
   dotnet run
   ```

   The API will be available at `http://localhost:5000` and the Swagger UI at `http://localhost:5000/swagger`.

---

## Environment Variables

The application reads configuration from a `.env` file (via `DotNetEnv`) or from OS environment variables. All keys follow the `SECTION__KEY` convention.

| Variable | Description |
|---|---|
| `CONNECTIONSTRINGS__MYDB` | PostgreSQL connection string |
| `CONNECTIONSTRINGS__REDIS` | Redis connection string (e.g. `localhost:6379`) |
| `JWT__VALIDAUDIENCE` | JWT valid audience |
| `JWT__VALIDISSUER` | JWT valid issuer |
| `JWT__SECRETKEY` | JWT signing secret (min 32 characters recommended) |
| `EMAILSETTINGS__SMTPSERVER` | SMTP server hostname |
| `EMAILSETTINGS__SMTPPORT` | SMTP server port |
| `EMAILSETTINGS__SENDEREMAIL` | Sender email address |
| `EMAILSETTINGS__SENDERPASSWORD` | Sender email password or app password |
| `EMAILSETTINGS__ENABLESSL` | `true` or `false` |
| `CLOUDINARYSETTINGS__CLOUDNAME` | Cloudinary cloud name |
| `CLOUDINARYSETTINGS__APIKEY` | Cloudinary API key |
| `CLOUDINARYSETTINGS__APISECRET` | Cloudinary API secret |

---

## Docker

A multi-stage `Dockerfile` is included at the repository root. The image targets .NET 8 on Linux and exposes port `8080`.

```bash
# Build the image
docker build -t foodify-backend .

# Run the container (supply environment variables via --env-file or -e flags)
docker run -p 8080:8080 --env-file .env foodify-backend
```

---

## CI/CD

A GitHub Actions workflow (`.github/workflows/master_api-foodify.yml`) automatically builds and deploys the application to **Azure App Service** (`api-foodify`) on every push to the `master` branch.

Pipeline stages:
1. **Build** — Restore dependencies, compile in Release mode, publish artifacts
2. **Deploy** — Download build artifacts and deploy to the Azure Web App Production slot

The Azure publish profile is stored as the repository secret `AZUREAPPSERVICE_PUBLISHPROFILE_FA4834F94A2A4B24A052003897367A16`.
