
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
[Mermaid Editor](https://mermaid.live/edit#pako:eNp9VG1P2zAQ_isnf9qklpWUvkUICQHSNgHqKNOkJXxwkyOx6tjBL0BH-e9z4mQpbTV_aH32c3fP3T3xG0lkiiQkj1y-JDlVBq7vYhELbZeZomUOV2mGsQC3_G938Voq1FpGidVGFrCaaiiooBmqBw_9AP-KvLxAYVDNZao7QLUWSFWSfzw7V4YlHLeQKFJv1JtdNikzUi1QPbMEdXRTE0mhPdhi5JHnZQn9_hncrBc_rrvL2ow-ub8nDlIAp9oAx-zzdoDqkCVw6vw3uqYOTxbVerNXyEcotxlwKVe23OyUd6iilvoFd_1FFd0jLfr_qWtul81Iosjt-40Rwq2bMJzPvwEVKSTKVfUoFZR2yZnOmciAJoZJoR8eqo5sqGcGzvuZ4YuG06X6ctaawESKr5u2sp3O1QFapPdzqQTLcsNdezqKh2jXzrce63k2HJtIdeaKb0Xf9_0Aj-8oVkz4YJdYcrmGpaIiyTs-BWUutpLp5qAqt4bRXd9hKX11Vru021kcS_QV7FfY2YcDVImafL8Uc3l0rZVOpFEn14Y_T-Ho9uoeqDsKBoOZk8BPjeoSK0HAviZ3S9wBl05P_Uqa-8j2C69xXtpwh9pyo6s2NhLeQBeR9EiByvU3dU_KW9WCmJgcC4xJ6LYpVauYxOLd4ag1crEWCQmNstgjStosJ-Ej5dpZtkypwUtG3bdQ_DstqfgtZdG6OJOEb-SVhMeD2dF4Ojk5Gc6OZ-PxaNojaxIOJ-Oj6WQQTKejWTAMhsfj9x75UwcYuIvRwK1gdjIJxqNg2COZqmg3bNxQUF1IK4yLHvQI1mO48Y9l_Wa2JP2AvOP7X7H9wlw)

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

[Mermaid Editor](https://mermaid.live/edit#pako:eNqFVW1v2zYQ_isHflYDx3GcRBjWuk0aGNuAoBkwYFY-0OJZIkKRGl-cpnb--46Ubamtk-qDRB559xyf53jasNIIZDlbKfNU1tx6-Pu60ECPC8vK8raGa8mVqT7Thm4hPp0tOs3u5r2Zf9iAq3mLORTMmqAFioJloPgSVbR9xJWxCLN5DlJ71B6clquV1BXQdh-_S-NdH3EPBc7bUPpg0UEZrOUeBSyf4clKj9adwNAlOLRQGgL46uNEgDdQo2oTCMaZrxGsrGoPvKI0CvbSB1h-2Lx5iKWsQFjkjQOjwbVYUoryG8GkWA64FhDa0jTxPLP5IThqUegf2EVR4eLWXHMhnuGGJg99Ip9q7iMds1YuBuMcSoKxXEWu6JxNE7QsuZdGP8Bv7979viUdZanQbeELCukW6Q2adqPijtaOg3TO37EXjxJRCI5kKrevKf9mlC3cWSNIPnePdi1LHPiZVAaroBb9EGa7_B-AQnVneJ1BJdfYUg0Y3Yc9LP5Jize6ImGGBQIwd7Oo1WzNJemqcDGPskEyAidrNL5P-NukKtTcfVdY2b6C9xSRfU1pJB1crDGLa4lP25RDCvx2CgmLDATxX8CAWYTryDdQcapYu4PcwkfjiS6NpTe2j5rY2U-OoA69FsNJDnxAY0KxK15iPNNKVnTr0hXyXD3Gr-iuZCwC6Cj6qRBdXzGD0nhdxKU1T0Run-u9FPiHLB-7aorxI8_Uon4E2h7V-OAevSkVGjl0jhyk2CuXUU_gytcZNRfugzuSanzKX7SDCPVIUHk8GMabD0uMN5-nzOCuIzWl3jPbx_8n0hyUSHK3KlRSdy0E113daaEoBPmC9Md6yUHpxWE0AKKGSzS3rbG7Xre7VbdGxJ5zGyh9t-g-Od01aiuzbhvLWIO24VLQD2ITsQpGVdhgweK5BbePBSv0C-3jwZv7Z12ynNo0ZpGnqmb5iitHs9AK6tfUOUjr5mBtuf7XmGbvQlOWb9hXlo_PJydXk_F4enZ-eTaenGfsmeWnJ6Oz0Wg6vbi4GJ2PaPUlY9-S--jkanp6ejqaXo7HV5PJ9OIyY5WNSe9yibLYT6ScpzCTs4xRP6Gi_6v786Uf4D7Hm7TSeb78D22BXuI)



### Incorrect Attempt 

I left this here as a reference, this is slightly incorrect and I hit the edge of mermaid being pretty

[Mermaid View](https://mermaid.live/edit#pako:eNqNVW1v2jAQ_isnfx3toJSyRlUlRpG6bqtY6VZpCR9MciRWgx3ZDpSV_vfZzguBdaWREvnsu-eeOz92nkkoIiQemadiFSZUavh2F_CADwXXyPU8T-Ho6BICDua5knSuB1KzMEXVmLdPNX2PStuljZaUq7mQC2BcCxBpBNaiegPjfDZ6yiQq9d9UxiVlKsHoQLqxFNHeUhn-agZrfZYsTvRQLNHxXLIIBXwAxzeULDMEXaG_7IJq-IRU01TEG3ArttB3gdWlHAC0pVh-AVf5LJY0S2AUxVgUNkqpMhW74MiyA1p2YAMTpDJMCj77vpntz75rkcm6Ft9rTLOhaRDKO8yEbz8eJGYydJPAFHCxAgq3-KRvJkCzDFZMJ2DQM7MuRa5RTYuEuUo2cIP8kXHVTFIXNXiY-OaF0deJ6RTjsd2m6Xb_asctrbGI1La-6tnW7RdDGNMYp7tOTV36tZhecaw31b8Wq3ux74I8Okhx29ddinb-HRSd20GKzuttim5QDIcJ1TOhBxmDC3tMSm1Uyw2lMS3kBOWShaj875Qb7AiqiUaWGmEvUw1VxQzTXJnG-PdIF0dvAG7Pqu-b8VFpeHBrriYYjL8A5RGEUnB7gUBWHCejG6ChZoKraaG8UuVGlLhkuFJwMZMfLyvTCC3Cp80u_d30DuXWnud0XeQrc5VIDsHmtTSU285X8ErpO7ArzFKxhpm5DcIKxcQuKOP27ESbPQE1mlp21h4Ui1Sq5w5VnuqqtFIrpeXUsYGfCuUV2ibb-FoyeYiyuES3V1bAHyTTdt5pY_ea3OK4qILAjxzlumEXit9ztbKcpHnsjKa4__F07GrXmitpkQWaXwWLzJ_p2TYlIDrBBQbEM8OIyseABPzF-NFci8mah8TTMscWMVdRnBBvTlNlrDyLqMYrRo0yF_VsRvlvIRZViDGJ90yeiHfSOz0-Pz05Oev2PnVPTnstsiZe57jdbbfPzvr9frvXNqsvLfLHhbePz886nU67b97zfr_T77ZILC3pkovZQ5RDkXNtYCwculP2vfjluj9vxbE4f0Xky1-hGYLP)

