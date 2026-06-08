
# What is Spring Security?

**Spring Security** is a security framework for Java and Spring applications that handles:

- Authentication (Who are you?)
- Authorization (What are you allowed to do?)
- Protection against common attacks
    - CSRF
    - Session Fixation
    - Clickjacking
    - XSS-related protections through headers
- Password encryption
- JWT authentication
- OAuth2 login (Google, GitHub, etc.)

Think of it as a **security guard sitting in front of every request**.

---

## Real-Life Example

Imagine you're entering an office building.

### Step 1: Security Guard Checks Identity

You show your ID card.

This is:

**Authentication**

"Are you really Prashant?"

---

### Step 2: Check Permissions

After verifying identity, security checks:

- Employee → allowed
- Visitor → limited access
- Admin → full access

This is:

**Authorization**

"What can Prashant access?"

---

Spring Security does exactly this for your APIs.

---

## Without Spring Security

Suppose you have:

```
@RestController
@RequestMapping("/api")
public class UserController {

    @GetMapping("/profile")
    public String profile() {
        return "User Profile";
    }
}
```

Anyone can access:

```
GET /api/profile
```

No security.

---

## With Spring Security

Spring Security sits before your controller.

Request flow:

```
Client
  |
  v
Spring Security
  |
  v
Controller
```

Every request must pass through Spring Security first.

---

## Internal Architecture

Most developers know:

```
http.authorizeHttpRequests(...)
```

But behind the scenes there is much more.

---

## Core Components

```
Request
   |
   v
Security Filter Chain
   |
   +--> Authentication Filter
   |
   +--> Authorization Filter
   |
   +--> Exception Filter
   |
   +--> CSRF Filter
   |
   +--> Session Filter
   |
   v
Controller
```

The most important concept is:

## Security Filter Chain

---

## What is a Filter?

A filter intercepts every request before it reaches controller.


```
Browser
   |
GET /users
   |
   v
Filter
   |
Controller
```

Filter can:

- Allow request
- Reject request
- Modify request

---

## SecurityFilterChain

Spring Security registers many filters.

```
Request
   |
   v
Filter 1
   |
Filter 2
   |
Filter 3
   |
Filter 4
   |
Controller
```

This chain is called:

**SecurityFilterChain**

---

## How Request Travels

Suppose:

```
GET /api/profile
```

Request flow:

```
Client
  |
  v
Tomcat
  |
  v
DelegatingFilterProxy
  |
  v
Spring Security Filter Chain
  |
  v
Controller
```

---

## Important Interview Question

## What is DelegatingFilterProxy?

Servlet containers like:

- Tomcat
- Jetty

understand only Servlet Filters.

But Spring Security filters are Spring Beans.

So Spring creates:

```
DelegatingFilterProxy
```

which connects:

```
Tomcat
   |
DelegatingFilterProxy
   |
Spring Security Filters
```

---

## Internal Flow

```
Client Request
      |
      v
Tomcat
      |
      v
DelegatingFilterProxy
      |
      v
FilterChainProxy
      |
      v
SecurityFilterChain
      |
      v
Controller
```

Interviewers love this flow.

---

## FilterChainProxy

This is the actual entry point of Spring Security.

```
DelegatingFilterProxy
          |
          v
FilterChainProxy
```

It manages all security filters.

---

## Authentication Flow

Suppose:

```
POST /login
```

with:

```
{
  "username":"prashant",
  "password":"123"
}
```

---

### Step 1

Authentication Filter intercepts request.

```
UsernamePasswordAuthenticationFilter
```

Extracts:

```
username
password
```

---

### Step 2

Creates:

```
UsernamePasswordAuthenticationToken
```

Internally:

```
new UsernamePasswordAuthenticationToken(
    username,
    password
);
```

---
### Step 3

Calls:

```
AuthenticationManager
```

```
AuthenticationFilter
      |
      v
AuthenticationManager
```

---

## AuthenticationManager

Main job:

```
Authenticate User
```

```
authenticationManager.authenticate(token);
```

---

## AuthenticationProvider

AuthenticationManager delegates work.

```
AuthenticationManager
        |
        v
AuthenticationProvider
```

Think:

```
Manager delegates to Specialist
```

---

## DaoAuthenticationProvider

Most commonly used provider.

```
DaoAuthenticationProvider
```

Responsibilities:

1. Load User
2. Verify Password

---

Flow:

```
AuthenticationManager
      |
      v
DaoAuthenticationProvider
```

---

## UserDetailsService

Provider now asks:

```
Fetch User From DB
```

Using:

```
UserDetailsService
```

Example:

```
public interface UserDetailsService {

    UserDetails loadUserByUsername(
        String username
    );
}
```

---

Database:

```
users
--------------------
prashant
admin
john
```

---

## Password Verification

Stored password:

```
$2a$10$abcxyz...
```

(Bcrypt)

User enters:

```
123456
```

Provider calls:

```
passwordEncoder.matches(
    rawPassword,
    encodedPassword
);
```

---

## PasswordEncoder

Example:

```
BCryptPasswordEncoder
```

```
passwordEncoder.matches(
   "123456",
   hash
);
```

Never compare plain text passwords.

---

## Authentication Success

Spring creates:

```
Authentication auth
```

Containing:

```
Username
Roles
Permissions
```

Example:

```
User = Prashant

Roles:
ROLE_USER
ROLE_ADMIN
```

---

## SecurityContext

Authentication is stored in:

```
SecurityContext
```

```
SecurityContextHolder
       |
       v
SecurityContext
       |
       v
Authentication
```

---

Current logged-in user:

```
SecurityContextHolder
    .getContext()
    .getAuthentication();
```

---

## Authorization Flow

After authentication:

```
User Authenticated
      |
      v
Authorization Check
```

---

Example:

```
@GetMapping("/admin")
```

Configuration:

```
.requestMatchers("/admin/**")
.hasRole("ADMIN")
```

---

Spring checks:

```
Does user have ROLE_ADMIN?
```

If yes:

```
Allow
```

Else:

```
403 Forbidden
```

---

## Complete Request Flow

```
Request
   |
   v
SecurityFilterChain
   |
Authentication Filter
   |
AuthenticationManager
   |
AuthenticationProvider
   |
UserDetailsService
   |
PasswordEncoder
   |
SecurityContext
   |
Authorization Filter
   |
Controller
```

This is the most important flow to remember.

---

## SecurityContextHolder Internals

By default:

```
ThreadLocal
```

is used.

```
Thread
   |
SecurityContext
```

Meaning:

Each request thread stores its own logged-in user.

---

Example:

```
Authentication auth =
SecurityContextHolder
    .getContext()
    .getAuthentication();
```

Spring retrieves user from ThreadLocal.

---

## Example Security Configuration

```
@Configuration
@EnableWebSecurity
public class SecurityConfig {

    @Bean
    SecurityFilterChain securityFilterChain(
            HttpSecurity http)
            throws Exception {

        return http
            .csrf(csrf -> csrf.disable())
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/public/**")
                .permitAll()
                .requestMatchers("/admin/**")
                .hasRole("ADMIN")
                .anyRequest()
                .authenticated()
            )
            .build();
    }
}
```

---

## Production Example

Suppose you build an e-commerce application.

### APIs

```
POST /login

GET /products

POST /orders

DELETE /users
```

Rules:

```
/products      -> public

/orders        -> logged-in user

/users/delete  -> admin only
```

Spring Security automatically:

1. Intercepts request
2. Authenticates user
3. Checks role
4. Allows/denies request

without writing security code in every controller.

---

## Interview Answer (2-Min Version)

**Spring Security is a framework that provides authentication and authorization for Spring applications. Internally, every request first passes through the SecurityFilterChain. Authentication filters extract credentials and delegate them to AuthenticationManager, which uses AuthenticationProvider and UserDetailsService to validate users. On successful authentication, an Authentication object is stored inside SecurityContextHolder (ThreadLocal). Authorization filters then verify roles and permissions before allowing access to controllers.**

---

## Remember This Architecture

```
Client
  |
Tomcat
  |
DelegatingFilterProxy
  |
FilterChainProxy
  |
SecurityFilterChain
  |
Authentication Filter
  |
AuthenticationManager
  |
AuthenticationProvider
  |
UserDetailsService
  |
PasswordEncoder
  |
SecurityContextHolder
  |
Authorization Filter
  |
Controller
```


---
---
---
---
---


