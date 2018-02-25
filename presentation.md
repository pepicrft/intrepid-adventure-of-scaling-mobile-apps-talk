footer: @pepibumur
slidenumbers: true

# The intrepid adventure of scaling a mobile app
[.footer: ]

### **@pepibumur**

![](images/berlin.jpg)

---

# About me

- Production Engineer at Shopify.
- Building tools for mobile developers.
- Born and raised in the ❤️ Murcia.
- Suffering the ☔️ in Berlin.
- 📄 _ppinera.es_
- 🐦 _@pepibumur_
- 📧 _pedro.pinera@shopify.com_

![right](images/team.jpg)

[.footer: ]

---

# Scaling a mobile codebase

> "Scalability is the capability of a system, network, or process to handle a growing amount of work, or its potential to be enlarged to accommodate that growth" - Wikipedia

---

# The codebase doesn't scale when...

- 20 engineers produce the same as 10 👩‍👩‍👦‍👦
- You spend more time fighting the tools ⚙️
- You can prepare & drink a coffee while CI runs ☕️
- Random issues arise in a daily basis ⚠️
- There are conflicts very often 😒


---

# Why?
# __Apple__ and __Google__ don't optimise the tools for large codebases

---

# If you are not experimenting scalability issues, they will come up __sooner or later__

### _(don't build for scale, think for scale)_

---

# Changing environment

- Larger products
- More APIs
- More platforms to support
- New programming languages
- People joining/leaving the team


---

# Scaling mobile codebases is all about **challenging standards and tools** that we are given

---

# 🎸 SoundCloud
- Compilation times.
- Inconsistencies in settings.

# 🛒 Shopify
- Multiple mobile apps.
- Historically no shared code.
- Compilation times.

---

# Scalability areas

## **1.** Code
## **2.** UI
## **3.** Tools/Projects
## **4.** Teams

---

# 1. Code

![](images/code.jpg)

---

# Key points

1. Make features independent *(from UI to Data)*
 - Reduce friction
2. Reduce dependencies between features *(atomicity)*
 - Allow autonomy
3. Share as much code as possible
 - Save time

---

# Storing data
- *Collections* ➡️ SQLite / Core Data / Realm
- *Secure* ➡️ Keychain / AccountManager
- *Other* ➡️ UserDefaults / SharedPreferences

---

# Stores contain shared state leads to **unpredictable state** changes

---

# Example

- *Feature A:* Deletes a token from the Keychain
- *Feature B:* Expects the token to be there to work.
- *Result:* Feature B behaving unexpectedly

---

# Make your stores observable

And let other features know about the state changes

```swift

SecureStore.instance.observe(.token) { [weak self] token in
  if token == nil {
    self?.deleteData()
  }
}
```

---

# If you **serialize/save** content into disk. Control how the disk space is used 

---

# Example

- *Feature A:* Saves `/Documents/orders.json`.
- *Feature B:* Cleans the folder `/Documents`.
- *Feature A:* Tries to access `/Documents/orders.json`.
- *Result:* Unexpected behaviour from feature A

---

# Better approach

```swift
// A store to save orders related data
let store = StoreManager.instance.store(for: .orders)

// We interact with our own disk space
try! store.save(json, name: "orders.json")
try! store.cleanAll()
store.observe("orders.json") { orders in
  // Orders changed
}
```

---

# Relational databases

- Observing data.
- Accessing relationships.
- Performance.
- ORMs for free.

---

# SQL **creates implicit dependencies** at the data layer

### Unpredictable behaviours (bugs)
### Dependencies between teams

---

# Relationships

```swift
class Product: NSManagedObject {}
class Order: NSManagedObject {
  @dynamic var products: [Product]
}
class Client: NSManagedObject {
  @dynamic var orders: [Order]
}
```

- Can we safely delete an order?
- Do I have to update the order delivery address if the client address changes?
- Do I have to update an order if a product gets removed?

---

# Remove any kind of **implicitness**

---

# Explicit APIs

- If you use SQL, *don't use relationships*
- Aim for APIs that other features can consume.

```swift
let order = ordersRepository.new(clientId: clientId)
let token = ordersRepository.observe(clientId: clientId) { orders in
 // Client orders changed
}

```

---

# Maybe you don't need a SQL store 🤔. **Just serialize objects into disk**

---

# Leverage **modules** and **access levels** to decouple your code defining **public interfaces**

---

```swift
// Module Core
public class Client {
  public fund execute(request: Request) -> Response
  private fund privateMethod() { /* some code */ }
}
```

```swift
// Module App
class ProfileInteractor {

  let client: Client
  
  func sync() {
    client.privateMethod() // Compiler error! ⚠️
    let response = client.execute(request: User.me) // All right ✅
  }
}

```


---

# As part of your implementations. 
# **You should design 👩‍🎨 APIs**

---

# Default access levels

- Kotlin: *public* 👎
- Swift: *internal* 👍

---

# Other programming paradigms might have benefits, but its usage in large teams is **questionable**

---

# Use case: Reactive programming

- Everyone in your team needs to learn it.
- They need to learn it *properly*.
- You'll most likely rely on a *third party dependency* (RxSwift, RxJava...)

---

# Abstractions only if they are really necessary
# **Prefer patterns from the system frameworks 📦**

---

# Because...

- 👩‍💻👨‍💻 know about the system patterns.
- 🏝 You get updates/improvements from the platform.
- 😓 Building abstraction that scale is hard in practice.
- 😘 The fewer dependencies, the better.

---

# Minimize dependencies
- Dependencies save time initially, but need to *be maintained*.
- Do you need dependency X:
  - *Yes*: Use it but with an abstraction layer.
  - *No*: Then don't use it or build just what you need.

---

# 2. UI

![](images/ui.jpg)

---

# **Composition** over extension

---

1. I start defining my layout
2. I add this button here.
3. I add this list of images there.
4. I introduce this new button between these two elements
5. ...
6. ...
7. ...
8.  **No one can modify the XML/XIB/storyboard anymore**

---

# Use your **intuition**

### **if** the layout is growing considerably
### **then** split it in small layouts

---

#

# Composition means
### **Reusability**
### **Easy maintenance**

---

# 3. Tools/Projects
![](images/tools.jpg)

---

# **Automate** the automatable

### Code linting
### Project sanity checks (danger.systems)
### App releases

---

# Don't spend your developers' time
### There are great tools already built to help you with automation *(e.g. Fastlane)*

---

# Build times
### **The more code you have, the more it takes to compile**
### *(unless you use React Native)*

---

# Gradle & Xcode Build system build incrementally
### **Until you clean the build 😅**
### And I'm sure it's the first thing you do when your project doesn't compile.

---

# Let's do some maths

- Clean build: 10 minutes
- Cleans per day per developer: 10
- Developers in the team: 30

*Build time spent per month*
- 3000 hours
- 3.75 developers

---

# Make your app modular

- Cleaning should only affect the current module.
- Developers should only compile the module they are working on *(and its dependencies)*.
- Reference: [Microfeatures](https://github.com/pepibumur/microfeatures-guidelines)

---

# You can go further

### And try other build systems that build incrementally across developers in a team

## **Buck** from Facebook
## **Bazel** from Google

---

### If clean builds are slow...
# **So they are on CI**
### Continuous integration services/platforms
# Build from a **clean environment**

---

# Can we speed up clean builds on CI?
- Selective build/run tests based on the files that were modified.
- Parallelize work.
- Cache build artefacts across builds *(e.g. Pods, Bundler dependencies)*
- Use Buck/Bazel.

---

# Measure 📏
### **Before optimising, we need to have some values to compare with**

---

# What to measure? 📏
- Compilation time of each of the modules.
- App startup time.
- Flaky tests consistently failing.
- Failing builds and the reason.

---

# Build a dashboard 📈
### With all the project metrics
### [grafana.com](https://grafana.com/)

---

### **Reproducible builds**
# "My build is failing on CI but it doesn't fail locally" 😡

---

# How to make your builds reproducible

- Avoid custom logic that only runs on CI.
- Pin your dependencies versions.
- Bundle your dependencies into the repository and avoid access the shared system environment.
- Use [Nix](https://nixos.org/nix/about.html)

---

# 4. Teams

![](images/teams.jpg)

---

# If a team grows,
# it gets **splitted** into smaller teams

---

# Feature teams / Squads

👍 They are independent.
👎 They might duplicate code.
👎 Patterns/style might diverge a lot.

---

# Be consistent

- Build settings.
- Tools.
- Processes.
- Patterns.
- Programming languages.

---

# **Mob**ops team 👩‍💻👨‍💻

- Has an overview of the project as a whole.
- Identifies patterns across teams and extracts reusable components.
- Builds tools to make teams productive.
- Provides teams with communication tools/channels.
- Removes any friction between teams making them as atomic as possible.

---

# Collective meetings
### Where all the iOS/Android developers from different teams come together to talk about the vision of the project

![](images/meeting.jpg)

---

# Conclusions

![](images/conclusions.jpg)

---

- If you are not facing the challenges I talked about, you'll do sooner or later.
- Don't let yourself be influenced by hype technologies.
- Automate as much as you can.
- Don't expect Apple/Google to optimise the tools for scale.
- An awesome product is the result of motivated developers.

---

# Thanks
### **Questions?**
### Slides: https://goo.gl/2dbsej

![](images/thanks.jpg)