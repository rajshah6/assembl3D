# assembl3D

**Copilot for Assembly**

![Demo](.backend/demo_optimized.gif)

An AI-powered platform that transforms static PDF assembly manuals into interactive 3D assembly guides. The system leverages web scraping, computer vision, and 3D rendering to extract structured assembly instructions from furniture manuals and present them in an immersive, step-by-step visualization environment.

**[Watch Full Demo Video](https://www.youtube.com/watch?v=c-3XQTTbKns)** 

**[View Devpost Submission](https://devpost.com/software/assembl3d)**

## Architecture Overview

The application follows a microservices architecture with a Node.js/Express backend and a Next.js 15 frontend. The pipeline consists of four main stages:

1. **Web Scraping Layer**: Uses Bright Data's SERP API to discover product pages, Web Scraper to extract metadata, and Web Unlocker to bypass anti-scraping measures and download PDF manuals
2. **AI Processing Pipeline**: Converts PDF pages to images, feeds them to Google Gemini 2.0 Flash (vision model) for multi-modal analysis, and extracts structured data including assembly steps, part lists, tool requirements, and 3D spatial relationships
3. **Data Transformation**: Transforms AI-extracted data into Three.js-compatible scene graphs with geometric primitives, materials, and animations
4. **3D Rendering Engine**: React Three Fiber-based viewer with interactive controls, part highlighting, cumulative scene building, and real-time step navigation

## Quick Start

You'll need Node.js 18+ and API keys for [Bright Data](https://brightdata.com) and [Google Gemini](https://ai.google.dev).

**Backend:**
```bash
cd backend
npm install
cp .env.example .env
# Add your API keys
npm run dev
```

**Frontend:**
```bash
cd frontend
npm install
cp .env.example .env.local
# Add NEXT_PUBLIC_API_URL=http://localhost:3001
npm run dev
```

Visit `http://localhost:3000` to get started.

## Project Structure

```
assembl3D/
├── backend/                          # Express API server
│   ├── brightdata/                  # Web scraping module
│   │   ├── scraper.ts              # Main scraping orchestrator
│   │   ├── serp-search.ts          # SERP API integration
│   │   ├── web-scraper.ts          # Product page extraction
│   │   ├── pdf-downloader.ts       # PDF download via Web Unlocker
│   │   ├── scrape-top-products.ts  # Batch product scraping
│   │   ├── generate-top-50.ts      # Top products data generator
│   │   └── types.ts                # Scraping interfaces
│   │
│   ├── src/
│   │   ├── api/                    # REST API routes
│   │   │   ├── routes.ts          # Main route definitions
│   │   │   └── pdf-processor.route.ts  # PDF processing endpoint
│   │   │
│   │   ├── gemini/                 # AI processing pipeline
│   │   │   ├── processor.ts       # Main orchestrator (PDF → steps)
│   │   │   ├── pdf-parser.ts      # PDF to image conversion
│   │   │   ├── prompt-builder.ts  # Dynamic prompt generation
│   │   │   ├── scene-generator.ts # 3D scene JSON generation
│   │   │   └── types.ts           # AI extraction interfaces
│   │   │
│   │   ├── parser_docs/            # PDF processing documentation
│   │   └── index.ts                # Express server entry point
│   │
│   ├── data/
│   │   ├── images/                 # Cached product images
│   │   ├── top-50-products.json    # Pre-scraped product library
│   │   └── output/                 # Processed assembly steps (JSON)
│   │
│   └── models/                      # 3D model assets (.glb files)
│
├── frontend/                        # Next.js 15 application
│   ├── app/                         # App Router pages
│   │   ├── page.tsx                # Landing page
│   │   ├── assembly/[id]/          # Dynamic assembly viewer route
│   │   ├── api/
│   │   │   ├── assembly-chat/      # Reka AI chatbot API route
│   │   │   └── reka-vision/        # Vision API integration
│   │   └── layout.tsx              # Root layout
│   │
│   ├── components/
│   │   ├── assembly/               # Assembly UI components
│   │   │   ├── AssemblyPageClient.tsx    # Main assembly page logic
│   │   │   ├── AssemblyChatbot.tsx       # AI chatbot interface
│   │   │   ├── StepList.tsx              # Step navigation sidebar
│   │   │   ├── PartsList.tsx             # Parts list display
│   │   │   ├── ToolsList.tsx             # Tools required display
│   │   │   └── StepNavigation.tsx        # Previous/Next controls
│   │   │
│   │   ├── viewer/                 # 3D rendering components
│   │   │   ├── AssemblyViewer.tsx        # Main Three.js viewer
│   │   │   ├── DataDrivenScene.tsx       # Scene from JSON data
│   │   │   ├── CumulativeScene.tsx      # Progressive scene building
│   │   │   ├── PartHighlighter.tsx       # Part interaction system
│   │   │   ├── ViewerControls.tsx        # Camera/orbit controls UI
│   │   │   ├── SceneLoader.tsx           # Scene data loader
│   │   │   └── [AnimatedPart, Screw, Washer, LBracket].tsx  # 3D primitives
│   │   │
│   │   ├── search/                 # Search functionality
│   │   │   ├── search-section.tsx        # Main search component
│   │   │   ├── SearchResults.tsx         # Results display
│   │   │   ├── ProductCard.tsx           # Product card UI
│   │   │   └── SearchProgress.tsx        # Real-time progress indicator
│   │   │
│   │   ├── library/                # Product library
│   │   │   ├── library-section.tsx       # Library grid view
│   │   │   └── library-card.tsx          # Product card component
│   │   │
│   │   ├── landing/                # Landing page components
│   │   └── ui/                     # Shadcn/ui components
│   │
│   ├── lib/
│   │   ├── api-client.ts           # Backend API wrapper
│   │   ├── top-50-data.ts          # Product data utilities
│   │   └── utils.ts                # Helper functions
│   │
│   └── public/
│       └── products/                # Static product images
```

### Key Directories Explained

**`backend/brightdata/`**: Web scraping orchestration layer. Handles product discovery via SERP API, metadata extraction, and PDF acquisition through Bright Data's proxy network.

**`backend/src/gemini/`**: AI processing pipeline. Converts PDFs to images, constructs vision prompts, invokes Gemini API, and transforms responses into structured assembly data with 3D geometry.

**`frontend/components/viewer/`**: Three.js rendering engine. Implements scene graph construction, cumulative step visualization, part highlighting, and camera controls using React Three Fiber.

**`frontend/components/assembly/`**: Assembly instruction UI. Manages step navigation, parts/tools display, and integrates Reka AI chatbot for contextual assistance.

## Technical Workflow

1. **Product Discovery**: User submits search query or IKEA product URL → Bright Data SERP API performs semantic search across regional IKEA domains
2. **Data Extraction**: Web Scraper extracts product metadata (name, SKU, image URLs) → Web Unlocker bypasses bot detection and downloads assembly PDF
3. **PDF Processing**: PDF pages converted to base64-encoded images → Each page analyzed by Gemini 2.0 Flash vision model with structured prompts
4. **AI Extraction**: Gemini returns JSON with step descriptions, part quantities, tool requirements, and geometric data (positions, rotations, scales)
5. **Scene Generation**: Extracted data transformed into Three.js scene graph → Primitives (boxes, cylinders) positioned in 3D space → Materials and animations applied
6. **Rendering**: React Three Fiber renders scene → User navigates steps → Cumulative scene builds progressively → Parts highlight on hover/selection

## Tech Stack

- **Frontend**: Next.js 15 (App Router), React 19, TypeScript, Tailwind CSS, Shadcn/ui
- **3D Rendering**: Three.js, React Three Fiber, @react-three/drei
- **Backend**: Node.js, Express, TypeScript
- **AI/ML**: Google Gemini 2.0 Flash (vision), Reka AI Core (chatbot)
- **Web Scraping**: Bright Data (SERP API, Web Unlocker, Web Scraper, Residential Proxies)
- **PDF Processing**: pdf-lib, pdfjs-dist, Sharp (image conversion)

## Environment Variables

**Backend** (`backend/.env`):
```bash
GEMINI_API_KEY=your_key
BRIGHT_DATA_API_KEY=your_key
PORT=3001
```

**Frontend** (`frontend/.env.local`):
```bash
NEXT_PUBLIC_API_URL=http://localhost:3001
REKA_API_KEY=your_key
```

---

Built at Cal Hacks 12.0 🏆
