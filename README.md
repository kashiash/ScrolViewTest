# ScrolViewTest
Deep Dive into the New Features of ScrollView in SwiftUI 5

Wraz z SwiftUI 5.0, Apple znacząco rozbudowało funkcjonalność ScrollView. Dodano wiele nowych i ulepszonych interfejsów API. W tym artykule przedstawione zostaną te nowe funkcje, mając nadzieję, że pomogą deweloperom w korzystaniu z nich wcześniej i bardziej efektywnie.

```swift
public func contentMargins(_ edges: Edge.Set = .all, _ length: CGFloat?, for placement: ContentMarginPlacement = .automatic) -> some View
```

Dodaje margines do zawartości lub wskaźnika przewijania kontenera przewijanego.

- Nie jest ograniczone tylko do ScrollView, obsługuje wszystkie kontenery przewijane (w tym List, TextEditor, itp.).
- Traktuje wszystkie podwidoki w kontenerze przewijanym jako całość i dodaje do nich margines. Wcześniej było to trudne do osiągnięcia w przypadku List lub TextEditor.
- Domyślna wartość dla ContentMarginPlacement (.automatic) powoduje nierównomierną odległość między wskaźnikiem a zawartością. Jeśli chcesz zachować równomierną odległość, użyj .scrollContent.
- Dotyczy wszystkich kontenerów przewijanych w danym zakresie.

```swift
struct ContentMarginsForScrollView: View {
    @State var text = "Hello world"
    var body: some View {
        VStack {
            ScrollView(.horizontal) {
                HStack {
                    CellView(color: .yellow)
                        // niestandardowy widok nakładki do łatwego wyświetlania dodatkowych informacji
                        .idView("leading")
                    ForEach(0 ..< 5) { i in
                        CellView()
                            .idView(i)
                    }
                    CellView(color: .green)
                        .idView("trailing")
                }
            }

            // Również wpływa na contentMargins
            TextEditor(text: $text)
                .border(.red)
                .padding()
                .contentMargins(.all, 30, for: .scrollContent)
        }
        // Dotyczy wszystkich kontenerów przewijanych w danym zakresie
        .contentMargins(.horizontal, 50, for: .scrollContent)
    }
}
```
Dodaje margines do zawartości lub wskaźnika przewijania w kontenerze przewijanym.

- Nie jest to ograniczone tylko do ScrollView, obsługuje wszystkie kontenery przewijane (w tym List, TextEditor, itp.).
- Traktuje wszystkie podwidoki w kontenerze przewijanym jako całość i dodaje do nich margines. Wcześniej było to trudne do osiągnięcia w przypadku List lub TextEditor.
- Domyślnie marginesy są dodawane zarówno do zawartości, jak i wskaźnika przewijania. Jeśli chcesz zachować długość marginesów tylko dla zawartości, użyj `for: .scrollContent`.
- Dotyczy wszystkich kontenerów przewijanych w danym zakresie.

W strukturze `ContentMarginsForScrollView` mamy przykład zastosowania. Wewnątrz VStacka znajduje się ScrollView z układem HStack, zawierający widoki CellView. Dodatkowo, TextEditor również korzysta z modyfikatora `contentMargins`.


safeAreaPadding

Dodaje wypełnienie do bezpiecznej strefy widoku. W niektórych scenariuszach jest to podobne do korzystania z safeAreaInsets. Na przykład, w poniższym kodzie, dodanie marginesów do kierunku wiodącego ScrollView daje ten sam efekt co użycie bezpiecznych marginesów.

```swift
struct SafeAreaPaddingDemo: View {
    var body: some View {
        VStack {
            ScrollView {
                ForEach(0 ..< 20) { i in
                    CellView(width: nil)
                        .idView(i)
                }
            }
            .safeAreaPadding(.leading, 20)
            // .safeAreaInset(edge: .leading){
            //       Color.clear.frame(width:20)
            //  }
        }
    }
}
```

- Ta właściwość dotyczy nie tylko widoków przewijalnych, ale wszystkich typów widoków.
- Wpływa tylko na najbliższy widok.
- Logika obsługi safeAreaInset i safeAreaPadding dla dodatkowej bezpiecznej przestrzeni na pełnym ekranie jest niespójna.

Na przykład, w poniższych dwóch implementacjach, dolna przestrzeń ScrollView jest różna.

Używając safeAreaInset:

```swift
ScrollView {
    ForEach(0 ..< 20) { i in
        CellView(width: nil)
            .idView(i)
    }
}
.safeAreaInset(edge: .bottom){
    Text("Bottom View")
        .font(.title3)
        .foregroundColor(.indigo)
        .frame(maxWidth: .infinity, maxHeight: 40)
        .background(.green.opacity(0.6))
}
```



Używając safeAreaPadding:

```swift
ZStack(alignment: .bottom) {
    ScrollView {
        ForEach(0 ..< 20) { i in
            CellView(width: nil)
                .idView(i)
        }
    }
    .safeAreaPadding(.bottom, 40)

    Text("Bottom View")
        .font(.title3)
        .foregroundColor(.indigo)
        .frame(maxWidth: .infinity, maxHeight: 40)
        .background(.green.opacity(0.6))
}
```

scrollIndicatorsFlash

Kontroluje wskaźniki przewijania

Użycie scrollIndicatorsFlash(onAppear: true) spowoduje krótkie migotanie wskaźnika przewijania, gdy widok ScrollView się pojawia.

Użycie scrollIndicatorsFlash(trigger:) spowoduje krótkie migotanie wskaźnika przewijania we wszystkich kontenerach przewijalnych w zakresie modyfikatora, gdy podana wartość się zmienia.

```swift
struct ScrollIndicatorsFlashDemo: View {
    @State private var items = (0 ..< 50).map { Item(n: $0) }
    var body: some View {
        VStack {
            Button("Usuń pierwszy") {
                guard !items.isEmpty else { return }
                items.removeFirst()
            }.buttonStyle(.bordered)
            ScrollView {
                ForEach(items) { item in
                    CellView(width: 100, debugInfo: "\(item.n)")
                        .idView(item.n)
                        .frame(maxWidth:.infinity)
                }
            }
            .animation(.bouncy, value: items.count)
        }
        .padding(.horizontal,10)
        .scrollIndicatorsFlash(onAppear: true)
        .scrollIndicatorsFlash(trigger: items.count)
    }
}
```

scrollClipDisable

scrollClipDisable jest używane do kontrolowania czy zawartość przewijana jest przycinana, aby pasowała do granic kontenera przewijanego.

Kiedy scrollClipDisable jest ustawione na false, zawartość przewijana jest przycinana, aby pasowała do granic kontenera przewijanego. Każda część, która wykracza poza granice, nie będzie wyświetlana.

Kiedy scrollClipDisable jest ustawione na true, zawartość przewijana nie jest przycinana. Może się rozciągać poza granice kontenera przewijanego, umożliwiając wyświetlanie większej ilości zawartości.

    Dotyczy tylko ScrollView.
    Dotyczy wszystkich kontenerów przewijalnych w danym zakresie.

```swift
struct ScrollClipDisableDemo: View {
    @State private var disable = true
    var body: some View {
        VStack {
            Toggle("Wyłącz przycinanie", isOn: $disable)
                .padding(20)
            ScrollView {
                ForEach(0 ..< 10) { i in
                    CellView()
                        .idView(i)
                        .shadow(color: .black, radius: 50)
                }
            }
        }
        .scrollClipDisabled(disable)
    }
}
```



extra 1:

Framework SwiftUI udostępnia nowy modyfikator widoku `scrollTargetBehavior`, który pozwala nam określić konkretne zachowanie przyczepiania. Spójrzmy na szybki przykład.

```swift
struct ContentView: View {
    var body: some View {
        ScrollView {
            ForEach(0..<100) { number in
                Text(verbatim: String(number))
                    .font(.largeTitle)
                    .frame(minWidth: 0, maxWidth: .infinity, minHeight: 300)
                    .background(Rectangle().fill(Color.green))
            }
        }
        .scrollTargetBehavior(.paging)
    }
}
```

W tym przykładzie używamy modyfikatora `scrollTargetBehavior` z opcją `.paging`, aby włączyć przewijanie po stronach. W tym przypadku ScrollView używa rozmiaru kontenera do obliczenia następnego widocznego celu.

Instancja `paging` typu `PagingScrollTargetBehavior` spełnia protokół `ScrollTargetBehavior`. Innym typem spełniającym protokół `ScrollTargetBehavior` jest `ViewAlignedScrollTargetBehavior`.

```swift
struct ContentView: View {
    var body: some View {
        ScrollView {
            ForEach(0..<100) { number in
                Text(verbatim: String(number))
                    .font(.largeTitle)
                    .frame(minWidth: 0, maxWidth: .infinity, minHeight: 300)
                    .background(Rectangle().fill(Color.green))
                    .scrollTarget()
            }
        }
        .scrollTargetBehavior(.viewAligned)
    }
}
```

Jak widać w powyższym przykładzie, używamy modyfikatora `scrollTargetBehavior` z opcją `.viewAligned`, aby włączyć przyczepianie widoków. ScrollView automatycznie wyhamowuje po przewinięciu, aby wyrównać się z pierwszym widocznym elementem w widoku. ScrollView poszukuje widoków oznaczonych modyfikatorem `scrollTarget`, aby się do nich wyrównać.

Zazwyczaj definiuje się ScrollView z kontenerem układu wewnątrz, takim jak LazyVGrid lub LazyVStack. W tym przypadku należy użyć modyfikatora `scrollTargetLayout` na instancji LazyVGrid lub LazyVStack, aby umożliwić ScrollView ukierunkowanie widoków wewnątrz kontenera.

```swift
struct ExampleScrollView: View {
    var body: some View {
        ScrollView {
            LazyVStack {
                ForEach(0..<100) { number in
                    Text(verbatim: String(number))
                        .font(.largeTitle)
                        .frame(minWidth: 0, maxWidth: .infinity, minHeight: 300)
                        .background(Rectangle().fill(Color.green))
                }
            }
            .scrollTargetLayout()
        }
        .scrollTargetBehavior(.viewAligned)
    }
}
```

Możesz również użyć modyfikatorów `scrollTarget` i `scrollTargetLayout

` w pojedynczym ScrollView, aby oznaczyć poszczególne widoki jako cele przewijania wraz z elementami w kontenerze układu.

```swift
struct AnotherExampleScrollView: View {
    var body: some View {
        ScrollView {
            CustomHeaderView()
                .scrollTarget()
            
            LazyVStack {
                // tu znajduje się zawartość
            }
            .scrollTargetLayout()
            
            CustomFooterView()
                .scrollTarget()
        }
        .scrollTargetBehavior(.viewAligned)
    }
}
```

Pamiętaj, że możesz użyć modyfikatora `scrollTargetBehavior` do ustawienia celu przewijania poziomego i pionowego. To zależy od tego, jakie osie są włączone w ScrollView.

```swift
struct ExampleScrollView: View {
    var body: some View {
        ScrollView(.horizontal) {
            LazyHStack {
                ForEach(0..<100) { number in
                    Text(verbatim: String(number))
                        .font(.largeTitle)
                        .frame(minWidth: 0, maxWidth: .infinity, minHeight: 300)
                        .background(Rectangle().fill(Color.green))
                }
            }
            .scrollTargetLayout()
        }
        .scrollTargetBehavior(.viewAligned)
    }
}
```

SwiftUI udostępnia dwie opcje dla zachowania celu przewijania: `viewAligned` i `paging`. Ale można również stworzyć własne typy, spełniając protokół `ScrollTargetBehavior`.

```swift
struct CustomScrollTargetBehavior: ScrollTargetBehavior {
    func updateTarget(_ target: inout ScrollTarget, context: TargetContext) {
        if context.velocity.dy > 0 {
            target.rect.origin.y = context.originalTarget.rect.maxY
        } else if context.velocity.dy < 0 {
            target.rect.origin.y = context.originalTarget.rect.minY
        }
    }
}

extension ScrollTargetBehavior where Self == CustomScrollTargetBehavior {
    static var custom: CustomScrollTargetBehavior { .init() }
}

struct ContentView: View {
    var body: some View {
        ScrollView {
            ForEach(0..<100) { number in
                Text(verbatim: String(number))
                    .font(.largeTitle)
                    .frame(minWidth: 0, maxWidth: .infinity, minHeight: 300)
                    .background(Rectangle().fill(Color.green))
            }
        }
        .scrollTargetBehavior(.custom)
    }
}
```

Jak widać w powyższym przykładzie, definiujemy typ `CustomScrollTargetBehavior`, który spełnia protokół `ScrollTargetBehavior`. Musi on zaimplementować jedyną wymaganą funkcję o nazwie `updateTarget`. Funkcja `updateTarget` posiada dwa parametry: `target` (typu `inout ScrollTarget`) i `context` (typu `TargetContext`).

Parametr `target` to zmienna referencyjna typu `ScrollTarget`, która pozwala nam ustawić prostokąt i punkt kotwiczenia dla naszego celu wewnątrz ScrollView.

Parametr `context` to instancja typu `ScrollTargetBehaviorContext`, która udostępnia informacje o kontekście, w którym aktualizowany jest cel przewijania. Zapewnia dostęp do włączonych osi, prędkości, rozmiaru kontenera, rozmiaru zawartości, oryginalnego cel

u itp.
