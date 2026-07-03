BTP-RecipieDB
How to Run
This project is split into two applications:

server/ - Express API that interprets chat requests and forwards recipe lookups
client/ - React + Vite frontend for the chatbot UI
The app expects:

Node.js and npm installed
The external RecipeDB API service to be reachable
Run with start.bat on Windows
Install dependencies once:
cd server
npm install
cd ../client
npm install
cd ..
From the repository root, run:
.\start.bat
The batch file opens both services:
Server: http://localhost:5000
Client: http://localhost:5173
Run manually from the terminal
Open two terminals from the project root.

Terminal 1:

cd server
npm install
npm start
Terminal 2:

cd client
npm install
npm run dev
Then open http://localhost:5173 in the browser.

Important dependency: RecipeDB backend service
This repository does not contain the actual RecipeDB data service. The Express server in this repo acts as a chat-oriented middle layer and calls another backend through RECIPEDB_BASE_URL.

Current default in server/config.js:

PORT=5000
RECIPEDB_BASE_URL=http://192.168.1.92:3033/api
GROQ_API_KEY=...
If that RecipeDB API is not running or not reachable, the frontend may load, but recipe searches, details, nutrition, and meal-plan results will fail or return empty data.

Optional environment setup
Create server/.env if you want to override the defaults:

PORT=5000
RECIPEDB_BASE_URL=http://localhost:3033/api
GROQ_API_KEY=your_key_here
Project Overview
BTP-RecipieDB is a chat-style recipe search application. A user enters natural-language requests such as:

"find me pasta recipes"
"show vegetarian lunch options"
"recipes with protein between 20 and 40"
"generate a meal plan for 2000 calories with 3 meals"
The frontend sends the message to the local Express server, the server determines the intent, fetches matching data from the external RecipeDB API, and returns a structured response that the UI renders as:

plain chat replies
paginated recipe tables
meal-plan summaries
recipe detail, ingredient, and nutrition modals
Tech Stack
React 19 for the frontend UI
Vite for frontend development and build tooling
Tailwind CSS for styling
Express 5 for the local API server
Axios for outbound HTTP requests
cors, body-parser, and dotenv for server support
What the Application Supports
Search capabilities
The current implementation supports these intent types:

Intent	Example
SEARCH_BY_TITLE	"chocolate cake"
SEARCH_BY_CALORIES	"recipes between 200 and 400 calories"
SEARCH_BY_CARBS	"recipes with carbs between 10 and 20"
SEARCH_BY_PROTEIN	"high protein recipes between 30 and 50"
SEARCH_BY_DIET	"vegan recipes"
SEARCH_BY_REGION	"Indian recipes"
SEARCH_BY_REGION_DIET	"Indian vegetarian recipes"
SEARCH_BY_CATEGORY	"breakfast recipes"
SEARCH_BY_UTENSILS	"recipes made using a pan"
SEARCH_BY_COOKING_METHOD	"baked dishes"
SEARCH_BY_TOP_RATED	"top rated recipes"
SEARCH_BY_RANDOM	"give me random recipes"
GENERATE_MEAL_PLAN	"generate a meal plan for 1800 calories with 3 meals"
Frontend features
Chat interface with user and assistant message bubbles
Quick-start suggestion buttons in the left sidebar
Up to 5 saved chat threads in browser localStorage
Chat deletion and switching from the right sidebar
Paginated table results for recipe searches
Modal views for instructions, nutrition, and ingredients
Responsive layout for desktop and mobile
Session behavior
Client-side chats are stored in localStorage
Server-side session context is stored in memory
The server remembers prior messages for follow-up ranges such as "between 20 and 40" after a protein or calorie search
Server session history is cleared when the server restarts
Architecture
High-level flow
The user sends a message from the React frontend.
client/src/App.jsx posts that message to POST /api/chat.
server/index.js creates or reuses a session through chatContextStore.js.
server/llm.js detects the intent and extracts parameters.
server/recipedb.js calls the external RecipeDB REST API with Axios.
The server returns a structured response containing reply, type, data, intent, params, and sessionId.
The client renders the response as text, a data table, or a meal-plan panel.
Key implementation note
Even though the file is named llm.js and server/config.js contains Groq settings, the current intent pipeline is rule-based, not an active hosted LLM integration. Intent detection is done with pattern matching and keyword extraction inside server/llm.js.

That means:

Groq configuration is currently unused by the main request path
behavior is deterministic and regex-driven
adding new intents mainly requires editing server/llm.js, server/index.js, and server/recipedb.js
Local API Surface
These are the endpoints exposed by the local Express server in this repository.

POST /api/chat
Accepts a user message and returns a response tailored to the detected recipe intent.

Request body:

{
  "message": "show me Indian vegan recipes",
  "sessionId": "optional-existing-session-id"
}
Response shape:

{
  "reply": "Searching for indian vegan recipes.",
  "type": "table",
  "data": [],
  "intent": "SEARCH_BY_REGION_DIET",
  "params": {
    "region": "indian",
    "diet": "vegan"
  },
  "sessionId": "generated-or-reused-session-id"
}
GET /api/chat/context/:sessionId
Returns the saved in-memory conversation context for a session. This is useful for debugging follow-up behavior.

POST /api/recipes/paginate
Fetches another page for an already-detected intent.

Request body:

{
  "intent": "SEARCH_BY_TITLE",
  "params": {
    "title": "pasta"
  },
  "page": 2
}
GET /api/recipe/:id
Returns recipe details and instructions for a given recipe ID.

GET /api/recipe/:id/nutrition
Returns nutrition data for a given recipe ID.

External RecipeDB Endpoints Used
The local server wraps these external RecipeDB API routes:

Local server action	External RecipeDB route
Title search	/recipes/recipeByTitle
Calorie range	/recipes/calories
Carb range	/recipes/recipes-by-carbs
Protein range	/recipes/protein-range
Diet search	/recipes/recipe-diet
Region search	/recipes/recipes_cuisine/cuisine/:region
Region + diet	/recipes/region-diet
Category search	/recipes/category
Utensil search	/recipes/byutensils/utensils
Cooking method	/recipes/recipes-method/:method
Top rated	/recipes/top-rated
Random	/recipes/random
Meal plan	/recipes/meal-plan
Recipe details	/recipes/search-recipe
Instructions	/recipes/instructions/:id
Nutrition	/recipes/nutritioninfo/:id
API_QUICKSTART.md contains more examples for those backend-facing recipe queries.

Project Structure
BTP-RecipieDB/
|-- client/
|   |-- src/
|   |   |-- App.jsx                  # Main chat UI and state handling
|   |   |-- components/
|   |   |   |-- ChatMessage.jsx      # Renders chat bubbles and response blocks
|   |   |   |-- DataTable.jsx        # Paginated recipe result table
|   |   |   |-- RecipeModal.jsx      # Details, nutrition, and ingredient modal
|   |   |-- main.jsx                 # React entry point
|   |-- package.json
|   |-- vite.config.js
|   |-- tailwind.config.js
|
|-- server/
|   |-- index.js                     # Express routes and response orchestration
|   |-- llm.js                       # Rule-based intent detection
|   |-- recipedb.js                  # Wrapper for external RecipeDB API calls
|   |-- chatContextStore.js          # In-memory session storage
|   |-- config.js                    # Environment and base URL config
|   |-- package.json
|
|-- API_QUICKSTART.md                # Recipe API examples
|-- start.bat                        # Windows startup helper
|-- test-extraction.js               # Simple intent extraction smoke test
|-- README.md
Important Files and Responsibilities
server/index.js
defines the Express server
accepts chat requests
maps intents to backend API calls
exposes pagination, recipe detail, and nutrition routes
server/llm.js
extracts keywords from conversational prompts
matches recipe intents using regex and keyword dictionaries
supports contextual numeric follow-ups for calories, carbs, and protein
recognizes cooking methods and utensils using curated dictionaries
server/recipedb.js
centralizes all Axios requests to the external RecipeDB service
pads recipe IDs before detail/nutrition/instruction requests
returns safe fallback values ([] or null) on request failures
server/chatContextStore.js
creates session IDs with crypto.randomUUID()
stores chat history in memory
limits each session to 120 messages
client/src/App.jsx
stores chat threads and active thread state
sends chat requests to the server
handles pagination
manages modal selection for details, nutrition, and ingredients
client/src/components/RecipeModal.jsx
fetches recipe details or nutrition on demand
derives the ingredient view from the detail/instruction response
displays cooking steps, nutrient values, and metadata
Configuration Notes
Server defaults
Defined in server/config.js:

PORT=5000
RECIPEDB_BASE_URL=http://192.168.1.92:3033/api
GROQ_API_URL=https://api.groq.com/openai/v1/chat/completions
Client API target
The client currently uses hardcoded local API URLs:

client/src/App.jsx uses http://localhost:5000/api
client/src/components/RecipeModal.jsx uses http://localhost:5000/api/...
If the server runs on another host or port, those values must be updated or moved to environment-based frontend config.

Available Scripts
Root
node test-extraction.js - quick smoke test for keyword extraction and default title intent behavior
Server
npm start - starts the Express server
Client
npm run dev - starts the Vite development server
npm run build - builds the frontend for production
npm run lint - runs ESLint
npm run preview - previews the production build locally
Known Limitations
The RecipeDB data API is external to this repository and must be available separately.
Chat session state on the server is in memory only and does not survive restarts.
The frontend limits the UI to 5 saved chat threads.
start.bat is a Windows convenience script and is not cross-platform.
Client API URLs are hardcoded to localhost:5000.
The project is presented as an AI assistant in the UI, but the current intent engine is heuristic and rule-based.
There is no automated backend test suite in this repository at the moment.
Development Notes
The client lint script currently runs successfully with npm run lint.
test-extraction.js can be used to sanity-check keyword cleanup in server/llm.js.
API_QUICKSTART.md is useful when you want concrete examples of the external recipe-search routes.
Summary
This codebase is best understood as a full-stack chatbot frontend over a separate RecipeDB service:

React handles the conversational UI
Express handles intent routing and session context
the RecipeDB backend returns recipe data
If you want to extend the project, the usual path is:

add or refine intent detection in server/llm.js
connect that intent to a backend fetch in server/recipedb.js
handle the response in server/index.js
render the result in the React client if a new UI shape is needed
