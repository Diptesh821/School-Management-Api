## **Important Note on Duplicate Entries**

This API **prevents** adding a new school if there is already an existing school with the **same `name` and `address`**. To avoid an error response when adding a school:

1. **List Existing Schools First**  
   - Make a **GET** request to `/listSchools` (with any valid latitude and longitude).  
   - Review the returned list of schools to see which names and addresses are already in the database.

2. **Check for Duplicates**  
   - If you see a school with the **same** `name` and `address` as what you plan to add, either:
     - Change the `name` or `address`, or
     - Use a different school altogether.

3. **Then Add Your School**  
   - Make the **POST** request to `/addSchool` with a **unique** combination of `name` and `address`.

4. **Error on Duplicate**  
   - If you still attempt to add a school with a `name` and `address` that already exists, the API will return an **error** (e.g., `500 internal server error`), indicating that duplicate entries are not allowed.

---





# **School Management API**

A **Node.js** and **Express.js** application for managing school data in a **MySQL** database. This API allows users to **add new schools** and **retrieve a list of schools** sorted by proximity to a specified location. It includes **validation** rules (e.g., preventing duplicates, enforcing name length limits) and **error handling** to ensure data integrity.

---

## **Table of Contents**

1. [Key Features](#key-features)  
2. [Tech Stack](#tech-stack)  
3. [Project Structure](#project-structure)  
4. [Prerequisites](#prerequisites)  
5. [Installation](#installation)  
6. [Configuration](#configuration)  
7. [Database Setup](#database-setup)  
8. [Usage & Endpoints](#usage--endpoints)  
   - [Add School (POST /addSchool)](#1-add-school-post-addschool)  
   - [List Schools (GET /listSchools)](#2-list-schools-get-listschools)  
9. [Validation & Constraints](#validation--constraints)  
   - [Duplicate (Name + Address)](#duplicate-name--address)  
   - [Name Length](#name-length)  
   - [Address, Latitude, Longitude Checks](#address-latitude-longitude-checks)  
10. [Sample Data](#sample-data)  
11. [Testing with Postman](#testing-with-postman)  
   - [Examples in Postman](#examples-in-postman)  
12. [Deployment](#deployment)  
13. [Recommended Workflow](#recommended-workflow)  
14. [Contact Information](#contact-information)  

---

## **Key Features**

- **Add School**: Insert a new school record with `name`, `address`, `latitude`, and `longitude`.
- **List Schools**: Retrieve all schools, sorted by their **distance** from a user-supplied location.
- **Validation & Error Handling**:  
  - Duplicate `(name, address)` detection.  
  - Minimum/maximum name length enforcement.  
  - Proper data types for `latitude` and `longitude`.
- **MySQL Integration**: Uses a relational table `schools` with primary key and custom constraints.
- **Postman Collection**: A well-documented collection for easy testing and demonstration.

---

## **Tech Stack**

- **Node.js**: JavaScript runtime environment.  
- **Express.js**: Fast, unopinionated web framework for Node.js.  
- **MySQL**: Relational database management system.  
- **MySQL2**: Node.js driver for MySQL (supports async/await).  
- **Dotenv**: For environment variable management.  
- **Nodemon** (optional): For automatic server restarts in development.

---

## **Project Structure**

```
school-management-api/
├── db.js                 # MySQL connection pool
├── index.js              # Main Express server
├── models/
│   └── School.js         # Encapsulates DB operations for 'schools'
├── routes/
│   └── addSchool.js
|   └── listSchools.js
├── middlewares/
│   └── validateSchool.js # Custom validation middleware (optional)
├── .env                  # Environment variables (ignored by Git)
├── package.json
├── package-lock.json
└── README.md             # Project documentation
```

---

## **Prerequisites**

- **Node.js** (v14+ recommended)  
- **MySQL** installed (or a hosted MySQL database, e.g., PlanetScale, Railway, freesqldatabase.com etc.)  
- **npm** (comes with Node.js)

---

## **Installation**

1. **Clone the repository** (or download the ZIP):
   ```bash
   git clone https://github.com/your-username/school-management-api.git
   cd school-management-api
   ```
2. **Install dependencies**:
   ```bash
   npm install
   ```
3. **(Optional)** Install `nodemon` globally if you want auto-reload:
   ```bash
   npm install nodemon --save-dev
   ```

---

## **Configuration**

1. Create a **.env** file in the project root:
   ```bash
   DB_HOST=localhost
   DB_USER=root
   DB_PASSWORD=your_mysql_password
   DB_NAME=school_management
   PORT=3000
   ```
2. Adjust the variables as needed for your environment or hosting provider.

---

## **Database Setup**

1. **Start MySQL** and log in:
   ```bash
   mysql -u root -p
   ```
2. **Create the database**:
   ```sql
   CREATE DATABASE school_management;
   USE school_management;
   ```
3. **Create the `schools` table**:
   ```sql
   CREATE TABLE schools (
     id INT AUTO_INCREMENT PRIMARY KEY,
     name VARCHAR(255) NOT NULL,
     address VARCHAR(255) NOT NULL,
     latitude FLOAT NOT NULL,
     longitude FLOAT NOT NULL
   );
   ```
4. **(Optional)** Add a **unique composite constraint** if you want `(name, address)` to be unique:
   ```sql
   ALTER TABLE schools ADD UNIQUE (name, address);
   ```

---

## **Usage & Endpoints**

### **Start the Server**

```bash
npm start
# or
nodemon index.js
```
The server defaults to **http://localhost:3000** (or the port specified in `.env`).

---

### **1. Add School (POST `/addSchool`)**

- **Description**: Inserts a new school record into the database.  
- **Request Body (JSON)**:
  ```json
  {
    "name": "Greenwood High",
    "address": "123 Main St",
    "latitude": 12.9716,
    "longitude": 77.5946
  }
  ```
- **Responses**:
  - **201 Created**:  
    ```json
    {
      "message": "School added successfully",
      "id": 1
    }
    ```
  - **400 Bad Request** if validation fails (e.g., missing fields, invalid data types, name too short/long, or duplicate entry).
  - **500 Internal Server Error** for database issues.

---

### **2. List Schools (GET `/listSchools`)**

- **Description**: Retrieves all schools from the database and sorts them by distance from the provided coordinates.
- **Query Parameters**:
  - `latitude` (required)
  - `longitude` (required)
- **Example**:
  ```
  GET /listSchools?latitude=12.9716&longitude=77.5946
  ```
- **Responses**:
  - **200 OK**: Returns an array of school objects, each including a `distance` field:
    ```json
    [
      {
        "id": 1,
        "name": "Greenwood High",
        "address": "123 Main St",
        "latitude": 12.9716,
        "longitude": 77.5946,
        "distance": 0
      },
      ...
    ]
    ```
  - **400 Bad Request** if `latitude` or `longitude` is missing or invalid.
  - **500 Internal Server Error** for database issues.

---

## **Validation & Constraints**

### **Duplicate (Name + Address)**
- The API **rejects** adding a new school if **both** `name` and `address` match an existing record.
- **Error**: 400 Bad Request (or 409 Conflict) if duplicate is detected.
- **Recommendation**: Always **list existing schools** first to see what’s in the database before adding a new one.

### **Name Length**
- **Minimum** length: 2 characters  
- **Maximum** length: 100 characters  
- **Error**: 400 Bad Request if out of range.

### **Address, Latitude, Longitude Checks**
- **Address**: Non-empty string, max 255 characters.  
- **Latitude & Longitude**: Must be valid numbers (floats). Negative values allowed for Western/Southern hemispheres.

---

## **Sample Data**

To see sorting in action, you can **POST** these five schools:

```json
{
  "name": "Greenwood High",
  "address": "12 MG Road",
  "latitude": 12.9716,
  "longitude": 77.5946
}
{
  "name": "Eastside School",
  "address": "14 Church Street",
  "latitude": 12.9769,
  "longitude": 77.5998
}
{
  "name": "Riverside Academy",
  "address": "101 Riverside Lane",
  "latitude": 12.9833,
  "longitude": 77.6055
}
{
  "name": "Maple Leaf School",
  "address": "560 Maple Street",
  "latitude": 12.9604,
  "longitude": 77.5839
}
{
  "name": "St. Joseph's School",
  "address": "25 Brigade Road",
  "latitude": 12.9537,
  "longitude": 77.5988
}
```

Then call:
```
GET /listSchools?latitude=12.9716&longitude=77.5946
```
Observe schools sorted by proximity to **(12.9716, 77.5946)**.

---

## **Testing with Postman**

1. **Import** the Postman collection (JSON file).
2. **Requests** in the collection:
   - **Add School** (POST): A sample JSON body is provided.  
   - **List Schools** (GET): Query params for latitude/longitude.
3. **Multiple Examples**:  
   - Valid input, missing fields, invalid data types, duplicate entries, and name length errors.  
   - Check the **Examples** tab in Postman to see these scenarios.

### **Examples in Postman**
- **Add School**:
  - “Valid Input - Success”
  - “Name Too Short or Too Long - Error”
  - “Address Too Short or Too Long - Error”
  - “Duplicate Name + Address - Error”
  - “Missing Field - Error”
- **List Schools**:
  - “Valid Coordinates - Success”
  - “Missing Latitude/Longitude - Error”
  - “Negative Coordinates - Success”

---

## **Deployment**

- **Local Machine**: Run `npm start`.  
- **Hosting**:
  - **Render/Railway**: Push your code to GitHub, link the repo, set environment variables (DB_HOST, DB_USER, DB_PASSWORD, DB_NAME, etc.), and deploy.
  - **PlanetScale** (MySQL hosting) or **Railway MySQL** if you need a cloud DB or freesqldatabase.com(completely free).  
  - Update your `.env` with the new credentials for production.

---

## **Recommended Workflow**

1. **Check Existing Schools**:  
   - `GET /listSchools?latitude=<lat>&longitude=<lon>`  
   - Ensure you’re not duplicating `name` + `address`.
2. **Add New School**:  
   - `POST /addSchool` with a unique `name` and `address`.
3. **List Schools** again to see the newly added record and confirm sorting.
4. **Validate** error cases (short/long name, duplicates, missing fields) if needed.

---

## **Contact Information**

- **Name**: *Diptesh Singh*  
- **Skills**: Node.js (4/5), Express.js, MySQL  
- **Resume**: https://drive.google.com/file/d/1pQjm-0iAHaBxwE2fHEi9GUavw8nTmb83/view?usp=drivesdk
- **Phone**: *7847890495*  
- **Email**: *dipteshpiku@gmail.com*  
- **Source Code**: https://github.com/Diptesh821/Educase-Assignment
- **Live Endpoints**: *https://educase-assignment-tfkm.onrender.com*  
- **Postman Collection**: https://drive.google.com/file/d/1pQmo7Iphu5k7didULoz7K3vQ52Pvr1tR/view?usp=drivesdk

> For any queries regarding this API, feel free to reach out via email or GitHub issues.

---

This project demonstrates my ability to build a **RESTful Node.js** service with **MySQL** integration, **validation**, **distance-based sorting**, and comprehensive **Postman** testing.

