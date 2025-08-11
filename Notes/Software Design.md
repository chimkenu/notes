Most important idea: **Problem decomposition**

Concepts:
- Working code isn't enough: must minimize complexity
- Complexity comes from dependencies and obscurity
- Strategic vs tactical programming
- Classes should be deep
- General-purpose classes are deeper
- New layer, new abstraction
- Comments should describe things that are not obvious from the code
- Define errors out of existence
- Pull complexity downwards

## Classes should be deep
![[deep_classes.png]]
Idea: greatest benefit, least cost

> abstraction is trying to find a simple way of thinking about something that is very complicated underneath

A deep class is a great example of abstraction
cost is the complexity added from having a larger interface -> using the class is harder because there are more methods and things to think about (on the user end most likely but probably on the implementation as well)
therefore, a small interface that abstracts a more complex problem is provides the most value

this can be applied to any interface - implementation

example of shallow method (bad):
```java
private void addNullValueForAttribute(String attribute) {
	data.put(attribute, null);
}
```
you need to already know the ins and outs of the implementation to understand what the method does. this simply adds complexity and get nothing back for it

> any change to the implementation will change the interface

we want to make the common case really simple! - sensible defaults?

example of deep interface (good):
```c
// UNIX FILE I/O
int open(const char* path, int flags, mode_t permissions);
int close(int fd);
ssize_t read(int fd, void *buffer, size_t count);
ssize_t write(int fd, const void *buffer, size_t count);
off_t lseek(int fd, off_t offset, int referencePosition);
```
Hidden below this interface:
- on-disk representation, disk block allocation
- directory management, path lookup
- permission management
- disk scheduling
- block caching
- device independence

five functions that does a lot of stuff under the hood

## Define Errors Out of Existence
- exceptions: a huge source of complexity
- common wisdom: detect and throw as many errors as possible

- overall goal: minimize the number of places where exceptions must be handled
- better approach: define semantics to eliminate exceptions

### example mistakes:

#### tcl unset command (throws exception if variable doesnt exist)
the `unset` command throws an error if you try to 'delete' a variable that doesnt exist
however, many use cases (such as clean up) use unset to simply clean up a mess, regardless of whether a variable was used up or not
this creates a case where the unset command is very commonly surrounded by a try catch block that ignores the exception (therefore making the thrown exception useless in the first place)

solution: change the way unset is thought of (redefine semantics)
instead of 'deleting' a variable, think that 'unset' makes a variable not exist
now instead of throwing an exception when it is given a non-existent variable,  it is just a simple case where no work needs to be done

#### windows: cant delete file if open
if a program has a file open, you have to kill that program. 
this is bad because if we dont know who has it open, we have to search who is using it (not always easy)

solution: change the implementation
in Unix, deleting a file removes it from the directory/namespace - it no longer exists.
however, the contents of the file still exist, so any program currently using the file can continue to access the file. then when the last opened instance of the file is closed, then the os cleans up and throws it all away.

caveat to solution: accidentally deleting a necessary file/directory. 
since we dont error when a program is using it, a file that is very important for the system to run can very easily be deleted with no repercussions

#### java substring range exceptions
if the ends of the substring are invalid (outside, reverse order, etc.), substring throws exceptions. 
this is annoying because you may end up making code to make sure the two ends of the string you provide are within the bounds of the string you are getting the substring for

solution: limit output to range of string
if the ends are outside of the range of the string, cap it at its ends. in reverse order, return an empty string

caveat to solution: since we no longer throw exceptions, silent errors may appear
it also may be the case that this behavior might not be what a user of the class wants when there is a mistake on the inputs

### so, no exceptions?
when do we actually want to throw exceptions?
when we can no longer complete the contract or transaction (the thing the function is supposed to do in the first place)

```java
String.substring(int start, int end)
String.getCharAt(int index)
```

it was mentioned that substring should not really throw exceptions for invalid indices, but should this be case for `getCharAt`?
in both cases, they 'cannot' carry out the contract of the interface when they are given indices out of bounds.
however, it was mentioned that `substring` shouldn't throw exceptions. should `getCharAt` also not throw any exceptions?

you need to think about the common use cases of functions.
in the case of `substring`, very often will we have to error check our indices before calling substring anyway to make sure we are getting the right substring
however for `getCharAt` we typically expect a single character to be spit out and always expect the right one.

it is easy to catch an error for substring (just look at the substring!). for getting a character at an index, we typically want to fail early and see if we are accessing out of bounds.

## Tactical vs. Strategic Programming
tactical programming
- get next feature/bug fix working ASAP
- a few shortcuts and [[kluge|kluges]] are OK
- results in bad design and high complexity
- extreme: tactical tornadoes
complexity is incremental -> snowballs to spaghetti

> working code isn't enough

strategic programming
- goal: produce a great design
- simplify future development
- minimize complexity
- must sweat the small stuff

investment mindest
- take extra time today
- pays back in the long run




















