---
layout: post
title: ListView BaseAdapter与多布局 
tags:
- Android
categories: Android ListView
description: ListView BaseAdapter与多布局 
---

上一节说了ListView使用Android API 提供了的Adapter，下面我们自己自定义一个Adapter。一般我们都要继承BaseAdapter来实现自定义。BaseAdapter是以抽象类，我只需要实现其中的几个方法就OK。

1. 准备数据
2. 创建自定义的Adapter 类，并且继承BaseAdapter
3. 构建适配器
4. 把适配器添加到ListView，并显示。

首先准备数据：

创建一个对象：

	public class User {
	    public String name;
	    public String email;
	    public String address;
	}

创建集合数据：

    private List<User> mUsers;
    private void prepareUserList() {
        mUsers = new ArrayList<>();
        for (int i = 0; i < 15; i++) {
            User user = new User();
            user.name = "User Name " + i;
            user.email = "litengit@16" + i + ".com";
            user.address = "火星" + i + "号";
            mUsers.add(user);
        }
    }

然后自定义Adapter：

	public class UserListAdapter extends BaseAdapter {
	
	    private List<User> mUsers = null;
	    private LayoutInflater mInflater = null;
	
	    public UserListAdapter(Context context, List<User> users) {
	        mInflater = LayoutInflater.from(context);
	        mUsers = users;
	    }
	
	    @Override
	    public int getCount() { // ListView 的行数
	        return mUsers == null ? 0 : mUsers.size();
	    }
	
	    @Override
	    public Object getItem(int position) { // ListView 每一行的数据
	        return mUsers == null ? null : mUsers.get(position);
	    }
	
	    @Override
	    public long getItemId(int position) {// ListView每一行的Id
	        return position;
	    }
	
	    @Override
	    public View getView(int position, View convertView, ViewGroup parent) {
	        ViewHolder holder = null;
	        if (convertView == null) { // 对getView进行优化，利用缓存的item，减少不必要的View的创建
	            convertView = mInflater.inflate(R.layout.item_2, parent, false);
	            holder = new ViewHolder(convertView);
	            convertView.setTag(holder);
	        } else {
	            holder = (ViewHolder) convertView.getTag();
	        }
	        //填充每一行的数据
	        User user = mUsers.get(position);
	        holder.mTvName.setText(user.name);
	        holder.mTvEmail.setText(user.email);
	        holder.mTvAddress.setText(user.address);
	        return convertView;
	    }
	
	    /**
	     * 创建一个实体类，用来装每一行的控件,这样我们可以很容易的通过对象调用它们。
	     */
	    private static class ViewHolder {
	        public TextView mTvName;
	        public TextView mTvEmail;
	        public TextView mTvAddress;
	
	        public ViewHolder(View convertView) {
	            mTvName = (TextView) convertView.findViewById(R.id.tvName);
	            mTvEmail = (TextView) convertView.findViewById(R.id.tvEmail);
	            mTvAddress = (TextView) convertView.findViewById(R.id.tvAddress);
	        }
	    }
	}

构建适配器

	UserListAdapter adapter = new UserListAdapter(this,mUsers);

把适配器添加到ListView，并显示。

	mListView.setAdapter(adapter);



### ListView 多布局
ListView 多布局，就是ListView显示多种View，不再想之前每一行都显示一样的View。我们可以通过BaseAdapter的`getItemViewType`和`getViewTypeCount`来实现。

`getItemViewType`:返回布局类型有哪几个类型。类型为int型，且必须从0开始。

`getViewTypeCount`：返回布局类型数量。

我们可以根据布局的类型在`getView()`里做具体的数据填充。

首先我们准备一下数据,包含一个接口Entity,一个分组类Group，一个用户类User:
	
接口Entity:

	public  interface Entity {}

分组类Group:

	public class Group implements Entity {
	    public String name;
	    public String groupId;
	}

用户类User:

	public class User implements Entity {
	    public String name;
	    public String email;
	    public String address;
	}

准备List集合数据：

    private List<Entity> mEntities = null;

    private void prepareUserGroupList() {
        mEntities = new ArrayList<>();
        for (int i = 0; i < 20; i++) {
            Entity entity = null;
            if (i % 2 == 0) {
                entity = new Group();
                ((Group) entity).name = "分组 " + i;
                ((Group) entity).groupId = "" + i;
            } else {
                entity = new User();
                ((User) entity).name = "User Name " + i;
                ((User) entity).email = "litengit@16" + i + ".com";
                ((User) entity).address = "火星" + i + "号";
            }

            mEntities.add(entity);
        }
    }

准备布局：

分组布局：

	<?xml version="1.0" encoding="utf-8"?>
	<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
	    android:orientation="vertical" android:layout_width="match_parent"
	    android:layout_height="match_parent">
	
	    <TextView
	        android:padding="10dp"
	        android:textColor="@color/colorPrimary"
	        android:layout_width="wrap_content"
	        android:layout_height="wrap_content"
	        android:id="@+id/tvGroupName" />
	</LinearLayout>

用户布局：

	<?xml version="1.0" encoding="utf-8"?>
	<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
	    android:layout_width="match_parent"
	    android:layout_height="wrap_content"
	    android:orientation="vertical"
	    android:paddingLeft="10dp">
	
	    <TextView
	        android:id="@+id/tvName"
	        android:layout_width="match_parent"
	        android:layout_height="wrap_content"
	        android:paddingBottom="5dp"
	        android:paddingTop="5dp"
	        android:text="name"
	        android:textSize="18sp" />
	
	    <TextView
	        android:id="@+id/tvEmail"
	        android:layout_width="match_parent"
	        android:layout_height="wrap_content"
	        android:paddingBottom="5dp"
	        android:paddingTop="5dp"
	        android:text="email"
	        android:textSize="18sp" />
	
	    <TextView
	        android:id="@+id/tvAddress"
	        android:layout_width="match_parent"
	        android:layout_height="wrap_content"
	        android:paddingBottom="5dp"
	        android:paddingTop="5dp"
	        android:text="address"
	        android:textSize="18sp" />
	</LinearLayout>

自定义Adapter：

	public class UserGroupListAdapter extends BaseAdapter {
	    private final int COUNT = 2;
	
	    private final int TYPE_GROUP = 0;
	    private final int TYPE_USER = 1;
	
	    private List<Entity> mEntities;
	    private LayoutInflater mInflater;
	
	    public UserGroupListAdapter(Context context, List<Entity> entities) {
	        mEntities = entities;
	        mInflater = LayoutInflater.from(context);
	    }
	
	    @Override
	    public int getCount() { // ListView 的行数
	        return mEntities == null ? 0 : mEntities.size();
	    }
	
	    @Override
	    public Object getItem(int position) { // ListView 每一行的数据
	        return mEntities == null ? null : mEntities.get(position);
	    }
	
	    @Override
	    public long getItemId(int position) {// ListView每一行的Id
	        return position;
	    }
	
	    @Override
	    public int getItemViewType(int position) {
	        int type = TYPE_GROUP;
	        Entity entity = mEntities.get(position);
	        if (entity instanceof Group) {
	            type = TYPE_GROUP;
	        } else if (entity instanceof User) {
	            type = TYPE_USER;
	        }
	        return type;
	    }
	
	    @Override
	    public int getViewTypeCount() {
	        return COUNT;
	    }
	
	    @Override
	    public View getView(int position, View convertView, ViewGroup parent) {
	        ViewHolder holder = null;
	        int type = getItemViewType(position);
	        if (convertView == null) { // 对getView进行优化，利用缓存的item，减少不必要的View的创建
	            switch (type) {
	                case TYPE_GROUP:
	                    convertView = mInflater.inflate(R.layout.group_item, parent, false);
	                    holder = new GroupHolder(convertView);
	                    convertView.setTag(holder);
	                    break;
	                case TYPE_USER:
	                    convertView = mInflater.inflate(R.layout.item_2, parent, false);
	                    holder = new UserHolder(convertView);
	                    convertView.setTag(holder);
	                    break;
	                default:
	                    break;
	            }
	        } else {
	            holder = (ViewHolder) convertView.getTag();
	        }
	
	        if (holder instanceof GroupHolder) {
	            Group group = (Group) mEntities.get(position);
	            ((GroupHolder) holder).tvGroupName.setText(group.name);
	        }else if (holder instanceof UserHolder) {
	            User user = (User) mEntities.get(position);
	            ((UserHolder) holder).mTvName.setText(user.name);
	            ((UserHolder) holder).mTvEmail.setText(user.email);
	            ((UserHolder) holder).mTvAddress.setText(user.address);
	        }
	        return convertView;
	    }
	
	    public static class ViewHolder{
	    }
	
	    /**
	     * 分组布局的ViewHolder
	     */
	    public static class GroupHolder extends ViewHolder{
	        public TextView tvGroupName;
	
	        public GroupHolder(View convertView) {
	            this.tvGroupName = (TextView) convertView.findViewById(R.id.tvGroupName);
	        }
	    }
	
	    /**
	     * 用户布局的ViewHolder
	     */
	    public static class UserHolder extends ViewHolder{
	        public TextView mTvName;
	        public TextView mTvEmail;
	        public TextView mTvAddress;
	
	        public UserHolder(View convertView) {
	            mTvName = (TextView) convertView.findViewById(R.id.tvName);
	            mTvEmail = (TextView) convertView.findViewById(R.id.tvEmail);
	            mTvAddress = (TextView) convertView.findViewById(R.id.tvAddress);
	        }
	    }
	}


构建Adapter并显示：

	UserGroupListAdapter adapter = new UserGroupListAdapter(this, mEntities);
	mListView.setAdapter(adapter);

我们看效果：

![](http://7xrxe7.com1.z0.glb.clouddn.com/multi_layout.gif)
