## Feature Fabric

> I was going to originally talk through this project, but, most of the moving parts are in the application layer and the service parts seem trivial? We can poke at through some of this if interested.

> This had a huge impact with the tech boucing up to the CTO teams(s) because it was making the abilty for non-engineers to easily explore, create and run experiments .. and provided a mechanism to share and use experiment criteria across apps/teams.

This is an experimentation accelerator that I happened to help build and promote as an IC on the SERP team. When I joined the SERP team the team was very new to React and JavaScript/TypeScript. 

As I was helping them and the codebase level up, I saw they had a FeatureVariant model and a bunch of tedious json configuration they did by hand. I worked with the Director, Experimentation Team, Cart team, and multiple PMs to create a drop-in page solution for any team using React for in-page feature/content experiments that would accelerate our ability to create and share experiments successes and failures.


### Phase 0: Initial state
![alt text](FF-Serp-0.png)

### Phase 1: MVP+ Implementation
![alt text](FF-Serp-1.png)

### Phase 2: Integrated with Experiment service
![alt text](FF-Serp-2.png)


## SERP: (Search Results Page): No Feature Fabric: Phase 0
```mermaid
flowchart TD
    FindApi
    FindApi -->|result| ReactApp
    ReactApp -->|client query| FindApi[FindAPI query<br/>client-side only]

    subgraph edge
        subgraph SwitchBoard
            UserInterface --> ExperimentsDB
            ExperimentsDB[(ExperimentsDB)] --> SwitchboardApi
        end

        SwitchboardApi -->|experiment-ids| Express
        Express -->|app-id| SwitchboardApi

        subgraph SERP[SERP Route]
            Express[Express Server] -->|feature-config| SearchApp
        end
    end

    subgraph SearchApp[SearchApp *client*]
        ReactApp --> FeatureVariantsHOC
        FeatureVariantsHOC[[FeatureVariantsHOC]] -->|variants processed| Render[Render Results]
    end

    User -->|domain-search| Render
    Render -->|domain-results| User
```

## Sequence Chart
```mermaid
sequenceDiagram
  participant FaeatureFabric
  participant FaeatureFabric
  participant Editor
  participant DiffEditor
  note right of Editor: Editor is introduced<br/> in non-production env
  FaeatureFabric->>CMS: fetch fabricdata with app id
  CMS->>FaeatureFabric: fabricdata or error on creds/id miss: fails gracefully 
  FaeatureFabric->>AppBoundary: discovery phase
  AppBoundary->>FaeatureFabric: features/variants discovered
  alt is valid thumbprint
      FaeatureFabric->>Editor: is validate thumbprint?
  else is invalid thumbprint
      DiffEditor->>FaeatureFabric: update fabricData
  note over Editor, DiffEditor: diff editor compares<br/> discovery with loaded fabricdata<br/>helps correct it
      FaeatureFabric->>+DiffEditor: not validate thumbprint?
  end
  alt is save-action
      note over FaeatureFabric, Editor: resolves and updates fabricData in state
      FaeatureFabric->>AppBoundary: re-renders app with updated criteria
      AppBoundary->>Editor: re-loads editor
  end
  alt is publish-action
      Editor->>CMS: Publish
  end
```

```mermaid
sequenceDiagram
    participant React
    participant FeatureFabric
    React->>FeatureFabric:fabricdata
    create participant FeatureA
    FeatureFabric->>FeatureA: feature['A'].enabled
    create participant VariantA
    FeatureA->>VariantA: variant['A'].enabled
    note over FeatureFabric,FeatureA: children[]: many natural components
    create participant VariantB
    FeatureA->>VariantB: variant['B'].isenabled
    create participant ChildrenA
    VariantA->>ChildrenA: checks for contentmod[path].props
    ChildrenA->>VariantA: render variantA children applied props if available
    VariantA->>FeatureA: render VariantA
    FeatureA->>FeatureFabric: render next feature
    create participant ChildrenB
    VariantB->>ChildrenB: checks for contentmod[path].props
    ChildrenB->>VariantB: render variantB children applied props if available
    VariantB->>FeatureA: render VariantB
    FeatureA->>FeatureFabric: render next feature
    FeatureFabric->>React: resolved user interface with experiments
```

```mermaid
sequenceDiagram
    note left of Pod: Render Sequence
    participant Pod
    participant NextJS
    participant Switchboard
    participant Rendered App
    Pod->>NextJS: request
    NextJS<<->>Pod: cached fabric data
    NextJS->>Switchboard: facts data
    Switchboard->>NextJS: all active experiments
    note over NextJS, Switchboard: NextJS resolves fabric data
    NextJS->>Rendered App: resolved fabric data
```

```mermaid
flowchart TD
  Artifactory[Internal Artifactory] -->|npm i feature-fabric| FF

  subgraph SERPNJSSite
      subgraph ReactApp
          
          FF("`
          FeatureFabric
          --FeatureA
          --- Variant1
          --- Variant2
          --- Variant3
          FeatureFabric
          --FeatureA
          --- Variant1
          --- Variant2
          --- Variant3            
          `")
          
          -->
          
          Thumbprint("`
              Feature/Variant Data Model Generated or updated
          `")@{ shape: procs }

          -->

          Editor("`
              Fabric Data Editor Overlay
              - Feature Criteria
              - Variant Criteria
              - Variant props
              - Facts Fixtures
          `") --> Cached

          Editor -- test loop --> FF

      end
  end

  Cached("`
      NextJS static using criteria as key
  `")

  subgraph PodContainer
      NJS@{ shape: procs, label: "NJS Process"}
  end
  
  User -- request --> PodContainer
  PodContainer -- response --> User

  subgraph switchboard
      experiments@{ shape: procs } --> SBRules
      SBRules{is active} -->|yes| exp-results@{ shape: doc }
      SBRules -->|no| exp-results
  end

  PM[Engineer or PM] --> Editor
  PodContainer --> switchboard
  CMS <-- fabric data deployed --> CICD
  CICD -- deployment --> PodContainer
  switchboard -- availble experiments --> Editor
  switchboard --> PodContainer
  Editor -- saves changes --> CMS
  CMS -- loads latest data --> Editor
  NJS <--> SERPNJSSite
  Cached --> NJS
```