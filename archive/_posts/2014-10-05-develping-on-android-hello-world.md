---
layout: post
category: web micro log
tags: [android, java]
---

**This is a series of post as I will hopefully build a simple wallpaper app.**

When creating a new application using [Android Studio](https://developer.android.com/sdk/installing/index.html?pkg=studio), you will immediately be given a "Hello World!" boilerplate. But how can you extend it?

But firstly, let's have a quick look at the different files which build up an Android app.

## `res/layouts`

I like to think of layouts as your HTML code. It tells you where everything sits. Within Android studio, there is a simple point and click template which allows you to easily and quickly generate something to your liking.

![Point and click layouts](https://raw2.github.com/chappers/chappers.github.com/master/img/android/hello-world/designlayout.png)

The `.xml` file will then automatically be generated to reflect any changes you make. So what controls the "Hello World!" text?

This line within the `TextView` does it:

```xml
android:text="@string/hello_world"
```

## `res/values`

This is where your variables are listed for your layouts. See within `strings.xml` you have the resulting "Hello World!" string here:

```
<string name="hello_world">Hello world!</string>
```

But its always more interesting to be able to change these things within code. Lets look at how to do that next.

## Code

To be able to change the text within `.java` code, simply reference the field that you wish to change and the use the method `.setText()`. We then control when and how the text is to be changed. Does it change:

- When you first load up the application?
- When you change something in settings?
- When you click on something?

There are many aspects which we have to think about when and how we should change text. But for starters, lets just change it `onCreate`. You could see the `onCreate` method within your `.java` file. Simply add:

```java
TextView changeText = (TextView) findViewById(R.id.???);
changeText.setText("Hello Again");
```

But wait! what is the `id` which is referenced here?

If we go back to the `.xml` file within `res/layout`, we should add:

```java
android:id="@+id/YOUR_ID"
```

Then when you run the emulator you should be able to see the text updated to "Hello Again".

## Finally

Your files should look something like the ones below:

`res/layout`:

```xml
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:paddingLeft="@dimen/activity_horizontal_margin"
    android:paddingRight="@dimen/activity_horizontal_margin"
    android:paddingTop="@dimen/activity_vertical_margin"
    android:paddingBottom="@dimen/activity_vertical_margin"
    tools:context=".HelloActivity">

    <TextView
        android:id="@+id/hello_world"
        android:text="@string/hello_world"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content" />

</RelativeLayout>
```

`src`:

```java
import android.support.v7.app.ActionBarActivity;
import android.os.Bundle;
import android.view.Menu;
import android.view.MenuItem;
import android.widget.TextView;


public class HelloActivity extends ActionBarActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_hello);
    }


    @Override
    public boolean onCreateOptionsMenu(Menu menu) {
        // Inflate the menu; this adds items to the action bar if it is present.
        getMenuInflater().inflate(R.menu.hello, menu);

        TextView changeText = (TextView) findViewById(R.id.hello_world);
        changeText.setText("Chappers!");
        return true;
    }

    @Override
    public boolean onOptionsItemSelected(MenuItem item) {
        // Handle action bar item clicks here. The action bar will
        // automatically handle clicks on the Home/Up button, so long
        // as you specify a parent activity in AndroidManifest.xml.
        int id = item.getItemId();
        if (id == R.id.action_settings) {
            return true;
        }
        return super.onOptionsItemSelected(item);
    }
}
```

## Changing Fields to be Editable

To simply change text to be editable, all we have to do is change the "TextView" to EditText" in the `.xml` file. Then in our code, instead of declaring a "TextView" variable, we use an "EditText" variable, setting the `TextView.BufferType.EDITABLE`, i.e. :

```xml
    <EditText
        android:id="@+id/hello_world"
        android:text="@string/hello_world"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
    />
```

and

```java
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_simple_text);
        EditText editText = (EditText) findViewById(R.id.hello_world);
        editText.setText("Hello World...", TextView.BufferType.EDITABLE);
    }
```
