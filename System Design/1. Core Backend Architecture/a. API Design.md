
# Design REST API 
	Pagination - Already covered
	Filtering
	Sorting
	Versioning


## Filtering

Filtering means

> Return only the records matching certain conditions.

Without filtering:

```
GET /products
```

returns

```
1. iPhone 16
2. Samsung S26
3. Nike Shoes
4. Adidas Shoes
5. MacBook
6. Dell Laptop
...
100000 products
```

Client doesn't want all products.

Instead they want

> Only Nike Shoes under ₹5000.

Filtering does exactly that.

---

### Real World Example (Amazon)

Suppose you search

```
Shoes
```

Amazon shows filters

```
Brand
✓ Nike

Price
₹2000 - ₹5000

Rating
4★

Size
9

Color
Black
```

The frontend sends

```
GET /products?
category=shoes
&brand=Nike
&minPrice=2000
&maxPrice=5000
&rating=4
&color=black
&size=9
```

Backend generates

```
SELECT *
FROM products
WHERE category='shoes'
AND brand='Nike'
AND price BETWEEN 2000 AND 5000
AND rating >= 4
AND color='black'
AND size=9;
```

Only matching rows are returned.

---
#### Why Filtering is Needed

Imagine Instagram.

Without filtering

```
GET /posts
```

returns every post.

But user wants

```
My posts
```

API

```
GET /posts?userId=100
```

or

```
GET /users/100/posts
```

Now only one user's posts are returned.

---

### [ Filtering Best Practices ]

Good

```
GET /products?brand=Apple
```

Bad

```
POST /filterProducts
```

	Filtering is a read operation.

	Always use GET.

---

Don't create different APIs

Bad

```
GET /products/apple

GET /products/nike

GET /products/adidas
```

Better

```
GET /products?brand=Apple

GET /products?brand=Nike
```

One endpoint.

Many filters.

---

#### Spring Boot Example

Controller

```
@GetMapping("/products")
public List<Product> getProducts(
        @RequestParam(required = false) String brand,
        @RequestParam(required = false) String category,
        @RequestParam(required = false) Integer minPrice,
        @RequestParam(required = false) Integer maxPrice) {

    return productService.getProducts(
        brand,
        category,
        minPrice,
        maxPrice
    );
}
```

Notice

```
required=false
```

because every filter is optional.

---

## Sorting

Filtering decides

"What records?"

Sorting decides

"In what order?"

---

Suppose Amazon returns

```
Nike Shoes

₹3000

₹4500

₹2500

₹1000
```

Customer wants

Lowest price first.

---

Request

```
GET /products?sort=price
```

Backend

```
ORDER BY price ASC
```

Result

```
1000

2500

3000

4500
```

---

### Real World Examples

Amazon

```
Price Low → High

Price High → Low

Newest

Best Selling

Popularity
```

---

LinkedIn

```
Newest Jobs

Most Relevant
```

---

Instagram

```
Latest Posts
```

---

YouTube

```
Most Viewed

Latest

Trending
```

---

### Ascending

```
GET /products?sort=price
```

or

```
sort=price,asc
```

SQL

```
ORDER BY price ASC
```

---

### Descending

```
GET /products?sort=price,desc
```

SQL

```
ORDER BY price DESC
```

---

### Sorting by Date

```
GET /posts?sort=createdAt,desc
```

Newest first.

---

### Multiple Sorting

Suppose products have same rating.

Sort by rating

then price.

```
GET /products?sort=rating,desc&sort=price,asc
```

SQL

```
ORDER BY
rating DESC,
price ASC
```

Example

```
Product A
Rating 5
Price 4000

Product B
Rating 5
Price 3000

Product C
Rating 4
Price 1000
```

Result

```
B
A
C
```

Same rating

Lower price first.

---

### Spring Boot Example

Spring Data already supports this.

```
GET /products?sort=price,asc
```

Controller

```
@GetMapping
public Page<Product> getProducts(Pageable pageable) {
    return repository.findAll(pageable);
}
```

Spring automatically understands

```
sort=price,asc
```

Very common in interviews.

---

### Filtering + Sorting Together

Amazon example

```
GET /products?

category=shoe
&brand=Nike
&minPrice=2000
&maxPrice=5000
&sort=price,asc
```

### SQL

```
SELECT *
FROM products

WHERE
category='shoe'
AND brand='Nike'
AND price BETWEEN 2000 AND 5000

ORDER BY price ASC
LIMIT 20;
```

---

## Versioning

This is one of the most frequently asked API design topics.

---

### Why Versioning Exists

Imagine you launched

```
GET /users/10
```

Response

```
{
  "id":10,
  "name":"John"
}
```

Thousands of mobile apps are using this.

Now you change it to

```
{
  "id":10,
  "firstName":"John",
  "lastName":"Doe"
}
```

Old mobile app

expects

```
name
```

Now

```
name
```

doesn't exist.

App crashes.

This is called

**Breaking Change.**

---

### Versioning Prevents This

Instead of changing old API,

create new version.

Old

```
/api/v1/users/10
```

New

```
/api/v2/users/10
```

Both continue working.

---

#### Real World Example

Mobile apps.

A user may not update the app for months.

Old app

↓

Calls

```
v1
```

New app

↓

Calls

```
v2
```

Backend supports both.

No one breaks.

---

#### URI Versioning (Most Popular)

```
/api/v1/products

/api/v2/products
```

Most companies use this.

Easy.

Readable.

Interview favorite.


---

### When Should Version Increase?

### Breaking Change

Increase version.

Example

```
name

↓

firstName
lastName
```

Need new version.

---

#### Removing field

```
email
```

deleted.

Breaking.

---

#### Changing datatype

```
age

25

↓

"25"
```

Breaking.

---

#### Changing response structure

Old

```
{
    "user": {}
}
```

New

```
{
    "data": {
		 "user": {}
    }
}
```

Breaking.

---

### Real Company Example

Suppose your food delivery API

Version 1

```
GET /api/v1/restaurants
```

Response

```
{
    "name":"Burger King",
    "rating":4.4
}
```

Later business wants

```
location
deliveryTime
offers
```

Adding these optional fields?

No version.

---

But business decides

```
rating

↓

reviews
```

Now

```
rating
```

removed.

Need

```
/api/v2/restaurants
```

---

# Version Lifecycle

A mature API typically follows this process:

```
Release v1
      │
      ▼
Clients adopt v1
      │
      ▼
Need breaking changes
      │
      ▼
Release v2
      │
      ▼
Support v1 for a deprecation period
      │
      ▼
Notify clients to migrate
      │
      ▼
Retire v1
```

This avoids disrupting existing consumers while allowing the API to evolve.

---

### Complete Example

Imagine an e-commerce product API.

```
GET /api/v1/products?
category=electronics
&brand=Apple
&minPrice=50000
&maxPrice=150000
&sort=price,asc
&page=0
&size=20
```


Result:

```
{
  "content": [
    {
      "id": 101,
      "name": "iPhone 16",
      "price": 79999
    },
    {
      "id": 102,
      "name": "iPhone 16 Plus",
      "price": 89999
    }
  ],
  "page": 0,
  "size": 20,
  "totalElements": 157
}
```

---

## Interview Takeaways

- **Filtering** limits _which_ resources are returned (e.g., `brand=Apple`, `minPrice=1000`).
- **Sorting** determines _the order_ of the returned resources (e.g., `sort=price,asc` or `sort=createdAt,desc`).
- **Filtering → Sorting → Pagination** is the typical processing sequence because you first narrow the dataset, then order it, and finally return the requested page.
- **Versioning** exists to support API evolution without breaking existing clients. Introduce a new version only for **breaking changes**; additive, backward-compatible changes generally do not require a new version.