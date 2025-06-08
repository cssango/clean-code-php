# Clean Code PHP 8.4

## Table of Contents

  1. [Introduction](#introduction)
  2. [Variables](#variables)
     * [Use meaningful and pronounceable variable names](#use-meaningful-and-pronounceable-variable-names)
     * [Use the same vocabulary for the same type of variable](#use-the-same-vocabulary-for-the-same-type-of-variable)
     * [Use searchable names (part 1)](#use-searchable-names-part-1)
     * [Use searchable names (part 2)](#use-searchable-names-part-2)
     * [Use explanatory variables](#use-explanatory-variables)
     * [Avoid nesting too deeply and return early (part 1)](#avoid-nesting-too-deeply-and-return-early-part-1)
     * [Avoid nesting too deeply and return early (part 2)](#avoid-nesting-too-deeply-and-return-early-part-2)
     * [Avoid Mental Mapping](#avoid-mental-mapping)
     * [Don't add unneeded context](#dont-add-unneeded-context)
  3. [Comparison](#comparison)
     * [Use identical comparison](#use-identical-comparison)
     * [Null coalescing operator](#null-coalescing-operator)
     * [Null coalescing assignment operator](#null-coalescing-assignment-operator)
     * [Match expressions](#match-expressions)
  4. [Functions](#functions)
     * [Use default arguments instead of short circuiting or conditionals](#use-default-arguments-instead-of-short-circuiting-or-conditionals)
     * [Function arguments (2 or fewer ideally)](#function-arguments-2-or-fewer-ideally)
     * [Function names should say what they do](#function-names-should-say-what-they-do)
     * [Functions should only be one level of abstraction](#functions-should-only-be-one-level-of-abstraction)
     * [Don't use flags as function parameters](#dont-use-flags-as-function-parameters)
     * [Avoid Side Effects](#avoid-side-effects)
     * [Don't write to global functions](#dont-write-to-global-functions)
     * [Don't use a Singleton pattern](#dont-use-a-singleton-pattern)
     * [Encapsulate conditionals](#encapsulate-conditionals)
     * [Avoid negative conditionals](#avoid-negative-conditionals)
     * [Avoid conditionals](#avoid-conditionals)
     * [Avoid type-checking (part 1)](#avoid-type-checking-part-1)
     * [Avoid type-checking (part 2)](#avoid-type-checking-part-2)
     * [Remove dead code](#remove-dead-code)
  5. [Objects and Data Structures](#objects-and-data-structures)
     * [Use object encapsulation](#use-object-encapsulation)
     * [Make objects have private/protected members](#make-objects-have-privateprotected-members)
     * [Use readonly properties](#use-readonly-properties)
  6. [Classes](#classes)
     * [Prefer composition over inheritance](#prefer-composition-over-inheritance)
     * [Avoid fluent interfaces](#avoid-fluent-interfaces)
     * [Prefer final classes](#prefer-final-classes)
     * [Use property promotion](#use-property-promotion)
     * [Use enums for constants](#use-enums-for-constants)
  7. [SOLID](#solid)
     * [Single Responsibility Principle (SRP)](#single-responsibility-principle-srp)
     * [Open/Closed Principle (OCP)](#openclosed-principle-ocp)
     * [Liskov Substitution Principle (LSP)](#liskov-substitution-principle-lsp)
     * [Interface Segregation Principle (ISP)](#interface-segregation-principle-isp)
     * [Dependency Inversion Principle (DIP)](#dependency-inversion-principle-dip)
  8. [Don't repeat yourself (DRY)](#dont-repeat-yourself-dry)
  9. [Modern PHP Features](#modern-php-features)
     * [Union Types](#union-types)
     * [Named Arguments](#named-arguments)
     * [Attributes](#attributes)
     * [First-class Callable Syntax](#first-class-callable-syntax)

## Introduction

Software engineering principles, from Robert C. Martin's book
[*Clean Code*](https://www.amazon.com/Clean-Code-Handbook-Software-Craftsmanship/dp/0132350882),
adapted for PHP 8.4. This is not a style guide. It's a guide to producing
readable, reusable, and refactorable software in PHP.

Not every principle herein has to be strictly followed, and even fewer will be universally
agreed upon. These are guidelines and nothing more, but they are ones codified over many
years of collective experience by the authors of *Clean Code*.

Inspired from [clean-code-javascript](https://github.com/ryanmcdermott/clean-code-javascript).

This guide focuses on PHP 8.4+ features and modern syntax to write cleaner, more maintainable code.

## Variables

### Use meaningful and pronounceable variable names

**Bad:**

```php
$ymdstr = $moment->format('y-m-d');
```

**Good:**

```php
$currentDate = $moment->format('y-m-d');
```

**[⬆ back to top](#table-of-contents)**

### Use the same vocabulary for the same type of variable

**Bad:**

```php
function getUserInfo(): array { /* ... */ }
function getUserData(): array { /* ... */ }
function getUserRecord(): array { /* ... */ }
function getUserProfile(): array { /* ... */ }
```

**Good:**

```php
function getUser(): User
{
    // Return a proper User object instead of array
}
```

**[⬆ back to top](#table-of-contents)**

### Use searchable names (part 1)

We will read more code than we will ever write. It's important that the code we do write is
readable and searchable. By *not* naming variables that end up being meaningful for
understanding our program, we hurt our readers.
Make your names searchable.

**Bad:**

```php
// What the heck is 448 for?
$result = $serializer->serialize($data, 448);
```

**Good:**

```php
$result = $serializer->serialize(
    $data, 
    JSON_UNESCAPED_SLASHES | JSON_PRETTY_PRINT | JSON_UNESCAPED_UNICODE
);
```

### Use searchable names (part 2)

**Bad:**

```php
class User
{
    // What the heck is 7 for?
    public int $access = 7;
}

// What the heck is 4 for?
if ($user->access & 4) {
    // ...
}

// What's going on here?
$user->access ^= 2;
```

**Good:**

```php
enum UserPermission: int
{
    case READ = 1;
    case CREATE = 2;
    case UPDATE = 4;
    case DELETE = 8;
}

class User
{
    // User as default can read, create and update something
    public int $access = UserPermission::READ->value | 
                        UserPermission::CREATE->value | 
                        UserPermission::UPDATE->value;
}

if ($user->access & UserPermission::UPDATE->value) {
    // do edit ...
}

// Deny access rights to create something
$user->access ^= UserPermission::CREATE->value;
```

**[⬆ back to top](#table-of-contents)**

### Use explanatory variables

**Bad:**

```php
$address = 'One Infinite Loop, Cupertino 95014';
$cityZipCodeRegex = '/^[^,]+,\s*(.+?)\s*(\d{5})$/';
preg_match($cityZipCodeRegex, $address, $matches);

saveCityZipCode($matches[1], $matches[2]);
```

**Not bad:**

It's better, but we are still heavily dependent on regex.

```php
$address = 'One Infinite Loop, Cupertino 95014';
$cityZipCodeRegex = '/^[^,]+,\s*(.+?)\s*(\d{5})$/';
preg_match($cityZipCodeRegex, $address, $matches);

[, $city, $zipCode] = $matches;
saveCityZipCode($city, $zipCode);
```

**Good:**

Decrease dependence on regex by naming subpatterns.

```php
$address = 'One Infinite Loop, Cupertino 95014';
$cityZipCodeRegex = '/^[^,]+,\s*(?<city>.+?)\s*(?<zipCode>\d{5})$/';
preg_match($cityZipCodeRegex, $address, $matches);

saveCityZipCode($matches['city'], $matches['zipCode']);
```

**Best (PHP 8.4):**

Use structured approach with value objects.

```php
readonly class Address
{
    public function __construct(
        public string $street,
        public string $city,
        public string $zipCode
    ) {}
    
    public static function fromString(string $addressString): self
    {
        $regex = '/^(?<street>[^,]+),\s*(?<city>.+?)\s*(?<zipCode>\d{5})$/';
        preg_match($regex, $addressString, $matches);
        
        return new self($matches['street'], $matches['city'], $matches['zipCode']);
    }
}

$address = Address::fromString('One Infinite Loop, Cupertino 95014');
saveCityZipCode($address->city, $address->zipCode);
```

**[⬆ back to top](#table-of-contents)**

### Avoid nesting too deeply and return early (part 1)

Too many if-else statements can make your code hard to follow. Explicit is better
than implicit.

**Bad:**

```php
function isShopOpen($day): bool
{
    if ($day) {
        if (is_string($day)) {
            $day = strtolower($day);
            if ($day === 'friday') {
                return true;
            } elseif ($day === 'saturday') {
                return true;
            } elseif ($day === 'sunday') {
                return true;
            }
            return false;
        }
        return false;
    }
    return false;
}
```

**Good:**

```php
function isShopOpen(?string $day): bool
{
    if (empty($day)) {
        return false;
    }

    $openingDays = ['friday', 'saturday', 'sunday'];

    return in_array(strtolower($day), $openingDays, true);
}
```

**Best (PHP 8.4):**

```php
enum WeekDay: string
{
    case MONDAY = 'monday';
    case TUESDAY = 'tuesday';
    case WEDNESDAY = 'wednesday';
    case THURSDAY = 'thursday';
    case FRIDAY = 'friday';
    case SATURDAY = 'saturday';
    case SUNDAY = 'sunday';
}

function isShopOpen(?WeekDay $day): bool
{
    return match($day) {
        WeekDay::FRIDAY, WeekDay::SATURDAY, WeekDay::SUNDAY => true,
        default => false,
    };
}
```

**[⬆ back to top](#table-of-contents)**

### Avoid nesting too deeply and return early (part 2)

**Bad:**

```php
function fibonacci(int $n)
{
    if ($n < 50) {
        if ($n !== 0) {
            if ($n !== 1) {
                return fibonacci($n - 1) + fibonacci($n - 2);
            }
            return 1;
        }
        return 0;
    }
    return 'Not supported';
}
```

**Good:**

```php
function fibonacci(int $n): int
{
    return match(true) {
        $n === 0 || $n === 1 => $n,
        $n >= 50 => throw new InvalidArgumentException('Not supported'),
        default => fibonacci($n - 1) + fibonacci($n - 2),
    };
}
```

**[⬆ back to top](#table-of-contents)**

### Avoid Mental Mapping

Don't force the reader of your code to translate what the variable means.
Explicit is better than implicit.

**Bad:**

```php
$l = ['Austin', 'New York', 'San Francisco'];

for ($i = 0; $i < count($l); $i++) {
    $li = $l[$i];
    doStuff();
    doSomeOtherStuff();
    // ...
    // ...
    // ...
    // Wait, what is `$li` for again?
    dispatch($li);
}
```

**Good:**

```php
$locations = ['Austin', 'New York', 'San Francisco'];

foreach ($locations as $location) {
    doStuff();
    doSomeOtherStuff();
    // ...
    // ...
    // ...
    dispatch($location);
}
```

**[⬆ back to top](#table-of-contents)**

### Don't add unneeded context

If your class/object name tells you something, don't repeat that in your
variable name.

**Bad:**

```php
class Car
{
    public string $carMake;
    public string $carModel;
    public string $carColour;
}
```

**Good:**

```php
readonly class Car
{
    public function __construct(
        public string $make,
        public string $model,
        public string $colour
    ) {}
}
```

**[⬆ back to top](#table-of-contents)**

## Comparison

### Use [identical comparison](http://php.net/manual/en/language.operators.comparison.php)

**Not good:**

The simple comparison will convert the string into an integer.

```php
$a = '42';
$b = 42;

if ($a != $b) {
    // The expression will always pass
}
```

The comparison `$a != $b` returns `FALSE` but in fact it's `TRUE`!
The string `42` is different than the integer `42`.

**Good:**

The identical comparison will compare type and value.

```php
$a = '42';
$b = 42;

if ($a !== $b) {
    // The expression is verified
}
```

The comparison `$a !== $b` returns `TRUE`.

**[⬆ back to top](#table-of-contents)**

### Null coalescing operator

Null coalescing is an operator introduced in PHP 7. The null coalescing operator `??` returns its first operand if it exists and is not `null`; otherwise it returns its second operand.

**Bad:**

```php
if (isset($_GET['name'])) {
    $name = $_GET['name'];
} elseif (isset($_POST['name'])) {
    $name = $_POST['name'];
} else {
    $name = 'nobody';
}
```

**Good:**
```php
$name = $_GET['name'] ?? $_POST['name'] ?? 'nobody';
```

**[⬆ back to top](#table-of-contents)**

### Null coalescing assignment operator

PHP 7.4 introduced the null coalescing assignment operator `??=`.

**Bad:**

```php
$config['host'] = $config['host'] ?? 'localhost';
$config['port'] = $config['port'] ?? 3306;
```

**Good:**

```php
$config['host'] ??= 'localhost';
$config['port'] ??= 3306;
```

**[⬆ back to top](#table-of-contents)**

### Match expressions

The `match` expression (PHP 8+) is more powerful than `switch` statements.

**Bad:**

```php
function getHttpStatusMessage(int $code): string
{
    switch ($code) {
        case 200:
            return 'OK';
        case 404:
            return 'Not Found';
        case 500:
            return 'Internal Server Error';
        default:
            return 'Unknown';
    }
}
```

**Good:**

```php
function getHttpStatusMessage(int $code): string
{
    return match($code) {
        200 => 'OK',
        404 => 'Not Found',
        500 => 'Internal Server Error',
        default => 'Unknown',
    };
}
```

**[⬆ back to top](#table-of-contents)**

## Functions

### Use default arguments instead of short circuiting or conditionals

**Not good:**

This is not good because `$breweryName` can be `NULL`.

```php
function createMicrobrewery($breweryName = 'Hipster Brew Co.'): void
{
    // ...
}
```

**Good:**

You can use [type hinting](https://www.php.net/manual/en/language.types.declarations.php) and be sure that the `$breweryName` will not be `NULL`.

```php
function createMicrobrewery(string $breweryName = 'Hipster Brew Co.'): void
{
    // ...
}
```

**[⬆ back to top](#table-of-contents)**

### Function arguments (2 or fewer ideally)

Limiting the amount of function parameters is incredibly important because it makes
testing your function easier. Having more than three leads to a combinatorial explosion
where you have to test tons of different cases with each separate argument.

Zero arguments is the ideal case. One or two arguments is ok, and three should be avoided.
Anything more than that should be consolidated. Usually, if you have more than two
arguments then your function is trying to do too much. In cases where it's not, most
of the time a higher-level object will suffice as an argument.

**Bad:**

```php
class Questionnaire
{
    public function __construct(
        string $firstname,
        string $lastname,
        string $patronymic,
        string $region,
        string $district,
        string $city,
        string $phone,
        string $email
    ) {
        // ...
    }
}
```

**Good:**

```php
readonly class Name
{
    public function __construct(
        public string $firstname,
        public string $lastname,
        public string $patronymic
    ) {}
}

readonly class City
{
    public function __construct(
        public string $region,
        public string $district,
        public string $city
    ) {}
}

readonly class Contact
{
    public function __construct(
        public string $phone,
        public string $email
    ) {}
}

readonly class Questionnaire
{
    public function __construct(
        public Name $name,
        public City $city,
        public Contact $contact
    ) {}
}
```

**[⬆ back to top](#table-of-contents)**

### Function names should say what they do

**Bad:**

```php
class Email
{
    public function handle(): void
    {
        mail($this->to, $this->subject, $this->body);
    }
}

$message = new Email(/* ... */);
// What is this? A handle for the message? Are we writing to a file now?
$message->handle();
```

**Good:**

```php
class Email
{
    public function send(): void
    {
        mail($this->to, $this->subject, $this->body);
    }
}

$message = new Email(/* ... */);
// Clear and obvious
$message->send();
```

**[⬆ back to top](#table-of-contents)**

### Functions should only be one level of abstraction

When you have more than one level of abstraction your function is usually
doing too much. Splitting up functions leads to reusability and easier
testing.

**Bad:**

```php
function parseBetterPHPAlternative(string $code): void
{
    $regexes = [
        // ...
    ];

    $statements = explode(' ', $code);
    $tokens = [];
    foreach ($regexes as $regex) {
        foreach ($statements as $statement) {
            // ...
        }
    }

    $ast = [];
    foreach ($tokens as $token) {
        // lex...
    }

    foreach ($ast as $node) {
        // parse...
    }
}
```

**Good:**

```php
readonly class Tokenizer
{
    public function tokenize(string $code): array
    {
        $regexes = [
            // ...
        ];

        $statements = explode(' ', $code);
        $tokens = [];
        foreach ($regexes as $regex) {
            foreach ($statements as $statement) {
                $tokens[] = /* ... */;
            }
        }

        return $tokens;
    }
}

readonly class Lexer
{
    public function lexify(array $tokens): array
    {
        $ast = [];
        foreach ($tokens as $token) {
            $ast[] = /* ... */;
        }

        return $ast;
    }
}

readonly class BetterPHPAlternative
{
    public function __construct(
        private Tokenizer $tokenizer,
        private Lexer $lexer
    ) {}

    public function parse(string $code): void
    {
        $tokens = $this->tokenizer->tokenize($code);
        $ast = $this->lexer->lexify($tokens);
        foreach ($ast as $node) {
            // parse...
        }
    }
}
```

**[⬆ back to top](#table-of-contents)**

### Don't use flags as function parameters

Flags tell your user that this function does more than one thing. Functions should
do one thing. Split out your functions if they are following different code paths
based on a boolean.

**Bad:**

```php
function createFile(string $name, bool $temp = false): void
{
    if ($temp) {
        touch('./temp/' . $name);
    } else {
        touch($name);
    }
}
```

**Good:**

```php
function createFile(string $name): void
{
    touch($name);
}

function createTempFile(string $name): void
{
    touch('./temp/' . $name);
}
```

**[⬆ back to top](#table-of-contents)**

### Avoid Side Effects

A function produces a side effect if it does anything other than take a value in and
return another value or values. A side effect could be writing to a file, modifying
some global variable, or accidentally wiring all your money to a stranger.

Now, you do need to have side effects in a program on occasion. Like the previous
example, you might need to write to a file. What you want to do is to centralise where
you are doing this. Don't have several functions and classes that write to a particular
file. Have one service that does it. One and only one.

The main point is to avoid common pitfalls like sharing state between objects without
any structure, using mutable data types that can be written to by anything, and not
centralising where your side effects occur. If you can do this, you will be happier
than the vast majority of other programmers.

**Bad:**

```php
// Global variable referenced by following function.
// If we had another function that used this name, now it'd be an array and it could break it.
$name = 'Ryan McDermott';

function splitIntoFirstAndLastName(): void
{
    global $name;

    $name = explode(' ', $name);
}

splitIntoFirstAndLastName();

var_dump($name);
// ['Ryan', 'McDermott'];
```

**Good:**

```php
function splitIntoFirstAndLastName(string $name): array
{
    return explode(' ', $name);
}

$name = 'Ryan McDermott';
$newName = splitIntoFirstAndLastName($name);

var_dump($name);
// 'Ryan McDermott';

var_dump($newName);
// ['Ryan', 'McDermott'];
```

**Best (PHP 8.4):**

```php
readonly class PersonName
{
    public function __construct(
        public string $firstName,
        public string $lastName
    ) {}
    
    public static function fromFullName(string $fullName): self
    {
        [$firstName, $lastName] = explode(' ', $fullName, 2);
        return new self($firstName, $lastName);
    }
    
    public function getFullName(): string
    {
        return "{$this->firstName} {$this->lastName}";
    }
}

$originalName = 'Ryan McDermott';
$personName = PersonName::fromFullName($originalName);

var_dump($originalName); // 'Ryan McDermott'
var_dump($personName->firstName); // 'Ryan'
var_dump($personName->lastName); // 'McDermott'
```

**[⬆ back to top](#table-of-contents)**

### Don't write to global functions

Polluting globals is a bad practice in many languages because you could clash with another
library and the user of your API would be none-the-wiser until they get an exception in
production. Let's think about an example: what if you wanted to have configuration array?
You could write global function like `config()`, but it could clash with another library
that tried to do the same thing.

**Bad:**

```php
function config(): array
{
    return [
        'foo' => 'bar',
    ];
}
```

**Good:**

```php
readonly class Configuration
{
    public function __construct(
        private array $configuration
    ) {}

    public function get(string $key): mixed
    {
        return $this->configuration[$key] ?? null;
    }
}

$configuration = new Configuration([
    'foo' => 'bar',
]);
```

**[⬆ back to top](#table-of-contents)**

### Don't use a Singleton pattern

Singleton is an [anti-pattern](https://en.wikipedia.org/wiki/Singleton_pattern). It violates several SOLID principles and makes testing difficult.

**Bad:**

```php
class DBConnection
{
    private static ?self $instance = null;

    private function __construct(private string $dsn)
    {
        // ...
    }

    public static function getInstance(): self
    {
        if (self::$instance === null) {
            self::$instance = new self('default_dsn');
        }

        return self::$instance;
    }
}

$singleton = DBConnection::getInstance();
```

**Good:**

```php
readonly class DBConnection
{
    public function __construct(
        private string $dsn
    ) {}
    
    // ... connection methods
}

// Use dependency injection
$connection = new DBConnection($dsn);
```

**[⬆ back to top](#table-of-contents)**

### Encapsulate conditionals

**Bad:**

```php
if ($article->state === 'published') {
    // ...
}
```

**Good:**

```php
if ($article->isPublished()) {
    // ...
}
```

**Best (PHP 8.4):**

```php
enum ArticleState: string
{
    case DRAFT = 'draft';
    case PUBLISHED = 'published';
    case ARCHIVED = 'archived';
}

readonly class Article
{
    public function __construct(
        public string $title,
        public string $content,
        public ArticleState $state = ArticleState::DRAFT
    ) {}
    
    public function isPublished(): bool
    {
        return $this->state === ArticleState::PUBLISHED;
    }
}
```

**[⬆ back to top](#table-of-contents)**

### Avoid negative conditionals

**Bad:**

```php
function isDOMNodeNotPresent(\DOMNode $node): bool
{
    // ...
}

if (!isDOMNodeNotPresent($node)) {
    // ...
}
```

**Good:**

```php
function isDOMNodePresent(\DOMNode $node): bool
{
    // ...
}

if (isDOMNodePresent($node)) {
    // ...
}
```

**[⬆ back to top](#table-of-contents)**

### Avoid conditionals

This seems like an impossible task. Upon first hearing this, most people say,
"how am I supposed to do anything without an `if` statement?" The answer is that
you can use polymorphism to achieve the same task in many cases. The second
question is usually, "well that's great but why would I want to do that?" The
answer is a previous clean code concept we learned: a function should only do
one thing. When you have classes and functions that have `if` statements, you
are telling your user that your function does more than one thing. Remember,
just do one thing.

**Bad:**

```php
class Airplane
{
    public function getCruisingAltitude(): int
    {
        return match ($this->type) {
            '777' => $this->getMaxAltitude() - $this->getPassengerCount(),
            'Air Force One' => $this->getMaxAltitude(),
            'Cessna' => $this->getMaxAltitude() - $this->getFuelExpenditure(),
            default => throw new InvalidArgumentException('Unknown aircraft type'),
        };
    }
}
```

**Good:**

```php
interface Airplane
{
    public function getCruisingAltitude(): int;
}

readonly class Boeing777 implements Airplane
{
    public function __construct(
        private int $maxAltitude,
        private int $passengerCount
    ) {}

    public function getCruisingAltitude(): int
    {
        return $this->maxAltitude - $this->passengerCount;
    }
}

readonly class AirForceOne implements Airplane
{
    public function __construct(
        private int $maxAltitude
    ) {}

    public function getCruisingAltitude(): int
    {
        return $this->maxAltitude;
    }
}

readonly class Cessna implements Airplane
{
    public function __construct(
        private int $maxAltitude,
        private int $fuelExpenditure
    ) {}

    public function getCruisingAltitude(): int
    {
        return $this->maxAltitude - $this->fuelExpenditure;
    }
}
```

**[⬆ back to top](#table-of-contents)**

### Avoid type-checking (part 1)

PHP has strong typing capabilities in modern versions. Use them instead of manual type checking.

**Bad:**

```php
function travelToTexas($vehicle): void
{
    if ($vehicle instanceof Bicycle) {
        $vehicle->pedalTo(new Location('texas'));
    } elseif ($vehicle instanceof Car) {
        $vehicle->driveTo(new Location('texas'));
    }
}
```

**Good:**

```php
interface Vehicle
{
    public function travelTo(Location $location): void;
}

function travelToTexas(Vehicle $vehicle): void
{
    $vehicle->travelTo(new Location('texas'));
}
```

**[⬆ back to top](#table-of-contents)**

### Avoid type-checking (part 2)

If you are working with basic primitive values like strings, integers, and arrays,
use PHP's type declarations and strict mode.

**Bad:**

```php
function combine($val1, $val2): int
{
    if (!is_numeric($val1) || !is_numeric($val2)) {
        throw new InvalidArgumentException('Must be of type Number');
    }

    return $val1 + $val2;
}
```

**Good:**

```php
function combine(int|float $val1, int|float $val2): int|float
{
    return $val1 + $val2;
}
```

**[⬆ back to top](#table-of-contents)**

### Remove dead code

Dead code is just as bad as duplicate code. There's no reason to keep it in
your codebase. If it's not being called, get rid of it! It will still be safe
in your version history if you still need it.

**Bad:**

```php
function oldRequestModule(string $url): void
{
    // ...
}

function newRequestModule(string $url): void
{
    // ...
}

$request = newRequestModule($requestUrl);
inventoryTracker('apples', $request, 'www.inventory-awesome.io');
```

**Good:**

```php
function requestModule(string $url): void
{
    // ...
}

$request = requestModule($requestUrl);
inventoryTracker('apples', $request, 'www.inventory-awesome.io');
```

**[⬆ back to top](#table-of-contents)**

## Objects and Data Structures

### Use object encapsulation

In PHP you can set `public`, `protected` and `private` keywords for methods.
Using it, you can control properties modification on an object.

* When you want to do more beyond getting an object property, you don't have
to look up and change every accessor in your codebase.
* Makes adding validation simple when doing a `set`.
* Encapsulates the internal representation.
* Easy to add logging and error handling when getting and setting.
* Inheriting this class, you can override default functionality.
* You can lazy load your object's properties, let's say getting it from a
server.

Additionally, this is part of [Open/Closed](#openclosed-principle-ocp) principle.

**Bad:**

```php
class BankAccount
{
    public int $balance = 1000;
}

$bankAccount = new BankAccount();

// Buy shoes...
$bankAccount->balance -= 100;
```

**Good:**

```php
class BankAccount
{
    public function __construct(
        private int $balance = 1000
    ) {}

    public function withdraw(int $amount): void
    {
        if ($amount > $this->balance) {
            throw new InvalidArgumentException('Amount greater than available balance.');
        }

        $this->balance -= $amount;
    }

    public function deposit(int $amount): void
    {
        $this->balance += $amount;
    }

    public function getBalance(): int
    {
        return $this->balance;
    }
}

$bankAccount = new BankAccount();

// Buy shoes...
$bankAccount->withdraw($shoesPrice);

// Get balance
$balance = $bankAccount->getBalance();
```

**[⬆ back to top](#table-of-contents)**

### Make objects have private/protected members

* `public` methods and properties are most dangerous for changes, because some outside code may easily rely on them and you can't control what code relies on them. **Modifications in class are dangerous for all users of class.**
* `protected` modifier are as dangerous as public, because they are available in scope of any child class. This effectively means that difference between public and protected is only in access mechanism, but encapsulation guarantee remains the same. **Modifications in class are dangerous for all descendant classes.**
* `private` modifier guarantees that code is **dangerous to modify only in boundaries of single class** (you are safe for modifications and you won't have [Jenga effect](http://www.urbandictionary.com/define.php?term=Jengaphobia&defid=2494196)).

Therefore, use `private` by default and `public/protected` when you need to provide access for external classes.

**Bad:**

```php
class Employee
{
    public string $name;

    public function __construct(string $name)
    {
        $this->name = $name;
    }
}

$employee = new Employee('John Doe');
// Employee name: John Doe
echo 'Employee name: ' . $employee->name;
```

**Good:**

```php
readonly class Employee
{
    public function __construct(
        private string $name
    ) {}

    public function getName(): string
    {
        return $this->name;
    }
}

$employee = new Employee('John Doe');
// Employee name: John Doe
echo 'Employee name: ' . $employee->getName();
```

**[⬆ back to top](#table-of-contents)**

### Use readonly properties

PHP 8.1 introduced readonly properties, and PHP 8.2 introduced readonly classes. Use them to create immutable objects.

**Bad:**

```php
class Point
{
    private float $x;
    private float $y;

    public function __construct(float $x, float $y)
    {
        $this->x = $x;
        $this->y = $y;
    }

    public function getX(): float
    {
        return $this->x;
    }

    public function getY(): float
    {
        return $this->y;
    }
}
```

**Good:**

```php
readonly class Point
{
    public function __construct(
        public float $x,
        public float $y
    ) {}
    
    public function distanceFrom(Point $other): float
    {
        return sqrt(($this->x - $other->x) ** 2 + ($this->y - $other->y) ** 2);
    }
}
```

**[⬆ back to top](#table-of-contents)**

## Classes

### Prefer composition over inheritance

As stated famously in [*Design Patterns*](https://en.wikipedia.org/wiki/Design_Patterns) by the Gang of Four,
you should prefer composition over inheritance where you can. There are lots of
good reasons to use inheritance and lots of good reasons to use composition.
The main point for this maxim is that if your mind instinctively goes for
inheritance, try to think if composition could model your problem better. In some
cases it can.

You might be wondering then, "when should I use inheritance?" It
depends on your problem at hand, but this is a decent list of when inheritance
makes more sense than composition:

1. Your inheritance represents an "is-a" relationship and not a "has-a"
relationship (Human->Animal vs. User->UserDetails).
2. You can reuse code from the base classes (Humans can move like all animals).
3. You want to make global changes to derived classes by changing a base class.
(Change the caloric expenditure of all animals when they move).

**Bad:**

```php
class Employee
{
    public function __construct(
        private string $name,
        private string $email
    ) {}
}

// Bad because Employees "have" tax data.
// EmployeeTaxData is not a type of Employee
class EmployeeTaxData extends Employee
{
    public function __construct(
        string $name,
        string $email,
        private string $ssn,
        private string $salary
    ) {
        parent::__construct($name, $email);
    }
}
```

**Good:**

```php
readonly class EmployeeTaxData
{
    public function __construct(
        public string $ssn,
        public string $salary
    ) {}
}

readonly class Employee
{
    public function __construct(
        public string $name,
        public string $email,
        public ?EmployeeTaxData $taxData = null
    ) {}
    
    public function setTaxData(EmployeeTaxData $taxData): self
    {
        return new self($this->name, $this->email, $taxData);
    }
}
```

**[⬆ back to top](#table-of-contents)**

### Avoid fluent interfaces

A [Fluent interface](https://en.wikipedia.org/wiki/Fluent_interface) is an object
oriented API that aims to improve the readability of the source code by using
[Method chaining](https://en.wikipedia.org/wiki/Method_chaining).

While there can be some contexts, frequently builder objects, where this
pattern reduces the verbosity of the code, more often it comes at some costs:

1. Breaks [Encapsulation](https://en.wikipedia.org/wiki/Encapsulation_%28object-oriented_programming%29).
2. Breaks [Decorators](https://en.wikipedia.org/wiki/Decorator_pattern).
3. Is harder to [mock](https://en.wikipedia.org/wiki/Mock_object) in a test suite.
4. Makes diffs of commits harder to read.

**Bad:**

```php
class Car
{
    private string $make = 'Honda';
    private string $model = 'Accord';
    private string $colour = 'white';

    public function setMake(string $make): self
    {
        $this->make = $make;
        return $this;
    }

    public function setModel(string $model): self
    {
        $this->model = $model;
        return $this;
    }

    public function setColour(string $colour): self
    {
        $this->colour = $colour;
        return $this;
    }

    public function dump(): void
    {
        var_dump($this->make, $this->model, $this->colour);
    }
}

$car = (new Car())
    ->setColour('pink')
    ->setMake('Ford')
    ->setModel('F-150')
    ->dump();
```

**Good:**

```php
readonly class Car
{
    public function __construct(
        public string $make = 'Honda',
        public string $model = 'Accord',
        public string $colour = 'white'
    ) {}

    public function withMake(string $make): self
    {
        return new self($make, $this->model, $this->colour);
    }

    public function withModel(string $model): self
    {
        return new self($this->make, $model, $this->colour);
    }

    public function withColour(string $colour): self
    {
        return new self($this->make, $this->model, $colour);
    }

    public function dump(): void
    {
        var_dump($this->make, $this->model, $this->colour);
    }
}

$car = (new Car())
    ->withColour('pink')
    ->withMake('Ford')
    ->withModel('F-150');
$car->dump();
```

**[⬆ back to top](#table-of-contents)**

### Prefer final classes

The `final` keyword should be used whenever possible:

1. It prevents an uncontrolled inheritance chain.
2. It encourages [composition](#prefer-composition-over-inheritance).
3. It encourages the [Single Responsibility Principle](#single-responsibility-principle-srp).
4. It encourages developers to use your public methods instead of extending the class to get access to protected ones.
5. It allows you to change your code without breaking applications that use your class.

The only condition is that your class should implement an interface and no other public methods are defined.

**Bad:**

```php
final class Car
{
    public function __construct(
        private string $colour
    ) {}

    public function getColour(): string
    {
        return $this->colour;
    }
}
```

**Good:**

```php
interface Vehicle
{
    public function getColour(): string;
}

final readonly class Car implements Vehicle
{
    public function __construct(
        private string $colour
    ) {}

    public function getColour(): string
    {
        return $this->colour;
    }
}
```

**[⬆ back to top](#table-of-contents)**

### Use property promotion

PHP 8.0 introduced constructor property promotion, which reduces boilerplate code significantly.

**Bad:**

```php
class User
{
    private string $name;
    private string $email;
    private int $age;

    public function __construct(string $name, string $email, int $age)
    {
        $this->name = $name;
        $this->email = $email;
        $this->age = $age;
    }

    public function getName(): string
    {
        return $this->name;
    }

    public function getEmail(): string
    {
        return $this->email;
    }

    public function getAge(): int
    {
        return $this->age;
    }
}
```

**Good:**

```php
readonly class User
{
    public function __construct(
        public string $name,
        public string $email,
        public int $age
    ) {}
}
```

**[⬆ back to top](#table-of-contents)**

### Use enums for constants

PHP 8.1 introduced enums, which are perfect for representing a fixed set of possible values.

**Bad:**

```php
class OrderStatus
{
    public const PENDING = 'pending';
    public const CONFIRMED = 'confirmed';
    public const SHIPPED = 'shipped';
    public const DELIVERED = 'delivered';
    public const CANCELLED = 'cancelled';
}

class Order
{
    public function __construct(
        public string $status = OrderStatus::PENDING
    ) {}
}
```

**Good:**

```php
enum OrderStatus: string
{
    case PENDING = 'pending';
    case CONFIRMED = 'confirmed';
    case SHIPPED = 'shipped';
    case DELIVERED = 'delivered';
    case CANCELLED = 'cancelled';
    
    public function canBeModified(): bool
    {
        return match($this) {
            self::PENDING, self::CONFIRMED => true,
            default => false,
        };
    }
}

readonly class Order
{
    public function __construct(
        public OrderStatus $status = OrderStatus::PENDING
    ) {}
    
    public function canModify(): bool
    {
        return $this->status->canBeModified();
    }
}
```

**[⬆ back to top](#table-of-contents)**

## SOLID

**SOLID** is the mnemonic acronym introduced by Michael Feathers for the first five principles named by Robert Martin, which meant five basic principles of object-oriented programming and design.

 * [S: Single Responsibility Principle (SRP)](#single-responsibility-principle-srp)
 * [O: Open/Closed Principle (OCP)](#openclosed-principle-ocp)
 * [L: Liskov Substitution Principle (LSP)](#liskov-substitution-principle-lsp)
 * [I: Interface Segregation Principle (ISP)](#interface-segregation-principle-isp)
 * [D: Dependency Inversion Principle (DIP)](#dependency-inversion-principle-dip)

### Single Responsibility Principle (SRP)

As stated in Clean Code, "There should never be more than one reason for a class
to change". It's tempting to jam-pack a class with a lot of functionality, like
when you can only take one suitcase on your flight. The issue with this is
that your class won't be conceptually cohesive and it will give it many reasons
to change. Minimising the amount of times you need to change a class is important.
It's important because if too much functionality is in one class and you modify a piece of it,
it can be difficult to understand how that will affect other dependent modules in
your codebase.

**Bad:**

```php
class UserSettings
{
    public function __construct(
        private User $user
    ) {}

    public function changeSettings(array $settings): void
    {
        if ($this->verifyCredentials()) {
            // ...
        }
    }

    private function verifyCredentials(): bool
    {
        // ...
    }
}
```

**Good:**

```php
readonly class UserAuth
{
    public function __construct(
        private User $user
    ) {}

    public function verifyCredentials(): bool
    {
        // ...
    }
}

readonly class UserSettings
{
    public function __construct(
        private User $user,
        private UserAuth $auth
    ) {}

    public function changeSettings(array $settings): void
    {
        if ($this->auth->verifyCredentials()) {
            // ...
        }
    }
}
```

**[⬆ back to top](#table-of-contents)**

### Open/Closed Principle (OCP)

As stated by Bertrand Meyer, "software entities (classes, modules, functions,
etc.) should be open for extension, but closed for modification." What does that
mean though? This principle basically states that you should allow users to
add new functionalities without changing existing code.

**Bad:**

```php
abstract class Adapter
{
    abstract public function getName(): string;
}

class AjaxAdapter extends Adapter
{
    public function getName(): string
    {
        return 'ajaxAdapter';
    }
}

class NodeAdapter extends Adapter
{
    public function getName(): string
    {
        return 'nodeAdapter';
    }
}

class HttpRequester
{
    public function __construct(
        private Adapter $adapter
    ) {}

    public function fetch(string $url): Promise
    {
        return match ($this->adapter->getName()) {
            'ajaxAdapter' => $this->makeAjaxCall($url),
            'httpNodeAdapter' => $this->makeHttpCall($url),
            default => throw new InvalidArgumentException('Unknown adapter'),
        };
    }

    private function makeAjaxCall(string $url): Promise
    {
        // request and return promise
    }

    private function makeHttpCall(string $url): Promise
    {
        // request and return promise
    }
}
```

**Good:**

```php
interface Adapter
{
    public function request(string $url): Promise;
}

final readonly class AjaxAdapter implements Adapter
{
    public function request(string $url): Promise
    {
        // request and return promise
    }
}

final readonly class NodeAdapter implements Adapter
{
    public function request(string $url): Promise
    {
        // request and return promise
    }
}

final readonly class HttpRequester
{
    public function __construct(
        private Adapter $adapter
    ) {}

    public function fetch(string $url): Promise
    {
        return $this->adapter->request($url);
    }
}
```

**[⬆ back to top](#table-of-contents)**

### Liskov Substitution Principle (LSP)

This is a scary term for a very simple concept. It's formally defined as "If S
is a subtype of T, then objects of type T may be replaced with objects of type S
(i.e., objects of type S may substitute objects of type T) without altering any
of the desirable properties of that program (correctness, task performed,
etc.)." That's an even scarier definition.

The best explanation for this is if you have a parent class and a child class,
then the base class and child class can be used interchangeably without getting
incorrect results. This might still be confusing, so let's take a look at the
classic Square-Rectangle example. Mathematically, a square is a rectangle, but
if you model it using the "is-a" relationship via inheritance, you quickly
get into trouble.

**Bad:**

```php
class Rectangle
{
    public function __construct(
        protected int $width = 0,
        protected int $height = 0
    ) {}

    public function setWidth(int $width): void
    {
        $this->width = $width;
    }

    public function setHeight(int $height): void
    {
        $this->height = $height;
    }

    public function getArea(): int
    {
        return $this->width * $this->height;
    }
}

class Square extends Rectangle
{
    public function setWidth(int $width): void
    {
        $this->width = $this->height = $width;
    }

    public function setHeight(int $height): void
    {
        $this->width = $this->height = $height;
    }
}

function printArea(Rectangle $rectangle): void
{
    $rectangle->setWidth(4);
    $rectangle->setHeight(5);

    // BAD: Will return 25 for Square. Should be 20.
    echo sprintf('%s has area %d.', get_class($rectangle), $rectangle->getArea()) . PHP_EOL;
}

$rectangles = [new Rectangle(), new Square()];

foreach ($rectangles as $rectangle) {
    printArea($rectangle);
}
```

**Good:**

The best way is separate the quadrangles and allocation of a more general subtype for both shapes.

```php
interface Shape
{
    public function getArea(): int;
}

final readonly class Rectangle implements Shape
{
    public function __construct(
        public int $width,
        public int $height
    ) {}

    public function getArea(): int
    {
        return $this->width * $this->height;
    }
}

final readonly class Square implements Shape
{
    public function __construct(
        public int $length
    ) {}

    public function getArea(): int
    {
        return $this->length ** 2;
    }
}

function printArea(Shape $shape): void
{
    echo sprintf('%s has area %d.', get_class($shape), $shape->getArea()) . PHP_EOL;
}

$shapes = [new Rectangle(4, 5), new Square(5)];

foreach ($shapes as $shape) {
    printArea($shape);
}
```

**[⬆ back to top](#table-of-contents)**

### Interface Segregation Principle (ISP)

ISP states that "Clients should not be forced to depend upon interfaces that
they do not use."

A good example to look at that demonstrates this principle is for
classes that require large settings objects. Not requiring clients to set up
huge amounts of options is beneficial, because most of the time they won't need
all of the settings. Making them optional helps prevent having a "fat interface".

**Bad:**

```php
interface Employee
{
    public function work(): void;
    public function eat(): void;
}

final readonly class HumanEmployee implements Employee
{
    public function work(): void
    {
        // ....working
    }

    public function eat(): void
    {
        // ...... eating in lunch break
    }
}

final readonly class RobotEmployee implements Employee
{
    public function work(): void
    {
        //.... working much more
    }

    public function eat(): void
    {
        //.... robot can't eat, but it must implement this method
    }
}
```

**Good:**

Not every worker is an employee, but every employee is a worker.

```php
interface Workable
{
    public function work(): void;
}

interface Feedable
{
    public function eat(): void;
}

interface Employee extends Feedable, Workable
{
}

final readonly class HumanEmployee implements Employee
{
    public function work(): void
    {
        // ....working
    }

    public function eat(): void
    {
        //.... eating in lunch break
    }
}

// robot can only work
final readonly class RobotEmployee implements Workable
{
    public function work(): void
    {
        // ....working
    }
}
```

**[⬆ back to top](#table-of-contents)**

### Dependency Inversion Principle (DIP)

This principle states two essential things:
1. High-level modules should not depend on low-level modules. Both should
depend on abstractions.
2. Abstractions should not depend upon details. Details should depend on
abstractions.

This can be hard to understand at first, but if you've worked with PHP frameworks (like Symfony), you've seen an implementation of this principle in the form of Dependency
Injection (DI). While they are not identical concepts, DIP keeps high-level
modules from knowing the details of its low-level modules and setting them up.
It can accomplish this through DI. A huge benefit of this is that it reduces
the coupling between modules. Coupling is a very bad development pattern because
it makes your code hard to refactor.

**Bad:**

```php
class Employee
{
    public function work(): void
    {
        // ....working
    }
}

class Robot extends Employee
{
    public function work(): void
    {
        //.... working much more
    }
}

class Manager
{
    public function __construct(
        private Employee $employee
    ) {}

    public function manage(): void
    {
        $this->employee->work();
    }
}
```

**Good:**

```php
interface Employee
{
    public function work(): void;
}

final readonly class Human implements Employee
{
    public function work(): void
    {
        // ....working
    }
}

final readonly class Robot implements Employee
{
    public function work(): void
    {
        //.... working much more
    }
}

final readonly class Manager
{
    public function __construct(
        private Employee $employee
    ) {}

    public function manage(): void
    {
        $this->employee->work();
    }
}
```

**[⬆ back to top](#table-of-contents)**

## Don't repeat yourself (DRY)

Try to observe the [DRY](https://en.wikipedia.org/wiki/Don%27t_repeat_yourself) principle.

Do your absolute best to avoid duplicate code. Duplicate code is bad because
it means that there's more than one place to alter something if you need to
change some logic.

Imagine if you run a restaurant and you keep track of your inventory: all your
tomatoes, onions, garlic, spices, etc. If you have multiple lists that
you keep this on, then all have to be updated when you serve a dish with
tomatoes in them. If you only have one list, there's only one place to update!

Often you have duplicate code because you have two or more slightly
different things, that share a lot in common, but their differences force you
to have two or more separate functions that do much of the same things. Removing
duplicate code means creating an abstraction that can handle this set of different
things with just one function/module/class.

Getting the abstraction right is critical, that's why you should follow the
SOLID principles laid out in the [Classes](#classes) section. Bad abstractions can be
worse than duplicate code, so be careful! Having said this, if you can make
a good abstraction, do it! Don't repeat yourself, otherwise you'll find yourself
updating multiple places any time you want to change one thing.

**Bad:**

```php
function showDeveloperList(array $developers): void
{
    foreach ($developers as $developer) {
        $expectedSalary = $developer->calculateExpectedSalary();
        $experience = $developer->getExperience();
        $githubLink = $developer->getGithubLink();

        render([$expectedSalary, $experience, $githubLink]);
    }
}

function showManagerList(array $managers): void
{
    foreach ($managers as $manager) {
        $expectedSalary = $manager->calculateExpectedSalary();
        $experience = $manager->getExperience();
        $githubLink = $manager->getGithubLink();

        render([$expectedSalary, $experience, $githubLink]);
    }
}
```

**Good:**

```php
interface Employee
{
    public function calculateExpectedSalary(): int;
    public function getExperience(): int;
    public function getGithubLink(): string;
}

function showEmployeeList(array $employees): void
{
    foreach ($employees as $employee) {
        render([
            $employee->calculateExpectedSalary(),
            $employee->getExperience(),
            $employee->getGithubLink()
        ]);
    }
}
```

**[⬆ back to top](#table-of-contents)**

## Modern PHP Features

### Union Types

PHP 8.0 introduced union types, allowing a value to be one of several types.

**Good:**

```php
function processId(int|string $id): string
{
    return match(true) {
        is_int($id) => "Processing ID: {$id}",
        is_string($id) => "Processing string ID: {$id}",
    };
}

class ApiResponse
{
    public function __construct(
        public array|object $data,
        public int|null $errorCode = null
    ) {}
}
```

### Named Arguments

PHP 8.0 introduced named arguments, making function calls more readable.

**Good:**

```php
readonly class User
{
    public function __construct(
        public string $name,
        public string $email,
        public int $age,
        public bool $isActive = true,
        public ?string $department = null
    ) {}
}

// Clear and readable
$user = new User(
    name: 'John Doe',
    email: 'john@example.com',
    age: 30,
    department: 'Engineering'
);
```

### Attributes

PHP 8.0 introduced attributes as a structured form of metadata.

**Good:**

```php
#[Attribute]
readonly class Route
{
    public function __construct(
        public string $path,
        public string $method = 'GET'
    ) {}
}

#[Attribute]
readonly class Validate
{
    public function __construct(
        public string $rule
    ) {}
}

class UserController
{
    #[Route('/users', 'POST')]
    public function createUser(
        #[Validate('required|email')] string $email,
        #[Validate('required|min:3')] string $name
    ): User {
        // ...
    }
}
```

### First-class Callable Syntax

PHP 8.1 introduced first-class callable syntax for creating closures from callable.

**Good:**

```php
$users = [
    new User('Alice', 'alice@example.com', 25),
    new User('Bob', 'bob@example.com', 30),
    new User('Charlie', 'charlie@example.com', 35),
];

// First-class callable syntax
$names = array_map($users->getName(...), $users);
$emails = array_map($users->getEmail(...), $users);

// Or for static methods
$validated = array_filter($emails, EmailValidator::isValid(...));
```

**[⬆ back to top](#table-of-contents)**
