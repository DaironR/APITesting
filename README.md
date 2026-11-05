# API Testing POC — REST Assured + TestNG

A small, friendly proof-of-concept that shows how to automate **REST API tests** in Java.
It runs real tests against the free public API [ReqRes](https://reqres.in) and produces a
nice **Allure** report you can open in your browser.

If you've never touched Java automation before — don't worry. Just follow the steps below in order.

---

## What's inside

| Thing | What it does |
|-------|--------------|
| **REST Assured** | Sends HTTP requests and checks the responses |
| **TestNG** | Runs the tests and groups them into a suite |
| **Allure** | Turns test results into a pretty HTML report |
| **Maven** | Downloads libraries and runs everything |
| **GitHub Actions** | Runs the tests automatically every time you push code |

The tests cover the everyday API scenarios:
- **GET** — list users, get one user, handle "not found" (404), pagination
- **POST / PUT / PATCH / DELETE** — the full create-read-update-delete cycle
- **Login / Register** — including data-driven negative cases
- **JSON Schema validation** — make sure the response shape is correct
- **Response time** — fail if the API is too slow

---

## Before you start (one-time setup)

You only need **one** thing installed on your computer: **Java 21 or newer**.

### Step 1 — Check if you already have Java

Open a terminal and run:

```bash
java -version
```

- If you see something like `openjdk version "21..."` → you're good, skip to the next section.
- If you see `command not found` or a version **lower than 21** → install Java below.

### Step 2 — Install Java (only if needed)

**On macOS** (using [Homebrew](https://brew.sh)):
```bash
brew install openjdk@21
```

**On Windows / Linux:**
Download and install **Temurin 21** from [adoptium.net](https://adoptium.net/).

> You do **NOT** need to install Maven. This project ships with the "Maven Wrapper"
> (`mvnw`), which downloads the correct Maven version for you automatically.

---

## How to run the tests

### Step 1 — Get the project

Clone it (or download the ZIP and unzip it):

```bash
git clone <your-repo-url>
cd "API POC"
```

### Step 2 — Run the tests

**On macOS / Linux:**
```bash
./mvnw test
```

**On Windows:**
```bash
mvnw.cmd test
```

The first run takes a minute or two (it downloads the libraries). After that it's fast.

When it finishes, you should see:

```
Tests run: 15, Failures: 0, Errors: 0, Skipped: 0
BUILD SUCCESS
```

🎉 That's it — your API tests just ran!

---

## Viewing the Allure report

After running the tests, turn the results into a shiny report:

```bash
./mvnw allure:serve
```

This builds the report and **opens it in your browser automatically**. You'll see:
- A dashboard with pass/fail charts
- Each test with its full **request and response** attached
- Timings, categories, and history

> Prefer to just generate the report without opening it? Use:
> ```bash
> ./mvnw allure:report
> ```
> The HTML lands in `target/site/allure-maven-plugin/`.

---

## Running just some tests

Run a single test class:
```bash
./mvnw test -Dtest=AuthTest
```

Run everything but change a setting on the fly (e.g. a stricter response-time limit):
```bash
./mvnw test -Dresponse.time.threshold.ms=1500
```

---

## Project layout (the short tour)

```
API POC/
├── pom.xml                         # Project + dependency configuration
├── mvnw / mvnw.cmd                 # Maven Wrapper (so you don't install Maven)
├── .github/workflows/api-tests.yml # The GitHub Actions CI pipeline
│
├── src/main/java/org/dsg/api/
│   ├── config/ConfigManager.java   # Reads settings from config.properties
│   ├── specs/ApiSpecs.java         # Shared request/response setup (base URL, headers)
│   └── models/                     # Java classes that mirror the JSON bodies
│
└── src/test/
    ├── java/org/dsg/api/
    │   ├── base/BaseApiTest.java   # Common setup every test inherits
    │   └── tests/                  # The actual tests live here
    │       ├── GetUsersTest.java
    │       ├── UserCrudTest.java
    │       └── AuthTest.java
    └── resources/
        ├── config.properties       # Base URL, API key, thresholds — edit me!
        ├── testng.xml              # Which tests to run + parallel settings
        ├── allure.properties       # Allure output location
        └── schemas/                # JSON Schemas used for validation
```

**Want to change the target API or a setting?** Edit
`src/test/resources/config.properties` — no code changes needed.

> Note: ReqRes now requires a free API key header (`x-api-key: reqres-free-v1`).
> It's already set up in `config.properties`, so everything works out of the box.

### Heads-up: ReqRes has a daily limit

The free ReqRes demo allows **~40 requests per day per IP address**. This POC makes
about 15 requests per run, so if you run it several times in a row you may start
seeing failures with HTTP **429 (`rate_limit_exceeded`)**. This is **not a bug** in
the tests — the limit simply resets after 24 hours.

To make the suite more resilient, failed tests are **automatically retried up to 2 times**
(see `RetryAnalyzer`). You can tune this:

```bash
./mvnw test -Dtest.retry.count=3
```

If you need to run heavily without limits, create a free ReqRes account for your own
API key and update `api.key.value` in `config.properties`, or point the tests at a
different API by changing `base.uri`.

---

## Getting your own free ReqRes API key

A free ReqRes account gives you a **much higher daily request limit** than the shared
anonymous demo key, plus persistent data and auth features. Here's how to set it up.

### Step 1 — Create a free account

1. Go to [reqres.in](https://reqres.in) and click **Sign up**, or open the sign-up page
   directly: [app.reqres.in/sign-up](https://app.reqres.in/sign-up)
2. Enter your email and a password (or use Google/GitHub if offered)
3. Confirm your email if they send a verification link

### Step 2 — Find your API key

Once you're logged in, open your **dashboard / account settings**. Your personal API
key is listed there — it's a longer token, not the shared `reqres-free-v1` demo key.

### Step 3 — Use it in this project

Just update **one line** in the config — no code changes needed:

```properties
# src/test/resources/config.properties
api.key.value=YOUR_PERSONAL_API_KEY_HERE
```

Or override it at runtime without editing the file:

```bash
./mvnw test -Dapi.key.value=YOUR_PERSONAL_API_KEY_HERE
```

### Step 4 (for CI) — keep your key secret

Never commit your real key to the repository. For GitHub Actions, store it as a secret:

1. On GitHub, go to **Settings → Secrets and variables → Actions → New repository secret**
   - **Name:** `REQRES_API_KEY`
   - **Value:** your personal key
2. Reference it in `.github/workflows/api-tests.yml` when running the tests:

   ```yaml
         - name: Run tests
           run: ./mvnw test -Dapi.key.value=${{ secrets.REQRES_API_KEY }}
   ```

This way your key stays private and cloud runs use it automatically.

---

## Running in the cloud (GitHub Actions)

This project comes with a ready-made CI pipeline at
`.github/workflows/api-tests.yml`. Once you push the project to GitHub, it will
**automatically**:

1. Run on every push and pull request to `main`/`master`
2. Install Java 21
3. Run all the API tests
4. Build the Allure report
5. Upload the report and results as **downloadable artifacts**

### How to see the results on GitHub

1. Go to your repository on GitHub
2. Click the **Actions** tab
3. Open the latest run
4. Scroll to **Artifacts** and download **`allure-report`**
5. Unzip it and open `index.html` in your browser

You can also trigger a run manually anytime from the **Actions** tab
(the "Run workflow" button).

---

## Troubleshooting

| Problem | Fix |
|---------|-----|
| `command not found: java` | Java isn't installed — see the setup section above |
| `Unsupported class file major version` | Your Java is older than 21 — install Java 21 |
| `./mvnw: Permission denied` (macOS/Linux) | Run `chmod +x mvnw` once |
| `allure:serve` opens nothing | Make sure you ran `./mvnw test` first so results exist |
| Tests fail with `401` / `403` | ReqRes changed its API key — update it in `config.properties` |
| Many tests fail with `429` | You hit ReqRes's ~40 requests/day limit — wait 24h or use your own API key |

---

## Quick command cheat sheet

```bash
./mvnw test                    # Run all tests
./mvnw test -Dtest=AuthTest    # Run one test class
./mvnw allure:serve            # Build + open the report
./mvnw allure:report           # Build the report only
./mvnw clean                   # Delete old results (fresh start)
```

Happy testing! 🚀
