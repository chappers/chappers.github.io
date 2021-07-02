---
layout: post
category : 
tags : 
tagline: 
---

Because I couldn't find an example onlline...Here is a "combined" example to respond to keyboard events based on compiling some examples online. 

Now time to automatically port some stuff over...


```
import kivy
# kivy.require('1.0.8')

from kivy.app import App
from kivy.core.window import Window
from kivy.uix.widget import Widget
from kivy.uix.gridlayout import GridLayout
from kivy.uix.boxlayout import BoxLayout
from kivy.uix.label import Label

class MyKeyboardListener(BoxLayout):

    def __init__(self, **kwargs):
        super(MyKeyboardListener, self).__init__(**kwargs)
        self.info = Label(text="Hello World!\n123", font_name="RobotoMono-Regular")
        self.add_widget(self.info)
        self._keyboard = Window.request_keyboard(
            self._keyboard_closed, self, 'text')
        if self._keyboard.widget:
            # If it exists, this widget is a VKeyboard object which you can use
            # to change the keyboard layout.
            pass
        self._keyboard.bind(on_key_down=self._on_keyboard_down)
        

    def _keyboard_closed(self):
        print('My keyboard have been closed!')
        self._keyboard.unbind(on_key_down=self._on_keyboard_down)
        self._keyboard = None

    def _on_keyboard_down(self, keyboard, keycode, text, modifiers):
        print('The key', keycode, 'have been pressed')
        print(' - text is %r' % text)
        print(' - modifiers are %r' % modifiers)

        key_register = modifiers + [text]
        self.info.text = "Key input received is:\n{}".format(key_register)

        # Keycode is composed of an integer + a string
        # If we hit escape, release the keyboard
        if keycode[1] == 'escape':
            keyboard.release()

        # Return True to accept the key. Otherwise, it will be used by
        # the system.
        return True

class MyApp(App):
    def build(self):
        # return Label(text="hello")
        return MyKeyboardListener()


if __name__ == '__main__':
    MyApp().run()
```
