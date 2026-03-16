## Yelp Camp Web Application

Yelp Camp is a **3‑tier full‑stack web application** where users can create, browse, review, and rate campgrounds based on location.  
It is inspired by *“The Web Developer Bootcamp”* by Colt Steele, with additional improvements, bug fixes, and a containerized deployment using Docker.

### Key Features
- **Campground management**: Create, edit, and delete campgrounds with descriptions, prices, and locations.
- **User authentication**: Sign up, log in, and manage your own campgrounds and reviews using **Passport (local strategy)**.
- **Reviews and ratings**: Leave reviews and ratings for any campground.
- **Interactive maps**: View campgrounds on an interactive map powered by **Mapbox**, with clustering for better UX.
- **Image uploads**: Upload and store images in the cloud via **Cloudinary**.
- **Secure configuration**: Secrets are stored in environment variables using **dotenv**, and **Helmet** plus sanitization libraries harden security.
- **Dockerized deployment**: Run the entire Node.js app easily via `docker-compose`.

### Tech Stack
- **Backend**: Node.js, Express
- **Frontend**: EJS templates, Bootstrap
- **Database**: MongoDB Atlas (hosted MongoDB)
- **Authentication**: Passport, passport-local, passport-local-mongoose
- **Storage & Media**: Cloudinary, multer, multer-storage-cloudinary
- **Maps & Geocoding**: @mapbox/mapbox-sdk, Mapbox
- **Security & Validation**: Helmet, express-mongo-sanitize, sanitize-html, Joi
- **Session & Flash**: express-session, connect-flash, connect-mongo
- **Tooling**: dotenv, nodemon (for local development), Docker & Docker Compose

---

## 1. Prerequisites

Before running the application, make sure you have:

- **Node.js 18+** and **npm** (for local, non-Docker runs)
- **Docker** and **Docker Compose** (for containerized runs)
- Accounts and API keys for:
  - **MongoDB Atlas** (database)
  - **Cloudinary** (image storage)
  - **Mapbox** (maps)

---

## 2. Environment Configuration

The application expects a `.env` file in the **project root** (same folder as `app.js`) with the following variables:

```sh
CLOUDINARY_CLOUD_NAME=[Your Cloudinary Cloud Name]
CLOUDINARY_KEY=[Your Cloudinary Key]
CLOUDINARY_SECRET=[Your Cloudinary Secret]
MAPBOX_TOKEN=[Your Mapbox Token]
DB_URL=[Your MongoDB Atlas Connection URL]
SECRET=[Your Chosen Secret Key] # This can be any value you prefer
```

> **Note**: Never commit your real secrets to version control. Use example env files or CI/CD secret stores instead.

---

## 3. Running the App with Docker (Recommended)

This project includes a `Dockerfile` and `docker-compose.yml` for easy startup.

1. **Create the `.env` file** as described above.
2. From the project root, build and start the containers:

   ```sh
   docker compose up --build
   ```

3. Once the container is running, open your browser and navigate to:

   ```
   http://localhost:3000
   ```

4. Stop the containers when you are done:

   ```sh
   docker compose down
   ```

---

## 4. Running the App Locally (Without Docker)

If you prefer to run the app directly on your machine:

1. **Install dependencies**:

   ```sh
   npm install
   ```

2. **Create the `.env` file** as described in the configuration section.
3. **Start the application**:

   ```sh
   npm start
   ```

   By default, the server will start on port **3000**:

   ```
   http://localhost:3000
   ```

> You can also use `nodemon` during development (if configured in your scripts) to enable live reloads.

---

## 5. Project Structure (High Level)

While file names may vary, the project generally follows this layout:

- `app.js` – Main Express application entry point.
- `models/` – Mongoose models (e.g., `Campground`, `Review`, `User`).
- `routes/` – Express route handlers for campgrounds, reviews, and authentication.
- `views/` – EJS templates for pages (home, campgrounds, auth, etc.).
- `public/` – Static assets such as CSS, client-side JS, and images.
- `Dockerfile` / `docker-compose.yml` – Containerization and orchestration config.

---

## 6. CI/CD Overview

This project is designed to work with a **Jenkins + SonarQube** CI pipeline and a separate **CD repository (`camp-cd`)** that deploys to a Kubernetes (kind) cluster.

### 6.1 CI Pipeline (This Repository)

The CI pipeline is defined in the `Jenkinsfiles` file in this repo and typically runs inside WSL using Docker Compose (Jenkins + SonarQube services). The high-level stages are:

- **Clean workspace**: Remove any previous build artifacts (`cleanWs()`).
- **Git checkout**: Clone the main branch of this repo from GitHub.
- **Install dependencies**: Run `npm install`.
- **Test**: Run `npm test`.
- **Filesystem security scan**: Use **Trivy** to scan the working directory (`trivy fs`).
- **SonarQube analysis**: Run `sonar-scanner` with the configured `SCANNER_HOME` and `SONAR_TOKEN` using the `withSonarQubeEnv('sonarqube')` block.
- **Docker image build**: Build the application image with a tag like `rahul6364/yelp-camp:1.0.${BUILD_NUMBER}`.
- **Container image scan**: Use **Trivy** to scan the built Docker image.
- **Push to Docker Hub**: Log in with `dockerhub` credentials and push the built image.
- **Trigger CD job**: Call a downstream Jenkins job named **`camp-cd`**, passing `IMAGE_TAG` as a parameter (asynchronous, `wait: false`).

### 6.2 CD Repository: `camp-cd`

The **CD job** is defined in a separate GitHub repository: [`camp-cd`](https://github.com/rahul6364/camp-cd). That repo typically contains:

- A **Jenkinsfile** that receives the `IMAGE_TAG` parameter from this CI pipeline.
- **Kubernetes manifests** (Deployments, Services, etc.) that reference the `rahul6364/yelp-camp:${IMAGE_TAG}` image.
- Logic to deploy to a **kind** cluster running inside WSL (e.g., using `kubectl apply -f` against the manifest files).

Overall flow:

1. Code pushed to this repo → Jenkins CI pipeline runs (SonarQube, Trivy, build, push).
2. CI pipeline triggers the **`camp-cd`** Jenkins job with the new image tag.
3. The `camp-cd` job updates/deploys the app to the **kind** Kubernetes cluster using the Kubernetes manifests in the `camp-cd` repo.

---

## 7. Application Screenshots

![](./images/home.jpg)
![](./images/campgrounds.jpg)
![](./images/register.jpg)

---

## 8. Notes & Credits

- Based on the **YelpCamp** project from *“The Web Developer Bootcamp”* by Colt Steele, with additional enhancements and a containerized setup.
- All API keys, secrets, and credentials used in development or production should be stored securely (environment variables, secret managers, etc.).