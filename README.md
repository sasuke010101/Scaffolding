# Introduction to Scaffolding

## What is Scaffolding
Scaffolding is a MV framework for Unity that Preloaded has used for a number of projects. It is created and maintained by Jon Reid, with any feedback from the team being merged in so it is an ongoing and growing framework.

## Future plans
- [ ] Transitions
- [ ] Audio framework
- [ ] Refactor ViewManagerBase to Facade pattern
- [ ] Basic graphics for generated views
- [ ] Graph based code generation to quickly link up and edit flows

## Structure
Scaffolding is made up of a few components, the view library which is the central editing tool for Scaffolding, the views themselves, and inputs.

Ideally, views shouldn’t contain much logic, they should just be there for the visual representation. The Model should contain the logic of what should be shown on the view itself, with the view requesting information when it loads. This can be done in Setup so that the data is loaded during any transitions.

There should be sub components in a view which are responsible for smaller behaviours, such as a carousel, or any specific view components. 

Game or app data should be loaded into a proxy or a model, never in a view itself. Proxies are preferred, e.g a GameDataProxy, that the model can then request specific data from, which is passed into the view. Views should be as detached as possible, which enables Unit testing and integration testing to be performed as well as other testing, such as the potential to run the app headlessly for speed testing, or skinning.

## Structure of a view
`Setup();` - Called as soon as the view is created.

`OnShowStart(SObject data);` - Called directly after Setup.

`OnShowComplete();` - Called by OnShowStart, either after the InAnimation has completed, or immediately. Can be called manually by not calling base in OnShowStart. E.G for coded in animations.

`OnHideStart();` - Called when Scaffolding processes the request to change view.

`OnHideComplete();` - Called by OnHideStart, either after the OutAnimation has completed, or immediately. Can be called manually by not calling base in OnHideStart. E.G for coded out animations.


## What is the view library
The View Library is the easiest way to navigate through all the views in the project. Its a central point for managing views, creating new ones, deleting existing ones and setting the starting view.

Its a single menu that is easy to just dock as a standard part of the UI as its something you’ll be most likely be using constantly.



## How to open the view library
It’s simple to open the View Library, just do the following:
![open view library](http://jonsgames.com/public/images/scaffolding/readme/open_view_library.png)
This will open the following panel:
![open view library](http://jonsgames.com/public/images/scaffolding/readme/view_library2.png)
Just dock this panel into part of the unity UI.

## Make a new view

In the View Library, click on the “Create New” button.
![Make a new view](http://jonsgames.com/public/images/scaffolding/readme/view_library1.png)

This will open up the “Create New View” panel. 
![Make a new view](http://jonsgames.com/public/images/scaffolding/readme/create_new_view2.png)

Here you will be presented with all the settings you will need to create a new view. 

The name. This should be something descriptive such as MainMenuView, TutorialView, PauseView etc.
The base class it extends from. By default, this is AbstractView. But if you wanted to create your own base class that has specific behaviours each view needs, then you can extend from that instead and set that here.
Create a model. This will generate a standard Model class for this view. The view will automatically register to the Model when it is opened and closed. This allows for quick communication between the view and model.
Add a canvas. Not always required, even with Unity 4.6+ as you can put the canvas inside the view. Mostly just a preference.
Directories. You can choose where to save the script and the prefab parts of the view in your project. These are setup within the “Settings” panel in the View Library.

## Request a view
As there can only be one view open at a time, you need to request a view to move to the next view in the flow. This is done through the very simple call:
`RequestView<MyViewName>();`

## Request an overlay
Requesting an overlay is very much like requesting a view, however, there can be as many overlays open at a time as you like.
To do so, call:
`RequestOverlay<MyOverlayName>();`

However, there is an extra parameter that can be passed through to the overlay, whether or not you want to disable all inputs on overlays underneath. 

## Close overlay
Overlays can close themselves if needed, as well as other views or overlays. To do so, call:
`RequestOverlayClose<MyOverlayName>();`
## Inputs
Scaffolding manages all its internal inputs that have been created with the AbstractInput class. This class is the base for all ScaffoldingButtons, ScaffoldingUGUIButtons and others. If you want to create your own button or input item, it is HIGHLY recommended that you use the AbstractInput class as a base as this provides you with all the needed functions to get multitouch input from Scaffoldings InputManager as well as making sure your input is enabled and disabled during and after transitions as well as disabling inputs underneath overlays to stop clickthroughs.

If you are using Unity 4.6+, it’s advised to use the ScaffoldingUGUIButton as this has a number of shortcut methods, including:
Getting the button quickly by calling `GetButtonForName(“NameOfButton);`
Registering button callbacks with `AddButtonPressedHandler(MyHandleButtonPressedFunction);`
as well as internal state management.

Example class:
``` c#
using UnityEngine;
using Scaffolding;

public class MainMenu : AbstractView {
	 
    public override void Setup(ViewManagerBase manager)
    {
       base.Setup(manager);
       //you should call these button shortcut methods AFTER setup, as that is where the button references are created.
       //The “NoButton” variety means there is no reference of the button passed in. Great for when there is only a few buttons.
		GetButtonForName(“Next”).AddButtonPressedHandlerNoButton(NextPressed);
    }

    public override void OnShowStart(SObject data)
    {
        base.OnShowStart(data);
    }

    public override void OnShowComplete()
    {
        base.OnShowComplete();
    }

    public override void OnHideStart()
    {
        base.OnHideStart();
    }

    public override void OnHideComplete()
    {
        base.OnHideComplete();
    }

    private void NextPressed()
    {
	   Debug.Log(“My button has been pressed!”);
    }
}
```

If you need to create your own input, but are unable to extend from AbstractInput for whatever reason, you can do so by registering for input events from the input manager like so;

``` c#
private InputManager _inputManager;
private InputTracker _currentTracker;
	 
	public override void Setup(ViewManagerBase manager)
    {
        base.Setup(manager);

		_inputManager = FindObjectOfType<InputManager>();
		_inputManager.EventPressed += HandleEventPressed;
		_inputManager.EventReleased += HandleEventReleased;
		_inputManager.EventDragged += HandleEventDragged;
    }

	void HandleEventPressed (InputTracker tracker)
	{
		if(_currentTracker == null)
		{
			_currentTracker = tracker;
		}
	}

	void HandleEventDragged (InputTracker tracker)
	{
		if(tracker == _currentTracker)
		{
			//do some dragging code.
		}
	}

	void HandleEventReleased (InputTracker tracker)
	{
		if(_currentTracker == tracker)
		{
			_currentTracker = null;
		}
	}
```

The example code above registers for the basic press, drag and release events. It also shows you how the input trackers work. There is a single tracker for each finger or input on the screen. This allows you to explicitly check a cached input tracker against the one passed in from the event. 

Doing this, you can limit the multitouch code to a single touch, or you can handle all the touches if you would like. 

The tracker item contains a lot of information about the active input, e.g. position, velocity and what item its currently raycast against. This will just use the main camera by default, but if you wanted to raycast against a specific camera, you can pass that in by doing the following:

`_inputManager.AddCameraToInputCameras(_myCamera);`

And then check for inputs on the tracker by doing:

``` c#
if(tracker.HitGameObject(_targetObject))
{
		//Hit my object!
}
```

The input system underlying scaffolding is really solid, and is tied closely to the view system, allowing for view level mass management of inputs. If you want to use your own raycasts, you can, but be careful to enable and disable them when transitioning between views.
## Pass data to views
When requesting a view or overlay, you can pass data in in the form of an SObject. This is just a holder class for various data types.

Data is sent in in the following way:

``` c#
SObject data = new SObject();
data.AddInt(“PlayerScore”, _score);	
data.AddString(“PlayerName”, _playerName);
SendDataToView<GameOverView>(data);
```



Retrieving data is in a view is done during the OnShowStart() function:
``` c#
public override void OnShowStart(SObject data)
{
    base.OnShowStart(data);

    if(data != null)
    {
        if(data.HasKey(“PlayerScore”))
        {
			_playerScore = data.GetInt(“PlayerScore”);
		}

		if(data.HasKey(“PlayerName”))
		{
			_playerName = data.GetString(“PlayerName”);
		}
    }
}
```

Data is checked for null only because data is not needed to open a view. Its possible for you to request a view with and without data if the flow originates from a different point.

# Newer features - Popups and transitions

## Popups
Modal popups are a new feature for Scaffolding, its just a simpler way of creating the modal popup without having to write the same code each project. The popup is built in a way to force component reuse, as its rare that each popup in your game or application would be visually different.

The modal popup exists in two pieces, the view prefab itself, and the code. 
The code for the view is straightforward, as its largely about setting text on the various components. 

First you’ll need to create the popup view. To do this, create a view as you have done previously, but set the base class to AbstractModalPopup.
![Popups](http://jonsgames.com/public/images/scaffolding/readme/create_new_popup.png)

This will create the various view components as needed, and set the underlying modal code for the popup.

The example AreYouSurePopup in the scaffolding project has the following structure:
![Popups](http://jonsgames.com/public/images/scaffolding/readme/popup_heirarchy.png)

The only important part of the structure here is to name the OK and dismiss buttons as they are shown above. This is because the modalPopup code is looking for these buttons by name to make sure the callbacks are correct.

If you are making your own popup, with your own hierarchy, please make sure that you have Scaffolding buttons named “ButtonOK” and “ButtonDismiss” in the popup.

You are not restricted to two buttons, the popup will also work with a single “ButtonOK” popup, and the code for requesting the view also has tis variant built in.

To request a popup, use the following example:
``` c#
RequestModalPopup<AreYouSurePopup>(OKPressedHandler,”OK”, DismissPressedHandler, “Cancel”, “Are you sure?”);
```


## Transitions
Transitions are a newer part of Scaffolding, and are largely untested. The idea is to create  a system that can present a fullscreen transition masking the loading of a view.

The project includes a few basic transitions, including DoorTransition, which is a basic, but effective transition.

To request a view with a transition, call the following code:

``` c#
TransitionTo<IntroView,DoorsTransition>();
```

And once you know that IntroView has finished loading, you can then call LoadComplete(); 
E.g:

``` c#
public override void OnShowComplete()
{
	base.OnShowComplete();
	LoadComplete();
}
```

Transitions are essentially an overlay which extends AbstractTransition instead of AbstractView. 

Transitions are made up of an InTransition and an OutTransition, with the idea being that you can mix and match transitions, setting them up in the editor.

Taking a look at the DoorsTransition, you can see there is an InTransition and an OutTransition, these are just looking after the animation direction, and can be any class that extends TransitionComponentBase. So if you wanted to make a custom transition, you would make a new transition view, which extends AbstractTransition, as well as a new transition component which extends TransitionComponentBase.

![Transitions](http://jonsgames.com/public/images/scaffolding/readme/doors_component.png)

As Transitions are still an ongoing feature, either take a look at how the example transitions are constructed, or ask me directly for further information.