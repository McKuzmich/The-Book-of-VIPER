## Simple vs Complex Module

At the time we have started to employ VIPER architecture in our work, we have used "one screen - one module" concept.
It worked fine because of our screens were mostly simple tables. But problems arise with an introduction of first complex screen.

## Examples of complex moudules in email application - settings screen and message view screen
![submodules.001](../Resources/submodules/submodules.001.png)
Complex screens could be various. For instance, settings screen manages mulpitple elements not related to each other.
Message view screen includes message header, contact collections, attachments and message view, which requires specific transformations.  

## Downsides of complex module
![submodules.002](../Resources/submodules/submodules.002.png)
What problems are brought by complex modules?

- Disparate data in one module
- Complex logic
- Hard to test
- Impossible to reuse
- Hard to alter functions or configuration

For example, settings module should store information on name and signature, connected mailbox list, notification status. Every cell may affect its neighbors by section but in theory these cells should not affect any other cells. Though they are able to.
Such complex module can't be reused, because everything in it was designed for the settings of a particular app. Any additions or changes require familiarization with a whole module.

## Settings module and message view module separation into submodules
![submodules.003](../Resources/submodules/submodules.003.png)
Obvious enough separation of settings module is separation by sections. This is logical and clear. The first module - user information, a second module - a list of connected mailboxes and so on.

Message view screen would consist of following submodules: header, contacts, folding attachments and message body view.

## Submodules: benefits and drawbacks

### Submodule benefits:

- Single responsibility - every module may and ideally should be responsible for one function.
- Testability - small modules are easier to test.
- Reusability - contacts or attachments module can be used on compose message screen and even in other apps, e.g. instant messengers.
- To add a new feature, add a new submodule.
- Ability to create various configurations, add developer-only options for example.

### Drawbacks:

- Additional code. It provides provisioning and coordinated work of submodules. It is required to test thoroughly.
- Risk of excessive separation. Systems with too small modules tend to crumble. For example, you should not design a module for every option in settings.
- Data flow complication. In case of submodule's submodule returns some data to main module chain of calls would be more complex than in case of monolith module.

That is why separation into submodules requires adequate analysis.

## A survey of separation methods
![submodules.004](../Resources/submodules/submodules.004.png)
We have used 5 variants of composite modules while working on our projects:

- Containter module
- Scroll View Container module
- Table with section modules
- Table with cell modules
- View module

Let's examine them in detail

## Container module
![submodules.005](../Resources/submodules/submodules.005.png)

This is similar to the Container and EmbedSegue, but managed from module.
Presenter requests router to add child submodule. Then router initializes this submodule, passes data to submodule and adds submodule's ViewController as child ViewController to module's ViewController.

This approach fits good when the module has its own logic like tables which have a complex header.

## Example of container module

Imagine a list of user's posts, a header with avatar and a send message function on top of them.
The module processes only posts - loads, presents, handles selecting post's cell. If we add profile loading function and transition to composing message to the module, it will become more complex. So it is better to move these functions to separate module. In order to work this submodule requires only user ID.

## Scroll View Controller
![submodules.006](../Resources/submodules/submodules.006.png)
Sophisticated case of container module. It is how message view screen of the "Рамблер/почта" application is implemented. There are multiple containers inside ScrollView, every container is managed independently by a submodule. Contacts module works only with contact loading and presenting, responsible for collapsing/unfolding list. Attachments module separates attachments to pictures and documents, and is responsible for collapsing/unfolding list. Message view module processes the body of email message, adds hyphenation, loads and caches inline attachments with pictures.

Plus, as it should be in the VIPER, everything related to the loading, processing and business logic is done in submodules' interactors. In order to do this, they have references to the services. Technically, each such submodule can be maximized on the entire screen.

## TableView with section modules
![submodules.007](../Resources/submodules/submodules.007.png)

This approach was used for the settings table. Interactor is given a list of submodules to display on initialization. It polls each submodule and asynchronously receives view-model array for each submodule, combines them to one array and passes the array to display. The cell factory gets all the necessary data for the creation and configuration of the cell from the cell model, therefore it is universal for all submodules.

The cell-model factories are used as a View in submodules, they convert data from a presenter to suitable for display as a list of cells data, broadcasts events from cells to presenter, that is, fully perform all of the work of a View.

It allows notification module to work only with notifications, and allows connected mailbox module to load mailbox list for displaying from its service.

## Table with cell modules
![submodules.008](../Resources/submodules/submodules.008.png)
There are situations where content that is displayed in the table is very complex, the cell handles too many user actions, the cell needs to download extra data or to display CollectionView inside. In this case, each cell can be a separate module.

The difficulty is that we need to make not only cells reusable, but also VIPER modules. Cell Factory should configure the module state when displaying cell. For example in the case of CollectionView inside the cell we need to pass it not only a list of objects for display, but also to set corresponding ContentOffset.

But it allows handling all actions and events within this submodule including connection with the server.

## View module
![submodules.009](../Resources/submodules/submodules.009.png)
The most obvious case - View module. This method easily incorporated with the others. For example, cell module could have a gallery or video-player submodule. Such submodules are easy to reuse on different screens.

## When to make use of submodules
![submodules.010](../Resources/submodules/submodules.010.png)

- Reduces compexity of main module
- Easy submodule reuse
- Simplifies adding new functionality
- Simplifies testing

## When to avoid submodules
![submodules.011](../Resources/submodules/submodules.011.png)

- Significantly increases amount of code
- Complicates logic
- Complicates debug
- Hard to maintain
