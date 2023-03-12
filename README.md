# ModifierSampleCode
# Activity Indicator with SwiftUI Modifier

[Medium Link](https://medium.com/@davidhu-sg/activity-indicator-with-swiftui-modifier-5234b033d39a)

While working on my latest update for [MapKit in SwiftUI](), I noticed the need for a loading animation to indicate when the app is waiting for a response from the server.

To achieve this, I decided to create a custom view that would cover the entire screen and display an indicator until the data was loaded. After some research, I discovered that using the `.modifier()` method in SwiftUI was the perfect solution for my needs.

> In SwiftUI, a modifier is a method that you can call on a view to change its appearance or behavior. A modifier takes the form of a function that takes a view as its input and returns a modified view.
> 
> Modifiers allow you to customize the appearance and behavior of a view without having to create a new subclass of that view. They are composable, which means you can chain them together to create more complex view hierarchies.

Before we can display a loading animation in our SwiftUI app, we need to create a custom view that communicates with UIKit and calls the `UIActivityIndicatorView` API. This view can be implemented using the `UIViewRepresentable` protocol, which allows us to wrap a UIKit view and use it in our SwiftUI code. We can then use this custom view in our app to display the loading animation and cover the entire screen while we wait for a response from the server.

```swift
struct ActivityIndicator: UIViewRepresentable {
    @Binding var isAnimating: Bool
    let style: UIActivityIndicatorView.Style

    func makeUIView(context: UIViewRepresentableContext<ActivityIndicator>) -> UIActivityIndicatorView {
        return UIActivityIndicatorView(style: style)
    }

    func updateUIView(_ uiView: UIActivityIndicatorView, context: UIViewRepresentableContext<ActivityIndicator>) {
        isAnimating ? uiView.startAnimating() : uiView.stopAnimating()
    }
}
```

After creating our custom view that wraps the `UIActivityIndicatorView`, we can implement the loading animation using the `ZStack` and `background` modifiers in SwiftUI. The loading view can be designed to cover the entire screen and display an animation that indicates the app is waiting for a response from the server. This can be achieved by adding the loading view to the `ZStack` and setting its background color to a semi-transparent black to overlay the main content.

![screenshot](https://cdn-images-1.medium.com/max/1600/1*dr80rEHj_GRt2hXTAqnTBg.png)

The sample code below demonstrates how to use the `AnimatableModifier` protocol to create a loading view that can be animated in SwiftUI. When creating a custom modifier for animation, it's important to ensure that the modifier conforms to the `AnimatableModifier` protocol. This allows SwiftUI to animate the view's changes over time, giving us the smooth loading animation we're looking for.

```swift
struct ActivityIndicatorModifier: AnimatableModifier {
    var isLoading: Bool

    init(isLoading: Bool, color: Color = .primary, lineWidth: CGFloat = 3) {
        self.isLoading = isLoading
    }

    var animatableData: Bool {
        get { isLoading }
        set { isLoading = newValue }
    }

    func body(content: Content) -> some View {
        ZStack {
            if isLoading {
                GeometryReader { geometry in
                    ZStack(alignment: .center) {
                        content
                            .disabled(self.isLoading)
                            .blur(radius: self.isLoading ? 3 : 0)

                        VStack {
                            Text("Now Loading")
                            ActivityIndicator(isAnimating: .constant(true), style: .large)
                        }
                        .frame(width: geometry.size.width / 2,
                               height: geometry.size.height / 5)
                        .background(Color.secondary.colorInvert())
                        .foregroundColor(Color.primary)
                        .cornerRadius(20)
                        .opacity(self.isLoading ? 1 : 0)
                        .position(x: geometry.frame(in: .local).midX, y: geometry.frame(in: .local).midY)
                    }
                }
            } else {
                content
            }
        }
    }
}
```

To see the loading animation in action, we need to write some sample code that demonstrates how it works. The example code below provides a basic implementation of the loading view that can be used in a SwiftUI app. This code shows how to create the custom `LoadingView` modifier that conforms to `AnimatableModifier` and how to use it in a simple view to display the loading animation over the main content.

```swift
import SwiftUI

struct ContentView: View {
    @State private var isLoading: Bool = false
    
    var body: some View {
        Button {
            // mock a network request or any loading action
            isLoading = true
            DispatchQueue.main.asyncAfter(deadline: .now() + 2.0) {
                isLoading = false
            }
        } label: {
            VStack {
                Image(systemName: "arrow.triangle.2.circlepath.circle")
                    .imageScale(.large)
                    .foregroundColor(.accentColor)
                Text("Mock Loading")
            }
        }
        .padding()
        .modifier(ActivityIndicatorModifier(isLoading: isLoading))
    }
}

struct ContentView_Previews: PreviewProvider {
    static var previews: some View {
        ContentView()
    }
}
```

That's it! Now you know how to use the `Modifier` protocol to show an activity indicator in SwiftUI. If you have any additional ideas or feedback on this implementation, please feel free to leave them in the comments below. I'd also like to extend a special thanks to the author of this [StackOverflow post](https://stackoverflow.com/questions/56496638/activity-indicator-in-swiftui/56496896#56496896) for their helpful insights and code examples.
