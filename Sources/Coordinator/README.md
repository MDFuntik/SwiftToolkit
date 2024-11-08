# Swift Toolkit Coordinator

A powerful and flexible coordination system for managing navigation and modal presentations in SwiftUI applications.

## Features

- **Type-safe navigation coordination**
- **Modal presentation management**
- **Alert handling**
- **Memory-safe coordinator references**
- **Support for nested navigation flows**
- **iOS 15.0+ support**

## Table of Contents

1. [Introduction](#introduction)
2. [Getting Started](#getting-started)
   - [Creating a Coordinator](#creating-a-coordinator)
   - [Using the Coordinator](#using-the-coordinator)
3. [Advanced Features](#advanced-features)
   - [Modal Presentation Styles](#modal-presentation-styles)
   - [Presentation Resolution](#presentation-resolution)
   - [Nested Navigation](#nested-navigation)
   - [Custom Coordinators](#custom-coordinators)
4. [Best Practices](#best-practices)
5. [Requirements](#requirements)
6. [Conclusion](#conclusion)

## Introduction

The **Swift Toolkit Coordinator** simplifies navigation management in SwiftUI applications by centralizing navigation logic within coordinator classes. This approach promotes a clear separation of concerns, making your codebase more maintainable and scalable.

## Getting Started

### Creating a Coordinator

To create a basic coordinator that handles both navigation and modal presentations, follow these steps:

1. **Define the Coordinator Class**

   Create a coordinator class that inherits from `NavigationModalCoordinator`. This class will manage the navigation flows and modal presentations within your application.

   ```swift:Sources/Infrastructure/AppCoordinator.swift
   import DI
   import SwiftUI
   import Domain
   import Coordinator
   
   // MARK: - AppCoordinator
   
   public final class AppCoordinator: NavigationModalCoordinator {
       public init() {
           super.init(initialScreen: .home)
       }
   
       // MARK: - Navigation Screens
   
       public enum Screen: Hashable, ScreenProtocol {
           case home
           case detail(Item)
           case favorites
           case categoryItems(String)
           case sourceItems(Source)
           case search
       }
   
       public func destination(for screen: Screen) -> some View {
           switch screen {
           case .home:
               HomeView(viewModel: makeHomeViewModel())
           case let .detail(item):
               DetailView(
                   viewModel: makeDetailViewModel(),
                   item: item)
           case .favorites:
               FavoritesView(viewModel: makeFavoritesViewModel())
           case let .categoryItems(category):
               CategoryItemsView(
                   viewModel: makeCategoryItemsViewModel(),
                   category: category)
           case let .sourceItems(source):
               SourceItemsView(
                   viewModel: makeSourceItemsViewModel(),
                   source: source)
           case .search:
               SearchView(viewModel: makeSearchViewModel())
           }
       }
   
       // MARK: - Modal Flows
   
       public enum ModalFlow: Hashable, ModalProtocol {
           case settings
           case addItem
           case shareItem(Item)
           case filter(FilterOptions)
           case alert(AlertType)
   
           var style: ModalStyle {
               switch self {
               case .settings:
                   return .cover
               case .addItem:
                   return .sheet
               case .shareItem:
                   return .overlay
               case .filter:
                   return .sheet
               case .alert:
                   return .overlay
               }
           }
       }
   
       public func destination(for flow: ModalFlow) -> some View {
           switch flow {
           case .settings:
               SettingsView(viewModel: makeSettingsViewModel())
           case .addItem:
               AddItemView(viewModel: makeAddItemViewModel())
           case let .shareItem(item):
               ShareSheet(item: item)
           case let .filter(options):
               FilterView(
                   viewModel: makeFilterViewModel(),
                   options: options)
           case let .alert(type):
               alertView(for: type)
           }
       }
   
       // MARK: - Alert Types
   
       public enum AlertType: Hashable {
           case error(Error)
           case deleteConfirmation(Item)
           case markAllRead
           case clearCache
       }
   
       private func alertView(for type: AlertType) -> some View {
           switch type {
           case let .error(error):
               AlertView(
                   title: "Error",
                   message: error.localizedDescription,
                   primaryButton: .default("OK"),
                   secondaryButton: .default("Retry") {
                       handleErrorRetry(error)
                   })
           case let .deleteConfirmation(item):
               AlertView(
                   title: "Delete Item",
                   message: "Are you sure you want to delete '\(item.title)'?",
                   primaryButton: .destructive("Delete") {
                       handleItemDelete(item)
                   },
                   secondaryButton: .cancel())
           case .markAllRead:
               AlertView(
                   title: "Mark All as Read",
                   message: "Are you sure you want to mark all items as read?",
                   primaryButton: .default("Mark All") {
                       handleMarkAllRead()
                   },
                   secondaryButton: .cancel())
           case .clearCache:
               AlertView(
                   title: "Clear Cache",
                   message: "This will delete all cached content. Continue?",
                   primaryButton: .destructive("Clear") {
                       handleClearCache()
                   },
                   secondaryButton: .cancel())
           }
       }
   
       // MARK: - Error Handling
   
       private func handleErrorRetry(_ error: Error) {
           // Implement error retry logic
       }
   
       private func handleItemDelete(_ item: Item) {
           // Implement item deletion
       }
   
       private func handleMarkAllRead() {
           // Implement mark all as read
       }
   
       private func handleClearCache() {
           // Implement cache clearing
       }
   
       // MARK: - Navigation Helpers
   
       public func showDetail(_ item: Item) {
           show(.detail(item))
       }
   
       public func showCategory(_ category: String) {
           show(.categoryItems(category))
       }
   
       public func showSource(_ source: Source) {
           show(.sourceItems(source))
       }
   
       public func showError(_ error: Error) {
           present(.alert(.error(error)))
       }
   
       public func showShareSheet(for item: Item) {
           present(.shareItem(item))
       }
   
       // MARK: - ViewModels Factory
   
       private func makeHomeViewModel() -> HomeViewModel {
           DI.resolve()
       }
   
       private func makeDetailViewModel() -> DetailViewModel {
           DI.resolve()
       }
   
       private func makeFavoritesViewModel() -> FavoritesViewModel {
           DI.resolve()
       }
   
       private func makeSettingsViewModel() -> SettingsViewModel {
           DI.resolve()
       }
   
       private func makeAddItemViewModel() -> AddItemViewModel {
           DI.resolve()
       }
   
       private func makeFilterViewModel() -> FilterViewModel {
           DI.resolve()
       }
   
       private func makeCategoryItemsViewModel() -> CategoryItemsViewModel {
           DI.resolve()
       }
   
       private func makeSourceItemsViewModel() -> SourceItemsViewModel {
           DI.resolve()
       }
   
       private func makeSearchViewModel() -> SearchViewModel {
           DI.resolve()
       }
   }
   ```

2. **Define Navigation Screens**

   Enumerate all the possible screens your application can navigate to. Each case in the enum represents a unique screen or view.

   ```swift:Sources/Infrastructure/AppCoordinator.swift
   public enum Screen: Hashable, ScreenProtocol {
       case home
       case detail(Item)
       case favorites
       case categoryItems(String)
       case sourceItems(Source)
       case search
   }
   ```

3. **Define Modal Flows**

   Similar to navigation screens, define an enum for modal flows such as presenting settings, adding new items, sharing content, filtering options, or displaying alerts.

   ```swift:Sources/Infrastructure/AppCoordinator.swift
   public enum ModalFlow: Hashable, ModalProtocol {
       case settings
       case addItem
       case shareItem(Item)
       case filter(FilterOptions)
       case alert(AlertType)
   
       var style: ModalStyle {
           switch self {
           case .settings:
               return .cover
           case .addItem:
               return .sheet
           case .shareItem:
               return .overlay
           case .filter:
               return .sheet
           case .alert:
               return .overlay
           }
       }
   }
   ```

### Using the Coordinator

#### Setting up the Root View

Integrate the coordinator into your root view to manage the initial navigation flow.

```swift:Sources/NewsPresentation/App/NewsApp.swift
import SwiftUI

@main
struct NewsApp: App {
    @StateObject private var coordinator = AppCoordinator()
    
    var body: some Scene {
        WindowGroup {
            coordinator.view(for: .home)
                .environmentObject(coordinator)
        }
    }
}
```

#### Navigation

Use the coordinator to navigate between different screens.

**Push a new view onto the navigation stack:**

```swift
coordinator.present(.detail(item))
```

**Pop the top view:**

```swift
coordinator.pop()
```

**Pop to root:**

```swift
coordinator.popToRoot()
```

#### Modal Presentation

Manage modal presentations through the coordinator.

**Present a modal view:**

```swift
coordinator.present(.settings)
```

**Dismiss the current modal:**

```swift
coordinator.dismiss()
```

#### Alerts

Display alerts using the coordinator.

**Show a simple alert:**

```swift
coordinator.present(.alert(.error(someError)))
```

**Show an alert with custom actions:**

```swift
coordinator.present(.alert(.deleteConfirmation(item)))
```

## Advanced Features

### Modal Presentation Styles

The coordinator supports various modal presentation styles to fit different use cases.

- `.sheet`: Partial screen cover (default)
- `.cover`: Full screen cover
- `.overlay`: Overlay on top of current navigation

Define the presentation style within your `ModalFlow` enum:

```swift:Sources/Infrastructure/AppCoordinator.swift
public enum ModalFlow: Hashable, ModalProtocol {
    case settings
    case profile(User)
    
    var style: ModalStyle {
        switch self {
        case .settings:
            return .sheet
        case .profile:
            return .cover
        }
    }
}
```

### Presentation Resolution

Control how new presentations handle existing modals.

```swift:Sources/Infrastructure/AppCoordinator.swift
// Present over all existing modals
coordinator.present(.settings, resolve: .overAll)

// Dismiss current modal and present a new one
coordinator.present(.settings, resolve: .replaceCurrent)
```

### Nested Navigation

Handle complex navigation flows by embedding child coordinators.

```swift:Sources/Infrastructure/AppCoordinator.swift
public enum ModalFlow: Hashable, ModalProtocol {
    case childFlow(ChildCoordinator = .init())
}
```

### Custom Coordinators

For simpler flows, leverage `CustomCoordinator` to manage specific navigation logic.

```swift:Sources/Infrastructure/SimpleCoordinator.swift
import SwiftUI
import Coordinator

final class SimpleCoordinator: CustomCoordinator {
    func destination() -> some View {
        Text("Simple View")
    }
}
```

## Best Practices

1. **Define Clear Enums:** Define your screens and modal flows as enums conforming to `ScreenProtocol` and `ModalProtocol` to ensure type safety.
2. **Single Responsibility:** Keep each coordinator focused on a specific part of the application to maintain clarity and ease of maintenance.
3. **Use Child Coordinators:** For complex or nested flows, utilize child coordinators to manage their own navigation stacks.
4. **Choose Appropriate Modal Styles:** Select modal presentation styles (`.sheet`, `.cover`, `.overlay`) based on the context and user experience requirements.
5. **Manage Memory Effectively:** Use weak references where necessary to prevent memory leaks and ensure that coordinators are deallocated appropriately.

## Requirements

- **iOS 15.0+**
- **Swift 5.7+**
- **SwiftUI**

## Conclusion

Implementing the **Swift Toolkit Coordinator** in your SwiftUI applications enhances scalability and maintainability by centralizing navigation logic. By following this guide, you can efficiently manage complex navigation flows, modal presentations, and alert handling, resulting in a clean and organized codebase.

Если у вас возникнут вопросы или потребуется дополнительная помощь, не стесняйтесь обращаться!
