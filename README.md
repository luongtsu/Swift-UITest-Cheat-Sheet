# Swift-UITest-Cheat-Sheet

*Swift UITest Cheat Sheet (Updated for Swift 3.0)*

## Table of Contents

* [How to prepare for UITest in XCode](#how-to-prepare-for-uitest-in-xcode)
* [Basic functionalities](#basic-functionalities)
  * Launch app in test class's setUp() function to ensure the app launches clean for every test
  * Print the accessibility hierarchy
  * Testing if an element exists (even if the element is off the screen)
  * Checking if a view is on the screen
  * Testing if textview with given text exist
  * Waiting for an element to appear
  * Assertions 
* [Interactions with system controls](#interactions-with-system-controls)
  * Tap a button
  * Type into textField
  * Dismiss an alert
  * Handle system alerts (location services, push notifications, access microphone, contacts or access to your photos)
  * Slide a slider
  * Interact with one wheel picker
  * Interact with multiple wheels picker: set up accessibility so that different wheels could be identified
  * Tap on a link in web views UIWebView/WKWebView
* [Other interactions](#other-interactions)
  * Longpress a button
  * Swipe action
  * Two finger tap
  * Pinch action
  * Check the current controller's title
  * Reorder tableviewcells
  * Pull to refresh
  * Check if view controller is pushed successful
  * Check after popping a view controller (tap on Back button)
* [Generic querying syntaxes](#generic-querying-syntaxes)

## How to prepare for UITest in XCode
- Check your projects if UITest target is existing. If not, don't worry. Let's add it by opening XCode, choose File/New/Target, choose iOS UI Testing Bundle, Next & Finish.
- User **@testable** macro to allow UITest class access you test class  
```swift 
@testable import My_UITest_Example
```   
- Access application through XCUIApplication, you can **launch()**, you also can **terminate()** it.
- Access the device through XCUIDevice:   
	get new instance of it using **-sharedDevice**   
	get UIDeviceOrientation type from public accessible property **-orientation**
- Query for elements using XCUIElementQuery (see *Generic querying syntax*)

## Basic functionalities
- Launch app in test class's **setUp()** function to ensure the app launches clean for every test    
```swift
override func setUp() {    
        super.setUp()    
        let app = XCUIApplication()    
        app.launch(    
}
```
In this ducumentation, whenever we refer to "app" that mean "app" in **let app = XCUIApplication()**
- Print the accessibility hierarchy
```swift
print(app.debugDescription)
```
- Testing if an element exists (even if the element is off the screen)
```swift
XCTAssert(app.staticTexts["Welcome"].exists)
```
- Checking if a view is on the screen
```swift
let window = app.windows.element(boundBy: 0)
let btnShowMore = app.buttons["Show more"]
XCTAssert(CGRectContainsRect(window.frame, btnShowMore.frame))
```
- Testing if textview with given text exist
```swift
let anyTextView = app.staticTexts["Hi John"]
XCTAssert(anyTextView.exists)
```

- Waiting for an element to appear
```swift
let detailsLabel = app.staticTexts["Details"]
XCTAssertFalse(detailsLabel.exists)

let exists = NSPredicate(format: "exists == true")
expectation(for: exists, evaluatedWithObject: detailsLabel, handler: nil)

app.buttons["Show details"].tap()
waitForExpectations(timeout: 5, handler: nil)
XCTAssert(detailsLabel.exists)
//If five seconds pass before the expectation is met then the test will fail.
```

- Assertions: just type **XCTAssert** then XCode will show the hint with different type of assertions for you to use (*XCTAssertEquals*,*XCTAssertNotNil*,etc)

## Interactions with system controls
- Tap a button
```swift
app.buttons["Add"].tap()
```

- Type into textField
```swift
let textField = app.textFields["Username"]
textField.tap()
textField.typeText("John")
```

- Dismiss an alert
```swift
app.alerts["Alert Title"].buttons["Button Title"].tap()
```

- Handle system alerts (location services, push notifications, access microphone, contacts or access to your photos)   
	*Present a photo access request dialog to the user and dismiss it with the following code. Before presenting the alert add a UI Interuption Handler. When this fires, dismiss with the "Allow" button.*

```swift
addUIInterruptionMonitor(withDescription: \"Access Photos\") { (alert) -> Bool in
	alert.buttons["Allow"].tap()
    return true
}

app.buttons["Your Photos"].tap()
app.tap() // need to interact with the app again for the handler to fire
```

- Slide a slider   
*// set value to 90%*   
```swift
app.sliders.element.adjust(toNormalizedSliderPosition: 0.9)
```

- Interact with one wheel picker
```swift
app.pickerWheels.element.adjust(toPickerWheelValue: "Picker Wheel Item Title")
```

- Interact with multiple wheels picker: set up accessibility so that different wheels could be identified   
```swift
let firstPredicate = NSPredicate(format: "label BEGINSWITH 'First Picker'")
let firstPicker = app.pickerWheels.element(matching: firstPredicate)
firstPicker.adjust(toPickerWheelValue: "first value")

let secondPredicate = NSPredicate(format: "label BEGINSWITH 'Second Picker'")
let secondPicker = app.pickerWheels.element(matching: secondPredicate)
secondPicker.adjust(toPickerWheelValue: "second value")
```

- Tap on a link in web views UIWebView/WKWebView
```swift
app.links["Tweet this"].tap()
```

## Other interactions

- Longpress a button
```swift
app.pickers.elementAtIndex(0).pressForDuration(0.1, thenDragToElement: someElement)
```

- Swipe action
```swift
app.tables.staticTexts["Swipe me"].swipeUp()
```

- Two finger tap
```swift
let view = app.descendantsMatchingType(.Unknow)["myView"]
view.twoFingerTap()
```

- Pinch action
```swift
app.maps.element.pinchWithScale(1.5, velocity: 1)
```

- Check the current controller's title
```swift
XCTAssert(app.navigationBars["Main menu"].exists)
```

- Reorder tableviewcells
*If you have a UITableViewCell with default style and set the text to "Title", the reorder control's accessibility label becomes "Reorder Title". Using this we can drag one reorder control to another, essentially reordering the cells.*
```swift
let topButton = app.buttons["Reorder Top Cell"]
let bottomButton = app.buttons["Reorder Bottom Cell"]
bottomButton.press(forDuration: 0.5, thenDragTo: topButton)

XCTAssertLessThanOrEqual(bottomButton.frame.maxY, topButton.frame.minY)
```

- Pull to refresh
*Create a XCUICoordinate from the first cell in your table and another one with a dy of six. Then drag the first coordinate to the second.*
```swift
let firstCell = app.staticTexts["Adrienne"]
let start = firstCell.coordinate(withNormalizedOffset: (CGVectorMake(0, 0))
let finish = firstCell.coordinate(withNormalizedOffset: (CGVectorMake(0, 6))
start.press(forDuration: 0, thenDragTo: finish)
```

- Check if view controller is pushed successful
```swift
app.buttons["Details Info"].tap()   
XCTAssert(app.navigationBars["Details"].exists
```

- Check after popping a view controller (tap on Back button)
```swift
app.navigationBars.buttons.elementBoundByIndex(0).tap()
XCTAssert(app.navigationBars["Volley"].exists)
```

## Generic querying syntaxes
UI Testing uses **XCUIElementQuery** to query elements in the app's view hierarchy. The syntax creates a buildable set of instructions to drill down to different parts of the screen.

- **app.labels.element** returns the one and only UILabel
- **app.buttons["Save"]** returns the "Save" button (via accessibility)
- **app.cells[4]** returns the fifth table view cell
- **app.tables.element.cells.elementBoundByIndex(4).tap()**
- **app.tables.element.cells["Call Mom"].buttons["More"].tap()**   
	By referencing **-element** you are essentially saying, "I know there is only one of these, I want that one."
- These convenience methods work by calling **-elementMatchingType:identifier:** with **XCUIElementTypeAny** and the string you passed in. Check out **XCUIElementType** for all the available options.
