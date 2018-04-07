footer: @endocrimes [she/they] - iOSCon 2018


## Sustainable Releases
### A tale of teams and automation

### **@endocrimes**

^
As software developers, one of our primary objectives is to ship software to people who want to use it - yet it is one of the most misunderstood and under developed parts of being a mobile developer. In this talk, I want to start talking about how we can understand these processes, and how to improve them.

---

## $ whoami

^
For some background on me, hey, I'm Dani. I'm a software developer, and I work on a few open source projects that you may have used.

---

## CocoaPods

^
I maintain CocoaPods, a dependency manager for Mac and iOS development - which has now been used in over 3 million applications.

---

## 🚀 _fastlane_ 🚀
^
I'm also on the core team of Fastlane, an an open source platform aimed at simplifying Android and iOS deployment.

---

## CircleCI

^
And I currently work as a Staff Software Engineer at CircleCI where I work on build infrastructure and developer tools.

---

# 📱
^
Prior to that, I spent a long time working as an iOS developer in many different organisations, and would frequently run into issues caused by lack of tooling, investment in developers, and general lack of process.

---

# 🤖🤖🤖

^
I also really like building robots to automate various tasks that are commonly forgotten by myself or by people on teams I work with, such as automatically deleting branches when pull requests are merged, synchronising different systems, linters for common issues, and general developer experience improvements. These bots give me the ability to extend what I do, without having to actually do more work.

---

## This is not a talk about how __CD__ will solve all of your problems

^
So now that introductions are out of the way, I’m pretty sure that some of you are now worrying that I’m about to suggest that continuous delivery is going to solve all your problems. Don’t worry, it won't, and I’m not. 

---

# 🙅🦄

^
There are no magical unicorn solutions to building and releasing software.
I am going to talk about some of the common issues that I see and have seen when working on or with mobile teams, that pertain to how quickly you can release your application, and how to enable your team to be more effective.

---

# 👩‍🔧👨‍🔧

^
I'm also going to provide some tools and strategies for fixing some of these common tooling and process issues, and show you that many things that are seen as only being useful by larger organisations can be applied to everyone. 

---

## Backstory

^
When I was still an iOS contractor, I would frequently join teams and have issues with building projects, lack of CI, or difficult release processes.

^
On a specific project a couple of years ago, on the first day, I checked out the codebase, followed the README instructions to setup dependencies, opened Xcode and tried to build the project.
How many of you can guess what happened?

---

## It didn’t build.

^
Xcode spit out 40 errors and actually stopped trying to list warnings. There were too many for the inspector to show.
It turned out, that someone had missed checking in their Xcode project changes to avoid committing the codesigning settings used for shipping a nightly build to QA. 

---

# 🚂🏡

^
The fun thing about these nightly builds was that they were shipped 30 minutes before the tech lead left the office for his train home.

---

# 😔

^
This very quickly led to some frustrating situations when they were sick, went on vacation, or simply forgot. Because nobody else in the organisation had the certificates required for QA or Prod builds of the app, and their work would be blocked.
As you can imagine, this led to the QA lead coming to our desks at 4PM every day, trying to find out when they could get a build with the days changes, often to be sent away, with an unclear, or unrealistic expectation.

---

# 📆

^
Whenever we needed to cut an App Store release, development basically halted for a week while we compiled screenshots in several different languages, for multiple device sizes, while we went around the organisation getting various pieces of updated documentation and prepared for relevant user messaging.

---

## Sound Familiar?

^
I'm sure that some of you have been, or are in varyingly similar situations, where a smaller group of people are often tasked with performing repetitive work as part of enabling the organisation to deliver. A lot of this repetitive work can be problematic, as people often forget to do it, or don't prioritise it over other tasks, and has a wider impact than just your team.

---

# 🕵️‍♀️

^
The most important part of trying to define a process is to first understand the actual underlying problems that you need to solve.

---

![](https://media0.giphy.com/media/pIMlKqgdZgvo4/giphy.gif)

## 🤖 all the things?

^
Some people might suggesting automating all of the things, and while I agree in principle, it is rarely a good investment to start automating things before you know what you are automating and why.

---

## No amount of 🤖 will save you from a __dysfunctional__ environment.

^
No amount of automation will save you from dysfunction unless you have a clear understanding of the goals you're trying to achieve. If you build tooling for something that doesn't deliver on your goals, you'll just be doing the wrong thing very quickly.

---

## It's about
## **delivering value**

^
It's important to remember that building software is about delivering value to your customers. As software developers, we deliver value by building software systems that fill some need in people's lives.
Software that hasn't been released is software that isn't delivering value to anybody, and the less often you ship, the fewer opportunities you have to deliver that value.

---

# ♼

^
You also need to do this in a sustainable way, which the dictionary described as something that can be maintained at a certain rate or level. Which to me, is something that you pace and design in a way such that it can be kept indefinitely, without causing undue stress or work.

^
There are definitely times when you will do unsustainable things, but they should be short term, and have some priority attached to fixing them.

---

## How often do you ship?

^
Raise your hand if you work on a public iOS app. Keep your hand up if you release your app more than once per year [pause] more than twice per year [pause] once per quarter [pause] once per month [pause] twice per month [pause] once per week [pause] once per day

---

## Release more often

^
Whichever one of those you are currently at, it should be a goal to increase the rate in which you release to your customers, this is for many reasons.

---

## Every release is an opportunity to make users happy.

^
The first is that every time you release your app, you get an opportunity to deliver value to your users, to fix that annoying bug. To add that feature that makes their lives easier.

---
[.build-lists: true]

## TypicalCorp: Milestone based releases

- Y Features need to be in by X date
- On X date we send to QA
- After QA we fix bugs
- Then we ship

^
[click] Many companies release in milestones, where you start out by deciding what features and changes are going into a release, and set a target release date. [click] You then send the build to QA on the date [click] and when QA find a bunch of places where things are broken or don't match the design, then you spend some time fixing bugs, pushing past your original deadline [click] and finally you ship and start again.

---

![fit inline](https://wac-cdn.atlassian.com/dam/jcr:e017085a-fef3-4c15-b7b0-4f582d6e721d/manualdeliverycycle.png?cdnVersion=ka)

#### Atlassian, the business case for continuous delivery blog

^
And while many mobile teams have a process that looks like this, where there is some deadline set by the business and product teams, that needs to include some set of features, before sending to any testers that you have, and then shipping. The process just doesn't work for many teams. There usually results in some cautious optimism as you start a mile stone. 

^
This optimism then begins to drain as you approach your development cut off. As you approach the cut off, you approach the anti pattern of crunch time. During this crunch time people ship more bugs and get less sleep. Then the expected release date appears, and you miss it. The Crunch period continues, before you ship and send your app to apple.

---

![](https://media3.giphy.com/media/3Q2hJ4FLN1UvS/giphy.gif)

^
Nobody wants to be doing this. It is bad for your health, your team, and for your product and users. Quote unquote waterfall development practices have long lead times on software being written to given to users. They also have a large cognitive overhead for everyone working on the project and frequently lead to people getting burnt out and leaving.


---

## So what can we improve?

^
I think many would agree that a cycle of imminent deadlines is painful to work with, so how do we start shipping more often, and not simply cram that work into a shorter period?

---

## You can ship smaller changes

^
Having frequent opportunities for release also means that your change set in any release is relatively small, so you are less likely to ship significant breaking changes, either in terms of bugs, or making major changes to the way the software works that is likely to scare users.

---

## Manage Expectations

^
When you move to a regular cadence, it's important to manage the expectations and internal communications. You should make it clear that a change is staged for release, and also communicate when it has gone out.

---

![fit](/Users/danielle/Desktop/Screen Shot 2018-03-21 at 18.49.34.png)

^
On fastlane we have a bot that comments on PRs when they have landed and when they have shipped. You can do something similar with your ticketing tools of choice, or have some other regular way to share information and be transparent with the organisation.

---

## You don't need deadlines.

^
When you have well set goals, and build trust with your ability to deliver, you can also remove the incentive for deadlines.

^
Instead you can focus on working on what is important to meet the organisations current goals, and focus on getting things right. If a certain feature misses the release train, it doesn't matter because it can land in the next one. This is significantly less scary when the next train arrives in the next week or two, than in 6 months time.

^
If you want features to roll out with a marketing announcement or for an event or similar, then there are ways that you can do that, that don't need to align with a deploy of your application. Tools like feature toggles are incredibly useful for this, and we'll come back to them later.

---

## ⚗️ Experiment!

^
Releasing your application more often also gives you more opportunities to try and validate different ideas, and experiment with different paradigms, and the ability to make smaller design changes and refinements. This applies to both experimenting with the way you work, and experimenting with changes to your product.

---

# 🚨

^
It is also important to define how your team should handle emergencies, for example, if you ship a bug that prevents users from using a critical part of your app, how do you escalate that issue? How do you get a release? How do you decide that a bug is worth escalating to a hot fix?

---


## 📉 Metrics

^
It's important to gather metrics as part of your software delivery process, both on how your team works, and on how your users are adopting the software. Gathering metrics on your teams cycle time, time to release, and escaped bugs can help you identify bottlenecks and issues with your process or infrastructure. For example, if you frequently have bugs in a certain part of the application, and have metrics that reflect it, you can use it as a way to prioritise spending time on fixing the underlying issues. Be it lack of testing, or an architectural decision that is difficult to maintain.

^
If you gather metrics on things like the application versions in use, you can also use it to inform decisions about api backwards compatibility, which helps other teams inside the company prioritise their work based on user need.

---

## So what now?

^
Now that we know there is some benefit to having a faster release cadence, and we know some of the things we need to care about as we build it out. How do we actually start to improve that?

---

# Release 🚂

^
The way that larger companies have moved to, and the way that I personally have found works for me is with release trains.

^
A release train is the concept of having a rolling release cycle whereby you cut a release every N timeframe, regardless of the changes that have made it in to the application.

---
[.build-lists: true]

## Some Requirements

- Automated build + test for branches/prs
- Automated internal builds
- Automated App Store builds


^
There are a few technical things that become necessary as you move towards a release train model, {click} The first is automated testing for your branches and pull-requests. This is because any change from your mainline branch might be the one that ends up being the next release commit. Having some level of testing for changes going in, means you're less likely to have big issues later.

^
{click} Another valuable step is to automate regular (daily) builds of your app to some internal test environment like Crashlytics Beta. This gets you comfortable with automated signing and release of your app in a way that doesn't directly impact users.

^
{click} And then finally automating your actual App Store release process, so that you can quickly release when needed.

---
[.build-lists: true]

## But it's not just automation

- Policy for escalating hot fix builds
- Set release cadence for the App Store
- Prioritised backlog of work

^
Although automating things is valuable, teams start seeing way more value out of moving to an actual continuous release process. Being successful at this requires {click} having a way to release a hot fix to the App Store quickly and easily, everyone sometimes ships bugs that need to be fixed now, that can't necessarily wait until the next train leaves.

^
{click} You also actually need to set your release cadence, and stick to that plan. This can be tricky to start with, especially when there are still many manual parts to the release, but you'll only really see the pain points when doing it so frequently.

^
{click} You also need to ensure you have a well prioritised backlog, that is enough work for a few releases, to avoid slipping into mini milestone mode, and to guide some of your technical decisions. It's also important to remember to include time for technical debt cleanup here.

---

## RandomCorp: Release Trains

- Sends a nightly internal+qa build
- Releases every other Tuesday w/ CI
- Uses a release branch to make pushing hotfixes easier

---

# 👩‍💻😊👨‍💻

^
Bridge into talking about some of the common issues people will face.

---

## Testing

^
A common issue is testing that your application actually works sucks. Even in 2018 many iOS developers still don't really write much in the way of unit tests for their code. But there is much more to ensuring that your app works than having some percentage of code coverage in your unit tests.
It's often important to have some kind of higher level of testing for your application. A common form of this is 'acceptance testing'. 

---

## Acceptance Testing

^
The purpose of acceptance testing is to evaluate the system’s compliance with the business requirements and assess whether it is acceptable for delivery.

^
I have seen this done informally by the releasing developer manually clicking through an application as they prepare to release it, by a manual QA team going through a set of testing criteria, or by emailing a build of your app and a large word document to an external tester. Not only is the feedback cycle on this very slow, prone to human error, it is also incredibly expensive when those people could be spending time on trying to improve the overall quality of the software.

---

## XCUITest

^
I'm not going to go too deeply into it in this talk, but a good solution for this is Apple's Xcode UI Testing framework, it's not the fastest, but it gives you a lot of help in the beginning by letting you click around your application and it will record the actions that you took.

---

# 📝

^
Writing release notes is something that has never been easy, even in the early days when everyone would do large milestone releases, there was a lot of balance between highlighting new features and communicating smaller fixes. With more frequent releases, there are often fewer changes at a given time, and providing a general "bug fixes" update can be pretty sad. Release notes are also decreasingly read by users now that many have automatic updates or mash the update all button - especially as bigger companies usually have a default "We bring you updates every n weeks" response.

---

## Changelog!

^
Maintaining a changelog that can be used to auto generate release notes can be very useful for this, especially in smaller teams where a lot of the work is more freeform. It depends however on the kind of release notes you want to offer.

---

# 📱📸

^
App Store submissions also require a lot of screenshots these days, there are multiple different device screen sizes, and also many different supported languages.  For example, if you have 5 App Store screenshots, only support the iPhone, and 4 languages, you still need to take between 80 and 100 screenshots. You then usually have to clean the status bar (because your phone is probably on energy saving mode by the time you get to the last language), and make sure that they are all showing the right data.  This can take hours to get right.

---

![fit 25%](https://raw.githubusercontent.com/fastlane/fastlane/master/snapshot/assets/snapshot.png)

^
Fastlane provides a tool for automating a lot of this pain called Snapshot, using the Xcode UI test framework, it allows you to automate going to various parts of your application and automatically take the screenshots.

---

```ruby
scheme("SnapshotGenerator")

devices([
  "iPhone 6",
  "iPhone 6 Plus",
  "iPhone 5",
  "iPhone 4s"
])

languages([
  "en-US",
  "de-DE",
  "es-ES",
  ["pt", "pt_BR"] # Portuguese with Brazilian locale
])

launch_arguments(["-username Felix"])

output_directory('./screenshots')

clear_previous_screenshots(true)
```

^
Snapshot lets you configure the devices and languages you want to test with in the Ruby DSL, then will run your scheme to generate the variants. It will also render a web page that you can use to go and inspect the screenshots across different devices and languages, so can also be useful for general QA.

---

## Localisation

^
One common issue faced by people working on all kinds of client side software, but mostly mobile developers is having localisations ready for use in your application. iOS apps are some of the most translated pieces of software that are in use. The iPhone's international popularity forces organisations of all sizes to care about internationalisation to access the market.

---

# 🗺

^
Many organisations defer getting translations until a feature is almost ready to be released, because of small last minute tweaks that happen during the development of bigger features. This cycle can make it difficult to merge features early, and can cause a lot of extra cleanup time when preparing for an app release. You can solve this with feature toggles.

---

## 🎛 Feature Toggles

^
It can also be incredibly valuable to have some basic ability to have feature flags and toggles in your application. Feature toggles allow you to separate deployment of your application from the release of a feature. This is super valuable for staged marketing releases, or being able to integrate changes without necessarily wanting them to be used yet, for example when rolling out changes to a beta group.  

---

### Toggles are intentional Technical Debt

^
Let you roll out features ready for a marketing release
But they are also technical debt
Delete them as quickly as possible

---

## Review Times

^
There is also the historically long and often inconsistent times spent waiting for App Store processing and review. Now down to two hours at best.

---

## Tooling

^
Historically was very difficult to automate much of this tooling. Made much easier by things like fastlane. Lots of community driven support.

---

## Real World Example*

#### __*by which i mean a totally contrived example.__

---

![fit](/Users/danielle/Desktop/Screen Shot 2018-03-19 at 09.08.27.png)

^
https://twitter.com/endocrimes/status/973318165794119680

^
I recently tweeted, asking for a volunteer iOS app to use as the subject for this talk, and my friend Cate bravely volunteered the Wordpress app, so here goes.

---

# 🙏

---

# Thank **you**
### @endocrimes

![](https://media1.giphy.com/media/tczJoRU7XwBS8/giphy.gif)
