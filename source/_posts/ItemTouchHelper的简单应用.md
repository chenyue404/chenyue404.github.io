---
title: ItemTouchHelper的简单应用
date: 2020-02-23 13:00:00
urlname: use_itemtouchhelper_simplely
tags: 
  Android
categories: 
  开发
---

新需求，表格显示照片，并可以拖动排序。
第一反应，RecyclerView啊。那拖动怎么办？搜了一下，系统提供了ItemTouchHelper，直接用就行了。下面简单记录一下使用方法，及过程中遇到的一些问题。
<!-- more -->
## 使用方法
### 首先先写好一个RecyclerView。
以下是adapter的关键代码：固定第一个item用来显示拍照，其他item显示传入的数据
```Kotlin
override fun getItemCount(): Int {
    //加一显示第一个拍照item
  return 1 + if (dataArray.isNullOrEmpty()) {
        0
  } else {
        dataArray.size
  }
}

override fun onBindViewHolder(holder: ViewHolder, position: Int) {
    //第一个固定显示拍照
  if (position == 0) {
        holder.tv_content.text = "拍照"
  holder.tv_content.setBackgroundColor(
            ContextCompat.getColor(
                holder.itemView.context,
                android.R.color.black
  )
        )
        return
  }

    val dataPos = position - 1;
    val text = dataArray[dataPos][0].toString()
    val bgColor = ContextCompat.getColor(
        holder.itemView.context,
        dataArray[dataPos][1]
    )
    holder.tv_content.text = text
    holder.tv_content.setBackgroundColor(bgColor)
}
```

### 将ItemTouchHelper附加到RecyclerView上
那我们就需要new一个ItemTouchHelper，而ItemTouchHelper需要传入一个ItemTouchHelper.Callback，接下来新建一个类继承子ItemTouchHelper.Callback
```Kotlin
class MyItemTouchHelperCallback(val listener: MyTouchHelperListener) : ItemTouchHelper.Callback() {
    override fun getMovementFlags(
        recyclerView: RecyclerView,
        viewHolder: RecyclerView.ViewHolder
    ): Int {
        val dragFlags =
            ItemTouchHelper.UP or ItemTouchHelper.DOWN or ItemTouchHelper.LEFT or ItemTouchHelper.RIGHT
        val swipeFlags = ItemTouchHelper.LEFT
        return makeMovementFlags(dragFlags, 0)
    }

    override fun onMove(
        recyclerView: RecyclerView,
        viewHolder: RecyclerView.ViewHolder,
        target: RecyclerView.ViewHolder
    ): Boolean {
        listener?.onMove(viewHolder.adapterPosition, target.adapterPosition)
        return true
  }

    override fun onSwiped(viewHolder: RecyclerView.ViewHolder, direction: Int) {
    }

    interface MyTouchHelperListener {
        fun onMove(fromPos: Int, toPos: Int)
    }
}
```
必须重写3个方法：
1. getMovementFlags
控制Item往哪些方向可以拖，往哪些方向可以滑
2. onMove
Item移动的时候会被调用。我们在这里把它传到adapter里面，进行Item的交换。创建Listener，并在adapter里面实现
```Kotlin
class ListAdapter(private val dataArray: Array<Array<Int>>) :
    RecyclerView.Adapter<ListAdapter.ViewHolder>(),
    MyItemTouchHelperCallback.MyTouchHelperListener {
    //省略
    override fun onMove(fromPos: Int, toPos: Int) {
        notifyItemMoved(fromPos, toPos)
    }
}
```
3. onSwiped
Item滑动的时候被调用。这个例子用不到。
最后再把ItemTouchHelp附加到RecyclerView上
```Kotlin
ItemTouchHelper(MyItemTouchHelperCallback(listAdapter)).attachToRecyclerView(rv_list)
```

好的，到这里为止，我们已经实现了拖拽交换item了。那么接下来还有哪些问题呢？

## 各种问题

### onMove一直被调用
拖拽交换item不是UI交换就完事了，数据也是需要变的。然而我们发现这个onMove方法一直在调用，本地临时数据倒无所谓，一直更改也没什么，但是我们是需要提交网络的，总不能一直提交吧？
重写ItemTouchHelper.Callback的clearView方法，这个方法会在所有手势和动画完成之后被调用，第一次onMove的时候记录开始的位置，clearView的时候记录结束的位置，然后再处理数据。
MyItemTouchHelperCallback
```Kotlin
override fun clearView(recyclerView: RecyclerView, viewHolder: RecyclerView.ViewHolder) {
    super.clearView(recyclerView, viewHolder)
    listener.onEnd(viewHolder.adapterPosition)
}

interface MyTouchHelperListener {
    fun onMove(fromPos: Int, toPos: Int)
    fun onEnd(endPos: Int)
}
```
ListAdapter
```Kotlin
private var startDataPos = -1 
private var endDataPos = -1 
override fun onMove(fromPos: Int, toPos: Int) {
    notifyItemMoved(fromPos, toPos)
    startDataPos = fromPos - 1
}

override fun onEnd(endPos: Int) {
    endDataPos = endPos - 1;
    //处理数据，脚标归零
    dataArray[startDataPos] = dataArray[endDataPos].also {
      dataArray[endDataPos] = dataArray[startDataPos]
    }
    startDataPos = 0
    endDataPos = 0 
}
```

### 第一个item不能拖动
第一个位置是用来显示拍照的，不能和普通照片一样被拖动，怎么办？
两个办法：
1. 在getMovementFlags里面判断，根据MyTouchHelperListener新增一个方法通过ListAdapter返回bool值，或者根据viewHolder的位置，来判断能不能拖拽。
2. 重写ItemTouchHelper.Callback的isLongPressDragEnabled方法，返回false，禁止长按拖拽。再给item添加长按事件，调用ItemTouchHelper.startDrag来开启拖拽。

### 第一个item不能被移动
第一个位置同样不应该能被其他的item挤走，怎么办？
这个好说，也是两个办法：
1. 重写canDragOver，返回false即可。具体什么时候返回false，参考上面一个问题的方法1
2. 更简单一点，接收到toPos是0，就不交换UI

### 怎么在长按的时候显示删除按钮
要求长按的时候，不仅可以拖动，而且在所有item的右上角显示删除按钮，就像手机桌面一样，怎么整？
更改flag值，notifydatachanged？肯定不行啊，这个方法要把所有item重新测量，布局的，拖拽的手势就打断了，item就回到原位了，而且整个RecyclerView都会闪一下。
解决办法就是，先默认不允许长按拖动，再监听item的长按事件，长按的时候调用ItemTouchHelper.startDrag来开启拖拽。那怎么显示删除按钮呢？直接用RecyclerView来获取所有ViewHolder，直接操作ViewHolder，显示出删除按钮。

---
好的，以上就是我这次使用ItemTouchHelper的过程了。
demo代码在这里[https://github.com/chenyue404/ItemTouchHelperDemo](https://github.com/chenyue404/ItemTouchHelperDemo)