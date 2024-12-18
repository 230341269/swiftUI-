在 SwiftUI 中，可以使用 **`UserDefaults`** 或 **`CoreData`** 等方式來實現數據的持久化存儲。為了簡化實現，我將以 **`UserDefaults`** 和 **`Codable`** 的方式來修改代碼，實現數據的保存和加載。

---

### 修改步驟

#### 1. **讓數據模型支持 Codable**

我們需要讓數據模型遵循 `Codable` 協議，這樣它可以被編碼為 JSON 格式存儲在 `UserDefaults` 中，或者從中解碼。

```swift
struct Item: Identifiable, Codable {
    let id: UUID // 唯一標識符
    var name: String // 項目的名稱
    var description: String // 項目的描述
}
```

---

#### 2. **封裝數據的讀取和保存邏輯**

添加一個專門的 `PersistenceManager` 類，用來處理數據的讀取和保存操作。我們將數據存儲在 `UserDefaults` 中，存儲的數據會以 JSON 格式保存。

```swift
class PersistenceManager {
    static let shared = PersistenceManager()
    private let key = "items_key" // 存儲數據的鍵值

    // 保存數據到 UserDefaults
    func saveItems(_ items: [Item]) {
        do {
            let data = try JSONEncoder().encode(items) // 將數組編碼為 JSON
            UserDefaults.standard.set(data, forKey: key) // 存儲 JSON
        } catch {
            print("保存數據失敗: \(error)")
        }
    }

    // 從 UserDefaults 加載數據
    func loadItems() -> [Item] {
        guard let data = UserDefaults.standard.data(forKey: key) else {
            return [] // 如果沒有數據，返回空數組
        }
        do {
            return try JSONDecoder().decode([Item].self, from: data) // 解碼 JSON
        } catch {
            print("加載數據失敗: \(error)")
            return []
        }
    }
}
```

---

#### 3. **修改主視圖以支持數據持久化**

我們需要在主視圖中，將數據的加載與保存集成到 `@State` 數組中。

- **在應用啟動時加載數據**
- **在數據變更時保存數據**

```swift
struct ContentView: View {
    @State private var items: [Item] = [] // 保存對象數組
    @State private var isShowingSheet = false // 是否顯示表單

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
            .onAppear {
                // 加載數據
                items = PersistenceManager.shared.loadItems()
            }
        }
    }

    // 刪除項目
    private func deleteItem(at offsets: IndexSet) {
        items.remove(atOffsets: offsets)
        PersistenceManager.shared.saveItems(items) // 保存數據
    }
}
```

---

#### 4. **在添加新項目時保存數據**

當用戶添加新項目時，除了更新數組，也需要保存數據到磁盤。

```swift
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
        PersistenceManager.shared.saveItems(items) // 保存數據
    }
}
```

---

#### 5. **完整代碼**

以下是實現數據持久化的完整代碼：

```swift
import SwiftUI

// 數據模型
struct Item: Identifiable, Codable {
    let id = UUID() // 唯一標識符
    var name: String // 項目的名稱
    var description: String // 項目的描述
}

// 持久化管理器
class PersistenceManager {
    static let shared = PersistenceManager()
    private let key = "items_key" // 存儲數據的鍵值

    // 保存數據到 UserDefaults
    func saveItems(_ items: [Item]) {
        do {
            let data = try JSONEncoder().encode(items) // 將數組編碼為 JSON
            UserDefaults.standard.set(data, forKey: key) // 存儲 JSON
        } catch {
            print("保存數據失敗: \(error)")
        }
    }

    // 從 UserDefaults 加載數據
    func loadItems() -> [Item] {
        guard let data = UserDefaults.standard.data(forKey: key) else {
            return [] // 如果沒有數據，返回空數組
        }
        do {
            return try JSONDecoder().decode([Item].self, from: data) // 解碼 JSON
        } catch {
            print("加載數據失敗: \(error)")
            return []
        }
    }
}

// 主視圖
struct ContentView: View {
    @State private var items: [Item] = [] // 保存對象數組
    @State private var isShowingSheet = false // 是否顯示表單

    var body: some View {
        NavigationView {
            VStack {
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
                .listStyle(InsetGroupedListStyle())
                
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
            .navigationTitle("項目列表")
            .toolbar {
                EditButton()
            }
            .sheet(isPresented: $isShowingSheet) {
                AddItemView(items: $items)
            }
            .onAppear {
                items = PersistenceManager.shared.loadItems() // 加載數據
            }
        }
    }

    private func deleteItem(at offsets: IndexSet) {
        items.remove(atOffsets: offsets)
        PersistenceManager.shared.saveItems(items) // 保存數據
    }
}

// 詳細視圖
struct DetailView: View {
    let item: Item

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
        .navigationTitle("詳細信息")
    }
}

// 添加項目表單
struct AddItemView: View {
    @Binding var items: [Item]
    @Environment(\.dismiss) var dismiss

    @State private var name = ""
    @State private var description = ""

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
            .navigationTitle("添加新項目")
            .toolbar {
                ToolbarItem(placement: .cancellationAction) {
                    Button("取消") {
                        dismiss()
                    }
                }
                ToolbarItem(placement: .confirmationAction) {
                    Button("保存") {
                        addItem()
                        dismiss()
                    }
                }
            }
        }
    }

    private func addItem() {
        let newItem = Item(name: name, description: description)
        items.append(newItem)
        PersistenceManager.shared.saveItems(items) // 保存數據
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

- **數據持久化**：數據會自動保存到 `UserDefaults`，應用重啟後會重新加載。
- **數據更新**：在添加或刪除項目時，數據會自動保存。
- **簡單實現**：使用 `UserDefaults` 和 `Codable`，適合小型數據的存儲場景。

如果需要處理更大數據集或更高級的數據操作，可以考慮使用 **CoreData** 或 **Realm**。

