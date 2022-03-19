# Dynamic UIScrollView content height with UIStackView

## Introduction
In iOS development with UIKit you often face the problem that the content of a `UIScrollView` is not statically defined. You may get text for a label from user input or through an API.

As there are many different device sizes, it's important to make the child views of the scroll view grow dynamically with their content.

## Possible solutions

| Solution | Applicable | Note |
| ----------- | ----------- | ----------- |
| Calculate the height on your own | ❌ | This could work, but it requires a lot of code and can be error-prone when it comes to `UILabel`s.  |
| Setup the constraints correctly and use a `UIStackView` | ✅ | While you don't necessarly need to use a stack view, this often makes it easier for dynamic content. |

In this article I'm going to describe the latter as this is the easier and stable approach.

## Setup the constraints correctly and use a `UIStackView`

### 0. Prerequisites

- You have a Xcode iOS project.
- You have a `UIViewController` that can be accessed in your app, so you can test the solution.
- You have a Storyboard setup (this approach also works in code, but for this guide I'm going to use a Storyboard-based setup).

### 1. Add the `UIScrollView` to the Storyboard scene

Open the Storyboard that includes your `UIViewController` that should have the dynamic scroll view. 

![The empty Storyboard](/UIKit/DynamicUIScrollView/media/image1.png)

Add a scroll view on top of the current view. For that, you can use the keyboard shortcut `Shift + CMD + L` to open the object library. Type in scroll view and drag & drop the scroll view to the current view.

![Added the scroll view](/UIKit/DynamicUIScrollView/media/image2.png)

You now need to set the correct constraints so the `UIScrollView` knows its own size. 

> ⚠️ Very important: The scroll view itself doesn't have a dynamic height. It's size should also be smaller or equal the size of its parent. The only thing that can be larger is the content of the scroll view.

Set top, trailing, bottom, leading constraints to 0 of the parent view (this could also be smaller, depending on the use case).

![Correct constraints for the scroll view](/UIKit/DynamicUIScrollView/media/image3.png)

Right now, Xcode still complains about the missing constraints. Therefore, it won't update the size of the scroll view accordingly. I tend to change the scroll view size by dragging the edges of the scroll view to their respective size.

![Dragged the scroll view to the correct size](/UIKit/DynamicUIScrollView/media/image4.png)

### 2. Add the `UIStackView` inside of the `UIScrollView`

Next, we want to add the stack view in a way that it can grow on its own and also resolves all the missing constraint issues that Xcode is complaining about.

Again open the object library and select and drag the 'Vertical Stack View' on top of the scroll view.

![Added the stack view accordingly](/UIKit/DynamicUIScrollView/media/image5.png)

Before setting up the constraints, make sure to remove the content layout guides of the scroll view by selecting the scroll view and unchecking the checkbox 'Content Layout Guides' in the Size Inspector (`Option + CMD + 6`).

![Uncheck the Content Layout Guides checkbox](/UIKit/DynamicUIScrollView/media/image6.png)

Now add the same constraints to the stack view as well. Top, trailing, bottom, leading all should be set to 0.

![The basic stack view constraints](/UIKit/DynamicUIScrollView/media/image7.png)

However, this alone is not enough. The stack view (or the content of the scroll view) need to have an anchor point for the width as well that is more than just being the same size.

That's why we need a constraint from the stack view to the root view. Use control-drag on your stack view to your 'View' in the current scene.

In the pop-up choose the option 'Equal widths'. 

![Select equal widths in the popup](/UIKit/DynamicUIScrollView/media/image8.png)

This may just create a proportinal width. If that's the case click on edit next to the proportional width constraint in the Size Inspector and set the multiplier to 1.

![Make sure it does not only set a proportinal width but really the same width by setting the constraint multiplier to 1](/UIKit/DynamicUIScrollView/media/image9.png)

Right now, you still see the warnings of Xcode. The `UIStackView` has no elements and therefore no height. But as soon as you add another element. Let's say a `UILabel` to the stack view, all missing constraints issue should be gone.

Now your stack view will grow accordingly and take as much space as it needs for the content.

### 3. A small example

In this view I added 3 views (2 labels and an image view). 

![An example of with 3 views (2 labels and one image view)](/UIKit/DynamicUIScrollView/media/image10.png)

I set different background colors for all these views. However, the stack view needs to know about the size of the image. I simply set the height constraint to be 200.

Additionally, I set the number of lines for both labels to be 0 which means they take as much lines as they need.

Lastly, I added outlets from the labels to the `ViewController` that we are using. For this, simply open the `ViewController` in an assistant editor (`Option + Click`). Then control drag from the label to the assistant editor and give the outlet a name.

![Make sure you create outlets from your labels to your ViewController)](/UIKit/DynamicUIScrollView/media/image11.png)

Now in `viewDidAppear(_:)` method of the view controller set the labels' text attribute to any value you want.

![Here we set the correct values for both labels in viewDidAppear)](/UIKit/DynamicUIScrollView/media/image12.png)

> ⚠️ Very important: Don't use `viewDidLoad()` for this, as the outlet connection is not done which will result in a crash of the app.

In the example below I used smaller text which won't fill the entire screen and also the content will then not fill more than it needs.

![The first example in the Simulator where the content is smaller than the scroll view height](/UIKit/DynamicUIScrollView/media/image13.png)

In the next example we set a longer text and the stack view is larger than the scroll view (which still is only as large as it's parent). But also the scroll does work as can be seen below.

![Here you can see the scrolling in action because the stack view is larger than the scroll view)](/UIKit/DynamicUIScrollView/media/scrollGif.gif)

And the code for the second example.

![Here we set the correct values for both labels in viewDidAppear this time both texts are longer](/UIKit/DynamicUIScrollView/media/image14.png)

Very important here is, that there were no changes needed. You only need to set the correct text and the dynamic content height of the scroll view takes care of the rest.






