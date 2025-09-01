# PHP Web Application README

This README provides an overview of the PHP web application code snippets provided, covering database connections, user authentication, CRUD operations, file handling, email functionality, security measures, and more.

## Table of Contents
1. [Project Setup](#project-setup)
2. [Database Connection](#database-connection)
3. [User Authentication](#user-authentication)
4. [CRUD Operations](#crud-operations)
5. [File Handling](#file-handling)
6. [Email Sending](#email-sending)
7. [Security Measures](#security-measures)
8. [Pagination](#pagination)
9. [URL Rewriting](#url-rewriting)
10. [Date and Time Utilities](#date-and-time-utilities)
11. [JSON Handling](#json-handling)
12. [Video Duration Conversion](#video-duration-conversion)
13. [Automatic Logout](#automatic-logout)
14. [Export to CSV](#export-to-csv)
15. [Error Handling](#error-handling)

## Project Setup
- **Base URL Configuration**: Define `BASE_URL` and `ADMIN_URL` for consistent URL management:
  ```php
  define("BASE_URL", "http://localhost/project/");
  define("ADMIN_URL", BASE_URL."admin/");
  ```
- **Dependencies**: Install PHPMailer for email functionality using Composer:
  ```bash
  composer require phpmailer/phpmailer
  ```
- **Directory Structure**: Ensure an `uploads/` directory for file uploads and `vendor/` for Composer dependencies.

## Database Connection
Connect to a MySQL database using PDO with error handling:
```php
$dbhost = 'localhost';
$dbname = 'db_name';
$dbuser = 'db_username';
$dbpass = 'db_pass';
try {
    $pdo = new PDO("mysql:host={$dbhost};dbname={$dbname}", $dbuser, $dbpass);
    $pdo->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION);
} catch (PDOException $exception) {
    echo "Connection error: " . $exception->getMessage();
}
```

## User Authentication
- **Login**: Validates email and password, stores user data in session, and redirects to the admin dashboard:
  ```php
  if ($_POST['email'] == '') { throw new Exception("Email can not be empty"); }
  if (!filter_var($_POST['email'], FILTER_VALIDATE_EMAIL)) { throw new Exception("Email is invalid"); }
  if ($_POST['password'] == '') { throw new Exception("Password can not be empty"); }
  $q = $pdo->prepare("SELECT * FROM admins WHERE email=?");
  $q->execute([$_POST['email']]);
  // Additional logic for password verification and session management
  ```
- **Registration**: Hashes passwords, generates a token, and sends a verification email.
- **Password Reset**: Sends a reset link via email and updates the password upon verification.
- **Logout**: Clears session data and redirects to the login page.

## CRUD Operations
Perform Create, Read, Update, and Delete operations using prepared statements:
- **Insert**:
  ```php
  $statement = $pdo->prepare("INSERT INTO users (firstname, lastname) VALUES (?, ?)");
  $statement->execute([$_POST['firstname'], $_POST['lastname']]);
  ```
- **Update**, **Delete**, and **Select** operations follow similar patterns with prepared statements for security.

## File Handling
- **File Upload**: Validates and processes image/PDF uploads, resizes images:
  ```php
  $path = $_FILES['my_file']['name'];
  $path_tmp = $_FILES['my_file']['tmp_name'];
  $extension = pathinfo($path, PATHINFO_EXTENSION);
  $filename = time() . "." . $extension;
  // Validation and resizing logic
  ```
- **Image Resize**: Resizes images to specified dimensions:
  ```php
  function image_resize($source, $destination, $new_width=100, $new_height=100) {
      // Image resizing logic for JPEG, PNG, GIF
  }
  ```
- **File Delete**: Removes files from the server:
  ```php
  unlink('uploads/1674955499-small.jpg');
  ```

## Email Sending
Uses PHPMailer for sending emails (e.g., registration verification, password reset):
```php
use PHPMailer\PHPMailer\PHPMailer;
$mail = new PHPMailer(true);
try {
    $mail->isSMTP();
    $mail->Host = SMTP_HOST;
    // SMTP configuration and email sending logic
} catch (Exception $e) {
    echo "Message could not be sent. Mailer Error: {$mail->ErrorInfo}";
}
```

## Security Measures
- **SQL Injection**: Use prepared statements and type casting for GET parameters.
- **XSS Prevention**: Sanitize inputs with `strip_tags` and output with `htmlspecialchars`.
- **Session Timeout**: Automatically logs out users after inactivity:
  ```php
  $sessionTimeout = 60; // 1 minute
  if (isset($_SESSION['last_activity']) && (time() - $_SESSION['last_activity']) > $sessionTimeout) {
      session_unset();
      session_destroy();
      header('location: login.php');
      exit;
  }
  ```

## Pagination
Implements pagination for large datasets, with optional category/slug support:
```php
$per_page = 4;
$q = $pdo->prepare("SELECT * FROM students");
$q->execute();
$total = $q->rowCount();
$total_pages = ceil($total / $per_page);
// Pagination logic for displaying records and navigation links
```

## URL Rewriting
Uses `.htaccess` for clean URLs:
```apache
RewriteEngine On
RewriteRule ^category/([^/]+)/(\d+)$ category.php?slug=$1&p=$2 [L]
```
Transforms URLs like `category/travel/3` to `category.php?slug=travel&p=3`.

## Date and Time Utilities
- **Format Date**: Displays formatted dates/times:
  ```php
  date("Y-m-d H:i:s"); // e.g., 2023-08-09 13:49:58
  ```
- **Time Difference and Age Calculation**: Calculates differences between dates/times.
- **Time Ago**: Displays relative time (e.g., "3 hours ago").

## JSON Handling
- **Encode**: Converts arrays to JSON:
  ```php
  $studentInfo = ['firstName' => 'Smith', 'lastName' => 'Cooper'];
  echo json_encode($studentInfo);
  ```
- **Decode**: Parses JSON to arrays:
  ```php
  $data = '{"firstName":"Smith","lastName":"Cooper"}';
  $result = json_decode($data, true);
  ```

## Video Duration Conversion
- **To Seconds**: Converts `HH:MM:SS` or `MM:SS` to seconds:
  ```php
  function convert_duration_to_seconds($duration) {
      // Conversion logic
  }
  ```
- **To Readable Format**: Converts seconds to `Xh Ym Zs`:
  ```php
  function convert_seconds_to_minutes_hours($seconds) {
      // Conversion logic
  }
  ```

## Automatic Logout
Logs out users after a specified inactivity period (e.g., 1 minute).

## Export to CSV
Exports data (e.g., subscribers) to a CSV file:
```php
header('Content-Type: text/csv; charset=utf-8');
header('Content-Disposition: attachment; filename=subscribers_'.date('Y-m-d H:i:s').'.csv');
// CSV export logic
```

## Error Handling
- **404 Page**: Configures custom error pages via `.htaccess`:
  ```apache
  ErrorDocument 404 404.php
  ```
- **Try-Catch**: Handles exceptions gracefully:
  ```php
  try {
      // Code that may throw an exception
  } catch (Exception $e) {
      $error_message = $e->getMessage();
  }
  ```

## Usage
1. Configure the database credentials in the connection script.
2. Set up SMTP credentials for email functionality.
3. Place `.htaccess` in the root directory for URL rewriting.
4. Ensure the `uploads/` directory is writable for file uploads.
5. Test all functionalities (login, registration, CRUD, etc.) in a local or production environment.

## Notes
- Replace placeholder values (e.g., `SMTP_HOST`, `db_name`) with actual values.
- Ensure proper file permissions for uploads and session handling.
- Test email functionality with a valid SMTP service (e.g., Mailtrap for testing).
