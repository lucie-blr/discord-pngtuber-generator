# Discord PNGTuber Generator

A lightweight, web-based application designed to generate and manage PNGTuber avatars for Discord. Built with Vue.js and Vite, this project offers a fast, modern development experience and includes out-of-the-box Docker support for seamless deployment.

## 🚀 Features

* **Interactive UI:** Powered by Vue.js (`App.vue`) for a highly responsive and reactive user interface.
* **Lightning-Fast Build:** Utilizes Vite (`vite.config.js`) for optimized builds and rapid Hot Module Replacement (HMR) during development.
* **Container Ready:** Includes a `Dockerfile` for standardized deployment across any environment.

## 🛠️ Prerequisites

Before you begin, ensure you have the following installed on your machine:

* [Node.js](https://nodejs.org/) (Version 16.x or higher recommended)
* npm (comes with Node.js) or Yarn
* [Docker](https://www.docker.com/) *(Optional, if you wish to run the containerized version)*

## 📦 Local Setup & Development

**1. Navigate to the project directory:**

```bash
cd discord-pngtuber-generator

```

**2. Install dependencies:**

```bash
npm install

```

**3. Start the Vite development server:**

```bash
npm run dev

```

The console will provide a local URL (usually `http://localhost:5173/`). Open this link in your browser to view the application.

## 🏗️ Building for Production

To compile and minify the application for a production environment, run:

```bash
npm run build

```

This command bundles the Vue app into static files located in a newly generated `dist/` directory, ready to be served by any static file host.

## 🐳 Docker Deployment

If you prefer to run or deploy the application using Docker, follow these steps:

**1. Build the Docker image:**

```bash
docker build -t discord-pngtuber-generator .

```

**2. Run the container:**

```bash
docker run -d -p 8080:80 discord-pngtuber-generator

```

*(Note: Be sure to map the external port `8080` to whatever internal port is exposed in your `Dockerfile`.)*

## 📂 Project Structure Overview

* **`/src`**: Contains the core application code, including the main entry point (`main.js`), the root component (`App.vue`), and stylesheets (`/assets`).
* **`/public`**: Static files that are served directly, such as the `favicon.ico`.
* **`/Dockerfile`**: Contains the blueprint for creating the project's Docker container.
* **`vite.config.js`**: The configuration file for the Vite build tool.
* **`jsconfig.json`**: Provides JavaScript language service configuration for your IDE.
