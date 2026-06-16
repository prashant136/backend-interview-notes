
# ==what is Validations in Spring Boot ? (why its needed and how it works) How do you validate request data in a Spring Boot controller? @Valid and @Validated?

his is a **very important Spring Boot interview topic**.

Many developers know how to write `@Valid`, but interviewers often ask:

> Why do we need validation?  
> What problem does it solve?  
> How does `@Valid` work internally?  
> Difference between `@Valid` and `@Validated`?

Let's build the intuition first.

---

## 1. What is Validation?

Validation means:

> Checking whether incoming data is correct before processing it.

For example:

A user sends:

```
{
    "name": "",
    "email": "abc",
    "age": -5
}
```

Clearly:

- Name should not be empty
- Email format is wrong
- Age cannot be negative

If we save this directly into database:

```
userRepository.save(user);
```

we will store invalid data.

Validation prevents this.

---

## 2. Why Validation is Needed?

Imagine you're building a Banking App.

API:

```
POST /transfer
```

Request:

```
{   "amount": -10000  }
```

Without validation:

```
account.transfer(-10000);
```

Now your business logic becomes messy.

Every service method must check:

```
if(amount <= 0)

if(email == null)

if(name.isEmpty())
```

everywhere.

This creates:

❌ Duplicate code
❌ Hard maintenance
❌ Bugs
❌ Invalid DB records

---

## 3. Validation Flow in Spring Boot

Suppose request comes:

```
{
   "name":"",
   "email":"abc"
}
```

Flow:

```
Client
  |
  v
Controller
  |
  v
Validation (@Valid)
  |
  |---- Fail -> 400 Bad Request
  |
  |---- Pass
  |
  v
Service
  |
  v
Database
```

Validation acts like a security guard.

```
Invalid Data
      |
      v
   BLOCKED
```

before reaching business logic.

---

## 4. Bean Validation API

Spring Boot uses:

```
Jakarta Bean Validation
```

Implementation:

```
Hibernate Validator
```

Think:

```
Spring Boot
      |
      v
Bean Validation API
      |
      v
Hibernate Validator
```

Hibernate Validator actually performs validation.

---

## 5. Example Without Validation

DTO:

```
public class UserRequest {

    private String name;
    private String email;
    private Integer age;

}
```

Controller:

```
@PostMapping("/users")
public String createUser(
        @RequestBody UserRequest request) {

    return "User Created";
}
```

Now Spring accepts:

```
{
   "name":"",
   "email":"wrong",
   "age":-100
}
```

No checks.

Bad data enters system.

---

## 6. Adding Validation

DTO:

```
import jakarta.validation.constraints.*;

public class UserRequest {

    @NotBlank
    private String name;

    @Email
    private String email;

    @Min(18)
    private Integer age;

    // getters setters
}
```

Controller:

```
@PostMapping("/users")
public String createUser(
        @Valid @RequestBody UserRequest request) {

    return "User Created";
}
```

Now validation is active.

---

## 7. Common Validation Annotations

### @NotNull

Value cannot be null.

```
@NotNull
private String name;
```

Allowed:

```
"John"
```

Not Allowed:

```
null
```

---

### @NotEmpty

For String/List/Set.

```
@NotEmpty
private String name;
```

Not allowed:

```
null
""
```

Allowed:

```
" "
```

(space is allowed)

---

### @NotBlank

Most commonly used.

```
@NotBlank
private String name;
```

Not allowed:

```
null
""
"     "
```

Allowed:

```
"John"
```

---

### @Email

```
@Email
private String email;
```

Valid:

```
abc@gmail.com
```

Invalid:

```
abc
```

---

### @Min

```
@Min(18)
private Integer age;
```

Valid:

```
20
```

Invalid:

```
15
```

---

### @Max

```
@Max(100)
private Integer age;
```

---

### @Size

```
@Size(min=3, max=20)
private String username;
```

Valid:

```
john
```

Invalid:

```
ab
```

---

### @Pattern

For regex validation.

Phone number:

```
@Pattern(
 regex = "^[0-9]{10}$"
)
private String mobile;
```

Valid:

```
9876543210
```

Invalid:

```
123
```

---

## 8. What Does @Valid Do?

Controller:

```
@PostMapping("/users")
public String createUser(
        @Valid @RequestBody UserRequest request) {

    return "Success";
}
```

When request arrives:

Spring sees:

```
@Valid
```

and says:

```
Before calling controller method,
validate UserRequest object.
```

Internally:

```
JSON
 |
 v
Object Creation
 |
 v
@Valid
 |
 v
Hibernate Validator
 |
 v
Check annotations
```

Checks:

```
@NotBlank
```

```
@Email
```

```
@Min
```

etc.

If any fail:

```
MethodArgumentNotValidException
```

is thrown.

Controller method never executes.

---


## 9. Custom Error Message

```
@NotBlank(
    message = "Name is mandatory"
)
private String name;
```

```
@Email(
    message = "Invalid Email"
)
private String email;
```

Now response:

```
{
   "name":"Name is mandatory",
   "email":"Invalid Email"
}
```

---

## 10. What is @Validated?

Many developers think:

```
@Valid
```

and

```
@Validated
```

are same.

Not exactly.

---

### ==@Valid

Comes from:

```
jakarta.validation.Valid
```

Used for:

```
Request Body Validation
```

Example:

```
@PostMapping
public void create(
    @Valid @RequestBody UserRequest request) {

}
```

---

### ==@Validated

Comes from:

```
org.springframework.validation.annotation.Validated
```

Spring-specific annotation.

Used for:

### Method Parameter Validation

Example:

```
@RestController
@Validated
public class UserController {

    @GetMapping("/users")
    public String getUser(
            @Min(1)
            @RequestParam Long id) {

        return "User";
    }
}
```

Request:

```
/users?id=-5
```

Validation fails.

Without:

```
@Validated
```

Spring won't validate request parameters.

---

## Why @Validated is Needed?

Because:

```
@Valid
```

validates object fields.

But:

```
@RequestParam
@PathVariable
```

are not objects.

Example:

```
@GetMapping("/{id}")
public User getUser(
      @PathVariable
      @Min(1) Long id)
```

For this to work:

```
@Validated
```

must be on class.

```
@RestController
@Validated
public class UserController
```

---

## Real Production Example

DTO:

```
public class RegisterUserRequest {

    @NotBlank
    private String name;

    @Email
    private String email;

    @Size(min = 8)
    private String password;
}
```

Controller:

```
@RestController
@RequestMapping("/users")
@Validated
public class UserController {

    @PostMapping
    public String register(
            @Valid
            @RequestBody RegisterUserRequest request) {

        return "User Registered";
    }

    @GetMapping("/{id}")
    public String getUser(
            @PathVariable
            @Min(1)
            Long id) {

        return "User";
    }
}
```


---

## Interview Answer (2 Minutes)

>**Validation** is used to ensure incoming request data is correct before business logic executes. It prevents invalid data from reaching services and databases.

> Spring Boot uses **Jakarta Bean Validation API** with **Hibernate Validator** as the implementation.

>We add validation annotations like:

```
@NotBlank
@Email
@Min
@Max
@Size
@Pattern
```

on DTO fields.

>`@Valid` is placed on `@RequestBody` objects and tells Spring to validate the DTO before invoking the controller method.

>If validation fails, Spring throws `MethodArgumentNotValidException` and returns `400 Bad Request`.

>`@Validated` is a Spring-specific annotation mainly used for validating method parameters such as `@RequestParam` and `@PathVariable`, and also supports validation groups.

>**Rule to remember:**

```
@RequestBody DTO  -> @Valid

@RequestParam /
@PathVariable     -> @Validated
```


#
----
----
----
---

# ==What is BindingResult?

This is a favorite interview question.

---

### First Understand the Problem

Normally:

```
@PostMapping
public String create(
      @Valid @RequestBody UserRequest request) {

    return "Success";
}
```

If validation fails:

```
MethodArgumentNotValidException
```

is thrown immediately.

Controller method never executes.

---

### What if we don't want an Exception?

Maybe we want:

```
Check errors manually
```

instead of throwing exception.

That's where:

```
BindingResult
```

comes in.

---

### Definition

`BindingResult` stores:

1. Data binding errors
2. Validation errors

Think:

```
JSON Request
      |
      v
Java Object
      |
      +--> Validation Errors
      |
      +--> BindingResult
```

---
### Example

DTO

```
public class UserRequest {

    @NotBlank
    private String name;

    @Email
    private String email;
}
```

Controller:

```
@PostMapping("/users")
public ResponseEntity<?> createUser(
        @Valid
        @RequestBody UserRequest request,
        BindingResult result) {

    if(result.hasErrors()) {

        List<String> errors =
                result.getFieldErrors()
                      .stream()
                      .map(FieldError::getDefaultMessage)
                      .toList();

        return ResponseEntity
                .badRequest()
                .body(errors);
    }

    return ResponseEntity.ok("Success");
}
```

---
### Request

```
{
   "name":"",
   "email":"abc"
}
```

---
### Output

```
[
   "must not be blank",
   "must be a well-formed email address"
]
```

Notice:

No Exception.

Controller still executes.

Because errors are stored in:

```
BindingResult
```

---

## Internal Working

Spring Flow:

```
JSON Request
      |
      v
Object Creation
      |
      v
Validation
      |
      +---- Errors Found
      |
      v
BindingResult
      |
      v
Controller Method
```

Without BindingResult:

```
Validation Fail
      |
      v
MethodArgumentNotValidException
      |
      v
Global Exception Handler
```

With BindingResult:

```
Validation Fail
      |
      v
BindingResult
      |
      v
Controller Executes
```

---

## Common BindingResult Methods

### hasErrors()

```
result.hasErrors()
```

Any validation error?

---
### getFieldErrors()

```
result.getFieldErrors()
```

Returns all field errors.

Example:

```
name -> must not be blank

email -> invalid email
```

---

### getErrorCount()

```
result.getErrorCount()
```

Example:

```
2
```

---

### getFieldValue()

```
result.getFieldValue("name")
```

Returns submitted value.

## REST APIs: BindingResult vs Global Exception Handler

### REST APIs

Most companies prefer:

```
@RestControllerAdvice
```

because:

- Centralized
- Clean
- Reusable
- Less code

Example:

```
50 APIs
```

One exception handler.

#
----
---
---

# ==Validation on Nested Properties ?

Suppose you're creating an Order API.

Request:

```
{
  "orderId": 101,
  "customer": {
    "name": "",
    "email": "abc"
  }
}
```

You already know how to validate:

```
public class OrderRequest {

    private Long orderId;

    private CustomerRequest customer;
}
```

and

```
public class CustomerRequest {

    @NotBlank
    private String name;

    @Email
    private String email;
}
```

Question:

Will Spring validate `customer.name` and `customer.email` automatically?

### Answer

❌ No

Spring validates only the top-level object.

---

## Solution: Nested @Valid

```
public class OrderRequest {

    private Long orderId;

    @Valid
    private CustomerRequest customer;
}
```

Now Spring does:

```
OrderRequest
      |
      v
CustomerRequest
      |
      v
Validate fields
```

Request:

```
{
  "orderId":101,
  "customer":{
      "name":"",
      "email":"wrong"
  }
}
```

Validation fails.

---

### Real Production Example

```
public class AddressRequest {

    @NotBlank
    private String city;

    @NotBlank
    private String country;
}
```

```
public class UserRequest {

    @NotBlank
    private String name;

    @Valid
    private AddressRequest address;
}
```

Request:

```
{
  "name":"Prashant",
  "address":{
      "city":"",
      "country":"India"
  }
}
```

Spring will validate:

```
UserRequest
      |
      v
AddressRequest
      |
      v
city cannot be blank
```

---

## Internal Working of Nested Validation

When Spring sees:

```
@Valid
private CustomerRequest customer;
```

it performs:

```
validate(OrderRequest)

  |
  +-- validate(customer)

          |
          +-- validate(name)

          +-- validate(email)
```

This is called:
### Cascading Validation

or,
### Recursive Validation

Interview keyword:

> `@Valid` on nested objects enables Cascading Validation.

#
----
----
----

# ==Global Exception Handling for Validation


Default Spring response:

```
{
  "timestamp":"...",
  "status":400,
  "error":"Bad Request"
}
```

Not user friendly.

Frontend wants:

```
{
  "name":"Name is required",
  "email":"Invalid email"
}
```

---
### Solution

Use:

```
@RestControllerAdvice
```

---

### Create Global Exception Handler

```
@RestControllerAdvice
public class GlobalExceptionHandler {}
```

Think:

```
All Controllers
      |
      v
Exception
      |
      v
Global Handler
```

---

### Handle Validation Exceptions

```
@ExceptionHandler(
    MethodArgumentNotValidException.class
)
public ResponseEntity<Map<String,String>>
handleValidationException(
      MethodArgumentNotValidException ex) {

    Map<String,String> errors = new HashMap<>();

    ex.getBindingResult()
      .getFieldErrors()
      .forEach(error -> {

          errors.put(
              error.getField(),
              error.getDefaultMessage()
          );
      });

    return ResponseEntity
            .badRequest()
            .body(errors);
}
```

---

### Request

```
{
  "name":"",
  "email":"wrong"
}
```

---

## Response

```
{
  "name":"must not be blank",
  "email":"must be a well-formed email address"
}
```

Much cleaner.

---

## Production Standard Error Response

Most companies create a custom response.

```
public class ErrorResponse {

    private String message;

    private Map<String,String> errors;
}
```

Response:

```
{
   "message":"Validation Failed",
   "errors":{
      "name":"Name is required",
      "email":"Invalid email"
   }
}
```

This is what you usually see in production APIs.

#
----
-----
----


Most developers know:

```
@NotNull@NotBlank@Email
```

But in real projects, business rules are often custom.

Examples:

- Password must contain uppercase + lowercase + number + special character
- Employee ID must start with EMP-
- Product SKU must follow company format
- User age must be >= 18 if country = India
- Catalog feed URL must be from approved domains

For these, built-in annotations are not enough.

That's why we create **Custom Validation Annotations**.

---

## Step 1: First Understand How Built-in Validation Works

When you write:

```
@NotBlank
private String name;
```

You are actually using:

```
Annotation
    +
Validator Class
```

Internally Spring/Hibernate Validator does:

```
@NotBlank
      |
      v
NotBlankValidator
      |
      v
true / false
```

---

## Real Goal

We want to create our own annotation:

```
@ValidEmployeeId
private String employeeId;
```

Usage:

```
{
   "employeeId":"EMP-123"
}
```

Valid ✅

```
{
   "employeeId":"ABC-123"
}
```

Invalid ❌

---

## Overall Architecture

Custom Validation always has 2 parts:

```
1. Annotation
       |
       v
2. Validator Class
       |
       v
Validation Logic
```

Like:

```
@ValidEmployeeId
        |
        v
EmployeeIdValidator
        |
        v
check EMP-
```

---

## PART 1: Custom Validation Annotation

Create:

```
ValidEmployeeId.java
```

```
import jakarta.validation.Constraint;
import jakarta.validation.Payload;

import java.lang.annotation.*;

@Target(ElementType.FIELD)
@Retention(RetentionPolicy.RUNTIME)
@Constraint(validatedBy = EmployeeIdValidator.class)
public @interface ValidEmployeeId {

    String message()
            default "Invalid Employee ID";

    Class<?>[] groups() default {};

    Class<? extends Payload>[] payload()
            default {};
}
```

Looks scary.

Let's understand every line.

----

### @Target

Question: Where can this annotation be used?

Example:

```
@Target(ElementType.FIELD)
```

means:

```
@ValidEmployeeId
private String employeeId;
```

Allowed.

---

Not allowed:

```
@ValidEmployeeId
public void save() {}
```

because method isn't FIELD.

---

Common Targets

#### FIELD

```
@Target(ElementType.FIELD)
```

Used on fields.

---

#### METHOD

```
@Target(ElementType.METHOD)
```

Used on methods.

---

#### PARAMETER

```
@Target(ElementType.PARAMETER)
```

Used on method parameters.

---

#### TYPE

```
@Target(ElementType.TYPE)
```

Used on class level.

Example:

```
@ValidPasswordMatch
public class RegisterRequest {}
```

---

### @Retention

Question:

How long annotation should exist?

```
@Retention(RetentionPolicy.RUNTIME)
```

means:

```
Keep annotation available at runtime.
```

Why?

Because Hibernate Validator reads it during runtime.

---

Without RUNTIME:

```
Annotation disappears
```

Validator can't find it.

Validation won't work.

---

Think:

```
Compile
   |
   v
Runtime
   |
   v
Hibernate Validator reads annotation
```

---

### @Constraint

Most important line.

```
@Constraint(
    validatedBy = EmployeeIdValidator.class
)
```

Meaning:

```
Whenever @ValidEmployeeId is found,
use EmployeeIdValidator
```

Think:

```
@ValidEmployeeId
          |
          v
EmployeeIdValidator
```

Spring now knows:

> Which validator should execute.

----

## PART 2: Create Validator Class

Now create:

```
EmployeeIdValidator.java
```

```
import jakarta.validation.ConstraintValidator;import jakarta.validation.ConstraintValidatorContext;public class EmployeeIdValidator        implements ConstraintValidator<            ValidEmployeeId,            String> {    @Override    public boolean isValid(            String value,            ConstraintValidatorContext context) {        if(value == null)            return false;        return value.startsWith("EMP-");    }}
```

---

## What is ConstraintValidator?

This is the actual validation engine.

```
ConstraintValidator<A,T>

A = Annotation
T = Data Type
```


```
Annotation = ValidEmployeeIdValue Type = String
```

---

### Understanding isValid()

Method:

```
@Override
public boolean isValid(
       String value,
       ConstraintValidatorContext context)
```

Suppose:

```
{
  "employeeId":"EMP-100"
}
```

Then:

```
value = "EMP-100"
```

---

Your logic:

```
return value.startsWith("EMP-");    // return true
```

Validation passes.

---

Request:

```
{
   "employeeId":"ABC-100"
}

// output will be - false
```

Validation fails.

-------

## What is ConstraintValidatorContext ?

Simple definition:

> `ConstraintValidatorContext` allows us to customize validation error messages and control how validation errors are reported.

Think of it as:

```
Validation Logic
       |
       v
ConstraintValidatorContext
       |
       v
Build Error Response
```

Without context:

```
return false;
```

Spring returns:

```
{
   "employeeId":"Invalid Employee ID"
}
```

using the default message from annotation.

### Why Do We Need ConstraintValidatorContext?

Suppose business wants:

```
If value is null
    -> Employee ID cannot be null

If value doesn't start with EMP-
    -> Employee ID must start with EMP-

If length < 10
    -> Employee ID length must be at least 10
```

Different errors for different situations.

Static annotation message is not enough.

This is where `ConstraintValidatorContext` helps.

---

## Example 1: Dynamic Error Message

Validator:

```
public class EmployeeIdValidator
implements ConstraintValidator<
        ValidEmployeeId,
        String> {

    @Override
    public boolean isValid(
            String value,
            ConstraintValidatorContext context) {

        if(value == null) {

            context.disableDefaultConstraintViolation();

            context
                .buildConstraintViolationWithTemplate(
                    "Employee ID cannot be null")
                .addConstraintViolation();

            return false;
        }

        if(!value.startsWith("EMP-")) {

            context.disableDefaultConstraintViolation();

            context
                .buildConstraintViolationWithTemplate(
                    "Employee ID must start with EMP-")
                .addConstraintViolation();

            return false;
        }

        return true;
    }
}
```

---

Request:

```
{
   "employeeId":"ABC-123"
}
```

Response:

```
{
   "employeeId":
   "Employee ID must start with EMP-"
}
```

---

### Understanding the Methods

---

#### disableDefaultConstraintViolation()

By default:

```
@ValidEmployeeId(
   message="Invalid Employee ID"
)
```

Spring uses:

```
Invalid Employee ID
```

message.

We want our custom message.

So:

```
context.disableDefaultConstraintViolation();
```

means:

```
Ignore annotation message.
I will provide my own message.
```

---

#### buildConstraintViolationWithTemplate()

Used to create custom message.

```
context
    .buildConstraintViolationWithTemplate(
       "Employee ID must start with EMP-"
    )
```

Think:

```
Create Validation Error
```

---

#### addConstraintViolation()

Finally register error.

```
.addConstraintViolation();
```

Think:

```
Add error into validation result
```

Without this:

```
buildConstraintViolationWithTemplate(...)
```

does nothing.

---

### Real Life Flow

Request:

```
{
   "employeeId":"ABC123"
}
```

Spring:

```
@ValidEmployeeId
       |
       v
EmployeeIdValidator
       |
       v
isValid()
       |
       v
ConstraintValidatorContext
       |
       v
Custom Error Message
       |
       v
400 Bad Request
```

---

## Example 2: Strong Password Validation

Suppose password rules:

```
Minimum 8 chars
One uppercase
One lowercase
One number
```

Validator:

```
@Override
public boolean isValid(
        String password,
        ConstraintValidatorContext context) {

    if(password == null) {

        context.disableDefaultConstraintViolation();

        context
           .buildConstraintViolationWithTemplate(
             "Password cannot be null")
           .addConstraintViolation();

        return false;
    }

    if(password.length() < 8) {

        context.disableDefaultConstraintViolation();

        context
          .buildConstraintViolationWithTemplate(
             "Password must be at least 8 characters")
          .addConstraintViolation();

        return false;
    }

    return true;
}
```

Now users get meaningful messages.

---

## Interview-Level Understanding

### ConstraintValidator

```
ConstraintValidator<
    Annotation,
    DataType
>
```

contains validation logic.

### ConstraintValidatorContext

```
ConstraintValidatorContext
```

contains metadata about the validation process and allows us to:

1. Disable default error message
2. Create custom error messages
3. Add custom validation violations
4. Control how validation errors are reported

---

## Production Example

Imagine your PIM project.

Feed URL validation:

```
@ValidFeedUrl
private String feedUrl;
```

Validator:

```
@Override
public boolean isValid(
        String url,
        ConstraintValidatorContext context) {

    if(!url.startsWith("https://")) {

        context.disableDefaultConstraintViolation();

        context
            .buildConstraintViolationWithTemplate(
               "Feed URL must use HTTPS")
            .addConstraintViolation();

        return false;
    }

    return true;
}
```

Request:

```
{
   "feedUrl":"http://amazon.com/feed.xml"
}
```

Response:

```
{
   "feedUrl":"Feed URL must use HTTPS"
}
```

This is exactly how `ConstraintValidatorContext` is commonly used in production systems.

### Short Interview Answer

> `ConstraintValidatorContext` is provided inside the `isValid()` method and is used to customize validation errors. By default, validation failures use the message defined in the annotation. Using `ConstraintValidatorContext`, we can disable the default message and create dynamic, business-specific validation messages using `buildConstraintViolationWithTemplate()` and `addConstraintViolation()`.


#
----
----
----

# ==How do you validate nested objects in a request payload? How do you validate collections (e.g., list of objects) in a request body?

## 1. Nested object + collection validation

**Interviewer answer:**

For nested objects, I use `@Valid` on the nested DTO field. This enables cascading validation.

```
public class OrderRequest {

    @NotNull
    private Long orderId;

    @Valid
    @NotNull
    private CustomerRequest customer;
}
```

```
public class CustomerRequest {

    @NotBlank
    private String name;

    @Email
    private String email;
}
```

Controller:

```
@PostMapping("/orders")
public ResponseEntity<?> createOrder(
        @Valid @RequestBody OrderRequest request) {

    return ResponseEntity.ok("Order created");
}
```

So Spring validates `OrderRequest`, then goes inside `CustomerRequest` because of `@Valid`.

For collections, I use `@Valid` on the list element type.

```
public class OrderRequest {

    @NotEmpty
    private List<@Valid ProductRequest> products;
}
```

```
public class ProductRequest {

    @NotBlank
    private String productName;

    @Min(1)
    private Integer quantity;
}
```

This validates every object inside the list.


# ------------------------------------------

# ==How do you handle validation for path variables and query parameters?

For `@PathVariable` and `@RequestParam`, I use validation annotations directly on method parameters and add `@Validated` at controller class level.

```
@RestController
@RequestMapping("/users")
@Validated
public class UserController {

    @GetMapping("/{id}")
    public ResponseEntity<?> getUser(
            @PathVariable
            @Min(1)
            Long id) {

        return ResponseEntity.ok("User found");
    }

    @GetMapping
    public ResponseEntity<?> searchUsers(
            @RequestParam
            @NotBlank
            String email) {

        return ResponseEntity.ok("User searched");
    }
}
```

`@Validated` enables method-level validation for path variables and query parameters.

# 
------------------------------
-----------------------------
--------------------------------
--------------

# ==What is cross-field validation ? How do you implement cross-field validation (e.g., password and confirm password ) ?

## Cross-field validation

Cross-field validation means:

> Validating two or more fields together.

Normal field validation checks one field only:

```
@NotBlank
private String password;
```

But cross-field validation checks relation between fields:

```
password == confirmPassword
```

---

## Why is it needed?

Because some business rules depend on multiple fields.

Examples:

```
password and confirmPassword should match

startDate must be before endDate

discountPrice must be less than actualPrice

minSalary must be less than maxSalary

deliveryDate cannot be before orderDate
```


---

## Example: Password and Confirm Password

Request:

```
{
  "email": "abc@gmail.com",
  "password": "Pass@123",
  "confirmPassword": "Pass@123"
}
```

Valid.

```
{
  "email": "abc@gmail.com",
  "password": "Pass@123",
  "confirmPassword": "Pass@456"
}
```

Invalid.

---

## How to Implement Cross-Field Validation

For cross-field validation, we put custom annotation on the **class**, not field.

Why?

Because validator needs access to both fields.

```
@PasswordMatches
public class RegisterRequest {}
```

---

### Step 1: Create Custom Annotation

```
import jakarta.validation.Constraint;
import jakarta.validation.Payload;

import java.lang.annotation.*;

@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Constraint(validatedBy = PasswordMatchesValidator.class)
public @interface PasswordMatches {

    String message() default "Password and confirm password do not match";

    Class<?>[] groups() default {};

    Class<? extends Payload>[] payload() default {};
}
```

Important part:

```
@Target(ElementType.TYPE)
```

Means this annotation is used on class level.

---

### Step 2: Create DTO

```
@PasswordMatches
public class RegisterRequest {

    @NotBlank(message = "Email is required")
    @Email(message = "Invalid email")
    private String email;

    @NotBlank(message = "Password is required")
    private String password;

    @NotBlank(message = "Confirm password is required")
    private String confirmPassword;

    public String getEmail() {
        return email;
    }

    public String getPassword() {
        return password;
    }

    public String getConfirmPassword() {
        return confirmPassword;
    }
}
```

---

### Step 3: Create Validator

```
import jakarta.validation.ConstraintValidator;
import jakarta.validation.ConstraintValidatorContext;

public class PasswordMatchesValidator
        implements ConstraintValidator<PasswordMatches, RegisterRequest> {

    @Override
    public boolean isValid(
            RegisterRequest request,
            ConstraintValidatorContext context) {

        if (request == null) {
            return true;
        }

        if (request.getPassword() == null ||
            request.getConfirmPassword() == null) {
            return true;
        }

        boolean matches =
                request.getPassword()
                       .equals(request.getConfirmPassword());

        if (!matches) {
            context.disableDefaultConstraintViolation();

            context.buildConstraintViolationWithTemplate(
                    "Confirm password must match password")
                   .addPropertyNode("confirmPassword")
                   .addConstraintViolation();
        }

        return matches;
    }
}
```

---

### Step 4: Use in Controller

```
@PostMapping("/register")
public ResponseEntity<String> register(
        @Valid @RequestBody RegisterRequest request) {

    return ResponseEntity.ok("User registered");
}
```

---

## Why addPropertyNode("confirmPassword")?

Without this:

```
context.buildConstraintViolationWithTemplate(...)
       .addConstraintViolation();
```

error may come at object level:

```
{
  "registerRequest": "Password and confirm password do not match"
}
```

With:

```
.addPropertyNode("confirmPassword")
```

error points to field:

```
{
  "confirmPassword": "Confirm password must match password"
}
```

This is better for frontend.

---

# Interview answer

>Cross-field validation means validating multiple fields together when the rule depends on their relationship. For example, password and confirmPassword should match, startDate should be before endDate, or discountPrice should be less than actualPrice.

>To implement it, I create a custom class-level annotation using `@Target(ElementType.TYPE)`, connect it with a validator using `@Constraint`, and implement `ConstraintValidator<Annotation, DTOClass>`. Inside `isValid()`, I compare the required fields. For password confirmation, I compare `request.getPassword()` and `request.getConfirmPassword()`. If they do not match, I return false or use `ConstraintValidatorContext` to attach the error to `confirmPassword`.


# ==## Validation Groups 

do it later

# ==How do you customize the MethodValidationPostProcessor for advanced use cases?

do it later


# ==## Validation in the Service Layer  

do it later