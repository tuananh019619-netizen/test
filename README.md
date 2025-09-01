# PHP Web Application

This README provides an overview of the PHP web application code snippets provided, covering database connections, user authentication, CRUD operations, file handling, email functionality, payment integrations, chained select functionality, security measures, and more.

## Table of Contents
1. [Project Setup](#project-setup)
2. [Database Connection](#database-connection)
3. [User Authentication](#user-authentication)
4. [CRUD Operations](#crud-operations)
5. [File Handling](#file-handling)
6. [Email Sending](#email-sending)
7. [Payment Integrations](#payment-integrations)
8. [Chained Select](#chained-select)
9. [Security Measures](#security-measures)
10. [Pagination](#pagination)
11. [URL Rewriting](#url-rewriting)
12. [Date and Time Utilities](#date-and-time-utilities)
13. [JSON Handling](#json-handling)
14. [Video Duration Conversion](#video-duration-conversion)
15. [Automatic Logout](#automatic-logout)
16. [Export to CSV](#export-to-csv)
17. [Error Handling](#error-handling)

## Project Setup
- **Base URL Configuration**: Define `BASE_URL` and `ADMIN_URL` for consistent URL management:
  ```php
  define("BASE_URL", "http://localhost/project/");
  define("ADMIN_URL", BASE_URL."admin/");
  ```
- **Dependencies**: Install PHPMailer, Omnipay (PayPal), and Stripe PHP SDK using Composer:
  ```bash
  composer require phpmailer/phpmailer
  composer require league/omnipay omnipay/paypal
  composer require stripe/stripe-php
  ```
- **Directory Structure**: Ensure an `uploads/` directory for file uploads and `vendor/` for Composer dependencies.
- **External Libraries**: Include jQuery for AJAX functionality (used in chained select):
  ```html
  <script src="https://code.jquery.com/jquery-3.6.0.min.js"></script>
  ```

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

## Payment Integrations
### PayPal Integration
- **PayPal Information**:
  - Developer Account: `developer@cwa.com`
  - Personal Account: `sb-vhfyz28322584@personal.example.com`
  - Business Account: `sb-6l43bc27296072@business.example.com`
  - Client ID: `AUepW_R8YYWL7R9nASWIkYSvoLg_3KzYFeb-tt0KMWuWOBwX_JmYlMGKMWbsg_bhPIB2CoNNy5AGk1dm`
  - Secret: `EFuwGqxMAPpSMCoxkmo6-WWnt02EjZFNtdN39Z9Ay-rmruF2gR2MmCPdQn1Rk1fH5z93yd96fB5hqP6s`
- **GitHub**: [Omnipay PayPal](https://github.com/thephpleague/omnipay-paypal)
- **Install**: 
  ```bash
  composer require league/omnipay omnipay/paypal
  ```
- **Configuration** (`config.php`):
  ```php
  require_once "vendor/autoload.php";
  use Omnipay\Omnipay;
  define('CLIENT_ID', 'your_client_id');
  define('CLIENT_SECRET', 'your_client_secret');
  define('PAYPAL_RETURN_URL', 'http://localhost/payment/success.php');
  define('PAYPAL_CANCEL_URL', 'http://localhost/payment/cancel.php');
  define('PAYPAL_CURRENCY', 'USD');
  $gateway = Omnipay::create('PayPal_Rest');
  $gateway->setClientId(CLIENT_ID);
  $gateway->setSecret(CLIENT_SECRET);
  $gateway->setTestMode(true);
  ```
- **Form Submit** (`index.php`): Initiates payment and redirects to PayPal.
- **Success Page** (`success.php`): Processes successful transactions and stores details.
- **Cancel Page** (`cancel.php`): Handles canceled payments.

### Stripe Integration
- **GitHub**: [Stripe PHP](https://github.com/stripe/stripe-php)
- **Install**:
  ```bash
  composer require stripe/stripe-php
  ```
- **Configuration**:
  ```php
  define('STRIPE_TEST_PK', 'publishable_key');
  define('STRIPE_TEST_SK', 'secret_key');
  define('STRIPE_SUCCESS_URL', 'http://localhost/payment/success.php');
  define('STRIPE_CANCEL_URL', 'http://localhost/payment/cancel.php');
  ```
- **Form Submit** (`index.php`): Creates a Stripe Checkout session and redirects.
- **Success Page** (`success.php`): Verifies payment and displays confirmation.
- **Cancel Page** (`cancel.php`): Handles canceled payments.

## Chained Select
Implements dynamic dropdowns using AJAX to populate schedules based on selected days:
- **Frontend** (`ajax.php`):
  ```php
  <script src="https://code.jquery.com/jquery-3.6.0.min.js"></script>
  <select id="schedule_day" name="schedule_day_id">
      <option value="">-- Select a Day --</option>
      <!-- Populated from schedule_days table -->
  </select>
  <select id="schedule" name="schedule_id">
      <option value="">-- Select a Schedule --</option>
  </select>
  <script>
      $(document).ready(function() {
          $('#schedule_day').on('change', function() {
              var scheduleDayId = $(this).val();
              $('#schedule').html('<option value="">-- Select a Schedule --</option>');
              if (scheduleDayId) {
                  $.ajax({
                      url: 'ajax1.php',
                      type: 'POST',
                      data: { schedule_day_id: scheduleDayId },
                      dataType: 'json',
                      success: function(response) {
                          if (response.length > 0) {
                              $.each(response, function(index, schedule) {
                                  $('#schedule').append(
                                      `<option value='${schedule.id}'>${schedule.name} - ${schedule.time}</option>`
                                  );
                              });
                          } else {
                              $('#schedule').append('<option value="">No schedules found</option>');
                          }
                      }
                  });
              }
          });
      });
  </script>
  ```
- **Backend** (`ajax1.php`):
  ```php
  $scheduleDayId = $_POST['schedule_day_id'];
  $statement = $pdo->prepare("SELECT * FROM schedules WHERE schedule_day_id=? ORDER BY item_order");
  $statement->execute([$scheduleDayId]);
  $schedules = $statement->fetchAll(PDO::FETCH_ASSOC);
  header('Content-Type: application/json');
  echo json_encode($schedules);
  ```
- **Database**: Requires `schedule_days` (columns: `id`, `day`, `order1`) and `schedules` (columns: `id`, `schedule_day_id`, `name`, `time`, `item_order`) tables.

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
3. Configure PayPal and Stripe credentials in `config.php`.
4. Place `.htaccess` in the root directory for URL rewriting.
5. Ensure the `uploads/` directory is writable for file uploads.
6. Set up `schedule_days` and `schedules` tables for chained select functionality.
7. Test all functionalities (login, registration, CRUD, payments, chained select, etc.) in a local or production environment.

## Notes
- Replace placeholder values (e.g., `SMTP_HOST`, `db_name`, `your_client_id`, `publishable_key`) with actual values.
- Ensure proper file permissions for uploads and session handling.
- Test email and payment functionalities with valid services (e.g., Mailtrap for email, PayPal Sandbox, Stripe Test Mode).
- Switch PayPal `setTestMode` to `false` for production.
- Ensure jQuery is accessible for chained select functionality.
