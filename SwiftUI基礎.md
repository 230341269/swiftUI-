以下是使用 SwiftUI 編寫的一個完整的 `ContentView.swift` 文件，展示了 SwiftUI 的基本用法，包括所需的所有元素（導航欄、按鈕、文字、圖片、布局、Sheet、路由、列表、編輯、表單、對象數組保存、`@State` 和 `@Binding`）。我將逐步講解代碼構成，最後提供完整代碼。

---

### 第一步：定義數據模型

首先，我們需要定義一個數據模型來表示我們的對象數組，並遵循 `Identifiable` 協議，這樣可以用於 SwiftUI 的列表。

```swift
import SwiftUI

// 定義一個數據模型，用於表示列表中的項目
struct Item: Identifiable {
    let id = UUID() // 唯一標識符
    var name: String // 項目的名稱
    var description: String // 項目的描述
}
```

---

### 第二步：創建主視圖 `ContentView`

主視圖中包括一個導航欄、列表、按鈕，並使用 `@State` 保存數據。還添加了一個 `Sheet`，用於彈出表單來添加新對象。

```swift
// 主視圖
struct ContentView: View {
    @State private var items: [Item] = [] // 保存對象數組
    @State private var isShowingSheet = false // 是否顯示表單
    @State private var selectedItem: Item? // 用於路由到詳細視圖

    var body: some View {
        NavigationView {
            VStack {
                // 列表顯示對象數組
                List {
                    ForEach(items) { item in
                        NavigationLink(destination: DetailView(item: item)) {
                            HStack {
                                Text(item.name)
                                    .font(.headline)
                                Spacer()
                                Image(systemName: "chevron.right")
                                    .foregroundColor(.gray)
                            }
                        }
                    }
                    .onDelete(perform: deleteItem) // 支持刪除
                }
                .listStyle(InsetGroupedListStyle()) // 列表樣式
                
                // 添加新項目的按鈕
                Button(action: {
                    isShowingSheet = true
                }) {
                    Text("添加新項目")
                        .font(.headline)
                        .padding()
                        .frame(maxWidth: .infinity)
                        .background(Color.blue)
                        .foregroundColor(.white)
                        .cornerRadius(10)
                        .padding()
                }
            }
            .navigationTitle("項目列表") // 導航欄標題
            .toolbar {
                EditButton() // 編輯按鈕
            }
            .sheet(isPresented: $isShowingSheet) {
                AddItemView(items: $items) // 表單用於添加新項目
            }
        }
    }

    // 刪除項目
    private func deleteItem(at offsets: IndexSet) {
        items.remove(atOffsets: offsets)
    }
}
```

---

### 第三步：詳細視圖

詳細視圖顯示選中的項目詳細信息。

```swift
// 詳細視圖
struct DetailView: View {
    let item: Item // 接收選中的項目

    var body: some View {
        VStack(spacing: 20) {
            Text(item.name)
                .font(.largeTitle)
                .fontWeight(.bold)
            Text(item.description)
                .font(.body)
                .foregroundColor(.gray)
            Spacer()
        }
        .padding()
        .navigationTitle("詳細信息") // 導航欄標題
    }
}
```

---

### 第四步：添加項目表單

表單用於添加新項目，使用 `@Binding` 將新項目添加到主視圖的數組中。

```swift
// 添加項目表單
struct AddItemView: View {
    @Binding var items: [Item] // 綁定到主視圖的數據
    @Environment(\.dismiss) var dismiss // 用於關閉表單

    @State private var name = "" // 表單中的名稱
    @State private var description = "" // 表單中的描述

    var body: some View {
        NavigationView {
            Form {
                Section(header: Text("名稱")) {
                    TextField("輸入名稱", text: $name)
                }
                Section(header: Text("描述")) {
                    TextField("輸入描述", text: $description)
                }
            }
            .navigationTitle("添加新項目") // 導航欄標題
            .toolbar {
                ToolbarItem(placement: .cancellationAction) {
                    Button("取消") {
                        dismiss() // 關閉表單
                    }
                }
                ToolbarItem(placement: .confirmationAction) {
                    Button("保存") {
                        addItem() // 保存新項目
                        dismiss() // 關閉表單
                    }
                }
            }
        }
    }

    // 添加新項目
    private func addItem() {
        let newItem = Item(name: name, description: description) // 創建新項目
        items.append(newItem) // 添加到數組
    }
}
```

---

### 第五步：完整代碼

以下是完整的代碼，包括數據模型、主視圖、詳細視圖和表單。

```swift
import SwiftUI

// 定義數據模型
struct Item: Identifiable {
    let id = UUID() // 唯一標識符
    var name: String // 項目的名稱
    var description: String // 項目的描述
}

// 主視圖
struct ContentView: View {
    @State private var items: [Item] = [] // 保存對象數組
    @State private var isShowingSheet = false // 是否顯示表單
    @State private var selectedItem: Item? // 用於路由到詳細視圖

    var body: some View {
        NavigationView {
            VStack {
                // 列表顯示對象數組
                List {
                    ForEach(items) { item in
                        NavigationLink(destination: DetailView(item: item)) {
                            HStack {
                                Text(item.name)
                                    .font(.headline)
                                Spacer()
                                Image(systemName: "chevron.right")
                                    .foregroundColor(.gray)
                            }
                        }
                    }
                    .onDelete(perform: deleteItem) // 支持刪除
                }
                .listStyle(InsetGroupedListStyle()) // 列表樣式
                
                // 添加新項目的按鈕
                Button(action: {
                    isShowingSheet = true
                }) {
                    Text("添加新項目")
                        .font(.headline)
                        .padding()
                        .frame(maxWidth: .infinity)
                        .background(Color.blue)
                        .foregroundColor(.white)
                        .cornerRadius(10)
                        .padding()
                }
            }
            .navigationTitle("項目列表") // 導航欄標題
            .toolbar {
                EditButton() // 編輯按鈕
            }
            .sheet(isPresented: $isShowingSheet) {
                AddItemView(items: $items) // 表單用於添加新項目
            }
        }
    }

    // 刪除項目
    private func deleteItem(at offsets: IndexSet) {
        items.remove(atOffsets: offsets)
    }
}

// 詳細視圖
struct DetailView: View {
    let item: Item // 接收選中的項目

    var body: some View {
        VStack(spacing: 20) {
            Text(item.name)
                .font(.largeTitle)
                .fontWeight(.bold)
            Text(item.description)
                .font(.body)
                .foregroundColor(.gray)
            Spacer()
        }
        .padding()
        .navigationTitle("詳細信息") // 導航欄標題
    }
}

// 添加項目表單
struct AddItemView: View {
    @Binding var items: [Item] // 綁定到主視圖的數據
    @Environment(\.dismiss) var dismiss // 用於關閉表單

    @State private var name = "" // 表單中的名稱
    @State private var description = "" // 表單中的描述

    var body: some View {
        NavigationView {
            Form {
                Section(header: Text("名稱")) {
                    TextField("輸入名稱", text: $name)
                }
                Section(header: Text("描述")) {
                    TextField("輸入描述", text: $description)
                }
            }
            .navigationTitle("添加新項目") // 導航欄標題
            .toolbar {
                ToolbarItem(placement: .cancellationAction) {
                    Button("取消") {
                        dismiss() // 關閉表單
                    }
                }
                ToolbarItem(placement: .confirmationAction) {
                    Button("保存") {
                        addItem() // 保存新項目
                        dismiss() // 關閉表單
                    }
                }
            }
        }
    }

    // 添加新項目
    private func addItem() {
        let newItem = Item(name: name, description: description) // 創建新項目
        items.append(newItem) // 添加到數組
    }
}

// 預覽
struct ContentView_Previews: PreviewProvider {
    static var previews: some View {
        ContentView()
    }
}
```

---

### 功能總結

1. **導航欄**：使用 `NavigationView` 和 `NavigationLink`。
2. **按鈕**：添加新項目和編輯按鈕。
3. **文字**：顯示項目名稱和描述。
4. **圖片**：使用 SF Symbols 在列表中顯示箭頭。
5. **布局**：使用 `VStack` 和 `HStack`。
6. **Sheet**：彈出表單以添加新項目。
7. **路由**：使用 `NavigationLink` 跳轉到詳細視圖。
8. **列表**：顯示項目數組。
9. **編輯**：支持刪除項目。
10. **表單**：用於輸入新項目的名稱和描述。
11. **對象數組保存**：使用 `@State` 保存數據。
12. **State 和 Binding**：實現雙向數據綁定。





