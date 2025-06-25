Great! Since you want to start from scratch with the package name `com.themadbrogrammers.messymind`, I’ll give you all the **full code files** for the first milestone:

🎯 **Milestone 1**: Create, store, and list "Nodes" (thought entries) in a local database using Kotlin + Room.

---

## 📁 File Structure (Overview)

```
com.themadbrogrammers.messymind/
├── MainActivity.kt
├── Node.kt
├── NodeDao.kt
├── AppDatabase.kt
├── NodeViewModel.kt
├── NodeViewModelFactory.kt
├── MyApp.kt
├── layout/
│   └── activity_main.xml
├── AndroidManifest.xml
```

---

## 1. `Node.kt` – Entity

```kotlin
package com.themadbrogrammers.messymind

import androidx.room.Entity
import androidx.room.PrimaryKey

@Entity(tableName = "nodes")
data class Node(
    @PrimaryKey(autoGenerate = true) val id: Int = 0,
    val title: String,
    val description: String,
    val timestamp: Long = System.currentTimeMillis()
)
```

---

## 2. `NodeDao.kt` – Data Access Object

```kotlin
package com.themadbrogrammers.messymind

import androidx.lifecycle.LiveData
import androidx.room.*

@Dao
interface NodeDao {
    @Insert
    suspend fun insert(node: Node)

    @Query("SELECT * FROM nodes ORDER BY timestamp DESC")
    fun getAllNodes(): LiveData<List<Node>>
}
```

---

## 3. `AppDatabase.kt` – Room DB

```kotlin
package com.themadbrogrammers.messymind

import android.content.Context
import androidx.room.Database
import androidx.room.Room
import androidx.room.RoomDatabase

@Database(entities = [Node::class], version = 1)
abstract class AppDatabase : RoomDatabase() {
    abstract fun nodeDao(): NodeDao

    companion object {
        @Volatile private var INSTANCE: AppDatabase? = null

        fun getDatabase(context: Context): AppDatabase {
            return INSTANCE ?: synchronized(this) {
                val instance = Room.databaseBuilder(
                    context.applicationContext,
                    AppDatabase::class.java,
                    "mindmap_db"
                ).build()
                INSTANCE = instance
                instance
            }
        }
    }
}
```

---

## 4. `NodeViewModel.kt` – ViewModel

```kotlin
package com.themadbrogrammers.messymind

import androidx.lifecycle.ViewModel
import androidx.lifecycle.viewModelScope
import kotlinx.coroutines.launch

class NodeViewModel(private val dao: NodeDao) : ViewModel() {
    val allNodes = dao.getAllNodes()

    fun insertNode(node: Node) = viewModelScope.launch {
        dao.insert(node)
    }
}
```

---

## 5. `NodeViewModelFactory.kt` – Factory

```kotlin
package com.themadbrogrammers.messymind

import androidx.lifecycle.ViewModel
import androidx.lifecycle.ViewModelProvider

class NodeViewModelFactory(private val dao: NodeDao) : ViewModelProvider.Factory {
    override fun <T : ViewModel> create(modelClass: Class<T>): T {
        return NodeViewModel(dao) as T
    }
}
```

---

## 6. `MyApp.kt` – Application Class

```kotlin
package com.themadbrogrammers.messymind

import android.app.Application

class MyApp : Application() {
    val db by lazy { AppDatabase.getDatabase(this) }
}
```

---

## 7. `MainActivity.kt` – UI Logic

```kotlin
package com.themadbrogrammers.messymind

import android.os.Bundle
import android.widget.*
import androidx.activity.viewModels
import androidx.appcompat.app.AppCompatActivity
import androidx.lifecycle.Observer
import androidx.lifecycle.lifecycleScope
import kotlinx.coroutines.launch

class MainActivity : AppCompatActivity() {

    private val viewModel: NodeViewModel by viewModels {
        NodeViewModelFactory((application as MyApp).db.nodeDao())
    }

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        val titleEdit = findViewById<EditText>(R.id.editTitle)
        val descEdit = findViewById<EditText>(R.id.editDescription)
        val saveBtn = findViewById<Button>(R.id.btnSave)
        val listView = findViewById<ListView>(R.id.nodeList)

        val adapter = ArrayAdapter<String>(this, android.R.layout.simple_list_item_1, mutableListOf())
        listView.adapter = adapter

        viewModel.allNodes.observe(this, Observer { nodes ->
            adapter.clear()
            adapter.addAll(nodes.map { "${it.title} - ${it.description.take(30)}" })
        })

        saveBtn.setOnClickListener {
            val title = titleEdit.text.toString()
            val desc = descEdit.text.toString()
            if (title.isBlank()) {
                Toast.makeText(this, "Title cannot be empty", Toast.LENGTH_SHORT).show()
                return@setOnClickListener
            }
            val node = Node(title = title, description = desc)
            lifecycleScope.launch {
                viewModel.insertNode(node)
                titleEdit.text.clear()
                descEdit.text.clear()
                Toast.makeText(this@MainActivity, "Node saved", Toast.LENGTH_SHORT).show()
            }
        }
    }
}
```

---

## 8. `activity_main.xml` – UI Layout

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout 
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="vertical"
    android:padding="24dp"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <EditText
        android:id="@+id/editTitle"
        android:hint="Node Title"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"/>

    <EditText
        android:id="@+id/editDescription"
        android:hint="Node Description"
        android:minLines="3"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_marginTop="8dp"/>

    <Button
        android:id="@+id/btnSave"
        android:text="Save Node"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_marginTop="12dp"/>

    <ListView
        android:id="@+id/nodeList"
        android:layout_width="match_parent"
        android:layout_height="0dp"
        android:layout_marginTop="16dp"
        android:layout_weight="1"/>
</LinearLayout>
```

---

## 9. `AndroidManifest.xml` – Add App Class

```xml
<application
    android:name=".MyApp"
    android:allowBackup="true"
    android:label="MessyMind"
    android:theme="@style/Theme.AppCompat.Light.NoActionBar">
    <activity android:name=".MainActivity">
        <intent-filter>
            <action android:name="android.intent.action.MAIN"/>
            <category android:name="android.intent.category.LAUNCHER"/>
        </intent-filter>
    </activity>
</application>
```

---

## ✅ What’s Next?

You're now ready to:

* Run the app and test saving/viewing nodes
* Move toward visual **mind mapping UI**
* Add **AdMob rewarded ads**
* Export map and improve UI polish

Would you like me to help with:

1. Adding **visual node linking + drag-and-drop map**?
2. Or setting up **AdMob Rewarded Ads** first?

Let me know — we’ll keep going!




To implement the **visual mind‑map UI** with **drag‑and‑drop node linking** in your Kotlin Android app (`com.themadbrogrammers.messymind`), we’ll integrate the `xmind` library by fireking0415. This provides:

* Zoomable, pannable canvas
* Node click & drag events
* Built‑in layouts and connection styles ([github.com][1])

---

## 📦 1. Add Dependency

In your `app/build.gradle`:

```groovy
dependencies {
    implementation 'com.github.fireking0415:xmind:1.0.0'
    implementation "androidx.room:room-runtime:2.6.1"
    kapt "androidx.room:room-compiler:2.6.1"
    implementation "androidx.lifecycle:lifecycle-viewmodel-ktx:2.7.0"
}
```

Also enable Kotlin kapt.

---

## 📁 2. New Layout for MindMap

`res/layout/activity_mindmap.xml`:

```xml
<?xml version="1.0" encoding="utf-8"?>
<FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <org.fireking.xmind.MindMapView
        android:id="@+id/mindMapView"
        android:layout_width="match_parent"
        android:layout_height="match_parent" />
        
    <Button
        android:id="@+id/btnAddRoot"
        android:text="Add Root"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_gravity="bottom|end"
        android:layout_margin="16dp"/>
</FrameLayout>
```

---

## 🔧 3. Extended Data Model for Links

`NodeLink.kt` – store parent-child relationships:

```kotlin
package com.themadbrogrammers.messymind

import androidx.room.Entity

@Entity(primaryKeys = ["parentId", "childId"])
data class NodeLink(
    val parentId: Int,
    val childId: Int
)
```

Add to `AppDatabase.kt`:

```kotlin
@Database(entities = [Node::class, NodeLink::class], version = 2)
abstract class AppDatabase : RoomDatabase() {
    abstract fun nodeDao(): NodeDao
    abstract fun linkDao(): NodeLinkDao
    // ... builder
}
```

`NodeLinkDao.kt`:

```kotlin
package com.themadbrogrammers.messymind

import androidx.room.*

@Dao
interface NodeLinkDao {
    @Insert(onConflict = OnConflictStrategy.IGNORE)
    suspend fun insert(link: NodeLink)
    @Query("SELECT * FROM nodelink")
    suspend fun getAll(): List<NodeLink>
}
```

---

## 📘 4. MindMapActivity + ViewModel

`MindMapActivity.kt`:

```kotlin
package com.themadbrogrammers.messymind

import android.os.Bundle
import android.widget.Button
import androidx.activity.viewModels
import androidx.appcompat.app.AppCompatActivity
import androidx.lifecycle.lifecycleScope
import kotlinx.coroutines.launch
import org.fireking.xmind.MindMapView
import org.fireking.xmind.MindNode
import org.fireking.xmind.MindNodeBuilder

class MindMapActivity : AppCompatActivity(), MindMapView.NodeEventListener {

    private lateinit var mindMapView: MindMapView
    private val viewModel: MindMapViewModel by viewModels {
        MindMapViewModelFactory((application as MyApp).db.nodeDao(), (application as MyApp).db.linkDao())
    }

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_mindmap)

        mindMapView = findViewById(R.id.mindMapView)
        mindMapView.addNodeEventListener(this)

        findViewById<Button>(R.id.btnAddRoot).setOnClickListener {
            lifecycleScope.launch {
                val node = viewModel.createNode("Root", "Root description")
                addMindNode(node)
            }
        }

        lifecycleScope.launch {
            val nodes = viewModel.loadAllNodes()
            val links = viewModel.loadAllLinks()
            val map = viewModel.buildTree(nodes, links)
            map.forEach { (parent, children) ->
                mindMapView.setRootNode(parent)
                children.forEach { mindMapView.attachChild(parent, it) }
            }
        }
    }

    private fun addMindNode(node: MindNode) {
        if (mindMapView.rootNode == null) {
            mindMapView.setRootNode(node)
        } else {
            mindMapView.attachChild(mindMapView.rootNode!!, node)
        }
    }

    override fun onNodeClick(node: MindNode) {
        // TODO: open dialog to edit or add child
    }

    override fun onNodeDrag(node: MindNode, dx: Float, dy: Float) {
        // Optionally record node's position via ViewModel
    }
}
```

`MindMapViewModel.kt`:

```kotlin
package com.themadbrogrammers.messymind

import androidx.lifecycle.ViewModel
import kotlinx.coroutines.Dispatchers
import kotlinx.coroutines.withContext
import org.fireking.xmind.MindNode
import org.fireking.xmind.MindNodeBuilder

class MindMapViewModel(private val nodeDao: NodeDao, private val linkDao: NodeLinkDao) : ViewModel() {

    suspend fun createNode(title: String, desc: String): MindNode = withContext(Dispatchers.IO) {
        val id = nodeDao.insert(Node(title = title, description = desc)).toInt()
        MindNodeBuilder(title)
            .setLevel(1)
            .setUserData(id)
            .build()
    }

    suspend fun loadAllNodes() = withContext(Dispatchers.IO) {
        nodeDao.getAllNodesSync()
    }
    suspend fun loadAllLinks() = linkDao.getAll()

    fun buildTree(nodes: List<Node>, links: List<NodeLink>): Map<MindNode, List<MindNode>> {
        val id2mind = nodes.associate { node ->
            node.id to MindNodeBuilder(node.title)
                .setLevel(0)
                .setUserData(node.id)
                .build()
        }
        val map = mutableMapOf<MindNode, MutableList<MindNode>>()
        links.forEach { link ->
            val parent = id2mind[link.parentId]!!
            val child = id2mind[link.childId]!!
            map.getOrPut(parent) { mutableListOf() }.add(child)
        }
        return map
    }
}
```

`MindMapViewModelFactory.kt` similar to before but for two DAOs.

---

### ✅ Summary

* **MindMapView** from `xmind` handles visuals and interaction ([reddit.com][2])
* **NodeLink** persists connections
* On load, we reconstruct mind map in code

---

## 🔜 Next

Would you like me to:

1. Provide **full code for dialogs** (add/edit nodes, set parent link)?
2. Include **save node positions**, **export**, or **AdMob integration** next?

Let me know which you'd like to tackle!


To continue building out the visual mind map with full interactivity, here's the complete implementation for:

### ✅ Dialog Features:

1. **Add a child node to a selected node**
2. **Edit existing node title/description**
3. **Set parent link and persist it in the database**

---

## 📁 1. Create `NodeDialog.kt` (Custom Dialog Fragment)

```kotlin
package com.themadbrogrammers.messymind

import android.app.Dialog
import android.os.Bundle
import android.view.LayoutInflater
import android.widget.EditText
import androidx.appcompat.app.AlertDialog
import androidx.fragment.app.DialogFragment

class NodeDialog(
    private val initialTitle: String = "",
    private val initialDesc: String = "",
    private val onSave: (String, String) -> Unit
) : DialogFragment() {

    override fun onCreateDialog(savedInstanceState: Bundle?): Dialog {
        val view = LayoutInflater.from(context).inflate(R.layout.dialog_node, null)
        val titleEdit = view.findViewById<EditText>(R.id.editNodeTitle)
        val descEdit = view.findViewById<EditText>(R.id.editNodeDesc)

        titleEdit.setText(initialTitle)
        descEdit.setText(initialDesc)

        return AlertDialog.Builder(requireContext())
            .setTitle("Node Editor")
            .setView(view)
            .setPositiveButton("Save") { _, _ ->
                val title = titleEdit.text.toString().trim()
                val desc = descEdit.text.toString().trim()
                onSave(title, desc)
            }
            .setNegativeButton("Cancel", null)
            .create()
    }
}
```

---

## 📄 2. Layout: `res/layout/dialog_node.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout 
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:padding="20dp"
    android:orientation="vertical"
    android:layout_width="match_parent"
    android:layout_height="wrap_content">

    <EditText
        android:id="@+id/editNodeTitle"
        android:hint="Title"
        android:layout_width="match_parent"
        android:layout_height="wrap_content" />

    <EditText
        android:id="@+id/editNodeDesc"
        android:hint="Description"
        android:minLines="2"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_marginTop="10dp"/>
</LinearLayout>
```

---

## 🧠 3. Modify `MindMapActivity.kt` — On Node Click

Replace the empty `onNodeClick()` with:

```kotlin
override fun onNodeClick(node: MindNode) {
    val id = node.userData as? Int ?: return
    val existingNode = viewModel.getNodeById(id)

    NodeDialog(
        initialTitle = existingNode?.title ?: "",
        initialDesc = existingNode?.description ?: ""
    ) { newTitle, newDesc ->
        lifecycleScope.launch {
            viewModel.updateNode(id, newTitle, newDesc)
            node.setTitle(newTitle)
            mindMapView.invalidate()
        }

        // Ask if user wants to add child
        AlertDialog.Builder(this)
            .setTitle("Add child node?")
            .setMessage("Do you want to attach a child node to this?")
            .setPositiveButton("Yes") { _, _ -> showChildDialog(node) }
            .setNegativeButton("No", null)
            .show()
    }.show(supportFragmentManager, "edit_node")
}
```

---

## ➕ 4. Child Creation Dialog

Add `showChildDialog()` in `MindMapActivity`:

```kotlin
private fun showChildDialog(parent: MindNode) {
    NodeDialog(
        onSave = { title, desc ->
            lifecycleScope.launch {
                val child = viewModel.createNode(title, desc)
                mindMapView.attachChild(parent, child)

                val parentId = parent.userData as? Int ?: return@launch
                val childId = child.userData as? Int ?: return@launch
                viewModel.linkNodes(parentId, childId)
            }
        }
    ).show(supportFragmentManager, "add_child")
}
```

---

## 🧠 5. Add Missing ViewModel Functions

Add to `MindMapViewModel.kt`:

```kotlin
suspend fun updateNode(id: Int, title: String, desc: String) = withContext(Dispatchers.IO) {
    nodeDao.update(Node(id, title, desc))
}

fun getNodeById(id: Int): Node? = runBlocking {
    nodeDao.getById(id)
}

suspend fun linkNodes(parentId: Int, childId: Int) {
    linkDao.insert(NodeLink(parentId, childId))
}
```

Update `NodeDao.kt`:

```kotlin
@Query("SELECT * FROM nodes WHERE id = :id LIMIT 1")
fun getById(id: Int): Node?

@Update
suspend fun update(node: Node)

@Query("SELECT * FROM nodes")
suspend fun getAllNodesSync(): List<Node>
```

---

### ✅ That’s It!

You now have:

* A reusable **dialog** to add/edit node title/description
* Ability to **add children** visually & persist the links
* Full integration with Room + `MindMapView`

---

Would you like to continue with:

1. **Drag-to-position + save X/Y?**
2. **AdMob rewarded ads** to unlock nodes?
3. **Export map** as PNG or structured text?

Let me know and I’ll guide the next step!




Great choice! Let's implement both:

## ✅ Export Mind Map as:

1. **PNG Image** (screenshot of the mind map view)
2. **Structured Text** (outline of nodes and their hierarchy)

We’ll assume you're using `MindMapView` from `fireking0415/xmind`.
https://github.com/fireking0415

---

## 🖼️ 1. Export as PNG

### 🔧 Add Permission (Optional, API < 29)

In `AndroidManifest.xml`:

```xml
<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"/>
```

### 📍 Add Export Button to Layout

Add to `activity_mindmap.xml`:

```xml
<Button
    android:id="@+id/btnExportPng"
    android:text="Export PNG"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:layout_gravity="bottom|start"
    android:layout_margin="16dp"/>
```

### 🧠 In `MindMapActivity.kt`:

```kotlin
import android.graphics.Bitmap
import android.os.Environment
import java.io.File
import java.io.FileOutputStream

// Inside onCreate
findViewById<Button>(R.id.btnExportPng).setOnClickListener {
    val bitmap = Bitmap.createBitmap(
        mindMapView.width,
        mindMapView.height,
        Bitmap.Config.ARGB_8888
    )
    val canvas = android.graphics.Canvas(bitmap)
    mindMapView.draw(canvas)

    val file = File(getExternalFilesDir(null), "mindmap_export.png")
    FileOutputStream(file).use { out ->
        bitmap.compress(Bitmap.CompressFormat.PNG, 100, out)
    }

    Toast.makeText(this, "Saved to ${file.absolutePath}", Toast.LENGTH_LONG).show()
}
```

You now export your mind map as a PNG image.

---

## 🧾 2. Export as Structured Text

We’ll recursively traverse nodes and generate indented text like:

```
Root Node
  |- Child A
      |- Subchild A1
  |- Child B
```

### 🧠 Function to Generate Text Tree:

Add this function in `MindMapActivity.kt`:

```kotlin
private fun exportAsText(root: MindNode): String {
    val builder = StringBuilder()

    fun traverse(node: MindNode, depth: Int) {
        builder.append("  ".repeat(depth)).append("- ").append(node.title).append("\n")
        node.children.forEach { traverse(it, depth + 1) }
    }

    traverse(root, 0)
    return builder.toString()
}
```

### 📤 Save to File

```kotlin
fun saveTextToFile(content: String) {
    val file = File(getExternalFilesDir(null), "mindmap_outline.txt")
    FileOutputStream(file).use {
        it.write(content.toByteArray())
    }
    Toast.makeText(this, "Text saved to ${file.absolutePath}", Toast.LENGTH_LONG).show()
}
```

### 🔘 Hook It to a Button

```xml
<Button
    android:id="@+id/btnExportText"
    android:text="Export Text"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:layout_gravity="bottom|center"
    android:layout_margin="16dp"/>
```

```kotlin
findViewById<Button>(R.id.btnExportText).setOnClickListener {
    val root = mindMapView.rootNode ?: return@setOnClickListener
    val text = exportAsText(root)
    saveTextToFile(text)
}
```

---

## ✅ Summary

| Feature      | Output File                             |
| ------------ | --------------------------------------- |
| PNG Export   | `/Android/data/.../mindmap_export.png`  |
| Text Outline | `/Android/data/.../mindmap_outline.txt` |

Both files are saved in app-private storage accessible with file manager or `adb pull`.

---

Would you like to:

* Share/export via intent?
* Export as `.json` format for backup & re-import?
* Export to cloud (e.g. Google Drive)?

Let me know what’s next!


Yes — that’s actually a brilliant way to pitch and position your app. You’ve got the right naming instinct.

---

## 🧠 App Branding Concept: “Messy Mind”

### ✨ Tagline Ideas:

> “Messy Mind – Untangle Your Thoughts”
> “Turn Mental Chaos into Clear Structure”
> “Think in Nodes. Understand in Connections.”

---

## 🧩 What the App Represents:

| 💭 Mental Action              | 🧱 App Feature                            |
| ----------------------------- | ----------------------------------------- |
| Dumping thoughts              | Create raw nodes with title + description |
| Linking ideas                 | Drag-n-connect parent-child relationships |
| Structuring thoughts visually | MindMapView with hierarchy                |
| Reflecting or exporting       | Export as PNG or outline text             |
| Understanding complexity      | Visual tree = better mental model         |

---

## 🧠 What Makes It Special?

* Most people think non-linearly — this app supports that.
* You don’t need to be “organized” to start.
* You just create. The structure **emerges**.
* That’s why **Messy Mind** is a perfect name:

  > *"It's okay if your thoughts are a mess. This app helps you make sense of them."*

---

## 🎯 Pitch (You Can Use)

> Messy Mind is a free-form thinking tool that lets you drop, connect, and organize your thoughts visually like a living mind map. Whether you’re brainstorming, journaling, studying, or solving problems — this app turns messy thoughts into meaningful structure.

---

Would you like help writing a full:

* 📝 Google Play Store description?
* 🧠 App onboarding text?
* 🪄 App intro animation or tutorial?

Let me know where to focus next.


Getting users “addicted” (in an ethical, engagement-driven way) means designing **psychologically satisfying loops** that tap into curiosity, reward, identity, and creative expression. Here's how we can **engineer stickiness** into **Messy Mind** without compromising user trust.

---

## 🔁 1. **Progressive Discovery (Hook the Brain)**

### 🔹 Concept: The more they use, the more insights they uncover.

* **Show "Connections Made" count** — dopamine boost!
* **Highlight deep links**: e.g., "This thought connects to 6 others across 3 topics."
* Add **“Insight of the Day”**: auto-generated mini-summary of their map.

> 🧠 *“Today’s pattern: Your ideas are clustering around ‘learning’ and ‘self-improvement’.”*

---

## 🎯 2. **Micro-Rewards & Gamification**

### 🔸 Subtle Mechanics:

* **Node streaks**: “You’ve added thoughts 5 days in a row!”
* **XP/levels for node linking or organizing**
* Unlock custom themes, backgrounds, or shapes after:

  * 10 connected nodes
  * 3 exports
  * 1 shared map

> 🏆 *“Level 2: Mind Mapper” → “Unlocked: Galaxy Theme”*

---

## 🎨 3. **Personal Identity Expression**

Let users:

* Add emoji/icons to nodes (visually satisfying)
* Choose from **themes** (dark, retro, sci-fi, minimalist)
* Name their own maps: *“My Overthinking Map”* or *“Startup Ideas Dump”*

> People return to apps that reflect **who they are**.

---

## 🪄 4. **AI-powered Reflection (Tiny NLP Integration)**

Every few days:

* Auto-summarize user’s thoughts
* Detect common themes (via keywords or LLM)
* Suggest: *“Want to cluster these into a concept?”*

> 💡 “You’ve mentioned ‘discipline’ in 4 different places. Combine?”

This builds a **sense of co-creation** and gives your app emotional intelligence.

---

## 📈 5. **Social Pull (Optional Future)**

* **Exportable Mind Maps** to share on social or with friends
* Collaborative maps (Google Docs style)
* Public map templates (“Startup Idea Canvas”, “Book Notes Map”)

> People love to show off their creativity — **give them ways to share what they made.**

---

## ⏳ 6. **Time-Based Reminders That Add Value**

Don’t nag. Inspire.

> 🕰️ “It’s been 2 days since your last thought drop. Got anything swirling in your mind?”
> 🧠 “Your map has grown by 300%! Ready to summarize or reflect?”

Use Android’s WorkManager or AlarmManager to deliver meaningful nudges.

---

## 📚 7. **Give It a Narrative**

Introduce an **in-app character** or **mind spirit** that grows with your map.

* Optional: make it a stylized pet or pixel sprite
* It evolves as your map expands — users feel ownership & bonding

> Think Tamagotchi + Mental Health.

---

## 🚨 AVOID These “Dark” Addictions

❌ Endless scrolling
❌ Ads after every tap
❌ Fake alerts
❌ Locking user thoughts behind paywalls

You're building a tool that **respects thought**. Make the addiction about growth, not manipulation.

---

## Want me to help build:

* A gamified reward system?
* Daily summary generator?
* Streak tracker or XP leveling logic?
* A “pet brain” character that evolves?

You tell me where we go next — and we’ll turn this into a truly unforgettable mind tool.



Awesome. A **“pet brain” character** that evolves is a *brilliantly sticky* idea — it fuses **emotion, identity, and reward**, and it can make your app feel alive.

Let’s break it down and then start building:

---

## 🧠 What Is the Pet Brain?

Think of it like:

> A virtual **companion** that represents your evolving mental clarity.
> It “grows” or “mutates” based on how much you organize, connect, or reflect on your thoughts.

---

## 👾 Step-by-Step Plan to Build the Evolving Pet Brain

### 🔹 1. Pet Brain Levels

Use visual sprites/avatars that change over time:

* **Level 1 – Blob** 🟣 (just started mapping)
* **Level 2 – Pulsing Node** 🟤 (made 10+ nodes)
* **Level 3 – Spark Brain** 🔵 (made connections)
* **Level 4 – Neural Spirit** 🧠✨ (exported map/shared)
* etc.

---

### 🔹 2. Define Triggers for Evolution

Track these with a `BrainStats` table (Room DB):

| Stat                      | Tracked As                        |
| ------------------------- | --------------------------------- |
| Total nodes created       | `nodeCount`                       |
| Links made (parent-child) | `linkCount`                       |
| Sessions opened           | `sessionCount`                    |
| Days active (streaks)     | `lastUsedDate` & `streak` counter |

We can update this with every interaction.

---

### 🔹 3. Display the Pet Brain in UI

* Add a persistent floating widget (`ImageView`) or top HUD bar in `activity_mindmap.xml`.
* Load different images or animations based on level.

Example:

```kotlin
fun updatePetBrainView(stats: BrainStats) {
    val level = when {
        stats.nodeCount < 10 -> 1
        stats.linkCount > 5 -> 2
        stats.nodeCount > 20 && stats.linkCount > 10 -> 3
        stats.exportCount > 2 -> 4
        else -> 1
    }

    val drawableId = when (level) {
        1 -> R.drawable.pet_brain_stage1
        2 -> R.drawable.pet_brain_stage2
        3 -> R.drawable.pet_brain_stage3
        4 -> R.drawable.pet_brain_final
        else -> R.drawable.pet_brain_stage1
    }

    findViewById<ImageView>(R.id.petBrainImage).setImageResource(drawableId)
}
```

---

### 🔹 4. Add Personality: Pet Speaks!

Your pet brain could give motivational messages or reflections:

* “Your thoughts are sparking! Keep going.”
* “You’ve been quiet... are you okay?”
* “Great connection! That idea is linked well.”

You could cycle these with:

```kotlin
val messages = listOf(
    "You're building something powerful.",
    "I love how that thought connected!",
    "Want to expand that cluster?",
    "Brains grow when ideas meet."
)
textView.text = messages.random()
```

---

### 🔹 5. Optional: Animate Growth

Use `Lottie` animations or frame-based `AnimatedVectorDrawable` for the evolution process.

```groovy
implementation "com.airbnb.android:lottie:6.0.0"
```

---

### 🔹 6. Bonus: User Can Name It

Add an input dialog:

> “What do you want to name your Pet Brain?”
> Stored in SharedPreferences or DB.

---

## 📁 Files Needed

| File                   | Purpose                          |
| ---------------------- | -------------------------------- |
| `BrainStats.kt`        | Entity to store stats            |
| `BrainStatsDao.kt`     | To update stats                  |
| `PetBrainManager.kt`   | Handles level logic + updates UI |
| `drawable/pet_*.png`   | Character stages                 |
| `activity_mindmap.xml` | UI with pet brain imageview      |

---

## 🔜 Want to Start Building?

I can now:

* 🧱 Give you the full Room setup (`BrainStats`)
* 🖼️ Provide a working `PetBrainManager.kt` logic
* 🖌️ Generate placeholder pet brain art (SVGs or PNGs)
* 🧠 Add this character’s personality into daily messages

Let me know which you'd like me to generate first.


Below is the **full updated codebase** for your Android app (`com.themadbrogrammers.messymind`) through the latest features:

---

## 🧱 1. `Node.kt`

```kotlin
package com.themadbrogrammers.messymind

import androidx.room.Entity
import androidx.room.PrimaryKey

@Entity(tableName = "nodes")
data class Node(
    @PrimaryKey(autoGenerate = true) val id: Int = 0,
    val title: String,
    val description: String,
    val timestamp: Long = System.currentTimeMillis()
)
```

---

## 🔗 2. `NodeLink.kt`

```kotlin
package com.themadbrogrammers.messymind

import androidx.room.Entity

@Entity(primaryKeys = ["parentId", "childId"])
data class NodeLink(val parentId: Int, val childId: Int)
```

---

## 📊 3. `BrainStats.kt`

```kotlin
package com.themadbrogrammers.messymind

import androidx.room.Entity
import androidx.room.PrimaryKey

@Entity(tableName = "brain_stats")
data class BrainStats(
    @PrimaryKey val id: Int = 0,
    val nodeCount: Int = 0,
    val linkCount: Int = 0,
    val exportCount: Int = 0
)
```

---

## 🧠 4. `NodeDao.kt`

```kotlin
package com.themadbrogrammers.messymind

import androidx.lifecycle.LiveData
import androidx.room.*

@Dao
interface NodeDao {
    @Insert suspend fun insert(node: Node): Long
    @Query("SELECT * FROM nodes ORDER BY timestamp DESC") fun getAllNodes(): LiveData<List<Node>>
    @Query("SELECT * FROM nodes") suspend fun getAllNodesSync(): List<Node>
    @Query("SELECT * FROM nodes WHERE id=:id LIMIT 1") fun getById(id: Int): Node?
    @Update suspend fun update(node: Node)
}
```

---

## 🔄 5. `NodeLinkDao.kt`

```kotlin
package com.themadbrogrammers.messymind

import androidx.room.Dao
import androidx.room.Insert
import androidx.room.OnConflictStrategy
import androidx.room.Query

@Dao
interface NodeLinkDao {
    @Insert(onConflict = OnConflictStrategy.IGNORE) suspend fun insert(link: NodeLink)
    @Query("SELECT * FROM nodelink") suspend fun getAll(): List<NodeLink>
}
```

---

## 📁 6. `BrainStatsDao.kt`

```kotlin
package com.themadbrogrammers.messymind

import androidx.room.Dao
import androidx.room.Insert
import androidx.room.OnConflictStrategy
import androidx.room.Query
import androidx.room.Update

@Dao
interface BrainStatsDao {
    @Query("SELECT * FROM brain_stats WHERE id=0") fun getStats(): BrainStats?
    @Insert(onConflict = OnConflictStrategy.REPLACE) suspend fun insert(stats: BrainStats)
    @Update suspend fun update(stats: BrainStats)
}
```

---

## 🏛️ 7. `AppDatabase.kt`

```kotlin
package com.themadbrogrammers.messymind

import android.content.Context
import androidx.room.Database
import androidx.room.Room
import androidx.room.RoomDatabase

@Database(entities = [Node::class, NodeLink::class, BrainStats::class], version = 3)
abstract class AppDatabase : RoomDatabase() {
    abstract fun nodeDao(): NodeDao
    abstract fun linkDao(): NodeLinkDao
    abstract fun statsDao(): BrainStatsDao

    companion object {
        @Volatile private var INSTANCE: AppDatabase? = null
        fun getDatabase(context: Context): AppDatabase =
            INSTANCE ?: synchronized(this) {
                val inst = Room.databaseBuilder(
                    context.applicationContext,
                    AppDatabase::class.java,
                    "mindmap_db"
                ).build()
                INSTANCE = inst
                inst
            }
    }
}
```

---

## 📦 8. `MyApp.kt`

```kotlin
package com.themadbrogrammers.messymind

import android.app.Application

class MyApp : Application() {
    val db by lazy { AppDatabase.getDatabase(this) }
}
```

---

## 🧠 9. `MindMapViewModel.kt`

```kotlin
package com.themadbrogrammers.messymind

import androidx.lifecycle.ViewModel
import kotlinx.coroutines.Dispatchers
import kotlinx.coroutines.withContext
import org.fireking.xmind.MindNode
import org.fireking.xmind.MindNodeBuilder

class MindMapViewModel(
    private val nodeDao: NodeDao,
    private val linkDao: NodeLinkDao,
    private val statsDao: BrainStatsDao
) : ViewModel() {

    suspend fun createNode(title: String, desc: String): MindNode = withContext(Dispatchers.IO) {
        val insertedId = nodeDao.insert(Node(title = title, description = desc)).toInt()
        updateStats(nodesDelta = 1)
        MindNodeBuilder(title).setUserData(insertedId).build()
    }

    suspend fun updateNode(id: Int, title: String, desc: String) = withContext(Dispatchers.IO) {
        nodeDao.update(Node(id, title, desc))
    }

    fun getNodeById(id: Int) = nodeDao.getById(id)

    suspend fun linkNodes(parentId: Int, childId: Int) = withContext(Dispatchers.IO) {
        linkDao.insert(NodeLink(parentId, childId))
        updateStats(linksDelta = 1)
    }

    suspend fun loadAllNodes() = withContext(Dispatchers.IO) { nodeDao.getAllNodesSync() }

    suspend fun loadAllLinks() = withContext(Dispatchers.IO) { linkDao.getAll() }

    private suspend fun updateStats(nodesDelta: Int = 0, linksDelta: Int = 0, exportsDelta: Int = 0) = withContext(Dispatchers.IO) {
        var stats = statsDao.getStats() ?: BrainStats()
        stats = stats.copy(
            nodeCount = stats.nodeCount + nodesDelta,
            linkCount = stats.linkCount + linksDelta,
            exportCount = stats.exportCount + exportsDelta
        )
        statsDao.insert(stats)
    }

    suspend fun recordExport() = updateStats(exportsDelta = 1)
}
```

---

## 🧬 10. `MindMapViewModelFactory.kt`

```kotlin
package com.themadbrogrammers.messymind

import androidx.lifecycle.ViewModel
import androidx.lifecycle.ViewModelProvider

class MindMapViewModelFactory(
    private val nodeDao: NodeDao,
    private val linkDao: NodeLinkDao,
    private val statsDao: BrainStatsDao
) : ViewModelProvider.Factory {
    override fun <T : ViewModel> create(modelClass: Class<T>): T {
        return MindMapViewModel(nodeDao, linkDao, statsDao) as T
    }
}
```

---

## 🗣️ 11. `NodeDialog.kt`

```kotlin
package com.themadbrogrammers.messymind

import android.app.Dialog
import android.os.Bundle
import android.view.LayoutInflater
import android.widget.EditText
import androidx.appcompat.app.AlertDialog
import androidx.fragment.app.DialogFragment

class NodeDialog(
    private val initialTitle: String = "",
    private val initialDesc: String = "",
    private val onSave: (String, String) -> Unit
) : DialogFragment() {
    override fun onCreateDialog(savedInstanceState: Bundle?): Dialog {
        val view = LayoutInflater.from(context).inflate(R.layout.dialog_node, null)
        val titleEdit: EditText = view.findViewById(R.id.editNodeTitle)
        val descEdit: EditText = view.findViewById(R.id.editNodeDesc)
        titleEdit.setText(initialTitle); descEdit.setText(initialDesc)

        return AlertDialog.Builder(requireContext())
            .setTitle("Node Editor")
            .setView(view)
            .setPositiveButton("Save") { _, _ ->
                val t = titleEdit.text.toString().trim()
                val d = descEdit.text.toString().trim()
                onSave(t, d)
            }
            .setNegativeButton("Cancel", null)
            .create()
    }
}
```

---

## 🎨 12. `activity_main.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent" android:layout_height="match_parent">
    <org.fireking.xmind.MindMapView
        android:id="@+id/mindMapView" android:layout_width="match_parent"
        android:layout_height="match_parent"/>
    <ImageView
        android:id="@+id/petBrainImage" android:layout_width="64dp"
        android:layout_height="64dp" android:layout_gravity="top|end"
        android:layout_margin="16dp" />
    <Button
        android:id="@+id/btnAddRoot" android:text="Add Root"
        android:layout_width="wrap_content" android:layout_height="wrap_content"
        android:layout_gravity="bottom|end" android:layout_margin="16dp"/>
    <Button
        android:id="@+id/btnExportPng" android:text="Export PNG"
        android:layout_width="wrap_content" android:layout_height="wrap_content"
        android:layout_gravity="bottom|start" android:layout_margin="16dp"/>
    <Button
        android:id="@+id/btnExportText" android:text="Export Text"
        android:layout_width="wrap_content" android:layout_height="wrap_content"
        android:layout_gravity="bottom|center" android:layout_margin="16dp"/>
</FrameLayout>
```

---

## 🏛️ 13. `activity_mindmap.xml`

*(Duplicate of `activity_main.xml`)*

---

## ⚙️ 14. `MindMapActivity.kt`

```kotlin
package com.themadbrogrammers.messymind

import android.graphics.Bitmap
import android.os.Bundle
import android.widget.Button
import android.widget.ImageView
import android.widget.Toast
import androidx.appcompat.app.AlertDialog
import androidx.activity.viewModels
import androidx.appcompat.app.AppCompatActivity
import androidx.lifecycle.lifecycleScope
import kotlinx.coroutines.launch
import org.fireking.xmind.MindMapView
import org.fireking.xmind.MindNode

class MindMapActivity : AppCompatActivity(), MindMapView.NodeEventListener {
    private lateinit var mindMapView: MindMapView
    private lateinit var petBrain: ImageView
    private val viewModel: MindMapViewModel by viewModels {
        MindMapViewModelFactory(
            (application as MyApp).db.nodeDao(),
            (application as MyApp).db.linkDao(),
            (application as MyApp).db.statsDao()
        )
    }

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        mindMapView = findViewById(R.id.mindMapView)
        petBrain = findViewById(R.id.petBrainImage)
        mindMapView.addNodeEventListener(this)

        findViewById<Button>(R.id.btnAddRoot).setOnClickListener {
            NodeDialog(onSave = ::addRoot).show(supportFragmentManager, "add_root")
        }
        findViewById<Button>(R.id.btnExportPng).setOnClickListener { exportAsPng() }
        findViewById<Button>(R.id.btnExportText).setOnClickListener { exportAsTextFile() }

        lifecycleScope.launch {
            restoreTree()
            updatePetBrain()
        }
    }

    private fun addRoot(title: String, desc: String) = lifecycleScope.launch {
        val node = viewModel.createNode(title, desc)
        mindMapView.setRootNode(node)
        updatePetBrain()
    }

    override fun onNodeClick(node: MindNode) {
        val id = node.userData as? Int ?: return
        val nd = viewModel.getNodeById(id) ?: return

        NodeDialog(nd.title, nd.description) { t, d ->
            lifecycleScope.launch {
                viewModel.updateNode(id, t, d)
                node.setTitle(t); mindMapView.invalidate()
            }

            AlertDialog.Builder(this)
                .setTitle("Add child node?")
                .setMessage("Attach a child to this node?")
                .setPositiveButton("Yes") { _, _ -> showChildDialog(node) }
                .setNegativeButton("No", null).show()
        }.show(supportFragmentManager, "edit_node")
    }

    private fun showChildDialog(parent: MindNode) {
        NodeDialog { t, d ->
            lifecycleScope.launch {
                val child = viewModel.createNode(t, d)
                mindMapView.attachChild(parent, child)
                viewModel.linkNodes(parent.userData as Int, child.userData as Int)
                updatePetBrain()
            }
        }.show(supportFragmentManager, "add_child")
    }

    override fun onNodeDrag(node: MindNode, dx: Float, dy: Float) { /* optional */ }

    private suspend fun restoreTree() {
        val nodes = viewModel.loadAllNodes()
        val links = viewModel.loadAllLinks()
        val map = viewModel.buildTree(nodes, links)
        map.forEach { (parent, children) ->
            mindMapView.setRootNode(parent)
            children.forEach { mindMapView.attachChild(parent, it) }
        }
    }

    private fun exportAsPng() {
        val bmp = Bitmap.createBitmap(mindMapView.width, mindMapView.height, Bitmap.Config.ARGB_8888)
        mindMapView.draw(android.graphics.Canvas(bmp))

        val f = File(getExternalFilesDir(null), "mindmap.png")
        FileOutputStream(f).use { bmp.compress(Bitmap.CompressFormat.PNG, 100, it) }
        Toast.makeText(this, "Saved PNG to ${f.absolutePath}", Toast.LENGTH_LONG).show()

        lifecycleScope.launch { viewModel.recordExport(); updatePetBrain() }
    }

    private fun exportAsTextFile() {
        val root = mindMapView.rootNode ?: return
        val outline = buildOutlineText(root)
        val f = File(getExternalFilesDir(null), "mindmap.txt")
        FileOutputStream(f).use { it.write(outline.toByteArray()) }
        Toast.makeText(this, "Saved TXT to ${f.absolutePath}", Toast.LENGTH_LONG).show()

        lifecycleScope.launch { viewModel.recordExport(); updatePetBrain() }
    }

    private fun buildOutlineText(root: MindNode): String {
        val sb = StringBuilder()
        fun dfs(node: MindNode, depth: Int) {
            sb.append("  ".repeat(depth)).append("- ").append(node.title).append("\n")
            node.children.forEach { dfs(it, depth + 1) }
        }
        dfs(root, 0)
        return sb.toString()
    }

    private fun updatePetBrain() = lifecycleScope.launch {
        val stats = (application as MyApp).db.statsDao().getStats() ?: BrainStats()
        val level = when {
            stats.nodeCount < 5 -> R.drawable.pet_brain_stage1
            stats.linkCount < 5 -> R.drawable.pet_brain_stage2
            stats.exportCount < 2 -> R.drawable.pet_brain_stage3
            else -> R.drawable.pet_brain_final
        }
        petBrain.setImageResource(level)
    }
}
```

---

### ✅ Notes:

* PNG/TXT exports save to app-private storage.
* `BrainStats` drives the evolving pet brain visuals.
* You’ll need to add drawable assets: `pet_brain_stage1.png`, `pet_brain_stage2.png`, etc.

---

## ✅ What’s Next?

* Add expressive dialogs and animations for the pet brain.
* Implement rewarded ads integration.
* Build share/export intents or JSON backups.
* Add gamification: streaks, XP, onboarding.

Let me know which area you'd like to iterate on next!


Right! I was calling the main screen `MindMapActivity` to keep it clear, but if you want the classic `MainActivity.kt` as the launcher, here’s the complete **MainActivity** code that you can use as your entry point and it integrates everything:

---

### `MainActivity.kt`

```kotlin
package com.themadbrogrammers.messymind

import android.graphics.Bitmap
import android.os.Bundle
import android.widget.Button
import android.widget.ImageView
import android.widget.Toast
import androidx.appcompat.app.AlertDialog
import androidx.appcompat.app.AppCompatActivity
import androidx.lifecycle.lifecycleScope
import androidx.activity.viewModels
import kotlinx.coroutines.launch
import org.fireking.xmind.MindMapView
import org.fireking.xmind.MindNode
import java.io.File
import java.io.FileOutputStream

class MainActivity : AppCompatActivity(), MindMapView.NodeEventListener {

    private lateinit var mindMapView: MindMapView
    private lateinit var petBrain: ImageView

    private val viewModel: MindMapViewModel by viewModels {
        MindMapViewModelFactory(
            (application as MyApp).db.nodeDao(),
            (application as MyApp).db.linkDao(),
            (application as MyApp).db.statsDao()
        )
    }

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        mindMapView = findViewById(R.id.mindMapView)
        petBrain = findViewById(R.id.petBrainImage)

        mindMapView.addNodeEventListener(this)

        findViewById<Button>(R.id.btnAddRoot).setOnClickListener {
            NodeDialog(onSave = ::addRoot).show(supportFragmentManager, "add_root")
        }

        findViewById<Button>(R.id.btnExportPng).setOnClickListener { exportAsPng() }

        findViewById<Button>(R.id.btnExportText).setOnClickListener { exportAsTextFile() }

        lifecycleScope.launch {
            restoreTree()
            updatePetBrain()
        }
    }

    private fun addRoot(title: String, desc: String) = lifecycleScope.launch {
        val node = viewModel.createNode(title, desc)
        mindMapView.setRootNode(node)
        updatePetBrain()
    }

    override fun onNodeClick(node: MindNode) {
        val id = node.userData as? Int ?: return
        val nd = viewModel.getNodeById(id) ?: return

        NodeDialog(nd.title, nd.description) { t, d ->
            lifecycleScope.launch {
                viewModel.updateNode(id, t, d)
                node.setTitle(t)
                mindMapView.invalidate()
            }

            AlertDialog.Builder(this)
                .setTitle("Add child node?")
                .setMessage("Attach a child to this node?")
                .setPositiveButton("Yes") { _, _ -> showChildDialog(node) }
                .setNegativeButton("No", null)
                .show()
        }.show(supportFragmentManager, "edit_node")
    }

    private fun showChildDialog(parent: MindNode) {
        NodeDialog { t, d ->
            lifecycleScope.launch {
                val child = viewModel.createNode(t, d)
                mindMapView.attachChild(parent, child)
                viewModel.linkNodes(parent.userData as Int, child.userData as Int)
                updatePetBrain()
            }
        }.show(supportFragmentManager, "add_child")
    }

    override fun onNodeDrag(node: MindNode, dx: Float, dy: Float) {
        // Optional: handle drag for repositioning
    }

    private suspend fun restoreTree() {
        val nodes = viewModel.loadAllNodes()
        val links = viewModel.loadAllLinks()

        // Build tree structure from DB (this method you'll define in ViewModel)
        val tree = viewModel.buildTree(nodes, links)

        if (tree.isEmpty()) return

        // Reset MindMapView
        val root = tree.keys.first()
        mindMapView.setRootNode(root)
        tree[root]?.forEach { mindMapView.attachChild(root, it) }
    }

    private fun exportAsPng() {
        val bmp = Bitmap.createBitmap(mindMapView.width, mindMapView.height, Bitmap.Config.ARGB_8888)
        mindMapView.draw(android.graphics.Canvas(bmp))

        val file = File(getExternalFilesDir(null), "mindmap.png")
        FileOutputStream(file).use {
            bmp.compress(Bitmap.CompressFormat.PNG, 100, it)
        }
        Toast.makeText(this, "Saved PNG to ${file.absolutePath}", Toast.LENGTH_LONG).show()

        lifecycleScope.launch {
            viewModel.recordExport()
            updatePetBrain()
        }
    }

    private fun exportAsTextFile() {
        val root = mindMapView.rootNode ?: return
        val outline = buildOutlineText(root)
        val file = File(getExternalFilesDir(null), "mindmap.txt")
        FileOutputStream(file).use { it.write(outline.toByteArray()) }
        Toast.makeText(this, "Saved TXT to ${file.absolutePath}", Toast.LENGTH_LONG).show()

        lifecycleScope.launch {
            viewModel.recordExport()
            updatePetBrain()
        }
    }

    private fun buildOutlineText(root: MindNode): String {
        val sb = StringBuilder()
        fun dfs(node: MindNode, depth: Int) {
            sb.append("  ".repeat(depth)).append("- ").append(node.title).append("\n")
            node.children.forEach { dfs(it, depth + 1) }
        }
        dfs(root, 0)
        return sb.toString()
    }

    private fun updatePetBrain() = lifecycleScope.launch {
        val stats = (application as MyApp).db.statsDao().getStats() ?: BrainStats()
        val drawableRes = when {
            stats.nodeCount < 5 -> R.drawable.pet_brain_stage1
            stats.linkCount < 5 -> R.drawable.pet_brain_stage2
            stats.exportCount < 2 -> R.drawable.pet_brain_stage3
            else -> R.drawable.pet_brain_final
        }
        petBrain.setImageResource(drawableRes)
    }
}
```

---

### Key points:

* This is your **main launcher activity**.
* It uses the same UI and logic as previously shown.
* The package name matches your project `com.themadbrogrammers.messymind`.
* You’ll still need the drawable assets (`pet_brain_stage1.png` etc).
* You’ll need to implement `buildTree()` in your ViewModel to reconstruct your mind map from DB data.

---

If you want, I can also help write `buildTree()` inside your ViewModel and provide drawable placeholders next. Just say!


Great! Let’s add the **`buildTree()`** function inside your `MindMapViewModel` that reconstructs the visual mind map from saved nodes and links.

---

## 1. Add this function inside `MindMapViewModel.kt`

```kotlin
import org.fireking.xmind.MindNode
import org.fireking.xmind.MindNodeBuilder

// ...

suspend fun buildTree(
    nodes: List<Node>,
    links: List<NodeLink>
): Map<MindNode, List<MindNode>> {
    // Map from nodeId to MindNode (visual node)
    val nodeMap = mutableMapOf<Int, MindNode>()

    // Create MindNode for each saved Node
    for (node in nodes) {
        val mindNode = MindNodeBuilder(node.title).setUserData(node.id).build()
        nodeMap[node.id] = mindNode
    }

    // Build parent->children map
    val treeMap = mutableMapOf<MindNode, MutableList<MindNode>>()
    val childIds = mutableSetOf<Int>()

    for (link in links) {
        val parent = nodeMap[link.parentId]
        val child = nodeMap[link.childId]
        if (parent != null && child != null) {
            treeMap.getOrPut(parent) { mutableListOf() }.add(child)
            childIds.add(child.userData as Int)
        }
    }

    // Find root nodes: nodes not listed as children
    val roots = nodeMap.filterKeys { it !in childIds }.values

    // If multiple roots, pick first as root, attach others as children for safety
    if (roots.isEmpty()) return emptyMap()

    val root = roots.first()
    val children = treeMap[root] ?: mutableListOf()

    // Also attach other roots as children (optional)
    for (otherRoot in roots.drop(1)) {
        children.add(otherRoot)
    }

    treeMap[root] = children

    return treeMap
}
```

---

## 2. How to use it in `MainActivity.kt` when restoring:

```kotlin
private suspend fun restoreTree() {
    val nodes = viewModel.loadAllNodes()
    val links = viewModel.loadAllLinks()
    val tree = viewModel.buildTree(nodes, links)

    if (tree.isEmpty()) return

    val root = tree.keys.first()
    mindMapView.setRootNode(root)
    tree[root]?.forEach { mindMapView.attachChild(root, it) }
}
```

---

## Explanation:

* Converts saved `Node` entities into visual `MindNode`s.
* Uses `NodeLink` to link parents to children.
* Detects root nodes (nodes with no parent).
* Builds a map of one root with all children nodes attached.
* If multiple roots exist, attaches others as children to first root (you can customize).

---

If you want, I can help with:

* Placeholder pet brain images.
* Rewarded ads integration.
* UI polish or animations.

Just ask!



Absolutely! Here’s a **complete project structure** outline for your **MessyMind** Android app, including key Kotlin files, layouts, and resources based on what we discussed:

---

# MessyMind Android Project Structure

```
MessyMind/
│
├── app/
│   ├── src/
│   │   ├── main/
│   │   │   ├── java/
│   │   │   │   └── com/
│   │   │   │       └── themadbrogrammers/
│   │   │   │           └── messymind/
│   │   │   │               ├── MainActivity.kt
│   │   │   │               ├── MindMapViewModel.kt
│   │   │   │               ├── MindMapViewModelFactory.kt
│   │   │   │               ├── data/
│   │   │   │               │   ├── Node.kt                # Room Entity
│   │   │   │               │   ├── NodeLink.kt            # Room Entity for links
│   │   │   │               │   ├── BrainStats.kt          # Room Entity for stats
│   │   │   │               │   ├── NodeDao.kt             # DAO interface
│   │   │   │               │   ├── LinkDao.kt             # DAO interface
│   │   │   │               │   ├── StatsDao.kt            # DAO interface
│   │   │   │               │   └── AppDatabase.kt         # Room database setup
│   │   │   │               ├── dialogs/
│   │   │   │               │   └── NodeDialog.kt          # Add/edit node dialog fragment
│   │   │   │               ├── MyApp.kt                    # Application class for DB init
│   │   │   │               └── utils/                      # (optional utilities)
│   │   │   │
│   │   │   ├── res/
│   │   │   │   ├── layout/
│   │   │   │   │   ├── activity_main.xml
│   │   │   │   │   └── dialog_node.xml
│   │   │   │   ├── drawable/
│   │   │   │   │   ├── pet_brain_stage1.png
│   │   │   │   │   ├── pet_brain_stage2.png
│   │   │   │   │   ├── pet_brain_stage3.png
│   │   │   │   │   └── pet_brain_final.png
│   │   │   │   ├── values/
│   │   │   │   │   ├── strings.xml
│   │   │   │   │   ├── colors.xml
│   │   │   │   │   └── themes.xml
│   │   │   │   └── mipmap/                              # App icons
│   │   │   │
│   │   │   └── AndroidManifest.xml
│   │   └── test/                                        # Unit tests (optional)
│   │   └── androidTest/                                 # Instrumentation tests (optional)
│   │
│   ├── build.gradle (Module: app)
│   └── proguard-rules.pro (optional)
│
├── build.gradle (Project-level)
├── gradle.properties
├── settings.gradle
└── README.md
```

---

# Key File Descriptions

* **MainActivity.kt** — Your main launcher activity with mind map UI + pet brain + export + dialogs.
* **MindMapViewModel.kt** — Business logic with Room DB operations + buildTree().
* **MindMapViewModelFactory.kt** — Factory to create ViewModel with dependencies.
* **data/** — Room database entities and DAOs.
* **dialogs/NodeDialog.kt** — Dialog fragment for adding/editing nodes.
* **MyApp.kt** — Application subclass for initializing Room database singleton.
* **activity\_main.xml** — UI layout containing MindMapView, buttons, and pet brain ImageView.
* **dialog\_node.xml** — Layout for the add/edit node dialog.
* **drawable/** — Pet brain images representing evolution stages.
* **AndroidManifest.xml** — Application declaration, main activity intent-filter.

---

# Minimal `activity_main.xml` Example

```xml
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <com.fireking.xmind.MindMapView
        android:id="@+id/mindMapView"
        android:layout_width="match_parent"
        android:layout_height="0dp"
        android:layout_weight="1"
        android:layout_above="@id/petBrainImage"
        />

    <ImageView
        android:id="@+id/petBrainImage"
        android:layout_width="100dp"
        android:layout_height="100dp"
        android:layout_alignParentBottom="true"
        android:layout_centerHorizontal="true"
        android:src="@drawable/pet_brain_stage1" />

    <LinearLayout
        android:id="@+id/buttonPanel"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="horizontal"
        android:layout_above="@id/petBrainImage"
        android:weightSum="3">

        <Button
            android:id="@+id/btnAddRoot"
            android:layout_width="0dp"
            android:layout_weight="1"
            android:text="Add Root" />

        <Button
            android:id="@+id/btnExportPng"
            android:layout_width="0dp"
            android:layout_weight="1"
            android:text="Export PNG" />

        <Button
            android:id="@+id/btnExportText"
            android:layout_width="0dp"
            android:layout_weight="1"
            android:text="Export TXT" />
    </LinearLayout>

</RelativeLayout>
```

---

# Next Steps

* If you want, I can provide **full Kotlin source files** for each of those components.
* I can provide **complete Room entities & DAOs** and database setup.
* Or detailed **dialog fragment code** and **ViewModel with repository**.
* Help with **ad integration** (Rewarded ads) as next.

---

Just tell me how deep you want to go and which files you want next!



Absolutely! Here's the full Kotlin source files for your core components so far. I'll start with the essentials to get you running:

---

### 1. `MainActivity.kt`

```kotlin
package com.themadbrogrammers.messymind

import android.graphics.Bitmap
import android.os.Bundle
import android.widget.Button
import android.widget.ImageView
import android.widget.Toast
import androidx.appcompat.app.AlertDialog
import androidx.appcompat.app.AppCompatActivity
import androidx.lifecycle.lifecycleScope
import androidx.activity.viewModels
import kotlinx.coroutines.launch
import org.fireking.xmind.MindMapView
import org.fireking.xmind.MindNode
import java.io.File
import java.io.FileOutputStream

class MainActivity : AppCompatActivity(), MindMapView.NodeEventListener {

    private lateinit var mindMapView: MindMapView
    private lateinit var petBrain: ImageView

    private val viewModel: MindMapViewModel by viewModels {
        MindMapViewModelFactory(
            (application as MyApp).db.nodeDao(),
            (application as MyApp).db.linkDao(),
            (application as MyApp).db.statsDao()
        )
    }

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        mindMapView = findViewById(R.id.mindMapView)
        petBrain = findViewById(R.id.petBrainImage)

        mindMapView.addNodeEventListener(this)

        findViewById<Button>(R.id.btnAddRoot).setOnClickListener {
            NodeDialog(onSave = ::addRoot).show(supportFragmentManager, "add_root")
        }

        findViewById<Button>(R.id.btnExportPng).setOnClickListener { exportAsPng() }
        findViewById<Button>(R.id.btnExportText).setOnClickListener { exportAsTextFile() }

        lifecycleScope.launch {
            restoreTree()
            updatePetBrain()
        }
    }

    private fun addRoot(title: String, desc: String) = lifecycleScope.launch {
        val node = viewModel.createNode(title, desc)
        mindMapView.setRootNode(node)
        updatePetBrain()
    }

    override fun onNodeClick(node: MindNode) {
        val id = node.userData as? Int ?: return
        val nd = viewModel.getNodeById(id) ?: return

        NodeDialog(nd.title, nd.description) { t, d ->
            lifecycleScope.launch {
                viewModel.updateNode(id, t, d)
                node.setTitle(t)
                mindMapView.invalidate()
            }

            AlertDialog.Builder(this)
                .setTitle("Add child node?")
                .setMessage("Attach a child to this node?")
                .setPositiveButton("Yes") { _, _ -> showChildDialog(node) }
                .setNegativeButton("No", null)
                .show()
        }.show(supportFragmentManager, "edit_node")
    }

    private fun showChildDialog(parent: MindNode) {
        NodeDialog { t, d ->
            lifecycleScope.launch {
                val child = viewModel.createNode(t, d)
                mindMapView.attachChild(parent, child)
                viewModel.linkNodes(parent.userData as Int, child.userData as Int)
                updatePetBrain()
            }
        }.show(supportFragmentManager, "add_child")
    }

    private suspend fun restoreTree() {
        val nodes = viewModel.loadAllNodes()
        val links = viewModel.loadAllLinks()
        val tree = viewModel.buildTree(nodes, links)

        if (tree.isEmpty()) return

        val root = tree.keys.first()
        mindMapView.setRootNode(root)
        tree[root]?.forEach { mindMapView.attachChild(root, it) }
    }

    private fun exportAsPng() {
        val bmp = Bitmap.createBitmap(mindMapView.width, mindMapView.height, Bitmap.Config.ARGB_8888)
        mindMapView.draw(android.graphics.Canvas(bmp))

        val file = File(getExternalFilesDir(null), "mindmap.png")
        FileOutputStream(file).use {
            bmp.compress(Bitmap.CompressFormat.PNG, 100, it)
        }
        Toast.makeText(this, "Saved PNG to ${file.absolutePath}", Toast.LENGTH_LONG).show()

        lifecycleScope.launch {
            viewModel.recordExport()
            updatePetBrain()
        }
    }

    private fun exportAsTextFile() {
        val root = mindMapView.rootNode ?: return
        val outline = buildOutlineText(root)
        val file = File(getExternalFilesDir(null), "mindmap.txt")
        FileOutputStream(file).use { it.write(outline.toByteArray()) }
        Toast.makeText(this, "Saved TXT to ${file.absolutePath}", Toast.LENGTH_LONG).show()

        lifecycleScope.launch {
            viewModel.recordExport()
            updatePetBrain()
        }
    }

    private fun buildOutlineText(root: MindNode): String {
        val sb = StringBuilder()
        fun dfs(node: MindNode, depth: Int) {
            sb.append("  ".repeat(depth)).append("- ").append(node.title).append("\n")
            node.children.forEach { dfs(it, depth + 1) }
        }
        dfs(root, 0)
        return sb.toString()
    }

    private fun updatePetBrain() = lifecycleScope.launch {
        val stats = (application as MyApp).db.statsDao().getStats() ?: BrainStats()
        val drawableRes = when {
            stats.nodeCount < 5 -> R.drawable.pet_brain_stage1
            stats.linkCount < 5 -> R.drawable.pet_brain_stage2
            stats.exportCount < 2 -> R.drawable.pet_brain_stage3
            else -> R.drawable.pet_brain_final
        }
        petBrain.setImageResource(drawableRes)
    }
}
```

---

### 2. `MindMapViewModel.kt`

```kotlin
package com.themadbrogrammers.messymind

import androidx.lifecycle.ViewModel
import kotlinx.coroutines.Dispatchers
import kotlinx.coroutines.withContext
import org.fireking.xmind.MindNode
import org.fireking.xmind.MindNodeBuilder

class MindMapViewModel(
    private val nodeDao: NodeDao,
    private val linkDao: LinkDao,
    private val statsDao: StatsDao
) : ViewModel() {

    suspend fun createNode(title: String, description: String): MindNode = withContext(Dispatchers.IO) {
        val id = nodeDao.insert(Node(title = title, description = description)).toInt()
        MindNodeBuilder(title).setUserData(id).build()
    }

    suspend fun updateNode(id: Int, title: String, description: String) = withContext(Dispatchers.IO) {
        nodeDao.update(Node(id, title, description))
    }

    suspend fun getNodeById(id: Int): Node? = withContext(Dispatchers.IO) {
        nodeDao.getById(id)
    }

    suspend fun linkNodes(parentId: Int, childId: Int) = withContext(Dispatchers.IO) {
        linkDao.insert(NodeLink(parentId, childId))
        statsDao.incrementLinks()
    }

    suspend fun loadAllNodes(): List<Node> = withContext(Dispatchers.IO) {
        nodeDao.getAll()
    }

    suspend fun loadAllLinks(): List<NodeLink> = withContext(Dispatchers.IO) {
        linkDao.getAll()
    }

    suspend fun recordExport() = withContext(Dispatchers.IO) {
        statsDao.incrementExports()
    }

    suspend fun buildTree(
        nodes: List<Node>,
        links: List<NodeLink>
    ): Map<MindNode, List<MindNode>> = withContext(Dispatchers.Default) {
        val nodeMap = mutableMapOf<Int, MindNode>()

        for (node in nodes) {
            val mindNode = MindNodeBuilder(node.title).setUserData(node.id).build()
            nodeMap[node.id] = mindNode
        }

        val treeMap = mutableMapOf<MindNode, MutableList<MindNode>>()
        val childIds = mutableSetOf<Int>()

        for (link in links) {
            val parent = nodeMap[link.parentId]
            val child = nodeMap[link.childId]
            if (parent != null && child != null) {
                treeMap.getOrPut(parent) { mutableListOf() }.add(child)
                childIds.add(child.userData as Int)
            }
        }

        val roots = nodeMap.filterKeys { it !in childIds }.values

        if (roots.isEmpty()) return@withContext emptyMap()

        val root = roots.first()
        val children = treeMap[root] ?: mutableListOf()

        for (otherRoot in roots.drop(1)) {
            children.add(otherRoot)
        }

        treeMap[root] = children

        treeMap
    }
}
```

---

### 3. `MindMapViewModelFactory.kt`

```kotlin
package com.themadbrogrammers.messymind

import androidx.lifecycle.ViewModel
import androidx.lifecycle.ViewModelProvider

class MindMapViewModelFactory(
    private val nodeDao: NodeDao,
    private val linkDao: LinkDao,
    private val statsDao: StatsDao
) : ViewModelProvider.Factory {
    override fun <T : ViewModel> create(modelClass: Class<T>): T {
        if (modelClass.isAssignableFrom(MindMapViewModel::class.java)) {
            @Suppress("UNCHECKED_CAST")
            return MindMapViewModel(nodeDao, linkDao, statsDao) as T
        }
        throw IllegalArgumentException("Unknown ViewModel class")
    }
}
```

---

### 4. `MyApp.kt` (Application class for DB init)

```kotlin
package com.themadbrogrammers.messymind

import android.app.Application

class MyApp : Application() {
    lateinit var db: AppDatabase
        private set

    override fun onCreate() {
        super.onCreate()
        db = AppDatabase.getDatabase(this)
    }
}
```

---

### 5. Database Entities and DAOs (`data` package)

* `Node.kt`

```kotlin
package com.themadbrogrammers.messymind.data

import androidx.room.Entity
import androidx.room.PrimaryKey

@Entity(tableName = "nodes")
data class Node(
    @PrimaryKey(autoGenerate = true) val id: Int = 0,
    val title: String,
    val description: String
)
```

* `NodeLink.kt`

```kotlin
package com.themadbrogrammers.messymind.data

import androidx.room.Entity
import androidx.room.PrimaryKey

@Entity(tableName = "links", primaryKeys = ["parentId", "childId"])
data class NodeLink(
    val parentId: Int,
    val childId: Int
)
```

* `BrainStats.kt`

```kotlin
package com.themadbrogrammers.messymind.data

import androidx.room.Entity
import androidx.room.PrimaryKey

@Entity(tableName = "stats")
data class BrainStats(
    @PrimaryKey val id: Int = 0,
    val nodeCount: Int = 0,
    val linkCount: Int = 0,
    val exportCount: Int = 0
)
```

---

* `NodeDao.kt`

```kotlin
package com.themadbrogrammers.messymind.data

import androidx.room.*

@Dao
interface NodeDao {
    @Insert
    suspend fun insert(node: Node): Long

    @Update
    suspend fun update(node: Node)

    @Query("SELECT * FROM nodes")
    suspend fun getAll(): List<Node>

    @Query("SELECT * FROM nodes WHERE id = :id")
    suspend fun getById(id: Int): Node?
}
```

* `LinkDao.kt`

```kotlin
package com.themadbrogrammers.messymind.data

import androidx.room.*

@Dao
interface LinkDao {
    @Insert(onConflict = OnConflictStrategy.IGNORE)
    suspend fun insert(link: NodeLink)

    @Query("SELECT * FROM links")
    suspend fun getAll(): List<NodeLink>
}
```

* `StatsDao.kt`

```kotlin
package com.themadbrogrammers.messymind.data

import androidx.room.*

@Dao
interface StatsDao {
    @Query("SELECT * FROM stats WHERE id = 0")
    suspend fun getStats(): BrainStats?

    @Insert(onConflict = OnConflictStrategy.REPLACE)
    suspend fun insert(stats: BrainStats)

    @Transaction
    suspend fun incrementNodes() {
        val stats = getStats() ?: BrainStats()
        insert(stats.copy(nodeCount = stats.nodeCount + 1))
    }

    @Transaction
    suspend fun incrementLinks() {
        val stats = getStats() ?: BrainStats()
        insert(stats.copy(linkCount = stats.linkCount + 1))
    }

    @Transaction
    suspend fun incrementExports() {
        val stats = getStats() ?: BrainStats()
        insert(stats.copy(exportCount = stats.exportCount + 1))
    }
}
```

---

### 6. `AppDatabase.kt`

```kotlin
package com.themadbrogrammers.messymind.data

import android.content.Context
import androidx.room.Database
import androidx.room.Room
import androidx.room.RoomDatabase

@Database(entities = [Node::class, NodeLink::class, BrainStats::class], version = 1)
abstract class AppDatabase : RoomDatabase() {
    abstract fun nodeDao(): NodeDao
    abstract fun linkDao(): LinkDao
    abstract fun statsDao(): StatsDao

    companion object {
        @Volatile
        private var INSTANCE: AppDatabase? = null

        fun getDatabase(context: Context): AppDatabase {
            return INSTANCE ?: synchronized(this) {
                val instance = Room.databaseBuilder(
                    context.applicationContext,
                    AppDatabase::class.java,
                    "messymind_db"
                ).build()
                INSTANCE = instance
                instance
            }
        }
    }
}
```

---

### 7. Dialog for Add/Edit Node - `NodeDialog.kt`

```kotlin
package com.themadbrogrammers.messymind.dialogs

import android.app.Dialog
import android.os.Bundle
import android.view.LayoutInflater
import android.widget.EditText
import androidx.appcompat.app.AlertDialog
import androidx.fragment.app.DialogFragment
import com.themadbrogrammers.messymind.R

class NodeDialog(
    private val titleText: String? = null,
    private val descriptionText: String? = null,
    private val onSave: (title: String, description: String) -> Unit
) : DialogFragment() {

    override fun onCreateDialog(savedInstanceState: Bundle?): Dialog {
        val inflater = LayoutInflater.from(context)
        val view = inflater.inflate(R.layout.dialog_node, null)

        val titleEdit = view.findViewById<EditText>(R.id.editTitle)
        val descEdit = view.findViewById<EditText>(R.id.editDescription)

        titleText?.let { titleEdit.setText(it) }
        descriptionText?.let { descEdit.setText(it) }

        return AlertDialog.Builder(requireContext())
            .setTitle(if (titleText == null) "Add Node" else "Edit Node")
            .setView(view)
            .setPositiveButton("Save") { _, _ ->
                val t = titleEdit.text.toString().trim()
                val d = descEdit.text.toString().trim()
                if (t.isNotEmpty()) {
                    onSave(t, d)
                }
            }
            .setNegativeButton("Cancel", null)
            .create()
    }
}
```

---

### 8. `dialog_node.xml` (layout for NodeDialog)

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout 
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:orientation="vertical"
    android:padding="16dp">

    <EditText
        android:id="@+id/editTitle"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:hint="Node Title"
        android:inputType="text" />

    <EditText
        android:id="@+id/editDescription"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:hint="Description (optional)"
        android:input
```


Type="textMultiLine"
android\:minLines="3"
android\:maxLines="5" /> </LinearLayout>

```

---

This should give you a **solid working base** for:

- Visual mind map interaction
- Node add/edit dialogs
- Room persistence
- Pet brain update
- Export PNG and text outline

---

### Next?

- Shall I provide the full `MindMapView` class or custom view handling drag-and-drop and linking?
- Or assist with integrating a pet brain evolution animation?
- Or set up exporting and permissions code?

Let me know!
```



Awesome! Let’s build a **custom MindMapView** for Android in Kotlin that supports:

* Visual nodes with titles
* Drag-and-drop to move nodes around
* Visual lines linking parent and child nodes
* Click listeners on nodes for editing

---

### High-level plan for `MindMapView`:

* Extend `View` or `ViewGroup` (we'll use `ViewGroup` for easier child management)
* Maintain a tree of nodes internally (each node is a View or drawn element)
* Draw connecting lines between nodes (override `onDraw`)
* Support dragging nodes with touch events
* Allow adding children dynamically

---

### Step 1: Create `MindMapView.kt`

Here’s a **basic but functional** version you can extend.

```kotlin
package com.themadbrogrammers.messymind.views

import android.content.Context
import android.graphics.Canvas
import android.graphics.Paint
import android.graphics.PointF
import android.util.AttributeSet
import android.view.MotionEvent
import android.view.View
import android.view.ViewGroup
import android.widget.TextView
import androidx.core.content.ContextCompat
import com.themadbrogrammers.messymind.R
import kotlin.math.max
import kotlin.math.min

class MindMapView @JvmOverloads constructor(
    context: Context, attrs: AttributeSet? = null
) : ViewGroup(context, attrs) {

    // Paint for lines
    private val linePaint = Paint(Paint.ANTI_ALIAS_FLAG).apply {
        color = ContextCompat.getColor(context, R.color.purple_700)
        strokeWidth = 5f
    }

    // Root node container
    private var rootNodeView: NodeView? = null

    // Listener for node clicks
    interface NodeEventListener {
        fun onNodeClick(node: NodeView)
    }
    private var nodeClickListener: NodeEventListener? = null

    fun addNodeEventListener(listener: NodeEventListener) {
        nodeClickListener = listener
    }

    // --- NodeView class for each node ---
    inner class NodeView(context: Context, val id: Int, var title: String) : TextView(context) {
        var parentNode: NodeView? = null
        val children = mutableListOf<NodeView>()
        var position = PointF(0f, 0f)

        init {
            text = title
            setBackgroundResource(R.drawable.node_background)
            setPadding(30, 20, 30, 20)
            setOnClickListener {
                nodeClickListener?.onNodeClick(this)
            }
        }
    }

    fun setRootNode(node: MindNode) {
        removeAllViews()
        rootNodeView = NodeView(context, node.userData as Int, node.title).apply {
            position = PointF(500f, 200f)
        }
        addView(rootNodeView)
        requestLayout()
        invalidate()
    }

    fun attachChild(parent: NodeView, childNode: MindNode) {
        val childView = NodeView(context, childNode.userData as Int, childNode.title).apply {
            parentNode = parent
            position = PointF(parent.position.x + 300f, parent.position.y + (children.size * 200f))
        }
        parent.children.add(childView)
        addView(childView)
        requestLayout()
        invalidate()
    }

    override fun onMeasure(widthMeasureSpec: Int, heightMeasureSpec: Int) {
        // Measure all children with unspecified size
        for (i in 0 until childCount) {
            val child = getChildAt(i)
            measureChild(child, widthMeasureSpec, heightMeasureSpec)
        }
        val w = MeasureSpec.getSize(widthMeasureSpec)
        val h = MeasureSpec.getSize(heightMeasureSpec)
        setMeasuredDimension(w, h)
    }

    override fun onLayout(changed: Boolean, l: Int, t: Int, r: Int, b: Int) {
        // Position children based on their stored positions
        for (i in 0 until childCount) {
            val child = getChildAt(i) as NodeView
            val left = child.position.x.toInt()
            val top = child.position.y.toInt()
            child.layout(left, top, left + child.measuredWidth, top + child.measuredHeight)
        }
    }

    override fun dispatchDraw(canvas: Canvas) {
        super.dispatchDraw(canvas)
        // Draw lines connecting parent and children
        for (i in 0 until childCount) {
            val child = getChildAt(i) as NodeView
            child.parentNode?.let { parent ->
                canvas.drawLine(
                    parent.position.x + parent.width / 2,
                    parent.position.y + parent.height / 2,
                    child.position.x + child.width / 2,
                    child.position.y + child.height / 2,
                    linePaint
                )
            }
        }
    }

    // Drag support
    private var draggingNode: NodeView? = null
    private var dragStartPoint = PointF()
    private var nodeStartPosition = PointF()

    override fun onTouchEvent(event: MotionEvent): Boolean {
        when (event.actionMasked) {
            MotionEvent.ACTION_DOWN -> {
                // Check if touch is on any node
                val node = findNodeUnder(event.x, event.y)
                if (node != null) {
                    draggingNode = node
                    dragStartPoint = PointF(event.x, event.y)
                    nodeStartPosition = PointF(node.position.x, node.position.y)
                    return true
                }
            }
            MotionEvent.ACTION_MOVE -> {
                draggingNode?.let {
                    val dx = event.x - dragStartPoint.x
                    val dy = event.y - dragStartPoint.y
                    it.position.x = max(0f, min(width - it.width.toFloat(), nodeStartPosition.x + dx))
                    it.position.y = max(0f, min(height - it.height.toFloat(), nodeStartPosition.y + dy))
                    requestLayout()
                    invalidate()
                    return true
                }
            }
            MotionEvent.ACTION_UP, MotionEvent.ACTION_CANCEL -> {
                draggingNode = null
            }
        }
        return super.onTouchEvent(event)
    }

    private fun findNodeUnder(x: Float, y: Float): NodeView? {
        for (i in childCount - 1 downTo 0) {
            val child = getChildAt(i) as NodeView
            val left = child.position.x
            val top = child.position.y
            val right = left + child.width
            val bottom = top + child.height
            if (x in left..right && y in top..bottom) {
                return child
            }
        }
        return null
    }
}
```

---

### Step 2: Create `node_background.xml` (drawable for nodes)

Put in `res/drawable/node_background.xml`:

```xml
<shape xmlns:android="http://schemas.android.com/apk/res/android" >
    <solid android:color="#FFD54F"/>
    <corners android:radius="12dp"/>
    <stroke
        android:width="2dp"
        android:color="#FFA000"/>
</shape>
```

---

### Step 3: Update `activity_main.xml`

Add `MindMapView` and buttons:

```xml
<?xml version="1.0" encoding="utf-8"?>
<FrameLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/rootFrame"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <com.themadbrogrammers.messymind.views.MindMapView
        android:id="@+id/mindMapView"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:background="#FFF" />

    <LinearLayout
        android:orientation="horizontal"
        android:layout_gravity="bottom|center_horizontal"
        android:padding="12dp"
        android:background="#AA000000"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content">

        <Button
            android:id="@+id/btnAddRoot"
            android:text="Add Root"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"/>
        
        <Button
            android:id="@+id/btnExportPng"
            android:text="Export PNG"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"/>

        <Button
            android:id="@+id/btnExportText"
            android:text="Export Text"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"/>
    </LinearLayout>
</FrameLayout>
```

---

### Explanation

* `MindMapView` holds nodes as `TextView`-based `NodeView` objects.
* Nodes can be dragged anywhere inside the view.
* Lines connect parent nodes to children.
* Clicking a node triggers your listener to edit or add child nodes.
* Root node is centered initially.
* Child nodes are offset from their parent by default on attach.

---

### Next steps

* Integrate this `MindMapView` into your `MainActivity` (already prepared in the code I shared).
* You can improve node styles, animations, and add multi-touch gestures later.
* Optionally add zoom/pan for large maps.

---

If you want, I can help you polish this further or provide the full project files with this integrated. Just say!




Sure! Here's a **minimal but complete Android project** integrating the custom `MindMapView` with drag-and-drop nodes and linking. This will give you a working base for your **MessyMind** app with visual mind mapping.

---

# MessyMind Android App — Full Project (Kotlin + Room + MindMapView)

---

## 1. Project Structure

```
app/
├── src/
│   └── main/
│       ├── java/com/themadbrogrammers/messymind/
│       │   ├── MainActivity.kt
│       │   ├── data/
│       │   │   ├── MindMapDatabase.kt
│       │   │   ├── MindNode.kt
│       │   │   ├── MindNodeDao.kt
│       │   │   └── MindNodeRepository.kt
│       │   ├── viewmodels/
│       │   │   └── MindMapViewModel.kt
│       │   ├── views/
│       │   │   └── MindMapView.kt
│       │   └── MessyMindApplication.kt
│       ├── res/
│       │   ├── drawable/
│       │   │   └── node_background.xml
│       │   ├── layout/
│       │   │   └── activity_main.xml
│       │   ├── values/
│       │   │   └── colors.xml
│       └── AndroidManifest.xml
├── build.gradle (app)
└── build.gradle (project)
```

---

## 2. Source Files

### `MessyMindApplication.kt`

```kotlin
package com.themadbrogrammers.messymind

import android.app.Application
import com.themadbrogrammers.messymind.data.MindMapDatabase

class MessyMindApplication : Application() {
    val database by lazy { MindMapDatabase.getDatabase(this) }
}
```

---

### `MainActivity.kt`

```kotlin
package com.themadbrogrammers.messymind

import android.graphics.Bitmap
import android.os.Bundle
import android.widget.Button
import android.widget.Toast
import androidx.activity.viewModels
import androidx.appcompat.app.AppCompatActivity
import androidx.lifecycle.lifecycleScope
import com.themadbrogrammers.messymind.data.MindNode
import com.themadbrogrammers.messymind.views.MindMapView
import com.themadbrogrammers.messymind.viewmodels.MindMapViewModel
import com.themadbrogrammers.messymind.views.MindMapView.NodeView
import kotlinx.coroutines.launch

class MainActivity : AppCompatActivity() {

    private lateinit var mindMapView: MindMapView
    private val viewModel: MindMapViewModel by viewModels()

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        mindMapView = findViewById(R.id.mindMapView)
        val btnAddRoot = findViewById<Button>(R.id.btnAddRoot)
        val btnExportPng = findViewById<Button>(R.id.btnExportPng)
        val btnExportText = findViewById<Button>(R.id.btnExportText)

        mindMapView.addNodeEventListener(object : MindMapView.NodeEventListener {
            override fun onNodeClick(node: NodeView) {
                Toast.makeText(this@MainActivity, "Clicked: ${node.text}", Toast.LENGTH_SHORT).show()
                // TODO: Open dialog to edit or add child node here
            }
        })

        btnAddRoot.setOnClickListener {
            // For demo, add fixed root node
            val rootNode = MindNode(id = 1, title = "Root Node", parentId = null, userData = 1)
            mindMapView.setRootNode(rootNode)
        }

        btnExportPng.setOnClickListener {
            val bitmap = exportMindMapAsBitmap()
            // TODO: Save or share bitmap
            Toast.makeText(this, "Exported mind map as PNG (bitmap)", Toast.LENGTH_SHORT).show()
        }

        btnExportText.setOnClickListener {
            val text = exportMindMapAsText()
            // TODO: Share or save text
            Toast.makeText(this, "Exported mind map as text:\n$text", Toast.LENGTH_LONG).show()
        }
    }

    private fun exportMindMapAsBitmap(): Bitmap {
        val bitmap = Bitmap.createBitmap(
            mindMapView.width,
            mindMapView.height,
            Bitmap.Config.ARGB_8888
        )
        val canvas = android.graphics.Canvas(bitmap)
        mindMapView.draw(canvas)
        return bitmap
    }

    private fun exportMindMapAsText(): String {
        // Simple example: list all nodes visible
        val sb = StringBuilder()
        for (i in 0 until mindMapView.childCount) {
            val node = mindMapView.getChildAt(i) as NodeView
            sb.append("- ${node.text}\n")
        }
        return sb.toString()
    }
}
```

---

### `MindMapView.kt`

```kotlin
package com.themadbrogrammers.messymind.views

import android.content.Context
import android.graphics.Canvas
import android.graphics.Paint
import android.graphics.PointF
import android.util.AttributeSet
import android.view.MotionEvent
import android.view.ViewGroup
import android.widget.TextView
import androidx.core.content.ContextCompat
import com.themadbrogrammers.messymind.R
import kotlin.math.max
import kotlin.math.min

class MindMapView @JvmOverloads constructor(
    context: Context, attrs: AttributeSet? = null
) : ViewGroup(context, attrs) {

    private val linePaint = Paint(Paint.ANTI_ALIAS_FLAG).apply {
        color = ContextCompat.getColor(context, R.color.purple_700)
        strokeWidth = 5f
    }

    interface NodeEventListener {
        fun onNodeClick(node: NodeView)
    }
    private var nodeClickListener: NodeEventListener? = null

    fun addNodeEventListener(listener: NodeEventListener) {
        nodeClickListener = listener
    }

    inner class NodeView(context: Context, val id: Int, title: String) : TextView(context) {
        var parentNode: NodeView? = null
        val children = mutableListOf<NodeView>()
        var position = PointF(0f, 0f)

        init {
            text = title
            setBackgroundResource(R.drawable.node_background)
            setPadding(30, 20, 30, 20)
            setOnClickListener {
                nodeClickListener?.onNodeClick(this)
            }
        }
    }

    private var rootNodeView: NodeView? = null

    fun setRootNode(node: com.themadbrogrammers.messymind.data.MindNode) {
        removeAllViews()
        rootNodeView = NodeView(context, node.userData as Int, node.title).apply {
            position = PointF(500f, 200f)
        }
        addView(rootNodeView)
        requestLayout()
        invalidate()
    }

    fun attachChild(parent: NodeView, childNode: com.themadbrogrammers.messymind.data.MindNode) {
        val childView = NodeView(context, childNode.userData as Int, childNode.title).apply {
            parentNode = parent
            position = PointF(parent.position.x + 300f, parent.position.y + (parent.children.size * 200f))
        }
        parent.children.add(childView)
        addView(childView)
        requestLayout()
        invalidate()
    }

    override fun onMeasure(widthMeasureSpec: Int, heightMeasureSpec: Int) {
        for (i in 0 until childCount) {
            val child = getChildAt(i)
            measureChild(child, widthMeasureSpec, heightMeasureSpec)
        }
        val w = MeasureSpec.getSize(widthMeasureSpec)
        val h = MeasureSpec.getSize(heightMeasureSpec)
        setMeasuredDimension(w, h)
    }

    override fun onLayout(changed: Boolean, l: Int, t: Int, r: Int, b: Int) {
        for (i in 0 until childCount) {
            val child = getChildAt(i) as NodeView
            val left = child.position.x.toInt()
            val top = child.position.y.toInt()
            child.layout(left, top, left + child.measuredWidth, top + child.measuredHeight)
        }
    }

    override fun dispatchDraw(canvas: Canvas) {
        super.dispatchDraw(canvas)
        for (i in 0 until childCount) {
            val child = getChildAt(i) as NodeView
            child.parentNode?.let { parent ->
                canvas.drawLine(
                    parent.position.x + parent.width / 2,
                    parent.position.y + parent.height / 2,
                    child.position.x + child.width / 2,
                    child.position.y + child.height / 2,
                    linePaint
                )
            }
        }
    }

    private var draggingNode: NodeView? = null
    private var dragStartPoint = PointF()
    private var nodeStartPosition = PointF()

    override fun onTouchEvent(event: MotionEvent): Boolean {
        when (event.actionMasked) {
            MotionEvent.ACTION_DOWN -> {
                val node = findNodeUnder(event.x, event.y)
                if (node != null) {
                    draggingNode = node
                    dragStartPoint = PointF(event.x, event.y)
                    nodeStartPosition = PointF(node.position.x, node.position.y)
                    return true
                }
            }
            MotionEvent.ACTION_MOVE -> {
                draggingNode?.let {
                    val dx = event.x - dragStartPoint.x
                    val dy = event.y - dragStartPoint.y
                    it.position.x = max(0f, min(width - it.width.toFloat(), nodeStartPosition.x + dx))
                    it.position.y = max(0f, min(height - it.height.toFloat(), nodeStartPosition.y + dy))
                    requestLayout()
                    invalidate()
                    return true
                }
            }
            MotionEvent.ACTION_UP, MotionEvent.ACTION_CANCEL -> {
                draggingNode = null
            }
        }
        return super.onTouchEvent(event)
    }

    private fun findNodeUnder(x: Float, y: Float): NodeView? {
        for (i in childCount - 1 downTo 0) {
            val child = getChildAt(i) as NodeView
            val left = child.position.x
            val top = child.position.y
            val right = left + child.width
            val bottom = top + child.height
            if (x in left..right && y in top..bottom) {
                return child
            }
        }
        return null
    }
}
```

---

### `MindNode.kt`

```kotlin
package com.themadbrogrammers.messymind.data

import androidx.room.Entity
import androidx.room.PrimaryKey

@Entity(tableName = "mind_nodes")
data class MindNode(
    @PrimaryKey(autoGenerate = true) val id: Int = 0,
    val title: String,
    val parentId: Int? = null,
    val userData: Any? = null
)
```

---

### `MindNodeDao.kt`

```kotlin
package com.themadbrogrammers.messymind.data

import androidx.room.Dao
import androidx.room.Insert
import androidx.room.Query
import androidx.room.Update

@Dao
interface MindNodeDao {
    @Insert
    suspend fun insert(node: MindNode): Long

    @Update
    suspend fun update(node: MindNode)

    @Query("SELECT * FROM mind_nodes WHERE parentId IS NULL")
    suspend fun getRootNodes(): List<MindNode>

    @Query("SELECT * FROM mind_nodes WHERE parentId = :parentId")
    suspend fun getChildNodes(parentId: Int): List<MindNode>

    @Query("SELECT * FROM mind_nodes")
    suspend fun getAllNodes(): List<MindNode>
}
```

---

### `MindMapDatabase.kt`

```kotlin
package com.themadbrogrammers.messymind.data

import android.content.Context
import androidx.room.Database
import androidx.room.Room
import androidx.room.RoomDatabase

@Database(entities = [MindNode::class], version = 1)
abstract class MindMapDatabase : RoomDatabase() {
    abstract fun mindNodeDao(): MindNodeDao

    companion object {
        @Volatile
        private var INSTANCE: MindMapDatabase? = null

        fun getDatabase(context: Context): MindMapDatabase {
            return INSTANCE ?: synchronized(this) {
                val instance = Room.databaseBuilder(
                    context.applicationContext,
                    MindMapDatabase::class.java,
                    "mindmap_database"
                ).build()
                INSTANCE = instance
                instance
            }
        }
    }
}
```

---

### `MindNodeRepository.kt`

```kotlin
package com.themadbrogrammers.messymind.data

class MindNodeRepository(private val dao: MindNodeDao) {

    suspend fun insertNode(node: MindNode) = dao.insert(node)

    suspend fun updateNode(node: MindNode) = dao.update(node)

    suspend fun getRootNodes() = dao.getRootNodes()

    suspend fun getChildNodes(parentId: Int) = dao.getChildNodes(parentId)

    suspend fun getAllNodes() = dao.getAllNodes()
}
```

---

### `MindMapViewModel.kt`

```kotlin
package com.themadbrogrammers.messymind.viewmodels

import android.app.Application
import androidx.lifecycle.AndroidViewModel
import androidx.lifecycle.viewModelScope
import com.themadbrogrammers.messymind.data.MindMapDatabase
import com.themadbrogrammers.messymind.data.MindNode
import com.themadbrogrammers.messymind.data.MindNodeRepository
import kotlinx.coroutines.Dispatchers
import kotlinx.coroutines.launch

class MindMapViewModel(application: Application) : AndroidViewModel(application) {
    private val repository: MindNodeRepository

    init {
        val dao = MindMapDatabase.getDatabase(application).mindNodeDao()
        repository = MindNodeRepository(dao)
    }

    fun insertNode(node: MindNode) = viewModelScope.launch(Dispatchers.IO) {
        repository.insertNode(node)
    }

    fun updateNode(node: MindNode) = viewModelScope.launch(Dispatchers.IO) {
        repository.updateNode(node)
    }
}
```

---

## 3. Resources

### `res/drawable/node_background.xml`

```xml
<shape xmlns:android="http://schemas.android.com/apk/res/android" >
    <solid android:color="#FFD54F"/>
    <corners android:radius="12dp"/>
    <stroke
        android:width="2dp"
        android:color="#FFA000"/>
</shape>
```

---

### `res/layout/activity_main.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<FrameLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/rootFrame"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <com.themadbrogrammers.messymind.views.MindMapView
        android:id="@+id/mindMapView"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:background="#FFFFFF" />

    <LinearLayout
        android:orientation="horizontal"
        android:layout_gravity="bottom|center_horizontal"
        android:padding="12dp"
        android:background="#AA000000"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content">

        <Button
            android:id="@+id/btnAddRoot"
            android:text="Add Root"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"/>
        
        <Button
            android:id="@+id/btnExportPng"
            android:text="Export PNG"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"/>

        <Button
            android:id="@+id/btnExportText"
            android:text="Export Text"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"/>
    </LinearLayout>
</FrameLayout>
```

---

### `res/values/colors.xml`

```xml
<resources>
    <color name="purple_700">#6200EE</color>
</resources>
```

---

## 4. Manifest

### `AndroidManifest.xml`

```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.themadbrogrammers.messymind">

    <application
        android:name=".MessyMindApplication"
        android:allowBackup="true"
        android:label="MessyMind"
        android:icon="@mipmap/ic_launcher"
        android:roundIcon="@mipmap/ic_launcher_round"
        android:supportsRtl="true"
        android:theme="@style/Theme.AppCompat.Light.NoActionBar">

        <activity android:name=".MainActivity"
            android:exported="true">
            <intent-filter>
                <action android:name="android.intent.action.MAIN"/>
                <category android:name="android.intent.category.LAUNCHER"/>
            </intent-filter>
        </activity>

    </application>

</manifest>
```

---

## 5. Gradle (app/build.gradle)

Make sure you have these dependencies:

```gradle
plugins {
    id 'com.android.application'
    id 'kotlin-android'
    id 'kotlin-kapt'
}

android {
    compileSdk 33

    defaultConfig {
        applicationId "com.themadbrogrammers.messymind"
        minSdk 21
        targetSdk 33
        versionCode 1
        versionName "1.0"
    }

    buildFeatures {
        viewBinding true
    }

    compileOptions {
        sourceCompatibility JavaVersion.VERSION_1_8
        targetCompatibility JavaVersion.VERSION_1_8
    }

    kotlinOptions {
        jvmTarget = "1.8"
    }
}

dependencies {
    implementation "org.jetbrains.kotlin:kotlin-stdlib:1.9.0"
    implementation "androidx.core:core-ktx:1.10.1"
    implementation "androidx.appcompat:appcompat:1.6.1"
    implementation "com.google.android.material:material:1.9.0"

    // Lifecycle and ViewModel
    implementation "androidx.lifecycle:lifecycle-viewmodel-ktx:2.6.1"
    implementation "androidx.lifecycle:lifecycle-livedata-ktx:2.6.1"

    // Room components
    implementation "androidx.room:room-runtime:2.5.2"
    kapt "androidx.room:room-compiler:2.5.2"
    implementation "androidx.room:room-ktx:2.5.2"
}
```

---

## What You Get

* A **drag-and-drop visual mind map** of nodes with lines connecting parents to children.
* Nodes as tappable TextViews with background and padding.
* Basic buttons for adding a root node and exporting the map


Let's now add **zoom and pan support** to your custom `MindMapView` for handling large maps!

---

## ✅ Features Added

* **Pinch-to-zoom** (multi-touch scaling)
* **Two-finger pan** (drag the entire canvas)
* **Proper scaling + translation in `onDraw()` and layout**

---

## 🔧 Step-by-step Code Update

### ✅ Update `MindMapView.kt` — Add Scaling and Panning

**Add these properties at the top:**

```kotlin
private var scaleFactor = 1.0f
private var translationX = 0f
private var translationY = 0f

private var lastTouchX = 0f
private var lastTouchY = 0f
private var activePointerId = MotionEvent.INVALID_POINTER_ID

private var lastDistance = 0f
```

---

### ✅ Modify `dispatchDraw` to apply canvas transformations

Replace `dispatchDraw` with:

```kotlin
override fun dispatchDraw(canvas: Canvas) {
    canvas.save()

    // Apply scaling and panning
    canvas.translate(translationX, translationY)
    canvas.scale(scaleFactor, scaleFactor)

    // Draw connecting lines between parent and children
    for (i in 0 until childCount) {
        val child = getChildAt(i) as NodeView
        child.parentNode?.let { parent ->
            canvas.drawLine(
                parent.position.x + parent.width / 2,
                parent.position.y + parent.height / 2,
                child.position.x + child.width / 2,
                child.position.y + child.height / 2,
                linePaint
            )
        }
    }

    super.dispatchDraw(canvas)
    canvas.restore()
}
```

---

### ✅ Update `onTouchEvent` to support pan & zoom gestures

Replace the entire `onTouchEvent` function with:

```kotlin
override fun onTouchEvent(event: MotionEvent): Boolean {
    when (event.pointerCount) {
        1 -> handleSingleTouch(event)
        2 -> handleMultiTouch(event)
    }

    if (event.actionMasked == MotionEvent.ACTION_UP || event.actionMasked == MotionEvent.ACTION_CANCEL) {
        draggingNode = null
        activePointerId = MotionEvent.INVALID_POINTER_ID
    }

    return true
}
```

---

### ✅ Add helper methods for touch:

```kotlin
private fun handleSingleTouch(event: MotionEvent) {
    when (event.actionMasked) {
        MotionEvent.ACTION_DOWN -> {
            val x = (event.x - translationX) / scaleFactor
            val y = (event.y - translationY) / scaleFactor
            val node = findNodeUnder(x, y)
            if (node != null) {
                draggingNode = node
                dragStartPoint = PointF(x, y)
                nodeStartPosition = PointF(node.position.x, node.position.y)
            } else {
                lastTouchX = event.x
                lastTouchY = event.y
                activePointerId = event.getPointerId(0)
            }
        }

        MotionEvent.ACTION_MOVE -> {
            draggingNode?.let {
                val x = (event.x - translationX) / scaleFactor
                val y = (event.y - translationY) / scaleFactor
                val dx = x - dragStartPoint.x
                val dy = y - dragStartPoint.y
                it.position.x = nodeStartPosition.x + dx
                it.position.y = nodeStartPosition.y + dy
                requestLayout()
                invalidate()
            } ?: run {
                val dx = event.x - lastTouchX
                val dy = event.y - lastTouchY
                translationX += dx
                translationY += dy
                lastTouchX = event.x
                lastTouchY = event.y
                invalidate()
            }
        }
    }
}

private fun handleMultiTouch(event: MotionEvent) {
    if (event.pointerCount < 2) return

    val x1 = event.getX(0)
    val y1 = event.getY(0)
    val x2 = event.getX(1)
    val y2 = event.getY(1)

    val currentDistance = Math.hypot((x2 - x1).toDouble(), (y2 - y1).toDouble()).toFloat()

    if (lastDistance != 0f) {
        val scaleChange = currentDistance / lastDistance
        scaleFactor *= scaleChange
        scaleFactor = scaleFactor.coerceIn(0.5f, 3f)
        invalidate()
    }

    lastDistance = currentDistance
}
```

---

### 🧪 Test & Notes

* You can zoom between **0.5x to 3x** (adjustable)
* Use one finger to drag nodes, two fingers to move the canvas or zoom
* This works well with all existing node positioning logic

---

### ✅ Optional Improvements

* Add double-tap to reset zoom
* Add inertia scroll for pan
* Show mini-map / scale indicator

---

Would you like me to now help add **custom dialogs for editing nodes**, or focus on **saving/loading node positions with Room**?


Here’s the fully updated **MindMapView\.kt** with integrated **pan & zoom support**, ready to replace your existing implementation:

---

### `MindMapView.kt`

```kotlin
package com.themadbrogrammers.messymind.views

import android.content.Context
import android.graphics.Canvas
import android.graphics.Paint
import android.graphics.PointF
import android.util.AttributeSet
import android.view.MotionEvent
import android.view.ViewGroup
import android.widget.TextView
import androidx.core.content.ContextCompat
import com.themadbrogrammers.messymind.R
import kotlin.math.max
import kotlin.math.min

class MindMapView @JvmOverloads constructor(
    context: Context, attrs: AttributeSet? = null
) : ViewGroup(context, attrs) {

    private val linePaint = Paint(Paint.ANTI_ALIAS_FLAG).apply {
        color = ContextCompat.getColor(context, R.color.purple_700)
        strokeWidth = 5f
    }

    interface NodeEventListener {
        fun onNodeClick(node: NodeView)
    }
    private var nodeClickListener: NodeEventListener? = null

    fun addNodeEventListener(listener: NodeEventListener) {
        nodeClickListener = listener
    }

    inner class NodeView(context: Context, val id: Int, title: String) : TextView(context) {
        var parentNode: NodeView? = null
        val children = mutableListOf<NodeView>()
        var position = PointF(0f, 0f)

        init {
            text = title
            setBackgroundResource(R.drawable.node_background)
            setPadding(30, 20, 30, 20)
            setOnClickListener {
                nodeClickListener?.onNodeClick(this)
            }
        }
    }

    private var rootNodeView: NodeView? = null

    // Scaling & panning
    private var scaleFactor = 1.0f
    private var translationX = 0f
    private var translationY = 0f
    private var lastTouchX = 0f
    private var lastTouchY = 0f
    private var activePointerId = MotionEvent.INVALID_POINTER_ID
    private var lastDistance = 0f

    private var draggingNode: NodeView? = null
    private var dragStartPoint = PointF()
    private var nodeStartPosition = PointF()

    fun setRootNode(node: com.themadbrogrammers.messymind.data.MindNode) {
        removeAllViews()
        rootNodeView = NodeView(context, node.userData as Int, node.title).apply {
            position = PointF(500f, 200f)
        }
        addView(rootNodeView)
        requestLayout()
        invalidate()
    }

    fun attachChild(parent: NodeView, childNode: com.themadbrogrammers.messymind.data.MindNode) {
        val childView = NodeView(context, childNode.userData as Int, childNode.title).apply {
            parentNode = parent
            position = PointF(parent.position.x + 300f, parent.position.y + (parent.children.size * 200f))
        }
        parent.children.add(childView)
        addView(childView)
        requestLayout()
        invalidate()
    }

    override fun onMeasure(widthMeasureSpec: Int, heightMeasureSpec: Int) {
        for (i in 0 until childCount) {
            val child = getChildAt(i)
            measureChild(child, widthMeasureSpec, heightMeasureSpec)
        }
        setMeasuredDimension(MeasureSpec.getSize(widthMeasureSpec), MeasureSpec.getSize(heightMeasureSpec))
    }

    override fun onLayout(changed: Boolean, l: Int, t: Int, r: Int, b: Int) {
        for (i in 0 until childCount) {
            val child = getChildAt(i) as NodeView
            val left = child.position.x.toInt()
            val top = child.position.y.toInt()
            child.layout(left, top, left + child.measuredWidth, top + child.measuredHeight)
        }
    }

    override fun dispatchDraw(canvas: Canvas) {
        canvas.save()
        canvas.translate(translationX, translationY)
        canvas.scale(scaleFactor, scaleFactor)

        for (i in 0 until childCount) {
            val child = getChildAt(i) as NodeView
            child.parentNode?.let { p ->
                canvas.drawLine(
                    p.position.x + p.width / 2,
                    p.position.y + p.height / 2,
                    child.position.x + child.width / 2,
                    child.position.y + child.height / 2,
                    linePaint
                )
            }
        }

        super.dispatchDraw(canvas)
        canvas.restore()
    }

    override fun onTouchEvent(event: MotionEvent): Boolean {
        when (event.pointerCount) {
            1 -> handleSingleTouch(event)
            2 -> handleMultiTouch(event)
        }
        if (event.actionMasked in listOf(MotionEvent.ACTION_UP, MotionEvent.ACTION_CANCEL)) {
            draggingNode = null
            activePointerId = MotionEvent.INVALID_POINTER_ID
        }
        return true
    }

    private fun handleSingleTouch(event: MotionEvent) {
        when (event.actionMasked) {
            MotionEvent.ACTION_DOWN -> {
                val x = (event.x - translationX) / scaleFactor
                val y = (event.y - translationY) / scaleFactor
                val node = findNodeUnder(x, y)
                if (node != null) {
                    draggingNode = node
                    dragStartPoint = PointF(x, y)
                    nodeStartPosition = PointF(node.position.x, node.position.y)
                } else {
                    lastTouchX = event.x
                    lastTouchY = event.y
                    activePointerId = event.getPointerId(0)
                }
            }

            MotionEvent.ACTION_MOVE -> {
                draggingNode?.let {
                    val x = (event.x - translationX) / scaleFactor
                    val y = (event.y - translationY) / scaleFactor
                    val dx = x - dragStartPoint.x
                    val dy = y - dragStartPoint.y
                    it.position.x = nodeStartPosition.x + dx
                    it.position.y = nodeStartPosition.y + dy
                    requestLayout()
                    invalidate()
                } ?: run {
                    val dx = event.x - lastTouchX
                    val dy = event.y - lastTouchY
                    translationX += dx
                    translationY += dy
                    lastTouchX = event.x
                    lastTouchY = event.y
                    invalidate()
                }
            }
        }
    }

    private fun handleMultiTouch(event: MotionEvent) {
        if (event.pointerCount < 2) return
        val x1 = event.getX(0)
        val y1 = event.getY(0)
        val x2 = event.getX(1)
        val y2 = event.getY(1)
        val currentDistance = kotlin.math.hypot((x2 - x1), (y2 - y1))
        if (lastDistance != 0f) {
            val scaleChange = currentDistance / lastDistance
            scaleFactor = (scaleFactor * scaleChange).coerceIn(0.5f, 3f)
            invalidate()
        }
        lastDistance = currentDistance
    }

    private fun findNodeUnder(x: Float, y: Float): NodeView? {
        for (i in childCount - 1 downTo 0) {
            val c = getChildAt(i) as NodeView
            val left = c.position.x
            val top = c.position.y
            if (x in left..left + c.width && y in top..top + c.height) return c
        }
        return null
    }
}
```

---

### ✅ Integration Steps

1. **Replace** your old `MindMapView.kt` with this file.
2. Ensure `activity_main.xml` references this view:

   ```xml
   <com.themadbrogrammers.messymind.views.MindMapView
       android:id="@+id/mindMapView"
       ...
   />
   ```
3. Update your `MainActivity.kt` to use methods `setRootNode(...)`, `attachChild(...)` as before.

---

### ✅ Next Up:

* **Node editing dialogs**: tap to edit or attach children.
* **Saving node positions & scale in Room**.
* **Draggable zoom reset/double-tap reset**.
* **Pet brain animations & AdMob ads integration**.

Let me know which feature you'd like to tackle next!
