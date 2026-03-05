# AI Shorts Video Generator

A monorepo project for generating AI-powered short videos. The application combines multiple AI services to create videos from text prompts, including content generation, text-to-speech, image generation, and automated captioning.

![A demonstration video showing the application in action](./_docs/demo-video.mp4)

## Architecture

The application consists of two main components:

- **Frontend** ([docs](./frontend/README.md)): Next.js application that provides the user interface and handles video rendering using Remotion on AWS Lambda
- **Backend** ([docs](./backend/README.md)): ASP.NET Core API that orchestrates AI services and manages video data

### Video Generation Workflow

1. User submits video topic, style, and duration
2. Backend generates script using Gemini API
3. Backend converts script to audio using Google Cloud Text-to-Speech
4. Backend generates images for each scene using Cloudflare Workers AI
5. Backend creates captions using AssemblyAI
6. Frontend triggers Remotion Lambda to render final video
7. Rendered video is stored in Cloudinary

## Table of Contents

- [Architecture](#architecture)
- [Features](#features)
- [Requirements](#requirements)
- [Installation](#installation)
- [Environment Variables](#environment-variables)
- [Development](#development)
- [Docker](#docker)
- [API Reference](#api-reference)
- [Deployment](#deployment)
- [Usage](#usage)

## Features

### Frontend

- **Video Creation**: Users can create short videos by selecting topics, styles, and durations.
- **Video Preview**: After generating a video, users can preview it with generated captions and images.
- **Video Deletion**: Users can delete the video from the database and the cloud if video was once rendered.
- **Export Video**: Users can export the generated video once the rendering is completed.
- **Theming**: Supports light and dark themes using `next-themes`.

### Backend

- **Content Generation**: Create video content based on user input using Google's Gemini API.
- **Audio Synthesis**: Convert text input to speech using Google Cloud Text-to-Speech API.
- **Caption Generation**: Generate captions for audio or video files using AssemblyAI.
- **Image Generation**: Generate images from text prompts using Cloudflare's AI API.
- **Video Storage**: Save and retrieve video data, including associated content and captions.

## Requirements

### Local Development

- Node.js (v20.x LTS or later)
- [.NET 9 SDK](https://dotnet.microsoft.com/download/dotnet)
- [PostgreSQL](https://www.postgresql.org/download/) (or use [Neon Serverless Postgres](https://neon.tech/))
- Docker and Docker Compose (optional, for containerized development)

### External Services

- [Cloudinary](https://cloudinary.com/) account and API key
- [Cloudflare Workers AI](https://developers.cloudflare.com/workers-ai/) account and API key (using [flux-1-schnell Model](https://developers.cloudflare.com/workers-ai/models/flux-1-schnell/))
- [AssemblyAI](https://www.assemblyai.com/) API key
- [Google Cloud](https://cloud.google.com/) project with Text-to-Speech and Gemini APIs enabled
- [AWS Account](https://aws.amazon.com/) for Remotion Lambda (video rendering)

## Installation

1. Clone the repository to your local machine:

    ```bash
    git clone https://github.com/yourusername/AiShortsVideoGenerator.git
    cd AiShortsVideoGenerator
    ```

2. Install frontend dependencies:

    ```bash
    cd frontend
    npm install
    ```

3. Install backend dependencies:

    ```bash
    cd ../backend
    dotnet restore
    ```

## Environment Variables

### Frontend (`.env.local`)

```plaintext
NEXT_PUBLIC_API_URL=http://localhost:8080
REMOTION_AWS_SERVE_URL=<Your AWS Serve URL for Remotion>
REMOTION_AWS_BUCKET_NAME=<Your AWS Bucket Name for Remotion>
```

See [Remotion Lambda setup guide](https://www.remotion.dev/docs/lambda/setup) for AWS configuration.

### Backend (User Secrets for Development)

For local development, use .NET user secrets instead of committing API keys:

```bash
cd backend
dotnet user-secrets init
dotnet user-secrets set "GoogleApi:GeminiKey" "your-gemini-api-key"
dotnet user-secrets set "GoogleApi:TextToSpeechKey" "your-text-to-speech-api-key"
dotnet user-secrets set "AssemblyAi:ApiKey" "your-assemblyai-api-key"
dotnet user-secrets set "CloudinaryUrl" "your-cloudinary-url"
dotnet user-secrets set "Cloudflare:ApiKey" "your-cloudflare-api-key"
dotnet user-secrets set "Cloudflare:AccountId" "your-cloudflare-account-id"
dotnet user-secrets set "ConnectionStrings:DefaultConnection" "your-postgresql-connection-string"
```

For production, use `appsettings.Production.json` or environment variables.

### Google Cloud Setup

1. Create a service account in Google Cloud Console
2. Download the service account JSON key
3. Set `GOOGLE_APPLICATION_CREDENTIALS` environment variable to the key path
4. Enable Text-to-Speech and Gemini APIs in your project

## Development

### Frontend
To run the frontend application in development mode:

```bash
cd frontend
npm install
npm run dev
```

The frontend will be available at `http://localhost:3000`.

### Backend
To run the backend application:

```bash
cd backend
dotnet restore
dotnet ef database update
dotnet run
```

The backend API will be available at `http://localhost:8080`.

### CORS Configuration

Ensure the backend is configured to accept requests from the frontend. Update `appsettings.Development.json` or use user secrets to configure CORS if needed.

## Docker

To run the application in Docker:

1. Configure environment variables in `.env.local`:

```plaintext
NEXT_PUBLIC_API_URL=http://localhost:8080
```

2. Set backend secrets or create `appsettings.Production.json` with API keys

3. Run the containers:

```bash
docker compose up
```

This starts:
- Backend API on `http://localhost:8080`
- Frontend on `http://localhost:3000`
- PostgreSQL database (if configured in compose.yaml)

## API Reference

### Base URL

```
http://localhost:8080
```

### Endpoints

- **GET /videos**: Retrieve a list of all videos stored in the database.
- **POST /generate-content**: Generate video content from user input using Google's Gemini API.
- **POST /generate-audio**: Convert text input to audio (MP3) using Google Cloud Text-to-Speech API.
- **POST /generate-captions**: Generate captions for an audio or video file using AssemblyAI.
- **POST /generate-image**: Generate an image from a text prompt using Cloudflare's AI API.
- **POST /save-video**: Save a video record to the database.
- **PUT /videos/{id}**: Update video Output File and Render Id.
- **DELETE /videos/{id}**: Delete video from the database.

## Dependencies

### Frontend

- **Next.js**: React framework for building server-side rendered applications.
- **TailwindCSS**: Utility-first CSS framework for styling the app.
- **Remotion**: Library for creating videos programmatically with React.
- **Axios**: HTTP client for making requests to the API.
- **shadcn**: A collection of re-usable components that you can copy and paste into your apps.

### Backend

- **Google.Cloud.TextToSpeech.V1**: Google Cloud Text-to-Speech client.
- **AssemblyAI**: Client library for AssemblyAI services.
- **CloudinaryDotNet**: Cloudinary SDK for uploading media.
- **Npgsql.EntityFrameworkCore.PostgreSQL**: PostgreSQL support for Entity Framework Core.

## Deployment

### Frontend

Deploy to Vercel or similar platforms:

```bash
cd frontend
npm run build
```

Set environment variables in your deployment platform:
- `NEXT_PUBLIC_API_URL`: Production backend URL
- `REMOTION_AWS_SERVE_URL`: AWS Lambda endpoint
- `REMOTION_AWS_BUCKET_NAME`: S3 bucket for video outputs

### Backend

Deploy to Azure App Service, AWS, or similar:

1. Build the application:

```bash
cd backend
dotnet publish -c Release -o ./publish
```

2. Set production environment variables or use `appsettings.Production.json`

3. Ensure database migrations run on startup or manually:

```bash
dotnet ef database update --configuration Release
```

### AWS Lambda (Remotion)

Follow the [Remotion Lambda setup guide](https://www.remotion.dev/docs/lambda/setup) to configure video rendering on AWS Lambda.

## Usage

- **Dashboard**: Users can view and manage generated short videos.
![Image of the dashboard page](./_docs/dashboard.png)
- **Create New Video**: Users can define the video topic, style, and duration, and the app will generate the video content, audio, captions, and images using AI.
![Image of the create new video page](./_docs/create-new.png)
- **Video Preview & Export**: After the video is generated, users can preview it in the dialog and export it once rendered.
![Image of the preview, export and delete video](./_docs/preview-video.png)
