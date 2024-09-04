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

# Examples

These examples can be run. Copy and paste them to you PWSH editor of choice to Run

## Execute single elements
```
   Install-Module -Name PS.Selenium
   Import-Module -Name PS.Selenium

   $ErrorActionPreference = "Stop" # any error stop

   $waitSeconds = 0 # change this to > 0 if you want to see it working slowly !

   $driver = New-Driver -Driver Chrome `
                        -MaxRetries 5 `
                        -Fullscreen `
                        -LogLevel Information `
                        -LeaveBrowserRunning

   $browserOptions = Get-BrowserOptions
   Write-Host "Using browser $($browserOptions.BrowserDriver) version $($browserOptions.DriverVersion)"
   # open the browser
   Invoke-Element -Type Url -Path "https://selenium.infinityfreeapp.com/"    
   # wait for page 
   Invoke-Element -Type WaitForUrl -Path "?i=1"    
   # input some data 
   Invoke-Element -Type Input -Selector XPath -Path "//input[@id='formGroupExampleInput']" -Value "Input some text" -WaitSeconds $waitSeconds
   # set select element 
   Invoke-Element -Type Select -Selector XPath -Path "//select[@id='formSelect']" -Value "Select Me" -WaitSeconds 0
   # check a check box 
   Invoke-Element -Type Button -Selector XPath -Path "//input[@id='flexCheckDefault']" -WaitSeconds $waitSeconds
   # check a radio 
   Invoke-Element -Type Button -Selector XPath -Path "//input[@id='flexRadioDefault1']"
   # click a button and assert the button action - the button should wait for 5 seconds before displaying the output and PS.Selenium should wait for it 
   Invoke-Element -Type Button -Selector XPath -Path "//button[contains(text(), 'Button Will')]" -Script { 
      param($parentElement)
      Find-Element -Selector XPath -Path "//label[@id='buttonPressed']" -Script { 
                     param($element)
                     Assert-Element -Element $element -AssertionType IsEqualTo -Value "Button Pressed"  
      }
   }

   Invoke-Element -Type Image -TakeImage "Screen 1" # take a screen shot and call it screen 1

   # click on table page 
   Invoke-Element -Type Button -Selector XPath -Path "//a[@id='linkPressed']" 
   # wait for the page 
   Invoke-Element -Type WaitForUrl -Path "/TableTest"  
   # get table and set an input value
   Get-Table -Selector XPath -Path "//table[@id='testTable']" | `                       # returns a type of TableModel
            Get-TableColumn -Row 1 -Column 4 -Selector XPath -Path "./input" | `        # ./ means relative to the cell element 
            Invoke-WebElement -Type Input -Value "Riddle me this" -WaitSeconds $waitSeconds                      # returns IWebElement
   # OR multiple changes from one tabe definition
   Get-Table -Selector XPath -Path "//table[@id='testTable']" -Script { 
      param($table) # type of TableModel
      Get-TableColumn -Table $table -Row 1 -Column 4 -Selector XPath -Path "./input" | Invoke-WebElement -Type Input -Value "Riddle me that" -WaitSeconds $waitSeconds
      Get-TableColumn -Table $table -Row 2 -Column 5 -Selector XPath -Path "./input" | Invoke-WebElement -Type Button -WaitSeconds $waitSeconds 
   }

   Invoke-Element -Type Image -TakeImage "Screen 2" # take a screen shot and call it screen 1

   # save screen shots and logs 

   $dir = "$env:TEMP\Selenium"

   Save-ScreenShot -Path $dir -ScreenShotType All   
   Save-Log -Path $dir -Type Text #default -Type is json , can specify filename via -Name parameter

   Close-Browser # close browser and any associated selenium processes
```   
## Execute Tests
```
   Install-Module -Name PS.Selenium
   Import-Module -Name PS.Selenium

   $ErrorActionPreference = "Stop" # any error stop



   $testPath = "$env:Temp\SeleniumTests"
   $page1Path = Join-Path -Path $testPath -ChildPath "Page1.json"

   # define a set of elements on screen one 
   function New-Page1Elements {
      New-ElementCollection -Page "Page1" | `
         Add-Element -Type Url -Name "Page1Url" -Path "https://selenium.infinityfreeapp.com/" | `
         Add-Element -Type WaitForUrl -Name "Page1WaitForUrl" -Path "?i=1" | `
         Add-Element -Type Input -Name "TestInput" -Selector XPath -Path "//input[@id='formGroupExampleInput']" | `
         Add-Element -Type Select -Name "TestSelect" -Selector XPath -Path "//select[@id='formSelect']" -Value "Select Me" | `
         Add-Element -Type Button -Name "TestButton" -Selector XPath -Path "//input[@id='flexCheckDefault']" | `
         Add-Element -Type Button -Name "TestLink" -Selector XPath -Path "//a[@id='linkPressed']" -TakeImage "Screen 1" | `
         Save-Elements -Path $page1Path # optional - (see import-elements)  
   }  


   function New-Page2Elements {
      New-ElementCollection -Page "Page2" | `
         Add-element -Type WaitForUrl -Name "WaitForPage2" -Path "/TableTest" -Script { # wait for the page and then manipulate the table
               Get-Table -Selector XPath -Path "//table[@id='testTable']" | `
               Get-TableColumn -Row 1 -Column 4 -Selector XPath -Path "./input" | `
               Invoke-WebElement -Type Input -Value "Riddle me this" 
         }  
   }

   # define the page elements 
   New-Page1Elements  
   New-Page2Elements

   # previoulsy saved element definitions can be loaded into storage (un-comment dollowing line)
   # Import-Elements -Path $page1Path 

   # define a test and load some pages into it 
   # Add-Page can add a page to the test from an element colletion (-Page) , from storage (-Name) or from a file (-Path)  
   $test1 = New-Test -Name "PageTests" | Add-Page -Name "Page1" | Add-Page -Name "Page2"


   # startup the driver 
   $driver = New-Driver -Driver Chrome `
                        -MaxRetries 5 `
                        -Fullscreen `
                        -LogLevel Information `
                        -LeaveBrowserRunning

   # execute test. but before the page is executed set some values of the elements                      
   $testResults = Invoke-Test -Name "PageTests" -OnBeforePage { 
      param($page)
      switch ($page.Name) {
         "Page1" {
               # can set element properties before the test runs  
               Set-Element -Name "TestInput" -PageObject $page -Value "Test Input" #could be sensitive 
         }
      }
   }                    

   # todo - do something with the results
   foreach ($result in $testResults.PageResults) {
      Write-Host ($result.Result | Format-List | Out-String)
   }

   Close-Browser 
```

&nbsp;
## Cmdlet Summary
   | Cmdlet | Description | 
   | --- | --- | 
   | [New-Driver](#new-driver) | Creates the selenium session 
   | [Invoke-Element](#invoke-element) | Interacts with an element 
   | [Find-Element](#find-element) | Locates an element in the current page 
   | [New-ElementCollection](#new-elementcollection) | creates a new collection of elements
   | [Add-Element](#add-element) | Adds an element definition to an element collection
   | [Save-Elements](#save-elements) | Saves a collection of elements to a file
   | [Import-Elements](#import-elements) | Imports (into memory) a collection of elements from a file 
   | [New-Test](#new-test) | creates a new test  
   | [Add-Page](#add-page) | Adds an existing element collection (page) to an existing test 


&nbsp;
# CMDLETS

# New-Driver 

this must be done _before_ you execute an element , page or test. The correct driver for the selected browser 
will be automatically downloaded. If there is no internet connection - or there is a contraint on downloding from the internet then you should set the -DriverPath parameter to the location of the correct driver for the browser being used  

```
 New-Driver -Driver Chrome `
            -MaxRetries 5 `
            -Fullscreen ` 
            -LogLevel Information `
            -LeaveBrowserRunning 
```
#### Parameters

__-Driver__

The name of the browser driver to use. This can (currently) be __Chrome__ or __Edge__

*if the client has internet access, the driver that is relevant to the clients version of the browser will automatically be downloaded and used*  

__-DriverPath__ (optional)

The full path of the drvier required for the selected browser. Use this if there is no internet connection 

__-LogPath__ (optional)

The path (and name) where logs should be written - use Save-Logs to define at run time.    

__-MaxRetries__

The Maximum number of times to retry a selenium action before an exception is thrown. 

*note - each iteration of a wait will be multiplied by this value - so keep it small!*

*The default is 1 second*

__-MaxRetryTimeout__

The Maximum timeout value (in seconds) that each retry should be performed 

*The default is 50*

__-Headless__ (switch parameter)

Run the browser in headless mode (do NOT display the browser window) 

__-Fullscreen__ (switch parameter)

Run the browser in fullscreen 

__-LeaveBrowserRunning__  (switch parameter)__

At the end of the session leave the browser running.

*In healess mode the browser and the driver will be terminated*

__-LogLevel__ (optional)

log level can be :-
   
- Information
- Warning
- Error
- Debug

*The default is Information*

__-BrowserStartupOptions__ (optional string[])

string[] list of optional parameters to pass to the browser driver (e.g --disable-gpu)

___

# Invoke-Element

Invokes an element on a page 

```
   # open the browser with the specified url  
   Invoke-Element -Type Url -Path "www.google.com" 
   # update search input element 
   Invoke-Element -Type Input Selector XPath -Path "//input[@id='input']" -Value "Upgrade to Selenium 4" 
   # click on first matching href
   Invoke-Element -Type Button Selector XPath -Path "//a[contains(@href,'www.selenium.dev')]" 
```
#### Parameters

__-Type__ (Mandatory - Enum)

The type of Action on this element can be :-

- Input                  
- Button
- Select
- InputAndPressEnter
- WaitForUrl
- WaitForElement 
- Wait
- Url
- Id
- Image

__-Selector__ 

The element selector. i.e they way to find the element on the page :-

- XPath
- Name
- CssSelector
- ClassName
- Id
- TagName

__-Path__ (Mandatory - string)

The path of (the specified selectory) - e.q -Path "\\\\input(@id, 'someid')"

__-Value__ (string)

The value to enter (typically for a input/textarea)

__-TakeImage__ (string)

Take an image *after* this element has been processed. the name of the image will be  the value of this parameter (**see also Get-ScreenShot and Save-ScreenShot**)

__-WaitSeconds__ (int)

Waits the specified number of seconds *before* processing the element (this should not be required since PS.Selenium will retry the action based on the maxtries definition in the new-driver parameters)

__-Optional (Switch parameter)__

if this is set then PS.Selenium will not throw an error if the element is not found

__-Maxtries__ (int)

overrides the (default) value of the number of retires for the element processing

__-Script__ 

executes the given script element *after* the element has been proecessed. the ScriptBlock will be passed an IWebElement of the processed element. e.q 

```
    Invoke-Element -Type Input -Selector XPath -Path "someXpath" -Value "hello" -Script { 
      param($iwebElement)
      Assert-Element -Element $iwebElement -AssertionType IsEqualTo -Value "hello"
    }  
```
___
# Find-Element

Searches for the given element and returns the *IWebElement* associted with it.   

```
   $element = Find-Element -Selector XPath -Path "someXpath" 
```

#### Parameters

__-Selector__ (Mandatory - Enum)

The element selector. i.e they way to find the element on the page :-

- XPath
- Name
- CssSelector
- ClassName
- Id
- TagName

__-Path__ (Mandatory - string)

The path of (the specified selectory) - e.q -Path "\\\\input(@id, 'someid')"

__-Script__ 

executes the given script element *after* the element has been proecessed. the ScriptBlock will be passed an IWebElement of the processed element. e.q 

```
   Find-Element -Selector XPath -Path "someElement" -Script { 
      param($iwebElement)
      # do something
   }
```
___

&nbsp;
# New-ElementCollection

Creates a new collection of elements in storage (a page)

```
    New-ElementCollection -Page "Page1"
```

#### Parameters

__Page__ (required)

The name of the page that contains the elements 

__Returns__

ElementCollection object 

___

# Add-Element

Adds an element definition of a element collection

```
    $collection = New-ElementCollection -Page "somePage" | `
        Add-Element -Type Url -Name "Page1Url" -Path "https://selenium.infinityfreeapp.com/" | `
        Add-Element -Type WaitForUrl -Name "Page1WaitForUrl" -Path "?i=1" 
```

#### Parameters

__-Elements__ (required - value from pipeline)

The ElementCollection to add the element to. 

__-Type__ (required - enum)

The type of the element 

- Input                  
- Button
- Select
- InputAndPressEnter
- WaitForUrl
- WaitForElement 
- Wait
- Url
- Id
- Image

__-Selector__ (optional - but in most cases required)

The selector to use when finding the element

- XPath
- Name
- CssSelector
- ClassName
- Id
- TagName

__-Name__

A unique name to assign to this element 

__-Value__ (string - optional) 

A string value to update an element with 

__-Path__

The path of (the specified selectory) - e.q -Path "\\\\input(@id, 'someid')"

__-TakeImage__ (optional string)

Takes an image of the current page. Can be retrieved later with Get-ScreenShot 

__-WaitSeconds__ (optional int)

Waits for the specified number of seconds before executing the action on the element

__-Optional__ (switch)

if specified tells PS.Selenium NOT to throw an error if the element does not exist


___
&nbsp;
# Save-Elements

Saves an element collection to a file

```
    Save-Elements -Elements $elementCollection -Path "Somewhere.json"
```

#### Parameters

__-Elements__ (required)

ElementCollection object 

__-Path__ (required)

Full path to the file location where the elementcollection should be saved
___

&nbsp;
# Import-Elements

Imports a collection of elements from a file and adds it to the current storage

```
   Import-Elements -Path "pathTojsonfile.json"
```
#### Parameters

__Path__ (required)

The full path to the file containing the elements to load 

___

&nbsp;
# New-Test

Creates a new test 

```
   New-Test -Name "Test01"
```

#### Parameters

__-Name__ (required - string)

The Name of the test

___

&nbsp;
# Add-Page 

Adds an existing element collection (page) to an existing Test. 

#### Parameters

