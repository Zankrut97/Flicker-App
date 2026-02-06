# BrighterScript Template

This project is created from the template project written in [BrighterScript](https://github.com/rokucommunity/brighterscript)

## Project Architecture

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                              ROKU DEVICE                                    │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  ┌────────────────────────────────────────────────────────────────────────┐ │
│  │                           MainScene                                    │ │
│  │  ┌──────────────────┐  ┌──────────────────┐  ┌──────────────────────┐  │ │
│  │  │ LoadingIndicator │  │ CategoryRowList  │  │    DetailScreen      │  │ │
│  │  │                  │  │                  │  │                      │  │ │
│  │  │  ┌────────────┐  │  │  ┌────────────┐  │  │  ┌────────────────┐  │  │ │
│  │  │  │BusySpinner │  │  │  │  RowList   │  │  │  │  Photo Info    │  │  │ │
│  │  │  └────────────┘  │  │  │            │  │  │  │  - Title       │  │  │ │
│  │  └──────────────────┘  │  │  ┌──────┐  │  │  │  │  - Owner       │  │  │ │
│  │                        │  │  │ Card │  │  │  │  │  - Description │  │  │ │
│  │                        │  │  │      │  │  │  │  │  - Dimensions  │  │  │ │
│  │                        │  │  │Poster│  │  │  │  └────────────────┘  │  │ │
│  │                        │  │  │Label │  │  │  │                      │  │ │
│  │                        │  │  └──────┘  │  │  │  ┌────────────────┐  │  │ │
│  │                        │  └────────────┘  │  │  │  Detail Image  │  │  │ │
│  │                        └──────────────────┘  │  └────────────────┘  │  │ │
│  │                                              └──────────────────────┘  │ │
│  └────────────────────────────────────────────────────────────────────────┘ │
│                                                                             │
│  ┌────────────────────────────────────────────────────────────────────────┐ │
│  │                         TASK THREADS                                   │ │
│  │  ┌─────────────────────────┐    ┌─────────────────────────┐            │ │
│  │  │  FlickrCategoryTask     │    │  FlickrPhotoInfoTask    │            │ │
│  │  │  - flickr.photos.search │    │  - flickr.photos.getInfo│            │ │
│  │  │  - Fetches 20 photos    │    │  - flickr.photos.getSizes│           │ │
│  │  │    per category         │    │  - Fetches dimensions   │            │ │
│  │  └─────────────────────────┘    └─────────────────────────┘            │ │
│  └────────────────────────────────────────────────────────────────────────┘ │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
                                      │
                                      ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                           FLICKR REST API                                   │
│                     https://api.flickr.com/services/rest/                   │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Folder Structure

```
src/
├── manifest                    # Channel metadata (title, icons, splash)
├── source/
│   ├── main.bs                 # Entry point (Main, RunUserInterface)
│   └── promises.bs             # Async promise utilities
├── components/
│   ├── MainScene.xml/bs        # Root scene, orchestrates app flow
│   ├── CategoryRowList/        # RowList displaying photo categories
│   │   ├── CategoryRowList.xml
│   │   └── CategoryRowList.bs
│   ├── Card/                   # Individual photo card component
│   │   ├── Card.xml
│   │   └── Card.bs
│   ├── LoadingIndicator/       # Centered loading spinner
│   │   ├── LoadingIndicator.xml
│   │   └── LoadingIndicator.bs
│   ├── Screens/
│   │   └── DetailScreen/       # Full-screen photo detail view
│   │       ├── DetailScreen.xml
│   │       └── DetailScreen.bs
│   └── tasks/
│       ├── FlickrCategoryTask/ # Fetches photos by category
│       │   ├── FlickrCategoryTask.xml
│       │   └── FlickrCategoryTask.bs
│       └── FlickrPhotoInfoTask/# Fetches photo details & sizes
│           ├── FlickrPhotoInfoTask.xml
│           └── FlickrPhotoInfoTask.bs
└── images/                     # App assets (icons, placeholders)
```

### Data Flow

```
┌─────────────┐    fetch     ┌──────────────────────┐    HTTP    ┌─────────────┐
│  MainScene  │ ──────────▶  │ FlickrCategoryTask   │ ─────────▶ │ Flickr API  │
│             │              │ (parallel per cat.)  │            │             │
└─────────────┘              └──────────────────────┘            └─────────────┘
       │                              │
       │ promises.all()               │ result
       │                              ▼
       │                     ┌──────────────────────┐
       │ ◀───────────────────│   Category Data      │
       │                     │   {nature: [...],    │
       │                     │    animals: [...]}   │
       ▼                     └──────────────────────┘
┌─────────────────────┐
│  CategoryRowList    │
│  ┌───────────────┐  │
│  │ Nature Row    │  │
│  │ ┌────┐ ┌────┐ │  │
│  │ │Card│ │Card│ │  │
│  │ └────┘ └────┘ │  │
│  └───────────────┘  │
│  ┌───────────────┐  │
│  │ Animals Row   │  │
│  └───────────────┘  │
└─────────────────────┘
       │
       │ onItemSelected
       ▼
┌─────────────────────┐    fetch     ┌──────────────────────┐
│   DetailScreen      │ ──────────▶  │ FlickrPhotoInfoTask  │
│   - Large image     │              │ - Description        │
│   - Title, Owner    │ ◀────────────│ - Dimensions         │
│   - Description     │   result     └──────────────────────┘
│   - Dimensions      │
└─────────────────────┘
```

### Key Technologies

| Technology         | Purpose                                      |
| ------------------ | -------------------------------------------- |
| **BrighterScript** | TypeScript-like superset of BrightScript     |
| **SceneGraph**     | Roku's XML-based UI framework                |
| **Promises**       | Async handling via `@rokucommunity/promises` |
| **Flickr API**     | Photo search and metadata                    |
| **RowList**        | Horizontal scrolling rows for categories     |
| **Task Nodes**     | Background threads for network requests      |

## Setup Instructions

1. Install [NodeJS](https://nodejs.org)
2. Clone the repository:
   ```bash
   git clone https://github.com/Zankrut97/Flicker-App.git
   ```
3. Navigate to the app directory:
   ```bash
   cd Flicker-App
   ```
4. Install npm dependencies:
   ```bash
   npm install
   ```
5. Build the project:
   ```bash
   npm run build
   ```

## Running the App

### Option 1: VS Code (Recommended)

1. Install the [BrightScript Language](https://marketplace.visualstudio.com/items?itemName=RokuCommunity.brightscript) extension for VS Code
2. Open the project in VS Code
3. Click **Run > Start Debugging** to deploy and launch the app on your Roku device

### Option 2: Manual Deployment

1. Build a zip package:
   ```bash
   npm run package
   ```
2. Enable Developer Mode on your Roku device
3. Upload the generated `.zip` file via the Roku Developer Application Installer

## Debugging

This repository comes pre-configured to work with the [BrightScript Language](https://github.com/rokucommunity/vscode-brightscript-language) extension for Visual Studio Code.

## Future Improvements

| Feature                        | Description                                                                                                                                                               |
| ------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **REST Service Layer**         | Create a centralized HTTP request handler to manage all API calls, handle response parsing, timeouts, and provide a clean abstraction over `roUrlTransfer`                |
| **Screen Manager Integration** | Utilize the existing ScreenManager component for proper screen navigation, history stack management, and transitions                                                      |
| **Better Error Handling**      | Implement comprehensive error handling with user-friendly error messages, retry logic for failed network requests, and graceful degradation when services are unavailable |
| **Pagination Support**         | Add infinite scroll or "Load More" functionality for categories with many photos                                                                                          |
