---
layout: post
category: web micro log
tags: [android, java]
---

**This is a series of post as I will hopefully build a simple wallpaper app.**

In this post, we will build a simple "guess the number game". There will be a input area, which you enter you guess, and simple prompt to tell you whether your guess was too high or too low.

![lower higher layouts](https://raw2.github.com/chappers/chappers.github.com/master/img/android/lowerhigher/layout.png)

```xml
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:paddingLeft="@dimen/activity_horizontal_margin"
    android:paddingRight="@dimen/activity_horizontal_margin"
    android:paddingTop="@dimen/activity_vertical_margin"
    android:paddingBottom="@dimen/activity_vertical_margin"
    tools:context=".LowerHigher">

    <TextView
        android:text="Enter a Guess. The random number will be between 1-10 inclusive."
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:id="@+id/textView2"
        android:layout_above="@+id/guess_result"
        android:layout_alignRight="@+id/button"
        android:layout_alignEnd="@+id/button"
        android:layout_alignParentTop="true"
        android:layout_alignParentLeft="true"
        android:layout_alignParentStart="true" />

    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="@string/guess_result"
        android:id="@+id/guess_result"
        android:layout_above="@+id/guess"
        android:layout_alignParentLeft="true"
        android:layout_alignParentStart="true"
        android:layout_alignRight="@+id/button"
        android:layout_alignEnd="@+id/button" />

    <Button
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Guess"
        android:id="@+id/button"
        android:layout_alignParentBottom="true"
        android:layout_alignParentRight="true"
        android:layout_alignParentEnd="true" />

    <EditText
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:inputType="numberSigned"
        android:hint="Enter a Guess"
        android:id="@+id/guess"
        android:layout_alignParentBottom="true"
        android:layout_alignParentLeft="true"
        android:layout_alignParentStart="true"
        android:layout_alignTop="@+id/button"
        android:layout_toLeftOf="@+id/button"
        android:layout_toStartOf="@+id/button" />



</RelativeLayout>
```

This should be nothing new regarding to what you would have seen before. Perhaps the only thing slightly more interesting is using the `android:inputType="numberSigned"` which provides the keyboard type to your device.

## `.java`

This is where all the action will happen.

###Setting the random number###

From the image above I have created the game to be between 1-10 inclusive. To do this in Java would look like this:

```java
Random rand = new Random();
int randNum = rand.nextInt(9)+1;
```

This will produce a random integer between 1-10 to the variable `randNum`.

###Adding Activity to the button###

We have previously seen how the button activities is to be added within Java. So the following code would not be too surprising. The general idea is that we want to check the input text, and then alter the output text if it doesn't match. But if it does work we will have a dialogue which tells the user that they have won with a button to reset the game.

Firstly, in order to extract the string from the `EditText` it would look like this:

```java
EditText checkNum = (EditText) findViewById(R.id.guess);
int guessNum = Integer.parseInt(checkNum.getText().toString());
```

This will parse and take in the number which was guessed.

Now we can then compare `guessNum` with `randNum` and determine the next course of action.

```java
if (randNum < guessNum) {
    TextView changeText = (TextView) findViewById(R.id.guess_result);
    changeText.setText("Your guess of " + guessNum + " is too high");
} else if (randNum > guessNum) {
    TextView changeText = (TextView) findViewById(R.id.guess_result);
    changeText.setText("Your guess of " + guessNum + " is too low");
} else {
    // open another layout somehow.
    TextView changeText = (TextView) findViewById(R.id.guess_result);
    // set the alert dialog here
    changeText.setText("You've guessed the correct number!");
    // set alert dialog
    restartGameAlert(v);
}
```

Lastly we have to fill out what the dialogue actually does in the `restartGameAlert` function.

###Creating a Dialogue###

The gist of creation of a dialogue is the follow:

1. Use `AlertDialog.Builder` to create a new `Builder`.
2. Set up the framework of the `Builder`, including things like, the title, message, button text, and button actions (similiar to above)
3. Run the `create()` method to actually create the `AlertDialog`.
4. Run the `show()` method to finally display the dialogue.

The Java code is below:

```java
private void restartGameAlert(View v) {
    AlertDialog.Builder alertDialog = new Builder(LowerHigher.this);

    alertDialog.setTitle("You Win!");
    alertDialog.setMessage("You Win!");
    alertDialog.setNeutralButton("Restart Game", new DialogInterface.OnClickListener() {
        @Override
        public void onClick(DialogInterface dialog, int which) {
            randNum = rand.nextInt(9)+1; // reset the number
            TextView changeText = (TextView) findViewById(R.id.guess_result);
            changeText.setText("Try a guess...");
        }
    });
    AlertDialog alertD = alertDialog.create();
    alertD.show();

}
```

This should produce a simple working game in Android!
