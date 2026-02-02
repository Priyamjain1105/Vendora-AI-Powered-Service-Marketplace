# Vendora Documentation

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
12. [Getting Started](#getting-started)
13. [API Routes Reference](#api-routes-reference)

---

## Introduction

**Vendora** is a comprehensive web platform designed specifically for local vendors in India (barbers, mechanics, plumbers, and other service providers) to promote their businesses and connect with customers. The platform enables vendors to create personalized business pages where customers can discover services, view business information, and book appointments seamlessly.

### Mission Statement
To empower local Indian vendors by providing them with a digital presence and helping customers easily discover and book services from trusted local businesses.

---

## Project Overview

### Technology Stack
- **Backend Framework**: Flask (Python)
- **Database**: MySQL (hosted on Aiven Cloud)
- **ORM**: SQLAlchemy
- **Authentication**: Flask-Login with Bcrypt password hashing
- **Database Migrations**: Flask-Migrate (Alembic)
- **File Storage**: Cloudinary (for image uploads)
- **Frontend**: HTML5, Tailwind CSS v3, JavaScript
- **Font**: Inter (Google Fonts)
- **Deployment**: Vercel

### Key Capabilities
- Dual-role user system (Customer & Vendor)
- Secure authentication with role-based access control
- Vendor profile management with business information
- Customer profile management
- Vendor discovery and search functionality
- Business image gallery management
- Profile image management
- Cloud-based image storage
- Responsive mobile-first design

---

## File Architecture

### Project Structure

```
vendora2/
├── vendora_app/                    # Main application package
│   ├── __init__.py                 # Package initialization
│   ├── app.py                      # Flask app factory & configuration
│   ├── extensions.py               # Extension initialization (DB, Login, Bcrypt)
│   ├── models.py                   # Shared models (if any)
│   ├── templates/                  # Base templates
│   │   ├── base.html              # Main base template
│   │   ├── authbase.html          # Authentication base template
│   │   ├── theme.md               # Design theme documentation
│   │   ├── headers/               # Header components
│   │   │   ├── guest_header.html
│   │   │   └── user_header.html
│   │   └── sidebar/               # Sidebar components
│   │       ├── sidebar_base.html
│   │       ├── sidebar_customer.html
│   │       └── sidebar_vendor.html
│   ├── blueprints/                # Modular blueprint structure
│   │   ├── auth/                  # Authentication module
│   │   │   ├── __init__.py
│   │   │   ├── models.py          # User & Note models
│   │   │   ├── routes.py          # Auth routes (login, signup, logout)
│   │   │   └── templates/         # Auth templates
│   │   ├── vendor/                # Vendor module
│   │   │   ├── __init__.py
│   │   │   ├── models.py          # Vendor model
│   │   │   ├── routes.py          # Vendor routes
│   │   │   └── templates/         # Vendor templates
│   │   ├── customer/              # Customer module
│   │   │   ├── __init__.py
│   │   │   ├── models.py          # Customer model
│   │   │   ├── routes.py          # Customer routes
│   │   │   └── templates/         # Customer templates
│   │   ├── appointment/           # Appointment module
│   │   │   ├── __init__.py
│   │   │   ├── model.py           # Appointment model (if exists)
│   │   │   ├── routes.py          # Appointment routes
│   │   │   └── templates/         # Appointment templates
│   │   └── core/                  # Core/utility routes
│   │       ├── __init__.py
│   │       ├── routes.py          # Home, about, developer pages
│   │       └── templates/         # Core templates
│   └── migrations/                # Database migrations
│       ├── versions/              # Migration scripts
│       ├── env.py                 # Alembic environment
│       └── alembic.ini            # Alembic configuration
├── run.py                         # Application entry point
├── config.py                      # Configuration file (if exists)
├── requirements.txt               # Python dependencies
├── vercel.json                    # Vercel deployment configuration
├── .env                           # Environment variables (not in repo)
└── readme.md                      # This documentation file
```

### Architecture Benefits

#### 1. **Blueprint Pattern (Modular Architecture)**
   - **Separation of Concerns**: Each feature (auth, vendor, customer, appointment) is isolated in its own blueprint
   - **Scalability**: Easy to add new features without modifying existing code
   - **Maintainability**: Changes to one module don't affect others
   - **Team Collaboration**: Multiple developers can work on different blueprints simultaneously
   - **Code Reusability**: Blueprints can be reused or extracted for other projects

#### 2. **Template Inheritance**
   - **DRY Principle**: Base templates (`base.html`, `authbase.html`) eliminate code duplication
   - **Consistent UI**: All pages inherit common structure (header, footer, styling)
   - **Easy Updates**: Changes to base templates automatically reflect across all pages
   - **Component-Based**: Headers and sidebars are modular components

#### 3. **Model Separation**
   - **Single Responsibility**: Each blueprint has its own models file
   - **Clear Relationships**: Models are co-located with their routes
   - **Easy Navigation**: Developers can quickly find related code

#### 4. **Migration System**
   - **Version Control**: Database schema changes are tracked
   - **Rollback Capability**: Can revert to previous database states
   - **Team Synchronization**: All developers can apply same migrations
   - **Production Safety**: Controlled database updates

#### 5. **Extension Pattern**
   - **Lazy Initialization**: Extensions initialized before app creation
   - **Testability**: Easy to create test instances with different configurations
   - **Flexibility**: Can swap implementations (e.g., different databases)

---

## Database Architecture (DBMS)

### Database System
- **Type**: MySQL (Relational Database Management System)
- **Hosting**: Aiven Cloud
- **Connection**: PyMySQL driver
- **ORM**: SQLAlchemy (Flask-SQLAlchemy)

### Database Schema

#### 1. **users** Table (Core User Account)
```sql
CREATE TABLE users (
    uid INTEGER PRIMARY KEY AUTO_INCREMENT,
    username VARCHAR(100) UNIQUE NOT NULL,
    password VARCHAR(80) NOT NULL,
    is_customer BOOLEAN DEFAULT FALSE,
    is_vendor BOOLEAN DEFAULT FALSE,
    last_role VARCHAR(20),
    name VARCHAR(50),
    profile_img VARCHAR(500),
    business_imgs JSON
);
```

**Purpose**: Central authentication table storing user credentials and role information.

**Key Features**:
- Single account can have both customer and vendor roles (`is_customer`, `is_vendor`)
- `last_role` tracks the most recently used role for session management
- `profile_img` stores Cloudinary URL for user profile picture
- `business_imgs` stores JSON array of business image URLs (for vendors)

**Relationships**:
- One-to-One with `customer` table
- One-to-One with `vendor` table
- One-to-Many with `note` table

---

#### 2. **customer** Table
```sql
CREATE TABLE customer (
    id INTEGER PRIMARY KEY AUTO_INCREMENT,
    user_id INTEGER NOT NULL,
    full_name VARCHAR(100),
    address VARCHAR(200),
    phone VARCHAR(20),
    FOREIGN KEY (user_id) REFERENCES users(uid) ON DELETE CASCADE
);
```

**Purpose**: Stores customer-specific profile information.

**Relationships**:
- Foreign Key to `users.uid` (One-to-One relationship)
- Cascade delete: If user is deleted, customer profile is automatically deleted

---

#### 3. **vendor** Table
```sql
CREATE TABLE vendor (
    id INTEGER PRIMARY KEY AUTO_INCREMENT,
    user_id INTEGER NOT NULL,
    name VARCHAR(150) NOT NULL,
    email VARCHAR(120),
    business_name VARCHAR(100),
    business_address VARCHAR(200),
    phone_number VARCHAR(20) NOT NULL,
    whatsapp_number VARCHAR(20),
    open_duration VARCHAR(100),
    payment_type VARCHAR(50),
    year_of_establishment INTEGER,
    rating FLOAT,
    rater_no INTEGER,
    FOREIGN KEY (user_id) REFERENCES users(uid) ON DELETE CASCADE
);
```

**Purpose**: Stores comprehensive vendor business information.

**Key Fields**:
- `rating` & `rater_no`: For future review/rating system
- `open_duration`: Business hours (stored as string, e.g., "9 AM - 6 PM")
- `payment_type`: Accepted payment methods
- `whatsapp_number`: Important for Indian market communication

**Relationships**:
- Foreign Key to `users.uid` (One-to-One relationship)
- Cascade delete: If user is deleted, vendor profile is automatically deleted

---

#### 4. **note** Table
```sql
CREATE TABLE note (
    id INTEGER PRIMARY KEY AUTO_INCREMENT,
    content TEXT NOT NULL,
    date_created DATETIME DEFAULT CURRENT_TIMESTAMP,
    user_id INTEGER NOT NULL,
    FOREIGN KEY (user_id) REFERENCES users(uid)
);
```

**Purpose**: User notes/reminders feature (currently implemented but may be expanded).

**Relationships**:
- Foreign Key to `users.uid` (One-to-Many relationship)

---

### Database Design Principles

#### 1. **Normalization**
   - **3NF Compliance**: Tables are normalized to eliminate redundancy
   - **User-Centric Design**: `users` table is central, with role-specific tables linked

#### 2. **Referential Integrity**
   - **Foreign Keys**: All relationships enforced at database level
   - **Cascade Deletes**: Ensures data consistency when users are deleted

#### 3. **Flexibility**
   - **Dual Roles**: Single user account can be both customer and vendor
   - **JSON Storage**: `business_imgs` uses JSON for flexible image array storage

#### 4. **Scalability Considerations**
   - **Indexed Fields**: `username` is unique and indexed for fast lookups
   - **Rating System**: `rating` and `rater_no` fields ready for review system
   - **Extensible Schema**: Easy to add new fields via migrations

---

## System Design

### Application Flow

```
┌─────────────────────────────────────────────────────────────┐
│                    User Access Flow                         │
└─────────────────────────────────────────────────────────────┘

1. Guest User
   ├── Browse vendors (public)
   ├── View vendor details (public)
   └── Register/Login required for booking

2. Customer Flow
   ├── Register/Login (as customer)
   ├── Complete customer profile (if first time)
   ├── Browse vendors
   ├── Search vendors
   ├── View vendor details
   └── Book appointments (future feature)

3. Vendor Flow
   ├── Register/Login (as vendor)
   ├── Complete vendor profile (if first time)
   ├── Dashboard (business overview)
   ├── Manage profile
   ├── Upload business images
   ├── View appointments (future)
   ├── Analytics (future)
   └── Reviews (future)
```

### Component Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                      Flask Application                       │
├─────────────────────────────────────────────────────────────┤
│                                                               │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │   Core BP    │  │   Auth BP    │  │  Vendor BP   │      │
│  │  (Home, etc) │  │ (Login, etc) │  │ (Dashboard)  │      │
│  └──────────────┘  └──────────────┘  └──────────────┘      │
│                                                               │
│  ┌──────────────┐  ┌──────────────┐                         │
│  │ Customer BP  │  │ Appointment  │                         │
│  │ (Browse, etc)│  │     BP       │                         │
│  └──────────────┘  └──────────────┘                         │
│                                                               │
├─────────────────────────────────────────────────────────────┤
│                    Extensions Layer                          │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐                 │
│  │    DB    │  │  Login    │  │  Bcrypt  │                 │
│  │(SQLAlch) │  │  Manager  │  │ (Hash)   │                 │
│  └──────────┘  └──────────┘  └──────────┘                 │
├─────────────────────────────────────────────────────────────┤
│                    External Services                         │
│  ┌──────────────┐  ┌──────────────┐                        │
│  │   Cloudinary  │  │  MySQL DB    │                        │
│  │ (Image Store) │  │  (Aiven)     │                        │
│  └──────────────┘  └──────────────┘                        │
└─────────────────────────────────────────────────────────────┘
```

### Request-Response Flow

1. **User Request** → Flask Router
2. **Route Matching** → Blueprint Route Handler
3. **Authentication Check** → `@login_required` decorator (if needed)
4. **Business Logic** → Route function
5. **Database Query** → SQLAlchemy ORM
6. **Template Rendering** → Jinja2 Templates
7. **Response** → HTML to User

---

## Design Policies & Best Practices

### 1. **Security Best Practices**

#### Password Security
- **Bcrypt Hashing**: All passwords are hashed using Flask-Bcrypt
- **No Plain Text Storage**: Passwords never stored in plain text
- **Salt Rounds**: Bcrypt automatically handles salting

#### Authentication
- **Session Management**: Flask-Login handles secure session management
- **Role-Based Access**: Routes protected with `@login_required` and role checks
- **Secure Redirects**: Users redirected based on their role and profile completion status

#### SQL Injection Prevention
- **ORM Usage**: SQLAlchemy ORM prevents SQL injection
- **Parameterized Queries**: All queries use ORM methods, not raw SQL

#### Environment Variables
- **Sensitive Data**: Database credentials and API keys stored in `.env` file
- **Not in Repository**: `.env` file excluded from version control

---

### 2. **Code Organization Best Practices**

#### Separation of Concerns
- **Models**: Database models separated by feature (auth, vendor, customer)
- **Routes**: Business logic in route handlers
- **Templates**: Presentation layer separated from logic

#### DRY (Don't Repeat Yourself)
- **Template Inheritance**: Base templates eliminate duplication
- **Reusable Components**: Headers and sidebars are modular
- **Shared Extensions**: Database, login manager initialized once

#### Single Responsibility Principle
- **Blueprint Isolation**: Each blueprint handles one feature domain
- **Model Relationships**: Clear, well-defined relationships

---

### 3. **Database Best Practices**

#### Migrations
- **Version Control**: All schema changes tracked via Alembic migrations
- **Incremental Updates**: Each migration is a small, focused change
- **Rollback Support**: Can revert migrations if needed

#### Relationships
- **Cascade Deletes**: Proper cleanup when parent records deleted
- **Foreign Key Constraints**: Data integrity enforced at database level

---

### 4. **Frontend Best Practices**

#### Responsive Design
- **Mobile-First**: Tailwind CSS mobile-first approach
- **Breakpoints**: Responsive layouts for all screen sizes
- **Flexible Grids**: CSS Grid and Flexbox for layouts

#### Accessibility
- **Semantic HTML**: Proper use of HTML5 semantic elements
- **ARIA Labels**: Buttons and interactive elements have labels
- **Keyboard Navigation**: All functionality accessible via keyboard

#### Performance
- **CDN Resources**: Tailwind CSS and fonts loaded from CDN
- **Image Optimization**: Cloudinary handles image optimization
- **Minimal JavaScript**: Vanilla JavaScript for interactivity

---

### 5. **Error Handling**

#### User Feedback
- **Flash Messages**: User-friendly error and success messages
- **Categorized Messages**: Different styles for info, success, warning, error
- **Dismissible Alerts**: Users can dismiss flash messages

#### Graceful Degradation
- **404 Handling**: `get_or_404()` for missing resources
- **Access Denied**: 403 errors for unauthorized access
- **Form Validation**: Client and server-side validation

---

## Project Design Theme

### Color Palette

The application follows a clean, professional color scheme optimized for readability and user experience:

- **Primary Color**: `#1D4ED8` (Blue-700)
  - Used for: Buttons, links, primary actions, brand elements
  - Psychology: Trust, professionalism, reliability

- **Secondary Color**: `#4B5563` (Gray-600)
  - Used for: Secondary text, labels, borders
  - Psychology: Neutral, professional

- **Background**: `#F3F4F6` (Gray-100)
  - Used for: Page backgrounds, card backgrounds
  - Psychology: Clean, spacious feeling

- **Surface**: `#FFFFFF` (White)
  - Used for: Cards, containers, content areas
  - Psychology: Clean, modern

- **Footer**: `#111827` (Gray-900)
  - Used for: Footer background
  - Psychology: Professional, grounded

### Typography

- **Font Family**: Inter (Google Fonts)
  - **Weights**: 400 (Regular), 500 (Medium), 600 (Semibold), 700 (Bold)
  - **Characteristics**: Modern, readable, professional
  - **Usage**: All text throughout the application

### Design Principles

#### 1. **Minimalism**
   - Clean, uncluttered interfaces
   - Focus on essential information
   - White space for breathing room

#### 2. **Flat Design**
   - No heavy shadows or 3D effects
   - Subtle shadows for depth (`shadow-md`, `shadow-sm`)
   - Flat color blocks

#### 3. **Consistency**
   - Uniform spacing (Tailwind spacing scale)
   - Consistent button styles
   - Standardized card layouts

#### 4. **Rounded Corners**
   - Buttons: `rounded-lg` (0.5rem)
   - Cards: `rounded-xl` (0.75rem)
   - Inputs: `rounded` (0.25rem)

#### 5. **Shadow System**
   - Cards: `shadow-md` (medium shadow)
   - Buttons: `shadow-md` on hover
   - Headers: `shadow-sm` (subtle shadow)

### Component Styles

#### Buttons
```html
<!-- Primary Button -->
<button class="bg-primary text-white font-medium py-2 px-4 rounded-lg 
               shadow-md hover:bg-blue-700 transition duration-300">
```

#### Cards
```html
<!-- Standard Card -->
<div class="bg-white rounded-xl shadow-md border border-gray-200 p-6">
```

#### Input Fields
```html
<!-- Form Input -->
<input class="w-full mt-1 border rounded px-3 py-2 focus:outline-none 
              focus:ring-2 focus:ring-primary">
```

### Responsive Breakpoints

- **Mobile**: Default (< 640px)
- **Tablet**: `sm:` (≥ 640px)
- **Desktop**: `md:` (≥ 768px), `lg:` (≥ 1024px)

---

## Business Logic

### User Registration & Role Management

#### Registration Flow
1. User provides `username`, `password`, and selects `role` (customer or vendor)
2. System checks if username already exists
3. **If new user**:
   - Password is hashed with Bcrypt
   - User record created with `is_customer=True` or `is_vendor=True`
   - Flash message: "Account created successfully! Please login."
4. **If existing user**:
   - Password is verified
   - **If role not yet assigned**: New role is added (`is_customer` or `is_vendor` set to True)
   - **If role already exists**: Flash message: "You are already registered as a {role}"
   - **If password incorrect**: Flash message: "Incorrect password for this username."

#### Key Business Rule
- **One Account, Multiple Roles**: A single user account can have both customer and vendor roles
- **Role Addition**: Users can add additional roles to existing accounts
- **Password Verification**: When adding a role, the system verifies the password to prevent unauthorized role additions

---

### Authentication & Login

#### Login Flow
1. User provides `username`, `password`, and selects `role`
2. **Authentication Check**:
   - System queries `User` table by username
   - Password verified using `bcrypt.check_password_hash()`
   - If invalid: Flash "Invalid username or password" → Redirect to login
3. **Role Validation**:
   - **If role is 'vendor' and user is not registered as vendor**:
     - Flash: "You are not registered as a vendor."
     - Redirect to signup page
   - **If role is 'customer' and user is not registered as customer**:
     - Flash: "You are not registered as a customer."
     - Redirect to signup page
4. **Successful Login**:
   - `login_user(user)` creates session
   - `user.last_role = role` (tracks current session role)
   - Database commit
5. **Profile Check & Redirect**:
   - **For Customer**:
     - If `customer_profile` doesn't exist → Redirect to `customer.profile_setup`
     - If exists → Redirect to `customer.dashboard`
   - **For Vendor**:
     - If `vendor_profile` doesn't exist → Redirect to `vendor.profile_setup`
     - If exists → Redirect to `vendor.dashboard`

#### Critical Business Rule
- **Role-Based Login Restriction**: Users can ONLY login with a role they are registered for
- **Example**: If a user registered only as a customer, they CANNOT login as a vendor
- **Security**: This prevents unauthorized access to vendor features

---

### Profile Management

#### Customer Profile Setup
1. **Trigger**: After first customer login (if profile doesn't exist)
2. **Required Fields**:
   - `full_name`: Customer's full name
   - `address`: Customer address
   - `phone`: Contact phone number
3. **Validation**: 
   - Profile can only be created once per user
   - If profile exists, redirect to dashboard
4. **Database**: Creates `Customer` record linked to `User` via `user_id`

#### Vendor Profile Setup
1. **Trigger**: After first vendor login (if profile doesn't exist)
2. **Required Fields**:
   - `name`: Vendor's name
   - `email`: Business email
   - `business_name`: Name of the business
   - `business_address`: Physical address
   - `phone_number`: Primary contact (required)
   - `whatsapp_number`: WhatsApp contact (optional, important for India)
   - `open_duration`: Business hours (e.g., "9 AM - 6 PM")
   - `payment_type`: Accepted payment methods
   - `year_of_establishment`: Year business started
3. **Image Upload**:
   - Multiple business images can be uploaded
   - Images uploaded to Cloudinary in folder: `vendora/{user_id}/business_img/`
   - URLs stored in `User.business_imgs` JSON array
   - Images can be added incrementally (existing images preserved)
4. **Initialization**:
   - `rating` set to 0
   - `rater_no` set to 0 (for future rating system)
5. **Validation**: Profile can only be created once per user

#### Profile Updates
- **Vendor Profile Update**: All fields can be updated, additional images can be added
- **User Profile Update**: Name and profile image can be updated via `/auth/profile_update`

---

### Vendor Discovery

#### Vendor Listing (`/customer/homepage`)
1. **Search Functionality**:
   - Query parameter: `q` (search query)
   - Case-insensitive search on `Vendor.business_name`
   - Uses SQL `ILIKE` for pattern matching
2. **Display**:
   - All vendors listed if no search query
   - Filtered vendors if search query provided
   - Each vendor card shows:
     - Business image (first image from `business_imgs` array)
     - Business name
     - Business address
     - Link to vendor detail page

#### Vendor Detail Page (`/customer/vendor/<vendor_id>`)
1. **Data Retrieval**: Vendor fetched by ID using `get_or_404()`
2. **Displayed Information**:
   - Business images (from `business_imgs`)
   - Business name
   - Vendor name
   - Business address
   - Contact information (phone, WhatsApp, email)
   - Business hours (`open_duration`)
   - Payment methods
   - Year of establishment
   - Rating (if available)

---

### Image Management

#### Profile Image
- **Storage**: Cloudinary
- **Path**: `vendora/{user_id}/profile_img`
- **Overwrite**: `overwrite=True` (only one profile image per user)
- **Field**: `User.profile_img` (single URL string)

#### Business Images
- **Storage**: Cloudinary
- **Path**: `vendora/{user_id}/business_img/img_{index}`
- **Multiple Images**: Stored as JSON array in `User.business_imgs`
- **Incremental Addition**: New images appended to existing array
- **Indexing**: Images numbered sequentially (`img_1`, `img_2`, etc.)

---

### Access Control

#### Route Protection
- **`@login_required`**: Decorator ensures user is authenticated
- **Role Checks**: Routes check `current_user.is_vendor` or `current_user.is_customer`
- **Profile Checks**: Some routes require profile completion

#### Example: Vendor Dashboard
```python
@vendor.route('/dashboard')
@login_required
def dashboard():
    if current_user.is_vendor != True:
        return "Access denied", 403
    # ... rest of logic
```

#### Example: Customer Dashboard
```python
@customer.route('/dashboard')
@login_required
def dashboard():
    if current_user.is_customer != True:
        return "Access denied", 403
    # ... rest of logic
```

---

### Session Management

#### Last Role Tracking
- **Purpose**: Remember which role user logged in with
- **Storage**: `User.last_role` field in database
- **Usage**: 
  - Set during login: `user.last_role = role`
  - Used in templates to show appropriate header/sidebar
  - Allows users to switch between customer and vendor views (if they have both roles)

#### Logout
- **Action**: `logout_user()` clears Flask-Login session
- **Session Clear**: `session.clear()` removes all session data
- **Redirect**: Redirects to public vendor listing page

---

### Account Deletion

#### User Account Deletion
- **Route**: `/auth/delete_account` (POST)
- **Process**:
  1. User logged out first
  2. User record deleted from database
  3. **Cascade Delete**: Related `Customer` and `Vendor` profiles automatically deleted (due to `cascade="all, delete-orphan"`)
  4. Session cleared
  5. Redirect to public page

#### Vendor Profile Deletion
- **Route**: `/vendor/delete_profile`
- **Process**:
  1. User logged out
  2. Vendor profile deleted
  3. `is_vendor` set to False
  4. User account remains (can still be customer)

---

## Authentication Implementation

### Detailed Authentication Flow

#### 1. User Registration (`/auth/signup`)

**GET Request**: Displays signup form
**POST Request**: Processes registration

```python
# Pseudo-code flow
if user exists:
    if password matches:
        if role not assigned:
            assign role
            flash: "You are now also registered as a {role}!"
        else:
            flash: "You are already registered as a {role}"
    else:
        flash: "Incorrect password for this username"
else:
    create new user
    hash password with bcrypt
    assign role (is_customer or is_vendor)
    flash: "Account created successfully! Please login."
    
redirect to login
```

**Key Points**:
- Password is hashed: `bcrypt.generate_password_hash(password)`
- Role flags set: `is_customer=True` or `is_vendor=True`
- Username must be unique (enforced at database level)

---

#### 2. User Login (`/auth/login`)

**GET Request**: Displays login form
**POST Request**: Authenticates user

```python
# Step 1: Find user by username
user = User.query.filter(User.username == username).first()

# Step 2: Verify password
if not user or not bcrypt.check_password_hash(user.password, password):
    flash('Invalid username or password', 'danger')
    return redirect to login

# Step 3: Verify role registration
if role == 'vendor' and not user.is_vendor:
    flash('You are not registered as a vendor.', 'danger')
    return redirect to signup
elif role == 'customer' and not user.is_customer:
    flash('You are not registered as a customer.', 'danger')
    return redirect to signup

# Step 4: Login successful
login_user(user)  # Creates Flask-Login session
user.last_role = role  # Track current session role
db.session.commit()

# Step 5: Redirect based on profile status
if role == 'customer':
    if not user.customer_profile:
        redirect to customer.profile_setup
    else:
        redirect to customer.dashboard
elif role == 'vendor':
    if not user.vendor_profile:
        redirect to vendor.profile_setup
    else:
        redirect to vendor.dashboard
```

**Critical Security Feature**:
- **Role Verification**: Users CANNOT login with a role they haven't registered for
- **Example Scenario**:
  - User "john" registers only as customer (`is_customer=True`, `is_vendor=False`)
  - User tries to login as vendor
  - System checks: `role == 'vendor' and not user.is_vendor` → True
  - Result: Flash message + redirect to signup (to register as vendor)

---

#### 3. Session Management

**Flask-Login Integration**:
```python
# In app.py
@login_manager.user_loader
def load_user(uid):
    return User.query.get(uid)
```

**How It Works**:
1. When user logs in, Flask-Login stores user ID in session
2. On each request, `user_loader` function retrieves user from database
3. `current_user` object available in all routes and templates
4. `@login_required` decorator checks if `current_user.is_authenticated`

**Session Data**:
- User ID stored in encrypted session cookie
- `current_user` object loaded on each request
- `last_role` stored in database (persists across sessions)

---

#### 4. Role-Based Access Control

**Template-Level**:
```jinja2
{% if current_user.is_authenticated %}
    {% if current_user.last_role == 'customer' %}
        {% include 'headers/user_header.html' %}
        {% include 'sidebar/sidebar_customer.html' %}
    {% elif current_user.last_role == 'vendor' %}
        {% include 'headers/user_header.html' %}
        {% include 'sidebar/sidebar_vendor.html' %}
    {% endif %}
{% else %}
    {% include 'headers/guest_header.html' %}
{% endif %}
```

**Route-Level**:
```python
@vendor.route('/dashboard')
@login_required
def dashboard():
    if current_user.is_vendor != True:
        return "Access denied", 403
    # ... vendor dashboard logic
```

---

#### 5. Password Security

**Hashing Algorithm**: Bcrypt
- **Salt**: Automatically generated (unique per password)
- **Rounds**: Default 12 rounds (configurable)
- **Storage**: Hashed password stored as string in database

**Verification**:
```python
# During login
bcrypt.check_password_hash(stored_hash, provided_password)
# Returns True if password matches, False otherwise
```

**Security Benefits**:
- Even if database is compromised, passwords cannot be reversed
- Each password has unique salt (prevents rainbow table attacks)
- Computationally expensive (slows brute force attacks)

---

#### 6. Logout Process

```python
@auth.route('/logout')
def logout():
    logout_user()  # Clears Flask-Login session
    session.clear()  # Clears all session data
    return redirect(url_for('customer.vendors'))
```

**What Happens**:
1. Flask-Login session cleared
2. All session variables removed
3. User redirected to public page
4. `current_user` becomes anonymous

---

## Features

### Implemented Features

#### 1. **User Authentication**
   - ✅ User registration (customer/vendor)
   - ✅ User login with role selection
   - ✅ Password hashing (Bcrypt)
   - ✅ Session management (Flask-Login)
   - ✅ Logout functionality
   - ✅ Account deletion
   - ✅ Profile update (name, profile image)

#### 2. **Customer Features**
   - ✅ Customer registration
   - ✅ Customer profile setup
   - ✅ Customer dashboard
   - ✅ Browse all vendors
   - ✅ Search vendors (by business name)
   - ✅ View vendor details
   - ✅ Responsive customer interface

#### 3. **Vendor Features**
   - ✅ Vendor registration
   - ✅ Vendor profile setup
   - ✅ Vendor dashboard
   - ✅ Vendor profile update
   - ✅ Business image upload (multiple images)
   - ✅ Business information management
   - ✅ Vendor profile deletion

#### 4. **Image Management**
   - ✅ Profile image upload (Cloudinary)
   - ✅ Business image gallery (multiple images)
   - ✅ Cloud-based storage
   - ✅ Image URL management

#### 5. **UI/UX Features**
   - ✅ Responsive design (mobile-first)
   - ✅ Tailwind CSS styling
   - ✅ Flash messages (success, error, info, warning)
   - ✅ Navigation headers (guest, customer, vendor)
   - ✅ Sidebar navigation (customer, vendor)
   - ✅ Footer with links
   - ✅ Developer page (route listing)

#### 6. **Database Features**
   - ✅ MySQL database (Aiven Cloud)
   - ✅ SQLAlchemy ORM
   - ✅ Database migrations (Flask-Migrate)
   - ✅ Relationship management (cascade deletes)
   - ✅ JSON field support (business images)

---

### Planned/Future Features

#### 1. **Appointment System**
   - ⏳ Appointment booking (customer → vendor)
   - ⏳ Appointment management (vendor dashboard)
   - ⏳ Appointment calendar view
   - ⏳ Appointment status (pending, confirmed, completed, cancelled)
   - ⏳ Appointment reminders

#### 2. **Review & Rating System**
   - ⏳ Customer reviews for vendors
   - ⏳ Star rating system
   - ⏳ Rating calculation (average rating)
   - ⏳ Review display on vendor pages

#### 3. **Analytics Dashboard**
   - ⏳ Vendor analytics (views, bookings, revenue)
   - ⏳ Customer analytics (bookings history)
   - ⏳ Charts and graphs

#### 4. **Communication Features**
   - ⏳ In-app messaging
   - ⏳ WhatsApp integration (for Indian market)
   - ⏳ Email notifications

#### 5. **Advanced Search & Filters**
   - ⏳ Filter by location
   - ⏳ Filter by service type
   - ⏳ Filter by rating
   - ⏳ Sort by popularity, rating, distance

#### 6. **Payment Integration**
   - ⏳ Payment gateway integration
   - ⏳ Online payment support
   - ⏳ Payment history

---

## Deployment & Cloud Infrastructure

### Current Deployment

#### Platform: Vercel
- **Configuration File**: `vercel.json`
- **Build Command**: Automatic (detects Python/Flask)
- **Runtime**: Python 3.x (serverless functions)

#### Vercel Configuration
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

**How It Works**:
- Vercel detects `run.py` as entry point
- Uses `@vercel/python` builder for Flask apps
- All routes (`/(.*)`) are handled by Flask application
- Serverless functions created automatically

---

### Cloud Services

#### 1. **Database: Aiven Cloud (MySQL)**
- **Provider**: Aiven
- **Type**: Managed MySQL database
- **Connection**: 
  - Host: `mysql-3b0838a6-priyamjainsocial-b642.i.aivencloud.com`
  - Port: `27509`
  - User: `avnadmin`
  - Password: Stored in environment variable `DB_PASSWORD`
  - Database: `defaultdb`

**Connection String**:
```python
f'mysql+pymysql://avnadmin:{os.getenv("DB_PASSWORD")}@mysql-3b0838a6-priyamjainsocial-b642.i.aivencloud.com:27509/defaultdb'
```

**Benefits**:
- Managed service (backups, updates handled)
- High availability
- Scalable
- Secure (SSL connections)

---

#### 2. **File Storage: Cloudinary**
- **Service**: Cloudinary (cloud-based image management)
- **Configuration**: Via environment variables
  - `c_cloud_name`: Cloudinary cloud name
  - `c_api_key`: API key
  - `c_api_secret`: API secret

**Usage**:
- Profile images: `vendora/{user_id}/profile_img`
- Business images: `vendora/{user_id}/business_img/img_{index}`

**Features Used**:
- Image upload
- Automatic optimization
- CDN delivery
- Secure URLs

---

### Environment Variables

Required environment variables (stored in `.env` file):

```env
# Database
DB_PASSWORD=your_aiven_db_password

# Cloudinary
c_cloud_name=your_cloudinary_cloud_name
c_api_key=your_cloudinary_api_key
c_api_secret=your_cloudinary_api_secret

# Flask
SECRET_KEY=your_secret_key_for_sessions
```

**Security Note**: `.env` file should NEVER be committed to version control.

---

### Deployment Process

#### Local Development
1. Create virtual environment: `python -m venv vendenv`
2. Activate virtual environment: `vendenv\Scripts\activate` (Windows)
3. Install dependencies: `pip install -r requirements.txt`
4. Create `.env` file with required variables
5. Run migrations: `flask db upgrade`
6. Run application: `python run.py`

#### Production Deployment (Vercel)
1. Push code to Git repository
2. Connect repository to Vercel
3. Configure environment variables in Vercel dashboard
4. Deploy (automatic on push or manual)

---

### Scalability Considerations

#### Current Architecture
- **Serverless**: Vercel uses serverless functions (auto-scaling)
- **Database**: Aiven MySQL can scale vertically and horizontally
- **CDN**: Cloudinary provides CDN for images

#### Future Scalability Options
- **Load Balancing**: Vercel handles this automatically
- **Database Replication**: Aiven supports read replicas
- **Caching**: Can add Redis for session caching
- **API Rate Limiting**: Can be added for API protection

---

## Getting Started

### Prerequisites
- Python 3.8 or higher
- MySQL database (or use Aiven Cloud)
- Cloudinary account (for image storage)
- Git (for version control)

### Installation Steps

#### 1. Clone Repository
```bash
git clone <repository-url>
cd vendora2
```

#### 2. Create Virtual Environment
```bash
# Windows
python -m venv vendenv
vendenv\Scripts\activate

# Linux/Mac
python3 -m venv vendenv
source vendenv/bin/activate
```

#### 3. Install Dependencies
