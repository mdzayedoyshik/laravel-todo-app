
# Laravel Todo API

A JSON-only REST API for a todo app, built with Laravel 12. The domain is small on purpose, so the structure is easy to see. It is how I lay out a Laravel service: thin controllers, form requests for validation, services for business rules, repositories behind interfaces, and side effects such as email and streaks moved out of the request through events and queued listeners.

## How a request flows

```
Route (api-user-v1.php) -> FormRequest (validation) -> Controller -> Service -> Repository (interface) -> Eloquent
                                                                         |
                                     TodoObserver / TodoCompleted event -+-> queued listeners: completion email, streak update
```

Responses go through API Resources and a shared `ApiResponder` trait, so every endpoint returns the same envelope. Business rule violations, such as completing a todo twice or going over the per-user limit, throw a `BusinessException` that renders itself as a JSON error.

## Endpoints

All routes are versioned under `/api/v1` and split into three files by audience.

| Method | Path | Auth |
|---|---|---|
| GET | `/public/health` | none |
| POST | `/public/register`, `/public/sign-in` | none |
| GET | `/user/auth/me` · POST `/user/auth/sign-out` | Sanctum token |
| GET, PUT | `/user/profile` | Sanctum token |
| GET, POST, PUT, DELETE | `/user/todos`, `/user/todos/{id}` | Sanctum token |
| PATCH | `/user/todos/{id}/complete` | Sanctum token |
| GET, DELETE | `/admin/users`, `/admin/users/{id}` | Sanctum token + `role:admin` |
| GET | `/admin/stats`, `/admin/stats/todos` | Sanctum token + `role:admin` |

## Features

- User registration and authentication (Sanctum)
- Todo CRUD operations with priorities, due dates, and completion tracking
- Completion streaks per user, updated by a queued listener
- Completion email sent through a queued job
- Admin statistics: overall counts and completions over time
- RESTful API structure (public, user, admin routes)
- JSON only and API first, with no frontend build step
- Database seeding with sample users and todos

## Requirements

- PHP ^8.2
- Composer
- SQLite (default), MySQL, or PostgreSQL
- [Optional] Mail server for email notifications

## Installation

1. **Clone the repository:**
	```bash
	git clone https://github.com/xX0ZAK0Xx/laravel-todo-app.git
	cd laravel-todo-app
	```

2. **Install PHP dependencies:**
	```bash
	composer install
	```

3. **Copy and configure environment file:**
	```bash
	cp .env.example .env
	```
	- Set your database connection in `.env` (default is SQLite).
	- Configure mail settings if you want email notifications.

4. **Generate application key:**
	```bash
	php artisan key:generate
	```

5. **Run migrations:**
	```bash
	php artisan migrate
	```

## Seeding the Database

To populate the database with sample users and todos:

```bash
php artisan db:seed
```

- Seeds an admin user (`mdzayed@gmail.com` / `password`) and 10 regular users.
- Each user gets sample todos (including completed ones).

## Running the Application

Start the local development server:

```bash
php artisan serve
```

The API will be available at [http://localhost:8000](http://localhost:8000).

To also run the queue worker and log tailer alongside the server:

```bash
composer dev
```

## Running Tests

To run the test suite:

```bash
php artisan test
```
or
```bash
vendor/bin/phpunit
```

## Project Structure

- `app/Models/` - Eloquent models (`User`, `Todo`)
- `app/Repositories/` - Data access logic
- `app/Services/` - Business logic
- `app/Events/`, `app/Listeners/`, `app/Observers/`, `app/Jobs/` - Side effects that run outside the request
- `routes/api-{public,user,admin}-v1.php` - Routes split by audience
- `database/seeders/` - Database seeders

## What I would add next

- Feature tests for the todo and admin endpoints. The test suite is still the Laravel default.
- An OpenAPI spec generated from the form requests.

## License

This project is open-sourced under the [MIT license](https://opensource.org/licenses/MIT).
