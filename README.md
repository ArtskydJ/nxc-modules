nxc-modules
===========

Handy little modules for random NXT stuff!

For the NXT, written in C, compiled with the free NXC compiler.

- None of the functions mutate the inputs.
- Everything should work with the default firmware, unless it specifically says otherwise.
- If something doesn't work, or has a dumb api, create an issue!


## Including a module into your code

Currently, I expect a folder structure like this:
```
- /code
	|- /nxc-modules
	|	|- Button+-.nxc
	|	|- Menu.nxc
	|	|- ...
	|
	|- /your-project
		\- your-file.nxc
```

And then `your-file.nxc` will do `#include "../nxc-modules/Button+-.nxc"`

Note that some nxc-modules will `#include` other nxc-modules. They use `../nxc-modules/file_to_include.nxc` so that the relative path works from `/your-project` as well as from `/nxc-modules`. As far as I can tell, the NBC compiler always resolves relative paths to whichever file you're compiling, not to the file that included it. That is why I do `../nxc-modules/file_to_include.nxc` internally.

## `Button+-.nxc`

Set a number using the Left/Right buttons or the Up/Down buttons on the NXT.

### exposes

- `int senseButton(int val, bool lr_ud, int add, int high, bool cut)`
	- `val` is the starting value.
	- `lr_ud` is what set of buttons this watches.
		- `true` for the left and right buttons.
		- `false` for the up and down buttons. *Requires enchanced NBC/NXC firmware.*
	- `add` is the value that is added to or subtracted from val when a button is pressed.
	- `high` is the maximum value that is allowed.
	- `cut` is whether when the maximum value is reached the value should be cut (`true`) or wrap (`false`).
	- Returns an modified `val`.

### example
```c
#include "../nxc-modules/Button+-.nxc"

int age = 10;

while (ButtonPressed(BTNCENTER) == false) {
	age = senseButton(age, false, 1, 100, true)
	NumOut(10, 10, age, 0);
}
```



## `Menu.nxc`

*Requires enchanced NBC/NXC firmware.*

Create a sweet menu to select stuff from. Great for game difficulty levels, options, and whatever!

### exposes

- `int menu(int numChoices, string menuDisp[], int titleX)`
	- `numChoices` is how many choices you have to select.
	- `menuDisp[]` are the strings to display on each menu line. Must be at least 8 elements long.
	- `titleX` is the x posistion at which to display `menuDisp[0]`

### to do

I know that this function works, but needs work to become (more) awesome:

- Requires enhanced firmware.
	- It could use the left/right buttons.
	- It would need another function or parameter to specify different buttons.
- Only allows 6 selections.
	- The menu should be able to go off-screen.
- The first 2 strings in `menuDisp[]` are unselectable.
	- This should be configurable. What an opinionated module! :)
- All the strings you want to see are passed in.
	- You should only have to pass in the selections, number of selections, and which row to start on.
	- The programmer should handle the title etc.
	- This module shouldn't be trying to handle things it doesn't need to know about!

### examples
```c
#include "../nxc-modules/Menu.nxc"

string menuDisp[8] = {"","","","","","","",""};
menuDisp[0] = "MY AWESOME GAME";
menuDisp[1] = "-Level Select-";
menuDisp[2] = "Easy";
menuDisp[3] = "Medium";
menuDisp[4] = "Hard";
int difficulty = menu(3, menuDisp, 0);
```

```c
#include "../nxc-modules/Menu.nxc"

string menuDisp[8]={
	"GAME",
	"Speed:",
	"Slow",
	"Medium",
	"Fast",
	"", "", ""
};
int speed = menu(3, menuDisp, 38);
```



## `MorseInput.nxc`

Use morse code as a keyboard. Left button is a dot, right button is a dash, and the center button goes to the next character.

### exposes

- `decode()`
	- Used internally
	- Should be split into it's own module?
- `waitForButton()`
	- Used internally
	- Should be split into it's own module.
- `string morseCode()`
	- Starts the process.
	- Returns the resulting string on exit.

```c
#include "../nxc-modules/MorseInput.nxc"

string morseCodeString;
TextOut(0, LCD_LINE1, "Type your name", 0);
TextOut(0, LCD_LINE2, "using morse code!", 0);
morseCodeString = morseCode();
TextOut(0, LCD_LINE3, morseCodeString, 0);
```



## `HighScore.nxc`

A collection of functions to read and write persistent high scores.

### exposes

- `int readHigh(string filename)`
	- Returns the highscore.
	- Runs on files created with `writeHigh()` or `writeHighName()`.
- `string readHighName(string filename)`
	- Returns the name of the player who made the highscore.
	- Runs on files created with `writeHighName()`.
- `void writeHigh(string filename, int newHighscore)`
	- Writes a highscore to a file.
- `void writeHighName(string filename, int newHighscore)`
	- Writes a highscore to a file.
	- Opens a "keyboard" for the player to input their name. (See [TextInput.nxc](#textinputnxc).)

### to do

- Create a function that, when given a score, checks if it's higher before writing.
- Create a function that returns both the name and the score. (In a string? IDK...)
- Create a function that is given a score, and displays a splash-screen.
- Save more than just the *highest* score.
- Save scores in a standard way. (E.g., JSON, .properties, yml)

### example

```c
#include "../nxc-modules/HighScore.nxc"

TextOut(0, LCD_LINE5, "GAME OVER!", 0);

if (score > readHigh("GAME_high.dat")) {
	TextOut(0, LCD_LINE1, "You made a high", 0);
	TextOut(0, LCD_LINE2, "score! :)", 0);
	Wait(1500);
	writeHighName("GAME_high.dat", score); //opens a keyboard
} else {
	TextOut(0, LCD_LINE1, "You did not make", 0);
	TextOut(0, LCD_LINE2, "a high score. :(", 0);
	Wait(1500);
}
ClearScreen();
TextOut(0, LCD_LINE3, "Your score:", 0);
NumOut(0, LCD_LINE4, score, 0)
TextOut(0, LCD_LINE5, "High score:", 0);
NumOut(0, LCD_LINE6, readHigh("GAME_high.dat"), 0)
TextOut(0, LCD_LINE7, "Made by:", 0);
NumOut(0, LCD_LINE8, readHighName("GAME_high.dat"), 0)
Wait(5000);
```


<!--
moved this to nxc-tetris
	
## `RotatedNumbers.nxc`

For when you want a game in portrait mode, and want to display numbers, e.g. score.

For the numbers to be up-side-right, you must hold your NXT so that the top is pointing left.

### exposes

- `void RotatedNumbersOut(int x, int y, long num)`
	- `x` is the x position at which the numbers starts printing.
	- `y` is the y position at which the numbers starts printing.
	- `num` is number that gets printed.
- `void RotatedStringOfNumbersOut(int x, int y, string str)`
	- `x` is the x position at which the numbers starts printing.
	- `y` is the y position at which the numbers starts printing.
	- `str` is the string of numbers that gets printed.

### example

```c
#include "../nxc-modules/RotatedNumbers.nxc"

long score = 28173;
RotatedNumbersOut(94, 60, score);
```
-->


## `TextReader.nxc`

*Requires enchanced NBC/NXC firmware.*

Display a text file on-screen. Could be used for printing instructions for a game.

- `void readText(string filename, int lnjmp, bool eight)`
	- `filename` is the name of the file. E.g. `"cool.txt"`
	- `lnjmp` is how many lines jump when the down button is pressed.
	- `eight` is whether eight (`true`) or seven (`false`) lines are used to display the file. The bottom line is the toggled line.

### to do

- dumb line controls
	- have parameters that specify the starting line, and ending line.
	- get rid of `eight`
- Requires enhanced firmware.
	- It could use the left/right buttons.
	- It would need another function or parameter to specify different buttons.

### example

```c
#include "../nxc-modules/TextReader.nxc"

readText("TextFileRead.txt", 7, true);
```



## `TextInput.nxc`

*Requires enchanced NBC/NXC firmware.*

Throw a keyboard on the screen, so the user can input text!

### exposes

- `display()`
	- Used internally
- `output()`
	- Used internally
- `debugOut()`
	- Used internally
- `senseButtonModded()`
	- Used internally
- `string keyboard()`
	- Returns the string that you type

### example

```c
#include "../nxc-modules/TextInput.nxc"

string name;
TextOut(0, LCD_LINE1, "Type your name:", 0);
name = keyboard();
ClearScreen();
TextOut(0, LCD_LINE5, StrCat("Hello ", name, "!"), 0);
```



## `SoundEnd.nxc`

A small compilation of sounds to play at the end of a game.

### exposes

- `void SoundEnd(int snd)`
	- `snd` is the sound to play:
		- `soundWin` is a constant that can be passed into `SoundEnd()`.
		- `soundDraw` is a constant that can be passed into `SoundEnd()`.
		- `soundLose` is a constant that can be passed into `SoundEnd()`.
- `PlayToneWait()`
	- Used internally
	- Should be split into it's own module.

### to do

- more sounds!
- add `#download ...` directives to automatically include the sounds.

### example

```c
TextOut(0, LCD_LINE1, "GAME OVER!!!", 0);
SoundEnd( score > 50 ? soundWin : soundLose )
```



## License

[VOL](http://veryopenlicense.com)
