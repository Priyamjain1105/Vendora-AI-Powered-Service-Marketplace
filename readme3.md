# Vendora - Complete Documentation

## Table of Contents

1. [Introduction](#introduction)
2. [Project Overview](#project-overview)
3. [File Architecture](#file-architecture)
4. [Database Architecture (DBMS)](#database-architecture-dbms)
5. [System Design](#system-design)
6. [Design Policies & Best Practices](#design-policies--best-practices)
7. [Project Design Theme](#project-design-theme)
8. [Business Logic](#business-logic)
9. [Authentication Implementation](#authentication-implementation)
10. [Features](#features)
11. [Deployment & Cloud Infrastructure](#deployment--cloud-infrastructure)
12. [Quick Navigation Guide](#quick-navigation-guide)

---

## Introduction

**Vendora** is a comprehensive web platform designed to bridge the gap between local service providers and customers in India. The platform enables local vendors such as barbers, mechanics, plumbers, electricians, and other service providers to create their professional online presence and allows customers to discover, view, and book appointments with these vendors seamlessly.

### Mission Statement
To empower local vendors across India by providing them with a digital platform to showcase their services, manage their business, and connect with customers in their community.

### Target Audience
- **Vendors**: Local service providers (barbers, mechanics, plumbers, electricians, etc.)
- **Customers**: Individuals seeking local services and wanting to book appointments

---

## Project Overview

### Technology Stack

**Backend Framework:**
- **Flask** (Python) - Lightweight and flexible web framework
- **Flask-SQLAlchemy** - ORM for database operations
- **Flask-Migrate** - Database migration management
- **Flask-Login** - User session management
- **Flask-Bcrypt** - Password hashing and security

**Database:**
- **MySQL** (via Aiven Cloud) - Production database
- **PyMySQL** - MySQL connector for Python

**Cloud Services:**
- **Cloudinary** - Image storage and management
- **Aiven Cloud** - Managed MySQL database hosting
- **Vercel** - Application deployment platform

**Frontend:**
- **Tailwind CSS v3** - Utility-first CSS framework
- **Jinja2** - Template engine
- **Inter Font** - Modern typography

**Development Tools:**
- **Python-dotenv** - Environment variable management
- **Alembic** - Database migration tool

---

## File Architecture

### Project Structure

```
vendora2/
├── run.py                          # Application entry point
├── config.py                       # Configuration file (currently empty)
├── requirements.txt                # Python dependencies
├── vercel.json                     # Vercel deployment configuration
├── vendora_app/                    # Main application package
│   ├── __init__.py                 # Package initialization
│   ├── app.py                      # Flask app factory and configuration
│   ├── extensions.py               # Extension initialization (DB, Bcrypt, LoginManager)
│   ├── models.py                   # Shared models (currently empty)
│   ├── templates/                  # Base templates
│   │   ├── base.html               # Main base template with Tailwind
│   │   ├── authbase.html           # Authentication pages base template
│   │   ├── theme.md                # Design theme documentation
│   │   ├── headers/                # Header components
│   │   │   ├── guest_header.html   # Header for unauthenticated users
│   │   │   └── user_header.html    # Header for authenticated users
│   │   └── sidebar/                # Sidebar components
│   │       ├── sidebar_base.html    # Base sidebar template
│   │       ├── sidebar_customer.html # Customer sidebar
│   │       └── sidebar_vendor.html  # Vendor sidebar
│   ├── migrations/                 # Database migrations
│   │   ├── versions/               # Migration version files
│   │   └── env.py                  # Alembic environment configuration
│   └── blueprints/                 # Modular route organization
│       ├── auth/                   # Authentication module
│       │   ├── __init__.py
│       │   ├── models.py           # User and Note models
│       │   ├── routes.py           # Auth routes (login, signup, logout, profile)
│       │   └── templates/          # Auth templates
│       ├── vendor/                 # Vendor module
│       │   ├── __init__.py
│       │   ├── models.py           # Vendor model
│       │   ├── routes.py           # Vendor routes (dashboard, profile, appointments)
│       │   └── templates/          # Vendor templates
│       ├── customer/               # Customer module
│       │   ├── __init__.py
│       │   ├── models.py           # Customer model
│       │   ├── routes.py           # Customer routes (dashboard, vendors list)
│       │   └── templates/          # Customer templates
│       ├── appointment/            # Appointment module
│       │   ├── __init__.py
│       │   ├── model.py            # Appointment model (currently empty)
│       │   ├── routes.py           # Appointment routes
│       │   └── templates/          # Appointment templates
│       └── core/                   # Core/utility routes
│           ├── __init__.py
│           ├── routes.py           # Home, about, developer pages
│           └── templates/          # Core templates
└── vendenv/                        # Virtual environment (not in version control)
```

### Architecture Benefits

#### 1. **Blueprint Pattern (Modular Architecture)**
- **Separation of Concerns**: Each module (auth, vendor, customer, appointment) is self-contained
- **Scalability**: Easy to add new features without affecting existing code
- **Maintainability**: Changes in one module don't impact others
- **Team Collaboration**: Multiple developers can work on different modules simultaneously
- **Code Reusability**: Shared utilities and templates can be easily accessed

#### 2. **Factory Pattern (App Factory)**
- **Testability**: Easy to create multiple app instances for testing
- **Configuration Management**: Different configurations for development, testing, and production
- **Extension Initialization**: Clean separation of extension setup

#### 3. **Template Inheritance**
- **DRY Principle**: Base templates eliminate code duplication
- **Consistent UI**: All pages inherit the same base structure
- **Easy Theming**: Theme changes in base template reflect across all pages
- **Component Reusability**: Headers, sidebars, and footers are reusable components

#### 4. **Database Migrations**
- **Version Control**: Track database schema changes
- **Rollback Capability**: Easy to revert database changes
- **Team Synchronization**: All team members can apply the same migrations
- **Production Safety**: Tested migrations before applying to production

#### 5. **Environment-Based Configuration**
- **Security**: Sensitive data (API keys, passwords) stored in environment variables
- **Flexibility**: Different configurations for different environments
- **No Hardcoding**: Credentials not stored in source code

---

## Database Architecture (DBMS)

### Database System
- **Type**: MySQL (Relational Database Management System)
- **Hosting**: Aiven Cloud (Managed MySQL service)
- **Connection**: PyMySQL connector
- **ORM**: SQLAlchemy (via Flask-SQLAlchemy)

### Database Schema

#### 1. **users** Table
Primary user authentication and profile table.

| Column          | Type         | Constraints                 | Description                                  |
| --------------- | ------------ | --------------------------- | -------------------------------------------- |
| `uid`           | INTEGER      | PRIMARY KEY, AUTO_INCREMENT | Unique user identifier                       |
| `username`      | VARCHAR(100) | UNIQUE, NOT NULL            | User's login username                        |
| `password`      | VARCHAR(80)  | NOT NULL                    | Bcrypt hashed password                       |
| `is_customer`   | BOOLEAN      | DEFAULT FALSE               | Flag indicating customer role                |
| `is_vendor`     | BOOLEAN      | DEFAULT FALSE               | Flag indicating vendor role                  |
| `last_role`     | VARCHAR(20)  | NULLABLE                    | Last logged-in role (for session management) |
| `name`          | VARCHAR(50)  | NULLABLE                    | User's display name                          |
| `profile_img`   | VARCHAR(500) | NULLABLE                    | Cloudinary URL for profile image             |
| `business_imgs` | JSON         | NULLABLE                    | Array of Cloudinary URLs for business images |

**Relationships:**
- One-to-One with `customer` table
- One-to-One with `vendor` table
- One-to-Many with `note` table

#### 2. **customer** Table
Customer-specific profile information.

| Column      | Type         | Constraints                       | Description              |
| ----------- | ------------ | --------------------------------- | ------------------------ |
| `id`        | INTEGER      | PRIMARY KEY, AUTO_INCREMENT       | Customer profile ID      |
| `user_id`   | INTEGER      | FOREIGN KEY → users.uid, NOT NULL | Reference to users table |
| `full_name` | VARCHAR(100) | NULLABLE                          | Customer's full name     |
| `address`   | VARCHAR(200) | NULLABLE                          | Customer's address       |
| `phone`     | VARCHAR(20)  | NULLABLE                          | Customer's phone number  |

**Relationships:**
- Many-to-One with `users` table (via `user_id`)

#### 3. **vendor** Table
Vendor-specific business information.

| Column                  | Type         | Constraints                       | Description                 |
| ----------------------- | ------------ | --------------------------------- | --------------------------- |
| `id`                    | INTEGER      | PRIMARY KEY, AUTO_INCREMENT       | Vendor profile ID           |
| `user_id`               | INTEGER      | FOREIGN KEY → users.uid, NOT NULL | Reference to users table    |
| `name`                  | VARCHAR(150) | NOT NULL                          | Vendor's name               |
| `email`                 | VARCHAR(120) | NULLABLE                          | Business email              |
| `business_name`         | VARCHAR(100) | NULLABLE                          | Name of the business        |
| `business_address`      | VARCHAR(200) | NULLABLE                          | Business location           |
| `phone_number`          | VARCHAR(20)  | NOT NULL                          | Primary contact number      |
| `whatsapp_number`       | VARCHAR(20)  | NULLABLE                          | WhatsApp contact            |
| `open_duration`         | VARCHAR(100) | NULLABLE                          | Business hours              |
| `payment_type`          | VARCHAR(50)  | NULLABLE                          | Accepted payment methods    |
| `year_of_establishment` | INTEGER      | NULLABLE                          | Business establishment year |
| `rating`                | FLOAT        | NULLABLE                          | Average customer rating     |
| `rater_no`              | INTEGER      | NULLABLE                          | Number of ratings received  |

**Relationships:**
- Many-to-One with `users` table (via `user_id`)

#### 4. **note** Table
User notes (utility feature).

| Column         | Type     | Constraints                       | Description              |
| -------------- | -------- | --------------------------------- | ------------------------ |
| `id`           | INTEGER  | PRIMARY KEY, AUTO_INCREMENT       | Note ID                  |
| `content`      | TEXT     | NOT NULL                          | Note content             |
| `date_created` | DATETIME | DEFAULT CURRENT_TIMESTAMP         | Creation timestamp       |
| `user_id`      | INTEGER  | FOREIGN KEY → users.uid, NOT NULL | Reference to users table |

**Relationships:**
- Many-to-One with `users` table (via `user_id`)

### Database Design Principles

1. **Normalization**: Tables are normalized to 3NF (Third Normal Form) to eliminate redundancy
2. **Referential Integrity**: Foreign key constraints ensure data consistency
3. **Cascade Deletes**: When a user is deleted, associated customer/vendor profiles are automatically deleted
4. **Flexible Role System**: Users can have both customer and vendor roles simultaneously
5. **JSON Storage**: Business images stored as JSON array for flexibility

### Migration History

The database schema has evolved through multiple migrations:
1. **f9d7a9142b82**: Initial schema creation (users, customer, vendor, note tables)
2. **919b857b1ba5**: Added image_url and business_imgs to users table
3. **e236dde75f84**: Added name field to users table
4. **788d22f93a1e**: Updated vendor table structure (removed shop_name, gst_number; added comprehensive fields)
5. **623259c255d7**: Replaced image_url with profile_img in users table

---

## System Design

### Application Flow

```
┌─────────────────┐
│   User Visits   │
│   Homepage      │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  Guest/Logged   │
│  In Decision    │
└────────┬────────┘
         │
    ┌────┴────┐
    │         │
    ▼         ▼
┌────────┐ ┌──────────┐
│ Guest  │ │ Logged   │
│ Access │ │ In User  │
└───┬────┘ └────┬─────┘
    │           │
    │      ┌────┴────┐
    │      │         │
    │      ▼         ▼
    │  ┌────────┐ ┌─────────┐
    │  │Customer│ │ Vendor  │
    │  │ Portal │ │ Portal  │
    │  └───┬────┘ └────┬─────┘
    │      │           │
    └──────┴───────────┘
           │
           ▼
    ┌─────────────┐
    │ Browse/Book │
    │ Vendors     │
    └─────────────┘
```

### Request-Response Cycle

1. **User Request** → Flask receives HTTP request
2. **Route Matching** → Flask matches URL to blueprint route
3. **Authentication Check** → `@login_required` decorator verifies user session
4. **Role Verification** → Checks if user has required role (customer/vendor)
5. **Business Logic** → Route handler processes request
6. **Database Query** → SQLAlchemy queries database if needed
7. **Template Rendering** → Jinja2 renders HTML template
8. **Response** → Flask sends HTML response to client

### Component Interaction

```
┌──────────────┐
│   Browser    │
└──────┬───────┘
       │ HTTP Request
       ▼
┌──────────────┐
│   Flask App  │
│  (run.py)    │
└──────┬───────┘
       │
       ├──► Blueprints (Routes)
       │    ├── auth
       │    ├── vendor
       │    ├── customer
       │    └── appointment
       │
       ├──► Models (SQLAlchemy)
       │    ├── User
       │    ├── Customer
       │    ├── Vendor
       │    └── Note
       │
       ├──► Extensions
       │    ├── Database (SQLAlchemy)
       │    ├── Login Manager
       │    └── Bcrypt
       │
       └──► Templates (Jinja2)
            └── HTML + Tailwind CSS
```

### Security Layers

1. **Password Hashing**: Bcrypt with salt rounds
2. **Session Management**: Flask-Login handles secure sessions
3. **Role-Based Access Control**: Routes check user roles before access
4. **SQL Injection Prevention**: SQLAlchemy ORM prevents SQL injection
5. **CSRF Protection**: Flask-WTF can be added for form protection
6. **Environment Variables**: Sensitive data not in source code

---

## Design Policies & Best Practices

### 1. **Separation of Concerns**
- **Models**: Data structure and database operations
- **Routes**: Request handling and business logic
- **Templates**: Presentation layer
- **Extensions**: Reusable components

### 2. **DRY (Don't Repeat Yourself)**
- Base templates for common HTML structure
- Reusable header and sidebar components
- Shared utility functions

### 3. **Single Responsibility Principle**
- Each blueprint handles one domain (auth, vendor, customer)
- Each route function has a single, clear purpose
- Models represent single entities

### 4. **Factory Pattern**
- `create_app()` function allows multiple app instances
- Easy configuration management
- Testable architecture

### 5. **Dependency Injection**
- Extensions initialized separately and injected into app
- Database, login manager, and bcrypt initialized in extensions.py

### 6. **Error Handling**
- Flash messages for user feedback
- 404 errors for missing resources (`get_or_404`)
- 403 errors for unauthorized access

### 7. **Code Organization**
- Blueprints for modular routing
- Templates organized by feature
- Migrations version controlled

### 8. **Security Best Practices**
- Passwords never stored in plain text
- Environment variables for secrets
- Role-based access control
- Session management with Flask-Login

### 9. **Database Best Practices**
- Migrations for schema changes
- Foreign key constraints for integrity
- Cascade deletes for data consistency
- Indexed primary keys for performance

### 10. **Frontend Best Practices**
- Mobile-first responsive design
- Semantic HTML
- Accessible UI components
- Consistent design system

---

## Project Design Theme

### Color Palette

The application follows a carefully chosen color scheme for consistency and professionalism:

- **Primary Color**: `#1D4ED8` (Blue-700)
  - Used for: Buttons, links, active states, brand elements
  - Represents: Trust, professionalism, reliability

- **Secondary Color**: `#4B5563` (Gray-600)
  - Used for: Secondary text, borders, subtle elements
  - Represents: Neutrality, balance

- **Background**: `#F3F4F6` (Gray-100)
  - Used for: Page backgrounds, card backgrounds
  - Represents: Clean, modern aesthetic

- **Surface**: `#FFFFFF` (White)
  - Used for: Cards, containers, content areas
  - Represents: Clarity, focus

- **Footer**: `#111827` (Gray-900)
  - Used for: Footer background
  - Represents: Stability, professionalism

### Typography

- **Font Family**: Inter (Google Fonts)
- **Font Weights**: 400 (Regular), 500 (Medium), 600 (Semibold), 700 (Bold)
- **Characteristics**: Modern, clean, highly readable

### Design Principles

1. **Minimalism**: Clean, uncluttered interfaces
2. **Flat Design**: No heavy shadows or gradients
3. **Consistency**: Same design patterns across all pages
4. **Mobile-First**: Responsive design starting from mobile screens
5. **Accessibility**: Semantic HTML and proper contrast ratios

### UI Components

#### Buttons
- **Primary**: Blue background (`bg-primary`), white text, rounded corners
- **Secondary**: Gray background, rounded corners
- **Hover States**: Darker shade on hover for feedback

#### Cards
- White background
- Rounded corners (`rounded-xl`)
- Subtle shadows (`shadow-md`)
- Border for definition (`border border-gray-200`)

#### Forms
- Clean input fields with borders
- Focus states with ring effects
- Consistent spacing and padding

#### Navigation
- Sticky header for easy access
- Sidebar for authenticated users
- Breadcrumbs where applicable

### Responsive Breakpoints

Following Tailwind CSS defaults:
- **sm**: 640px (Small tablets)
- **md**: 768px (Tablets)
- **lg**: 1024px (Laptops)
- **xl**: 1280px (Desktops)

---

## Business Logic

### User Registration Flow

1. **User visits signup page** (`/auth/signup`)
2. **User provides**: username, password, role (customer/vendor)
3. **System checks**: If username already exists
   - **If new user**: Creates account with selected role
   - **If existing user**: 
     - Validates password
     - If correct password and role not assigned: Adds role to existing account
     - If role already assigned: Shows info message
     - If incorrect password: Shows error
4. **Redirects to login page** with appropriate flash message

### User Login Flow

1. **User visits login page** (`/auth/login`)
2. **User provides**: username, password, role (customer/vendor)
3. **System validates**:
   - Checks if username exists
   - Verifies password using bcrypt
   - **Critical**: Checks if user is registered for selected role
     - If user tries to login as vendor but `is_vendor = False`: Redirects to signup
     - If user tries to login as customer but `is_customer = False`: Redirects to signup
4. **If valid**:
   - Creates session using Flask-Login
   - Sets `last_role` to remember current role
   - Checks if profile exists:
     - **Customer**: If no customer profile → redirects to profile setup
     - **Vendor**: If no vendor profile → redirects to profile setup
     - If profile exists → redirects to respective dashboard

### Role-Based Access Control

**Key Implementation Details:**

1. **Dual Role Support**: Users can have both `is_customer = True` and `is_vendor = True`
2. **Role Verification on Login**: Users can ONLY login with a role they are registered for
3. **Session-Based Role**: `last_role` field tracks which role user logged in with
4. **Route Protection**: 
   - `@login_required` ensures user is authenticated
   - Additional checks like `if current_user.is_vendor != True: return 403` protect vendor routes
   - Similar checks for customer routes

### Vendor Profile Setup

1. **Trigger**: After vendor login, if no vendor profile exists
2. **User provides**:
   - Name, email
   - Business name, business address
   - Phone number, WhatsApp number
   - Open duration (business hours)
   - Payment type
   - Year of establishment
   - Business images (multiple)
3. **System processes**:
   - Uploads images to Cloudinary in folder: `vendora/{user_id}/business_img/`
   - Stores image URLs in `user.business_imgs` JSON array
   - Creates Vendor record linked to User
   - Sets initial rating to 0, rater_no to 0
4. **Redirects to vendor dashboard**

### Customer Profile Setup

1. **Trigger**: After customer login, if no customer profile exists
2. **User provides**:
   - Full name
   - Address
   - Phone number
3. **System processes**:
   - Creates Customer record linked to User
4. **Redirects to customer dashboard**

### Vendor Discovery (Customer Flow)

1. **Customer visits homepage** (`/customer/homepage` or `/`)
2. **System displays**: List of all vendors with:
   - Business name
   - Business images (first image from array)
   - Business address
   - Rating (if available)
3. **Search functionality**: 
   - Customer can search by business name
   - Case-insensitive search using `ilike` operator
4. **Vendor details**: Customer can click to view full vendor profile

### Profile Update Flow

**User Profile Update** (`/auth/profile_update`):
- Updates name
- Uploads/updates profile image to Cloudinary
- Stores in `user.profile_img`

**Vendor Profile Update** (`/vendor/profile_update`):
- Updates all vendor fields
- Can add additional business images
- Images appended to existing `business_imgs` array

### Image Management

**Cloudinary Integration:**
- **Profile Images**: Stored at `vendora/{user_id}/profile_img`
- **Business Images**: Stored at `vendora/{user_id}/business_img/img_{index}`
- **Overwrite Policy**: Profile images overwrite, business images append
- **Storage Format**: URLs stored in database, not files

### Account Deletion

1. **User initiates deletion** (`/auth/delete_account`)
2. **System processes**:
   - Logs out user
   - Deletes user record (cascade deletes customer/vendor profiles)
   - Clears session
3. **Redirects to homepage**

### Notes Feature

- Users can create notes (`/auth/new_note`)
- Notes are linked to user via `user_id`
- Displayed in list format (`/auth/show_notes`)

---

## Authentication Implementation

### Authentication System Overview

Vendora uses **Flask-Login** for session management and **Flask-Bcrypt** for password security, implementing a robust role-based authentication system.

### Password Security

**Implementation:**
```python
# Password Hashing (Signup)
hashed_password = bcrypt.generate_password_hash(password)

# Password Verification (Login)
bcrypt.check_password_hash(user.password, password)
```

**Security Features:**
- **Bcrypt Hashing**: Passwords are hashed using bcrypt algorithm
- **Salt Rounds**: Automatic salt generation for each password
- **One-Way Hashing**: Passwords cannot be reversed
- **No Plain Text Storage**: Passwords never stored in readable format

### Session Management

**Flask-Login Integration:**
```python
@login_manager.user_loader
def load_user(uid):
    return User.query.get(uid)
```

**Session Lifecycle:**
1. **Login**: `login_user(user)` creates session
2. **Session Storage**: Flask-Login stores user ID in session cookie
3. **Request Handling**: `@login_required` decorator checks session
4. **User Loading**: `load_user()` retrieves user from database
5. **Logout**: `logout_user()` destroys session

### Role-Based Authentication

**Critical Implementation: Role Verification on Login**

The system ensures users can ONLY login with roles they are registered for:

```python
# Login Route Logic
if role == 'vendor' and not user.is_vendor:
    flash('You are not registered as a vendor.', 'danger')
    return redirect(url_for('auth.signup'))
elif role == 'customer' and not user.is_customer:
    flash('You are not registered as a customer.', 'danger')
    return redirect(url_for('auth.signup'))
```

**How It Works:**
1. User selects role during login (customer or vendor)
2. System checks `user.is_customer` or `user.is_vendor` flag
3. **If flag is False**: User is redirected to signup with error message
4. **If flag is True**: Login proceeds

**This prevents:**
- Customers from accessing vendor dashboard
- Vendors from accessing customer dashboard
- Unauthorized role access

### User Model Implementation

**UserMixin Integration:**
```python
class User(db.Model, UserMixin):
    def get_id(self):
        return self.uid
```

**Required Methods:**
- `get_id()`: Returns unique identifier for session
- `is_authenticated`: Provided by UserMixin
- `is_active`: Provided by UserMixin
- `is_anonymous`: Provided by UserMixin

### Protected Routes

**Implementation:**
```python
@login_required
def dashboard():
    if current_user.is_vendor != True:
        return "Access denied", 403
    # Route logic
```

**Protection Levels:**
1. **Authentication Required**: `@login_required` decorator
2. **Role Verification**: Manual checks for `is_vendor` or `is_customer`
3. **Profile Verification**: Checks if profile exists before access

### Multi-Role Support

**Feature**: Users can have both customer and vendor roles

**Implementation:**
- `is_customer` and `is_vendor` are independent boolean flags
- User can register for both roles
- `last_role` tracks which role user logged in with
- Session maintains current role context

**Use Case:**
- A barber (vendor) might also want to book appointments with other vendors (customer)
- Single account, dual functionality

### Profile-Based Access

**Profile Completion Flow:**
1. User logs in successfully
2. System checks if profile exists:
   - Customer: `user.customer_profile`
   - Vendor: `user.vendor_profile`
3. **If no profile**: Redirects to profile setup
4. **If profile exists**: Redirects to dashboard

**This ensures:**
- Users complete their profiles before accessing features
- Data integrity (profiles required for core functionality)

### Logout Implementation

```python
@auth.route('/logout')
def logout():
    logout_user()
    session.clear()
    return redirect(url_for('customer.vendors'))
```

**Process:**
1. `logout_user()`: Removes user from session
2. `session.clear()`: Clears all session data
3. Redirects to public page

### Security Best Practices Implemented

1. ✅ **Password Hashing**: Bcrypt with salt
2. ✅ **Session Security**: Flask-Login secure sessions
3. ✅ **Role Verification**: Strict role checking
4. ✅ **Route Protection**: Decorators and manual checks
5. ✅ **Environment Variables**: Secrets not in code
6. ✅ **SQL Injection Prevention**: ORM usage
7. ⚠️ **CSRF Protection**: Can be added with Flask-WTF
8. ⚠️ **Rate Limiting**: Can be added for login attempts

---

## Features

### 1. User Authentication & Authorization

#### Signup
- **Route**: `/auth/signup`
- **Features**:
  - Username and password registration
  - Role selection (Customer/Vendor)
  - Existing user role addition
  - Password validation
  - Flash message feedback

#### Login
- **Route**: `/auth/login`
- **Features**:
  - Username and password authentication
  - Role-based login (strict verification)
  - Automatic profile setup redirect
  - Session creation
  - Last role tracking

#### Logout
- **Route**: `/auth/logout`
- **Features**:
  - Session termination
  - Complete session cleanup
  - Redirect to homepage

#### Profile Management
- **Route**: `/auth/profile_update`
- **Features**:
  - Name update
  - Profile image upload (Cloudinary)
  - Image overwrite capability

#### Account Deletion
- **Route**: `/auth/delete_account`
- **Features**:
  - Complete account removal
  - Cascade deletion of profiles
  - Session cleanup

### 2. Vendor Features

#### Vendor Dashboard
- **Route**: `/vendor/dashboard`
- **Features**:
  - Vendor profile overview
  - Business information display
  - Access control (vendor-only)

#### Vendor Profile Setup
- **Route**: `/vendor/profile_setup`
- **Features**:
  - Complete business information form
  - Multiple business image uploads
  - Cloudinary integration
  - Initial rating setup (0.0)

#### Vendor Profile Update
- **Route**: `/vendor/profile_update`
- **Features**:
  - Edit all business fields
  - Add additional business images
  - Image management

#### Vendor Appointments
- **Route**: `/vendor/appointments`
- **Features**:
  - View appointments (UI ready)
  - Appointment management interface
  - *Note: Backend logic can be extended*

#### Vendor Analytics
- **Route**: `/vendor/analytics`
- **Features**:
  - Analytics dashboard (UI ready)
  - *Note: Analytics logic can be implemented*

#### Vendor Reviews
- **Route**: `/vendor/reviews`
- **Features**:
  - Reviews display (UI ready)
  - Rating system foundation
  - *Note: Review submission can be added*

#### Vendor Support
- **Route**: `/vendor/support`
- **Features**:
  - Support page (UI ready)
  - *Note: Support ticket system can be added*

### 3. Customer Features

#### Customer Dashboard
- **Route**: `/customer/dashboard`
- **Features**:
  - Customer profile overview
  - Access control (customer-only)

#### Vendor Discovery (Homepage)
- **Route**: `/customer/homepage` or `/`
- **Features**:
  - List all vendors
  - Search functionality (business name)
  - Vendor cards with images
  - Business information preview
  - Responsive grid layout

#### Vendor Details
- **Route**: `/customer/vendor/<vendor_id>`
- **Features**:
  - Complete vendor profile view
  - Business images gallery
  - Contact information
  - Business hours
  - Rating display
  - *Note: Appointment booking can be added*

#### Customer Profile Setup
- **Route**: `/customer/profile_setup`
- **Features**:
  - Customer information form
  - Name, address, phone
  - One-time setup

### 4. Core Features

#### Homepage
- **Route**: `/`
- **Features**:
  - Redirects to vendor discovery page
  - Public access

#### About Page
- **Route**: `/about`
- **Features**:
  - Platform information
  - Public access

#### Developer Page
- **Route**: `/developer`
- **Features**:
  - Route listing
  - Development status
  - Project information

### 5. Utility Features

#### Notes System
- **Routes**: `/auth/new_note`, `/auth/show_notes`
- **Features**:
  - Create personal notes
  - View all user notes
  - Note history

### 6. Image Management

#### Profile Images
- **Storage**: Cloudinary
- **Path**: `vendora/{user_id}/profile_img`
- **Features**:
  - Single image per user
  - Overwrite on update
  - Secure URL storage

#### Business Images
- **Storage**: Cloudinary
- **Path**: `vendora/{user_id}/business_img/img_{index}`
- **Features**:
  - Multiple images per vendor
  - Append capability
  - JSON array storage
  - Indexed naming

### 7. Search & Discovery

#### Vendor Search
- **Implementation**: Case-insensitive search
- **Method**: SQL `ilike` operator
- **Scope**: Business name
- **Features**:
  - Real-time filtering
  - Partial match support

### Feature Status

**✅ Fully Implemented:**
- User authentication (signup, login, logout)
- Role-based access control
- Vendor profile management
- Customer profile management
- Vendor discovery
- Image upload (Cloudinary)
- Search functionality
- Profile updates

**🚧 UI Ready (Backend Extendable):**
- Appointment management
- Analytics dashboard
- Reviews system
- Support page

**📋 Planned/Extendable:**
- Appointment booking backend
- Review submission
- Rating calculation
- Analytics data collection
- Support ticket system
- Email notifications
- SMS notifications
- Payment integration

---

## Deployment & Cloud Infrastructure

### Deployment Platform: Vercel

**Configuration File**: `vercel.json`

```json
{
    "version": 2,
    "builds": [
        {
            "src": "run.py",
            "use": "@vercel/python"
        }
    ],
    "routes": [
        {
            "src": "/(.*)",
            "dest": "run.py"
        }
    ]
}
```

**Deployment Process:**
1. Application entry point: `run.py`
2. Vercel Python runtime handles Flask app
3. All routes proxy to Flask application
4. Serverless function deployment

### Database: Aiven Cloud MySQL

**Connection Details:**
- **Host**: `mysql-3b0838a6-priyamjainsocial-b642.i.aivencloud.com`
- **Port**: `27509`
- **Database**: `defaultdb`
- **User**: `avnadmin`
- **Password**: Stored in environment variable `DB_PASSWORD`

**Connection String Format:**
```
mysql+pymysql://avnadmin:{DB_PASSWORD}@mysql-3b0838a6-priyamjainsocial-b642.i.aivencloud.com:27509/defaultdb
```

**Benefits:**
- Managed service (no server maintenance)
- Automatic backups
- High availability
- SSL encryption
- Scalable infrastructure

### Image Storage: Cloudinary

**Configuration:**
```python
cloudinary.config(
    cloud_name = os.getenv('c_cloud_name'),
    api_key    = os.getenv('c_api_key'),
    api_secret = os.getenv('c_api_secret')
)
```

**Storage Structure:**
- **Profile Images**: `vendora/{user_id}/profile_img`
- **Business Images**: `vendora/{user_id}/business_img/img_{index}`

**Features:**
- CDN delivery (fast image loading)
- Automatic optimization
- Transformations available
- Secure URLs
- Overwrite capability

### Environment Variables

**Required Variables:**
- `DB_PASSWORD`: MySQL database password
- `c_cloud_name`: Cloudinary cloud name
- `c_api_key`: Cloudinary API key
- `c_api_secret`: Cloudinary API secret
- `SECRET_KEY`: Flask session secret (currently hardcoded, should be in env)

**Security Note:**
- All sensitive data stored in environment variables
- Never committed to version control
- Different values for development/production

### Deployment Checklist

**Pre-Deployment:**
- [ ] Set all environment variables in Vercel
- [ ] Run database migrations
- [ ] Test database connection
- [ ] Verify Cloudinary credentials
- [ ] Update SECRET_KEY to environment variable

**Deployment Steps:**
1. Push code to repository
2. Vercel automatically detects changes
3. Builds application using `@vercel/python`
4. Deploys serverless functions
5. Routes configured via `vercel.json`

**Post-Deployment:**
- [ ] Verify application is accessible
- [ ] Test authentication flows
- [ ] Test image uploads
- [ ] Monitor error logs
- [ ] Check database connectivity

### Scalability Considerations

**Current Architecture:**
- Serverless deployment (auto-scaling)
- Managed database (scalable)
- CDN for images (global distribution)

**Future Enhancements:**
- Redis for session storage
- Load balancing for high traffic
- Database read replicas
- Caching layer
- Background job processing

### Monitoring & Logging

**Available Tools:**
- Vercel Analytics (can be enabled)
- Aiven Cloud monitoring
- Cloudinary usage analytics
- Flask logging

**Recommended:**
- Error tracking (Sentry)
- Performance monitoring
- User analytics
- Database query monitoring

---

## Quick Navigation Guide

### For Developers

#### Setting Up Development Environment

1. **Clone Repository**
   ```bash
   git clone <repository-url>
   cd vendora2
   ```

2. **Create Virtual Environment**
   ```bash
   python -m venv vendenv
   vendenv\Scripts\activate  # Windows
   # or
   source vendenv/bin/activate  # Linux/Mac
   ```

3. **Install Dependencies**
   ```bash
   pip install -r requirements.txt
   ```

4. **Set Environment Variables**
   Create `.env` file:
   ```
   DB_PASSWORD=your_db_password
   c_cloud_name=your_cloudinary_name
   c_api_key=your_cloudinary_key
   c_api_secret=your_cloudinary_secret
   SECRET_KEY=your_secret_key
   ```

5. **Run Database Migrations**
   ```bash
   flask db upgrade
   ```

6. **Run Application**
   ```bash
   python run.py
   ```

#### Common Development Tasks

**Create Database Migration:**
```bash
flask db migrate -m "description"
flask db upgrade
```

**Access Routes:**
- Homepage: `http://localhost:5000/`
- Login: `http://localhost:5000/auth/login`
- Signup: `http://localhost:5000/auth/signup`
- Vendor Dashboard: `http://localhost:5000/vendor/dashboard`
- Customer Dashboard: `http://localhost:5000/customer/dashboard`

### For Users

#### Customer Journey

1. **Visit Homepage** → Browse vendors
2. **Sign Up** → Select "Customer" role
3. **Login** → Complete profile setup
4. **Discover Vendors** → Search and browse
5. **View Vendor Details** → See business information
6. **Book Appointment** → *Feature to be implemented*

#### Vendor Journey

1. **Sign Up** → Select "Vendor" role
2. **Login** → Complete vendor profile setup
3. **Upload Business Images** → Showcase services
4. **Manage Profile** → Update business information
5. **View Appointments** → Manage bookings
6. **Check Analytics** → Track performance

### Route Reference

#### Authentication Routes
- `GET/POST /auth/signup` - User registration
- `GET/POST /auth/login` - User login
- `GET /auth/logout` - User logout
- `GET/POST /auth/profile_update` - Update user profile
- `POST /auth/delete_account` - Delete account
- `GET/POST /auth/new_note` - Create note
- `GET /auth/show_notes` - View notes

#### Vendor Routes
- `GET /vendor/dashboard` - Vendor dashboard
- `GET/POST /vendor/profile_setup` - Setup vendor profile
- `GET/POST /vendor/profile_update` - Update vendor profile
- `GET /vendor/appointments` - View appointments
- `GET /vendor/analytics` - View analytics
- `GET /vendor/reviews` - View reviews
- `GET /vendor/support` - Support page
- `GET /vendor/delete_profile` - Delete vendor profile

#### Customer Routes
- `GET /customer/homepage` - Browse vendors
- `GET /customer/dashboard` - Customer dashboard
- `GET/POST /customer/profile_setup` - Setup customer profile
- `GET /customer/vendor/<vendor_id>` - View vendor details

#### Core Routes
- `GET /` - Homepage (redirects to vendor list)
- `GET /about` - About page
- `GET /developer` - Developer page

### File Locations Reference

**Models:**
- User, Note: `vendora_app/blueprints/auth/models.py`
- Vendor: `vendora_app/blueprints/vendor/models.py`
- Customer: `vendora_app/blueprints/customer/models.py`

**Routes:**
- Auth: `vendora_app/blueprints/auth/routes.py`
- Vendor: `vendora_app/blueprints/vendor/routes.py`
- Customer: `vendora_app/blueprints/customer/routes.py`
- Core: `vendora_app/blueprints/core/routes.py`

**Templates:**
- Base: `vendora_app/templates/base.html`
- Auth: `vendora_app/blueprints/auth/templates/`
- Vendor: `vendora_app/blueprints/vendor/templates/`
- Customer: `vendora_app/blueprints/customer/templates/`

**Configuration:**
- App Factory: `vendora_app/app.py`
- Extensions: `vendora_app/extensions.py`
- Entry Point: `run.py`
- Deployment: `vercel.json`

---

## Conclusion

Vendora is a well-architected platform that successfully bridges local vendors and customers in India. The application demonstrates:

- **Clean Architecture**: Modular blueprint pattern
- **Security**: Robust authentication and authorization
- **Scalability**: Cloud-based infrastructure
- **User Experience**: Modern, responsive design
- **Maintainability**: Well-organized codebase

The platform is ready for production use with core features implemented, and provides a solid foundation for future enhancements such as appointment booking, payment integration, and advanced analytics.

---

**Documentation Version**: 1.0  
**Last Updated**: 2025  
**Maintained By**: Vendora Development Team

---

*For questions or contributions, please refer to the developer page or contact the development team.*
