[![Open in GitHub Codespaces](https://github.com/codespaces/badge.svg)](https://codespaces.new/MicrosoftDocs/mslearn-advanced-copilot)

**Languages:** English | [Español](README.es.md)

# Apply advanced GitHub Copilot techniques
Discover new ways to leverage advanced GitHub Copilot techniques within a Python repository to implement an interactive HTML form and an Application Programming Interface (API) endpoint. 
Gain more practical experience by using this repository that contains a Python Web Application that hosts a Travel Weather API.

[![Powered by Awesome Copilot](https://img.shields.io/badge/Powered_by-Awesome_Copilot-blue?logo=githubcopilot)](https://aka.ms/awesome-github-copilot)

## Requirements

1. Enable your [GitHub Copilot service](https://github.com/github-copilot/signup)
1. Open [this repository with Codespaces](https://codespaces.new/MicrosoftDocs/mslearn-copilot-codespaces-python)

### Additional setup details

The original three-step exercise below is preserved, including its keyboard shortcuts and Copilot prompts. The following sections add instructions for starting the app, sending requests, and checking the results.

**Codespaces link clarification:** the original Requirements link above points to a different sample repository. For this lab, use the badge at the top of this README or [open this repository in Codespaces](https://codespaces.new/MicrosoftDocs/mslearn-advanced-copilot).

For a local setup, install Git, Python 3.10 or later, and VS Code with the Python and GitHub Copilot extensions. The dev container uses Python 3.10. Running the API requires no weather service account, API key, or database because it reads the included `weather.json`.

## Set up and start the API

Choose one of the following options. Run all terminal commands from the repository root, the folder containing `main.py` and `requirements.txt`.

### Option A: GitHub Codespaces

1. [Open this repository in Codespaces](https://codespaces.new/MicrosoftDocs/mslearn-advanced-copilot) and create a codespace.
2. Wait for the dev container to finish building. Its configuration in [`.devcontainer/devcontainer.json`](.devcontainer/devcontainer.json) installs the dependencies and starts Uvicorn automatically.
3. Open the **Ports** tab in VS Code, find port **8000**, and select **Open in Browser**. Append `/docs` to the forwarded URL if the API documentation does not open automatically.
4. If the server did not start, run this command in the terminal:

   ```bash
   /home/vscode/venv/bin/python -m uvicorn main:app --host 0.0.0.0 --port 8000 --reload
   ```

Do not start a second server if port 8000 is already serving the app. For terminal commands below, use `/home/vscode/venv/bin/python` if `python` points to a different environment. In browser examples, use your forwarded URL in place of `http://127.0.0.1:8000`; commands inside the codespace can use the local address.

### Option B: Run locally

Clone the repository and enter its directory. If you already have a checkout, open a terminal in that folder and skip this command block.

```bash
git clone https://github.com/MicrosoftDocs/mslearn-advanced-copilot.git
cd mslearn-advanced-copilot
```

Create and activate a virtual environment using the commands for your shell.

**Windows PowerShell:**

```powershell
python --version
python -m venv .venv
.\.venv\Scripts\Activate.ps1
```

**macOS / Linux (Bash or Zsh):**

```bash
python3 --version
python3 -m venv .venv
source .venv/bin/activate
```

With the virtual environment active, install the dependencies and start the server:

```bash
python -m pip install -r requirements.txt
python -m uvicorn main:app --host 127.0.0.1 --port 8000 --reload
```

Wait for `Application startup complete`, then open **http://127.0.0.1:8000/docs** in your browser. Keep this terminal running; use a second terminal for requests and tests. `--reload` restarts the app when you save Python files. Press **Ctrl+C** in the server terminal to stop it.

## Try the existing endpoints

Before changing any code, verify that the API works. These routes are already implemented in [`main.py`](main.py):

| Method | Path | Result |
| --- | --- | --- |
| `GET` | `/` | Redirects to `/docs` with HTTP `301`. |
| `GET` | `/countries` | Returns a JSON array of the available country names. |
| `GET` | `/countries/{country}/{city}/{month}` | Returns the monthly `high` and `low` values for a city. |

`GET /countries/{country}` is **not implemented yet**. Adding it is the exercise below. The monthly weather route already exists and serves a different purpose.

### What does `/countries/{country}/{city}/{month}` return?

This endpoint answers: **"What historical high and low temperature values are stored for this city in this month?"** It looks up one entry in [`weather.json`](weather.json) and returns that entry as JSON.

The parts in braces are placeholders. Replace them with actual values in the URL, without the braces:

| Path parameter | Meaning | Example |
| --- | --- | --- |
| `{country}` | The country containing the city. | `Spain` |
| `{city}` | A city available under that country in `weather.json`. | `Seville` |
| `{month}` | The month's English name, with the capitalization used in the dataset. | `January` |

For example, to look up **Seville, Spain, in January**, send a `GET` request to this URL, or open it directly in your browser while the server is running:

```text
http://127.0.0.1:8000/countries/Spain/Seville/January
```

In Codespaces, replace `http://127.0.0.1:8000` with your forwarded URL. No request body is needed. The response is HTTP **200** with:

```json
{"high": 61, "low": 41}
```

- `high`: the stored high temperature value for Seville in January, **61**.
- `low`: the stored low temperature value for Seville in January, **41**.

The app returns the values already in the file; it does not calculate an average or fetch current weather or a forecast. There is no year or day parameter: `January` selects the dataset's January entry, not a specific date. The dataset does not specify temperature units or the historical reference period.

### Test in your browser with Swagger UI

FastAPI provides an interactive API client at `/docs`. You do not need to install Postman or another HTTP client.

1. Open **http://127.0.0.1:8000/docs** (or your Codespaces forwarded URL followed by `/docs`).
2. Expand **GET /countries**, select **Try it out**, then **Execute**.
3. In **Server response**, confirm HTTP status **200** and this JSON body:

   ```json
   ["England", "France", "Germany", "Peru", "Portugal", "Italy", "Spain"]
   ```

4. Expand **GET /countries/{country}/{city}/{month}** and select **Try it out**.
5. Enter `Spain` for **country**, `Seville` for **city**, and `January` for **month**, then select **Execute**.
6. Confirm HTTP status **200** and this response body:

   ```json
   {"high": 61, "low": 41}
   ```

Use the English names and capitalization found in [`weather.json`](weather.json): `Spain`, `Seville`, and `January`, not `spain`, `Sevilla`, or `enero`. The existing monthly route does not handle missing data; an unknown country, city, or month currently causes an HTTP `500` error.

### Test from a terminal

Run these requests in a second terminal while the server is running. They perform the same `GET` requests as Swagger UI and require no request body or authentication.

**Windows PowerShell:**

```powershell
Invoke-RestMethod -Method Get -Uri "http://127.0.0.1:8000/countries"
Invoke-RestMethod -Method Get -Uri "http://127.0.0.1:8000/countries/Spain/Seville/January"
```

PowerShell displays the parsed response as objects. To inspect the HTTP status and raw JSON, use `curl.exe`:

```powershell
curl.exe -i "http://127.0.0.1:8000/countries/Spain/Seville/January"
```

**macOS / Linux / Codespaces:**

```bash
curl -i "http://127.0.0.1:8000/countries"
curl -i "http://127.0.0.1:8000/countries/Spain/Seville/January"
```

The `-i` option includes the response headers. Expect `HTTP/1.1 200 OK` followed by the JSON shown above.

## 💪🏽 Exercise
The current API is not exposing country/{country} which needs to be implemented to list cities. The route should allow only GET HTTP requests with a JSON response providing information from the historical high and low for that country, city, and given month.

As with any implementation, this addition should include at least one test function to work with the pytest runner and test framework. 

**Route clarification:** the route to add is `/countries/{country}`, as shown in the original example below. It lists the cities available for a country. Historical `high` and `low` values are already served by `/countries/{country}/{city}/{month}`; you do not need to replace that existing endpoint. For Spain, the new route should return HTTP `200` with `["Seville"]`.


### 🛠 Step 1: Add a new route 
On our first exercise we will create a new route in our API. Go to the main.py file, and by using the inline chat with the following command `ctrl` + `i` (on Windows) or  `cmd` + `i`(on Mac) ask GitHub Copilot to help you create a new API that shows you the cities of a country. 

`> Create a new route that exposes the cities of a country.`


This prompt should give you something similar like this:


```python
# Create a new route that exposes the cities of a country:
@app.get('/countries/{country}')
def cities(country: str):
    return list(data[country].keys())

```
Note: Try your new route and refine your prompt until the result is as desired.


#### Additional guidance: save and try the new route

Use the original inline-chat shortcut and prompt above. Place the generated route in `main.py` **before** the `# Generate the OpenAPI schema:` section, which calls `app.openapi()`. Otherwise, the endpoint can accept requests without appearing in `/docs`. Keep the existing routes and the existing `data` dictionary.

Save `main.py` and wait for Uvicorn to reload, then verify the route with Swagger UI:

1. Open **http://127.0.0.1:8000/docs**, or your Codespaces forwarded URL followed by `/docs`.
2. Refresh the page and expand **GET /countries/{country}**.
3. Select **Try it out**, enter `Spain` in **country**, and select **Execute**.
4. In **Server response**, check HTTP **200** and the JSON body `["Seville"]`.

You can also send the same request from a second terminal while the server is running.

**Windows PowerShell:**

```powershell
curl.exe -i "http://127.0.0.1:8000/countries/Spain"
```

**macOS / Linux / Codespaces:**

```bash
curl -i "http://127.0.0.1:8000/countries/Spain"
```

Expect `HTTP/1.1 200 OK` and `["Seville"]`. Before you implement the route, this URL returns `404` because it does not exist yet. The original route example assumes a country present in `weather.json`; use `Spain` with this capitalization for the lab.

### 🔎 Step 2: Create a test
Now that you have created a new route, let's create a test with Copilot Chat for this route that uses Spain as the country. Remember to select your code and ask Copilot Chat to help you with this specific API that we just have created.

`> /tests help me to create a new test for this route that uses Spain as the country.`

#### Additional guidance: choose the test framework

The first time you use `/tests`, VS Code may ask you to configure Python testing. This configuration supports the original prompt above.

1. In **Select a test framework/tool to enable**, choose **pytest — pytest framework**. If **unittest** is highlighted, click **pytest** or use the arrow keys to select it, then press **Enter**. This repository includes `pytest` in `requirements.txt` and uses functions named `test_...` in `test_main.py`.
2. If asked which folder contains the tests, select **`.` / Root directory**, the folder containing `test_main.py`. You do not need to create a `tests` folder.
3. With pytest, a filename-pattern prompt is normally unnecessary. Its default patterns, `test_*.py` and `*_test.py`, already include `test_main.py`. If you selected `unittest` and are asked for a pattern, cancel and reconfigure with pytest.
4. Open the **Testing** view (the beaker icon) and confirm that the existing `test_root` and `test_countries` tests are discovered. Save the new test so it can be discovered too.

If you selected the wrong framework, open the Command Palette (**Ctrl+Shift+P** on Windows/Linux or **Cmd+Shift+P** on Mac), run **Python: Configure Tests**, and choose **pytest** and the root folder. If necessary, check the workspace settings: `python.testing.pytestEnabled` should be `true` and `python.testing.unittestEnabled` should be `false`. See [Python testing in VS Code](https://code.visualstudio.com/docs/python/testing#_configure-tests).

If no tests appear, run **Python: Select Interpreter** from the Command Palette and select the project's environment: `/home/vscode/venv/bin/python` in Codespaces, or the local `.venv` interpreter. Then run **Test: Refresh Tests**. The interpreter must have the dependencies from `requirements.txt` installed; see the setup section above.

For the default filename patterns, see the [pytest test discovery documentation](https://docs.pytest.org/en/stable/how-to/usage.html).

![Copilot Chat image example](./images/ideascopilot.png)

**What does "try it" mean below?** Save the generated Python test in the project and run it with pytest, following [the save-and-run instructions below](#additional-guidance-save-and-run-the-test). Check that the request to `/countries/Spain` returns HTTP `200` and exactly `["Seville"]`. If the test passes, continue to Step 3; the corrective prompt below is an example for a failing test whose expected city list is incorrect.


Once Copilot has helped you to create your test, try it. If this is not functioning as expected, feel free to share those details with Copilot in the chat. For example:

`> This test is not quite right, it is not including cities that doesn't exist. Only Seville is part of the API.`


It should give you another solution. Keep trying until you achieve the desired result.


#### Additional guidance: save and run the test

##### Put the generated test in the existing file

Select the new route in `main.py`, including its `@app.get(...)` decorator and function body, and send the original `/tests` prompt above. Choose pytest and the root folder if VS Code asks you to configure testing.

Code displayed in Copilot Chat is a proposal; it may not have been added to a file. Use these steps to put the test in the project:

1. In VS Code's **Explorer**, open the **existing** [`test_main.py`](test_main.py) next to `main.py`. It already contains the imports, `client = TestClient(app)`, `test_root`, and `test_countries`.
2. Check whether Copilot has already inserted a new test in that file. If it has, review the changes and skip copying and pasting it again; continue with saving in item 5. Keep the two existing tests.
3. If the test is only in the chat, copy **only the new test function**, from its `def test_...` line through its final assertion. Do not copy the Markdown code fences or duplicate the imports and `client` setup already in the file. The example below names the function `test_cities_for_spain`.
4. Go to the end of `test_main.py`, after the complete `test_countries` function. Leave two blank lines and paste the new function there. Its `def` line must start at the beginning of the line, aligned with the other `def` lines; its body uses four spaces. It must not be inside `test_countries` or in `main.py`.

The new block to append is:

```python
def test_cities_for_spain():
    """Spain returns only the city included in the dataset."""
    response = client.get("/countries/Spain")
    assert response.status_code == 200
    assert response.json() == ["Seville"]
```

5. With the `test_main.py` editor tab active, press **Ctrl+S** on Windows/Linux or **Cmd+S** on Mac to save it. Also save `main.py` if your route changes are still unsaved. Leaving the code in the chat does not save a test.

The file should now contain its original imports and `client` setup, the two original test functions, and your new test function once. If Copilot used a different function name, use that name in the command below instead of `test_cities_for_spain`.

Before running it, compare the request URL with the original route in Step 1: `/countries/{country}`, so the Spain request is `/countries/Spain`. If Copilot suggested `/countries/{country}/cities`, review that difference against the original example before continuing; adding `/cities` changes the URL used by this lab.

This test checks three concrete things:

| Code | What it does or checks |
| --- | --- |
| `client.get("/countries/Spain")` | Sends a `GET` request to the new route for Spain using the app's test client. |
| `assert response.status_code == 200` | Checks that the request succeeded. |
| `assert response.json() == ["Seville"]` | Checks that the response is exactly the city list in `weather.json`, including no extra cities. |

##### Run the saved test and check the result

Open **Terminal > New Terminal** in VS Code so you can keep the server terminal open. Run the following commands from the repository root, the folder containing `main.py` and `test_main.py`, with the virtual environment from the setup section active.

First, run only the new test:

```bash
python -m pytest test_main.py::test_cities_for_spain -v
```

In Codespaces, you can call the configured interpreter directly:

```bash
/home/vscode/venv/bin/python -m pytest test_main.py::test_cities_for_spain -v
```

Expect a line containing `test_main.py::test_cities_for_spain PASSED` and a summary containing **`1 passed`**. Here, `test_main.py::test_cities_for_spain` selects one function in one file; it does not run the whole test suite.

Then check that the existing tests still pass:

```bash
python -m pytest -q
```

In Codespaces, use `/home/vscode/venv/bin/python -m pytest -q` if needed. The starting repository has **2 tests**. With the single new test above, the full run should report **`3 passed`**. If Copilot added more tests, the total will be higher.

Run these commands with pytest; using **Run Python File** on `test_main.py` does not execute these test functions. `TestClient` runs the app in process, so the tests do not require Uvicorn to be running or a forwarded port.

##### If the test fails, use the actual output to refine it

Read the terminal output before sending a follow-up prompt. A failure can come from the route, the generated test, or the environment. Use these checks to identify which one needs attention:

| Result in the terminal | What to check next |
| --- | --- |
| `not found` for the requested test | Save `test_main.py` and check that its function name matches the name after `::` in the command. |
| Only `2 passed` when running the full suite | Confirm that the new function was saved in `test_main.py`, that its name starts with `test_`, and that it is not nested inside another function. |
| The status assertion shows `404` instead of `200` | Save `main.py` and check that the new route exists at `/countries/{country}` and the test requests `/countries/Spain`. |
| The JSON assertion shows different city lists | Compare the actual response and the expected list with `weather.json`. Spain contains only `Seville`; a generated expectation containing `Madrid`, for example, is incorrect. |
| `KeyError` | Check the data keys and capitalization, especially `Spain`. `TestClient` can show the app's exception directly in the traceback. |
| `IndentationError` | Align the new function's `def` with the existing functions and indent its body by four spaces. |
| `ModuleNotFoundError` for a dependency | Use the project's virtual environment and install `requirements.txt` as described in setup. |

For example, if the generated test expects extra cities, use the original corrective prompt above and include the evidence. A more detailed follow-up can be:

```text
I ran: python -m pytest test_main.py::test_cities_for_spain -v
Expected behavior: GET /countries/Spain returns status 200 and ["Seville"].
Actual failure:
[Paste the failing assertion and relevant traceback here.]
Review main.py, test_main.py, and weather.json. Explain whether the route
or the assertion needs correcting, and propose the change.
```

Review the proposed correction, save the affected file, and rerun the single-test command. Once it passes, rerun the full suite. Continue to Step 3 when the new test and the existing tests pass.

### 🐍 Step 3: Use an agent to write the project
During this step we will be using an agent (workspace) to write the project documentation on how to run this project. In the GitHub Copilot Chat, we will try the following prompt:

`> @workspace help me to use an agent to write the project documentation on how to run it .`


#### Additional guidance: review the generated documentation

Use the original `@workspace` prompt above. Review the proposed documentation against `main.py`, `requirements.txt`, and `test_main.py`. It should explain where to run the commands, how to install dependencies and start Uvicorn, and how to send a request and compare its response with the expected JSON. The setup and verification sections in this README provide concrete examples to check against.

Finally, verify the new endpoint is working by trying it out by going to the `/docs` endpoint and confirming that the endpoint shows up.

For this final check, refresh `/docs`, expand **GET /countries/{country}**, select **Try it out**, enter `Spain`, and select **Execute**. Confirm HTTP **200** and `["Seville"]`, then run `python -m pytest -q` to confirm that the existing tests and your new test pass.



🚀 Congratulations, through the exercise, you haven't only used copilot to generate code but also done it in an interactive and fun way! You can use GitHub Copilot to not only generate code, but write documentation, test your applications and more.



## Troubleshooting

| Symptom | What to check or do |
| --- | --- |
| `python` is not found on Windows | Try `py --version`, then use `py` to create the virtual environment. Once activated, use `python` for the remaining commands. |
| PowerShell blocks `Activate.ps1` | You can skip activation and call the environment directly: `.\.venv\Scripts\python.exe -m pip install -r requirements.txt`, then `.\.venv\Scripts\python.exe -m uvicorn main:app --host 127.0.0.1 --port 8000 --reload`. For tests, use `.\.venv\Scripts\python.exe -m pytest -q`. |
| `No module named fastapi`, `uvicorn`, or `pytest` | Activate the virtual environment and run `python -m pip install -r requirements.txt`. In Codespaces, use `/home/vscode/venv/bin/python`. |
| `Could not import module "main"` or `No module named webapp` | Run from the folder containing `main.py` and use `main:app`. This repository has no `webapp` package. |
| Port 8000 is already in use | Check whether the API is already running. Stop your previous server with Ctrl+C, or use `--port 8001` and change the example URLs to port 8001. In Codespaces, forward that port in the Ports tab. |
| Browser or curl cannot connect | Keep the Uvicorn terminal open and check its startup output for errors. In Codespaces, open port 8000 through the Ports tab. |
| `/countries/Spain` returns `404` | Complete Step 1, save `main.py`, and wait for reload. This route is absent in the starting code. |
| The new route is missing from `/docs` | Put its definition before `app.openapi()`, restart the server if needed, and refresh the browser. Use the running app's `/docs`; the JSON schema files in the repository may be stale. |
| A weather request returns `500` | Check exact country, city, and English month names in `weather.json`, then inspect the server traceback. Missing-data handling for the monthly route is not implemented. |

## Project files

| File | Purpose |
| --- | --- |
| [`main.py`](main.py) | FastAPI application, routes, and schema generation. |
| [`weather.json`](weather.json) | Country, city, and monthly weather data used by the API. |
| [`test_main.py`](test_main.py) | Automated tests using pytest and FastAPI's `TestClient`. |
| [`requirements.txt`](requirements.txt) | Python dependencies for running and testing the app. |
| [`.devcontainer/`](.devcontainer/) | Codespaces and VS Code dev container setup. |

# Legal Notices

Microsoft and any contributors grant you a license to the Microsoft documentation and other content
in this repository under the [Creative Commons Attribution 4.0 International Public License](https://creativecommons.org/licenses/by/4.0/legalcode),
see the [LICENSE](LICENSE) file, and grant you a license to any code in the repository under the [MIT License](https://opensource.org/licenses/MIT), see the
[LICENSE-CODE](LICENSE-CODE) file.

Microsoft, Windows, Microsoft Azure and/or other Microsoft products and services referenced in the documentation
may be either trademarks or registered trademarks of Microsoft in the United States and/or other countries.
The licenses for this project do not grant you rights to use any Microsoft names, logos, or trademarks.
Microsoft's general trademark guidelines can be found at http://go.microsoft.com/fwlink/?LinkID=254653.

Privacy information can be found at https://privacy.microsoft.com/en-us/

Microsoft and any contributors reserve all other rights, whether under their respective copyrights, patents,
or trademarks, whether by implication, estoppel or otherwise.
