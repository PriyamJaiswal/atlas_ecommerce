# simple-ecom-mongo

Overview
--------
`simple-ecom-mongo` is a minimal Spring Boot backend that demonstrates a product catalog API backed by MongoDB Atlas. It supports product CRUD, image upload/download, and keyword search. Images are stored as `byte[]` in documents (good for small images). For large files, consider GridFS (see Notes).

Features
--------
- Product create/read/update/delete
- Multipart image upload and download
- Case-insensitive keyword search across name/description/brand/category
- Uses Spring Data MongoDB and MongoDB Atlas (configure via `spring.data.mongodb.uri`)

Prerequisites
-------------
- Java 21
- Maven (the project includes the Maven wrapper `mvnw`)
- A MongoDB Atlas cluster (or a MongoDB server reachable from your machine)

Quick setup
-----------
1. Open `src/main/resources/application.properties` and set your Atlas URI (example):

```ini
spring.data.mongodb.uri=mongodb+srv://<username>:<password>@cluster0.xyz.mongodb.net/my_ecom_db
# optionally
# spring.data.mongodb.database=my_ecom_db
```

2. Build and run (from project root). If your path contains spaces quote it:

```bash
cd "/path/to/first_Id 2"
./mvnw -DskipTests package
./mvnw spring-boot:run
```

API (endpoints)
----------------
Base URL: `http://localhost:8090/api` (port configured in `application.properties`)

- GET `/products`
  - Returns list of products.

- GET `/product/{id}`
  - Returns a single product by id (Mongo ObjectId string).

- GET `/product/{id}/image`
  - Returns image bytes with `Content-Type` set to the image MIME type. Example to download:

```bash
curl -s http://localhost:8090/api/product/<id>/image --output product-image.jpg
```

- POST `/product` (multipart/form-data)
  - Upload a product JSON + image file. Example curl:

```bash
curl -X POST http://localhost:8090/api/product \
  -F 'product={"name":"T-Shirt","description":"Soft cotton","brand":"Acme","price":19.99,"category":"clothing","releaseDate":"2024-09-01","productAvailable":true,"stockQuantity":100};type=application/json' \
  -F "imageFile=@/path/to/image.jpg"
```

Notes on the product JSON
- Fields expected (example):

```json
{
  "name": "T-Shirt",
  "description": "Soft cotton T-shirt",
  "brand": "Acme",
  "price": 19.99,
  "category": "clothing",
  "releaseDate": "2024-09-01",
  "productAvailable": true,
  "stockQuantity": 100
}
```

- PUT `/product/{id}` works similarly (multipart with `product` part and `imageFile`).
- DELETE `/product/{id}` deletes the product.
- GET `/product/search?keyword=...` performs a case-insensitive regex search across name/description/brand/category.

Development notes & troubleshooting
----------------------------------
- Line endings: This repo includes a `.gitattributes` file to normalize line endings. If you see warnings like "LF will be replaced by CRLF" run the following before committing:

```bash
git config --global core.autocrlf input   # on Linux / macOS
git add --renormalize .
git commit -m "Normalize line endings"
```

- Image storage: images are stored as `byte[]` in the `Product` document. For large files or many images, migrate to GridFS and store only references in the document.
- ID type: The `Product.id` field is a `String` that stores the Mongo ObjectId value. Endpoints expect id as a string.
- Dependency warnings: a static analysis may flag transitive dependency advisories from Spring Boot starter dependencies. To address those, update Spring Boot and/or specific dependencies.

Folder structure (important files)
- `src/main/java/com/my_ProdId/Model/Product.java` ? Mongo document model
- `src/main/java/com/my_ProdId/Repo/productRepo.java` ? `MongoRepository` + search query
- `src/main/java/com/my_ProdId/Service/ProductService.java` ? business logic
- `src/main/java/com/my_ProdId/controller/Controller.java` ? REST API
- `src/main/resources/application.properties` ? set `spring.data.mongodb.uri`

Testing the API quickly
-----------------------
- Create product (use the POST curl example above).
- List products: `curl http://localhost:8090/api/products | jq` (if `jq` installed)
- Search: `curl "http://localhost:8090/api/product/search?keyword=shirt" | jq`

Future improvements (ideas)
--------------------------
- Move image storage to GridFS and store only file id in documents.
- Add pagination to listing and search endpoint.
- Add authentication/authorization (Spring Security + JWT).
- Add OpenAPI / Swagger documentation.
