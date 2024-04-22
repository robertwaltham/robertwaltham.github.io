---
layout: post
title:  "AI but BYOC (Bring Your Own Cloud)"
date:   2024-04-21 15:20:06 -0700
categories: engineering
---
![A watercolour painting of a cat](/assets/images/IMG_0087.jpg)

With generative AI in vogue right now, I wanted a project to work on which allowed me to tinker with current models. This would let me learn how to set up and run the models, and to start understanding what their capabilities are and how those capabilities can add to human/computer interfaces in software. My area of expertise is primarily iOS app development, making it natural to choose an iOS app. 

I also wanted to explore running the models locally, as I have a Windows gaming PC which sits unused most of the time. This adds complexity to the project, as it would require a secure solution to connect it to a device via the internet, however it means that usage is basically free. [^1] The main tradeoff is that I would not be able to distribute the app on the app store [^2] as the usage doesn't really scale past a single user. The project goals are primarily about learning, so I decided this wasn't a big downside - and can refactor to use OpenAI's API or whatever later if needed.

The app I settled on trying to build was a companion app for Dungeon Masters [^3] to aid in generating artwork, backstories, characters, etc for use in tabletop games. This requires both text->text and text/image->image capabilities, meaning I would have to run at least two different types of models.

Goals 
- Host generative AI models on my Windows PC on my local network
- Build an iOS app which can access these models via the internet
- Do so in a way that is secure and cost effective 

Learning Outcomes
- Hosting generative AI toolchains
- Exploring the capabilities of current open source models
- Implement an iOS app using the latest ecosystem features (SwiftUi, Concurrency, etc)

App Workflows
- User can generate characters and items using basic D&D terminology (race, class, etc)
- Generation includes descriptions and portraits or artwork
- Generation does not include game stats or mechanics 
- User can share and export the generated assets 

![A watercolour painting of a cat](/assets/images/IMG_0088.jpg)

### Running Models Locally

With the requirement to generate both images and text, I needed to run both LLMs and Stable Diffusion locally. There exist toolkits for running both natively on Windows:
- Large Language Models -> [Ollama](https://ollama.com/)
- Stable Diffusion -> [Stable Diffusion Webui](https://github.com/AUTOMATIC1111/stable-diffusion-webui)

I had some pain points installing and running them, mostly to do with correctly installing dependencies and setting environment variables on Windows [^4]. They both provide a RESTful API to interact with the models so once they were running it was just a matter of connecting to them from the app.

### Connecting via the Internet

I needed a solution to connect services on my local Windows machine to the internet in a way that could navigate the NAT on my router without opening ports. Searching around I found that [Ngrok](https://ngrok.com/) neatly solves the problem; however, I wanted to use a custom domain which requires a $15/mo subscription - more than I wanted to spend on this project. 

I found an alternative [^5] to self host a similar solution, using a cloud VM to host a reverse proxy and an SSH tunnel, which is a much cheaper and flexible solution. [^6]

![Architecture Diagram](/assets/images/ai_architecture.png)

Getting all of this set up properly took some time [^7] but worked as soon as it was set up. The main problem I had to debug with the setup was adding `proxy_buffering off;` to stop Nginx from mucking up the stream generation from Ollama. 

See my sample Nginx conf: [here](https://github.com/robertwaltham/dungeonsandllamas/blob/main/sample.conf)

### Current Status

App code: [here](https://github.com/robertwaltham/dungeonsandllamas)

The app has some basic functionality working
- image->image generation from a Pencil Kit drawing 
- image->text, text->text with streaming text generation
- model/lora selection for Stable Diffusion
- Stable Diffusion generation history and remixing from history entries

I'm trying to use modern best practices and the latest tools available in the iOS ecosystem:
- SwiftUI with Coordinator pattern for navigation
- Swift Concurrency
- Swift Data (unimplemented currently, just using files for now)

The app architecture and design deserves it's own blog post and I hope to write more on this subject at a later date. 
### Next Steps

Now that I have the server set up and the iOS client connecting, the next main task is to test models and workflows to learn how to to get the result I'm looking for. Second, designing a proper interface for the iOS app to implement the functionality using those workflows. [^8]

Progress may be slow, as I keep getting distracted playing with the image generation ^^

### Samples 

Model: [Stable Diffusion 1.5](https://huggingface.co/runwayml/stable-diffusion-v1-5)
Lora: [GHIBLI_Background](https://civitai.com/models/54233/ghiblibackground)

![A watercolour painting of a cat](/assets/images/IMG_0066.jpg)

Model: [Dungeons and Diffusions](https://huggingface.co/0xJustin/Dungeons-and-Diffusion)
Lora: [GHIBLI_Background](https://civitai.com/models/54233/ghiblibackground)
![A dragonborn wizard](/assets/images/IMG_0064.jpg)

[^1]: Electricity here is ~11-14 cents/kWh. Not factoring the up front costs for buying the PC or wear/tear on the components as it's likely less than running Path of Exile for an extended period of time anyways.
[^2]: people who I really like might get Testflight access, but only if they ask nicely - and don't try to use it while I'm playing games in the evening
[^3]: or Storytellers or Game Masters or whatever you call the minor god who presides over your tabletop games
[^4]: As much as I rag on macOS for being difficult to navigate homebrew,  python/ruby versions, etc - it's a dream compared to how obtuse Windows is with everything
[^5]:  article: [Roll your own Ngrok with Nginx Letsencrypt, and SSH reverse tunnelling](https://jerrington.me/posts/2019-01-29-self-hosted-ngrok.html)
[^6]: $4/mo for a Digital Ocean Droplet, $20/yr for a domain, $12/yr for an SSL cert - less if you're smart and use a hosting provider that supports let's encrypt. I bought the domain before realizing the provider didn't support let's encrypt and the transfer fee was more than just buying a cert....
[^7]: Mostly the SSL cert validation taking forever - turns out my hosting provider wasn't publishing the DNS record properly? I switched to Digital Ocean's DNS and it worked first try.
[^8]: And spending $5000+ on hardware to make running bigger faster models possible... That doesn't count for my project budget right?