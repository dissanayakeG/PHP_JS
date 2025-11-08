# PHP

## Create a local PHP Server

- If you want to run a PHP project or test a PHP file outside of the default web server directories (like `/var/www/html` on `Linux` or `xampp/htdocs` on `Windows`), you can use PHP’s built-in development server.

```bash
php -S localhost:<port> -> php -S localhost:8000 # -S should be uppercase
php -S localhost:8000 -t public
#-t public -> Sets the document root (the folder PHP serves) to the public/ directory.
```

## What is PHP

- php is a scripting language like js, not a programming language
- programming language = compile
- scripting language = interpreted
- static keyword can use for caching the values.

## How PHP works (simple)
1. Browser requests a URL (e.g. `https://example.com/` → server receives `GET /`).
2. Web server (Apache, Nginx, etc.) checks the document root for configured index files (e.g. `index.php`, `index.html`).
3. If an index file is found and PHP is configured, the server hands the .php file to the PHP handler (via mod_php or PHP-FPM).
4. PHP parses and compiles the script into opcodes (cached by OPcache for performance), executes it, and returns the output (HTML, JSON, etc.) to the web server → then to the browser.
5. If no index file exists and directory listing is enabled, the server returns a file listing (a security risk).
6. Use server config (e.g. `Options -Indexes` in Apache or `autoindex off` in Nginx) to prevent directory listings.

<!-- ## PHP Fundamentals -->

## PHP Types

- PHP keywords are case-insensitive -> null,NULL,NuLL

### Type juggling

- When comparing variables of different types, PHP will convert them to the common, comparable type.

```php
<?php
$qty = 20;
if($qty == '20') {
    echo 'Equal'; //Equal
}

$total = 100;
$qty = "20 pieces";
$total = $total + $qty;

echo $total; // 120 //PHP casts the string “20 pieces” as an integer 20 before calculating the sum
```

<!-- ## Operators -->

<!-- ## Control flow -->

## Functions

- PHP 8.0 introduced named parameters.
- Use named arguments to pass arguments to a function based on the parameter names.
- Put the named arguments after the positional arguments in function calls.
- you can ignore passing values for default parameters
- A variadic function accepts a variable number of arguments.
- Do use the ... operator to define a variadic function.
- Only the last parameter can be variadic.

```php
<?php

function sum($a, $b, int ...$numbers): int
{
    return array_sum($numbers);
}
```

<!-- ## Array -->

<!-- ## Sorting Arrays -->

## Advanced Functions

### Anonymous functions

- An anonymous function is a function without a name.
- An anonymous function is a Closure object.
- To access the variables from the parent scope inside an anonymous function, place the variables in the use construct.
- An anonymous function can be assigned to a variable, passed to a function, or returned from a function.

### Arrow functions (php 7.4)

- An arrow function provides a shorter syntax for writing a short anonymous function.
- An arrow function starts with the fn keyword and contains only one expression, the function’s return value.
- An arrow function has access to the variables in its parent scope automatically.

### Variable functions

- Append parentheses () to a variable name to call the function whose name is the same as the variable’s value.
- Use the `$this->$variable()` to call a method of a class.
- Use the className::$variable() to call a static method of a class.

```php
<?php

$f = 'strlen';
echo $f('Hello'); # strlen('Hello) -> 5
```

<!-- ## Variable constructs -->

<!-- ## Advanced Array Operations -->

<!-- ## Organizing PHP files -->

<!-- ## State Management -->

## Processing Forms

- users can input scripts like `<script>alert('Hello');</script>` into input fields.
- Hackers can input malicious code from another server to the user’s web browser, the risk is higher.
- This type of attack is called `cross-site scripting (XSS)` attack.
- Therefore, before displaying user input on a webpage, you should always escape the data. To do that, 
    you use the `htmlspecialchars()` function:
- When you use get request `($_GET)`, it appends input as query data, so use `htmlspecialchars()` to prevent XSS.
- When submiting a form to same file, use htmlspecialchars() to prevent XSS.

```php
# ex 1
<?php
if (isset($_POST['name'], $_POST['email'])) {
	$name = htmlspecialchars($_POST['name']);
	$email = htmlspecialchars($_POST['email']);

	// show the $name and $email
	echo "Thanks $name for your subscription.<br>";
	echo "Please confirm it in your inbox of the email $email.";
} else {
	echo 'You need to provide your name and email address.';
}

# ex 2
<form action="<?php echo htmlspecialchars($_SERVER['PHP_SELF']) ?>" method="post">
```

### Form validation flow

`Validate → Sanitize → Escape`

Validate
- Ensure the input has the correct format, type, and value before using it.
- Use validation filters or regex — e.g. `filter_var($email, FILTER_VALIDATE_EMAIL)`

Sanitize
- Clean the input by removing or altering unwanted or dangerous characters before storing or processing.
- Use sanitization filters — e.g. `filter_var($name, FILTER_SANITIZE_STRING) or FILTER_SANITIZE_EMAIL`

Escape
- Neutralize special characters before displaying or outputting (HTML, JS, SQL, etc.) to prevent injection attacks.
- Use `htmlspecialchars()` when displaying in HTML, and `PDO prepared statements` when inserting into a database.

### isset() vs filter_has_var()

```php
<?php
	$_POST['email'] = 'example@phptutorial.net';
	if(isset($_POST['email'])) // return true

	$_POST['email'] = 'example@phptutorial.net';
	if(filter_has_var(INPUT_POST, 'email')) // return false //check request body


   //Use the filter_has_var() function to check if a variable exists in a specified type,
   //including INPUT_POST, INPUT_GET, INPUT_COOKIE, INPUT_SERVER, or INPUT_ENV.
```

- Use `filter_input()` and `filter_var()` functions to validate and sanitize data.

### filter_var() for sanitize and validate data

```php
//filter_var ( mixed $value , int $filter = FILTER_DEFAULT , array|int $options = 0 ) : mixed

<?php

if (filter_has_var(INPUT_GET, 'id')) {
	// sanitize id
	// FILTER_SANITIZE_NUMBER_INT will remove all characters except the digits, plus, and minus signs from the id variable.
	$clean_id = filter_var($_GET['id'], FILTER_SANITIZE_NUMBER_INT);

	// validate id
	$id = filter_var($clean_id, FILTER_VALIDATE_INT);

	// validate id with options
    $id = filter_var($clean_id, FILTER_VALIDATE_INT, ['options' => ['min_range' => 10]]);

	// show the id if it's valid
	echo $id === false ? 'Invalid id' : $id;
} else {
	echo 'id is required.';
}
```

### filter_input() for sanitize and validate user inputs (HTTP)

- `filter_input ( int $type , string $var_name , int $filter = FILTER_DEFAULT , array|int $options = 0 ) : mixed`
- allows you to get an external variable by its name and filter it using one or more built-in filters.
- The following example uses the filter_input() function to sanitize data for a search form:

```php
<?php

$term_html = filter_input(INPUT_GET, 'term', FILTER_SANITIZE_SPECIAL_CHARS); //value for showing on the search field
$term_url = filter_input(INPUT_GET, 'term', FILTER_SANITIZE_ENCODED); //value for displaying on the page

?>
<form action="search.php" method="get">
    <label for="term"> Search </label>
    <input type="search" name="term" id="term" value="<?php echo $term_html ?>">
    <input type="submit" value="Search">
</form>

<?php

if (null !== $term_html) {
	echo "The search result for <mark> $term_html </mark>.";
}
```

### filter_input vs. filter_var

- `filter_input` Validate and sanitize inputs like `POST request`. -> `INPUT_GET` and `INPUT_POST`
- `filter_var` Validate and sanitize a variable you already have in memory. -> `Any local variable`
- you just can `$input = $_POST['email']`, if you want to use` filter_var`, instead of `filter_input`

```php
$input = $_POST['email'] ?? '';
$validated = filter_var($input, FILTER_VALIDATE_EMAIL);

//OR just
$validated = filter_input(INPUT_POST, 'email', FILTER_VALIDATE_EMAIL);
```

- `filter_input` Filtered value on success, false on failure or not found
- `filter_var` Filtered value on success, false on validation failure
- `filter_input` fails when Variable is not set in the input type or validation fails
- `filter_var` fails when Variable is invalid or fails validation
- `filter_input` can be used for Validate / sanitize form input
- `filter_var` can be used for validate/ sanitize variables
- If a variable doesn’t exist, the `filter_input()` function returns `null` while the `filter_var()` function returns an `empty string` and issues a `notice of an undefined index`.
- Also, the `filter_input()` function doesn’t get the current values of the `$_GET`, `$_POST`, … superglobal variables. Instead, it uses the `original values submitted in the HTTP request`. For example:

### CSRF

- CSRF (Cross-Site Request Forgery) is when another website tricks your browser into sending a legitimate request to a site where you’re already logged in.

- The trick works because your browser automatically includes cookies/session data for any site you’re logged into — even if the request didn’t come directly from you.

#### 1️⃣ You’re logged into your bank

- You log in at `yourbank.com`.
- Your browser now has a session cookie (e.g. `sessionid=abc123`) that keeps you logged in.
- So, any request sent to `yourbank.com` will automatically include that cookie — even if you didn’t type or click anything.

#### 2️⃣ You visit a malicious website

- You go to `malicious-site.com`.
- That site secretly contains some HTML like this:

```html
<form action="https://yourbank.com/transfer-fund" method="POST" id="attackForm">
	<input type="hidden" name="amount" value="10000" />
	<input type="hidden" name="to_account" value="attacker123" />
</form>

<script>
	document.getElementById("attackForm").submit(); // auto-submit when the page loads
</script>
```

#### 3️⃣ What happens next

- As soon as the page loads:
    - That hidden form automatically submits a POST request to `https://yourbank.com/transfer-fund`.
    - Your browser includes your bank’s login cookie (since you’re logged in there).
    - To your bank’s server, it looks like you legitimately submitted the transfer form — even though you didn’t.
    - So the bank would execute the transfer if it doesn’t have protection (CSRF defense).

#### 4️⃣ Why CSRF works

- It abuses the browser’s automatic cookie handling.
- The attacker’s page never sees your cookies, but it can trigger a request that includes them.

#### How websites defend against it

- A secure website adds a CSRF token to every sensitive form.

```php
session_start();
$_SESSION['token'] = bin2hex(random_bytes(35));

// in the form
<input type="hidden" name="token" value="<?= $_SESSION['token'] ?? '' ?>">

//Check the submitted token with the one stored in the $_SESSION to prevent the CSRF attacks.
```

- Then the server checks:
    - If the request includes this token.
    - If it matches the token stored in your session.
- The malicious site can’t know or guess that token, so the forged request fails.

### PRG (Post-Redirect-Get)

When a user submits a form (like a registration form or payment form):
- The form sends data with a POST request.
- The server processes it (e.g., inserts into DB, sends an email).
- The server then displays a “Success” page.
- Now, if the user refreshes the page, the browser says: “Resubmit the form data?”
- If they click Yes, the same POST request runs again — meaning the same data might get inserted twice, or payment might process again.
- That’s the double-submit problem.

The PRG Solution
- `POST`:The user submits the form → server processes the data.
- `REDIRECT`:Instead of showing a success message directly, the server sends a redirect response (HTTP 302 or 303) to a GET URL.
- `GET`:The browser then automatically loads that new URL — a normal GET request — where you display the success message.
- `Result`: Refreshing the page now only reloads the GET page, not the original POST submission.

```php
<?php
session_start();

if ($_SERVER['REQUEST_METHOD'] === 'POST') {
    // Process form (e.g., save to DB)
    $_SESSION['message'] = 'Form submitted successfully!';

    // Redirect to the same page using GET
    header('Location: ' . $_SERVER['PHP_SELF']);
    exit;
}
?>

<!DOCTYPE html>
<html lang="en">
<head><meta charset="UTF-8"><title>PRG Example</title></head>
<body>
  <?php
  if (!empty($_SESSION['message'])) {
      echo '<p>' . htmlspecialchars($_SESSION['message']) . '</p>';
      unset($_SESSION['message']); // clear after showing
  }
  ?>

  <form method="post">
      <input type="text" name="name" placeholder="Enter name" required>
      <button type="submit">Submit</button>
  </form>
</body>
</html>
```

### File Upload

- Use the input with `type="file"` to create a file input element and include the `enctype="multipart/form-data"` attribute in the form to allow the file upload.
- Access the uploaded file information via the `$_FILES` array.
- Never trust the information on the `$_FILES` except the `tmp_name`.
- Always validate the information on the `$_FILES`.
- Use the `move_uploaded_file()` function to move the file from the temporary directory to another folder.
- File input element must have the multiple attribute and its name must have the square brackets `([])` like this

```php
<input type="file" name="files[]" id="files" multiple />
```

### password_hash and password_verify

- Use the PHP `password_hash()` function to create a hash password using a secure one-way hashing algorithm.
- Use the PHP `password_verify()` function to check if a password matches a hashed password created by the `password_hash()` function.

```php
password_hash(
     string $password,
     string|int|null $algo,
     array $options = []
): string

//PASSWORD_DEFAULT -> bcrypt
//PASSWORD_BCRYPT -> CRYPT_BLOWFISH
//PASSWORD_ARGON2I -> Argon2i
//PASSWORD_ARGON2ID -> Argon2id

<?php

$password = 'Password1';
echo password_hash($password, PASSWORD_DEFAULT);

password_verify(string $password, string $hash): bool
// $password is a plain text password to match.
// $hash is a hash created by the password_hash() function.
```

<!-- ## Login System -->

<!-- ## Working with Files -->

<!-- ## Working with Directories -->

<!-- ## String operations -->

<!-- ## Regular Expressions -->

<!-- ## PHP Date & Time -->

# OOP with PHP

## Objects & Classes

- The `$this` variable references the current object of the class.
- Do use the method chaining by returning `$this` from a method to make the code more concise.

## Constructor and Destructor

### constructor promotion (PHP 8.0)

- When a constructor parameter includes an access modifier (public, private, or protected) PHP will treat it as both a constructor’s argument and an object’s property. And it assigns the constructor argument to the property.
- PHP automatically invokes the destructor when the object is deleted or the script is terminated.

## Properties

- Typed properties were introduced in PHP 7.4.
- Untyped properties default to null.
- Typed properties start uninitialized until you explicitly assign a value.
- You can’t access a typed property before it’s initialized — doing so causes a Fatal Error.
- To allow null and avoid errors, you can declare the property as nullable and initialize it:

```php
public ?string $name = null;
```

## Inheritance

- The constructor of the child class doesn’t automatically call the constructor of its parent class.
- Use `parent::__construct(arguments)` to call the parent constructor from the constructor in the child class.
- Use `parent::` to call the overridden method in the overriding method. You can't use `$this->`, it will recursively call the same method.
- Use the `final` method when you don’t want a child class’s method to override a parent class’s method.

## Abstract classes

- Abstract classes cannot be instantiated. Typically, an abstract defines an interface for other classes to extend.
- Similar to an abstract class, an abstract method is a method that does not have an implementation.
- If a class contains one or more abstract methods, it must be an abstract class.
- A class that extends an abstract class needs to implement the abstract methods of the abstract class.
- Note that all the methods in the interface must be public.

## Abstract classes

- An interface can also include constants.
- A class can inherit from one class only. However, it can implement multiple interfaces.
- When a class implements an interface, it’s called a `concrete class`.

## Polymorphism

- To implement polymorphism in PHP, you can use either abstract classes or interfaces.
- Polymorphism helps you create a generic framework that takes the different object types that share the same interface.
- By using polymorphism, you can reduce coupling and increase code reusability.

## Traits

- Inheritance makes the code very tightly coupled -> introduced `traits` (php 5.4).
- Inheritance allows classes to reuse the code vertically while the traits allow classes reuse the code horizontally.
- PHP allows you to compose multiple traits into a trait by using the use statement in the trait’s declaration.

### PHP trait’s method conflict resolution

#### Overriding trait method

- When a class uses multiple traits that share the same method name, PHP will raise a fatal error.
- use `inteadof` keyword

```php
trait FileLogger
{
	public function log($msg)
	{
		echo 'File Logger ' . date('Y-m-d h:i:s') . ':' . $msg . '<br/>';
	}
}

trait DatabaseLogger
{
	public function log($msg)
	{
		echo 'Database Logger ' . date('Y-m-d h:i:s') . ':' . $msg . '<br/>';
	}
}

class Logger
{
	use FileLogger, DatabaseLogger{
		FileLogger::log insteadof DatabaseLogger;
	}
}

$logger = new Logger();
$logger->log('this is a test message #1');
$logger->log('this is a test message #2');
```

#### Aliasing trait method

- if you want to use both log() methods from the FileLogger and DatabaseLogger traits, you can use an alias for the method of the trait within the class that uses the trait.

```php
class Logger
{
	use FileLogger, DatabaseLogger{
		DatabaseLogger::log as logToDatabase;
		FileLogger::log insteadof DatabaseLogger;
	}
}

$logger = new Logger();
$logger->log('this is a test message #1');
$logger->logToDatabase('this is a test message #2');
```

## Working with Objects

# Serialization in PHP

Serialization is the process of converting a PHP value (object, array, etc.) into a storable string representation that can be saved to a file, database, or transmitted over a network. Unserialization (or deserialization) is the reverse process—reconstructing the original PHP value from that string.

## Why Use Serialization?

- Persistence: Store objects in sessions, databases, or files
- Caching: Save expensive-to-create objects for later reuse
- Data transfer: Send complex data structures between systems
- Message queues: Pass objects between different parts of an application

### serialize

- The `serialize()` function converts an object into a storable string representation (a byte stream).
- It only serializes object properties, not methods.
- If the class defines a `__sleep()` method, PHP will automatically call it before serialization.
    - `__sleep()` must return an array of property names that should be serialized.
    - This method is mainly used to clean up or prepare data before serialization.
- If the class defines a `__serialize()` method, PHP will call it instead.
    - `__serialize()` must return an associative array of property names and values.
    - This method was introduced in PHP 7.4 to replace `__sleep()`.
- If both `__sleep()` and `__serialize()` are defined, `__sleep()` will be ignored.

### unserialize

- Use the `unserialize()` method to convert a serialized string into an object.
- The `unserialize()` method calls the `__unserialize()` or `__wakeup()` method of the object to perform re-initialization tasks.
- The `unserialize()` method calls the `__unserialize()` method only if an object has both `__unserialize()` and `__wakeup()` methods.
- The `unserialize()` function creates a completely new object that does not reference the original object.

### Complete Example: Session Cache

```php
class UserSession {
    public $userId;
    public $username;
    public $loginTime;
    private $cache; // Large temporary data
    private $apiClient; // Non-serializable resource
    
    public function __construct($userId, $username) {
        $this->userId = $userId;
        $this->username = $username;
        $this->loginTime = time();
        $this->cache = []; // Loaded separately
        $this->apiClient = new ApiClient(); // Resource
    }
    
    public function __sleep() {
        // Save session to disk
        // Exclude cache (too large) and apiClient (resource)        
        // Could perform cleanup here
        $this->cache = null;        
        return ['userId', 'username', 'loginTime'];
    }
    
    public function __wakeup() {
        // Restore session from disk        
        // Reinitialize resources
        $this->cache = [];
        $this->apiClient = new ApiClient();
        
        // Validate session isn't expired
        if (time() - $this->loginTime > 3600) {
            throw new Exception('Session expired');
        }
    }
}

// Usage
$session = new UserSession(123, 'alice');
$_SESSION['user'] = serialize($session); // __sleep() called

// Later request
$session = unserialize($_SESSION['user']); // __wakeup() called

# after PHP 7.4
class ModernUser {
    private $data;
    
    public function __serialize(): array {
        // Return array of data to serialize
        return [
            'data' => $this->data,
            'timestamp' => time()
        ];
    }
    
    public function __unserialize(array $data): void {
        // Receive the serialized data
        $this->data = $data['data'];
        // Can use the timestamp for validation
    }
}
```

### Clone Object

- When you copy an object using the assignment operator (=), both variables refer to the same object in memory.
- Any changes made to one will affect the other.

#### Shallow Copy

- we can use `clone` keyword to create a `shallow copy of an object`.

```php
$bob = new Person('Bob');

// clone an object
$alex = clone $bob;
$alex->name = 'Alex';

// show both objects
var_dump($bob);
var_dump($alex);

# output
object(Person)#1 (1) {
  ["name"]=> string(3) "Bob"
}
object(Person)#2 (1) {
  ["name"]=> string(4) "Alex"
}
```

- A shallow copy means PHP copies all properties of the object.
- However, if a property references another object, the reference is copied — both cloned and original objects will still point to the same referenced object.

### Deep copy with __clone method

- A deep copy duplicates not just the object itself, but also any referenced objects within it.
- PHP automatically calls the `__clone()` method after cloning an object.
- We can implement `__clone()` to manually clone the referenced objects, ensuring full independence between the original and the clone.

### Deep copy using serialize and unserialize functions

```php
<?php

function deep_clone($object)
{
	return unserialize(serialize($object));
}
```

### Compare Objects

- The comparison operator `(==)` returns true if two objects are the same or different instances of a class with the same properties’ values.
- The identity operator `(===)` returns true if two objects reference the exact same object instance in memory.

```php
$p1 = new Point(10, 20);
$p2 = new Point(10, 20);

var_dump($p1 == $p2);  // true  → same class, same property values
var_dump($p1 === $p2); // false → different instances

$p3 = $p1;

var_dump($p1 === $p3); // true  → same reference (identical instance)
```

### Anonymous Class

- An anonymous class is a class without a declared name.
- Inside the parentheses, you can define constructor, destructor, properties, and methods for the anonymous class like a regular class.
- An anonymous can implement one or multiple interfaces.
- Like a regular class, a class can inherit from one named class.
- To pass an argument to the constructor, we place it in parentheses that follow the class keyword.

```php
<?php

$myObject = new class {}; //syntax

# Example
$logger = new class(true) extends SimpleLogger { //pass an argument, and extend from named abstract class
    public function __construct(bool $newLine)
    {
        parent::__construct($newLine);
    }
};

$logger->log('Hello');
$logger->log('Bye');
```

## Namespaces and Autoloading

- We can give a namespace to a class and import it for use.
- We can use aliases to prevent name collision.
- We can use `(\)` to use global classes such as built-in classes or user-defined classes without a namespace.

```php

<?php

namespace Store\Model;

class Customer
{
}

#index.php
<?php

require 'src/Model/Customer.php';
use Store\Model\Customer;
$customer = new Customer('Bob');
```

- We can use `spl_autoload_register` function to autoload the classes, interfaces, and traits.
- The `spl_autoload_register()` function allows you to use multiple autoloading functions.

```php
# functions.php
<?php

function load_model($class_name)
{
	$path_to_file = 'models/' . $class_name . '.php';

	if (file_exists($path_to_file)) {
		require $path_to_file;
	}
}
spl_autoload_register('load_model');

# index.php
<?php

require 'functions.php';

$contact = new Contact('john.doe@example.com');
```

#### Composer Autoload

- There are two ways to use composer for auto loading
1. classmap
2. PSR-4

```php
# composer.json
{
	"autoload": {
		"classmap": ["app/models", "app/services"]
    }
}

# run composer dump-autoload and it will generate /vendor/autoload.php
# You just can import /vendor/autoload.php in the index.php 

# index.php
<?php

require_once __DIR__ . '/../vendor/autoload.php';
$user = new User('admin', '$ecurePa$$w0rd1');
```

- PSR-4 is auto-loading standard

```php
# composer.json
{
    "autoload": {
        "psr-4": {
            "Acme\\":"app/Acme"
        }
    }
}
# run the composer dump-autoload command to generate the autoload.php file
# index.php
<?php

require_once __DIR__ . '/../vendor/autoload.php';

use Acme\Auth\User as User;
$user = new User('admin', '$ecurePa$$w0rd1');
```
