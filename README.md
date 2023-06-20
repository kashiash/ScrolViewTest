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

scrollTargetLayout

Ten modyfikator jest używany w połączeniu z scrollTargetBehavior(mode: ViewAlignedScrollTargetBehavior) lub scrollPosition(id:), które zostaną omówione poniżej.

Ten modyfikator powinien być zastosowany do kontenerów układu w ScrollView, które zawierają główną powtarzającą się zawartość, taką jak LazyHStack lub VStack.

```swift
@State private var isEnabled = true

ScrollView {
    LazyVStack {
        ForEach(items) { item in
            CellView(width: 200, height: 140)
                .idView(item.n)
        }
    }
    .scrollTargetLayout(isEnabled: isEnabled)
}
```

scrollPosition(initialAnchor:)

Używając tego modyfikatora, możesz określić punkt kotwiczenia początkowo widocznej części zawartości ScrollView. Ma to wpływ tylko na początkowy stan ScrollView i jest ustawiane raz. Jest zwykle stosowany do scenariuszy takich jak pokazywanie danych od dołu w aplikacjach IM lub wyświetlanie danych od końca. Możesz ustawić początkową pozycję dla obu osi jednocześnie za pomocą UnitPoint.

```swift
struct ScrollPositionInitialAnchorDemo: View {
    @State private var show = false
    @State private var position: Position = .leading
    var body: some View {
        VStack {
            Toggle("Pokaż", isOn: $show)
            Picker("Pozycja", selection: $position) {
                ForEach(Position.allCases) { p in
                    Text(p.rawValue).tag(p)
                }
            }
            .pickerStyle(.segmented)
            if show {
                ScrollView(.horizontal) {
                    LazyHStack {
                        ForEach(0 ..< 10000) { i in
                            CellView(debugInfo: "\(i)")
                                .idView(i)
                        }
                    }
                }
                .scrollPosition(initialAnchor: position.unitPoint)
            }
        }
        .padding()
    }

    enum Position: String, Identifiable, CaseIterable {
        var id: UnitPoint { unitPoint }
        case leading, center, trailing
        var unitPoint: UnitPoint {
            switch self {
            case .leading:
                .leading
            case .center:
                .center
            case .trailing:
                .trailing
            }
        }
    }
}
```

## scrollPosition(id:)

Używając tego modyfikatora, ScrollView może przewinąć się do określonej pozycji. Można go uznać za uproszczoną wersję ScrollViewReader.

    Dotyczy tylko ScrollView
    Gdy źródło danych w ForEach implementuje protokół Identifiable, nie ma potrzeby jawnego ustawiania identyfikatora za pomocą modyfikatora id.
    Gdy używany jest w połączeniu z scrollTargetLayout, można uzyskać aktualną pozycję przewijania (identyfikator widoku)
    Nie obsługuje ustawiania punktu kotwiczenia, a punkt kotwiczenia jest ustalony na środek podwidoku
    Jak wspomniano w artykule "Demystifying SwiftUI List Responsiveness: Best Practices for Large Datasets", mogą wystąpić problemy wydajnościowe, gdy zbiór danych jest duży.

```swift
struct ScrollPositionIDDemo: View {
    @State private var show = false
    @State private var position: Position = .trailing
    @State private var items = (0 ..< 500).map {
        Item(n: $0)
    }

    @State private var id: UUID?
    var body: some View {
        VStack {
            Picker("Position", selection: $position) {
                ForEach(Position.allCases) { p in
                    Text(p.rawValue).tag(p)
                }
            }
            .pickerStyle(.segmented)
            Text(id?.uuidString ?? "").fixedSize().font(.caption2)
            ScrollView(.horizontal) {
                LazyHStack {
                    ForEach(items) { item in
                        CellView(debugInfo: "\(item.n)")
                            .idView(item.n)
                    }
                }
            }
            .scrollPosition(id: $id)
            .scrollTargetLayout()
        }
        .animation(.default, value: id)
        .padding()
        .frame(height: 300)
        .task(id: position) {
            switch position {
            case .leading:
                id = items.first!.id
            case .center:
                id = items[250].id
            case .trailing:
                id = items.last!.id
            }
        }
    }
}
```

```
ScrollViewReader { proxy in
    ScrollView(.horizontal) {
        LazyHStack {
            ForEach(items) { item in
                CellView(debugInfo: "\(item.n)")
                    .idView(item.n)
                    .id(item.id)
            }
        }
    }
    .task(id: position) {
        switch position {
        case .leading:
            proxy.scrollTo(items.first!.id)
        case .center:
            proxy.scrollTo(items[250].id)
        case .trailing:
            proxy.scrollTo(items.last!.id)
        }
    }
}
```

ScrollViewReader jest wykorzystywany w celu odczytu pozycji i przewijania ScrollView. W tym przypadku tworzony jest ScrollView z układem HStack, zawierający widoki CellView dla każdego elementu w kolekcji "items". Każdy widok ma przypisaną unikalną wartość identyfikatora za pomocą metody idView(). Metoda task() jest wykorzystywana do zdefiniowania zadania, które zostanie wykonane po zmianie wartości zmiennej "position". W zależności od wartości "position", ScrollView zostanie przewinięte do odpowiedniego widoku na podstawie przypisanego identyfikatora.



```
scrollTargetBehavior

scrollTargetBehavior służy do ustawienia zachowania przewijania ScrollView: paginacji lub wyrównania do widoków podrzędnych.

Użycie .scrollTargetBehavior(.paging) powoduje, że ScrollView przewija się stronami, przewijając po jednej stronie na raz (czyli rozmiar widoczny ScrollView).

LazyVStack {
    ForEach(items) { item in
        CellView(width: 200, height: 140)
            .idView(item.n)
    }
}
.scrollTargetBehavior(.paging)
```

scrollTargetBehavior jest wykorzystywany do kontrolowania zachowania przewijania ScrollView. W powyższym przykładzie tworzony jest LazyVStack zawierający widoki CellView dla każdego elementu w kolekcji "items". Używając .scrollTargetBehavior(.paging), ScrollView przewija się stronami, przewijając po jednej stronie na raz, gdzie rozmiar strony odpowiada widocznej części ScrollView.


Gdy ustawione jest `.scrollTargetBehavior(.viewAligned)`, należy użyć również `scrollTargetLayout` w połączeniu z tym. Po zatrzymaniu przewijania, górna część kontenera zostanie wyrównana z górną częścią widoku podrzędnego (w trybie pionowym). Deweloperzy mogą kontrolować włączanie i wyłączanie `scrollTargetLayout`, aby przełączać zachowanie `viewAligned`.

```swift
struct ScrollTargetBehaviorDemo: View {
    @State var items = (0 ..< 100).map { Item(n: $0) }
    @State private var isEnabled = true
    var body: some View {
        VStack {
            Toggle("Layout enable", isOn: $isEnabled).padding()
            ScrollView {
                LazyVStack {
                    ForEach(items) { item in
                        CellView(width: 200, height: 95)
                            .idView(item.n)
                    }
                }
                .scrollTargetLayout(isEnabled: isEnabled)
            }
            .border(.red, width: 2)
        }
        .scrollTargetBehavior(.viewAligned)
        .frame(height: 300)
        .padding()
    }
}
```

W powyższym przykładzie tworzony jest ScrollView z LazyVStack zawierającym widoki CellView dla każdego elementu w kolekcji "items". Poprzez ustawienie `.scrollTargetBehavior(.viewAligned)` i użycie `scrollTargetLayout` w LazyVStack, po zatrzymaniu przewijania, górna część kontenera zostanie wyrównana z górną częścią widoku podrzędnego. Deweloperzy mogą kontrolować włączanie i wyłączanie `scrollTargetLayout` przy użyciu przełącznika Toggle.


With .scrollTargetBehavior(.viewAligned(limitBehavior:)), we can define the mechanism for aligning the scroll target behavior.

    .automatic is the default behavior, limited in compact horizontal size classes, otherwise unlimited.
    .always always limits the number of scrollable views.
    .never does not limit the number of scrollable views.

With ViewAlignedScrollTargetBehavior, developers can also override the scrolling position of the scroll view based on the system-provided target (implementation details have not been studied in detail yet).
NamedCoordinateSpace.scrollView

Apple introduced the NamedCoordinateSpace type in SwiftUI 5, which allows users to name coordinate systems and provides a preset .scrollView coordinate system (only supported by ScrollView). With this coordinate system, developers can easily obtain the positional relationship between subviews and the scroll view. Using this information, we can easily achieve many effects, especially when combined with another new API, the visualEffect modifier.

struct CoordinatorDemo: View {
    var body: some View {
        ScrollView {
            ForEach(0 ..< 30) { _ in
                CellView()
                    .overlay(
                        GeometryReader { proxy in
                            if let distanceFromTop = proxy.bounds(of: .scrollView)?.minY {
                                Text(distanceFromTop * -1, format: .number)
                            }
                        }
                    )
            }
        }
        .border(.blue)
        .contentMargins(30, for: .scrollContent)
    }
}

Unlike the coordinate system set with .coordinateSpace(.named("MyScrollView")), the default .scrollViewcoordinate system can correctly handle margins created with contentMargins.

ScrollView {
    ForEach(0 ..< 30) { _ in
        CellView()
            .overlay(
                GeometryReader { proxy in
                    if let distanceFromTop = proxy.bounds(of: .named("MyScrollView"))?.minY {
                        Text(distanceFromTop * -1, format: .number)
                    }
                }
            )
    }
}
.border(.blue)
.contentMargins(30, for: .scrollContent)
// margin not recognized
.coordinateSpace(.named("MyScrollView"))



`bounds(of coordinateSpace: NamedCoordinateSpace) -> CGRect?` to nowo dodane API w tym roku, służące do pobierania prostokąta granicznego określonej przestrzeni współrzędnych.
scrollTransition

Faktycznie, w wielu scenariuszach nie potrzebujemy bardzo precyzyjnych relacji pozycji za pomocą NamedCoordinateSpace.scrollView. Apple udostępnia nam inne API, które upraszczają ten proces.

Kiedy podwidok przesuwa się do i z obszaru widocznego przewijanego kontenera, scrollTransition stosuje podaną animację przejścia do widoku i płynnie przechodzi między różnymi etapami.

Obecnie zdefiniowane są trzy stany fazowe (Phase):

    topLeading: Widok przesuwa się do obszaru widocznego kontenera przewijanego
    identity: Oznacza, że widok jest obecnie w obszarze widocznym
    bottomTrailing: Widok przesuwa się poza obszar widocznego kontenera przewijanego

Zamykające przekształcenie (transition closure) scrollTransition wymaga zwrócenia typu, który jest zgodny z protokołem VisualEffect (protokół VisualEffect definiuje typ efektu, który nie wpływa na układ widoku, a Apple umożliwia wiele modyfikatorów zgodnych z tym protokołem).

```swift
struct ScrollTransitionDemo: View {
    @State var clip = false
    var body: some View {
        ZStack(alignment: .bottom) {
            ScrollView {
                ForEach(0 ..< 30) { i in
                    CellView()
                        .idView(i)
                        .scrollTransition(.animated) { content, phase in
                            content
                                .scaleEffect(phase != .identity ? 0.6 : 1)
                                .opacity(phase != .identity ? 0.3 : 1)
                        }
                }
            }
            .frame(height: 300)
            .scrollClipDisabled(clip)
            Toggle("Clip", isOn: $clip)
                .padding(16)
        }
    }
}
```

W powyższym przykładzie tworzony jest ScrollView zawierający wiele widoków CellView. Używając `.scrollTransition(.animated)` i dostarczając zamknięcie (transition closure), można płynnie animować widoki podczas ich przesuwania wewnątrz obszaru widocznego ScrollView. Zamknięcie otrzymuje widok zawierający i fazę (Phase) przesunięcia widoku i zwraca modyfikowany widok na podstawie fazy. Dodatkowo, można użyć `.scrollClipDisabled(clip)` do włączania lub wyłączania przycinania (clipping) widoku ScrollView.

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
