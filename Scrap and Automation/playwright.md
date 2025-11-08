## 01. Documentation
```
https://github.com/playwright-community/playwright-go
https://github.com/samratpro/playwright_chrome_manager
```
## 02. DOM Selecting extension
```
https://chromewebstore.google.com/detail/selectorshub/ndgimibanhlabgdgjcpbbndiehljcpfh
```
## 03. How to setup environment
```bash

```
## 04. Playwright with Pyinstaller
```bash
Recommended approaches
-Bundle minimal binary with go build — your Go program will call Playwright; browsers must be available on the system (installed via npx playwright install or system Chrome).
-Distribute with browser files — place Playwright browser bundles next to your binary and use ExecutablePath to point to them.
-Use CI to pre-download engines with npx playwright install and ship zipped engines/binary together.
-Example: use PLAYWRIGHT_BROWSERS_PATH-like approach (conceptual)
-If you have downloaded Chromium binary (e.g. from Playwright build or Chrome for Testing), put it under browsers/chrome-win/ and point ExecutablePath to the chrome.exe.
-For Windows: C:\path\to\chrome-win\chrome.exe
-For Linux/Mac point to correct binary.
```
## 05. Example of initial Code
i. synchronous way
```go
package main

import (
	"log"

	"github.com/playwright-community/playwright-go"
)

func main() {
	pw, err := playwright.Run()
	if err != nil {
		log.Fatalf("could not start Playwright: %v", err)
	}
	defer func() {
		if err := pw.Stop(); err != nil {
			log.Printf("error stopping Playwright: %v", err)
		}
	}()

	// Launch Chromium with anti-detection flag
	browser, err := pw.Chromium.Launch(playwright.BrowserTypeLaunchOptions{
		Headless: playwright.Bool(false),
		Args:     []string{"--disable-blink-features=AutomationControlled"},
	})
	if err != nil {
		log.Fatalf("could not launch browser: %v", err)
	}
	defer browser.Close()

	ctx, err := browser.NewContext()
	if err != nil {
		log.Fatalf("could not create context: %v", err)
	}

	page, err := ctx.NewPage()
	if err != nil {
		log.Fatalf("could not create page: %v", err)
	}

	_, err = page.Goto("https://example.com", playwright.PageGotoOptions{
		Timeout: playwright.Float(60000),
	})
	if err != nil {
		log.Fatalf("goto error: %v", err)
	}

	// Go back example (navigate back)
	if _, err := page.GoBack(); err != nil {
		log.Printf("go back error: %v", err)
	}
}
```
ii. asynchronous way with parallel (playwright_examples.md)
```go
// 05. limit concurrency ie 3 Tab Async
// 04. Fixed ie 3 Tab Async


// Example conceptual snippet for parallel tabs (limit concurrency to 3)
package main

import (
	"log"
	"sync"

	"github.com/playwright-community/playwright-go"
)

func worker(pw playwright.Playwright, url string, wg *sync.WaitGroup) {
	defer wg.Done()
	browser, _ := pw.Chromium.Launch(playwright.BrowserTypeLaunchOptions{Headless: playwright.Bool(true)})
	defer browser.Close()
	ctx, _ := browser.NewContext()
	page, _ := ctx.NewPage()
	_, err := page.Goto(url)
	if err != nil {
		log.Printf("error visiting %s: %v", url, err)
	}
	// do more ...
}

func main() {
	pw, _ := playwright.Run()
	defer pw.Stop()

	urls := []string{"https://a.com", "https://b.com", "https://c.com", "https://d.com"}
	var wg sync.WaitGroup
	sem := make(chan struct{}, 3) // concurrency limit 3

	for _, u := range urls {
		wg.Add(1)
		sem <- struct{}{}
		go func(url string) {
			defer func() { <-sem }()
			worker(pw, url, &wg)
		}(u)
	}

	wg.Wait()
}
```
iii. Define browser path
```go
browser, err := pw.Chromium.Launch(playwright.BrowserTypeLaunchOptions{
	Headless:       playwright.Bool(false),
	ExecutablePath: playwright.String(`C:\path\to\chrome-win\chrome.exe`), // Windows example
	Args:           []string{"--disable-blink-features=AutomationControlled"},
})
```
iv. Language
```go
ctx, err := browser.NewContext(playwright.BrowserNewContextOptions{
	NoDefaultViewport: playwright.Bool(true),
	Locale:             playwright.String("en-US"),
	Geolocation: &playwright.Geolocation{
		Longitude: 12.4924,
		Latitude:  41.8902,
	},
	Permissions: []string{"geolocation"}, // if you need permissions granted
})

```
iv.1. Mobile Browser
```go
device := playwright.Devices["iPhone 12"]
ctx, err := browser.NewContext(playwright.BrowserNewContextOptions{
	Viewport:     device.Viewport,
	UserAgent:    playwright.String(device.UserAgent),
	IsMobile:     playwright.Bool(true),
	DeviceScaleFactor: playwright.Float(device.DeviceScaleFactor),
	HasTouch:     playwright.Bool(true),
})
page, _ := ctx.NewPage()
```
v. change entire fingerprint
- method 1:
```bash
stealthScript := `/* small stealth stub: override navigator.webdriver etc */ 
Object.defineProperty(navigator, 'webdriver', {get: () => undefined});
`
ctx.AddInitScript(playwright.BrowserContextAddInitScriptOptions{
	Script: playwright.String(stealthScript),
})

```
- method 2:
```go
browser, _ := pw.Chromium.Launch(playwright.BrowserTypeLaunchOptions{
	Headless: playwright.Bool(false),
	Args: []string{
		"--disable-blink-features=AutomationControlled",
		"--start-maximized",
		"--disable-infobars",
		"--no-sandbox",
		"--enable-gpu",
		"--use-gl=desktop",
		"--enable-webgl",
		"--autoplay-policy=no-user-gesture-required",
		"--disable-dev-shm-usage",
		"--disable-extensions",
		"--remote-debugging-port=0",
		"--disable-web-security",
		"--enable-features=WebRTCPeerConnectionWithBlockIceAddresses",
		"--force-webrtc-ip-handling-policy=disable_non_proxied_udp",
	},
})

```
vi. Add Proxy
```go
browser, err := pw.Chromium.Launch(playwright.BrowserTypeLaunchOptions{
	Headless: playwright.Bool(false),
	Args:     []string{"--disable-blink-features=AutomationControlled"},
	Proxy: &playwright.BrowserTypeLaunchOptionsProxy{
		Server:   playwright.String("http://gw.dataimpulse.com:823"),
		Username: playwright.String("5505791abdxxxxxxxx__cr.us"),
		Password: playwright.String("f5xxxxxxxxxxxxxxx"),
	},
})

```
vii. Maximize Screen
```go
browser, _ := pw.Chromium.Launch(playwright.BrowserTypeLaunchOptions{
	Headless: playwright.Bool(false),
	Args:     []string{"--disable-blink-features=AutomationControlled", "--start-maximized"},
})
ctx, _ := browser.NewContext(playwright.BrowserNewContextOptions{
	NoDefaultViewport: playwright.Bool(true),
})
page, _ := ctx.NewPage()

```
iiX. Cookie
```py
// Save storage state
err = ctx.StorageState(playwright.BrowserContextStorageStateOptions{Path: playwright.String("state.json")})

// Load storage state on new context
ctx2, _ := browser.NewContext(playwright.BrowserNewContextOptions{
	StorageState: playwright.String("state.json"),
})

```
iX. multiple tab/page
```py
page1, _ := ctx.NewPage()
page1.Goto("https://a.example")
page2, _ := ctx.NewPage()
page2.Goto("https://b.example")

// Iterate through pages
pages, _ := ctx.Pages()
for _, p := range pages {
	url, _ := p.URL()
	fmt.Println(url)
}

```
x. working with real browser
```
browser, _ := pw.Chromium.Launch(playwright.BrowserTypeLaunchOptions{
	Headless:       playwright.Bool(false),
	ExecutablePath: playwright.String(`C:\Program Files\Google\Chrome\Application\chrome.exe`),
})

```
## 06. Using Cookie Example Code with Chromium/Custom Chrome
```
03. playwright_examples.md(01)
```
## 06.1 Cookie Use with real chrome browser
```
03. playwright_examples.md (03)
```
## 07. Creating New Page & Visiting individual page (Multi Tab)
```py
from playwright.sync_api import sync_playwright
with sync_playwright() as p:
    browser = p.chromium.launch(headless=False, args=['--disable-blink-features=AutomationControlled'],)
    context = browser.new_context()
    page = context.new_page()
    page.goto('link')
    all_pages = page.selector('all_pages') # Get all pages
    for single_page in all_pages:
        new_tab = context.new_page()
        new_tab.goto(single_page)
        new_tab.close()
    browser.close()
```
## 08. Screenshot
```py
page.screenshot(path='screenshot.png')
```
## 09. Reload
```py
page.reload()
```
## 10. with BeautifulSoup
```go
import (
	"github.com/PuerkitoBio/goquery"
)

html, _ := page.Content()
doc, _ := goquery.NewDocumentFromReader(strings.NewReader(html))
doc.Find("div[data-component-type='s-search-result']").Each(func(i int, s *goquery.Selection) {
	text := s.Text()
	fmt.Println(text)
})

```
## 11. Selecting element
```go
// Locator examples
elFirst := page.Locator("css=div.item").First()
allEl := page.Locator("css=div.item") // locator handle for multiple matches

// Query selectors (JS-like)
handle, _ := page.QuerySelector("div#my-id")
if handle != nil {
	prop, _ := handle.GetAttribute("href")
	fmt.Println(prop)
}

// XPath / Text
page.Click(`xpath=//button[.='Post']`)
page.Click(`text=Post`) // find by text

```

## 12. finding elements
```py
# Find By Text
page.click("text='Post'")  # Post is text of button
page.locator("text='Element Text Name").fill('my text')
elements = page.query_selector_all("text='Element Text Name'")
page.locator(//button[.='Post'])           # if text is exist in child level element of selected element

# find by class or id
page.locator(.class_name)     # single targeted class name
page.locator(#id_name)        # class name
page.locator(.class1.class2)  # multiple target class name from many class

# (//element[@identy='name'])[index]
element = page.locator("//any-path").first.text_content()            # Find first element's content -- suitable for mouse and keyword event
element = page.query_selector('//any-xpath').text_content()          # Find first path only -- suitable for get content
elements = page.query_selector_all('//any-xpath')                    # Find all path -- suitable for get All Content
elements = page.query_selector_all("text='Element Text Name'")        # Find all by text
for e in elements:
    print(e.text_content())

elements = page.query_selector_all('//any-xpath')[1]                            # Find second element
link = page.query_selector('//any-xpath').get_property('href')                  # Get link - img src, title, class, id
link = page.locator("//any-path").get_attribute('href')
inner_html = page.query_selector('//any-xpath').inner_html()                    # Get inner HTML of any selected part
inner_text = page.query_selector('//any-xpath').inner_text()                    # Get inner TEXT of any selected part

page.get_by_role()                     # locate by explicit and implicit accessibility attributes.
page.get_by_text()                     # locate by text content.
page.get_by_label()                    # locate a form control by associated label's text.
page.get_by_placeholder()              # locate an input by placeholder.
page.get_by_alt_text()                 # locate an element, usually image, by its text alternative.
page.get_by_title()                    # locate an element by its title attribute.
page.get_by_test_id()                  # locate an element based on its data-testid attribute (other attributes can be configured).
page.get_attribute()                   # argument can be 'href', 'src' etc
# https://playwright.dev/docs/locators 
```
## 13. Element Behaviour
```py
btn := page.Locator("//header[@class='_inline-sidesheet-header_1cn7lg']//button[2]")
isDisabled, _ := btn.IsDisabled()
isVisible, _ := btn.IsVisible()
isHidden, _ := btn.IsHidden()
isEditable, _ := btn.IsEditable()
isChecked, _ := btn.IsChecked()

```

## 14. finding Loop logic
```go
elements, _ := page.QuerySelectorAll("//any-xpath")
for i, e := range elements {
	txt, _ := e.TextContent()
	fmt.Printf("%d -> %s\n", i, txt)
}

// while-like loops in Go
for i := 1; ; i++ {
	selector := fmt.Sprintf("(//path)[%d]", i)
	elem, _ := page.QuerySelector(selector)
	if elem == nil {
		break
	}
	// process elem
}

```
## 15. Handle Multiple elements
```go
links := page.Locator("a.some-selector")
handles, _ := links.ElementHandles()
for _, h := range handles {
	href, _ := h.GetAttribute("href")
	fmt.Println(href)
}

```
## 16. Find element Validation
```go
el := page.Locator("path")
if count, _ := el.Count(); count > 0 {
	text, _ := el.InnerText()
	fmt.Println(text)
}

els := page.QuerySelectorAll("path")
if len(els) > 0 {
	fmt.Println("Work")
}

el := page.Locator("path")
if count, _ := el.Count(); count > 0 {
	_ = el.Click()
}
```

## 18. input / write text
```go
// type (simulate human typing)
page.Locator("input#name").Type("Text Here..", playwright.PageTypeOptions{Delay: playwright.Float(100)})

// fill (fast)
page.Locator("input#email").Fill("someone@example.com")

// Enter key
page.Keyboard().Press("Enter")
page.Keyboard().InsertText("A")

```
### 17.2 Advance Input in react base application
```go
content := "Hello world"
page.Evaluate(`({ selector, value }) => {
  const el = document.querySelector(selector);
  if (el) {
    el.focus();
    el.innerText = value;
    el.dispatchEvent(new Event('input', { bubbles: true }));
    el.dispatchEvent(new Event('change', { bubbles: true }));
    el.blur();
  }
}`, playwright.EvalOptions{Arg: map[string]string{"selector": "p[data-testid='editorParagraphText']", "value": content}})

```
## 18. DOM Content Update & JavaScript
```go
page.Fill("//input[@id='_job_application_deadline_date']", "05-04-2025")
page.Evaluate(`() => document.querySelector("#_job_application_deadline_date").value = '05-04-2025'`)

page.GrantPermissions([]string{"clipboard-read", "clipboard-write"}, playwright.PageGrantPermissionsOptions{Origin: playwright.String(page.URL())})
page.Click("selector-that-copies")
copied, _ := page.Evaluate("() => navigator.clipboard.readText()")
fmt.Println(copied)
```
## 19. How to intertact JS click from browser devs
```
- from inspect element of targeted element
- right click > copy > copy js path
- insert copy in JS console (F12) and add .click() function and inter
 ```
## 20. Click
```
https://playwright.dev/python/docs/input
```
```go
el := page.Locator("path").First()
el.Click()
el.DblClick()
el.Hover()
el.Type("text").Press("Enter")

page.Keyboard().Press("Enter")
page.GetByRole("button").Click()
page.GetByText("Item").DblClick()
page.GetByText("Item").Click(playwright.PageClickOptions{Button: playwright.MouseButtonRight})
page.GetByText("Item").Click(playwright.PageClickOptions{Modifiers: []string{"Shift"}})
page.GetByText("Item").Hover()
page.GetByText("Item").Click(playwright.PageClickOptions{Position: &playwright.Position{X: 0, Y: 0}})

```
## 21. Hover and hold or click  and hold
```go
hold := page.Locator("path")
hold.Click() // or hold.Hover()
page.Locator("inner_path").Click()
```
```
page.Mouse().Move(100, 100)
page.Mouse().Down()
page.Mouse().Move(120, 120)
page.Mouse().Up()
```
## 22. click blank space
```go
x, y := 500, 500
page.Mouse().Click(x, y)
page.Evaluate(fmt.Sprintf("document.elementFromPoint(%d, %d).click()", x, y))
```
```
x, y := 500, 500
page.Mouse().Click(x, y)
page.Evaluate(fmt.Sprintf("document.elementFromPoint(%d, %d).click()", x, y))
```
```
x, y := 500, 500
page.Mouse().Click(x, y)
page.Evaluate(fmt.Sprintf("document.elementFromPoint(%d, %d).click()", x, y))
```

## 23. Keyboard Behave
```go
page.Keyboard().Down("Shift")
page.Keyboard().Down("Control")
page.Keyboard().Down("Alt")
page.Keyboard().Up("Shift")
page.Keyboard().Up("Control")
page.Keyboard().Up("Alt")

page.Keyboard().Press("ArrowDown")
page.Keyboard().Press("ArrowUp")
page.Keyboard().Press("ArrowLeft")
page.Keyboard().Press("ArrowRight")

```
## 24. Wait
```
// 1. Goto with timeout
_, err = page.Goto("https://www.linkedin.com/sales/", playwright.PageGotoOptions{
	Timeout: playwright.Float(60000),
})
if err != nil {
	log.Fatal("Goto failed:", err)
}

// 2. Wait for navigation to complete
if err := page.WaitForLoadState(playwright.PageWaitForLoadStateOptions{
	State:   playwright.LoadStateLoad,
	Timeout: playwright.Float(30000),
}); err != nil {
	log.Fatal("WaitForLoadState failed:", err)
}

// 3. Wait for URL pattern
if err := waitForURL(page, "https://www.linkedin.com/sales/*", 30000); err != nil {
	log.Fatal(err)
}

// 4. Wait for selector (e.g., search box)
if err := waitForSelector(page, "input[aria-label='Search']", 30000); err != nil {
	log.Fatal(err)
}

// 5. Wait for JS condition
if err := waitForFunction(page, "document.readyState === 'complete'", 10000); err != nil {
	log.Fatal(err)
}

// 6. Optional: static wait
page.WaitForTimeout(5000) // 5 seconds

```

## 25. scrolling
```go
for i := 0; i < 10; i++ {
	page.Mouse().Wheel(0, 100)
	time.Sleep(300 * time.Millisecond)
}

scroll := 3
for scroll < 26 {
	selector := fmt.Sprintf("(//li[contains(@class,'artdeco-list__item pl3 pv3')])[%d]", scroll)
	page.Locator(selector).Click()
	scroll += 3
}

```
## 26. Dropdown
```go
page.QuerySelector("#Select2").SelectOption(playwright.SelectOptionValues{"value"})
page.Locator("#Select2").SelectOption(playwright.SelectOptionValues{"value"})
```

## 27. Iframe
```go
page.WaitForSelector("iframe[name='frameName']")
iframeEl, _ := page.QuerySelector("iframe[name='frameName']")
frame := iframeEl.ContentFrame()
frame.Locator("input#inside").Fill("some data")

```
## 28. Load and Automate extension
```go
browser, _ := pw.Chromium.Launch(playwright.BrowserTypeLaunchOptions{
	Headless: playwright.Bool(false),
	Args: []string{
		"--disable-extensions-except=/path/to/extension",
		"--load-extension=/path/to/extension",
	},
})


```
## 29. File Upload with Playwright
```go
filePath := filepath.Join(os.Getwd(), "file.extension")
chooser, _ := page.ExpectFileChooser(func() error {
	// click the button that triggers file input
	_, err := page.Click("file_upload_button")
	return err
})
chooser.SetFiles(filePath)
time.Sleep(1 * time.Second)
page.Click("submit_button")

```






