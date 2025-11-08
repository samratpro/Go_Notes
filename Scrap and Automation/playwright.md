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
```go
// Helper: safe error handling
func check(err error) {
	if err != nil {
		log.Fatal(err)
	}
}

// Click text
err = page.Locator(`text='Post'`).Click()
check(err)

// Fill text
err = page.Locator(`text='Element Text Name'`).Fill("my text")
check(err)

// Get all elements by text
elements, err := page.Locator(`text='Element Text Name'`).All()
check(err)
for _, el := range elements {
	text, _ := el.TextContent()
	fmt.Println("Text:", text)
}

// XPath with text in child: //button[.='Post']
err = page.Locator(`//button[.='Post']`).Click()
check(err)

// ===================================================================
// 2. FIND BY CLASS / ID
// ===================================================================

// Single class
err = page.Locator(".class_name").Click()
check(err)

// ID
err = page.Locator("#id_name").Click()
check(err)

// Multiple classes
err = page.Locator(".class1.class2").Click()
check(err)

// XPath with index: (//element[@identy='name'])[2] → second element
secondEl := page.Locator(`(//element[@identy='name'])[2]`)
text, _ := secondEl.TextContent()
fmt.Println("Second element text:", text)

// ===================================================================
// 3. LOCATOR + QUERY SELECTOR
// ===================================================================

// First element text (locator)
firstText, _ := page.Locator("//any-path").First().TextContent()
fmt.Println("First text:", firstText)

// First element (query selector)
el, err := page.QuerySelector("//any-xpath")
check(err)
text, _ = el.TextContent()
fmt.Println("QuerySelector text:", text)

// All elements
allEls, err := page.QuerySelectorAll("//any-xpath")
check(err)
for i, e := range allEls {
	t, _ := e.TextContent()
	fmt.Printf("Element %d: %s\n", i, t)
}

// All by text
textEls, err := page.QuerySelectorAll(`text='Element Text Name'`)
check(err)
for _, e := range textEls {
	t, _ := e.TextContent()
	fmt.Println("Text element:", t)
}

// Second element
second, err := page.QuerySelectorAll("//any-xpath")
check(err)
if len(second) > 1 {
	t, _ := second[1].TextContent()
	fmt.Println("Second element:", t)
}

// ===================================================================
// 4. GET ATTRIBUTES (href, src, etc.)
// ===================================================================

// Get href via locator
href, _ := page.Locator("//any-path").GetAttribute("href")
fmt.Println("HREF:", href)

// Get href via query selector
linkEl, err := page.QuerySelector("//any-xpath")
check(err)
hrefObj, _ := linkEl.Evaluate("el => el.href")
fmt.Println("HREF (JS):", hrefObj)

// Get src, title, class, id
src, _ := page.Locator("//img").GetAttribute("src")
title, _ := page.Locator("//any").GetAttribute("title")
class, _ := page.Locator("//any").GetAttribute("class")

fmt.Println("SRC:", src, "Title:", title, "Class:", class)

// ===================================================================
// 5. INNER HTML / TEXT
// ===================================================================

innerHTML, _ := page.QuerySelector("//any-xpath").InnerHTML()
innerText, _ := page.QuerySelector("//any-xpath").InnerText()
fmt.Println("Inner HTML:", innerHTML)
fmt.Println("Inner Text:", innerText)

// ===================================================================
// 6. ACCESSIBILITY LOCATORS (get_by_*)
// ===================================================================

// get_by_role
err = page.GetByRole("button", playwright.PageGetByRoleOptions{
	Name: playwright.String("Post"),
}).Click()
check(err)

// get_by_text
err = page.GetByText("Element Text Name").Fill("new text")
check(err)

// get_by_label
err = page.GetByLabel("Username").Fill("john")
check(err)

// get_by_placeholder
err = page.GetByPlaceholder("Search...").Fill("query")
check(err)

// get_by_alt_text
err = page.GetByAltText("Logo").Click()
check(err)

// get_by_title
err = page.GetByTitle("Download").Click()
check(err)

// get_by_test_id
err = page.GetByTestId("submit-button").Click()
check(err)

// ===================================================================
// 7. WAIT FOR ELEMENTS
// ===================================================================

// Wait for selector
_, err = page.WaitForSelector("//button", playwright.PageWaitForSelectorOptions{
	State:   playwright.WaitForSelectorStateVisible,
	Timeout: playwright.Float(10000),
})
check(err)

// Wait for text
_, err = page.WaitForSelector(`text='Ready'`, playwright.PageWaitForSelectorOptions{
	Timeout: playwright.Float(10000),
})
check(err)
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
```go
// 1. Goto with timeout
_, err = page.Goto("https://www.linkedin.com/sales/", playwright.PageGotoOptions{
	Timeout: playwright.Float(60000),
})
if err != nil {
	log.Fatal("Goto failed:", err)
}

// 2.0 simple wait for load
page.WaitForLoadState(playwright.PageWaitForLoadStateOptions{State: playwright.LoadStateLoad})
// 2.1 Wait for navigation to complete
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






