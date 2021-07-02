---
layout: post
category : web micro log
tags : [android, java]
---

**This is a series of post as I will hopefully build a simple wallpaper app.**

In this app, we will build a simple counter which counts the number of times a button in the app was pressed.

Firstly we should design layout of the app. It should have text which is the counter and a button. Be give the button and the counter id's so that it can be referenced in our code.

Here is my `.xml`, it is okay if your one is different!

```xml
    <TextView
        android:text="@string/hello_world"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_above="@+id/button_counter"
        android:layout_alignParentTop="true"
        android:layout_alignParentLeft="true"
        android:layout_alignParentStart="true" />

    <Button
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="The Button"
        android:id="@+id/button"
        android:layout_alignParentBottom="true"
        android:layout_alignParentLeft="true"
        android:layout_alignParentStart="true"
        android:layout_alignParentRight="true"
        android:layout_alignParentEnd="true" />

    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="@string/button_counter"
        android:id="@+id/button_counter"
        android:layout_above="@+id/button"
        android:layout_alignParentLeft="true"
        android:layout_alignParentStart="true"
        android:layout_alignParentRight="true"
        android:layout_alignParentEnd="true" />
```

Now that the layout is complete, we may want to add or alter the values in `string.xml`. Here is what my one looks like. 

```xml
    <string name="app_name">SimpleButton</string>
    <string name="hello_world">Hello world! This keeps track of the number of times you pressed the button</string>
    <string name="action_settings">Settings</string>
    <string name="button_counter">0</string>
```

Since all of this should be simple and revision, lets consider the `.java` code. Similar to my previous post, in order to add the activity, we will have define the activity after `onCreate` function.

The auto-complete function in Android studio will give you the boiler plate if you choose to use it. It will look something like this:

```java
        Button button = (Button) findViewById(R.id.button);
        button.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {

            }
        });
```

Now we simply have to define the activity within `onClick` to incrememt the text in our layout. One way would be to create a counter within the class and increment it globally, assigning it via `setText()`. 

This would look something like this:

```java
    public void incrementCount() {
        // increments the counter counting the number of times the button was pressed.
        TextView incrementCounter = (TextView) findViewById(R.id.button_counter);
        counter++;
        incrementCounter.setText(""+counter);
    }
```

Placing this function within `onClick` should provide the solution.