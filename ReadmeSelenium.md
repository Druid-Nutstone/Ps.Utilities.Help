# PS.Selenium

PS.Selenium is a set of c# Cmdlets that wrap the [Selenium]([https://](https://www.selenium.dev/documentation/)) library. It allows you to control a browser via Powershell commands. It allows you to manipulate browser page elements at :- 

- A single element level (i.e Open url, set input value , press a button)
- A page level by defining a list of elements on a page and then executing them 
- A test level by defining a set of pages and associtaing them with a named test

## Using PS.Selenium 
```
   Install-Module PS.Selenium
   Import-Module PS.Selenium
```

## Initialising the Selenium driver

this must be done _before_ you execute an element , page or test. 

```
 New-Driver -Driver Chrome `
            -LogPath "C:\deleteme\PS.Selenium.Runs\Log\selnium.txt" `
            -MaxRetries 5 `
            -Fullscreen ` 
            -LogLevel Information `
            -LeaveBrowserRunning 
```
### Parameters

---
__-Driver__

The name of the browser driver to use. This can (currently) be __Chrome__ or __Edge__

*if the client has internet access, the driver that is relevant to the clients version of the browser will automatically be downloaded and used*  

---

__-LogPath__

The path (and name) where logs should be written  

---
__-MaxRetries__

The Maximum number of times to retry a selenium action before an exception is thrown. 

*note - each iteration of a wait will be multiplied by this value - so keep it small!*

*The default is 1 second*

---
__-MaxRetryTimeout__

The Maximum timeout value (in seconds) that each retry should be performed 

*The default is 50*

---
__-Headless (switch parameter)__

Run the browser in headless mode (do NOT display the browser window) 

---

__-Fullscreen (switch parameter)__

Run the browser in fullscreen 

---

__-LeaveBrowserRunning (switch parameter)__

At the end of the session leave the browser running.

*In healess mode the browser and the driver will be terminated*

---

__-LogLevel__

log level can be :-
   
- Information
- Warning
- Error
- Debug

*The default is Information*

---

# Single Element Processing 

## Invoke-Element 

Invokes an element on a page 

```
   # open the browser with the specified url  
   Invoke-Element -Type Url -Path "www.google.com" 
   # update search input element 
   Invoke-Element -Type Input Selector XPath -Path "//input[@id='input']" -Value "Upgrade to Selenium 4" 
   # click on first matching href
   Invoke-Element -Type Button Selector XPath -Path "//a[contains(@href,'www.selenium.dev')]" 
```
