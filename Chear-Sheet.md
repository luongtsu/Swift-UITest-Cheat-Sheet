
## How to prepare for UITest
- Check your projects if UITest target is existing. If not, don't worry. Let add it by opening XCode, choose File/New/Target, choose iOS UI Testing Bundle, Next & Finish.
- User @testable macro to allow UITest class access you test class  
	`@testable import My_UITest_Example`   
- Access application through XCUIApplication, you can launch(), you also can terminate() it.
- Access the device through XCUIDevice:   
	get new instance of it using *-sharedDevice*   
	get UIDeviceOrientation type from public accessible property *-orientation*
- Query for elements using XCUIElementQuery (see *Generic querying syntax*)

## Basic functionality
- Launch app in test class's setUp() function to ensure the app launches clean for every test
`override func setUp() {
        super.setUp()
		let app = XCUIApplication()
        app.launch(
}`
*In this ducumentation, when ever I refer to "app" that mean "app" in `let app = XCUIApplication()`*
- Print the accessibility hierarchy
`print(app.debugDescription)`

- Testing if an element exists (even if the element is off the screen)
`XCTAssert(app.staticTexts["Welcome"].exists)`

- Check if a view is on the screen
`let window = app.windows.element(boundBy: 0)
let btnShowMore = app.buttons["Show more"]
XCTAssert(CGRectContainsRect(window.frame, btnShowMore.frame))`

- Testing if textview with given text exist
`let anyTextView = app.staticTexts["Hi John"]
XCTAssert(anyTextView.exists)`

- Waiting for an element to appear
`let detailsLabel = app.staticTexts["Details"]
XCTAssertFalse(detailsLabel.exists)

let exists = NSPredicate(format: "exists == true")
expectation(for: exists, evaluatedWithObject: detailsLabel, handler: nil)

app.buttons["Show details"].tap()
waitForExpectations(timeout: 5, handler: nil)
XCTAssert(detailsLabel.exists)
//If five seconds pass before the expectation is met then the test will fail.`

+Assertions: just type *XCTAssert* then XCode will show the hint with different type of assertions for you to use (*XCTAssertEquals*,*XCTAssertNotNil*,etc)

## Interacting with system controls
- Tap a button
`app.buttons["Add"].tap()`

- Type into textField
`let textField = app.textFields["Username"]
textField.tap()
textField.typeText("John")`

- Dismiss an alert
`app.alerts["Alert Title"].buttons["Button Title"].tap()`

- Handle system alerts (location services, push notifications, access microphone, contacts or access to your photos.)
*Present a photo access request dialog to the user and dismiss it with the following code. Before presenting the alert add a UI Interuption Handler. When this fires, dismiss with the "Allow" button.*

`addUIInterruptionMonitor(withDescription: "Access Photos") { (alert) -> Bool in
  alert.buttons["Allow"].tap()
  return true
}

app.buttons["Your Photos"].tap()
app.tap() // need to interact with the app again for the handler to fire`

- Slide a slider   
*// set value to 90%*   
`app.sliders.element.adjust(toNormalizedSliderPosition: 0.9)`

- Interact with one wheel picker
`app.pickerWheels.element.adjust(toPickerWheelValue: "Picker Wheel Item Title")`

- Interact with multiple wheels picker: set up accessibility so that different wheels could be identified   
`
	let firstPredicate = NSPredicate(format: "label BEGINSWITH 'First Picker'")
	let firstPicker = app.pickerWheels.element(matching: firstPredicate)
	firstPicker.adjust(toPickerWheelValue: "first value")

	let secondPredicate = NSPredicate(format: "label BEGINSWITH 'Second Picker'")
	let secondPicker = app.pickerWheels.element(matching: secondPredicate)
	secondPicker.adjust(toPickerWheelValue: "second value")`

- Tap on a link in web views UIWebView/WKWebView
`app.links["Tweet this"].tap()`

## Interactions

- Longpress a button
`app.pickers.elementAtIndex(0).pressForDuration(0.1, thenDragToElement: someElement)`

- Swipe action
`app.tables.staticTexts["Swipe me"].swipeUp()`

- Two finger tap
`let view = app.descendantsMatchingType(.Unknow)["myView"]
view.twoFingerTap()`

- Pinch action
`app.maps.element.pinchWithScale(1.5, velocity: 1)`

- Check the current controller's title
`XCTAssert(app.navigationBars["Main menu"].exists)`

- Reorder tableviewcells
*If you have a UITableViewCell with default style and set the text to "Title", the reorder control's accessibility label becomes "Reorder Title". Using this we can drag one reorder control to another, essentially reordering the cells.*
`let topButton = app.buttons["Reorder Top Cell"]
let bottomButton = app.buttons["Reorder Bottom Cell"]
bottomButton.press(forDuration: 0.5, thenDragTo: topButton)

XCTAssertLessThanOrEqual(bottomButton.frame.maxY, topButton.frame.minY)`

- Pull to refresh
*Create a XCUICoordinate from the first cell in your table and another one with a dy of six. Then drag the first coordinate to the second.*
`let firstCell = app.staticTexts["Adrienne"]
let start = firstCell.coordinate(withNormalizedOffset: (CGVectorMake(0, 0))
let finish = firstCell.coordinate(withNormalizedOffset: (CGVectorMake(0, 6))
start.press(forDuration: 0, thenDragTo: finish)`

- Check if view controller is pushed successful
`app.buttons["Details Info"].tap(XCTAssert(app.navigationBars["Details"].exists)`

- Check after popping a view controller (tap on Back button)
`app.navigationBars.buttons.elementBoundByIndex(0).tap()
XCTAssert(app.navigationBars["Volley"].exists)`

## Generic querying syntaxes
*UI Testing uses XCUIElementQuery to query elements in the app's view hierarchy. The syntax creates a buildable set of instructions to drill down to different parts of the screen.*

- `app.labels.element` returns the one and only UILabel
- `app.buttons["Save"]` returns the "Save" button (via accessibility)
- `app.cells[4]` returns the fifth table view cell
- `app.tables.element.cells.elementBoundByIndex(4).tap()`
- `app.tables.element.cells["Call Mom"].buttons["More"].tap()`
	By referencing `-element` you are essentially saying, "I know there is only one of these, I want that one."
- These convenience methods work by calling `-elementMatchingType:identifier:` with `XCUIElementTypeAny` and the string you passed in. Check out `XCUIElementType` for all the options if you want finer grain control.
