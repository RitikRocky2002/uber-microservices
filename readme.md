
# Uber Microservices API Documentation

This document provides detailed information on the APIs exposed by the Uber Microservices project. The project is built using a microservices architecture where separate services handle different business domains: **User Service**, **Captain Service**, **Ride Service**, and an **API Gateway**. Each service follows REST principles and communicates using JSON. 

> **Note:** The API Gateway listens on port 3000 and proxies requests to the underlying microservices:
> - User Service on port 3001
> - Captain Service on port 3002
> - Ride Service on port 3003

---

## Table of Contents

- [User Service API](#user-service-api)
- [Captain Service API](#captain-service-api)
- [Ride Service API](#ride-service-api)
- [Additional Details](#additional-details)

---

## User Service API

The User Service handles user registration, login, logout, profile retrieval, and waiting for ride acceptance notifications.

### Endpoints

#### 1. Register User
- **Method:** POST  
- **URL:** `/register`  
- **Description:** Registers a new user.
- **Request Body:**
  - `name` (string, required) – The full name of the user.
  - `email` (string, required) – The user's unique email address.
  - `password` (string, required) – The user's password.
  
> **Example Request:**
> ```json
> {
>   "name": "John Doe",
>   "email": "john@example.com",
>   "password": "securePassword123"
> }
> ```

- **Response:**
  - **Success (200):**
    ```json
    {
      "message": "User registered successfully"
    }
    ```
  - **Failure (400/500):** Error message in JSON format.

#### 2. Login User
- **Method:** POST  
- **URL:** `/login`  
- **Description:** Authenticates a registered user.
- **Request Body:**
  - `email` (string, required)
  - `password` (string, required)
  
> **Example Request:**
> ```json
> {
>   "email": "john@example.com",
>   "password": "securePassword123"
> }
> ```

- **Response:**
  - **Success (200):**
    ```json
    {
>      "message": "User logged in successfully"
>    }
> ```
  - **Failure (400/500):** Error message in JSON format.
  
> **Note:** Upon successful login, a JWT token is sent as an HTTP-only cookie.

#### 3. Logout User
- **Method:** POST  
- **URL:** `/logout`  
- **Description:** Logs out the user by blacklisting the token and clearing the authentication cookie.
- **Response:**
  - **Success (200):**
    ```json
    {
      "message": "User logged out successfully"
    }
    ```
  - **Failure (500):** Error message.

#### 4. Get User Profile
- **Method:** GET  
- **URL:** `/profile`  
- **Description:** Retrieves the profile information for the authenticated user.
- **Response Example:**
  ```json
  {
    "_id": "userId",
    "name": "John Doe",
    "email": "john@example.com"
  }
  ```

#### 5. Accept Ride Notification (Long Polling)
- **Method:** GET  
- **URL:** `/acceptedRide`  
- **Description:** Long-polling endpoint that waits for a `ride-accepted` event.  
- **Response:**
  - **Success (200):** Returns ride data:
    ```json
    {
      "rideId": "rideId",
      "user": "userId",
      "pickup": "Pickup location",
      "destination": "Destination location",
      "status": "accepted"
    }
    ```
  - **Timeout (204):** No content if no event is received within 30 seconds.

---

## Captain Service API

The Captain Service manages registration and login for captains, handles availability settings, and listens for new ride requests.

### Endpoints

#### 1. Register Captain
- **Method:** POST  
- **URL:** `/register`  
- **Description:** Registers a new captain.
- **Request Body:**
  - `name` (string, required)
  - `email` (string, required)
  - `password` (string, required)
  
> **Example Request:**
> ```json
> {
>   "name": "Captain Jack",
>   "email": "jack@captain.com",
>   "password": "piratePass123"
> }
> ```

- **Response:**
  - **Success (200):**
    ```json
    {
      "token": "jwt-token",
      "newcaptain": {
        "_id": "captainId",
        "name": "Captain Jack",
        "email": "jack@captain.com",
        "isAvailable": false
      }
    }
    ```
  - **Failure (400/500):** Error message.

#### 2. Login Captain
- **Method:** POST  
- **URL:** `/login`  
- **Description:** Authenticates a captain.
- **Request Body:**
  - `email` (string, required)
  - `password` (string, required)
  
> **Example Request:**
> ```json
> {
>   "email": "jack@captain.com",
>   "password": "piratePass123"
> }
> ```

- **Response:**
  - **Success (200):**
    ```json
    {
      "token": "jwt-token",
      "captain": {
>         "_id": "captainId",
>         "name": "Captain Jack",
>         "email": "jack@captain.com",
>         "isAvailable": false
>       }
>     }
> ```
  - **Failure (400/500):** Error message.

#### 3. Logout Captain
- **Method:** POST  
- **URL:** `/logout`  
- **Description:** Logs out the captain by blacklisting the token.
- **Response:**
  - **Success (200):**
    ```json
    {
      "message": "captain logged out successfully"
    }
    ```
  - **Failure (500):** Error message.

#### 4. Get Captain Profile
- **Method:** GET  
- **URL:** `/profile`  
- **Description:** Retrieves the authenticated captain's profile.
- **Response Example:**
  ```json
  {
    "_id": "captainId",
    "name": "Captain Jack",
    "email": "jack@captain.com",
    "isAvailable": false
  }
  ```

#### 5. Toggle Availability
- **Method:** PATCH (or POST, based on implementation)  
- **URL:** `/toggleAvailability`  
- **Description:** Toggles the captain's availability status.
- **Response:**
  - **Success (200):** Updated captain object with the new availability status.
  
#### 6. Wait for New Ride Request (Long Polling)
- **Method:** GET  
- **URL:** `/waitForNewRide`  
- **Description:** Long-polling endpoint for captains waiting for a new ride request.
- **Response:**
  - **Success (200):** Returns new ride data when a request is received.
  - **Timeout (204):** No content if no new ride is received within 30 seconds.

---

## Ride Service API

The Ride Service handles ride creation and the acceptance flow.

### Endpoints

#### 1. Create Ride
- **Method:** POST  
- **URL:** `/createRide`  
- **Description:** Allows an authenticated user to request a new ride.
- **Request Body:**
  - `pickup` (string, required) – Pickup location.
  - `destination` (string, required) – Destination location.
  
> **Example Request:**
> ```json
> {
>   "pickup": "123 Main St",
>   "destination": "456 Elm St"
> }
> ```

- **Response:**
  - **Success (200):** Returns the new ride object:
    ```json
    {
>      "_id": "rideId",
>      "user": "userId",
>      "pickup": "123 Main St",
>      "destination": "456 Elm St",
>      "status": "requested",
>      "createdAt": "timestamp",
>      "updatedAt": "timestamp"
>    }
> ```
  - **Failure (500):** Error message.

#### 2. Accept Ride
- **Method:** GET  
- **URL:** `/acceptRide`  
- **Description:** Updates the ride status to accepted. Accepts a query parameter.
- **Query Parameters:**
  - `rideId` (string, required) – The unique identifier of the ride.
  
> **Example Request:**  
> `/acceptRide?rideId=rideId`
  
- **Response:**
  - **Success (200):** Returns the updated ride object:
    ```json
    {
>      "_id": "rideId",
>      "user": "userId",
>      "pickup": "123 Main St",
>      "destination": "456 Elm St",
>      "status": "accepted",
>      "createdAt": "timestamp",
>      "updatedAt": "timestamp"
>    }
> ```
  - **Failure (404/500):** Error message if the ride is not found or an error occurs.

---

## Data Formats

All API endpoints:
- Expect and return data in JSON format.
- Use HTTP status codes to indicate success (`200 OK`) or errors (e.g., `400 Bad Request`, `404 Not Found`, `500 Internal Server Error`).

### Request Examples

- **User/Captain Registration & Login:**
  ```json
  {
    "name": "Alice",
    "email": "alice@example.com",
    "password": "yourPassword"
  }
  ```
- **Ride Creation:**
  ```json
  {
    "pickup": "Location A",
    "destination": "Location B"
  }
  ```

### Response Examples

- **Success Response:**
  ```json
  {
    "message": "Operation completed successfully",
    "data": { ... }
  }
  ```
- **Error Response:**
  ```json
  {
    "message": "Detailed error message describing the failure."
  }
  ```

### Long Polling Endpoints

Both `/acceptedRide` (User Service) and `/waitForNewRide` (Captain Service) are designed for long polling. They will:
- Wait up to 30 seconds for new data.
- Return a `204 No Content` status if no new event occurs within the timeout period.

---

## Additional Details

- **Authentication:**  
  All login endpoints generate a JWT token that is sent as an HTTP-only cookie. Ensure these cookies are included with subsequent requests to authenticated endpoints. 

- **Database Models:**  
  The models define the required fields and validation rules. For example, the **User** and **Captain** models require a `name`, unique `email`, and a hashed `password`. The **Ride** model includes references to the user and, once assigned, the captain.

- **Messaging Queue:**  
  The services use RabbitMQ to publish and subscribe to ride events (`new-ride` and `ride-accepted`), ensuring asynchronous communication between components.

---

This documentation should serve as a comprehensive guide to the API endpoints, including details on the request and response formats, as well as operational behavior for long polling and authentication.
