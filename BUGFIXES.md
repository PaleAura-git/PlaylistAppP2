# PlaylistApp bug fixes

A couple of bugs turned up in the PlaylistApp example, so here's what they
were, what caused them, and how they were fixed.

## 1. You can't delete tracks with an ID over 100

This is in `deleteTrack()` in `PlaylistController.java`.

The playlist is meant to hold no more than 100 songs, and the delete
prompt only accepts IDs from 1 to 100:

```java
int trackID = inputHelper.readInt("Enter Track ID to Delete", 100, 1);
```

That works at first, but the 100 limit is about how many songs are in the
playlist, while this is checking the track's ID, and those aren't the
same thing.

Every track gets its ID from a static counter in `Track`, so IDs just keep
going up (1, 2, 3...) and deleting a track doesn't free up its ID. That
part is fine, you don't want two tracks ever having the same ID. But it
means if you keep adding and deleting tracks, the IDs go past 100 even
though the playlist itself never has more than a few songs in it. Then
those newer tracks can't be deleted, you just keep getting
"Number out of range".

So I changed the max to `Integer.MAX_VALUE` (the biggest number an int can
hold), so any track's ID can be entered:

```java
int trackID = inputHelper.readInt("Enter Track ID to Delete", Integer.MAX_VALUE, 1);
```

The thing to take away from this is to make sure you're checking the right
value. "How many songs are in the playlist" and "what's this song's ID"
look similar at first, but they stop matching as soon as you start
deleting things.

## 2. The program crashes if you enter nothing at the menu

This one's in `readCharacter(String prompt, String validCharacters)` in
`InputHelper.java`.

When it asks "Enter choice:", if you just hit Enter without typing
anything, the whole program crashes with a
`StringIndexOutOfBoundsException` and your playlist is gone.

The original line was:

```java
inputText = reader.nextLine().toUpperCase().charAt(0);
```

If you type nothing, `nextLine()` gives back an empty string `""`, and
then `charAt(0)` tries to get the first character of a string that doesn't
have one. That's what crashes it.

The method was already checking if the character was one of the valid
options, it just never checked if there was a character at all. So I
added a check for that and made it ask again, same as it does for invalid
characters:

```java
public char readCharacter(String prompt, String validCharacters) {
    char inputText = ' ';
    boolean inputError;
    do {
        inputError = false;
        System.out.println(prompt + ": ");
        String line = reader.nextLine().toUpperCase();
        if (line.isEmpty()) {
            inputError = true;
            System.out.println("Nothing entered: please re-enter: ");
        } else {
            inputText = line.charAt(0);
            if (validCharacters.indexOf(inputText) == -1) {
                inputError = true;
                System.out.println("Character out of range: please re-enter: ");
            }
        }
    } while (inputError);
    return inputText;
}
```

`inputText` starts as `' '` now because otherwise Java complains that it
"might not have been initialized" (it can't tell for sure that the else
part ran before the return).
