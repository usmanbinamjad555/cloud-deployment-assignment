# Augment Fit Admin Dashboard: Containerized Deployment

An admin dashboard for the Augment Fit fitness platform, built with React (Create React App) and Firebase. This repository's focus is deployment: a multi-stage Docker build that serves the production bundle with nginx, Docker Compose files, and a Jenkins pipeline that runs the app and health-checks it on every build.

![React](https://img.shields.io/badge/React-18-61DAFB?logo=react&logoColor=black)
![Firebase](https://img.shields.io/badge/Firebase-Auth%20%7C%20Firestore%20%7C%20Storage-FFCA28?logo=firebase&logoColor=black)
![Tailwind CSS](https://img.shields.io/badge/Tailwind_CSS-3-06B6D4?logo=tailwindcss&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-multi--stage-2496ED?logo=docker&logoColor=white)
![nginx](https://img.shields.io/badge/nginx-1.23-009639?logo=nginx&logoColor=white)
![Jenkins](https://img.shields.io/badge/Jenkins-pipeline-D24939?logo=jenkins&logoColor=white)

## Features

**Application**

- **Admin login** with Firebase Authentication (email and password); only accounts listed in the Firestore `admin` collection can sign in, and all pages are behind a route guard.
- **Dashboard** with totals for users, subscribed users and trainers, plus Recharts charts of posts per user and a BMI classification of users.
- **User management**: search, view details and posts, edit profile fields, and delete users (Firestore `users` collection).
- **Trainer management**: search, edit and delete trainers (Firestore `trainers` collection).
- **Content verification**: review posts, highlight words found in a word list (`src/assets/filter-words.txt`) and delete posts.
- **Settings**: change the admin's email and password and upload a profile picture to Firebase Storage.

**Deployment**

- **Multi-stage Dockerfile**: builds the app with Node.js, then serves the static build from `nginx:1.23-alpine` on port 80.
- **Jenkins pipeline** (`Jenkinsfile`): checks out `main`, stops previous containers, starts the app with `docker-compose-ci.yaml`, then runs a `curl` health check on port 8082 and prints container logs.
- **Docker Compose** files for local and CI runs.
- **Netlify** SPA redirect rule in `public/netlify.toml`.

## Tech stack

| Area | Technology |
| --- | --- |
| Frontend | React 18, React Router 6, Create React App (react-scripts 5) |
| Styling | Tailwind CSS 3, custom CSS |
| Charts | Recharts |
| Backend services | Firebase Authentication, Cloud Firestore, Cloud Storage (Firebase JS SDK 9) |
| Container | Docker (multi-stage), nginx 1.23 |
| CI/CD | Jenkins, Docker Compose |

## Project structure

```
.
├── Dockerfile                 # Node build stage -> nginx runtime stage
├── docker-compose.yml         # Local compose file
├── docker-compose-ci.yaml     # Compose file used by Jenkins (port 8082)
├── Jenkinsfile                # CI/CD pipeline
├── public/                    # index.html, icons, netlify.toml
└── src/
    ├── firebase.js            # Firebase app, Auth, Firestore and Storage setup
    ├── index.js               # Router, auth provider, protected routes
    ├── App.js                 # Dashboard layout and page routes
    ├── contexts/              # Auth context (login, logout, auth check)
    ├── routes/RequireAuth.js  # Route guard
    ├── pages/                 # dashboard, users, userDetails, trainers, contentVerification, settings, login
    ├── components/            # Layout, tables, charts, modal, pagination
    ├── constants/             # Navigation options
    ├── assets/                # Images, icons, filter-words.txt
    └── styles/                # CSS files
```

## Getting started

### Prerequisites

- Node.js 18 and npm (the CI compose file uses `node:18-alpine`)
- Docker (and Docker Compose for the compose files)
- A Firebase project with Authentication, Firestore and Storage enabled, and at least one document in the `admin` collection whose `id` field matches an Auth user's UID

### Configuration

There are no environment variables. The Firebase web configuration is defined in `src/firebase.js`; to point the dashboard at your own Firebase project, replace the values in that file with your project's web app config from the Firebase console.

### Run locally

```bash
npm install
npm start          # development server on http://localhost:3000
npm run build      # production build in build/
```

### Run with Docker

```bash
docker build -t augment-fit . && docker run -p 80:80 augment-fit
```

The app is then served by nginx on `http://localhost`.

### Run the CI compose file

```bash
docker-compose -f docker-compose-ci.yaml up -d
```

This mounts the source into a `node:18-alpine` container, runs `npm install && npm start`, and exposes the dev server on `http://localhost:8082`. The Jenkins pipeline uses the same file with project name `salon`.

## Security note

The Firebase web config in `src/firebase.js` (including `apiKey`) is meant to be public: it identifies the Firebase project and is shipped to every browser that loads the app, so it is not a secret and is intentionally left in the source. Access control has to come from Firebase itself. The owner should:

- **Enforce Firestore and Storage security rules** so only authenticated admins can read and write the `admin`, `users`, `trainers`, `posts` and `webnotifications` collections and upload files. The admin check in this app runs only in the browser and does not protect the data on its own.
- **Restrict the API key** in Google Cloud Console (APIs & Services > Credentials) to the app's own domains (HTTP referrer restrictions) and to the Firebase APIs the app uses.

## Known limitations

- `docker-compose.yml` maps port `3000:3000`, but the Docker image serves on port 80 through nginx, so that mapping needs to be `3000:80` to reach the app.
- Both compose files start a MongoDB container that the app does not use; all data lives in Firebase.
- The CI compose file runs the development server (`npm start`), not the production nginx image, and the pipeline waits a fixed 180 seconds before its health check.
- The nginx image uses the default nginx config, so refreshing a deep link such as `/dashboard` returns 404; a `try_files ... /index.html` rule is needed for client-side routing.
- There are no tests beyond the default Create React App test file.

---

Author: Muhammad Usman — [LinkedIn](https://www.linkedin.com/in/muhammad-usman-the-dev) · [Portfolio](https://muhammad-usman-engineer.vercel.app)
