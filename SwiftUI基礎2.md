以下是使用 SwiftUI 編寫的一個完整 `ContentView.swift` 示例，展示了 SwiftUI 的常用功能，包括 `NavigationView`（導航欄）、`Button`（按鈕）、`Text`（文本）、`Image`（圖像）、`布局`、`Sheet`（彈出視圖）、路由、`List`（列表）、`Edit`（編輯）、`Form`（表單）、`State` 和 `@Binding`，以及數據的簡單持久化（使用 `UserDefaults`）。

以下分成逐步構建和完整代碼兩部分。

---

### **逐步構建代碼**

#### 1. 基本的 SwiftUI 視圖結構

```swift
import SwiftUI

struct ContentView: View {
    var body: some View {
        NavigationView {
            VStack {
                Text("歡迎來到 SwiftUI 示例")
                    .font(.largeTitle)
                    .padding()
            }
            .navigationTitle("主頁")
        }
    }
}

struct ContentView_Previews: PreviewProvider {
    static var previews: some View {
        ContentView()
    }
}
```

- **說明**：
  - 使用 `NavigationView` 包裹主視圖，提供導航欄功能。
  - `Text` 顯示一段文字，並應用字體和間距。

---

#### 2. 添加按鈕和導航到新頁面

```swift
struct ContentView: View {
    var body: some View {
        NavigationView {
            VStack(spacing: 20) {
                Text("歡迎來到 SwiftUI 示例")
                    .font(.largeTitle)
                    .padding()
                
                // 按鈕導航到詳細頁
                NavigationLink(destination: DetailView()) {
                    Text("點擊查看詳細頁")
                        .font(.headline)
                        .foregroundColor(.white)
                        .padding()
                        .background(Color.blue)
                        .cornerRadius(8)
                }
            }
            .navigationTitle("主頁")
        }
    }
}

struct DetailView: View {
    var body: some View {
        VStack {
            Text("這是詳細頁")
                .font(.title)
        }
        .navigationTitle("詳細頁")
    }
}
```

- **說明**：
  - 使用 `NavigationLink` 實現頁面跳轉。
  - 添加新的視圖 `DetailView` 作為目標頁面。

---

#### 3. 添加圖片和列表

```swift
struct ContentView: View {
    var body: some View {
        NavigationView {
            VStack(spacing: 20) {
                Image(systemName: "star.fill") // 使用 SF Symbols 圖標
                    .resizable()
                    .frame(width: 100, height: 100)
                    .foregroundColor(.yellow)
                
                NavigationLink(destination: ListView()) {
                    Text("點擊查看列表")
                        .font(.headline)
                        .foregroundColor(.white)
                        .padding()
                        .background(Color.green)
                        .cornerRadius(8)
                }
            }
            .navigationTitle("主頁")
        }
    }
}

struct ListView: View {
    let items = ["項目 1", "項目 2", "項目 3"] // 示例數據
    
    var body: some View {
        List(items, id: \.self) { item in
            Text(item)
        }
        .navigationTitle("列表頁")
    }
}
```

- **說明**：
  - 使用 `Image` 顯示圖片，並設置大小和顏色。
  - 使用 `List` 顯示簡單的數據列表。

---

#### 4. 添加表單和編輯功能

```swift
struct ContentView: View {
    @State private var showSheet = false // 控制 Sheet 彈出的狀態
    
    var body: some View {
        NavigationView {
            VStack(spacing: 20) {
                Button("打開表單頁") {
                    showSheet = true
                }
                .font(.headline)
                .foregroundColor(.white)
                .padding()
                .background(Color.orange)
                .cornerRadius(8)
                .sheet(isPresented: $showSheet) {
                    FormView()
                }
            }
            .navigationTitle("主頁")
        }
    }
}

struct FormView: View {
    @State private var name = "" // 用於表單輸入的狀態
    @State private var isEditing = false
    
    var body: some View {
        NavigationView {
            Form {
                Section(header: Text("個人信息")) {
                    TextField("輸入姓名", text: $name) // 綁定輸入框
                    Toggle("是否在編輯", isOn: $isEditing) // 切換開關
                }
            }
            .navigationTitle("表單")
            .toolbar {
                Button("保存") {
                    print("保存數據：\(name), 編輯狀態：\(isEditing)")
                }
            }
        }
    }
}
```

- **說明**：
  - 使用 `Form` 創建表單。
  - 使用 `TextField` 和 `Toggle` 接收用戶輸入，並使用 `@State` 進行狀態綁定。
  - 使用 `Sheet` 彈出表單頁。

---

#### 5. 添加數據持久化功能

```swift
struct ContentView: View {
    @AppStorage("name") private var name = "訪客" // 使用 AppStorage 保存數據
    
    var body: some View {
        NavigationView {
            VStack(spacing: 20) {
                Text("歡迎, \(name)")
                    .font(.title)
                
                NavigationLink(destination: EditNameView()) {
                    Text("修改名稱")
                        .font(.headline)
                        .foregroundColor(.white)
                        .padding()
                        .background(Color.purple)
                        .cornerRadius(8)
                }
            }
            .navigationTitle("主頁")
        }
    }
}

struct EditNameView: View {
    @AppStorage("name") private var name = "" // 與主頁共享數據
    @State private var newName = "" // 本地臨時數據
    
    var body: some View {
        VStack {
            TextField("輸入新名稱", text: $newName)
                .textFieldStyle(RoundedBorderTextFieldStyle())
                .padding()
            
            Button("保存") {
                name = newName // 保存到 AppStorage
            }
            .font(.headline)
            .foregroundColor(.white)
            .padding()
            .background(Color.blue)
            .cornerRadius(8)
        }
        .padding()
        .navigationTitle("修改名稱")
    }
}
```

- **說明**：
  - 使用 `@AppStorage` 實現數據持久化，保存用戶名稱。

---

### **完整代碼**

以下是完整的 `ContentView.swift` 文件，整合所有功能：

```swift
import SwiftUI

struct ContentView: View {
    @AppStorage("name") private var name = "訪客" // 使用 AppStorage 保存數據
    @State private var showSheet = false // 控制 Sheet 彈出的狀態
    
    var body: some View {
        NavigationView {
            VStack(spacing: 20) {
                Text("歡迎, \(name)")
                    .font(.title)
                
                Image(systemName: "star.fill")
                    .resizable()
                    .frame(width: 100, height: 100)
                    .foregroundColor(.yellow)
                
                NavigationLink(destination: ListView()) {
                    Text("查看列表")
                        .font(.headline)
                        .foregroundColor(.white)
                        .padding()
                        .background(Color.green)
                        .cornerRadius(8)
                }
                
                Button("打開表單頁") {
                    showSheet = true
                }
                .font(.headline)
                .foregroundColor(.white)
                .padding()
                .background(Color.orange)
                .cornerRadius(8)
                .sheet(isPresented: $showSheet) {
                    FormView()
                }
                
                NavigationLink(destination: EditNameView()) {
                    Text("修改名稱")
                        .font(.headline)
                        .foregroundColor(.white)
                        .padding()
                        .background(Color.purple)
                        .cornerRadius(8)
                }
            }
            .navigationTitle("主頁")
        }
    }
}

struct ListView: View {
    let items = ["項目 1", "項目 2", "項目 3"]
    
    var body: some View {
        List(items, id: \.self) { item in
            Text(item)
        }
        .navigationTitle("列表頁")
    }
}

struct FormView: View {
    @State private var name = ""
    @State private var isEditing = false
    
    var body: some View {
        NavigationView {
            Form {
                Section(header: Text("個人信息")) {
                    TextField("輸入姓名", text: $name)
                    Toggle("是否在編輯", isOn: $isEditing)
                }
            }
            .navigationTitle("表單")
            .toolbar {
                Button("保存") {
                    print("保存數據：\(name), 編輯狀態：\(isEditing)")
                }
            }
        }
    }
}

struct EditNameView: View {
    @AppStorage("name") private var name = ""
    @State private var newName = ""
    
    var body: some View {
        VStack {
            TextField("輸入新名稱", text: $newName)
                .textFieldStyle(RoundedBorderTextFieldStyle())
                .padding()
            
            Button("保存") {
                name = newName
            }
            .font(.headline)
            .foregroundColor(.white)
            .padding()
            .background(Color.blue)
            .cornerRadius(8)
        }
        .padding()
        .navigationTitle("修改名稱")
    }
}

struct ContentView_Previews: PreviewProvider {
    static var previews: some View {
        ContentView()
    }
}
```

---

### **總結**

這段代碼展示了 SwiftUI 的多個功能，包括導航、按鈕、列表、表單、彈出視圖、數據持久化和狀態管理。通過這些示例代碼，你可以深入了解 SwiftUI 的基本用法並進一步擴展你的應用功能。

