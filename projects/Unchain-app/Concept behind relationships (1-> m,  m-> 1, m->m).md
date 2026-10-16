
This is the code.
### Database Design


```
One user can select many goals.
One goal can be selected by many users.
```

---

### Update User Entity

`user/User.java`

```
package com.unchain.user;  
  
import jakarta.persistence.*;  
import lombok.*;  
  
import java.time.LocalDateTime;  
  
@Entity  
@Table(  
        name = "users",  
        uniqueConstraints = {  
                @UniqueConstraint(name = "uk_users_email", columnNames = "email")  
        }  
)  
@Getter  
@Setter  
@NoArgsConstructor  
@AllArgsConstructor  
@Builder  
public class User {  
  
    @Id  
    @GeneratedValue(strategy = GenerationType.IDENTITY)  
    private Long id;  
  
    @Column(nullable = false)  
    private String name;  
  
    @Column(nullable = false, length = 150)  
    private String email;  
  
    @Column(nullable = false)  
    private String password;  
  
    @Enumerated(EnumType.STRING)  
    @Column(nullable = false, length = 30)  
    private Role role;  
  
    @Column(nullable = false)  
    private Boolean isActive;  
  
    @Column(nullable = false)  
    private Boolean onboardingCompleted;  
  
    @Column(nullable = false, updatable = false)  
    private LocalDateTime createdAt;  
  
    private LocalDateTime updatedAt;  
  
    @PrePersist  
    void onCreate() {  
        this.createdAt = LocalDateTime.now();  
//        this.isActive = true;  
        this.onboardingCompleted = false;  
    }  
  
    @PreUpdate  
    void onUpdate() {  
        this.updatedAt = LocalDateTime.now();  
    }  
}
```

---

### GoalOption Entity

`onboarding/GoalOption.java`

```
package com.unchain.onboarding;

import jakarta.persistence.*;
import lombok.*;

@Entity
@Table(name = "goal_options")
@Getter
@Setter
@NoArgsConstructor
@AllArgsConstructor
@Builder
public class GoalOption {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false, unique = true)
    private String name;

    @Column(nullable = false, unique = true)
    private String code;

    @Column(nullable = false)
    private Boolean active;
}
```

Example data:

```
WAKE_UP_EARLY
GYM
DSA
CODING
READING
MEDITATION
SLEEP_EARLY
```

---

### UserGoal Entity

`onboarding/UserGoal.java`

```
package com.unchain.onboarding;

import com.unchain.user.User;
import jakarta.persistence.*;
import lombok.*;

import java.time.LocalDateTime;

@Entity
@Table(
        name = "user_goals",
        uniqueConstraints = {
                @UniqueConstraint(
                        name = "uk_user_goal",
                        columnNames = {"user_id", "goal_option_id"}
                )
        }
)
@Getter
@Setter
@NoArgsConstructor
@AllArgsConstructor
@Builder
public class UserGoal {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @ManyToOne(fetch = FetchType.LAZY, optional = false)
    @JoinColumn(name = "user_id", nullable = false)
    private User user;

    @ManyToOne(fetch = FetchType.LAZY, optional = false)
    @JoinColumn(name = "goal_option_id", nullable = false)
    private GoalOption goalOption;

    @Column(nullable = false)
    private LocalDateTime createdAt;

    @PrePersist
    void onCreate() {
        this.createdAt = LocalDateTime.now();
    }
}
```

Let's build the intuition first before touching annotations.

### Step 1: Think Like a Product Owner

Suppose Unchain asks a user:

> "What goals do you want to build?"

Options:

- Wake Up Early
- Gym
- DSA
- Coding
- Reading
- Meditation

Now imagine 3 users.

|User|Goals|
|---|---|
|Prashant|Gym, DSA, Coding|
|Rahul|Gym, Reading|
|Amit|Coding, Meditation|

Question:

Should we store goal names directly inside user table?

```
users
------------------------
id
name
email
goal1
goal2
goal3
```

Example:

```
1 | Prashant | Gym | DSA | Coding
```

Problem:

- How many goals can a user select?
- What if tomorrow user selects 10 goals?
- What if we rename "Gym" to "Workout"?

Bad design.

---

### Step 2: Separate Master Data

Instead create a table containing all possible goals.

## goal_options

```
id | name
---------------
1  | Gym
2  | DSA
3  | Coding
4  | Reading
5  | Meditation
```

This table is called:

> Master Table  
> Lookup Table  
> Reference Table

Meaning:

"These are all valid goals in the system."

Think:

```
goal_options
   ↓
catalog of goals
```

Just like Amazon has:

```
product_catalog
```

before customers buy products.

---

### Step 3: Now Store User Selections

Prashant selects:

```
Gym
DSA
Coding
```

We don't copy goal names.

Instead:

```
user_goals

user_id | goal_option_id
-------------------------
1       | 1
1       | 2
1       | 3
```

Meaning:

```
User 1 selected Goal 1
User 1 selected Goal 2
User 1 selected Goal 3
```

Very clean.

---

### Final Structure

## users

```
id
name
email
onboarding_completed
```

## goal_options

```
id
name
code
```

## user_goals

```
id
user_id
goal_option_id
```

Visual:

```
          users
            |
            |
            |
            V

        user_goals

            ^
            |
            |
            |

      goal_options
```

`user_goals` is the bridge table.

---

### Step 4: Why 3 Tables?

Because reality says:

#### One User → Many Goals

Prashant:

```
Gym
DSA
Coding
```

3 goals.

So:

```
User → Many Goals
```

---

#### One Goal → Many Users

Gym can be selected by:

```
Prashant
Rahul
Amit
```

So:

```
Goal → Many Users
```

---

Therefore:

```
User ↔ Goal

Many ↔ Many
```

Database cannot directly store Many-to-Many efficiently.

This converts:

```
Many-to-Many
```

into

```
One-to-Many + Many-to-One
```

which databases understand very well.

---

### How to Determine Relationship in Real Life?

Ask these 2 questions:

#### Question 1

Can one A have multiple B?

Example:

```
One User → many Goals ?
```

Yes.

Then:

```
User -> OneToMany -> Goal
```

---

#### Question 2

Can one B belong to multiple A?

Example:

```
One Goal -> many Users ?
```

Yes.

Then:

```
Goal -> OneToMany -> User
```

Since BOTH are many:

```
Many User ↔ Many Goal
```

Use join table.

---

### Why UserGoal Entity Exists?

Without UserGoal:

```
User ---- Goal
```

No place to store extra information.

But later you may need:

```
When user selected goal?
Priority?
Target streak?
Status?
```

Example:

```
user_goals

id
user_id
goal_option_id
selected_at
priority
target_days
```

Now you can store goal-specific data.

That's why production systems usually create a separate entity.

---

### Understanding the JPA Mapping

#### UserGoal

```
@ManyToOne(fetch = FetchType.LAZY)
@JoinColumn(name = "user_id")
private User user;
```

Meaning:

Many rows in `user_goals`

can belong to

One user.

Example:

```
user_goals

id | user_id | goal_id
----------------------
1  | 1       | 1
2  | 1       | 2
3  | 1       | 3
```

All belong to:

```
users

id=1
Prashant
```

So:

```
Many UserGoal
        ↓
      One User
```

Hence:

```
@ManyToOne
```

---

#### GoalOption Mapping

```
@ManyToOne(fetch = FetchType.LAZY)
@JoinColumn(name = "goal_option_id")
private GoalOption goalOption;
```

Example:

```
user_goals

1 | user=1 | goal=1
2 | user=2 | goal=1
3 | user=3 | goal=1
```

All point to:

```
goal_options

id=1
Gym
```

Therefore:

```
Many UserGoal
        ↓
    One Goal
```

Hence:

```
@ManyToOne
```

---

### Golden Rule for Interviews

Never think in terms of annotations first.

Think:

```
Real World Relationship
       ↓
Database Tables
       ↓
Foreign Keys
       ↓
JPA Annotations
```

For every new feature, ask:

#### Can one A have many B?

#### Can one B have many A?

Then:

|Answer|Relationship|
|---|---|
|One A → One B|OneToOne|
|One A → Many B|OneToMany|
|Many A → One B|ManyToOne|
|Many A ↔ Many B|Join Table (usually)|

## What does @JoinColumn mean?

```
@ManyToOne
@JoinColumn(name = "user_id")
private User user;
```

Means:

> The `user_goals` table contains a foreign key column named `user_id`.