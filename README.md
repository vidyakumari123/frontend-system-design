# 🚀 Frontend System Design Yatra (JSCafe)

Welcome to the **Frontend System Design** learning repository! This workspace is your master guide for learning how large-scale web applications communicate (Networking) and run incredibly fast (Performance).

Instead of searching through multiple files, this single document contains **each and everything you need to know**:
1. **🛠️ Prerequisites**: What to install on your computer.
2. **🏃 Quick Running Guide**: How to run any project in this repository.
3. **🌐 Section 1: Networking & Communication**: Detailed code walkthrough, run guide, and interview tricks for all 7 topics.
4. **⚡ Section 2: Performance & Core Web Vitals**: Detailed code walkthrough, run guide, and interview tricks for all 11 topics.
5. **🏆 Senior Engineer Pro-Tips**: Advanced knowledge that separates strong senior developers from juniors.
6. **🎓 Master Interview Cheat Sheet**: Quick reference guide.

---

## 🛠️ Prerequisites: What You Need to Install

Before running the code, make sure you have these free tools installed:

### 1. Node.js (and npm)
* **What is it?** Node.js is like the engine that runs JavaScript on your computer (instead of inside a browser). **npm** (Node Package Manager) comes with it and is used to download libraries for the projects.
* **How to install**: Go to [nodejs.org](https://nodejs.org/), download the **LTS** version, and run the installer.
* **Check if installed**: Open terminal and type `node -v` and `npm -v`.

### 2. A Code Editor (VS Code)
* **What is it?** The best free text editor for writing code. Download from [code.visualstudio.com](https://code.visualstudio.com/).

### 3. VS Code "Live Server" Extension (Optional)
* **What is it?** A tool that runs a mini web server so you can view static HTML files in your browser.
* **How to install**: Inside VS Code, search for **"Live Server"** in the Extensions tab and click **Install**.

---

## 🌐 Section 1: Networking & Communication (Deep Dive)

This section shows how a frontend application requests and receives data from a backend server.

---

### 1. REST API (`networking/rest`)
* **Concept in Simple Words**: A standard way for a client to request data. It uses verbs: GET (fetch data), POST (create data), PUT (replace data), PATCH (update data), and DELETE (remove data).
* **Code Walkthrough (`server.js`)**:
  * Uses `express` to spin up a server on port `3000`.
  * Keeps an array of `problems` in memory.
  * Demonstrates the difference between HTTP verbs:
    * `app.get("/api/problems")`: Returns the entire list.
    * `app.put("/api/problems/:id")`: Overwrites the entire problem object with the new input.
    * `app.patch("/api/problems/:id")`: Merges only the changed fields with the existing problem object.
* **How to Run**:
  ```bash
  cd networking/rest
  npm install
  npm start
  ```
  Open `http://localhost:3000/api/problems` in your browser.
* **🎓 Interview Tricks**:
  * **Idempotency**: An operation is *idempotent* if calling it multiple times has the same result. **GET, PUT, and DELETE are idempotent**. **POST is NOT idempotent** (calling it twice creates two items). **PATCH is generally NOT idempotent**.
  * **HTTP Status Codes**: Know the categories: `2xx` (Success), `3xx` (Redirection), `4xx` (Client Error, e.g. 404), and `5xx` (Server Error).

---

### 2. Short Polling (`networking/short-polling`)
* **Concept in Simple Words**: The client repeatedly asks the server *"Do you have new data?"* every few seconds (using `setInterval`).
* **Code Walkthrough**:
  * **Server (`server.js`)**: An Express server running on port `3000` exposing a `/api/problems` endpoint that returns static data.
  * **Client (`index.html`)**: Has a `fetchData` function wrapped in a `setInterval` that fires every **2000ms (2 seconds)**:
    ```javascript
    setInterval(() => {
      fetchData();
    }, 2000);
    ```
* **How to Run**:
  * **Server**:
    ```bash
    cd networking/short-polling/server
    npm install
    npm start
    ```
  * **Client**: Open `networking/short-polling/client/index.html` directly in your browser.
* **🎓 Interview Tricks**:
  * **The Pitfall**: High server overhead. If 10,000 clients poll every 2 seconds, the server gets 5,000 requests per second even if no data has changed.
  * **Thundering Herd**: If all clients poll at the exact same millisecond, the server crashes. Fix it using **Jitter** (adding a small random delay to each client's poll interval).

---

### 3. Long Polling (`networking/long-polling`)
* **Concept in Simple Words**: The client asks for data. The server holds the request open (doesn't reply) until it has new data. As soon as the client gets the reply, it immediately starts a new request to wait again.
* **Code Walkthrough**:
  * **Server (`server.js`)**: Uses Node's `events.EventEmitter`. When a GET request to `/messages` comes in, it subscribes to a `newMessage` event and **does not call `res.json()` yet**, keeping the connection pending:
    ```javascript
    messageEventEmitter.once("newMessage", (from, message) => {
      res.json({ from, message });
    });
    ```
    When a POST request to `/new-message` is made, it fires `.emit("newMessage")`, which resolves the pending GET request.
  * **Client (`index.html`)**: Employs a recursive asynchronous loop. As soon as `fetch` receives a message, it immediately calls itself again (`makeRequest()`) to wait for the next event:
    ```javascript
    const makeRequest = async () => {
      const res = await fetch("http://localhost:3000/messages");
      const data = await res.json();
      makeRequest(); // Immediately starts a new connection
    };
    ```
* **How to Run**:
  * **Server**:
    ```bash
    cd networking/long-polling/server
    npm install
    npm start
    ```
  * **Client**: Open `networking/long-polling/client/index.html` in your browser.
* **🎓 Interview Tricks**:
  * **HTTP Timeouts**: HTTP connections cannot stay open forever. The client must detect timeouts (usually 30-60 seconds) and reconnect automatically.
  * **Server Memory**: Holding connections open uses resources. In thread-per-request architectures, this is expensive. In Node.js (event-driven), it's highly efficient because connections are handled asynchronously.

---

### 4. WebSockets (`networking/websockets`)
* **Concept in Simple Words**: Creates a single, permanent, bi-directional connection between client and server. Both can send messages at any time.
* **Code Walkthrough**:
  * **Server (`server.js`)**: Standard Express server on port `3000` combined with a separate `ws` server running on port `8080`. When a message is received from any client, it iterates over all connected clients and broadcasts it:
    ```javascript
    wss.clients.forEach((client) => {
      if (client.readyState === WebSocket.OPEN) {
        client.send(message.toString());
      }
    });
    ```
  * **Client (`index.html`)**: Creates a WebSocket connection using native browser APIs (`new WebSocket("ws://localhost:8080")`). Listens to `"message"` events to append chat messages dynamically to the screen.
* **How to Run**:
  * **Server**:
    ```bash
    cd networking/websockets/server
    npm install
    npm start
    ```
  * **Client**: Open `networking/websockets/client/index.html` in your browser.
* **🎓 Interview Tricks**:
  * **Stateful Connections**: WebSockets are stateful. A client is connected to a *specific* server instance. 
  * **Scaling WS**: If you have 5 backend servers, Client A on Server 1 cannot talk to Client B on Server 2. You must use a **Redis Pub/Sub** message broker to broadcast messages across all server instances.
  * **Heartbeats**: To prevent routers from closing idle connections, implement a "Ping/Pong" heartbeat every 30 seconds.

---

### 5. GraphQL (`networking/graphql`)
* **Concept in Simple Words**: Instead of having multiple endpoints, GraphQL has one endpoint. The client sends a query specifying exactly what data it wants, and the server returns only that.
* **Code Walkthrough**:
  * **Server (`server.js`)**: Uses `ApolloServer`. Defines GraphQL schema types (e.g. `User`, `Problem`) and a relational mapping:
    ```javascript
    User: {
      problems: (user) => problems.filter(p => p.solvers.includes(user.id))
    }
    ```
  * **Client (`index.html`)**: Sends a standard HTTP POST request to `http://localhost:4000/` containing the exact JSON query structure:
    ```javascript
    body: JSON.stringify({
      query: "{users { id, name, problems {id, title} }}"
    })
    ```
* **How to Run**:
  * **Server**:
    ```bash
    cd networking/graphql/server
    npm install
    npm start
    ```
  * **Client**: Open `networking/graphql/client/index.html` in your browser.
* **🎓 Interview Tricks**:
  * **Over-fetching vs Under-fetching**: REST often returns too much data (over-fetching) or requires multiple API calls to get nested data (under-fetching). GraphQL solves both.
  * **N+1 Database Query Problem**: If resolving a list of users and their problems, a naive resolver might query the database once for users, and then N times for each user's problems. Explain how to solve this using **DataLoader** (which batches and caches database requests).
  * **Caching Difficulty**: Since all GraphQL requests are HTTP `POST` requests, standard browser and CDN caching (which relies on `GET` URLs) doesn't work. Use Persisted Queries or client-side cache stores (like Apollo Client).

---

### 6. tRPC (`networking/trpc`)
* **Concept in Simple Words**: Allows sharing TypeScript types directly between backend and frontend. You get auto-complete for your API routes in your React code.
* **Code Walkthrough**:
  * **Server (`index.ts`)**: Defines an Express-tRPC server. Initializes tRPC router procedures:
    ```typescript
    const appRouter = router({
      getAllProblems: publicProcedure.query(() => problems),
    });
    export type AppRouter = typeof appRouter;
    ```
  * **Client (`main.ts`)**: Imports the backend's type signature (`import type { AppRouter } from "../../server/index"`) and uses `createTRPCClient<AppRouter>` to call procedures as simple object methods with full autocomplete and type-checking.
* **How to Run**:
  * **Server**:
    ```bash
    cd networking/trpc/server
    npm install
    npm start
    ```
  * **Client**:
    ```bash
    cd networking/trpc/client
    npm install
    npm run dev
    ```
* **🎓 Interview Tricks**:
  * **Zero Code Generation**: tRPC does not compile or generate client code (unlike GraphQL or gRPC). It simply imports types from the backend source.
  * **Constraint**: Ideal for monorepos where both backend and frontend are written in TypeScript. It is not suitable if your backend is written in Python/Go/Java.

---

### 7. gRPC (`networking/grpc`)
* **Concept in Simple Words**: A highly performant API framework that uses Google's binary format (**Protocol Buffers**) instead of JSON to send data over **HTTP/2**.
* **Code Walkthrough**:
  * **Server (`server.js`)**: Starts a gRPC server on port `50051`. Loads API schemas from `problems.proto`.
  * **Client Bridge (`client.js`)**: Starts an Express server on port `3000` to serve as a translation bridge. It takes HTTP JSON requests from browser clients, converts them into gRPC format, calls the backend gRPC server, and returns JSON back to the browser.
* **How to Run**:
  * **Server**:
    ```bash
    cd networking/grpc
    npm install
    npm start
    ```
  * **Client Bridge**:
    ```bash
    cd networking/grpc
    node client.js
    ```
* **🎓 Interview Tricks**:
  * **HTTP/2 Multiplexing**: gRPC uses HTTP/2, allowing multiple requests to be sent concurrently over a single TCP connection.
  * **Protocol Buffers**: ProtoBufs compress data into small binary packages, making it much faster to transmit than large JSON text strings.
  * **Browser Limitation**: Web browsers cannot make direct native gRPC requests because they lack control over HTTP/2 frame headers. You must use a proxy like **gRPC-Web** or an Express API bridge (as shown in `client.js`).

---

## ⚡ Section 2: Performance & Core Web Vitals (Deep Dive)

This section covers how to make web pages load faster, feel smoother, and respond instantly to user interactions.

---

### 1. Cumulative Layout Shift (`performance/CLS`)
* **Concept in Simple Words**: CLS measures how much layout elements move around unexpectedly while loading.
* **Code Walkthrough (`src/App.jsx`)**:
  * Layout shifts are simulated by toggling `showAds` after a `setTimeout` of 100ms.
  * Notice that the layout shift is prevented because the dynamic advertisement image is loaded inside a wrapper with a fixed height:
    ```jsx
    <div style={{ height: "500px" }}>
      {showAds && <img src="cat.jpeg" />}
    </div>
    ```
    Even before the image loads, the browser allocates exactly `500px` of space, so the paragraph text below it does not shift.
* **How to Run**:
  ```bash
  cd performance/CLS
  npm install
  npm run dev
  ```
* **🎓 Interview Tricks**:
  * **Target Score**: **Under 0.1** is good.
  * **Fix**: Always reserve space for late-loading items (images, ads, widgets) by setting explicit `width` and `height` properties or using the CSS `aspect-ratio` property on their parent container.
  * **Animations**: Avoid animating CSS properties like `top`, `left`, `margin` because they trigger browser reflow. Use `transform: translate()` instead.

---

### 2. Interaction to Next Paint (`performance/INP`)
* **Concept in Simple Words**: Measures how fast a page gives a visual response after a user clicks or presses a key.
* **Code Walkthrough (`src/App.jsx`)**:
  * Demonstrates poor INP by running a long CPU-blocking task on button click before displaying the modal:
    ```jsx
    const handleClick = () => {
      for (let i = 0; i < 10000000; i++) {} // Blocks the main thread
      toggleModal((prev) => !prev);
    };
    ```
    This freezes the main JavaScript thread, blocking the browser from rendering the modal immediately.
* **How to Run**:
  ```bash
  cd performance/INP
  npm install
  npm run dev
  ```
* **🎓 Interview Tricks**:
  * **Target Score**: **Under 200 milliseconds** is good.
  * **Fix**: Break up long-running JavaScript execution (tasks over 50ms) using `setTimeout(..., 0)` or `requestIdleCallback` to allow the browser to paint the screen between executions.
  * **Web Workers**: Move complex, non-UI math operations off the main thread into a Web Worker.

---

### 3. Largest Contentful Paint (`performance/LCP`)
* **Concept in Simple Words**: Measures how long it takes for the largest visual block (like a hero banner image) to display on the screen.
* **Code Walkthrough (`index.html`)**:
  * Uses `srcset` responsive images to optimize LCP by loading a smaller file on mobile devices and a larger file on wider displays:
    ```html
    <img src="cat-1000.jpeg" srcset="cat-500.jpeg 1000w" />
    ```
* **How to Run**: Open `performance/LCP/index.html` in your browser.
* **🎓 Interview Tricks**:
  * **Target Score**: **Under 2.5 seconds** is good.
  * **Preload Hero**: Use `<link rel="preload">` in the HTML head and set `fetchpriority="high"` on the hero image tag to download it first.
  * **Never Lazy Load the Hero**: Adding `loading="lazy"` to a hero image delays it and harms LCP.
  * **Responsive Images**: Use `srcset` to serve smaller images to mobile screens.

---

### 4. Code Splitting (`performance/code-splitting`)
* **Concept in Simple Words**: Breaks a single massive JavaScript bundle into smaller files. The browser only loads the bundle for the page the user is currently viewing.
* **Code Walkthrough (`src/App.jsx`)**:
  * Imports routes using React's dynamic `lazy` compiler hook:
    ```jsx
    const Profile = lazy(() => import("./pages/Profile"));
    const About = lazy(() => import("./pages/About"));
    ```
    Wraps them inside `<Suspense fallback={<>...</>}>` to display a loader while loading pages.
* **How to Run**:
  ```bash
  cd performance/code-splitting
  npm install
  npm run dev
  ```
* **🎓 Interview Tricks**:
  * **React Implementation**: Use `React.lazy()` with dynamic `import()` wrapped in `<Suspense>`.
  * **Route-based vs Component-based**: Route-based splits the app by pages (e.g. login, dashboard). Component-based splits heavy hidden widgets (e.g. dynamic charts, rich-text editors).

---

### 5. Compression (`performance/compression`)
* **Concept in Simple Words**: Compressing files (like making a zip file) before they go over the internet.
* **Code Walkthrough (`webpack.config.js`)**:
  * Uses `compression-webpack-plugin` inside the Webpack compilation pipeline:
    ```javascript
    const CompressionPlugin = require("compression-webpack-plugin");
    plugins: [new CompressionPlugin({ algorithm: "gzip" })]
    ```
    This outputs `.gz` gzip-compressed bundles during building, which servers can serve to the browser.
* **How to Run**:
  ```bash
  cd performance/compression
  npm install
  npm run build
  ```
  *(Check the `dist` folder to see the generated `.gz` compressed files!)*
* **🎓 Interview Tricks**:
  * **Brotli vs Gzip**: Brotli is Google's compression algorithm. It is **15-20% smaller** than Gzip and supported by all modern browsers.
  * **Headers**: The browser tells the server what compression it supports using the `Accept-Encoding: gzip, deflate, br` header. The server responds with `Content-Encoding: br` (for Brotli) or `gzip`.

---

### 6. Import on Interaction (`performance/import-on-interaction`)
* **Concept in Simple Words**: Don't download the code for a feature until the user clicks or interacts with it.
* **Code Walkthrough (`src/App.jsx`)**:
  * Dynamic loads the component inside the application:
    ```jsx
    const Emoji = lazy(() => import("./Emoji"));
    ```
    The `Emoji` bundle is only requested when `showEmoji` state toggles to `true` (triggered by the user clicking "Show Emoji").
* **How to Run**:
  ```bash
  cd performance/import-on-interaction
  npm install
  npm run dev
  ```
* **🎓 Interview Tricks**:
  * **Use Cases**: Emoji picker, chat window, print button, payment portals.
  * **Why**: Reduces the initial bundle size, allowing the page to load and become interactive much faster.

---

### 7. Import on Visibility (`performance/import-on-visibility`)
* **Concept in Simple Words**: Don't download a component's code until the user scrolls down and the component actually comes onto the screen.
* **Code Walkthrough (`src/App.jsx`)**:
  * Monitors visibility using `react-intersection-observer`:
    ```jsx
    const { ref, inView } = useInView();
    ```
    Wraps the lazy loading component:
    ```jsx
    <div ref={ref}>
      <Suspense fallback="Loading">{inView && <Emoji />}</Suspense>
    </div>
    ```
    The browser only sends a request to load the `Emoji` chunk once `inView` is true.
* **How to Run**:
  ```bash
  cd performance/import-on-visibility
  npm install
  npm run dev
  ```
* **🎓 Interview Tricks**:
  * **Intersection Observer API**: Implement this using the browser's native `IntersectionObserver` API instead of listening to window `scroll` events (which can lag the page).
  * **Use Cases**: User reviews, footer widgets, comments, or related products sections.

---

### 8. Prefetch (`performance/prefetch`)
* **Concept in Simple Words**: Downloading code in the background during the browser's **idle time**, because the user *might* visit that page next.
* **Code Walkthrough (`src/App.js`)**:
  * Uses the magic comment inside dynamic import statements:
    ```javascript
    import(/* webpackPrefetch: true, webpackChunkName: "emoji" */ "./Emoji")
    ```
    This signals Webpack to append `<link rel="prefetch" href="emoji.js">` to the head of the document.
* **How to Run**:
  ```bash
  cd performance/prefetch
  npm install
  npm run dev
  ```
* **🎓 Interview Tricks**:
  * **Syntax**: In Webpack, use the magic comment `/* webpackPrefetch: true */`.
  * **Scenario**: Ideal for the "Next Step" button inside a multi-step checkout form.

---

### 9. Preload (`performance/preload`)
* **Concept in Simple Words**: Forcing the browser to download a critical resource **immediately** because it is needed for the current page.
* **Code Walkthrough (`webpack.config.js`)**:
  * Utilizes `@vue/preload-webpack-plugin`:
    ```javascript
    const PreloadWebpackPlugin = require("@vue/preload-webpack-plugin");
    plugins: [new PreloadWebpackPlugin()]
    ```
    This scans compiled scripts and injects `<link rel="preload">` tags inside the output html, prompting the browser to start downloads right away.
* **How to Run**:
  ```bash
  cd performance/preload
  npm install
  npm run dev
  ```
* **🎓 Interview Tricks**:
  * **Double Downloads**: Ensure headers match (like `crossorigin` on preloaded fonts). Otherwise, the browser may download the file twice.
  * **Scenario**: Custom font files or hero images that are loaded via CSS and aren't visible in the initial HTML scan.

---

### 10. Tree Shaking (`performance/tree-shaking`)
* **Concept in Simple Words**: Removing unused JavaScript code from your production bundle during the build step.
* **Code Walkthrough**:
  * `math.js` exports both `sum` and `multiply`.
  * `index.js` imports only `multiply`:
    ```javascript
    import { multiply } from "./math";
    ```
    Webpack builds it with minification. Since `sum` is unused, Webpack deletes it, preventing it from ending up in the final `dist/main.js` bundle.
* **How to Run**:
  ```bash
  cd performance/tree-shaking
  npm install
  npm run build
  ```
  *(Notice that the unused `sum` function in `math.js` is stripped out of the final bundle!)*
* **🎓 Interview Tricks**:
  * **ES Modules Requirement**: Tree shaking **only works with ESM** (`import` / `export`). It cannot work with CommonJS (`require`).
  * **Side Effects**: Add `"sideEffects": false` to a library's `package.json` to tell the bundler it is safe to remove unused exports.

---

### 11. List Virtualization (`performance/virtualization`)
* **Concept in Simple Words**: Only rendering the list elements that are currently visible on the screen, rather than thousands of hidden items off-screen.
* **Code Walkthrough (`src/App.jsx`)**:
  * Uses `react-window` library:
    ```jsx
    import { FixedSizeList as List } from "react-window";
    <List height={150} itemCount={1000} itemSize={35} width={300}>
      {Row}
    </List>
    ```
    Instead of drawing 1000 elements, the browser only creates enough list row elements to cover the `150px` height (approx. 5 elements) and swaps their text content during scroll.
* **How to Run**:
  ```bash
  cd performance/virtualization
  npm install
  npm run dev
  ```
* **🎓 Interview Tricks**:
  * **The Problem**: Rendering 10,000 DOM nodes eats up browser memory and slows down scrolling frames.
  * **Libraries**: Mention React libraries like **`react-window`** or **`react-virtualized`**.
  * **Mechanics**: Absolute positions elements based on scroll offset (`transform: translateY()`), creating a fake scrollbar height.

---

## 🏆 Senior Engineer Pro-Tips (How to Stand Out)

To stand out in high-level system design interviews, you need to understand engineering concepts beyond simple definitions. Mention these architectural patterns to impress interviewers:

### 1. HTTP/1.1 vs HTTP/2 vs HTTP/3 (Under the Hood)
* **HTTP/1.1**: Suffered from **Head-of-Line (HOL) Blocking** at the application layer. The browser could only open ~6 parallel TCP connections to a domain. If one request was slow, the others behind it had to wait.
* **HTTP/2**: Introduced **Multiplexing** over a single TCP connection. Multiple requests are split into binary frames and sent concurrently. However, it still suffers from **TCP Head-of-Line Blocking**—if one TCP packet is lost, all streams pause until it is retransmitted.
* **HTTP/3**: Solves this completely by switching from TCP to **UDP** using Google’s **QUIC protocol**. Packet loss in one stream does not block other streams.

### 2. Modern Rendering Paradigms
* **CSR (Client-Side Rendering)**: Slow Initial Page Load (high LCP) because the browser downloads an empty HTML file and waits for a huge JS bundle to render the content. Great for interactive dashboards.
* **SSR (Server-Side Rendering)**: Fast initial render (good LCP) but slow Time-To-First-Byte (TTFB) because the server dynamically generates HTML on every request. High server load.
* **SSG (Static Site Generation)**: Extremely fast render and cheap hosting (CDN). HTML is pre-built at compile time. Hard to use if data changes constantly.
* **RSC (React Server Components)**: A hybrid approach where components render on the server and stream their UI representation directly to the client without shipping any JavaScript bundle size for those components.

### 3. Secure Token Storage (Web Security)
* **Where to store auth JWT tokens?**
  * **LocalStorage**: **Highly Vulnerable** to XSS (Cross-Site Scripting). Any injected 3rd-party script can call `localStorage.getItem('token')` and steal it.
  * **HttpOnly Cookies**: **Secure against XSS** because JavaScript cannot read them. However, they are vulnerable to **CSRF (Cross-Site Request Forgery)**. Protect against CSRF by adding the `SameSite=Strict` cookie attribute and validating Anti-CSRF tokens.

### 4. Telemetry and Real User Monitoring (RUM)
* Do not just talk about running Google Lighthouse on your laptop. Lighthouse runs in synthetic lab environments.
* In production, capture actual user performance metrics using **RUM**. Explain how to write a `PerformanceObserver` instance in JavaScript to report metrics like **CLS** and **INP** directly from real user browsers to your analytics servers:
  ```javascript
  new PerformanceObserver((entryList) => {
    for (const entry of entryList.getEntries()) {
      console.log('Layout Shift detected: ', entry.value);
    }
  }).observe({ type: 'layout-shift', buffered: true });
  ```

---

## 🎯 Master Cheat Sheet for Frontend System Design Interviews

| System Design Choice | When to Choose it? | Trade-offs to Mention |
| :--- | :--- | :--- |
| **Short Polling** | Easiest to build, low real-time need. | Wastes network resources, doesn't scale. |
| **Long Polling** | Better than short polling, fallback. | Keeps connections hanging, uses server memory. |
| **WebSockets** | High-frequency, low-latency updates (e.g. Chat). | Stateful, needs Redis to scale to multiple servers. |
| **GraphQL** | Client needs flexible data formats. | Complex queries can slow down databases; caching is hard. |
| **gRPC** | Backend-to-backend communication. | Cannot run natively in web browsers without a proxy. |
| **Code Splitting** | Heavy single-page apps with many routes. | Adds slight latency when switching routes first time. |
| **Preload** | Assets needed on the *current* page immediately. | If overused, delays loading other key assets. |
| **Prefetch** | Assets needed on the *next* page soon. | Wastes user data if they don't go to that page. |
| **Virtualization** | Infinite scrolling feeds, giant tables. | Complex implementation; hard to do with dynamic item heights. |
