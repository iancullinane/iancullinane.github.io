## Docs as code

There are so many docs out there. There is a misconception that a software engineer is responsible for writing code. This is actually not true and only makes up a small portion of the job day-to-day (unless you are associate level). Much of the day is spent reading code, and after meetings and other overhead, you mostly have reading and writing docs. Often the priority of an engineer is exactly this order; get through everything at the bottom so I can code, and it often feels docs are exactly in the above order. Last.

Everyone talks about the best solution, and while I have some opinions, one thing I always try and remember is the following:

> I like writing code, how can I make my docs more like code?

While the wall-of-text remains occasionally inevitable, making my docs into code has invaluable. So what exactly do I mean?

Lets start with what a doc often looks like: 


![Figure 1](/assets/docs-as-code-1.png){:style="display:block; margin:auto;  width="80%"}

Lots of text with headers. Lots of blocks of tests. You will probably have to see who wrote it, find them, and clarify something. At best it will bore you to tears. 

The easiest way to write docs as code is to just use bullets for everything. 
* Bullets get to the point
 * They can be short
 * and are even better with links?
* Don't lie, **did you skip to the bullets?**
  * I think its because it looks more like a function but that is just me

That is so much easier to read. And just like our code, we can apply the same rules.

> Code rules as doc rules

* Bullet points are easier to read
* Bullets enforce more brevity in your writing
 * Most people tend to ramble when they write
* You can apply other rules
 * like
  * don't
   * use 
    * too many nested statements

## Break them out to a new function or package

Now lets take it further. Here is an example doc for listing a series of on-call actions:



This will display the three images side by side if the images are not too wide.

<p float="left">
  <img src="/assets/docs-as-code-2.png" width="60%" />
  <img src="/assets/docs-as-code-3.png" width="60%" /> 
</p>

* Pay attention only to structure
* The top is all links
  * These links are anything useful to the rest of the file
* Summary is a bulleted list
  * Easy to read, use of formatting to draw the eye
  * Heavy use of links to off load data to another place
    * Just like code, your black box doesn't need to be in main, give it its own package
  * Further down is a list of "functions"
    * These docs provide the engineer with structured information 

### Data structure of the "Functional" part of the above code

The most important of this particular document is the grey boxes at the bottom. I tried to write these as functions with there own signatures and outputs.

The pseudocode of a struct in this sense is the following (try and compare the structure visible above):

```
let event_mitigation = [
  message_from_alert_system: "Some obect representing Splunk or DataDog or something"
  image = An image of what this might look like, could be slack message, datadog graph, or spluk log
  description = "A bulleted list to what is happening"
  mitigation = A link (very important) on how to fix the problem
]
```

Every mitigation follows this structure, and so will new engineers who see its use. I called out the separate link for the mitigation itself as important. This is because once you start adding specifics to this file, it will become unmaintainable. Sound familiar? From here we can get any number of documents, but we don't need to worry about that here. We have on document that shows us how to solve the problem. 


