---
title: "MITRE - Software Internship 2017"
date: 2017-09-23T19:23:43+02:00
draft: false
tags: 
- work
- 2017
---

In the summer of 2017 I worked for MITRE, a government research and development summer. I worked on a project called SIMULARITY which worked on creating more realistic cybersecurity training sceneries. The basic idea is this: in the real world, the majority of the people using a computer network are *not* interested in hacking or defending a system. Thus, cybersecurity test scenarios should reflect this. But, in most cases, there are just two teams, a red team attacking a system and blue team defending a system. I helped this effort by exposing the **Windows Automation API** to a server so that it could be controlled by an AI bot. This would then allow the bot to control the human in a way similar to a human. 

To solve this, the SIMULARITY project is attempting to make a 'gray team'. This team will be composed of AI controlled bots that will simulate a normal user on a computer. The goal is to create *noise* in the system. This way the blue team will have to differentiate between arbitrary users and actual red team attackers, thus making a more realistic environment. 

## Technical Fun

Random set of interesting technical tools / problems I ran into during my time at MITRE. 

### Windows Automation API
I was involved in the low level system details required to make these AI bots. The brain of these bots (simulated humans) would control a set of virtual machines, pretending to be a human like user. So, if the bot wanted to move the cursor on ```Machine 1``` to the top of left corner, the bot would send a message such as 
``` json
{
    "function" : "move mouse", 
    "params" : {
        "x": 0, 
        "y": 0
    }
}
```

My job was to translate these message into actual actions, and provide event information back to the AI such as if a window opened as a result of an action the bot made. All of this was done using the **Windows Automation API** (All of this work was done on windows virtual machines). 

Here is a more concrete example, if a bot had the objective of going to *google.com*, it could do the following: 

1. Find the Chrome icon 

    This is actually more difficult than it would seem; the bot would need to *find* the chrome icon in the UI tree. All the elements on the screen are represented in the computer as a tree; each application or icon has a parent. So, if the chrome icon was on the taskbar, then the icon would have the first parent of the taskbar element, and the taskbar element would have the Desktop as a parent. So, to find the Chrome icon, the bot would need to look in all of the Desktop's children for the taskbar, and once it found the taskbar, the bot would need to look through all of the child elements to find the 'chrome' icon. 

2. Move the mouse to the chrome icon

3. Generate a click on the icon

4. Wait for the window to open

    This is an important step. When a window opens, an event is fired in the windows operating system indicating that a window has opened. Because the bot cannot *see* the screen the same way a computer can, it needs to be alerted that the window has, in fact, opened. 

5. Focus the search bar

    This can be done similarly to finding the chrome icon 

6. Type in google.com and hit enter

This is all done using the [Windows Automation API](https://msdn.microsoft.com/en-us/library/windows/desktop/ee684009.aspx)

### Memory leak in managed code

The windows automation API is interfaced using C/C++ and the Component Object Model (COM). Basically, do call a Windows Automation API, the user makes a struct and sends that struct through a C function to the windows kernel. This is slightly tricky, but ensures the automation library is as fast as possible. 

The application I wrote was written in a c# and c++/cli. c++/cli is a managed / mixed version of c++ which allows it to create the COM structs but still be called easily from c# because it is managed code. Basically, this allows use to choose when you want to use c# garbage collection or manual memory allocation in c++. Awesome! Almost..

The issue comes from the *interaction* between garbage collected and manually managed classes. When a garbage collected object has a reference to a standard c++ object, things are easy; the collected class just decrements a counter for the number of references to that object. If that object goes down to zero references, just delete that object. This is basically how an actual garbage collector works!

But, what if a standard manually managed object has a reference to a managed object? This means the managed object will *not* be collected until the standard object is deleted. And, if that managed object has a reference to the standard c++ object, then that standard object will *never* be deleted! 

So why doesn't this problem occur in normal c#? Well, the garbage collector is smarter than that; it [detects](https://stackoverflow.com/questions/8840567/garbage-collector-and-circular-reference) if there is a cycle. 

How can this be fixed? Weak Handles! Basically, the standard, non-managed c++ class cannot hold a real reference to the managed class. Rather, it must hold a [```WeakReference```](https://docs.microsoft.com/en-us/cpp/cppcx/weak-references-and-breaking-cycles-c-cx). Basically, the weak reference does not guarantee that the object it points to actually exists at any point in time; every time a weak reference is used, the user must check to make sure the object has not yet been garbage collected. This allows the c# garbage collector to collect this managed object even though this weak reference still exists, thus solving this cyclic issue. 

Here is some psuedo code to help illustrate this 

**class 1: Standard / Unmanaged**
``` cpp
class Class1 {
    WeakReference<Class2^> myWeakReference;  // ^ is the handle symbol in c++/cli
    ...
    SomeFunction () {
        if (myWeakReference is not garbage collected){
            do stuff
        }
        else {
            // probably delete myself
        }
    }
}
```

**Class 2: managed c++/cli class**
``` cpp 
class Class2 {
    Class1* unmanagedReference; 

    !Class2(){ // finalizer when called when garbage collected
        // decerement reference count for unmanagedReference object; 
    } 
}
```
This will *not* have a cycle. See [here](https://docs.microsoft.com/en-us/cpp/cppcx/weak-references-and-breaking-cycles-c-cx) for more information.

### Protobuf!

In Progress: awesome serialization and schema creation

