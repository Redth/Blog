# Building a Step-by-Step / Wizard Control in .NET MAUI

Recently I've been converting one of my apps over to .NET MAUI. Now, I'm obviously biased as the engineering lead for the product, but the migration of my app has been going quite well, and despite a few remaining rough edges that we continue to work on improving, .NET MAUI has come a ***long way*** since the initial release earlier this year.

As part of the migration, I've started adding a new onboarding experience in my app. The effect I am going for is a sort of wizard or step-by-step guide through entering the information that the app requires to be useful to a new user.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1671727971914/749a7a66-6cee-4ab2-a096-8218d11580ed.gif align="center")

I've distilled this experience into a few different requirements:

*   Each step should be its own 'screen'
    
*   Moving forward and backwards between steps should animate the next/previous screen sliding back and forth from the current screen
    
*   Each screen may require validation to move to the next (ie: required fields)
    

## Which control to use, and why isn't it CarouselView?

One of the reasons I value building apps with .NET MAUI in my spare time is to get a better understanding of what the actual product experience is like for customers, where some of the non-obvious pain points are, and to generally better understand the areas we need to focus on most.

The first thought that came to mind was to use a [CarouselView](https://learn.microsoft.com/dotnet/maui/user-interface/controls/carouselview/), so I initially started down this path. Now, I mentioned there are still some rough edges in the product, and full disclosure: one of those areas is CarouselView.

There were a few issues I ran into:

### 1\. CarouselView does not support direct/static content for items

The CarouselView is really good at displaying many different items for large lists of data, but since I am not repeating the same layout for any of my items, it's a bit of an awkward fit.

But, at this point I thought, no matter, and started down the path of creating a [DataTemplateSelector](https://learn.microsoft.com/dotnet/maui/fundamentals/datatemplate?#create-a-datatemplateselector) and DataTemplates for each of my unique items, and finally a data source to assign the CarouselView. The template selector would be responsible for returning each step's view/DataTemplate given the data source item (one for each step).

This is a workable solution, but admittedly a bit clunky...

### 2\. Accessibility Quirks

The .NET MAUI team has spent a significant amount of time and effort improving the accessibility support in the product, and trying to make it easy for every developer to create highly accessible and inclusive apps. Part of this effort has involved removing hacks upon hacks from Xamarin.Forms implementations and letting the underlying platform do its thing, intervening only when necessary. This has proven to be a very successful strategy and we are leaps and bounds ahead of where we were in Xamarin.Forms.

The current accessibility story is ok and workable on CarouselView, however there were two specific quirks that I was determined not to settle on:

*   [CarouselView items are focusable · Issue #12271 · dotnet/maui (](https://github.com/dotnet/maui/issues/12271)[github.com](http://github.com)[)](https://github.com/dotnet/maui/issues/12271)  
    This one is more cosmetic and less of an actual problem, but when tabbing through items, there's an extra focusable item which is the entire CarouselView item.
    
*   [CarouselView IsSwipeEnabled="False" still scrolls with tab navigation · Issue #12272 · dotnet/maui (](https://github.com/dotnet/maui/issues/12272)[github.com](http://github.com)[)](https://github.com/dotnet/maui/issues/12272)  
    This one was more of a deal breaker for me. On Android, you can bypass the disabled swiping between items by 'tabbing' through to the next item's focusable controls. This means there's potentially a way for the app to get into a bad state if the validation wasn't passing before moving on from the current step.
    

## Build your own control!

I think developers often overlook their own skills when they run into this type of situation. It's tempting to want and even *expect* everything to work perfectly out of the box for whatever use case you are hoping to achieve, however, it's impractical if not impossible to expect this.

Instead of giving up here, I decided this was a good opportunity to flex some creativity, maybe learn a new thing, and get my hands dirty.

The effect I am after is not that challenging if you break it down into basics. I decided that as a minimum viable approach, one option was to simply hide and show the different step views as the user progressed forwards and backwards between steps. This could be achieved by placing all of my step views inside a single grid, and just toggling each view's visibility.

```xml
<Grid>
  <local:SetupWizardWelcomeView IsVisible="True" />
  <local:SetupWizardPool1View IsVisible="False" />
  <local:SetupWizardPool2View IsVisible="False" />
  <local:SetupWizardPool3View IsVisible="False" />
  <local:SetupWizardPool4View IsVisible="False" />
</local:WizardStepsControl>
```

Now, I really did want some sliding/scrolling animations just like a CarouselView, so I decided to explore the animations API a bit. It's quite trivial to animate the effect of sliding the currently visible view out of place, and the 'next' view into place, something like this:

```csharp
// Prepare the 'next' view to show, moving it out of 
// view, by setting its x translation to the right of
// our container (container's width)
nextView.TranslationX = this.Width;
// Make the 'next' view visible so we see it sliding in
// now that it's translated outside of the container view and not 'seen'
nextView.IsVisible = true;

// Animate the translation of both the 'next' and 'current'
// views so that we get a slide effect
// The 'next' view slides in from the right and the
// current view slides out of view to the right
await Task.WhenAll(
  nextView.TranslateTo(0, 0, 500, Easing.CubicInOut),
  currentView.TranslateTo(-1 * this.Width, 0, 500, Easing.CubicInOut));

// Reset the visibility and translation of the
// current (now previous) view now that the animation is complete
currentView.IsVisible = false;
currentView.TranslationX = 0;
```

That's the magic! We can reverse the X translation numbers to go 'back' and simulate sliding the views out the other direction just as easily.

Now, to put this all together, it's useful to create a custom control to encapsulate the animation logic and keeping track of the currently visible item, and navigating between steps. To do this, I subclassed `Grid` and added just a bit of logic to my new control.

### Managing visibility of children

Since I am subclassing `Grid` (`public class WizardStepsControl : Grid { /* .. */ }`) which is a `Layout`, I found it easiest to override the `OnChildAdded(Element child)` to ensure any views added are hidden immediately unless there are no existing visible items yet:

```csharp
protected override void OnChildAdded(Element child)
{
	if (child is VisualElement ve)
	{
		ve.IsVisible = false;

		if (GetCurrentIndex() < 0)
		{
			ve.IsVisible = true;
		}
	}

	base.OnChildAdded(child);
}
```

I probably should also override the `OnChildRemoved` method to ensure there's always one view visible, if there is at least one child view, however for my own use case, I know I will never be removing items dynamically, and it's a corner case I need not worry about right now.

The `GetCurrentIndex()` call here is simply a method that loops through the `Children` of the `Layout` and returns the index of the first child with `IsVisible=true`:

```csharp
public int GetCurrentIndex()
{
	for (var i = 0; i < Children.Count; i++)
	{
		if (Children[i] is VisualElement ve && ve.IsVisible)
			return i;
	}

	return -1;
}
```

To move between 'steps' I added `Forward()` and `Back()` methods:

```csharp
public async Task Forward()
{
	var c = GetCurrentIndex();

	var currentIndex = c;
	var nextIndex = c+1;

	if (nextIndex >= Children.Count)
		nextIndex = 0;
	if (currentIndex == nextIndex)
		return;

	var currentView = Children[currentIndex] as VisualElement;
	var nextView = Children[nextIndex] as VisualElement;

	// Prepare the 'next' view to show, moving it out of 
	// view, by setting its x translation to the right of
	// our container (container's width)
	nextView.TranslationX = this.Width;
	// Make the 'next' view visible so we see it sliding in
	// now that it's translated outside of the container view and not 'seen'
	nextView.IsVisible = true;

	// Animate the translation of both the 'next' and 'current'
	// views so that we get a slide effect
	// The 'next' view slides in from the right and the
	// current view slides out of view to the right
	await Task.WhenAll(
		nextView.TranslateTo(0, 0, 500, Easing.CubicInOut),
		currentView.TranslateTo(-1 * this.Width, 0, 500, Easing.CubicInOut));

	// Reset the visibility and translation of the
	// current (now previous) view now that the animation is complete
	currentView.IsVisible = false;
	currentView.TranslationX = 0;

    // Invoke an event to know the step changed
	StepChanged?.Invoke(this, new StepChangedEventArgs(currentIndex, nextIndex));
}
```

The animation code in there should be familiar from above.

I won't repeat the code for the `Back()` method because it is almost identical to the one above, with the exception of `TranslateX` values and checking if `if (nextIndex < 0)` and looping to the end with `nextIndex = Children.Count - 1;` instead of the beginning (`0`) in our `Forward()` call.

I've also included a simple event with event args to raise when the step changes so that I can subscribe to it from my page.

```csharp
public event EventHandler<StepChangedEventArgs> StepChanged;

public class StepChangedEventArgs : EventArgs
{
	public StepChangedEventArgs(int previousStepIndex, int stepIndex)
		: base()
	{
		PreviousStepIndex = previousStepIndex;
		StepIndex = stepIndex;
	}

	public int PreviousStepIndex { get; }
	public int StepIndex { get; }
}
```

Finally, I can add the control to my XAML:

```csharp
<local:WizardStepsControl
  x:Name="stepsControl"
  StepChanged="stepsControl_StepChanged">
    <local:SetupWizardWelcomeView />
    <local:SetupWizardPool1View />
    <local:SetupWizardPool2View />
    <local:SetupWizardPool3View />
    <local:SetupWizardPool4View />
</local:WizardStepsControl>
```

And in my Page, I can simply call `await stepsControl.Forward()` to Navigate to the next step!

## That wasn't so hard!

Ultimately my custom control ended up being just over 100 lines of code to achieve the exact effect and functionality I was after. It's a good reminder that sometimes it's really not that difficult to roll your own solution to a problem.

With this in mind, .NET MAUI can be a very powerful platform to build your apps with, even while we continue to polish (and we will!!!) some of the rough edges.