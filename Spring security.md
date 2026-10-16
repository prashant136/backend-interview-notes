It helps protect api , web apps and microservices from unathuorized access, attacks and securoty threads.

Authentication - Authentication is the process of veryfing who you are. it grants the access.
Authorization - determines what you can do after authentication (what they are allow to access).

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</doxependency>

// isko pom.xml me add krne pr spring authomatically ek secuirty layer lga deta hai, ab hum url pe hit karenge toh login page dikhega. (but humne ab tak koi security configuration add nhi kiya hai)

// Architecture -
[ Client ]
    │
    ▼
┌───────────────────────────────────────┐
│               FILTERS                 │
│  ┌─────────────────────────────────┐  │
│  │          LoggingFilter          │  │
│  └────────────────┬────────────────┘  │
│                   │                   │
│                   ▼                   │
│  ┌─────────────────────────────────┐  │
│  │      AuthenticationFilter       │  │ 
│  └────────────────┬────────────────┘  │       
│                   │                   │
│                   ▼                   │
│  ┌─────────────────────────────────┐  │
│  │             Filter3             │  │
│  └────────────────┬────────────────┘  │
└───────────────────┼───────────────────┘
                    │ filters chain (filters are also part of servlet API)
                    ▼
┌─────────────────────────────────────────────────────────┐
│                   APPLICATION                           │
│          Dispatcher sarvlet and controller              │
│                                                         │
└─────────────────────────────────────────────────────────┘


// jab hum spring-boot-starter-security dep add krte hai toh filter chain me SecurityFilterChain add hota hai.

DelegatingFilterProxy that allows briding b/w servlet container's lifecycle and Spring ApplicationContext. it delegate all work to spring bean that implement filter.

SecurityFilterChain = Security ka main entry point. Ye Servlet Filters ka ek group (chain) hota hai. Har ek request application tak pahunchne se pehle is chain se hokar guzarti hai. Isme alag-alag filters hote hain jo check karte hain ki request authenticated hai ya nahi, CORS/CSRF headers sahi hain ya nahi.

AuthenticationManage = interface 

AuthenticationProvider = AuthenticationManage ko implement krta h. (actual authentication logic execute)

Ek application mein multiple providers ho sakte hain (jaise: ek Database login ke liye, ek OAuth/Google login ke liye)

UserDetailsService = username ke basis par User details (Username, Password, Roles/Authorities) ko fetch karke `UserDetails` object k form me rkhta hai.
fn - loadUserByUsername(String username)

PasswordEncoder = Ye passwords ko securely encode (hash) karne aur incoming plain-text password ko stored hashed password se compare karne ke kaam aata hai.
BCryptPasswordEncoder

SecurityContext = container hai jisme logged-in user ki authentication details (Authentication object) store rehti hain.

ExceptionTranslationFilter = Ye SecurityFilterChain mein security exceptions catch karta hai aur unhe HTTP status responses mein convert karta hai.
```

```
1. define SecurityConfig -> jisme hum security se related configuration karenge. @EnableWebSecurity annotate karenge.
```
 




