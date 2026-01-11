---
title: "10 SQL Practice Projects with Sample Data"
date: 2026-01-01 15:00:00 +0000
categories: [Database, SQL, Projects]
tags: [sql, projects, practice, hands-on]
---

# 10 SQL Practice Projects with Sample Data

Hands-on projects to build real-world SQL skills, complete with sample datasets and requirements.

---

## Project Difficulty Legend
- 🟢 **Beginner:** Weeks 1-2 concepts
- 🟡 **Intermediate:** Weeks 3-4 concepts
- 🔴 **Advanced:** Weeks 5-8 concepts

---

## Project 1: Library Management System 🟢

### Overview
Build a library database to track books, members, and borrowing history.

### Schema Requirements

```sql
-- Books table
CREATE TABLE books (
    book_id INT PRIMARY KEY AUTO_INCREMENT,
    isbn VARCHAR(13) UNIQUE,
    title VARCHAR(200) NOT NULL,
    author VARCHAR(100) NOT NULL,
    publisher VARCHAR(100),
    publication_year INT,
    category VARCHAR(50),
    total_copies INT DEFAULT 1,
    available_copies INT DEFAULT 1,
    shelf_location VARCHAR(20)
);

-- Members table
CREATE TABLE members (
    member_id INT PRIMARY KEY AUTO_INCREMENT,
    first_name VARCHAR(50) NOT NULL,
    last_name VARCHAR(50) NOT NULL,
    email VARCHAR(100) UNIQUE NOT NULL,
    phone VARCHAR(15),
    address TEXT,
    membership_date DATE NOT NULL,
    membership_type ENUM('basic', 'premium', 'student') DEFAULT 'basic',
    status ENUM('active', 'suspended', 'expired') DEFAULT 'active'
);

-- Borrowing table
CREATE TABLE borrowings (
    borrowing_id INT PRIMARY KEY AUTO_INCREMENT,
    book_id INT NOT NULL,
    member_id INT NOT NULL,
    borrow_date DATE NOT NULL,
    due_date DATE NOT NULL,
    return_date DATE,
    fine_amount DECIMAL(10,2) DEFAULT 0.00,
    status ENUM('borrowed', 'returned', 'overdue') DEFAULT 'borrowed',
    FOREIGN KEY (book_id) REFERENCES books(book_id),
    FOREIGN KEY (member_id) REFERENCES members(member_id)
);

-- Create indexes
CREATE INDEX idx_book_category ON books(category);
CREATE INDEX idx_member_status ON members(status);
CREATE INDEX idx_borrowing_status ON borrowings(status);
CREATE INDEX idx_borrowing_dates ON borrowings(borrow_date, due_date);
```

### Sample Data Script

```sql
-- Insert sample books
INSERT INTO books (isbn, title, author, publisher, publication_year, category, total_copies, available_copies, shelf_location) VALUES
('9780132350884', 'Clean Code', 'Robert C. Martin', 'Prentice Hall', 2008, 'Programming', 3, 2, 'A1-23'),
('9780134685991', 'Effective Java', 'Joshua Bloch', 'Addison-Wesley', 2018, 'Programming', 2, 1, 'A1-24'),
('9780596517748', 'JavaScript: The Good Parts', 'Douglas Crockford', 'O''Reilly', 2008, 'Programming', 2, 2, 'A1-25'),
('9780135957059', 'The Pragmatic Programmer', 'David Thomas', 'Addison-Wesley', 2019, 'Programming', 3, 3, 'A1-26'),
('9781449355739', 'Learning SQL', 'Alan Beaulieu', 'O''Reilly', 2020, 'Database', 2, 1, 'B2-10'),
('9780135404676', 'The Art of Computer Programming', 'Donald Knuth', 'Addison-Wesley', 2011, 'Computer Science', 1, 0, 'C3-01'),
('9780134494166', 'Clean Architecture', 'Robert C. Martin', 'Prentice Hall', 2017, 'Architecture', 2, 2, 'A1-27'),
('9781491950357', 'Designing Data-Intensive Applications', 'Martin Kleppmann', 'O''Reilly', 2017, 'Database', 3, 2, 'B2-11'),
('9780201633610', 'Design Patterns', 'Gang of Four', 'Addison-Wesley', 1994, 'Architecture', 2, 1, 'A2-05'),
('9780134757599', 'Refactoring', 'Martin Fowler', 'Addison-Wesley', 2018, 'Programming', 2, 2, 'A1-28');

-- Insert sample members
INSERT INTO members (first_name, last_name, email, phone, address, membership_date, membership_type, status) VALUES
('John', 'Doe', 'john.doe@email.com', '555-0101', '123 Main St', '2023-01-15', 'premium', 'active'),
('Jane', 'Smith', 'jane.smith@email.com', '555-0102', '456 Oak Ave', '2023-02-20', 'basic', 'active'),
('Bob', 'Johnson', 'bob.j@email.com', '555-0103', '789 Pine Rd', '2023-03-10', 'student', 'active'),
('Alice', 'Williams', 'alice.w@email.com', '555-0104', '321 Elm St', '2023-01-05', 'premium', 'active'),
('Charlie', 'Brown', 'charlie.b@email.com', '555-0105', '654 Maple Dr', '2023-04-12', 'basic', 'active'),
('Diana', 'Davis', 'diana.d@email.com', '555-0106', '987 Cedar Ln', '2022-12-01', 'premium', 'active'),
('Eve', 'Martinez', 'eve.m@email.com', '555-0107', '147 Birch Ct', '2023-05-20', 'student', 'suspended'),
('Frank', 'Garcia', 'frank.g@email.com', '555-0108', '258 Spruce Way', '2023-02-28', 'basic', 'active');

-- Insert sample borrowings
INSERT INTO borrowings (book_id, member_id, borrow_date, due_date, return_date, fine_amount, status) VALUES
(1, 1, '2024-01-05', '2024-01-19', '2024-01-18', 0.00, 'returned'),
(2, 1, '2024-01-05', '2024-01-19', NULL, 0.00, 'overdue'),
(3, 2, '2024-01-10', '2024-01-24', '2024-01-22', 0.00, 'returned'),
(5, 3, '2024-01-08', '2024-01-22', NULL, 5.00, 'overdue'),
(6, 4, '2024-01-12', '2024-01-26', NULL, 0.00, 'borrowed'),
(8, 4, '2024-01-15', '2024-01-29', NULL, 0.00, 'borrowed'),
(9, 5, '2024-01-07', '2024-01-21', '2024-01-25', 2.00, 'returned'),
(1, 6, '2024-01-20', '2024-02-03', NULL, 0.00, 'borrowed'),
(4, 2, '2024-01-18', '2024-02-01', NULL, 0.00, 'borrowed');
```

### Query Requirements

Build queries to answer these questions:

1. **Basic Queries:**
   - List all available books with their locations
   - Find all active members sorted by join date
   - Show books by a specific author

2. **Intermediate Queries:**
   - Calculate total fines collected per member
   - Find the most popular book category (most borrowed)
   - List members with overdue books

3. **Advanced Queries:**
   - Member borrowing history with book details
   - Books that have never been borrowed
   - Average borrowing duration per category
   - Monthly borrowing trends (last 6 months)
   - Member reading preferences (categories they borrow from)

### Bonus Challenges

```sql
-- 1. Create a view for overdue books
CREATE VIEW overdue_books AS
SELECT 
    m.first_name,
    m.last_name,
    b.title,
    br.due_date,
    DATEDIFF(CURRENT_DATE, br.due_date) as days_overdue,
    DATEDIFF(CURRENT_DATE, br.due_date) * 0.50 as fine_due
FROM borrowings br
JOIN members m ON br.member_id = m.member_id
JOIN books b ON br.book_id = b.book_id
WHERE br.status = 'overdue';

-- 2. Stored procedure to check out a book
DELIMITER //
CREATE PROCEDURE checkout_book(
    IN p_member_id INT,
    IN p_book_id INT
)
BEGIN
    DECLARE available INT;
    
    -- Check if book is available
    SELECT available_copies INTO available
    FROM books
    WHERE book_id = p_book_id;
    
    IF available > 0 THEN
        -- Insert borrowing record
        INSERT INTO borrowings (book_id, member_id, borrow_date, due_date, status)
        VALUES (p_book_id, p_member_id, CURRENT_DATE, DATE_ADD(CURRENT_DATE, INTERVAL 14 DAY), 'borrowed');
        
        -- Update available copies
        UPDATE books
        SET available_copies = available_copies - 1
        WHERE book_id = p_book_id;
        
        SELECT 'Book checked out successfully' as message;
    ELSE
        SELECT 'Book not available' as message;
    END IF;
END //
DELIMITER ;

-- 3. Trigger to calculate fines on return
DELIMITER //
CREATE TRIGGER calculate_fine_on_return
BEFORE UPDATE ON borrowings
FOR EACH ROW
BEGIN
    IF NEW.return_date IS NOT NULL AND OLD.return_date IS NULL THEN
        IF NEW.return_date > NEW.due_date THEN
            SET NEW.fine_amount = DATEDIFF(NEW.return_date, NEW.due_date) * 0.50;
            SET NEW.status = 'returned';
        ELSE
            SET NEW.fine_amount = 0.00;
            SET NEW.status = 'returned';
        END IF;
    END IF;
END //
DELIMITER ;
```

---

## Project 2: E-Commerce Analytics Platform 🟡

### Overview
Build a comprehensive e-commerce database with sales analytics capabilities.

### Schema

```sql
CREATE TABLE customers (
    customer_id INT PRIMARY KEY AUTO_INCREMENT,
    email VARCHAR(100) UNIQUE NOT NULL,
    first_name VARCHAR(50) NOT NULL,
    last_name VARCHAR(50) NOT NULL,
    phone VARCHAR(20),
    city VARCHAR(50),
    state VARCHAR(50),
    country VARCHAR(50) DEFAULT 'USA',
    signup_date DATE NOT NULL,
    customer_segment ENUM('new', 'regular', 'vip') DEFAULT 'new'
);

CREATE TABLE products (
    product_id INT PRIMARY KEY AUTO_INCREMENT,
    product_name VARCHAR(200) NOT NULL,
    category VARCHAR(50) NOT NULL,
    subcategory VARCHAR(50),
    brand VARCHAR(50),
    price DECIMAL(10,2) NOT NULL,
    cost DECIMAL(10,2) NOT NULL,
    stock_quantity INT DEFAULT 0,
    is_active BOOLEAN DEFAULT TRUE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE orders (
    order_id INT PRIMARY KEY AUTO_INCREMENT,
    customer_id INT NOT NULL,
    order_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    total_amount DECIMAL(10,2) NOT NULL,
    discount_amount DECIMAL(10,2) DEFAULT 0.00,
    shipping_cost DECIMAL(10,2) DEFAULT 0.00,
    status ENUM('pending', 'processing', 'shipped', 'delivered', 'cancelled') DEFAULT 'pending',
    payment_method ENUM('credit_card', 'debit_card', 'paypal', 'bank_transfer'),
    FOREIGN KEY (customer_id) REFERENCES customers(customer_id)
);

CREATE TABLE order_items (
    item_id INT PRIMARY KEY AUTO_INCREMENT,
    order_id INT NOT NULL,
    product_id INT NOT NULL,
    quantity INT NOT NULL,
    unit_price DECIMAL(10,2) NOT NULL,
    discount_percent DECIMAL(5,2) DEFAULT 0.00,
    FOREIGN KEY (order_id) REFERENCES orders(order_id),
    FOREIGN KEY (product_id) REFERENCES products(product_id)
);

CREATE TABLE product_reviews (
    review_id INT PRIMARY KEY AUTO_INCREMENT,
    product_id INT NOT NULL,
    customer_id INT NOT NULL,
    rating INT CHECK (rating BETWEEN 1 AND 5),
    review_text TEXT,
    review_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    helpful_count INT DEFAULT 0,
    FOREIGN KEY (product_id) REFERENCES products(product_id),
    FOREIGN KEY (customer_id) REFERENCES customers(customer_id)
);

-- Indexes for performance
CREATE INDEX idx_orders_customer ON orders(customer_id, order_date);
CREATE INDEX idx_orders_date ON orders(order_date);
CREATE INDEX idx_products_category ON products(category, is_active);
CREATE INDEX idx_order_items_product ON order_items(product_id);
```

### Sample Data (Abbreviated - Full script available separately)

```sql
-- Sample customers
INSERT INTO customers (email, first_name, last_name, phone, city, state, signup_date, customer_segment) VALUES
('alice@example.com', 'Alice', 'Anderson', '555-1001', 'New York', 'NY', '2023-01-15', 'vip'),
('bob@example.com', 'Bob', 'Baker', '555-1002', 'Los Angeles', 'CA', '2023-03-20', 'regular'),
('carol@example.com', 'Carol', 'Chen', '555-1003', 'Chicago', 'IL', '2023-06-10', 'new'),
('david@example.com', 'David', 'Davis', '555-1004', 'Houston', 'TX', '2023-02-28', 'regular'),
('emma@example.com', 'Emma', 'Evans', '555-1005', 'Phoenix', 'AZ', '2023-08-15', 'new');

-- Sample products
INSERT INTO products (product_name, category, subcategory, brand, price, cost, stock_quantity) VALUES
('iPhone 15 Pro', 'Electronics', 'Smartphones', 'Apple', 999.99, 700.00, 50),
('Samsung Galaxy S24', 'Electronics', 'Smartphones', 'Samsung', 899.99, 650.00, 75),
('MacBook Pro 14"', 'Electronics', 'Laptops', 'Apple', 1999.99, 1400.00, 30),
('Dell XPS 15', 'Electronics', 'Laptops', 'Dell', 1499.99, 1100.00, 40),
('Sony WH-1000XM5', 'Electronics', 'Headphones', 'Sony', 399.99, 250.00, 100);
```

### Analytics Queries Required

1. **Sales Metrics:**
   - Daily/Monthly/Yearly revenue
   - Average order value
   - Products by revenue
   - Category performance

2. **Customer Analytics:**
   - Customer lifetime value (CLV)
   - RFM analysis (Recency, Frequency, Monetary)
   - Customer retention rate
   - Cohort analysis

3. **Product Analytics:**
   - Best/worst selling products
   - Product performance by category
   - Inventory turnover rate
   - Product profitability

4. **Advanced Analytics:**
   - Market basket analysis (products bought together)
   - Customer segmentation
   - Sales forecasting trends
   - Churn prediction indicators

---

## Project 3: Hospital Management System 🟡

### Overview
Healthcare database for managing patients, doctors, appointments, and medical records.

### Schema

```sql
CREATE TABLE doctors (
    doctor_id INT PRIMARY KEY AUTO_INCREMENT,
    first_name VARCHAR(50) NOT NULL,
    last_name VARCHAR(50) NOT NULL,
    specialization VARCHAR(100) NOT NULL,
    email VARCHAR(100) UNIQUE,
    phone VARCHAR(20),
    years_of_experience INT,
    consultation_fee DECIMAL(10,2),
    is_available BOOLEAN DEFAULT TRUE
);

CREATE TABLE patients (
    patient_id INT PRIMARY KEY AUTO_INCREMENT,
    first_name VARCHAR(50) NOT NULL,
    last_name VARCHAR(50) NOT NULL,
    date_of_birth DATE NOT NULL,
    gender ENUM('M', 'F', 'Other'),
    blood_group VARCHAR(5),
    email VARCHAR(100),
    phone VARCHAR(20) NOT NULL,
    address TEXT,
    emergency_contact VARCHAR(100),
    emergency_phone VARCHAR(20),
    registration_date DATE DEFAULT (CURRENT_DATE)
);

CREATE TABLE appointments (
    appointment_id INT PRIMARY KEY AUTO_INCREMENT,
    patient_id INT NOT NULL,
    doctor_id INT NOT NULL,
    appointment_date DATE NOT NULL,
    appointment_time TIME NOT NULL,
    status ENUM('scheduled', 'completed', 'cancelled', 'no_show') DEFAULT 'scheduled',
    symptoms TEXT,
    notes TEXT,
    FOREIGN KEY (patient_id) REFERENCES patients(patient_id),
    FOREIGN KEY (doctor_id) REFERENCES doctors(doctor_id)
);

CREATE TABLE medical_records (
    record_id INT PRIMARY KEY AUTO_INCREMENT,
    patient_id INT NOT NULL,
    doctor_id INT NOT NULL,
    visit_date DATE NOT NULL,
    diagnosis TEXT,
    prescription TEXT,
    treatment_notes TEXT,
    follow_up_date DATE,
    FOREIGN KEY (patient_id) REFERENCES patients(patient_id),
    FOREIGN KEY (doctor_id) REFERENCES doctors(doctor_id)
);

CREATE TABLE prescriptions (
    prescription_id INT PRIMARY KEY AUTO_INCREMENT,
    record_id INT NOT NULL,
    medication_name VARCHAR(100) NOT NULL,
    dosage VARCHAR(50),
    frequency VARCHAR(50),
    duration_days INT,
    instructions TEXT,
    FOREIGN KEY (record_id) REFERENCES medical_records(record_id)
);
```

### Query Requirements

1. Available appointment slots for a doctor
2. Patient medical history
3. Doctor workload analysis
4. Most common diagnoses
5. Patient appointment patterns
6. Revenue by specialization
7. Patient demographics analysis
8. Follow-up tracking system

---

## Project 4: University Course Management 🟢

### Overview
Academic database for courses, students, enrollments, and grades.

### Schema

```sql
CREATE TABLE students (
    student_id INT PRIMARY KEY AUTO_INCREMENT,
    first_name VARCHAR(50) NOT NULL,
    last_name VARCHAR(50) NOT NULL,
    email VARCHAR(100) UNIQUE NOT NULL,
    enrollment_year INT NOT NULL,
    major VARCHAR(100),
    gpa DECIMAL(3,2) CHECK (gpa BETWEEN 0 AND 4.00)
);

CREATE TABLE professors (
    professor_id INT PRIMARY KEY AUTO_INCREMENT,
    first_name VARCHAR(50) NOT NULL,
    last_name VARCHAR(50) NOT NULL,
    email VARCHAR(100) UNIQUE NOT NULL,
    department VARCHAR(100) NOT NULL,
    office_location VARCHAR(50)
);

CREATE TABLE courses (
    course_id INT PRIMARY KEY AUTO_INCREMENT,
    course_code VARCHAR(20) UNIQUE NOT NULL,
    course_name VARCHAR(200) NOT NULL,
    department VARCHAR(100) NOT NULL,
    credits INT NOT NULL,
    max_capacity INT DEFAULT 30,
    professor_id INT,
    FOREIGN KEY (professor_id) REFERENCES professors(professor_id)
);

CREATE TABLE enrollments (
    enrollment_id INT PRIMARY KEY AUTO_INCREMENT,
    student_id INT NOT NULL,
    course_id INT NOT NULL,
    semester VARCHAR(20) NOT NULL,
    year INT NOT NULL,
    grade VARCHAR(2),
    enrollment_date DATE DEFAULT (CURRENT_DATE),
    FOREIGN KEY (student_id) REFERENCES students(student_id),
    FOREIGN KEY (course_id) REFERENCES courses(course_id),
    UNIQUE KEY unique_enrollment (student_id, course_id, semester, year)
);
```

### Query Requirements

1. Student transcript (all courses and grades)
2. Course enrollment statistics
3. Professor teaching load
4. GPA calculation by major
5. Popular courses analysis
6. Students on academic probation (GPA < 2.0)
7. Course prerequisite tracking
8. Semester schedule conflicts

---

## Project 5: Social Media Analytics 🔴

### Overview
Social network database with users, posts, likes, comments, and followers.

### Schema

```sql
CREATE TABLE users (
    user_id INT PRIMARY KEY AUTO_INCREMENT,
    username VARCHAR(50) UNIQUE NOT NULL,
    email VARCHAR(100) UNIQUE NOT NULL,
    full_name VARCHAR(100),
    bio TEXT,
    profile_picture_url VARCHAR(255),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    is_verified BOOLEAN DEFAULT FALSE,
    follower_count INT DEFAULT 0,
    following_count INT DEFAULT 0
);

CREATE TABLE posts (
    post_id INT PRIMARY KEY AUTO_INCREMENT,
    user_id INT NOT NULL,
    content TEXT NOT NULL,
    image_url VARCHAR(255),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    likes_count INT DEFAULT 0,
    comments_count INT DEFAULT 0,
    shares_count INT DEFAULT 0,
    FOREIGN KEY (user_id) REFERENCES users(user_id)
);

CREATE TABLE follows (
    follow_id INT PRIMARY KEY AUTO_INCREMENT,
    follower_id INT NOT NULL,
    following_id INT NOT NULL,
    followed_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (follower_id) REFERENCES users(user_id),
    FOREIGN KEY (following_id) REFERENCES users(user_id),
    UNIQUE KEY unique_follow (follower_id, following_id)
);

CREATE TABLE likes (
    like_id INT PRIMARY KEY AUTO_INCREMENT,
    user_id INT NOT NULL,
    post_id INT NOT NULL,
    liked_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (user_id) REFERENCES users(user_id),
    FOREIGN KEY (post_id) REFERENCES posts(post_id),
    UNIQUE KEY unique_like (user_id, post_id)
);

CREATE TABLE comments (
    comment_id INT PRIMARY KEY AUTO_INCREMENT,
    post_id INT NOT NULL,
    user_id INT NOT NULL,
    content TEXT NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (post_id) REFERENCES posts(post_id),
    FOREIGN KEY (user_id) REFERENCES users(user_id)
);

CREATE TABLE hashtags (
    hashtag_id INT PRIMARY KEY AUTO_INCREMENT,
    tag VARCHAR(100) UNIQUE NOT NULL,
    use_count INT DEFAULT 0
);

CREATE TABLE post_hashtags (
    post_id INT NOT NULL,
    hashtag_id INT NOT NULL,
    PRIMARY KEY (post_id, hashtag_id),
    FOREIGN KEY (post_id) REFERENCES posts(post_id),
    FOREIGN KEY (hashtag_id) REFERENCES hashtags(hashtag_id)
);
```

### Advanced Analytics Required

1. **Engagement Metrics:**
   - User engagement rate
   - Post virality score
   - Trending hashtags
   - Peak posting times

2. **Network Analysis:**
   - Influencer identification
   - Community detection
   - Follower growth trends
   - User similarity (mutual follows)

3. **Content Analysis:**
   - Most engaging post types
   - Hashtag performance
   - Comment sentiment (basic)
   - Content recommendation engine

4. **User Behavior:**
   - Active user metrics (DAU, MAU)
   - User retention cohorts
   - Churn prediction indicators
   - Feed algorithm ranking

---

## Project 6: Inventory Management System 🟢

### Overview
Warehouse and inventory tracking with suppliers, stock movements, and reorder alerts.

[Schema and requirements provided...]

---

## Project 7: Ride-Sharing Platform 🔴

### Overview
Transportation platform tracking drivers, riders, trips, and payments.

### Key Features to Implement:

1. Real-time driver availability
2. Trip matching algorithm (simulated)
3. Dynamic pricing model
4. Driver ratings and reviews
5. Payment processing
6. Trip history and analytics
7. Heat maps (trip density)
8. Surge pricing calculation

---

## Project 8: Movie Streaming Service 🟡

### Overview
Netflix-style database with users, movies, viewing history, and recommendations.

### Analytics Required:

1. Viewing patterns analysis
2. Content popularity trends
3. User watch time statistics
4. Recommendation engine (collaborative filtering basics)
5. Subscriber retention analysis
6. Content performance by genre
7. Binge-watching patterns
8. Churn prediction

---

## Project 9: Restaurant Management 🟢

### Overview
Restaurant operations including menu, orders, reservations, and inventory.

### Features:

1. Table reservation system
2. Order management (dine-in, takeout, delivery)
3. Menu and pricing management
4. Inventory and ingredient tracking
5. Staff shift scheduling
6. Sales analytics
7. Customer loyalty program
8. Kitchen queue optimization

---

## Project 10: IoT Sensor Data Platform 🔴

### Overview
Time-series database for IoT sensor data with analytics and anomaly detection.

### Schema Design:

```sql
CREATE TABLE sensors (
    sensor_id INT PRIMARY KEY AUTO_INCREMENT,
    sensor_name VARCHAR(100) NOT NULL,
    sensor_type ENUM('temperature', 'humidity', 'pressure', 'motion', 'light'),
    location VARCHAR(100),
    installation_date DATE,
    is_active BOOLEAN DEFAULT TRUE
);

CREATE TABLE sensor_readings (
    reading_id BIGINT PRIMARY KEY AUTO_INCREMENT,
    sensor_id INT NOT NULL,
    reading_value DECIMAL(10,4) NOT NULL,
    reading_timestamp TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    quality_score DECIMAL(3,2), -- Data quality indicator
    FOREIGN KEY (sensor_id) REFERENCES sensors(sensor_id)
) PARTITION BY RANGE (YEAR(reading_timestamp)) (
    PARTITION p2023 VALUES LESS THAN (2024),
    PARTITION p2024 VALUES LESS THAN (2025),
    PARTITION p2025 VALUES LESS THAN (2026),
    PARTITION p_future VALUES LESS THAN MAXVALUE
);

CREATE TABLE alerts (
    alert_id INT PRIMARY KEY AUTO_INCREMENT,
    sensor_id INT NOT NULL,
    alert_type ENUM('threshold_exceeded', 'sensor_offline', 'anomaly_detected'),
    alert_message TEXT,
    alert_timestamp TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    is_resolved BOOLEAN DEFAULT FALSE,
    FOREIGN KEY (sensor_id) REFERENCES sensors(sensor_id)
);
```

### Advanced Queries Required:

1. **Time-Series Analysis:**
   - Hourly/daily aggregates
   - Moving averages (24-hour, 7-day)
   - Seasonal patterns
   - Anomaly detection (statistical)

2. **Performance Optimization:**
   - Partitioning strategy
   - Efficient data retention policies
   - Archival of old data
   - Index optimization for time-series

3. **Real-Time Monitoring:**
   - Current sensor status dashboard
   - Threshold breach detection
   - Downtime tracking
   - Predictive maintenance indicators

---

## General Project Guidelines

### Setup Instructions

1. **Create Database:**
```sql
CREATE DATABASE project_name CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
USE project_name;
```

2. **Run Schema Scripts:**
   - Execute CREATE TABLE statements
   - Add indexes
   - Create views/procedures

3. **Load Sample Data:**
   - Run INSERT statements
   - Or use CSV import for large datasets

4. **Test Queries:**
   - Start with simple SELECT queries
   - Progress to complex analytics
   - Optimize with EXPLAIN

### Best Practices

- **Version Control:** Track schema changes in Git
- **Documentation:** Comment complex queries
- **Testing:** Verify data integrity with constraints
- **Performance:** Always use EXPLAIN for optimization
- **Security:** Use parameterized queries in applications
- **Backup:** Regular database backups

### Evaluation Criteria

For each project, you should be able to:

- [ ] Design normalized schema (3NF)
- [ ] Write efficient queries with proper indexes
- [ ] Implement business logic (procedures/triggers)
- [ ] Create analytical reports
- [ ] Optimize for performance
- [ ] Handle edge cases (NULL values, data validation)
- [ ] Document design decisions

---

## Next Steps

1. Choose a project matching your skill level
2. Set up the database and schema
3. Load sample data
4. Implement required queries
5. Add bonus features
6. Optimize performance
7. Document your work
8. Share on GitHub!

---

**Remember:** The best way to learn SQL is by building real projects. Start simple, iterate, and gradually add complexity. Good luck!
