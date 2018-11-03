# CeilingDemo
With a head at the top of the suspension layout

# 1.使用的过程中会涉及到design包中的控件，请自行了解

# 2.实现的过程中遇到问题可留言解决具体的地址在http://blog.csdn.net/bydbbb/article/details/78709732

# 3.由于项目需求需要，需要一个带有头部的吸顶布局，在网上搜索了好多实现办法，都不太理想，最终使用对RecyclerView添加分割线的方式，重写了RecyclerView的分割线来实现这个悬浮栏。效果比较简单，但是比较实用，下面主要介绍实现过程。
# 首先先贴上效果图：

## Demo
用模拟器运行的效果，鼠标拨动和模拟器太卡等原因，实际效果比效果图更炫哦～～
![](https://img-blog.csdn.net/20171204133548754?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYnlkYmJi/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

## 1.第一步
### 首先要明白数据的数据结构，我们需要将数据分为几组，每一组都是相同的一类，我这里将数据分为了三组，分别为类别一，类别二，类别三然后创建SectionDecoration让其继承 RecyclerView.ItemDecoration，类内部的实现如下：

```java
private PowerGroupListener mGroupListener;
    /**
     * 悬浮栏高度
     */
    private int mGroupHeight = 80;
    /**
     * 是否靠左边
     * true 靠左边
     * false 不靠左
     */
    private boolean isAlignLeft = true;
    /**
    通过构造函数将groupListener传递进来
    */
    private SectionDecoration(PowerGroupListener groupListener) {
        this.mGroupListener = groupListener;
    }
```

### PowerGroupListener 接口的实现：

```java
package byd.com.byd.ceilingdemo.listener;

import android.view.View;

/**
 * author：byd666 on 2017/12/2 15:13
 */
public interface PowerGroupListener {
    //获取每一组的组名
    String getGroupName(int position);
    //得到组View
    View getGroupView(int position);
```
## 2.第二步
### 实现RecyclerView.ItemDecoration里的两个方法：（1）getItemOffsets和（2）onDrawOver，这两个方法是干嘛的呢,(1)查找出RecyclerView的指定item的位移。(2)是给RecyclerView提供适当的装 饰到画布上。由这个方法将任何内容在项目视图绘制。具体实现：
```java
@Override
    public void getItemOffsets(Rect outRect, View view, RecyclerView parent, RecyclerView.State state) {
        super.getItemOffsets(outRect, view, parent, state);
         //得到你添加的分割线View的位置
        int pos = parent.getChildAdapterPosition(view);
         //获取组名
        String groupId = getGroupName(pos);
        if (groupId == null) {return;}
        //只有是同一组的第一个才显示悬浮栏
        if (pos == 0 || isFirstInGroup(pos)) {
            outRect.top = mGroupHeight;
        }
    }
```
### 获取组名的方法：
```java
 /**
     * 获取组名
     * @param position 
     * @return 组名
     */
    private String getGroupName(int position) {
        if (mGroupListener != null) {
            return mGroupListener.getGroupName(position);
        } else {
            return null;
        }
    }
```
### 判断是不是同一组的第一个：
```Java
/**
     * 判断是不是组中的第一个位置
     * 根据前一个组名，判断当前是否为新的组
     */
    private boolean isFirstInGroup(int pos) {
        if (pos == 0) {
            return true;
        } else {
            String prevGroupId = getGroupName(pos - 1);
            String groupId = getGroupName(pos);
            return !TextUtils.equals(prevGroupId, groupId);
        }
    }
```

### 最重要的方法onDrawOver的实现：
##### 代码中注释很清楚，自行理解其绘制过程
```java
@Override
    public void onDrawOver(Canvas c, RecyclerView parent, RecyclerView.State state) {
        super.onDrawOver(c, parent, state);
        //获取单条的数目
        int itemCount = state.getItemCount();
        int childCount = parent.getChildCount();
        //得出距离左边和右边的padding
        int left = parent.getPaddingLeft();
        int right = parent.getWidth() - parent.getPaddingRight();
        //开始绘制
        String preGroupName;
        String currentGroupName = null;
        for (int i = 0; i < childCount; i++) {
            View view = parent.getChildAt(i);
            int position = parent.getChildAdapterPosition(view);
            preGroupName = currentGroupName;
            currentGroupName = getGroupName(position);
            if (currentGroupName == null || TextUtils.equals(currentGroupName, preGroupName))
            {
                continue;
            }
            int viewBottom = view.getBottom();
            //top 决定当前顶部第一个悬浮Group的位置
            int top = Math.max(mGroupHeight, view.getTop());
            if (position + 1 < itemCount) {
                //获取下个GroupName
                String nextGroupName = getGroupName(position + 1);
                //下一组的第一个View接近头部
                if (!currentGroupName.equals(nextGroupName) && viewBottom < top) {
                    top = viewBottom;
                }
            }

            //根据position获取View
            View groupView = getGroupView(position);
            if (groupView == null){ return;}
            ViewGroup.LayoutParams layoutParams = new ViewGroup.LayoutParams(ViewGroup.LayoutParams.WRAP_CONTENT, mGroupHeight);
            groupView.setLayoutParams(layoutParams);
            groupView.setDrawingCacheEnabled(true);
            groupView.measure(
                    View.MeasureSpec.makeMeasureSpec(0, View.MeasureSpec.UNSPECIFIED),
                    View.MeasureSpec.makeMeasureSpec(0, View.MeasureSpec.UNSPECIFIED));
            //指定高度、宽度的groupView
            groupView.layout(0, 0, right, mGroupHeight);
            groupView.buildDrawingCache();
            Bitmap bitmap = groupView.getDrawingCache();
            int marginLeft = isAlignLeft ? 0 : right - groupView.getMeasuredWidth();
            c.drawBitmap(bitmap, left + marginLeft, top - mGroupHeight, null);
        }
    }
```
### getGroupView的方法实现：
```java
/**
     * 获取组View
     * @param position
     * @return 组名
     */
    private View getGroupView(int position) {
        if (mGroupListener != null) {
            return mGroupListener.getGroupView(position);
        } else {
            return null;
        }
    }
```
## 3.第三步
### 为了方便初始化和使用，创建一个类部类，用来 初始化 listener，设置Group高度，确定其是否靠左边：
```java
public static class Builder {
        SectionDecoration mDecoration;
        private Builder(PowerGroupListener listener) {
            mDecoration = new SectionDecoration(listener);
        }
        /**
         * 初始化 listener
         * @param listener
         * @return
         */
        public static Builder init(PowerGroupListener listener) {
            return new Builder(listener);
        }
        /**
         * 设置Group高度
         * @param groutHeight 高度
         * @return this
         */
        public Builder setGroupHeight(int groutHeight) {
            mDecoration.mGroupHeight = groutHeight;
            return this;
        }
        /**
         * 是否靠左边
         * true 靠左边（默认）、false 靠右边
         * @param b b
         * @return  this
         */
        public Builder isAlignLeft(boolean b){
            mDecoration.isAlignLeft = b;
            return this;
        }
        public SectionDecoration build() {
            return mDecoration;
        }
    }
```
### 到这咱们就将分割线创建完成了，具体怎么使用呢，
## 4. 第四步
### 具体的使用 ，为RecyclerView添加分割线即可。
```java
/**
     * 添加悬浮布局
     */
    private void initDecoration() {
        SectionDecoration decoration = SectionDecoration.Builder
                .init(new PowerGroupListener() {
                    @Override
                    public String getGroupName(int position) {
                        //获取组名，用于判断是否是同一组
                        if (dataList.size() > position) {
                            return dataList.get(position).getName();
                        }
                        return null;
                    }
                    @Override
                    public View getGroupView(int position) {
                        //获取自定定义的组View
                        if (dataList.size() > position) {
                            View view = getLayoutInflater().inflate(R.layout.item_group, null, false);
                            ((TextView) view.findViewById(R.id.tv)).setText(dataList.get(position).getName());
                            ((ImageView) view.findViewById(R.id.iv)).setImageResource(dataList.get(position).getIcon());
                            return view;
                        } else {
                            return null;
                        }
                    }
                })
                //设置高度
                .setGroupHeight(ScreenUtil.dip2px(this, 40))
                .build();
        mRecyclerView.addItemDecoration(decoration);
    } 
```
## 最后贴上xml布局
```xml
<?xml version="1.0" encoding="utf-8"?>
<android.support.design.widget.CoordinatorLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:fitsSystemWindows="true">

    <android.support.design.widget.AppBarLayout
        android:id="@+id/app_bar"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:fitsSystemWindows="true"
        android:theme="@style/AppTheme.AppBarOverlay">

        <android.support.design.widget.CollapsingToolbarLayout
            android:id="@+id/toolbar_layout"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:fitsSystemWindows="true"
            app:contentScrim="?attr/colorPrimary"
            app:layout_scrollFlags="scroll|exitUntilCollapsed">

            <LinearLayout
                android:layout_width="match_parent"
                android:layout_height="match_parent"
                android:orientation="vertical"
                app:layout_collapseMode="pin"
                app:popupTheme="@style/AppTheme.PopupOverlay">

                <TextView
                    android:id="@+id/title_tv"
                    android:layout_width="match_parent"
                    android:layout_height="56dp"
                    android:text="我是主标题"
                    android:gravity="center"
                    android:textSize="18sp"/>

                <TextView
                    android:id="@+id/header_tv"
                    android:background="@color/colorAccent"
                    android:layout_width="match_parent"
                    android:layout_height="200dp"
                    android:textColor="#ffffff"
                    android:text="我是头布局"
                    android:gravity="center"
                    android:textSize="16sp"/>
            </LinearLayout>
        </android.support.design.widget.CollapsingToolbarLayout>
    </android.support.design.widget.AppBarLayout>

    <android.support.v7.widget.RecyclerView
        android:id="@+id/rlv"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        app:layout_behavior="@string/appbar_scrolling_view_behavior" />

    <android.support.design.widget.FloatingActionButton
        android:id="@+id/fab"
        android:layout_width="64dp"
        android:layout_height="64dp"
        android:layout_above="@+id/bottom_navigation"
        android:layout_alignParentBottom="true"
        android:layout_alignParentRight="true"
        android:layout_gravity="bottom|right"
        android:layout_marginBottom="32dp"
        android:layout_marginRight="32dp"
        android:src="@mipmap/ic_launcher"
        app:backgroundTint="@color/colorPrimary"
        app:borderWidth="0dp"
        app:elevation="8dp"
        app:fabSize="normal" />
</android.support.design.widget.CoordinatorLayout>
```
## 整个带有头吸顶布局的实现过程就完成了，如果有需要代码的同学，点开clone即可，别忘了star

