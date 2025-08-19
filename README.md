# X Restaurant backend API documentation demo version-1

## Table of Contents

1. [Overview](#overview)
2. [Authentication](#authentication)
3. [API Endpoints](#api-endpoints)
4. [Error Handling](#error-handling)
5. [Data Models](#data-models)

## Overview

### Base URL

```
https://api.restaurant.com/v1
```

### Technology Stack

- **Framework**: FastAPI
- **Authentication**: JWT (JSON Web Tokens)
- **Payment**: Stripe Integration
- **AI**: OpenAI/Custom Chatbot Integration

### User Roles

- **Customer**: Browse menu, place orders, leave reviews
- **Staff**: Manage orders, respond to reviews, manage FAQ
- **Super Admin**: Full system access, analytics, user management

---

## Authentication

### JWT Token Authentication

All protected endpoints require a Bearer token in the Authorization header:

```
Authorization: Bearer <jwt_token>
```

### Auth Endpoints

#### POST /auth/register

Register a new user account.

**Request Body:**

```json
{
  "email": "user@example.com",
  "password": "securepassword",
  "full_name": "John Doe",
  "phone": "+1234567890",
  "role": "customer",
  "address": {
    "street": "123 Main St",
    "city": "New York",
    "postcode": "10001",
    "country": "USA"
  }
}
```

**Response:**

```json
{
  "success": true,
  "message": "User registered successfully",
  "user_id": "uuid-string",
  "access_token": "jwt-token",
  "token_type": "bearer"
}
```

#### POST /auth/login

Authenticate user and get access token.

**Request Body:**

```json
{
  "email": "user@example.com",
  "password": "securepassword"
}
```

**Response:**

```json
{
  "success": true,
  "access_token": "jwt-token",
  "token_type": "bearer",
  "expires_in": 3600,
  "user": {
    "id": "uuid",
    "email": "user@example.com",
    "full_name": "John Doe",
    "role": "customer"
  }
}
```

#### POST /auth/refresh

Refresh access token.

**Headers:**

```
Authorization: Bearer <refresh_token>
```

**Response:**

```json
{
  "success": true,
  "access_token": "new-jwt-token",
  "token_type": "bearer",
  "expires_in": 3600
}
```

---

## API Endpoints

### Customer Endpoints

#### GET /menu/categories

Get all menu categories.

**Response:**

```json
{
  "success": true,
  "data": [
    {
      "id": "uuid",
      "name": "Appetizers",
      "description": "Start your meal right",
      "image_url": "https://example.com/image.jpg",
      "is_active": true,
      "sort_order": 1
    }
  ]
}
```

#### GET /menu/categories/{category_id}/items

Get menu items by category.

**Query Parameters:**

- `page`: Page number (default: 1)
- `limit`: Items per page (default: 20)
- `sort_by`: Sort field (price, name, popularity)
- `sort_order`: asc/desc

**Response:**

```json
{
  "success": true,
  "data": {
    "items": [
      {
        "id": "uuid",
        "name": "Caesar Salad",
        "description": "Fresh romaine lettuce with caesar dressing",
        "price": 12.99,
        "category_id": "uuid",
        "images": ["url1", "url2"],
        "ingredients": ["lettuce", "parmesan", "croutons"],
        "allergens": ["gluten", "dairy"],
        "is_available": true,
        "is_vegetarian": true,
        "is_vegan": false,
        "preparation_time": 10,
        "popularity_score": 4.5,
        "created_at": "2024-01-01T00:00:00Z",
        "updated_at": "2024-01-01T00:00:00Z"
      }
    ],
    "pagination": {
      "page": 1,
      "limit": 20,
      "total": 50,
      "total_pages": 3
    }
  }
}
```

#### GET /menu/items/{item_id}

Get detailed information about a specific menu item.

**Response:**

```json
{
  "success": true,
  "data": {
    "id": "uuid",
    "name": "Caesar Salad",
    "description": "Fresh romaine lettuce with caesar dressing",
    "price": 12.99,
    "category": {
      "id": "uuid",
      "name": "Salads"
    },
    "images": ["url1", "url2"],
    "ingredients": ["lettuce", "parmesan", "croutons"],
    "allergens": ["gluten", "dairy"],
    "reviews_summary": {
      "average_rating": 4.5,
      "total_reviews": 125
    },
    "is_available": true,
    "preparation_time": 10
  }
}
```

#### GET /gallery

Get restaurant gallery images.

**Query Parameters:**

- `category`: Image category (food, interior, exterior, events)
- `limit`: Number of images (default: 20)

**Response:**

```json
{
  "success": true,
  "data": [
    {
      "id": "uuid",
      "url": "https://example.com/image.jpg",
      "title": "Restaurant Interior",
      "description": "Our cozy dining area",
      "category": "interior",
      "alt_text": "Restaurant dining room with warm lighting",
      "created_at": "2024-01-01T00:00:00Z"
    }
  ]
}
```

#### POST /reviews

Submit a review for the restaurant.

**Request Body:**

```json
{
  "rating": 5,
  "title": "Amazing food!",
  "comment": "The food was absolutely delicious...",
  "order_id": "uuid-optional"
}
```

**Response:**

```json
{
  "success": true,
  "message": "Review submitted successfully",
  "review_id": "uuid"
}
```

#### GET /reviews

Get restaurant reviews.

**Query Parameters:**

- `page`: Page number
- `limit`: Reviews per page
- `rating_filter`: Filter by rating (1-5)
- `sort_by`: newest, oldest, rating_high, rating_low

**Response:**

```json
{
  "success": true,
  "data": {
    "reviews": [
      {
        "id": "uuid",
        "user_name": "John Doe",
        "rating": 5,
        "title": "Amazing food!",
        "comment": "The food was absolutely delicious...",
        "created_at": "2024-01-01T00:00:00Z",
        "staff_reply": {
          "message": "Thank you for your feedback!",
          "replied_by": "Manager",
          "replied_at": "2024-01-02T00:00:00Z"
        }
      }
    ],
    "pagination": {
      "page": 1,
      "limit": 10,
      "total": 250,
      "total_pages": 25
    },
    "summary": {
      "average_rating": 4.3,
      "total_reviews": 250,
      "rating_distribution": {
        "5": 120,
        "4": 80,
        "3": 30,
        "2": 15,
        "1": 5
      }
    }
  }
}
```

#### GET /faq

Get frequently asked questions.

**Query Parameters:**

- `category`: FAQ category
- `search`: Search term

**Response:**

```json
{
  "success": true,
  "data": [
    {
      "id": "uuid",
      "question": "What are your opening hours?",
      "answer": "We are open from 11 AM to 10 PM daily.",
      "category": "general",
      "is_featured": true,
      "created_at": "2024-01-01T00:00:00Z",
      "updated_at": "2024-01-01T00:00:00Z"
    }
  ]
}
```

#### POST /contact

Submit a contact form.

**Request Body:**

```json
{
  "name": "John Doe",
  "email": "john@example.com",
  "phone": "+1234567890",
  "subject": "Booking Inquiry",
  "message": "I would like to make a reservation...",
  "inquiry_type": "reservation"
}
```

**Response:**

```json
{
  "success": true,
  "message": "Message sent successfully",
  "ticket_id": "uuid"
}
```

#### POST /delivery/check-postcode

Check delivery availability and pricing for postcode.

**Request Body:**

```json
{
  "postcode": "10001"
}
```

**Response:**

```json
{
  "success": true,
  "data": {
    "is_available": true,
    "delivery_fee": 3.99,
    "minimum_order": 25.0,
    "estimated_delivery_time": "30-45 minutes",
    "delivery_zones": [
      {
        "zone_name": "Downtown",
        "fee": 3.99,
        "min_order": 25.0
      }
    ]
  }
}
```

#### POST /orders

Create a new order.

**Request Body:**

```json
{
  "items": [
    {
      "menu_item_id": "uuid",
      "quantity": 2,
      "special_instructions": "No onions",
      "modifications": [
        {
          "type": "add",
          "item": "extra cheese",
          "price": 2.0
        }
      ]
    }
  ],
  "delivery_address": {
    "street": "123 Main St",
    "city": "New York",
    "postcode": "10001",
    "notes": "Ring doorbell twice"
  },
  "order_type": "delivery",
  "payment_method": "stripe",
  "special_requests": "Please call when arrived"
}
```

**Response:**

```json
{
  "success": true,
  "data": {
    "order_id": "uuid",
    "order_number": "ORD-001234",
    "total_amount": 45.97,
    "estimated_delivery_time": "45 minutes",
    "payment_intent_id": "pi_stripe_payment_intent",
    "status": "pending_payment"
  }
}
```

#### POST /payments/process

Process payment with Stripe.

**Request Body:**

```json
{
  "order_id": "uuid",
  "payment_method_id": "pm_stripe_payment_method",
  "return_url": "https://restaurant.com/payment/return"
}
```

**Response:**

```json
{
  "success": true,
  "data": {
    "payment_intent": {
      "id": "pi_stripe_payment_intent",
      "status": "succeeded",
      "amount": 45,
      "currency": "usd"
    },
    "order_status": "confirmed"
  }
}
```

### Staff Endpoints

#### POST /staff/menu-items

Add a new menu item (Staff only).

**Request Body:**

```json
{
  "name": "New Dish",
  "description": "Delicious new creation",
  "price": 15.99,
  "category_id": "uuid",
  "ingredients": ["ingredient1", "ingredient2"],
  "allergens": ["nuts"],
  "is_vegetarian": false,
  "is_vegan": false,
  "preparation_time": 20,
  "images": ["image1", "Image2", "...."]
}
```

**Response:**

```json
{
  "success": true,
  "message": "Menu item created successfully",
  "item_id": "uuid"
}
```

#### GET /staff/orders

Get orders for staff management.

**Query Parameters:**

- `status`: Order status filter
- `date_from`: Start date
- `date_to`: End date
- `order_type`: delivery/pickup/dine_in

**Response:**

```json
{
  "success": true,
  "data": [
    {
      "id": "uuid",
      "order_number": "ORD-001234",
      "customer": {
        "name": "John Doe",
        "phone": "+1234567890"
      },
      "items": [
        {
          "name": "Caesar Salad",
          "quantity": 2,
          "special_instructions": "No croutons"
        }
      ],
      "total_amount": 25.98,
      "status": "pending",
      "order_type": "delivery",
      "created_at": "2024-01-01T12:00:00Z",
      "estimated_ready_time": "2024-01-01T12:30:00Z"
    }
  ]
}
```

#### PUT /staff/orders/{order_id}/status

Update order status.

**Request Body:**

```json
{
  "status": "preparing",
  "estimated_ready_time": "2024-01-01T12:45:00Z",
  "notes": "Order is being prepared"
}
```

**Response:**

```json
{
  "success": true,
  "message": "Order status updated successfully"
}
```

#### POST /staff/reviews/{review_id}/reply

Reply to a customer review.

**Request Body:**

```json
{
  "message": "Thank you for your feedback! We're glad you enjoyed your meal."
}
```

**Response:**

```json
{
  "success": true,
  "message": "Reply posted successfully"
}
```

#### POST /staff/faq/{faq_id}/answer

Update FAQ answer.

**Request Body:**

```json
{
  "answer": "Updated answer to the frequently asked question."
}
```

**Response:**

```json
{
  "success": true,
  "message": "FAQ answer updated successfully"
}
```

### Super Admin Endpoints

#### PUT /admin/menu-items/{item_id}

Edit existing menu item.

**Request Body:**

```json
{
  "name": "Updated Dish Name",
  "description": "Updated description",
  "price": 18.99,
  "is_available": true,
  "category_id": "uuid"
}
```

**Response:**

```json
{
  "success": true,
  "message": "Menu item updated successfully"
}
```

#### POST /admin/staff

Add new staff member.

**Request Body:**

```json
{
  "email": "staff@restaurant.com",
  "full_name": "Jane Smith",
  "role": "staff",
  "phone": "+1234567890",
  "position": "Manager",
  "hire_date": "2024-01-01",
  "permissions": ["manage_orders", "reply_reviews"]
}
```

**Response:**

```json
{
  "success": true,
  "message": "Staff member added successfully",
  "staff_id": "uuid",
  "temporary_password": "temp_password_123"
}
```

#### GET /admin/staff

Get all staff members.

**Response:**

```json
{
  "success": true,
  "data": [
    {
      "id": "uuid",
      "email": "staff@restaurant.com",
      "full_name": "Jane Smith",
      "role": "staff",
      "position": "Server",
      "hire_date": "2024-01-01",
      "is_active": true,
      "last_login": "2024-01-15T10:30:00Z",
      "permissions": ["manage_orders", "reply_reviews"]
    }
  ]
}
```

#### PUT /admin/reviews/{review_id}

Edit customer review (admin only).

**Request Body:**

```json
{
  "comment": "Moderated review content",
  "is_hidden": false,
  "moderation_reason": "Inappropriate language removed"
}
```

**Response:**

```json
{
  "success": true,
  "message": "Review updated successfully"
}
```

#### GET /admin/dashboard/analytics

Get restaurant analytics and metrics.

**Query Parameters:**

- `period`: daily, weekly, monthly, yearly
- `date_from`: Start date
- `date_to`: End date

**Response:**

```json
{
  "success": true,
  "data": {
    "revenue": {
      "total": 125000.5,
      "period_change": "+12.5%"
    },
    "orders": {
      "total": 2450,
      "period_change": "+8.2%",
      "by_status": {
        "completed": 2380,
        "cancelled": 45,
        "pending": 25
      }
    },
    "customers": {
      "total": 1850,
      "new_customers": 125,
      "returning_customers": 1725
    },
    "popular_items": [
      {
        "name": "Caesar Salad",
        "orders": 245,
        "revenue": 3062.55
      }
    ],
    "reviews": {
      "average_rating": 4.3,
      "total_reviews": 892,
      "period_change": "+15.8%"
    },
    "delivery_stats": {
      "average_delivery_time": 35,
      "on_time_percentage": 92.5
    }
  }
}
```

#### GET /admin/reports/sales

Generate sales reports.

**Query Parameters:**

- `report_type`: daily, weekly, monthly
- `date_from`: Start date
- `date_to`: End date
- `format`: json, csv, pdf

**Response:**

```json
{
  "success": true,
  "data": {
    "report_id": "uuid",
    "download_url": "https://api.restaurant.com/reports/download/uuid",
    "generated_at": "2024-01-15T14:30:00Z",
    "summary": {
      "total_revenue": 45000.75,
      "total_orders": 1250,
      "average_order_value": 36.0
    }
  }
}
```

### AI Chatbot Endpoints

#### POST /chatbot/conversation

Start or continue a conversation with AI chatbot.

**Request Body:**

```json
{
  "message": "What are your vegetarian options?",
  "conversation_id": "uuid-optional",
  "context": {
    "user_id": "uuid-optional",
    "current_page": "menu"
  }
}
```

**Response:**

```json
{
  "success": true,
  "data": {
    "conversation_id": "uuid",
    "response": "We have several delicious vegetarian options including our popular Caesar Salad, Margherita Pizza, and Vegetable Pasta. Would you like me to show you the full vegetarian menu?",
    "suggested_actions": [
      {
        "type": "show_menu",
        "label": "View Vegetarian Menu",
        "filter": "vegetarian"
      }
    ],
    "confidence": 0.95,
    "response_time": 1.2
  }
}
```

#### GET /chatbot/conversations/{conversation_id}/history

Get chat history for a conversation.

**Response:**

```json
{
  "success": true,
  "data": {
    "conversation_id": "uuid",
    "messages": [
      {
        "id": "uuid",
        "type": "user",
        "message": "What are your vegetarian options?",
        "timestamp": "2024-01-15T14:30:00Z"
      },
      {
        "id": "uuid",
        "type": "bot",
        "message": "We have several delicious vegetarian options...",
        "timestamp": "2024-01-15T14:30:02Z"
      }
    ]
  }
}
```

---

## Error Handling

### Standard Error Response Format

```json
{
  "success": false,
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Validation failed for the request",
    "details": [
      {
        "field": "email",
        "message": "Invalid email format"
      }
    ]
  },
  "timestamp": "2024-01-15T14:30:00Z",
  "request_id": "uuid"
}
```

### HTTP Status Codes

- `200`: Success
- `201`: Created
- `400`: Bad Request
- `401`: Unauthorized
- `403`: Forbidden
- `404`: Not Found
- `422`: Validation Error
- `429`: Rate Limited
- `500`: Internal Server Error

### Common Error Codes

- `INVALID_CREDENTIALS`: Login credentials are incorrect
- `TOKEN_EXPIRED`: JWT token has expired
- `INSUFFICIENT_PERMISSIONS`: User lacks required permissions
- `VALIDATION_ERROR`: Request validation failed
- `RESOURCE_NOT_FOUND`: Requested resource doesn't exist
- `PAYMENT_FAILED`: Payment processing failed
- `ORDER_NOT_MODIFIABLE`: Order cannot be modified in current state

---

## Data Models

### User Model

```json
{
  "id": "uuid",
  "email": "user@example.com",
  "full_name": "John Doe",
  "phone": "+1234567890",
  "role": "customer|staff|super_admin",
  "is_active": true,
  "email_verified": true,
  "created_at": "2024-01-01T00:00:00Z",
  "updated_at": "2024-01-01T00:00:00Z",
  "last_login": "2024-01-15T10:30:00Z",
  "address": {
    "street": "123 Main St",
    "city": "New York",
    "postcode": "10001",
    "country": "USA"
  }
}
```

### Menu Item Model

```json
{
  "id": "uuid",
  "name": "Caesar Salad",
  "description": "Fresh romaine lettuce with caesar dressing",
  "price": 12.99,
  "category_id": "uuid",
  "images": ["url1", "url2"],
  "ingredients": ["lettuce", "parmesan", "croutons"],
  "allergens": ["gluten", "dairy"],
  "nutritional_info": {
    "calories": 250,
    "protein": 8,
    "carbs": 15,
    "fat": 18
  },
  "is_available": true,
  "is_vegetarian": true,
  "is_vegan": false,
  "preparation_time": 10,
  "popularity_score": 4.5,
  "created_at": "2024-01-01T00:00:00Z",
  "updated_at": "2024-01-01T00:00:00Z"
}
```

### Order Model

```json
{
  "id": "uuid",
  "order_number": "ORD-001234",
  "customer_id": "uuid",
  "items": [
    {
      "menu_item_id": "uuid",
      "quantity": 2,
      "unit_price": 12.99,
      "total_price": 25.98,
      "special_instructions": "No onions",
      "modifications": [
        {
          "type": "add",
          "item": "extra cheese",
          "price": 2.0
        }
      ]
    }
  ],
  "subtotal": 25.98,
  "delivery_fee": 3.99,
  "tax": 2.4,
  "total_amount": 32.37,
  "status": "pending|confirmed|preparing|ready|delivered|cancelled",
  "order_type": "delivery|pickup|dine_in",
  "delivery_address": {
    "street": "123 Main St",
    "city": "New York",
    "postcode": "10001",
    "notes": "Ring doorbell twice"
  },
  "payment_status": "pending|completed|failed|refunded",
  "payment_method": "stripe|cash|card",
  "special_requests": "Please call when arrived",
  "estimated_ready_time": "2024-01-01T12:30:00Z",
  "created_at": "2024-01-01T12:00:00Z",
  "updated_at": "2024-01-01T12:15:00Z"
}
```

---

## Rate Limiting

### Rate Limits by Role

- **Customer**: 100 requests per minute
- **Staff**: 200 requests per minute
- **Super Admin**: 500 requests per minute
- **Chatbot**: 50 requests per minute per conversation

### Rate Limit Headers

```
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 95
X-RateLimit-Reset: 1640995200
```

---

## Webhooks

### Stripe Payment Webhooks

**Endpoint**: `POST /webhooks/stripe`

**Events**:

- `payment_intent.succeeded`
- `payment_intent.payment_failed`
- `payment_method.attached`

### Order Status Webhooks (Optional)

**Endpoint**: `POST /webhooks/order-status`

**Payload**:

```json
{
  "event": "order.status_changed",
  "order_id": "uuid",
  "old_status": "preparing",
  "new_status": "ready",
  "timestamp": "2024-01-15T14:30:00Z"
}
```

---

## Environment Configuration

### Required Environment Variables

```bash
# Database
DATABASE_URL=postgresql://user:password@localhost/restaurant_db

# JWT
JWT_SECRET_KEY=your-secret-key
JWT_ALGORITHM=HS256
JWT_ACCESS_TOKEN_EXPIRE_MINUTES=60

# Stripe
STRIPE_PUBLIC_KEY=pk_test_...
STRIPE_SECRET_KEY=sk_test_...
STRIPE_WEBHOOK_SECRET=whsec_...

# AI Chatbot
OPENAI_API_KEY=sk-...
CHATBOT_MODEL=gpt-3.5-turbo

# File Upload
MAX_FILE_SIZE=10MB
ALLOWED_IMAGE_TYPES=jpg,jpeg,png,webp

# Email
SMTP_HOST=smtp.gmail.com
SMTP_PORT=587
SMTP_USER=notifications@restaurant.com
SMTP_PASSWORD=app-password

# Redis (for caching and rate limiting)
REDIS_URL=redis://localhost:6379/0
```

## Author Portfolio

[Farzine's Portfolio](https://farzine.vercel.app/).
[Siam's resume](https://drive.google.com/file/d/1oO8ZIcqzfowWKeSxvELNL9X2pdOkMTGz/view).
