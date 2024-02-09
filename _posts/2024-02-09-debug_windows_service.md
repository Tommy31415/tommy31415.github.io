---
layout: default
title: Debug windows service
date: 2024-02-09
categories: csharp
---

Helpful snippet to let you debug Windows Service

```c#
static void Main(string[] args)  
{  
    if (Environment.UserInteractive)  
    {  
        MyNewService service1 = new MyNewService();  
        service1.OnStart(args);  
    }  
    else  
    {  
        // Put the body of your old Main method here.  
    }  
}
```

Other solution would be to run in as a console application. The best approach is to give some argument via 'args' to detect if it should be run as a console application.