---
layout: post
title:  "Automation Use Cases "
date:   2018-03-08 11:36:40 -0600
categories: jekyll update
---

## Down the rabbit hole

__Alice__: Would you tell me, please, which way I ought to go from here?

__The Cheshire Cat__: That depends a good deal on where you want to get to.

__Alice__: I don't much care where.

__The Cheshire Cat__: Then it doesn't much matter which way you go.

__Alice__: ...So long as I get somewhere.

__The Cheshire Cat__: Oh, you're sure to do that, if only you walk long enough.”


## Alice in Wonderland?  I thought this was about Network Automation?


One of the most common conversations that I have with customers is regarding use cases.
Use cases are **the most important** part of network automation in my opinion.  
The ability to define *exactly* what the process you are trying to automate is ends up
being a huge predictor of success. It also turns out to be the hardest part in a lot of cases,
because the conversation usually starts something like this:


Me:  What would you like to automate?

Customer:  Everything!

Me:  Okay, what would you like to automate first?

Customer:  What do other customers usually start?

Me:  Mundane tasks they do often, and usually to a large number of devices..

Customer:  Like what?

Me:

![img](http://2.bp.blogspot.com/-ryv7vhCW8-w/U9AQikICJWI/AAAAAAAAUWM/jZJkP9RSLkw/s1600/Aw+Jeez,+not+this+shit+again!.jpg)


Followed by something resembling this gem...

![img2](https://i.imgflip.com/a5fsp.jpg)


My questions I like to ask to get the dialog going:

* What is something you really hate doing?
* What is something you do every day?  
* What did you have the last intern you brought in do?

Some common categories that start to emerge:

* Documentation
* IP Address Management
* Troubleshooting
* Monitoring


But inevitebly, it always comes back to what others have done, so here is a real world
scenario that recently fell into my lap, and thus the motivation behind this blog. I
found this to be a great use case, with the problem clearly identified, and provided a
good foundation to begin thinking about automation.

I received an email from some colleagues regarding one of their customers. The short version
goes something like this.  

* A Device rebooted
* Configuration had been changed but not saved.
* What should have been a short disruption turned into a long one.

These defined the use case:

#### We need the ability to detect whether a device has unsaved changes

So to automate the use case, we have a couple of questions that we need to think
through

1. How does one detect that the configuration has been changed by not saved?
2. What action do we want to take when this condition is detected?

As for #1, in this case the customer was a large NX-OS shop, and the `show running-config diff`
command was a great way of detecting whether or not the configuration had been saved, as well
as giving us information about `what` changes were unsaved, which could be extremely valuable
when determining #2

Now that we now how to identify the condition that we are looking for, the next step
is figuring out how we run that command against the entire fleet of devices.  Luckily,
there are plenty of packages out there such as [Netmiko](https://github.com/ktbyers/netmiko)
that make this super easy.  It also turns out that there are a ton of other use cases
which net out to "I'd like to run this(these) commands on 5000 devices" so hopefully we can
create a little bit of reusable code along the way.


Here's a simple example of how we could use Netmiko to solve this challenge?

{% highlight python %}

from netmiko import ConnectHandler

def run_commands(host, user, pw, command, device_type="cisco_nxos"):
    """
    Executes a commands on a device
    :param host: IP/hostname
    :param user: username to login to the switch
    :param pw:  password to login to the switch
    :param command: list of commands to execute on each device
    :param device_type: netmiko device type
    :return:
    """
    device = {"device_type": device_type,
              "ip": host.rstrip(),
              "username": user,
              "password": pw,
              }

    session = ConnectHandler(**device)
    output = session.send_command(command)
    return output

hosts = ['1.1.1.1', '2.2.2.2']

for h in hosts:
    unsaved_changes = run_commands(h, 'cisco', 'cisco', 'show running-config diff')
    if len(unsaved_changes) > 1:
        print("unsaved changes on {}".format(h))
        # potentially save automatically, or record delta's, etc?
{% endhighlight %}


With this in mind, we continue the discussion into what we want to next (#2 from above)
to which the possiblities are endless

* Save the config?
* Send an email?
* Rollback the unsaved changes?


The possibilities are limitless.  But the thing to take away from is that we've started
automating `something` and are beginnging to think about workflows and their ability to
be automated.  
