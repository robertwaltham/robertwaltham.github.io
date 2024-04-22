---
layout: post
title:  "90 Days of Rust"
date:   2023-09-16 08:59:06 -0700
categories: engineering
---
![Header](/assets/images/header.jpg)

I'm currently on sabbatical between jobs, and I wanted to learn Rust. Having heard a lot of people talk about how they liked working with the language and ecosystem despite the difficulties I was curious to get first hand experience with it. 

### Goals
- Break out of my mobile-first shell, and work with a language and toolchain that were not mobile-primary
- Practice coding challenges in a new language, in preparation for coding interviews
- Dive into some game dev, and/or continue with learning about graphics programming and shaders
- Have fun with it

### Leetcode

I started with writing some leetcode easy/medium problems in Rust. The whole programming challenge question thing was new to me, so there was a learning curve in just understanding what the problems were asking for and how to reason with them. 

#### Observations

- Rust project structure is different than what I was used to, and it took me a while to wrap my head around the module/crate system. I found the examples in the rust handbook tricky to understand.
- The rust standard library has a lot, especially with collections. Once I had wrapped my head around how to read rustdocs, they are actually really useful. 
- In a single threaded, uncomplicated program the borrow checker isn't difficult to work with. There are some gotchas that don't work as they do in other languages (the big one being passing a value to two different function calls in the same block) but the warnings and errors provided are generally very good at explaining the situation and recommending a solution or autofix. 
- The exception is working with data structures and linked lists. I abandoned my attempt to do any of the linked list problems in Rust at this stage because I didn't have the knowledge to navigate understanding ownership. 
- Often I found that the ownership rules in rust mean that it became really obvious and explicit in my code where value were being copied vs passing references to existing memory. For the purposes of coding challenges this is a boon to understanding the memory performance of your code. 

I went into this with some preconceived notions that I was going to have to fight Rust, and have a very steep learning curve with the language. In reality I found it actually quite easy to pick up, and very approachable. I appreciate the culture of "good docs, good error messages" in the rust ecosystem, and they went a long way to help me navigate some of the sharp edges in the toolchain.  

### Bevy

After some dabbling in seeing how many off-by-one errors and did-not-read-the-question errors I could make on leetcode, I next picked up trying to do some basic gamedev in Bevy. I was attracted to this engine because of it's data driven, coding first approach to game dev. This seemed in line with my goals, and having tinkered with editor-first game engines like unity in the past I knew I wanted something more code oriented. 

The Bevy ecosystem is very immature and new. The engine is far from it's 1.0 release and the engine developers adding new breaking changes every 3 months. Unlike a more mature engine like Unity, there is a lack of comprehensive documentation or examples on github (and the ones that are often don't build on the current release due to api changes). Despite this, I found a couple of youtubers who had some great content to push me in the right direction [^3].

#### Observations 

- Cargo is super easy to use for dependency management, and I had zero issues (outside of did-not-read-the-docs problems. see a theme here?) getting the project setup. 
- Compile speed for a full compile was decently slow on my intel mac (~7 mins), but incremental builds were very fast. I centralized my cargo folder for all my projects (primarily to keep icloud from trying to sync artifacts in my documents folder), and have had no issues with conflicts between projects. Upgrading to a M2 Pro mac, the full compile time is under 60 seconds, and this increase is (unscientifically) in line with the improvements I saw going i7 -> M2 in Swift projects. 
- "Rust doesn't have reflection" - true but also bevy-reflect does a very good job of implementing reflection, and outside of some clunkiness with type registration works smoothly (at least for the purposes of the Bevy engine, as I have not used it outside of Bevy)
- The ECS implementation of Bevy honestly felt like magic the first time I used it. The query system is entirely driven off of reflection and function signatures, and it felt like it shouldn't be possible to be so easy. 
- Same with multithreading. Systems (aka functions that get called every frame that implement game logic) are multithreaded by default; however, if you ask for a mutable reference to a resource or entity in different systems the engine handles this by running them sequentially (to avoid violating the 1 mutable reference rule). This is entirely transparent to the user, and requires no input or code changes to implement outside of changing the function signature to indicate you want a mutable reference.
- The ownership model of the Bevy ECS system is "everything is owned by the ECS, but you can borrow anything for up to the end of the frame". This meant that as a user of the engine, I had to think about lifetimes and global ownership rules zero times while implementing game rules (note, I haven't gotten to long running operations that last more than a frame yet)

Bevy is written (by design) to be incredibly generic, and leverages a lot of what Rust allows you to do with the type system to accomplish this. It took me a while to understand how to navigate and read complex types [^4]. Trying to read and understand how the engine works gave me an appreciation for how powerful Rust is and what it gives you the flexibility to do despite being famous for it's restrictions. 

One thing I did notice is how many `unsafe{}` blocks there were under the hood in Bevy. Talking to a friend of mine who has worked on some Rust projects professionally, he said that this is pretty standard for complex projects: start with an idea about how ownership is going to work and add `unsafe{}` blocks until it compiles. Then as the project matures slowly refactor to remove each of those unsafe blocks until only safe code remains. I get the sense that a lot of the unsafe code in Bevy revolves around how the game simulation and the render pipeline run in parallel, but often share memory. 

### Graphics Programming

After dabbling with basic Bevy games for a while, I moved on to trying to build some compute shader code in Rust. I previously had implemented Boids and Slime Mold using metal shaders as part of an iOS app, and had enjoyed the challenge. It often feels like in modern software development that things like performance, memory usage don't matter. Even mobile phones are basically supercomputers so who cares if your view controller code is inefficient, the limiting factor is probably in the 4k images you're loading anyways. I found it refreshingly challenging writing shader simulation code, where each operation in your code gets run potentially 100s of times each pixel for each frame and you have  < 1/60s time budget to finish the entire frame. 

My goal was to build something that looked cool and I could show off, while learning how to do graphics programming in Rust. The experience I had was that I was absolutely blown away at how cross-platform by default Rust is [^1]. 

#### Observations

- With wgpu, shaders are written in WGSL but are compiled at runtime to the native format whatever platform you're running on. This means that shader code is portable to mac/ios/windows/wasm/etc without code change. (unsure if there are any caveats or restrictions with this, as I only wrote fairly basic code)
- Following Bevy's compute-shader example I was able to get a project up and running quickly, and found the resulting code to be much more structured than my previous shader projects. I'm not sure how to approach loading/unloading of shader assets at a larger project scope, but from what I understand of game engines and game programming that's one of the key piece of complexity to solve (and even AAA titles get it wrong)
- Getting Bevy to run on wasm in a browser was very very easy and the only headaches I had were for getting WebGPU enabled (which is a very new and unsupported tech)
- Debugging shaders on macOS is a challenge. With an Xcode project the built in tools for Metal give you access to snapshots of gpu state (buffers, command queues, etc), but that doesn't extend to projects outside of Xcode. I think there might be a way to include Xcode in a Rust toolchain to use the debugger tools, but it would likely involve compiling a static library first and importing it. [RenderDoc](https://renderdoc.org/) seems to be the go-to tool for shader debugging but it's windows only. Chrome has an extension that is an alpha version of a webgpu shader debugger, but it's very barebones at the moment. 
- `Repr(C)` and bytemuck get you half the way there for byte alignment for passing data to WebGPU [^5], but it still requires manually padding structs where alignOf > sizeOf. 

#### Projects

I wrote a basic ray tracer, based on the [Ray Tracing in One Weekend](https://raytracing.github.io/) series. The basic premise of a ray tracer works, but there are a number of issues with floating point precision/randomness/bugs that make the output not always look great.
- [Code](https://github.com/robertwaltham/rusty-ray-tracing)
- [Github Pages](https://robertwaltham.github.io/rusty-ray-tracing/) - Note: only works in Chrome Desktop

![Ray Tracer](/assets/images/ray_tracer.jpg)

### Conclusion

I had a lot of fun with these projects, and ended up learning a lot. I really appreciate the work that the rust community has put into making the language, toolchain and ecosystem approachable. 

General thoughts about Rust

- It was a lot more approachable than I expected. I had assumed it would be like suffering through learning C the first time. 
- A lot of talk about the 'pros' of Rust start with memory safety, but if I were selling Rust to someone I'd start with mentioning how good the toolchain is (useful error messages, cargo, type system + compile time type-checking, etc)
- I still can't believe how easy it was to get my shader project to work on different platforms
- I had read some blogs about rationales for adopting Rust, and several of them suggested that one of the core value that Rust provided was that in mission critical code it was safe for a dev inexperienced with the project to jump in and make changes. This is unlike in C/C++ where there's a high chance of introducing memory leaks or crashes. This tracked with my experience with Bevy, where I found it very easy to build out game functionality, in what is ostensively a very complex project with all the async coordination of resources between GPU/CPU. I don't think I actually caused a single crash [^2].


[^1]: I've heard that Rust's ABI stability (or lack of formal stability) is an issue on certain types of cross-platform projects, but don't have knowledge of the specifics

[^2]: Ok  ok ok, I caused a lot of runtime panics, but most of them were very straightforward programming errors that could be rectified by reading the error message and following the instructions it provided
	- Runtime panics on startup from not using ECS patterns properly
	- Runtime panics on update from forgetting register types or resources with the ECS
	- Runtime panics unwrapping None, in places where it was an obvious code error
	- Runtime panics improperly padding structs or mis-sizing buffers being sent to the GPU
	- Shader compile errors, due to coding errors in my WGSL code
	- Deadlocked macOS due to infinite loop in my shader code

[^3]: [Chris Biscardi](https://www.youtube.com/@chrisbiscardi) and [Logic Projects](https://www.youtube.com/@logicprojects) specifically

[^4]: Thanks in part to [this youtube video](https://www.youtube.com/watch?v=uh9i3be2wIE)

[^5]: [WGSL alignment docs](https://www.w3.org/TR/WGSL/#alignment-and-size)