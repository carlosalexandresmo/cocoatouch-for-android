# CocoaTouch for Android

If you are here, I will assume:

  - You are an iOS Developer
  - You like CocoaTouch
  - You need to code Android but you don't want to learn it
  - You want to replicate your iOS code to Android


### 1. Why you should use it?

In Android, Activity is equivalent to UIViewController and it normally looks like:

```sh
public class MainActivity extends AppCompatActivity 
{
    Button button; // Equivalent to iOS UIButton
    ListView tableview; //Equivalent to iOS UITableView
    
    @Override
    protected void onCreate(Bundle savedInstanceState) 
    {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        
        //Link XML to variables
        this.button = this.findViewById(R.id.button);
        this.listView = this.findViewById(R.id.tableview);
        
        //Set action to button
        this.button.setOnClickListener(new View.OnclickListener()
        {
            @Override
            public void onClick(View view) 
            {
               ...
            }
        }
    }
}
```

Not bad. But using this library, it becomes:

```sh
public class MainActivity extends UIViewController 
{
    @IBOutlet(R.id.button)    UIButton button;
    @IBOutlet(R.id.tableview) UITableView tableview;
    
    
    @IBAction(R.id.button) public void click(UIButton sender)
    {
        ...
    }
}
```

Nice right? But it becomes better. The thing I most hate on Android SDK, it's how you can't alloc the activity to be presented and just pass objects. You need to transfer data via bundle and the SDK will create an activity for you. This starts to be annoying when you need to pass a custom object and you have to implement Parcelable or Serializable. This adds lots of lines of code, because Android copy objects between activities. This is an example of how to transfer a custom objects from one Activity to another in Android:

CustomObject.java

```sh
public class CustomObject extends Object implements Parcelable
{
    ... 
    public String name;

    public CustomObject() 
    {
        name = "";
    }
    
    //
    // Parcelable Trash code
    //
    public CustomObject(Parcel in) 
    {
        name = in.readString();
    }
    @Override public int describeContents() 
    {
        return 0;
    }
    @Override public void writeToParcel(Parcel dest, int flags) 
    {
        ...
        dest.writeString(name);
    }
    public static final Parcelable.Creator<CustomObject> CREATOR = new Parcelable.Creator<CustomObject>()
    {
        public CustomObject createFromParcel(Parcel in)
        {
            return new CustomObject(in);
        }
        public CustomObject[] newArray(int size)
        {
            return new CustomObject[size];
        }
    }
}
```
SenderActivity.java

```sh
public void presentReceiverController()
{
    CustomObject customObject = new CustomObject(...);
    
    Intent intent = new Intent(this, ReceiverActivity.class);
    intent.putParcelableExtra("key_customObject", customObject);
    
    this.startActivity(intent);
}
```

ReceiverActivity.java

```sh
public class ReceiverActivity extends AppCompatActivity 
{
    @Override
    protected void onCreate(Bundle savedInstanceState) 
    {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_receiver);
        
        //Get custom Object
        Bundle bundle = getIntent().getExtras();
        CustomObject customObject = bundle.getParcelable("key_customObject");
    }
}
```

Boring, right? Now, forget about Pacelable, Serializable, Bundle and Intent. Just alloc and pass data:

CustomObject.java

```sh
public class CustomObject extends NSObject
{
    ...
    public String name;
}
```

SenderController.java

```sh
public void presentReceiverController()
{
    CustomObject customObject = new CustomObject(...);
    
    ReceiverController controller = this.storyboard.instantiateViewControllerWithIdentifier(R.layout.receiver);
    controller.customObject = customObject;
    
    this.navigationController.pushViewController(controller, true);
}
```

ReceiverController.java

```sh
public class ReceiverController extends UIViewController 
{
    public CustomObject customObject;
    
    @Override public void viewDidLoad()
    {
        super.viewDidLoad();
        // Just access object normally
        NSLog(this.customObject.name);
    }
```


Sweet, right? Simple as you do in iOS.


### 2. How it works?

This framework is based on a Single Activity Application and each ViewController is represented by fragments using composition. 


### 3. How I receive onBackPressed Notifications

if you add this method to your controller, you will receive on back notifications. It's the same of receive a button pressed action. 

```sh
@IBAction(DefaultActions.onBackPressed) public void back(UIButton sender)
{
    //TODO
}
```

If you don't add it, the back button will close you app.

### 4. How Can I get Context or the Activity?

```sh

Activity activity = UIApplication.sharedApplication().activity();

Context context = UIApplication.sharedApplication().context();

```

### 5. Limitations

I created some structures such as UIViewContorller, UIButton, UITableView, UILabel... that I needed to create my app. But there are a lot of to do yet. iOS have a bunch of other elements that needs to be coded. Maybe I'm here looking for contributors. 

I added an example app that use some structures, you should try it.

### 6. //TODO

- Use composition instead of inheritance to simulate Android structures. Inheritance is bad because:
    - UIScrollView can be ViewerPager, ScrollView, HorizontalScrollView, VerticalScrollView.
    - You can not simulate all cases with inheritance.
- Figure out how to use composition and still can set class in XML.
    - Study links bellow and create a copy of android RelativeLayout's attributes to UIView, copy Button to UIButton...
        - http://developer.android.com/training/custom-views/create-view.html
        - http://developer.android.com/guide/topics/ui/custom-components.html
        - http://stackoverflow.com/questions/2695646/declaring-a-custom-android-ui-element-using-xml


### 7. Installation and Configure Project

7.1 Add to Gradle the last version:

```sh
compile 'com.hummingbird:cocoa-touch:0.0.27'
```

7.2 - Create file: MainStoryboard.java

```sh
public class MainStoryboard extends UIStoryboard
{
}
```

7.3 - Go to AndroidManifest and make it the fist launch Activity
```sh
        <activity android:name=".MainStoryboard">
            <intent-filter>
                <action android:name="android.intent.action.MAIN"/>
                <category android:name="android.intent.category.LAUNCHER"/>
            </intent-filter>
        </activity> 
```
7.4 - Create a XML in res/layout named mainstoryboard.xml. (Copy and paste this code)

```sh
<?xml version="1.0" encoding="utf-8"?>
<FrameLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:id="@+id/container">
</FrameLayout>
```

7.5 - Create file: AppDelegate.java

```sh
public class AppDelegate extends UIResponder implements UIApplicationDelegate
{
    @Override public Boolean applicationDidFinishLaunchingWithOptions(UIApplication application, NSDictionary launchOptions)
    {
        return true;
    }
    @Override public void applicationDidBecomeActive(UIApplication application)
    {
        
    }
    @Override public void applicationDidBecomeActive(UIApplication application, int requestCode, int resultCode, Intent data)
    {
        
    }
}
```
7.6 - Override methods in MainStoryboard, declaring the initial UIViewController id and mapping layout ids with classes

```sh
public class MainStoryboard extends UIStoryboard
{
    @Override public UIApplicationDelegate applicationDelegate()
    {
        return new AppDelegate();
    }
    @Override public int containerID()
    {
        return R.id.container;
    }
    @Override public int storyBoardID()
    {
        return R.layout.mainstoryboard;
    }
    @Override public int initialViewControllerID()
    {
        return R.layout.viewcontroller;
    }
    @Override public UIViewController viewControllerForIdentifier(int identifier)
    {
        switch (identifier)
        {
            case R.layout.viewcontroller:
                return new ViewController();
            default:
                throw new Resources.NotFoundException();
        }
    }
}
```

The initialViewControllerID is the id of the first ViewController. Your app will start there. You can set any ViewController you want.

### 8. Add new UIViewControllers to your project:

- Create a new java file and make your class inherit UIViewController
- Create a XML file in res/layout
- Go to MainStoryboard
    - Find the method viewControllerForIdentifier
    - Add the case R.laytout.YOUR_LAYOUT_NAME: return new YOUR_CLASS_NAME();
    
Example:

File: TutorialController.java
```sh
public class TutorialController extends UIViewController
```

File: tutorialcontroller.xml

Create it in the folder res->layout. You can edit your canvas and work with pure XML. Yeah you will need to learn how to handle pure XML for Android. (I miss you XCODE interfacebuilder...).

File: MainStoryboard.java
```sh
public class MainStoryboard extends UIStoryboard
{
    ...
    @Override public UIViewController viewControllerForIdentifier(int identifier)
    {
        switch (identifier)
        {
            ...
            case R.layout.tutorialcontroller:
                return new TutorialController();
            default:
                throw new Resources.NotFoundException();
        }
    }
}
```

You have your new UIViewController set =) 


### 9. How I created it?

- Think in one example or structure in iOS I would like to create
- Code the example in iOS
- Stackoverflow how to do the same/similar in Android
- Code it in Android and make it works
- Create classes to simulate iOS but do the Android code under the table

### 10. Enjoy

You can contact me here or just send me an email: rafael.castro@hummingbird.com.br

### 11. License

Apache-2.0

