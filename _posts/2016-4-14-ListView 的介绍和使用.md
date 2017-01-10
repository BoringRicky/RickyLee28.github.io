---
layout: post
title: ListView 的介绍和使用 
tags:
- Android
categories: Android ListView
description: ListView 的介绍和使用
---


ListView 在Android 开发中是比较常用的控件，ListView 主要用于显示列表。比如常见的新闻列表和商品列表等，虽然现在Google提供了功能更加强大的RecylerView，但是ListView仍然占有一席之地。我们今天就介绍一下ListView。

ListView 常用的属性：

	<!-- 默认的背景滑动缓存色，如果滑动时背景色是黑色，可以这样设置 -->
	android:cacheColorHint="@null"
	<!-- 行点击的selector -->
	android:listSelector="@drawable/selector_drawable"
	<!-- 行之间分割线的颜色 -->
	android:divider="@color/colorPrimaryDark"
	<!-- 行之间分割线的高度 -->
	android:dividerHeight="1dp"
	<!-- 填充ListView每一行的字符串数组 -->
	android:entries="@array/str_arrs"


ListView显示的是列表。它的每一行都是一个View。而且我们需要用它显示的数据大多数是数组或者是集合。我们需要把数组或者集合的每一项数据填充到ListView的每一行中；我们这就需要用Adapter。Adapter不止使用与Listview 也适用于GridView 和 RecyclerView。这里我们只讨论ListView。

使用ListView的一般步骤：
1. 准备数据，一般为集合或者数组
2. 准备ListView每一行显示的View(也可以使用系统提供的View)
3. 构建适配器：将集合或者数组的每项数据填充到ListView每一行
4. 把适配器添加到ListView，并显示。

我们在ListView或者GridView中一般使用的常见的Adapter有ArrayAdapter、SimpleAdapter、CursorAdapter、BaseAdapter等。

##### ArrayAdapter
ArrayAdapter使用比较简单，只需要我提供ListView的每一行的View和填充数据就OK。
ArrayAdapter只能显示单元素的TextView。ArrayAdapter的构造参数有以下几种：
 
	//这个构造，我们一般不用，因为里面提供的数据默认是空的
	ArrayAdapter(Context context, @LayoutRes int resource)
	//这个构造，我们一般也不用，因为里面提供的数据默认是空的
	ArrayAdapter(Context context, @LayoutRes int resource, @IdRes int textViewResourceId)
	//这个构造函数需要一个上下文Context,ListView的行View和一个数组
	ArrayAdapter(Context context, @LayoutRes int resource, @NonNull T[] objects)
	//这个构造函数需要一个上下文Context,ListView的行View布局，布局中的View的id和一个数组
	ArrayAdapter(Context context, @LayoutRes int resource, @IdRes int textViewResourceId,
            @NonNull T[] objects)
	//这个构造函数需要一个上下文Context,ListView的行View和一个集合
	ArrayAdapter(Context context, @LayoutRes int resource, @NonNull List<T> objects)
	//这个构造函数需要一个上下文Context,ListView的行View布局，布局中的View的id和一个集合
	ArrayAdapter(Context context, @LayoutRes int resource, @IdRes int textViewResourceId,
            @NonNull List<T> objects)
	
首先准备数据：

    private void prepareListDatas() {
        strList = new ArrayList();
        for (int i = 0; i < 20; i++) {
            strList.add("集合的 第" + i + "行");
        }
    }

    private void prepareArrDatas() {
        strArr = new String[20];
        for (int i = 0; i < 20; i++) {
            strArr[i] = "数组的 第" + i + "行";
        }
    }


构建适配器,这里我们使用的是系统提供的View `android.R.layout.simple_list_item_1`,用于ListView 的显示,以下分别为使用List和数组来填充数据：

	//填充集合数据
 	ArrayAdapter<String> adapter = new ArrayAdapter<>(this, android.R.layout.simple_list_item_1,strList);

	//填充数组数据
 	ArrayAdapter<String> adapter = new ArrayAdapter<>(this, android.R.layout.simple_list_item_1,strArr);

把适配器添加到ListView，并显示：

	mListView.setAdapter(adapter);

运行结果如下:
![](http://7xrxe7.com1.z0.glb.clouddn.com/arrayadapter.gif)

如果系统提供的布局不喜欢可以自己写一个`item`布局：
	
	<?xml version="1.0" encoding="utf-8"?>
	<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
	    android:orientation="vertical" android:layout_width="match_parent"
	    android:layout_height="match_parent">
	
	    <TextView
	        android:textColor="#0000ee"
	        android:textSize="24sp"
	        android:layout_width="wrap_content"
	        android:layout_height="wrap_content"
	        android:id="@+id/textView" />
	</LinearLayout>
我们的适配器构造如下:	

	ArrayAdapter<String> adapter = new ArrayAdapter<>(this, R.layout.item,R.id.textView,strList);
	mListView.setAdapter(adapter);

效果如下：

![](http://7xrxe7.com1.z0.glb.clouddn.com/arrayadapter2.gif)

剩下的其他的构造用法非常类似，大家可以研究一下。

行点击的事件，我们可以使用 OnItemClickListener：

        mListView.setOnItemClickListener(new AdapterView.OnItemClickListener() {
            @Override
            public void onItemClick(AdapterView<?> parent, View view, int position, long id) {
                Toast.makeText(MainActivity.this, "这是第" + position + "行", Toast.LENGTH_SHORT).show();
            }
        });

#### SimpleAdapter
这个adapter稍微麻烦一下，它可以设置每一行的多个元素，可以完成稍微复杂的界面。它可以根据传入的控件的id，去绑定数据。

它只有一个构造方法:
	
	SimpleAdapter(Context context, List<? extends Map<String, ?>> data, @LayoutRes int resource, String[] from, @IdRes int[] to)

1. Context ：上下文，一般指当前Activity
2. `List<? extends Map<String, ?>>` : 填充List的数据，它的元素为Map，到后面会根据Map的key值去映射控件的id。
3. int resource ：我们的自己定义的布局，用来显示ListView每一行
4. String[] from: 集合元素中的Map中的key
5. int[] to :resource中控件id,根据Map中值跟from中的值来填充数据。

首先我们准备数据：

	private List<Map<String, Object>> mMapList;

    public void prepareMapList() {
        mMapList = new ArrayList<>();
        for (int i = 0; i < 20; i++) {
            Map<String, Object> map = new HashMap<>();
            map.put("header", R.drawable.ic_launcher);
            map.put("name", "name = " + i);
            map.put("age", "age = " + (i * i));
            mMapList.add(map);
        }
    }

然后定义布局：

	<?xml version="1.0" encoding="utf-8"?>
	<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
	    android:layout_width="match_parent"
	    android:layout_height="match_parent"
	    android:orientation="vertical"
	    android:padding="5dp">
	    <ImageView
	        android:id="@+id/ivHeader"
	        android:layout_width="80dp"
	        android:layout_height="80dp"
	        android:src="@drawable/ic_launcher" />
	    <LinearLayout
	        android:layout_width="match_parent"
	        android:layout_height="wrap_content"
	        android:layout_alignBottom="@+id/ivHeader"
	        android:layout_alignTop="@id/ivHeader"
	        android:layout_toRightOf="@+id/ivHeader"
	        android:orientation="vertical"
	        android:paddingLeft="10dp">
	
	        <TextView
	            android:id="@+id/tvName"
	            android:layout_width="match_parent"
	            android:layout_height="wrap_content"
	            android:layout_weight="1"
	            android:gravity="center_vertical"
	            android:text="name"
	            android:textSize="18sp" />
	
	        <TextView
	            android:id="@+id/tvAge"
	            android:layout_width="match_parent"
	            android:layout_height="wrap_content"
	            android:layout_weight="1"
	            android:gravity="center_vertical"
	            android:text="age"
	            android:textSize="18sp"/>
	    </LinearLayout>
	</RelativeLayout>

最后构建适配器，并显示：
	
	String[] from = {"header", "name", "age"};
	int[] to = {R.id.ivHeader, R.id.tvName, R.id.tvAge};
	SimpleAdapter adapter = new SimpleAdapter(this, mMapList, R.layout.item_1, from, to);
	mListView.setAdapter(adapter);

我们看一下效果：

![](http://7xrxe7.com1.z0.glb.clouddn.com/simpleadapter.gif)
