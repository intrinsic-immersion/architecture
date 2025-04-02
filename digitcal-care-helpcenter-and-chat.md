
# Digital Care

After I shipped a react-navite based GoDaddy mobile app (discontinued) and was "done with mobile"
there was an openning for a new manager in Digital Care. 

I inherited the Digitcal Care apps services and and team(s). These team consisted of two very junior teams, I was able to fill in a couple senior roles though. I was intrumental in proposing and providing guidance and the encouragement to improve our systems. which touched many aspects of GoDaddy given all the pages used the chat client and all products have technical articles. 

The task was to modernize Help Center stack with React and NextJS, on-board LivePerson and level up our messaging support with more engaging experience and better chat bots and then replace the old 2008 in-house built MySQL backed article DB with a headless CMS

I also was mobbing with my team to convert all the pages from Jade to React, onboard Gasket (NextJS) for how-to videos which allowed us to move the rest of the site over to NextJS. I was a very hands-on manager with a pair station and would work coding with my team as much as possible. I hired my replacement manager and dropped to an IC during this time period to travel in our RV.   

I included what I started with and what we ended with.  

### The Start: Digital Care High Level
```mermaid
flowchart TD
  subgraph DigitalCare[Inital Digital Care]
    subgraph edge    
      
      HelpCenter[HelpCenter <br/> Tech Articles + Search + Static Pages]

      ChatClient[Chat Client<br/>simple idle chat client to queue for live agents]

      HelpCenter <--> 
      Elastic <--> 
      MySQL <--> 
      EditorApp <--> 
      Writer
      
      ChatClient <--> 
      AgentRouting[Agent Routing Service] <--> 
      GoDaddyAgent[GoDaddy Live Agents]

    end

    UserDevice <--> ChatClient
    UserDevice <--> HelpCenter
  end
```

### Digital Care: The Initial Helpcenter
```mermaid
flowchart LR

subgraph Edge
    
    subgraph Expresso[custom k8s manager]
        subgraph HelpCenterPods
            Search
            Articles
        end
    end

    subgraph EditorServices[Managed Services]
        EditorApp --> MySQL
        MySQL[(MySql on last leg)]
        Elastic <-->|search query| Search
        Elastic <-->|slug lookup| Articles
    end

    subgraph ServicesCluster[Team-Managed Services]
        PubExpress[[Pub-Express: Node API and cron for publishing actions]] -->|article previews <br/> preview index| Elastic
        MySQL -->|preview <br/> or nightly| PubExpress
        PubExpress -->|Nightly cron publish <br/> indexing for search| Elastic
        Jenkins -->|Deploy branch <br/> or main prod| HelpCenterPods
    end

    HelpCenterRepo -->|push| Jenkins -->|pubexpress| PubExpress
    PubExpressRepo -->|push| Jenkins

end

Writers <--> EditorApp[EditorApp <br/> old .NET app 2009]
UserDevice -->|search query| HelpCenterPods
UserDevice -->|page-slug| HelpCenterPods
Expresso -->|Search Results or Article| UserDevice
```

## Digital Care: Helpcenter we created with Contentful and Gasket (NextJS)

[Help Center Excalidraw](https://excalidraw.com/#json=Ba7auB9vcfzKoSrdLz4s9,8XOz6-10OuGOEtTTAP1Igw)

I broke mermaid...
![Help Center](help-center.png)



## Digital Care: Initial Chat Client

```mermaid
flowchart LR

    subgraph edge[Edge]
        AgentInterface
        ChatServices
        ChatClientRepo[ChatClientRepo: in-house built chat client] --> Jenkins --> DeployedBundle[Deploy bundle to CDN dev/test/prod]
        DeployedBundle -->|deploys latest version| HeaderServices
        HeaderServices -->|checks for update| DeployedBundle
        HeaderServices --> Deployed[Staged: placed on the page with header]
    end

    subgraph browser[User Device]
        ChatClient <-->|chat traffic| ChatServices <-->|chat traffic| AgentInterface
        ChatClientNote[KTLO: Simple chat client FSM that sent context/intent and put you queue for live chat if available; we were shopping for turn-key solutions]
    end

```

## Digital Care: Bot Agents with Sidekick

```mermaid
flowchart TD
    subgraph DialogFlow
        DialogflowAPI
        a@{ shape: "rounded", label: "Before AI: intent sniffing routing bots
          Dialog structures currated by writers. 
          user context used to help route to the right agent"}
        b@{shape: "rounded", label: "big dreams on specialized agents and upcoming AI"}
    end

    subgraph edge[GoDaddy Edge]
        ChatbotsApi[ChatbotsApi: central bot communication] <-->|articles| Redis[Redis not elastic]
        ChatbotsApi <-->|user context and bot traffic| DialogflowAPI
        ChatbotsApi <-->|user context| ProductsService
        Contentful[Contentful Articles] --> Redis
    end

    subgraph liveperson
        subgraph LiveEngage
            IsAgentAvailable[Is an Agent avaiable?] -->|agent has user context, intent and bot conversations to review| LiveAgent
            IsAgentAvailable -->|enter queue, use bot to gather intent| BotConnector
        end
        LiveAgent
        BotConnector[BotConnector: a liveperson interface configured to talk to dialog flow ] -->|bot communications traffic| ChatbotsApi
    end

    subgraph browser
        SideKick <-->|bot and chat communications| LiveEngage
        SideKick -->|track sessionid, intent, health, status| ChatbotsApi
        c@{shape: "rounded", label: "Sidekick: ended up being a Live Person chat interface 
        We could use plugins and event handlers in it"}
    end

    LiveAgent[LiveAgent interface for support agents] --> GodaddyGuides[Guides: Real Agents]
```


### I was taking shortcuts due to mermaid, so I drew it out.

```mermaid
flowchart LR

Contentful --> 
    DraftArticles --> 
        ArticlesTest -->|transform into old format| PubExpress

Contentful --> 
    PublishedArticles --> 
        ArticlesProd --> 
            PubExpress

Contentful

BrightCove -->|video + transcript| DraftVideos -->|video catalog| VideoTest
BrightCove -->|video + transcript| PublishedVideos -->|video catalog| VideoProd


subgraph Edge
    Elastic -->|draft articles| SearchTest
    Elastic -->|prod articles| SearchProd
    
    HelpCenterRepo[Repo: helpcenter is now a NextJS app with proper routes] -->|push| Jenkins
    
    subgraph AWS[AWS EKS + ingress]
        subgraph HelpCenterPodsTest
            SearchTest[Search Page]
            ArticlesTest[Articles Page]
            VideoTest[HowTo Page]
        end
        subgraph HelpCenterPodsProd
            SearchProd[Search Page]
            ArticlesProd[Articles Page]
            VideoProd[HowTo Page]
        end
    end

    ChatbotApi <--> Elastic

    subgraph EditorServices[Managed Services]
        Elastic
    end

    subgraph ServicesCluster[Team-Managed Services]
        PubExpress[[Pub-Express: Node API and cron for publishing actions]] -->|article previews <br/> preview index| Elastic
        PubExpress -->|Nightly cron publish <br/> indexing for search| Elastic
        Jenkins -->|Deploy branch <br/> or main prod| HelpCenterPods
    end
end

AWS -->|Search Results <br/> Article <br/> Video| UserDevice

VideoProducers --> BrightCove
Writers <--> Contentful

UserDevice --> SearchQuery --> SearchProd
UserDevice --> PageSlug --> ArticlesProd
UserDevice --> VideoSlug --> VideoProd
```