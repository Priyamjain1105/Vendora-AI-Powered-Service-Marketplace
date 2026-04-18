# 🌐 Vendora — AI-Powered Service Marketplace

> **Designing intelligent systems, not just building features.**

Vendora is a full-stack service marketplace that connects customers with vendors while leveraging **Machine Learning for personalization** and **Deep Learning for sentiment understanding**.

This project is built to demonstrate **real-world system design thinking** — combining:

* Scalable backend architecture
* Relational database modeling (MySQL)
* Microservice-based ML integration
* Clean role-based workflows

---

## 🚀 Why Vendora?

Most marketplace apps stop at CRUD.

**Vendora goes further:**

* Understands *user intent* through reviews (Sentiment Analysis)
* Learns *user behavior* to recommend vendors (Recommendation System)
* Maintains *data consistency* across complex relationships
* Handles *real-world workflows* like booking conflicts and lifecycle transitions

This project reflects the ability to **connect backend systems with intelligent services**, not just build isolated features.

---

## 🧠 Key Highlights

* 🏗️ Designed a **relational database system** with multiple interdependent entities
* 🔄 Built a **complete booking lifecycle engine** with conflict resolution
* ⚡ Integrated **FastAPI-based ML microservices** into a Flask app
* 🤖 Used **DistilBERT (PyTorch)** for real-time sentiment classification
* 🎯 Implemented **personalized recommendation system pipeline**
* 🔐 Applied **authentication, role management, and secure password handling**
* ☁️ Integrated **Cloudinary for scalable media handling**

---

## 🧩 System Architecture

```
Frontend (Jinja Templates)
        ↓
Flask Backend (Core System)
        ↓
MySQL Database (Relational Data)
        ↓
-------------------------------
| External ML Microservices   |
|-----------------------------|
| FastAPI - Sentiment Model   |
| Recommender System API      |
-------------------------------
```

---

## 🏗️ Core Features

### 👤 Customer

* Discover and explore vendors
* Book services with **real-time conflict handling**
* Manage appointments (update/cancel)
* Submit ratings and reviews
* Get **AI-powered recommendations**

---

### 🧑‍💼 Vendor

* Manage business profile and services
* Track appointments lifecycle:
  `pending → confirmed → completed / cancelled`
* View analytics (revenue, demand, customers)
* Analyze sentiment breakdown of reviews

---

### 🔐 Authentication

* Role-based access (`customer`, `vendor`, or both)
* Secure password hashing
* Profile and account management

---

## 🤖 Machine Learning Integration

### Sentiment Analysis

* FastAPI + Hugging Face + PyTorch (DistilBERT)
* Converts review text into:

  * positive / neutral / negative

**Flow:**

```
Review → API → Sentiment → Database
```

---

### Recommendation System

* API-based integration

**Flow:**

```
Customer ID → Recommendation API → Vendor IDs → DB → UI
```

---

## 🗄️ Database Design

### Core Entities:

* User
* Customer
* Vendor
* Service
* Appointment

### Relationships:

* User → Customer / Vendor (1:1)
* Vendor → Services (1:N)
* Appointment → links Customer, Vendor, Service

Ensures:

* Data integrity
* Real-world modeling
* Complex querying capability

---

## 📁 Project Structure

```
vendora2/
│
├── vendora_app/
│   ├── blueprints/
│   ├── extensions.py
│   ├── app.py
│   └── migrations/
│
├── Sentiment Analysis/
├── Recommendation System/
│
├── run.py
└── requirements.txt
```

---

## ⚙️ Tech Stack

### Backend

* Flask
* Flask-SQLAlchemy
* Flask-Migrate
* Flask-Login
* Flask-Bcrypt

### Database

* MySQL

### ML / AI

* FastAPI
* PyTorch
* Hugging Face Transformers

### Other

* Cloudinary
* Jinja2

---

## 🔄 User Flow

```
Signup → Role Selection
     ↓
Profile Setup
     ↓
Customer books service
     ↓
Vendor completes service
     ↓
Customer leaves review
     ↓
Sentiment Analysis
     ↓
Recommendations generated
```

---

## 🛠️ Setup

### Install

```
git clone <your-repo-link>
cd vendora2

python -m venv .venv
.venv\Scripts\activate

pip install -r requirements.txt
```

---

### Run App

```
python run.py
```

---

### Run Sentiment Service

```
cd "Sentiment Analysis"
pip install -r requirements.txt
uvicorn app.main:app --reload --port 7860
```

---

## 🔐 Production Notes

* Use environment variables for secrets
* Add CSRF protection
* Implement API retry mechanisms
* Add rate limiting
* Consider Docker for deployment

---

## 📈 Future Improvements

* Automated testing
* Caching ML responses
* Improved recommendation models
* Async processing
* Logging and monitoring

---

## 🧑‍💻 What This Project Demonstrates

* End-to-end system design
* Complex relational database modeling
* ML/DL integration in real applications
* Scalable backend architecture
* Practical problem-solving mindset

---

## 📌 Final Note

Vendora is an attempt to bridge:

> **Software Engineering + Machine Learning + Real-world usability**
